# ACESSANDO ARQUIVOS NA REDE
O Linux é muito versátil ao acessar arquivos pela rede. Diferente do Windows onde o compartilhamento de arquivos se dá apenas pelo protocolo smb/cifs do próprio Windows, no Linux, qualquer tipo de compartilhamento que tenha um protocolo de comunicação aberto pode ser montado em forma de pasta em seu sistema.  
Vamos considerar agora alguns tipos de compartilhamentos.  


## ACESSAR COMPARTILHAMENTOS NA REDE VIA GESTOR DE ARQUIVOS
Os gerenciadores de arquivos no Linux acessam **redes remotas** de diferentes tipos através de prefixos no início da URI.  
Por exemplo, para compartilhamentos **SMB/CIFS**:
```
smb://nas01/pub
# ou com autenticação:
smb://gsantana[:senha]@nas01/pub
```

E não seria diferente com outros protocolos como **FTP**, **SFTP (FTP sobre SSH)** ou **NFS**, por exemplo:
```
sftp://nas01/mnt/po_nas01/pub
# ou
sftp://gsantana[:senha]@nas01/mnt/po_nas01/pub
```
Se você usa KDE ou GNOME, acesse via URI as suas pastas na rede e crie bookmarks delas, os gestores de arquivos desses ambientes tem esse recurso e fazem o mapeamento apenas quando você tenta acessá-las.

## ACESSAR COMPARTILHAMENTOS NA REDE VIA TERMINAL
Nem tudo se resolve com KDE ou GNOME, e se em algumas situações queremos acessar esses compartilhamentos **pelo terminal**?  
Simples — basta criar um ponto de montagem, ou seja, uma pasta onde o conteúdo remoto será exibido:
```bash
sudo mkdir -p /media/pub
```

Conceda acesso à pasta para você mesmo, com o bit *setgid* (`2`) para que subpastas herdem as permissões da pasta-pai:
```bash
sudo chown $USER:$USER /media/pub 
sudo chmod 2777 /media/pub
```

Agora, monte o compartilhamento manualmente:
```bash
sudo mount -t cifs //nas01/pub /media/pub -o username=gsantana,password=suasenha,domain=localdomain.lan,users,rw,nosuid,nodev,file_mode=0777,dir_mode=0777
```

Esse comando funciona bem, mas é um tanto **longo** para digitar toda vez.  
Como a pasta será usada com frequência, o ideal é torná-la permanente no **/etc/fstab**.

### ACESSAR COMPARTILHAMENTOS NA REDE VIA TERMINAL - Montagem via `/etc/fstab`
Deixar programado pastas que apontem diretamente para uma unidade de rede nos permite simplificar as montagens de unidades, edite o arquivo a seguir:
```bash
sudo editor /etc/fstab
```

E adicione a linha:
| `/etc/fstab` |
|:--|
| `//nas01/pub /media/pub cifs -o username=gsantana,password=suasenha,domain=localdomain.lan,users,rw,auto,nosuid,nodev,file_mode=0777,dir_mode=0777` |

Agora **salve** o arquivo e feche o editor (`Ctrl+O`, `Enter`, `Ctrl+X`).  

Agora você pode simplesmente, executar no terminal:  
```bash
mount /media/pub
```
E nem precisa mais de usar o `sudo` por causa da diretiva **users** usada no fstab. Mas você percebeu um problema?, sim,  **usuário e senha ficam expostos** e não é uma boa ideia deixar assim, então, vamos melhorar isso.

Substitua a linha anterior por esta:
```
# Montagem da pasta pub 
//nas01/pub /media/pub cifs credentials=/etc/cifs-credentials.gsantana.vidycorp.lan,users,rw,nosuid,nodev,file_mode=0777,dir_mode=0777,noauto 0 0
```
Salve e feche o arquivo (`Ctrl+O`, `Enter`, `Ctrl+X`).   

No lugar do usuário e senha dentro do `fstab`, informamos um **arquivo externo para autenticação**.  
Vamos criá-lo:
```bash
sudo editor /etc/cifs-credentials.gsantana.localdomain.lan
```

E adicione o conteúdo:
```
username=gsantana
password=suasenha
domain=localdomain
```
Salve e feche o arquivo (`Ctrl+O`, `Enter`, `Ctrl+X`).   

