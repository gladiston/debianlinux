# ACESSANDO ARQUIVOS NA REDE

O Linux é muito versátil ao acessar arquivos pela rede. Diferente do Windows onde o compartilhamento de arquivos se dá apenas pelo protocolo smb/cifs do próprio Windows, no Linux, qualquer tipo de compartilhamento que tenha um protocolo de comunicação aberto pode ser montado em forma de pasta em seu sistema.  
Vamos considerar agora alguns tipos de compartilhamentos.

## ACESSAR COMPARTILHAMENTOS NA REDE VIA GERENCIADOR DE ARQUIVOS

Os gerenciadores de arquivos no Linux acessam **redes remotas** de diferentes tipos através de prefixos no início da URI.

Por exemplo, para compartilhamentos **SMB/CIFS**:

> smb://nas01/pub  
> Ou com autenticação:  
> smb://gsantana[:senha]@nas01/pub

E não seria diferente com outros protocolos como **FTP**, **SFTP (FTP sobre SSH)** ou **NFS**, por exemplo:

> sftp://nas01/mnt/po_nas01/pub  
> Ou:  
> sftp://gsantana[:senha]@nas01/mnt/po_nas01/pub

Se você usa KDE ou GNOME, acesse via URI as suas pastas na rede e adicione favoritos ou atalhos para elas, os gerenciadores de arquivos desses ambientes têm esse recurso e fazem o mapeamento apenas quando você tenta acessá-las.

## ACESSAR COMPARTILHAMENTOS NA REDE VIA TERMINAL

Nem tudo se resolve com KDE ou GNOME, e se em algumas situações queremos acessar esses compartilhamentos **pelo terminal**?

Simples — basta criar um ponto de montagem, ou seja, uma pasta onde o conteúdo remoto será exibido:

```bash
sudo mkdir -p /mnt/pub
```

Agora, monte o compartilhamento manualmente:

```bash
sudo mount -t cifs //nas01/pub /mnt/pub -o username=gsantana,password=suasenha,domain=localdomain.lan,users,rw,uid=gsantana,gid=gsantana,nosuid,nodev,file_mode=777,dir_mode=777,iocharset=utf8,vers=3.0,_netdev
```

Esse comando funciona bem, mas é um tanto **longo** para digitar toda vez e para ser sincero, fui rigoroso e coloquei opções demais, isto porque eu não sei qual distro baseada em Debian você pode estar usando e cada uma pode ter seus próprios padrões quando uma opção não é ofertada.  
Como a pasta será usada com frequência, o ideal é torná-la permanente no **`/etc/fstab`**.

### ACESSAR COMPARTILHAMENTOS NA REDE VIA TERMINAL — montagem via `/etc/fstab`

Deixar programado pastas que apontem diretamente para uma unidade de rede nos permite simplificar as montagens de unidades, edite o arquivo a seguir:

```bash
sudo editor /etc/fstab
```

E adicione ao **`/etc/fstab`** a seguinte linha (única linha):

```fstab
//nas01/pub /mnt/pub cifs username=gsantana,password=suasenha,domain=localdomain.lan,users,uid=gsantana,gid=gsantana,rw,nosuid,nodev,file_mode=0777,dir_mode=0777,iocharset=utf8,vers=3.0,_netdev,noauto,x-systemd.automount,x-systemd.mount-timeout=10s 0 0
```

Agora **salve** o arquivo e saia do editor.

Agora você pode simplesmente executar no terminal:

```bash
sudo mount /mnt/pub
```

E nem precisa mais de usar o `sudo` por causa da diretiva **`users`** usada no fstab. Mas você percebeu um problema? Sim, **usuário e senha ficam expostos** e não é uma boa ideia deixar assim, então vamos melhorar isso.

Substitua a linha anterior por esta:

```fstab
# Montagem da pasta pub
//nas01/pub /mnt/pub cifs credentials=/etc/cifs-credentials.gsantana.localdomain.lan,uid=gsantana,gid=gsantana,rw,nosuid,nodev,file_mode=0777,dir_mode=0777,iocharset=utf8,vers=3.0,_netdev 0 0
```

Agora **salve** o arquivo e saia do editor.

No lugar do usuário e senha dentro do `fstab`, informamos um **arquivo externo para autenticação**.  
Vamos criá-lo:

```bash
sudo editor /etc/cifs-credentials.gsantana.localdomain.lan
```

E adicione o conteúdo:

```text
username=gsantana
password=suasenha
domain=localdomain
```

Agora **salve** o arquivo e saia do editor.

Agora, proteja esse arquivo, impedindo que outros leiam suas credenciais:

```bash
sudo chmod 600 /etc/cifs-credentials.gsantana.localdomain.lan
```

Com a permissão acima, veja o que acontece quando um usuário comum tenta ver seu conteúdo:

```bash
cat /etc/cifs-credentials.gsantana.localdomain.lan
```

Saída esperada (exemplo):

```text
cat: /etc/cifs-credentials.gsantana.localdomain.lan: Permissão negada
```

É isso mesmo, usuários comuns são barrados!

Para saber se as credenciais estão certas, é recomendado executar no terminal exatamente o que você incluiu no **`/etc/fstab`**, veja:

```bash
sudo systemctl daemon-reload
sudo mount /mnt/pub
```