Agora, proteja esse arquivo, impedindo que outros leiam suas credenciais:
```bash
sudo chmod 600 /etc/cifs-credentials.gsantana.localdomain.lan
```
Com a permissão acima, veja o que acontece quando um usuário comum tenta ver seu conteúdo:
```bash
$ cat /etc/cifs-credentials.gsantana.localdomain.lan 
cat: /etc/cifs-credentials.gsantana.localdomain.lan: Permissão negada
```
É isso mesmo, usuários comuns são barrados!   
Para saber se as credenciais estão certas, é recomendado executar o terminal exatamente o que você incluiu no /etc/fstab, veja:
```bash
sudo systemctl daemon-reload
sudo mount -t cifs //nas01/pub /media/pub -o credentials=/etc/cifs-credentials.gsantana.localdomain.lan,rw,nosuid,nodev,file_mode=0777,dir_mode=0777
```

Se tudo correu bem, o conteúdo da pasta **/media/pub** será o mesmo do compartilhamento **//nas01/pub**.

Caso apareça o erro:
```
mount error(13): Permission denied
Refer to the mount.cifs(8) manual page (e.g. man mount.cifs) and kernel log messages (dmesg)
```
significa que o **usuário**, **senha** ou **domínio** estão incorretos.  
Lembre-se: para o campo *domain*, **use apenas o nome curto** (ex.: `localdomain`, e não `localdomain.lan`).  

Caso a montagem tenha dado certo, então está pronto para montar no terminal sem necessitar de credencais, apenas com o comando:  
```bash
mount /media/pub
```

Se a montagem é um pouco demorada e os compartilhamentos sao feitos em hosts com IPs fixos, voce pode acelerar a montagem, editando o arquivo /etc/hosts e adicionando-os à sua lista, edite o arquivo:
```
sudo editor /etc/hosts
```
e garanta que algo como o exemplo abaixo esteja presente:
```
192.168.1.10 nas01
```
Assim, a montagem será mais rápida e não dependerá apenas de resolução de nomes via rede, mas faça isso, apenas para hosts com IPs fixos.

✅ **Resumo:**
- `/etc/fstab` define montagens permanentes;  
- `credentials=` evita expor senhas;  
- `chmod 600` protege o arquivo de credenciais;  
- `domain=` é opcional e depende do ambiente;  
- para NTFS, prefira **UUIDs** (LABELs podem causar lentidão ou falhas na inicialização).  

### Montagem automática sob demanda com `systemd.automount`
Em sistemas modernos (Debian, Ubuntu, Fedora, etc.), você pode usar o recurso **systemd.automount** para que o compartilhamento seja montado **somente quando for acessado** — por exemplo, ao abrir `/media/pub`.  

Isso evita que o boot demore caso o servidor esteja desligado ou fora da rede.

Para configurar, edite novamente o `/etc/fstab` e adicione a opção `x-systemd.automount`, assim:
```
# Montagem sob demanda da pasta pub
//nas01/pub /media/pub cifs credentials=/etc/cifs-credentials.gsantana.localdomain.lan,users,rw,noauto,nosuid,nodev,file_mode=0777,dir_mode=0777,x-systemd.automount 0 0
```

Depois ative o comportamento:
```bash
sudo systemctl daemon-reexec
sudo systemctl restart remote-fs.target
```

A partir de agora, o diretório `/media/pub` será montado automaticamente **somente quando for acessado** — e desmontado após um tempo de inatividade. 
É uma ótima solução para compartilhamentos de **NAS**, **servidores de rede** ou **máquinas Windows** que nem sempre estão online.
Com isso, seu sistema acessará compartilhamentos de rede de forma **segura, automática e inteligente**, seja durante o boot, seja sob demanda, conforme sua necessidade.
O único problema dessa solução é que ambientes como KDE e GNOME fazem a leitura do /etc/fstab e testam a acessibilidade e então mostram na arvore do gestor de arquivos estas pastas como disponíveis e com isso, não-intencionalmente, fazem a montagem. Sob estas condições, é melhor "favoritar" endereços de rede no próprio gestor de arquivos.


### Explicando os parâmetros de montagem mais utilizados:  
|Parametro|Explicação|
|:--|:--|
|users|Permite que usuários normais montem e desmontem o compartilhamento, não apenas o superusuário (root).|
|rw|Especifica que o compartilhamento deve ser montado com permissões de leitura e escrita.|
|nosuid|Impede a execução de arquivos com permissões suid (Set User ID), o que pode ser um risco de segurança em compartilhamentos de rede.|
|nodev|Impede a criação de arquivos de dispositivo no compartilhamento montado.|
|file_mode=0777|Define as permissões para arquivos dentro do compartilhamento montado como 0777, o que concede permissões totais (read/write/execute) para todos os usuários.|
|dir_mode=0777|Define as permissões para diretórios dentro do compartilhamento montado como 0777, também concedendo permissões totais para todos os usuários.|
|auto|Faz a montagem diretamente no boot|
|noauto|Não faz a montagem automática durante o boot|
|x-systemd.automount|Faz a montagem sob demanda|
|zero e zero no final da linha|Desativa dump e fsck automático.|