Se tudo correu bem, o conteúdo da pasta **`/mnt/pub`** será o mesmo do compartilhamento **`//nas01/pub`**.

Caso apareça o erro:

```text
mount error(13): Permission denied
Refer to the mount.cifs(8) manual page (e.g. man mount.cifs) and kernel log messages (dmesg)
```

significa que o **usuário**, **senha** ou **domínio** estão incorretos.  
Lembre-se: para o campo *domain*, **use apenas o nome curto** (ex.: `localdomain`, e não `localdomain.lan`).

Caso a montagem tenha dado certo, então está pronto para montar no terminal sem precisar informar credenciais, apenas com o comando:

```bash
mount /mnt/pub
```

Se a montagem é um pouco demorada e os compartilhamentos são feitos em hosts com IPs fixos, você pode acelerar a montagem, editando o arquivo **`/etc/hosts`** e adicionando-os à sua lista:

```bash
sudo editor /etc/hosts
```

e garanta que algo como o exemplo abaixo esteja presente:

```text
192.168.1.10 nas01
```

Assim, a montagem será mais rápida e não dependerá apenas de resolução de nomes via rede, mas faça isso apenas para hosts com IPs fixos.

### Montagem automática sob demanda com **x-systemd.automount**

Em sistemas modernos (Debian, Ubuntu, Fedora, etc.), você pode usar o recurso **`x-systemd.automount`** para que o compartilhamento seja montado **somente quando for acessado** — por exemplo, ao abrir **`/mnt/pub`**.

Isso evita que o boot demore caso o servidor remoto esteja desligado ou fora da rede.

Para configurar, edite novamente o **`/etc/fstab`** e adicione a opção `x-systemd.automount`, assim:

```fstab
# Montagem sob demanda da pasta pub
//nas01/pub /mnt/pub cifs credentials=/etc/cifs-credentials.gsantana.localdomain.lan,uid=gsantana,gid=gsantana,rw,nosuid,nodev,file_mode=0777,dir_mode=0777,iocharset=utf8,vers=3.0,_netdev,noauto,x-systemd.automount,x-systemd.mount-timeout=10s 0 0
```

Depois ative o comportamento:

```bash
sudo systemctl daemon-reexec
sudo systemctl restart remote-fs.target
```

A partir de agora, o diretório **`/mnt/pub`** será montado automaticamente, mas apenas quando for acessado e desmontado após um tempo de inatividade.

É uma ótima solução para compartilhamentos remotos que nem sempre estão online.  
Com isso, seu sistema acessará compartilhamentos de rede de forma **segura, automática e inteligente**, seja durante o boot, seja sob demanda, conforme sua necessidade.

O único problema dessa solução é que algumas distros instalam programas que fazem a varredura destas pastas e não intencionalmente fazem a montagem. Se isso ocorre com você, ou seja, elas estão montadas assim que aparecem no gerenciador de arquivos, então você pode escolher deixar assim porque elas também são desmontadas quando não utilizadas ou remove-las do seu **`/etc/fstab`** e passar a montar as pastas diretamente no gerenciador de arquivos usando prefixos como `smb://` ou `sftp://`.

### Documentação dos Parâmetros de Montagem (`fstab` / CIFS)

| Parâmetro | Explicação |
| :--- | :--- |
| **`//nas01/pub`** | Caminho de origem (UNC) do compartilhamento no servidor NAS. |
| **`/mnt/pub`** | Ponto de montagem local onde os arquivos ficarão acessíveis. |
| **`cifs`** | Protocolo de rede utilizado (Common Internet File System / SMB). |
| **`credentials=`** | Caminho para o arquivo que armazena usuário e senha de forma protegida. |
| **`users`** | Permite que usuários comuns realizem a montagem e desmontagem. |
| **`rw`** | Permite leitura e escrita (**read-write**). |
| **`nosuid`** | Impede que programas com o bit SUID sejam executados com privilégios do dono. |
| **`nodev`** | Não permite a criação/interpretação de arquivos de dispositivos. |
| **`file_mode=0777`** | Atribui permissões totais (leitura/escrita/execução) para arquivos. |
| **`dir_mode=0777`** | Define permissões de diretórios (como nos exemplos acima). Não use sticky bit(1777) porque o kernel não gosta dele nas montagens. |
| **`iocharset=utf8`** | Define o conjunto de caracteres como UTF-8 para suportar acentuação. |
| **`vers=3.0`** | Especifica a versão 3.0 do protocolo SMB (segurança e performance melhoradas). |
| **`_netdev`** | Adia a montagem até que a pilha de rede do sistema esteja ativa. |
| **`noauto`** | Indica que o dispositivo não deve ser montado automaticamente no boot. |
| **`x-systemd.automount`** | Ativa a montagem sob demanda (o sistema monta ao acessar a pasta). |
| **`x-systemd.mount-timeout=10s`** | Define um limite de 10 segundos para tentativas de conexão antes de falhar. |
| **`0` (quinto campo)** | **Dump**: desativa o backup automático pelo utilitário dump. |
| **`0` (sexto campo)** | **Fsck**: ignora a verificação de integridade do disco na inicialização. |

---

[Clique aqui para retornar a página principal](../README.md#acessando-arquivos-na-rede)
