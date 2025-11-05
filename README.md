# PREPARAÇÃO DO AMBIENTE UBUNTU PARA DESENVOLVEDORES E PROFISSIONAIS DE TI

Este documento tem como objetivo preparar um ambiente Ubuntu sólido e funcional para desenvolvedores, administradores de sistemas e demais profissionais de TI.  
Não se trata apenas de uma sequência de comandos — aqui você também entenderá **como e por que** cada parte do sistema é configurada da maneira proposta.

A ideia é que, ao final, você não apenas tenha um ambiente pronto para trabalhar, mas também compreenda o que está por trás de cada ajuste:  
desde o gerenciamento de pacotes e permissões, até a montagem de partições, configuração de compartilhamentos de rede e automação de tarefas.  

Cada seção foi escrita para ser **autoexplicativa e prática**, apresentando o comando, o motivo de usá-lo e o que esperar de seu resultado.  
Assim, mesmo que você esteja acostumado a copiar instruções da internet, este guia vai além — ele busca **fortalecer seu entendimento** sobre o funcionamento do sistema Linux e seus principais utilitários.

As instruções foram testadas em versões recentes do Ubuntu (25.10 e derivadas), mas também servem como referência para outras distribuições baseadas em Debian.  
O foco é a eficiência, a estabilidade e o domínio do ambiente de desenvolvimento, sem depender de ferramentas gráficas para tarefas que podem (e devem) ser entendidas pelo terminal.

Prepare-se para não apenas configurar o seu sistema, mas **aprender o Linux de verdade**, enquanto monta um ambiente de trabalho robusto e otimizado para o dia a dia.  

---

## Sobre o particionamento (Btrfs vs ext4)
Se o seu foco for virtualização e você pretende usar snapshots (recurso em que o Btrfs brilha), o Btrfs pode ser excelente — mas há nuances para VMs (desempenho, CoW, layout de subvolumes) que exigem atenção.
Se você não precisa de snapshots ou prefere o caminho mais simples, ext4 é uma escolha direta e estável. No tópico específico de Btrfs explico quando e por que usá-lo (e como ajustar para uso de VMs).  
Se possivel, todas as partições que contêm dados importantes devem ter um *label* como #dados1, #dados2, #disco1, #home e assim por diante, sempre sendo fáceis de serem identificados quando executarmos o comando **lsblk -f**. Colocar labels em disco é vida!  

---

## Como usar este guia
A proposta deste guia não é **decorar comandos**, mas servir como um **repositório pessoal de referência**.  
Você lê este guia passo a passo e além de instalar, aprende para quê servem o que está instalando.  

Embora você possa ler este Guia passo a Passo diretamente do github, existe a opção de baixá-lo para lê-lo offline caso precise, eu recomendo fortemente que você **clone este projeto** localmente porque eu usei imagens nela e o github em planos gratuítos tem um limite mensal então se por acaso este limite for ultrapassado, não irá ver as imagens. Para clonar, execute:  
```bash
git clone https://github.com/gladiston/debianlinux.git
```
Para ler, execute:  
```bash
sudo apt install grip -y
cd debianlinux
grip
```
Você verá algo como:  
> * Serving Flask app 'grip.app'  
> * Debug mode: off  
> * Running on http://localhost:6419  

Então no seu navegador, aponte para o endereço acima e verá o conteúdo desse guia.  Pessoalmente eu gosto do jeito que o github renderiza porque eles colocam um botão de copiar que economiza tempo e a renderização local não faz isso. Em contrapartida, numa cópia local, você verá sempre as imagens sem depender dos limites impostos pelo github.  

A propósito, sinta-se à vontade para **adaptar os scripts ao seu cenário** pulando o que for indesejado.  

---

## Resultado esperado
Um sistema previsível e repetível, com configurações documentadas, pronto para trabalho diário, testes e virtualização.

---

## Os padrões usados neste HowTo
Para o correto entendimento deste HowTo, usarei alguns padrões:  
**Nome do host**: ti-01  
**Nome do usuário**: gsantana  
**Nome do dominio local**: localdomain.lan  
**Debian-Like**: É o termo que uso para distro Linux baseadas em Debian que pode se referir aos vários sabores do Ubuntu e também Linux Mint, Zorin OS,... quando algo for específico para o Debian, irei realçar.   
**Ubuntu-Like**: É o termo que uso para distro Linux baseadas em Ubuntu que pode se referir aos vários sabores do Ubuntu e também Linux Mint, Zorin OS,...   
**HowTo**: É o termo que designamos para guia passo-a-passo.    

As vezes, comandos que precisam ser executados no terminal são mesclados com o texto da saída do comando, quando isso acontecer, para que você diferencie, qual que é o comando e qual é a saída de texto dele, os comandos serão precedidos de "$", por exemplo:  
```
$ sudo apt update -y
Obter:1 https://dl.google.com/linux/chrome/deb stable InRelease [1.825 B]
Obter:2 https://dl.google.com/linux/chrome/deb stable/main amd64 Packages [1.210 B]                                           
Atingido:3 http://archive.ubuntu.com/ubuntu questing InRelease                                             
Atingido:4 http://security.ubuntu.com/ubuntu questing-security InRelease
Atingido:5 http://archive.ubuntu.com/ubuntu questing-updates InRelease
Atingido:6 http://archive.ubuntu.com/ubuntu questing-backports InRelease
Obter:7 http://archive.ubuntu.com/ubuntu questing/main Translation-pt_BR [349 kB]
Obter:8 http://archive.ubuntu.com/ubuntu questing/main Translation-pt [161 kB]
Obter:9 http://archive.ubuntu.com/ubuntu questing/universe Translation-pt [823 kB]
Obter:10 http://archive.ubuntu.com/ubuntu questing/universe Translation-pt_BR [1.668 kB]
Obter:11 http://archive.ubuntu.com/ubuntu questing/restricted Translation-pt_BR [584 B]
Obter:12 http://archive.ubuntu.com/ubuntu questing/restricted Translation-pt [588 B]
Obter:13 http://archive.ubuntu.com/ubuntu questing/multiverse Translation-pt [7.720 B]
Obter:14 http://archive.ubuntu.com/ubuntu questing/multiverse Translation-pt_BR [18,3 kB]
Obtidos 3.031 kB em 4s (848 kB/s)                                              
Todos os pacotes estão atualizados.         
Nota: Algumas fontes podem ser modernizadas. Execute 'apt modernize-sources' para fazer isso.
```

---

## NOTEBOOKs DA LINHA ACER NITRO
Se tiver um ACER NITRO ou outro computador similar com “Secure Boot”, siga essas instruções:  

[Se tiver um ACER NITRO, siga as instruções aqui](https://github.com/gladiston/fedorainstallonacernitro)

---

## UBUNTU vs DEBIAN: Qual Escolher para Sua Produtividade?

### 🎯 Sobre Distribuições Ubuntu

Quando menciono **Ubuntu**, refiro-me não apenas à distribuição oficial, mas também a suas derivadas baseadas em **Ubuntu LTS**, como Linux Mint e ZorinOS. Para ambientes de produção ou uso corporativo, **recomendo exclusivamente as edições LTS** (Long Term Support) — não as versões intermediárias de suporte curto.

### Por que apenas LTS?

As edições LTS recebem atualizações de segurança e correções por até 5 anos, enquanto as versões intermediárias expiram em 9 meses. Ao instalar programas sensíveis — como **VirtualBox**, **VMWare Workstation**, **drivers NVIDIA** ou software corporativo — atualizações frequentes de kernel podem quebrar compatibilidades e desperdiçar seu tempo com troubleshooting.

**Analogia Windows:**
- **Windows Server** ≈ Ubuntu/Debian LTS: Apenas correções críticas, máxima estabilidade
- **Windows 11** ≈ Ubuntu versão intermediária: Novos recursos a cada atualização, às vezes instáveis

O Windows Server 2025, por exemplo, está disponivel desde 2024, é extremamente estável justamente porque prioriza segurança sobre inovação — cujas abas no Explorer só apareceram na atualização de agosto/2025.

---

## 🔄 Debian 13 vs Ubuntu LTS: Qual a Diferença?

Costumo dizer que é como **matar a sede com água gelada (Ubuntu) ou com pedra de gelo (Debian)** — ambas resolvem o problema, mas de formas diferentes.

O Ubuntu é uma derivação do Debian otimizada para **desktops e usuários menos técnicos**, enquanto o Debian mantém sua filosofia minimalista e tradicional.

### Diferenças Práticas

| Aspecto | Debian 13 | Ubuntu LTS |
|---------|-----------|-----------|
| **Firewall padrão** | iptables (apenas CLI, muito técnico) | ufw (com GUI integrada no GNOME/KDE) |
| **Configuração de rede** | `/etc/network/interfaces` (tradicional) | netplan (moderno) |
| **Drivers proprietários** | Instalação manual | Pré-integrados (impressoras, Bluetooth, WiFi) |
| **Filosofia** | Minimalista, pouca mudança | Pronto para uso, focado em UX |

### Exemplo Prático: Impressora

Tenho uma **Epson L355**. No Ubuntu, é reconhecida automaticamente na rede sem qualquer configuração. No Debian, seria necessário instalar drivers manualmente, mas simples. No Windows, ainda é um processo sofrível.

---

## 💻 Minha Recomendação

| Cenário | Recomendação | Motivo |
|---------|--------------|--------|
| **Ambiente corporativo/servidor** | Debian 13 ou Ubuntu LTS (indiferente) | Ambas igualmente estáveis e confiáveis |
| **Desktop pessoal/casa** | **Ubuntu LTS** | Melhor reconhecimento de hardware, menos configuração manual |

**Conclusão:** Para trabalho, escolha qualquer uma — a diferença é mínima. Para uso doméstico, Ubuntu LTS - e seus derivados - oferecem melhor experiência de uso com menos ajustes técnicos necessários.


---

## GNOME ou KDE PLASMA: Qual Ambiente de Trabalho Escolher?
Em sistemas Linux, o **ambiente de trabalho** (ou **Desktop Environment - DE**) é a camada gráfica que interage diretamente com o usuário. Diferentemente do Windows ou macOS, que possuem uma interface fixa, **Linux oferece múltiplas opções de ambientes gráficos**, cada um com filosofia, funcionalidade e estética próprias. As mais populares no Linux são: KDE e GNOME, mas qual as diferenças?

Curioso para saber minha opinião? Clique  no link abaixo:  
[GNOME ou KDE PLASMA: Qual Ambiente de Trabalho Escolher?](docs/debian_kde_gnome.md)  


---

## INSTALAÇÃO
A instalação do Debian/Ubuntu não tem grandes mistérios — o ponto mais delicado é mesmo o **particionamento do disco**.  
Abaixo segue uma sugestão baseada na minha experiência pessoal:  

|sistema|Ponto de montagem|rótulo        |Tamanho       |   
|-------|-----------------|:------------:|:------------:|
|fat32  |/boot/efi        |Nenhum        |1 GB          |
|swap   |Nenhum           |Nenhum        |memória atual |
|ext4   |/boot            |#boot         |1 GB          |
|ext4   |/                |#disco1-root  |100 GB        |
|ext4   |/home            |#dados1       |máximo        |  

Se preferir usar o **Btrfs**, o particionamento muda um pouco, nesse caso, “/” e “/home” ficam na **mesma partição**, que ocupa todo o espaço restante do disco.  
O ideal seria ter subvolumes separados para “/”, "/var" e “/home”, mas o instalador padrão do Debian, Ubuntu e da maioria das distros ainda não permitem subvolumes. Dá para fazer manualmente, claro, mas exigiria etapas extras que deixariam este HowTo mais complicado — então vamos manter o foco no básico.

O Btrfs é um sistema de arquivos excelente para quem desenvolve software graças ao recurso de *snapshots*, onde é possível recuperar versões anteriores de arquivos apagados ou sobrescritos sem precisar recorrer a backups tradicionais.  
Além disso, a **compactação transparente** ajuda a economizar espaço sem perda de desempenho perceptível.

> **ALERTA:** Partições Btrfs não devem ultrapassar 80% de ocupação; acima disso, a performance cai bastante por causa do **Copy-on-Write (CoW)**.  

Caso prefira **ext4**, mantenha, se possível, “/” e “/home” em partições separadas.  
Especialistas em virtualização também recomendam ter **/var** em separado; contudo, neste HowTo, as VMs ficarão no `$HOME`.  

### SWAP
A memória **SWAP** é uma memória de fuga — é para onde os programas “correm” quando a RAM acaba.  
Se a memória se esgotar totalmente, o sistema pode travar.  
Sempre que programas usam a swap, espera-se que seja apenas um pico, e que logo deixem de precisar dela.  

Se a swap estiver **sempre em uso**, o ideal é **instalar mais memória RAM**, afinal viver uma “*vida loca*” é desperdício de tempo e compromete o desempenho.  

O tamanho mínimo de swap deve ser **igual à memória RAM** caso deseje usar **hibernação**, pois o sistema precisa salvar todo o conteúdo da RAM nela.  
Por exemplo:  
- Se seu computador tem **16 GB** de RAM e isso já basta, crie **16 GB** de swap.  
- Se gostaria de ter **32 GB de RAM** mas só possui 16 GB, então complemente com **16 GB de swap**, totalizando 32 GB (16GB de memoria real e 16GB de “memória de fuga”).  

Mas se seu computador **não hibernará**, essa regra deixa de valer, nesse caso, basta definir a swap com um tamanho que sirva como **plano de fuga razoável** para o uso pretendido.

## 'SUDO' - SOMENTE PARA O DEBIAN  
O sudoers e seu utilitário de linha de comando chamado `sudo` é o responsável por elevar as permissões do usuário para que ele consiga executar comandos que apenas o **root** teria acesso.  

Por padrão, o usuario comum não é do mesmo grupo do `sudo`, isso impossibilitará de usar o comando `sudo`, vamos resolver essa situação.  

Se você está usando o Debian, é imprescindivel que façamos isso, siga as orientações abaixo:  
[Fazendo a configuração do sudoers no Debian](docs/debian_sudoers_user.md).  

---  

## 'SUDO' - PERSONALIZANDO OPÇÕES
O sudoers e seu utilitário de linha de comando chamado `sudo` é o responsável por elevar as permissões do usuário para que ele consiga executar comandos que apenas o **root** teria acesso.  Se você é desenvolver ou administrador de sistemas, tem duas opções que flexibiizam o uso do comando `sudo`, e vou demonstrar cada uma delas.

Se você gostaria de personalizar as opções do sudoers, siga as orientações abaixo:  
[Fazendo a configuração do sudoers no Debian](docs/debian_sudoers_opt.md).  

---  

## PERMISSÃO AO JOURNAL 
O journal é o mecanismo de logs do systemd. Ele registra praticamente tudo o que ocorre no sistema — mensagens do kernel, inicialização de serviços, eventos de segurança, entre outros. Antigamente, esses registros eram armazenados em simples arquivos texto (como /var/log/syslog), acessíveis a qualquer usuário. Hoje, o journal é um serviço binário centralizado com restrições de acesso.
Essa restrição afeta alguns comandos como o **systemctrl status [serviço]**, veja este exemplo:  
```bash
systemctl status systemd-journald
```
Se você observar um aviso como este:
> Warning: some journal files were not opened due to insufficient permissions.

É porque o Debian explicita a mensagem, mas outras distros derivativas do Debian como os sabores do Ubuntu, LinuxMint, ZorinOS e assim por diante, podem tanto já vir com esse problema corrigido como também simplesmente não mostrar logs em comandos como o `systemctrl` e isso não é bom. Então o que iremos fazer é verificar se temos acesso ao **journal** e se não tivermos, iremos acrescentar.

[Concedendo acesso ao journal](docs/debian_journal.md).  

---  

## BACKUP/RESTORE DA CONFIGURAÇÃO ORIGINAL DE REDE
Vamos ser cautelosos e fazer um backup de nossa configuração de rede atual, assim se algo der errado, restauramos.

É imprescindivel que façamos isso, siga as orientações abaixo:  
[Fazendo o backup da configuração de rede](docs/debian_backup_restore_network.md).  

---  

## BLOQUEIO DE TELA AUTOMÁTICO
O sistema normalmente é ajustado automaticamente para bloquear após 5 minutos de atividade, mas ‘falta de atividade’ é um termo incorreto, o correto seria ‘tempo sem interatividade’, isto é, o tempo que você fica sem ter que interagir com o computador. Às vezes estamos processando algo demorado e temos de esperar ou acompanhar a movimentação de log de status e o computador durante este tempo estará tendo muito trabalho, porém com pouca interatividade, a tela será bloqueada. Então precisamos saber quanto tempo precisamos nas tarefas do dia a dia ou então desligá-la.  

Neste guia passo a passo, em algumas opotunidades ficará sem interatividade esperando downloads e aguardando procedimentos serem finalizados, então é uma boa ideia fazer esse ajuste agora.  Então, este é um bom momento para ajustá-la, siga as instruções:  

[Ajustando o bloqueio de tela automático](docs/debian_lock_screen.md).  

---  

## INSTALANDO O GOOGLE CHROME
Cada distro geralmente acompanha seu próprio navegador, mas geralmente é o Firefox. No entanto, o Google Chrome é muito popular e, de fato, alguns sites só funcionam bem com o motor dele, por essa razão recomendo sua instalação, siga as instruções no link abaixo:    
[Instalando o Google Chrome](docs/debian_google_chrome.md).  

---  

## INSTALANDO O MICROSOFT EDGE
Algumas pessoas são apaixonados pelo navegador da Microsoft, se este é o seu caso, o navegador Microsoft Edge também está disponível para Linux, os procedimentos de instalação são similares ao Google Chrome, o que muda é basicamente o link para download, então se for do seu interesse obter este navegador então siga as instruções no link abaixo:  

[Instalando o Microsoft EDGE](docs/debian_msedge.md).  

---  

## ADICIONANDO OS REPOSITORIOS 'CONTRIB' e 'NON-FREE' NO DEBIAN (SOMENTE PARA DEBIAN)

Primeiro, vamos fazer um backup do arquivo original sources.list:
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.ori
```
Depois edite o arquivo sources.list:
```
sudo editor /etc/apt/sources.list
```

A depender do repositório que escolheu durante a instalação, o sources.list estará parecido como este:  
>deb http://deb.debian.org/debian/ trixie main non-free-firmware  
>deb-src http://deb.debian.org/debian/ trixie main non-free-firmware  
>deb http://security.debian.org/debian-security trixie-security main non-free-firmware  
>deb-src http://security.debian.org/debian-security trixie-security main non-free-firmware  
>  
>\# trixie-updates, to get updates before a point release is made;  
>\# see https://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_updates_and_backports  
>deb http://deb.debian.org/debian/ trixie-updates main non-free-firmware  
>deb-src http://deb.debian.org/debian/ trixie-updates main non-free-firmware  

Como poderá notar, em cada linha cita **main** e  **non-free-firmware**, precisaremos acrescenter às essas linhas também os repositórios **contrib** e **non-free** na mesma linha, ficando assim:
```
deb http://deb.debian.org/debian/ trixie main non-free-firmware contrib non-free 
deb-src http://deb.debian.org/debian/ trixie main non-free-firmware contrib non-free 

deb http://security.debian.org/debian-security trixie-security main non-free-firmware contrib non-free 
deb-src http://security.debian.org/debian-security trixie-security main non-free-firmware contrib non-free 

# trixie-updates, to get updates before a point release is made;
# see https://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_updates_and_backports
deb http://deb.debian.org/debian/ trixie-updates main non-free-firmware contrib non-free 
deb-src http://deb.debian.org/debian/ trixie-updates main non-free-firmware contrib non-free 
```
Se não pretende fazer construção de pacotes (build) a partir dos fontes, poderá comentar a linhas que começam com 'deb-src', isso tornará os comandos de atualização mais rápidos. Enfim, agora precisaremos atualizar o banco de dados de pacotes, execute:  
```
sudo apt update -y
```
E todos os repositórios seráo verificados, inclusive os recém acrescentados:   

>Atingido:1 http://security.debian.org/debian-security trixie-security InRelease  
>Atingido:2 http://deb.debian.org/debian trixie InRelease      
>Atingido:3 http://deb.debian.org/debian trixie-updates InRelease              
>Obter:4 http://deb.debian.org/debian trixie/non-free Sources [75,8 kB]        
>Obter:5 http://deb.debian.org/debian trixie/contrib Sources [52,3 kB]          
>Obter:6 http://deb.debian.org/debian trixie/contrib amd64 Packages [53,8 kB]  
>Obter:7 http://deb.debian.org/debian trixie/contrib Translation-en [49,6 kB]                            
>Obter:8 http://deb.debian.org/debian trixie/contrib amd64 Components [41,5 kB]                                  
>Obter:9 http://deb.debian.org/debian trixie/contrib Icons (48x48) [60,1 kB]                                           
>Obter:10 http://deb.debian.org/debian trixie/contrib Icons (64x64) [132 kB]                      
>Obter:11 http://deb.debian.org/debian trixie/contrib Icons (128x128) [281 kB]               
>Obter:12 http://deb.debian.org/debian trixie/non-free amd64 Packages [100 kB]                  
>Obter:13 http://deb.debian.org/debian trixie/non-free Translation-en [67,1 kB]               
>Obter:14 http://deb.debian.org/debian trixie/non-free amd64 Components [3.784 B]              
>Obter:15 http://deb.debian.org/debian trixie/non-free Icons (48x48) [578 B]                       
>Obter:16 http://deb.debian.org/debian trixie/non-free Icons (64x64) [10,4 kB]            
>Obter:17 http://deb.debian.org/debian trixie/non-free Icons (128x128) [2.167 B]             
>Obter:18 https://dl.google.com/linux/chrome/deb stable InRelease [1.825 B]          
>Obter:19 https://dl.google.com/linux/chrome/deb stable/main amd64 Packages [1.206 B]
>Obtidos 933 kB em 1s (1.856 kB/s)      
>Todos os pacotes estão atualizados.  

---  

## APT
A partir do Debian 13 e do Ubuntu 25.04, or **APT** foi atualizado, e toda vez que você usar o comando `apt` poderá surgir uma nova mensagem ao final, veja este exemplo:

```bash
$ sudo apt update -y
Obter:1 https://dl.google.com/linux/chrome/deb stable InRelease [1.825 B]
Obter:2 https://dl.google.com/linux/chrome/deb stable/main amd64 Packages [1.210 B]                                           
Atingido:3 http://archive.ubuntu.com/ubuntu questing InRelease                                             
Atingido:4 http://security.ubuntu.com/ubuntu questing-security InRelease
Atingido:5 http://archive.ubuntu.com/ubuntu questing-updates InRelease
Atingido:6 http://archive.ubuntu.com/ubuntu questing-backports InRelease
Obter:7 http://archive.ubuntu.com/ubuntu questing/main Translation-pt_BR [349 kB]
Obter:8 http://archive.ubuntu.com/ubuntu questing/main Translation-pt [161 kB]
Obter:9 http://archive.ubuntu.com/ubuntu questing/universe Translation-pt [823 kB]
Obter:10 http://archive.ubuntu.com/ubuntu questing/universe Translation-pt_BR [1.668 kB]
Obter:11 http://archive.ubuntu.com/ubuntu questing/restricted Translation-pt_BR [584 B]
Obter:12 http://archive.ubuntu.com/ubuntu questing/restricted Translation-pt [588 B]
Obter:13 http://archive.ubuntu.com/ubuntu questing/multiverse Translation-pt [7.720 B]
Obter:14 http://archive.ubuntu.com/ubuntu questing/multiverse Translation-pt_BR [18,3 kB]
Obtidos 3.031 kB em 4s (848 kB/s)                                              
Todos os pacotes estão atualizados.         
Nota: Algumas fontes podem ser modernizadas. Execute 'apt modernize-sources' para fazer isso.
```

> **OBSERVE A NOTA:**  
> *Nota: Algumas fontes podem ser modernizadas. Execute 'apt modernize-sources' para fazer isso.*

### O que significa “modernizar fontes” (modernize sources)

O sistema provavelmente detectou em `/etc/apt/sources.list` ou algum arquivo dentro de `/etc/apt/sources.list.d/` usando o formato **antigo** do APT, em vez do novo formato **deb822**, que é mais estruturado — organizado em “blocos” com chaves como `Types:`, `URIs:`, etc.  
Esse novo formato torna os arquivos mais legíveis, seguros e flexíveis. O Debian embora esteja usando a nova versão APT, não sugere a migração do formato de arquivo. Caso não modernize, não há problemas, tanto o Debian como Ubuntu e derivados vão aceitar o "jeito" antigo.  

No meu caso, a mensagem apareceu porque a instalação do **Google Chrome** adicionou um repositório usando o formato antigo.  
Para resolver, basta executar:

```bash
sudo apt modernize-sources
```

Saída típica do comando:

```
The following files need modernizing:
  - /etc/apt/sources.list.d/google-chrome.list

Modernizing will replace .list files with the new .sources format,
add Signed-By values where they can be determined automatically,
and save the old files into .list.bak files.

This command supports the 'signed-by' and 'trusted' options. If you
have specified other options inside [] brackets, please transfer them
manually to the output files; see sources.list(5) for a mapping.

For a simulation, respond N in the following .
Reescrever 1 fontes? [S/n] s
Modernizing /etc/apt/sources.list.d/google-chrome.list...
- Writing /etc/apt/sources.list.d/google-chrome.sources
```

Daqui em diante, toda vez que você acrescentar um novo repositório ou editar algum arquivo em `/etc/apt/sources.list` ou `/etc/apt/sources.list.d`, se desejar, use o comando `sudo apt modernize-sources`, mas não irei mais comentar sobre ele no restante do tutorial para ele não ficar tão grande.  

---  

## INCLUINDO O REPOSITÓRIO DA MICROSOFT
Sim, a Microsoft tem um repositório para distribuições *Debian-like*, o que inclui também as outras derivações como **Ubuntu** e seus sabores, **Linux Mint**, **ZorinOS**, entre outras.  
Não vamos instalar nada de lá ainda; vamos apenas incluir o repositório. E, por mais paradoxal que seja, há um *download* e instalação necessários justamente para que possamos ter esse repositório.  

Descubra a versão do seu Ubuntu executando:
```bash
cat /etc/issue
```
Exemplo de saída:
```
Debian GNU/Linux 13 \n \l
```
No exemplo acima, a versão é o **Debian 13**.  
Agora visite a página:  
[https://packages.microsoft.com/config/](https://packages.microsoft.com/config/)  
Você verá algo assim:  
```
Index of config/
../
alma/                                                                                                 
amazonlinux/                                                                                          
centos/                                                                                               
debian/                                                                                               
fedora/                                                                                               
opensuse/                                                                                             
rhel/                                                                                                 
rocky/                                                                                                
sles/                                                                                                 
ubuntu/                                                                                               
```
Se for Ubuntu ou Debian, acesse a página correspondente a sua distribuição e uma vez que escolhê-la, aparecerá então pastas com as versões suportadas pela distro escolhida e então acesse a pasta correspondente à sua versão, e baixe o arquivo **packages-microsoft-prod.deb**.  
Depois, dê um duplo clique nele para instalá-lo.  

Atualize os repositórios:
```bash
sudo apt update -y
```

Isso criará o arquivo **/etc/apt/sources.list.d/microsoft-prod.list**, que aponta para o repositório oficial da Microsoft.  

Está curioso para saber o que a Microsoft está compartilhando?  
Então execute:
```bash
apt-cache policy | grep packages.microsoft.com
```
Mas antecipando, temos essencialmente as bibliotecas do `mono`.  

É curioso que a atualização do repositório da Microsoft seja mantida por um pacote que precisa ser instalado manualmente — e depois ele mesmo passa a ser atualizado pelo próprio repositório.  
Isso sim é uma implementação diferenciada.  
O time da Microsoft claramente não conhece a oração dos programadores em C/C++:  
> “Salve-nos da recursividade; main().”  - hahahhahahah.

---  

## REPOSITÓTIOS RESTRICTED E MULTIVERSE - APENAS PARA UBUNTU E DERIVADOS
Durante a instalação do Ubuntu, há uma opção para incluir os repositórios adicionais.  
Se você **não** habilitou essa opção, é recomendável fazê-lo agora.  

Abra o menu de configurações e procure por **“Repositórios”**.  
Provavelmente você encontrará o aplicativo **“Programas e Atualizações”** (no GNOME) ou o **Discover** (no KDE).  
Qualquer um desses programas possui uma seção chamada **“Configurações”** ou **“Repositórios”**, onde é possível habilitar os repositórios extras.  

No caso do Ubuntu e derivações, precisamos habilitar os repositórios **restricted** e **multiverse**.  
Esses repositórios **estendem o repertório de pacotes e programas** disponíveis no sistema, incluindo codecs, drivers e utilitários não livres.  

Com os repositórios adicionais ativados, podemos instalar alguns pacotes importantes que liberarão codecs e *players* de áudio e vídeo no sistema.

---  

## ATUALIZAÇÃO DO SISTEMA
Vamos atualizar o repositório de programas e atualizar o sistema:  
```  
sudo apt update -y
sudo apt upgrade -y 
```  
E então observe o resultado:  
>Resumo:                                     
>  Atualizando: 0, Instalando: 0, Removendo: 0, Não atualizando: 0

No meu exemplo, não há atualizações, mas talvez em seu computador seja direferente.  

---  

## EDITOR DE TEXTO VIM
O **Vim (Vi IMproved)** é um editor de texto poderoso e altamente configurável, baseado no clássico **Vi**, presente em praticamente todas as distribuições Unix e Linux.  
É amplamente utilizado por administradores de sistema e desenvolvedores por ser leve, rápido e disponível mesmo em ambientes sem interface gráfica.  

Para saber mais e como fazer, siga as instruções no link abaixo:  
[Instalando e usufruindo do editor vim](docs/debian_vim.md).  

---  

## EDITOR DE TEXTO PADRÃO PARA O TERMINAL
Por padrão, Debian e Ubuntu (e muitas distros derivadas) definem o **nano** como editor de texto padrão do terminal. Embora o **nano** seja simples e intuitivo, muitos administradores e desenvolvedores preferem editores mais avançados, como o **Vim**, **Neovim** ou **Micro**, que oferecem recursos adicionais — realce de sintaxe, atalhos poderosos e suporte a múltiplos modos de edição. Sempre que o sistema precisar abrir um editor — por exemplo, ao executar comandos como `crontab -e`, `visudo` ou `git commit` — ele usará o editor definido na variável de ambiente **EDITOR** (ou **VISUAL**).  

Para saber mais e como fazer, siga as instruções no link abaixo:  
[Selecionando o editor de texto padrão para o terminal](docs/debian_terminal_editor.md).  

---  

## INSTALANDO O GPARTED
O `gparted` é o programa mais eficiente para gerenciar discos permitindo criar, editar e excluir partições. Seria inapropriada não instalá-lo, execute:  

```bash
sudo apt install -y gparted
```

---  

## INSTALANDO CODECS E PLAYERS DE AUDIO/VIDEO
Com os repositórios adicionais ativados, podemos instalar alguns pacotes importantes que liberarão codecs e *players* de áudio e vídeo no sistema, ENTÃO EXECUTE:  

```bash
sudo apt install -y libavcodec-extra ffmpeg vlc
```

---  

## INSTALANDO O STRACE
O **strace** mostra as chamadas de sistema (útil para ver onde o erro ocorre) e também detectar recursos que estão sendo usados por outros programas.  
Para saber mais e como fazer, siga as instruções no link abaixo:  
[Instalando e usufruindo do strace](docs/debian_strace.md).  

---  

## MONITORANDO O SISTEMA COM O HTOP
O **htop** é um monitor interativo de processos para Linux, uma versão aprimorada e muito mais amigável do clássico comando `top`.  
Ele exibe em tempo real o uso da CPU, memória, swap, tarefas em execução, processos e suas prioridades, tudo com cores e atalhos intuitivos.  
Ao contrário do `top`, o **htop** permite **navegar com o teclado** pelas listas, **matar processos** com um clique ou tecla e **organizar colunas** conforme sua preferência.

Para saber mais e como fazer, siga as instruções no link abaixo:  
[Instalando e usufruindo do htop](docs/debian_htop.md).  

---  

## MONITORANDO TEMPERATURAS COM O LM-SENSORS
O **lm-sensors** mostra as temperaturas, tensões e rotações das ventoinhas da sua placa-mãe e processador.  
É leve, simples e ótimo para acompanhar o aquecimento do sistema.

Para saber mais e como fazer, siga as instruções no link abaixo:  
[Instalando e usufruindo do lmsensors](docs/debian_lmsensors.md).  

---  

## INSTALANDO O NOTIFY-SEND
O **notify-send** é um utilitário geralmente usado para enviar mensagens de um usuário para outro dentro do mesmo sistema.  
Ele é análogo ao comando `wall`, que exibe mensagens diretamente no terminal.  

Muitos scripts utilizam o **notify-send** para exibir notificações gráficas na área de trabalho — por exemplo, alertas de sistema ou lembretes automáticos.  
Vamos instalá-lo:

```bash
sudo apt install -y libnotify-bin
```

---  

## INSTALANDO O SILVERSEARCH-AG(ag)
O **Silversearcher-ag**, também conhecido simplesmente como **ag**, é uma ferramenta de busca extremamente rápida para código-fonte e arquivos de texto.  
Ele é similar ao comando `grep`, porém muito mais veloz e prático — ideal para desenvolvedores e administradores que precisam localizar trechos de texto em grandes projetos.

Instale-o com:
```bash
sudo apt install -y silversearcher-ag
```
O `silversearch`ou simplesmente `ag` é um comando relativamente novo, mas gradualmente, substitua o uso do `grep` por ele, e perceberá gradualmente as vantagens e especialmente velocidade quando se lida com arquivos grandes.  

---  

## INSTALANDO ADICIONAIS PARA O APT
O programa **apt** já está instalado por padrão, mas para algumas operações ele precisa de pacotes adicionais.  
Eles não são instalados automaticamente, mas — em minha opinião — são obrigatórios:

```bash
sudo apt install -y apt-transport-https gpg
```

---  

## INSTALAÇÃO DE FERRAMENTAS DE DOWNLOAD (WGET E CURL)
O comando abaixo instala duas ferramentas essenciais para realizar downloads e requisições web diretamente pelo terminal Linux:

```bash
sudo apt install -y wget curl
```

**Descrição dos componentes:**
- **WGET**: utilitário simples e confiável para baixar arquivos via HTTP, HTTPS e FTP.  
- **CURL**: ferramenta mais avançada para transferir dados ou interagir com APIs, compatível com diversos protocolos (HTTP, HTTPS, FTP, SCP, entre outros).  

Esses programas são amplamente usados em **scripts**, **automações** e **testes de conectividade**.

---  

## INSTALANDO COMPACTADORES/DESCOMPACTADORES DE ARQUIVOS
São instalados poucos formatos por padrão; portanto, é recomendável instalar os pacotes abaixo para garantir suporte aos formatos mais comuns e também a outros que, embora pouco usados por usuários comuns, são bastante utilizados por desenvolvedores — por exemplo, o formato **RAR**.

```bash
sudo apt install -y tar zip unzip p7zip-full p7zip-rar rar unrar lzip lzma xz-utils bzip2 gzip squashfs-tools cabextract
```

|Pacote|Função / Formato|
|:--|:--|
|tar|Criação e extração de arquivos `.tar` e `.tar.gz`|
|zip, unzip|Manipulação de arquivos `.zip`|
|p7zip-full|Suporte a arquivos `.7z` (formato 7-Zip)|
|p7zip-rar, rar, unrar|Suporte a arquivos `.rar`|
|lzip, lzma, xz-utils, bzip2, gzip|Compactações livres amplamente usadas em pacotes Linux|
|squashfs-tools|Criação e extração de arquivos `.squashfs`|

---  

## INSTALANDO PROGRAMAS BASICOS PARA COMPILAÇÃO DE FONTES
Os pacotes a seguir são essenciais para quem pretende compilar programas ou bibliotecas no ambiente Linux.  
Neste HowTo, vamos precisar deles, portanto execute:

```bash
sudo apt install -y build-essential
sudo apt install -y dh-make exuberant-ctags dpkg-dev debhelper fakeroot
sudo apt install -y exuberant-ctags module-assistant dkms patch libssl-dev
sudo apt install -y libncurses-dev ack fontconfig imagemagick git meson sassc tree
```

---  

## SUPORTE A NOVAS FONTES
A aparência das fontes influencia diretamente a legibilidade, o conforto visual e até mesmo a produtividade durante o uso do sistema. No Linux, especialmente em distribuições como Debian e Ubuntu, é possível personalizar facilmente o conjunto de fontes disponíveis — seja para o ambiente gráfico, terminais ou IDEs de programação.

Este tutorial mostra como instalar e gerenciar fontes no sistema, incluindo opções populares entre desenvolvedores e usuários avançados. Também aborda a instalação de fontes de terceiros (como as da Microsoft) e alternativas modernas e livres que oferecem excelente qualidade visual e compatibilidade.  

Para instalá-las, siga estas instruções:  
[Instalando suporte a novas fontes](docs/debian_new_fonts.md)  

---  

## ATIVE O SUPORTE A FLATPAK CENTRAL
Flatpak é um sistema de empacotamento e distribuição de aplicativos para Linux que permite instalar programas de forma isolada do restante do sistema, em um sandbox. Isso garante maior segurança e compatibilidade entre diferentes distribuições (como Debian, Fedora, Ubuntu, etc.), já que o aplicativo traz junto todas as suas dependências e Flathub é o repositório oficial e mais popular de aplicativos distribuídos via Flatpak.
Não são todas as distros que habilitam este repositório, especialmente o Ubuntu.   

Para habilitá-lo, siga estas instruções:  
[Instalando e habilitando o suporte a flatpak](docs/debian_flatpak.md)  

---  

## INSTALANDO O VSCODE
O **Visual Studio Code (VS Code)** é uma IDE leve, poderosa e multiplataforma desenvolvida pela Microsoft.  
Ele combina a simplicidade de um editor de texto com recursos avançados de programação, como **autocompletar inteligente (IntelliSense)**, **depuração integrada**, **controle de versão com Git** e uma ampla variedade de extensões para praticamente qualquer linguagem.  

Para instalar-lo, siga estas instruções:  
[Instalando o vscode](docs/debian_vscode.md)  

---  

## OBTENHA O KDE COMPLETO (OPCIONAL)  
A versão do **KDE** que acompanha o Debian, Ubuntu e outras distros derivadas costuma ser uma edição reduzida e personalizada pelos mantenedores da distribuição, contendo apenas os componentes essenciais e alguns ajustes visuais como papéis de parede, ícones e logotipos próprios que julgaram necessários. Por isso, muitos módulos e aplicativos originais do projeto KDE não vêm instalados por padrão. 

A versão completa inclui uma grande variedade de ferramentas — desde jogos simples (como o Paciência) até programas educativos e utilitários diversos. Embora boa parte deles seja dispensável para programadores e administradores de sistema, ela oferece uma experiência mais rica e próxima do que o time do KDE idealizou, lembrando em alguns aspectos o conforto visual do Windows.  

Minha recomendação como programador e administrador de sistemas, é que o `kde-full` é dispensável em ambientes corporativos, mas no seu computador de casa é bacana ver o que o time do KDE acha necessário.

Se você deseja instalar o ambiente KDE completo, execute:

```bash
sudo apt install -y kde-full
```

Depois disso, *recomendo que reinicie o computador*.

---  

## PRELOAD (OPCIONAL)
Se estiver usando **discos mecânicos**, provavelmente sente muita **latência** ao carregar certos programas.  
Numa situação assim, é bom instalar um serviço chamado **preload**: ele monitora os programas que você mais utiliza e, durante o boot, já os carrega antecipadamente.  

A vantagem é a **velocidade** para abrir esses programas pela primeira vez.  

Para saber mais e como fazer, siga as instruções no link abaixo:  
[Instalando e usufruindo do preload](docs/debian_preload.md).

---  


## INSTALANDO PERFIS DE PERFORMANCE (TUNED)
O **tuned** é um programa que permite trocar, em tempo real, o perfil de desempenho do computador por outro.  
Por exemplo, você pode usar o perfil **balanceado** enquanto navega na internet e, em seguida, alternar para o perfil **realtime** quando quiser maximizar a performance.  
Para instalar e entender melho, siga as orientações no link abaixo:  
[Instalando e usufruindo de perfis de performance](docs/debian_performance_tuned.md).

## COMPLETANDO O IDIOMA PORTUGUÊS
O idioma português-brasil não está completamente instalado, e nem o corretor ortográfico. Vamos corrigir isso, siga o link abaixo:  
[Completando o idioma português do Brasil](docs/debian_pt_br.md).  

---  

## GIT
O Git é um dos sistemas de controle de versão mais utilizados no mundo do desenvolvimento de software. Ele permite gerenciar projetos de forma colaborativa, acompanhar alterações no código e garantir segurança e rastreabilidade em cada modificação.

No Linux, especialmente nas distribuições Debian e Ubuntu, a instalação e configuração do Git são simples, mas recentes mudanças no GitHub exigem ajustes adicionais para autenticação segura. Este tutorial mostrará como preparar seu ambiente corretamente, compilar o suporte ao libsecret (necessário para armazenar credenciais com segurança) e configurar o Git para utilizar tokens de acesso pessoal, substituindo o antigo método de login por senha.

Vamos configurar e corrigir alguns problemas, siga o link abaixo:  
[Configurar e corrigir alguns problemas](docs/debian_git.md).  

---  

## MUDANDO O NOME DO HOST  
Durante a instalação, você provavelmente definiu um nome para o seu computador (**hostname**).  
Entretanto, caso queira modificá-lo depois, é possível fazer isso facilmente.  

Pelo ambiente gráfico (**KDE** ou **GNOME**), abra o aplicativo **Configurações do Sistema** e, na barra de pesquisa, digite “host”, “sistema” ou algo similar — essas informações serão exibidas e poderão ser modificadas.  

A cada nova versão do KDE e do GNOME, essas opções podem mudar de lugar ou ser traduzidas de forma diferente, o que impede a inclusão de uma captura de tela precisa.  

Pelo terminal, no entanto, o processo é mais direto e confiável. Basta executar:

```bash
sudo hostnamectl set-hostname novo-nome
```

---  

## COMPARTILHAMENTO DE ARQUIVOS
O Samba é o componente responsável por permitir que sistemas Linux e Windows troquem arquivos e pastas através da rede local, utilizando o protocolo SMB/CIFS. Ele possibilita tanto o acesso a compartilhamentos de outros computadores quanto a criação de compartilhamentos próprios, tornando a integração entre diferentes sistemas fluida e transparente.

Nas distribuições Debian e Ubuntu, o Samba costuma vir parcialmente instalado, mas requer alguns ajustes para funcionar de forma completa e adequada em ambientes domésticos ou corporativos.
Este tutorial mostrará como instalar os pacotes necessários, ajustar o workgroup ou domínio da rede, evitar que o serviço atue como controlador de domínio e configurar corretamente o compartilhamento de arquivos, garantindo compatibilidade com máquinas Windows e o melhor desempenho possível no seu sistema Linux:

[Compartilhamento de arquivos usando samba](docs/debian_samba.md)  

---  

## CRONTAB
No Linux, o crontab é um dos recursos mais práticos e poderosos para automatizar tarefas. Ele permite que comandos e scripts sejam executados automaticamente em horários pré-definidos, sem necessidade de intervenção do usuário.  
Com ele, é possível agendar desde tarefas simples — como enviar lembretes — até rotinas administrativas complexas, como backups, sincronizações, atualizações de sistema ou limpeza de arquivos temporários.  

O serviço responsável por isso é o cron daemon (crond), que fica em execução contínua no sistema, verificando minuto a minuto se há algo a ser executado.
Cada usuário possui seu próprio agendador pessoal, enquanto o sistema mantém um crontab global, permitindo tanto automações individuais quanto tarefas administrativas que beneficiam todos os usuários.  

Neste tutorial, você aprenderá a editar, configurar e testar seus agendamentos no formato correto, além de compreender as diferenças entre o crontab de usuário e o crontab do sistema:  
[Agendador de tarefas no Linux](docs/debian_crontab.md)  

---  

## FIREWALL 
O firewall é uma camada essencial de segurança responsável por controlar o tráfego de rede, permitindo ou bloqueando conexões conforme regras definidas. Embora o Linux já inclua o poderoso iptables, ele não vem ativado por padrão e geralmente não possui interface gráfica, o que evita problemas de conectividade para usuários iniciantes.

Neste tutorial, optamos pelo firewalld, uma solução moderna, compatível com Debian, Ubuntu e diversas outras distribuições. Ele oferece comandos padronizados, perfis de rede (zones) e integração com KDE e GNOME, tornando o gerenciamento de portas e serviços mais simples, seguro e alinhado com ambientes profissionais.

[Instalando e configurando o firewall no Linux](docs/debian_firewall.md)  

---  

## AJUSTANDO ALIASES PARA COMANDOS REPETITIVOS
Os aliases são atalhos criados para simplificar comandos repetitivos no terminal. Em vez de digitar longas instruções toda vez, você pode definir abreviações personalizadas — tornando o uso do sistema mais rápido e produtivo.  
Este recurso existe desde os primeiros sistemas Unix e continua sendo amplamente usado no Linux moderno, permitindo ajustar o ambiente de terminal ao seu estilo de trabalho. Neste tutorial, você aprenderá a criar, editar e ativar seus próprios aliases no arquivo ~/.bashrc.  

[Ajustando aliases para comandos repetitivos](docs/debian_cmd_aliases.md)  

---  

## AJUSTANDO O PROMPT DO TERMINAL
Às vezes o prompt do terminal pode incomodar alguns usuários.  
Por exemplo, é justo que ao logarmos em servidores o terminal revele no prompt o **usuário** e o **nome do computador**:  
![Prompt normal](img/mudando_prompt01.png)  
que tal deixá-lo assim:  
![Novo prompt](img/mudando_prompt06.png)  

Então, vamos ajustar o terminal para que o prompt possa ser personalizado conforme nossas necessidades, siga as instruções no link abaixo:    
[Ajustando o prompt do terminal](docs/debian_set_prompt.md).

---  

## ACESSAR PARTIÇÕES LINUX NO SISTEMA
Se você utiliza uma ou mais partições Linux que não são automaticamente montadas, pode usar o gerenciador de arquivos do KDE ou GNOME para acessá-las.  
No entanto, toda vez que fizer isso provavelmente será solicitada uma senha — e isso cansa até o desenvolvedor mais paciente.  

Para entender melhor e ajustar seu sistema de acordo, siga as orientações no link abaixo:  
[Acessando partições linux](docs/debian_fstab_linux.md).


## ACESSAR PARTIÇÕES NTFS NO SISTEMA
Se você utiliza uma partição Windows (NTFS) para gravar seus arquivos e dados a partir do Linux, pode simplesmente não fazer nada e usar o gerenciador de arquivos do GNOME, KDE e afins para entrar e sair do disco NTFS quando quiser.  

---  

Para entender melhor e ajustar seu sistema de acordo, siga as orientações no link abaixo:  
[Acessando partições Windows/NTFS](docs/debian_fstab_ntfs.md).

---  

## ACESSANDO ARQUIVOS NA REDE
O Linux é muito versátil ao acessar arquivos pela rede. Diferente do Windows onde o compartilhamento de arquivos se dá apenas pelo protocolo smb/cifs do próprio Windows, no Linux, qualquer tipo de compartilhamento que tenha um protocolo de comunicação aberto pode ser montado em forma de pasta em seu sistema.  
Vamos considerar agora alguns tipos de compartilhamentos no link abaixo:  
[Acessando compartilhamentos na rede Windows/outros](docs/debian_fstab_network.md).

---  

## INSTALANDO O UTILITÁRIO DE EMAIL Thunderbird
O Thunderbird é um cliente de email, calendário e gerenciador de contatos multiplataforma, desenvolvido pela Mozilla. Para administradores de sistemas, desenvolvedores e equipes de TI, o Thunderbird oferece suporte a múltiplos protocolos de email (IMAP, POP3, SMTP), integração com calendários (CalDAV), gerenciamento avançado de identidades e filtros automáticos. Sua arquitetura modular permite extensões que facilitam a organização corporativa, criptografia de mensagens (OpenPGP) e sincronização de dados entre múltiplos dispositivos.  

Para instalar siga as instruções no link abaixo:  
[Instalando o utiitário de e-mail Thunderbird'](docs/debian_thunderbird.md).


## INSTALANDO O UTILITÁRIO DE BACKUP E RESTAURAÇÃO TimeShift
O Timeshift é um utilitário de backup e restauração de sistemas de arquivos baseado em snapshots incrementais, desenvolvido especificamente para distribuições Linux. Para administradores de sistemas e desenvolvedores, o Timeshift oferece a capacidade de criar pontos de restauração automáticos ou manuais, permitindo reverter o sistema para um estado anterior em caso de falhas críticas, atualizações problemáticas ou corrupção de arquivos. Diferentemente de ferramentas de backup tradicionais, o Timeshift trabalha diretamente com o sistema de arquivos, proporcionando rapidez e eficiência operacional.  

Para instalar siga as instruções no link abaixo:  
[Instalando o utilitário de backup TimeShift](docs/debian_timeshift.md).

---  

## BANCO DE DADOS FIREBIRD
O FirebirdSQL é um banco de dados relacional open source, leve e poderoso, derivado do InterBase da Borland. Ele roda em Windows, Linux, macOS e ARM, e é amplamente usado em sistemas comerciais, ERP e aplicações embarcadas. 

Pontos positivos do FirebirdSQL:  
* Desempenho elevado — Trabalha bem com bancos grandes e muitos usuários simultâneos, mesmo em hardware modesto.
* Zero administração — Dispensa serviços complexos; pode funcionar como embedded, sem instalação de servidor.
* Segurança robusta — Criptografia nativa de dados e autenticação integrada.
* Transações completas — Suporte total a ACID, com isolamento configurável (Read Committed, Snapshot, Serializable).
* Portabilidade — Um único arquivo .fdb contém todo o banco, facilitando backup e migração.
* Ferramentas familiares — Linguagem SQL padrão, PSQL para stored procedures e triggers.
* Configuração simples — Fácil de instalar e manter; ideal para sistemas que exigem confiabilidade sem administração constante.

Para instalá-lo, siga as orientações no link abaixo:  
[Instalação do Firebird](docs/debian_firebird.md).

---  

## HABILITANDO AREA DE TRABALHO REMOTA
Vez ou outra precisaremos acessar nossa area de trabalho, as mais experientes recomendarão usar o 'ssh -x' ou usar 'xserver' e logar-se no ip de nosso desktop, no entanto, isso não é tão simples para novos usuários do linux e também não permite o acesso onde a origem é um desktop Windows. Portanto, minha recomendação é instalar o xrdp, um protocolo de compartilhamento de sessões compativel com o 'rdp' da Microsoft e assim poderemos acessar nosso terminal Linux até mesmo de um Windows através do programa 'Remote Deskop'. 

Para instalá-lo, siga as orientações no link abaixo:  
[Habilitando a Area de Trabalho Remota no Debian(rdp)](docs/debian_rdp.md).

---  

## INSTALANDO O CLIENTE DE ACESSO REMOTO 'REMMINA'
O Remmina é um cliente de acesso remoto versátil e leve, desenvolvido em GTK+, que suporta múltiplos protocolos de conexão remota (RDP, SSH, VNC, SPICE, X2Go, entre outros). Para ambientes corporativos que necessitam gerenciar múltiplas sessões remotas a partir de uma única aplicação, o Remmina oferece uma solução integrada e de fácil configuração. Se você é um administrador de sistemas ou desenvolvedor que precisa acessar outras máquinas seja Windows ou Linux, o Remmina é indispensável.  

Neste guia, utilizaremos a distribuição Flathub para a instalação, garantindo uma versão atualizada e isolada em containerização, evitando conflitos com dependências do sistema base do Debian. Além disso, abordaremos a migração de configurações de instalações anteriores.  

Para instalá-lo, siga as orientações no link abaixo:  
[Instalando o cliente de acesso remoto Remmina](docs/debian_remmina.md).

---  

## INSTALANDO O CLIENTE DE MENSAGERIA 'TELEGRAM'
O Telegram é um aplicativo de mensageria instantânea baseado em nuvem, reconhecido por sua segurança, velocidade e recursos avançados de comunicação. Para administradores de sistemas, desenvolvedores e equipes de TI, o Telegram oferece canais, grupos privados e bots automatizados que facilitam a colaboração, notificações de sistemas e automação de processos operacionais. A compatibilidade multiplataforma (Windows, macOS, Linux, iOS e Android) o torna uma solução ideal para comunicação corporativa distribuída.

Neste guia, utilizaremos a distribuição Flathub para a instalação, garantindo uma versão atualizada e isolada em containerização, evitando conflitos com dependências do sistema base do Debian. Além disso, abordaremos a migração de configurações de instalações anteriores.  

Para instalá-lo, siga as orientações no link abaixo:  
[Instalando o Telegram](docs/debian_telegram.md).

---  

## VIRTUALIZAÇÃO NATIVA QEMU+KVM
O Linux é capaz de criar máquinas virtuais e ele mesmo ser o hypervisor. Será um servidor de virtualização nivel 1, o mais rápido possivel, no entanto com algumas ausencia de recursos que facilitam a configuração que existem no VirtualBox e VMWare, por exemplo, criar redes virtuais com vários tipos de topologias, clipboard e transferencia de arquivos entre host e anfitrião e outras coisas.  

Para instalá-lo, siga as orientações no link abaixo:  
[Guia de instalação do QEMU/KVM](docs/debian_qemu_kvm.md)

---  

## VIRTUALBOX
O VirtualBox é outro virtualizador, ele é do tipo "2" e isto significa que é um pouco inferior em performance ao qemu+kvm, no entanto, ele tem muito mais recursos para desktop do que o QEMU+KVM, por exemplo, o SEAMLESS que permite puxar um aplicativo Windows dentro da VM para fora, isto é, para o sistema hospedeiro, causando a impressão que estamos rodando uma aplicação Linux nativa.
No entanto, ele enfrenta alguns bugs chatos desde que os ambientes Linux estão migrando do Xorg para o Wayland. Alguns são problemas grandes, o SEAMLESS não funciona mais, e outros são problemas aleatórios e irritantes como o conteúdo da área de clipboard entre anfitrião e convidado deixar de funcionar, cursor do mouse que deixa de funcionar e coisas assim. Espero que as próximas versões corrijam isso, afinal o VirtualBox é um bom virtualizador e tem uma opção que qemu+kvm não tem: transportar a VM para outros sistemas operacionais, isto é, você pode copiar a VM criado no Linux para rodar num hospedeiro Windows ou Mac OS.  

Para instalá-lo, siga as orientações no link abaixo:  
[Guia de instalação do VirtualBox no Debian](docs/debian_virtualbox.md)

---  


\#  
\#  
## DAQUI EM DIANTE SÃO PROGRAMAS RECOMENDADOS PARA USO PESSOAL QUE PODEM SER IGNORADOS
\#  
\#  

---  

## INSTALANDO IDE DE PROGRAMAÇÃO PASCAL COM A IDE LAZARUS
O Lazarus é uma IDE completa para desenvolvimento em Object Pascal, baseada no compilador Free Pascal (FPC).
Ela é multiplataforma (Linux, Windows e macOS) e oferece uma experiência muito próxima do antigo Delphi, com formulários visuais, depurador integrado e suporte a componentes visuais compatíveis com diversos sistemas gráficos (GTK, Qt, Win32, Cocoa etc.).   

Este tutorial mostra como instalar o Lazarus de forma limpa e moderna usando o fpcupdeluxe, uma ferramenta que automatiza a obtenção do código-fonte mais recente diretamente do repositório oficial e compila tudo localmente, sem interferir no sistema. O método é especialmente útil para quem deseja ter múltiplas versões do Lazarus, fazer cross-compile ou manter um ambiente isolado por usuário.   

Antes de começar, é importante garantir que o sistema esteja atualizado e que todas as dependências necessárias para o ambiente gráfico (Qt ou GTK) estejam instaladas. Nos passos a seguir, você verá como preparar o ambiente e instalar o Lazarus sem precisar de permissões administrativas (homeuser install), ideal para desenvolvedores que preferem manter o sistema limpo e facilmente reversível.   

Para instalá-lo, siga as orientações no link abaixo:   
[Guia de instalação do Lazarus](docs/debian_lazarus.md)

---  

## SOFTWARE PARA TREINAMENTO
Para criar material de treinamento que incluirá vídeo é sugerível instalar a seguinte extensão Draw On Your Screen cuja instrução para instalação se encontra em:
https://codeberg.org/som/DrawOnYourScreen

```
git clone https://codeberg.org/som/DrawOnYourScreen --depth=1 --single-branch --branch face ~/.local/share/gnome-shell/extensions/draw-on-your-screen@som.codeberg.org
```
Depois vá até .local/share/gnome-shell/extensions e abra o arquivo metadata.json e adicione "41" e então reinicie o gnome-shell.

---  

## ZOOM CLOUD MEETINGS
Para baixá-lo use a loja de software (Programas) e procure por “Zoom” e instale-o.

---  

## IMPRESSORA PDF
É muito útil instalar uma impressora PDF no sistema, pode ser usado para economizar impressões e arquivá-las ao escanear um documento e depois armazená-lo como PDF. 
Na realidade isso já vem instalado no sistema, apenas muda o método caso esteja usando KDE ou GNOME.

---  

## INSTALANDO A IMPRESSORA EPSON L355 LOCALIZADA NA REDE
Da ultima vez que verifiquei, essa impressora é reconhecida automaticamente.

---  

## INSTALANDO O SCANNER EPSON L355
Da ultima vez que verifiquei, o scanner integrado é reconhecida automaticamente.

---  

## INSTALANDO O LEITOR OCR
(todo)

---  

## LEITOR DE CERTIFICADO DIGITAL
Cada leitor e modelo pode ter instruções diferentes, é melhor procurar um howto na internet apropriado.

---  

## MICROSOFT OFFICE (web apps)
Visite a página a seguir usando um navegador Google Chrome(ou compatível com webapps):
[Site do office.com](office.com)   
E autentique-se com uma conta live da Microsoft.
Estando na home da página, precisará de usar o navegador para transformar a página em um aplicativo WEB. Usando o Google Chrome você iria nos 3 pontinhos no canto superior direito>Transmitir, salvar e compartilhar>Instalar página como app...   

![Instalação do app MSOFFICE](img/app-msoffice-web.png)  

Na realidade, para qualquer aplicativo WEB, você seguiria estas mesmas instruções.  

---  

## INSTALANDO O GIMP
Vá no app  Software e procure por GIMP no repositório do ‘Flathub’:

Clique nas propriedades dele e encontrará alguns plugins(complementos) que também poderá instalar, são eles:

### BIMP - Realizar operações em batch com vários arquivos
https://www.youtube.com/watch?v=CaeTkgPNkkg

### FocusBlur - Capacidade de efeito de profundidade
https://www.youtube.com/watch?v=u-YB-KipWzk

### Gimp transformação Fourier
Técnica para remover ou manipular padrões em fotos, geralmente as antigas
https://www.youtube.com/watch?v=se9I3uGITR0

### Gimp Lens Fans
Aplicar e corrigir efeitos por lentes
https://www.youtube.com/watch?v=FQGDgBT1tWk

### G’MIC
GRAYC’s Efeitos
https://www.youtube.com/watch?v=kZnEpkNsDK0
https://www.youtube.com/watch?v=VOPHbSgJUSI

### LiquidRescale
Permite remover um elemento e redimensionar uma imagem como se o elemento nunca estivesse existido.
https://www.youtube.com/watch?v=hhFVWKJA76U

### Resynthesizer
Retire manchas ou outros defeitos de imagens
https://www.youtube.com/watch?v=n76owcpShqw

## ESPELHAMENTO DE CELULAR  
Usaremos um programa chama scrcpy, para instalar executamos:  

```
sudo dnf install -y adb android-tools 
sudo dnf install -y scrcpy scrcpy-server
```
Para usá-lo terá de ativar o modo de depuração de seu celular ou tablet:  
(https://developer.android.com/studio/command-line/adb#Enabling)

E em alguns aparelhos também:  
(https://github.com/Genymobile/scrcpy/issues/70#issuecomment-373286323)

Existe também uma GUI que para alguns simplifica algumas operações, faça a instalação a partir da loja de aplicativos:
```
guiscrcpy
```

---  

## USANDO O CELULAR COMO WEBCAM
Instruções: https://www.dev47apps.com/droidcam/linux/

No celular Android instale o app DroidCam.
No terminal, execute:
```
sudo apt install -y android-tools-adb
cd /tmp
wget https://files.dev47apps.net/linux/droidcam_latest.zip
unzip droidcam_latest.zip -d droidcam
cd droidcam
sudo ./install-client
```
E prossiga com a instalação, para acionar a parte de vídeo:
```
sudo ./install-video
```
E prossiga com a instalação, para acionar a parte de áudio:
```
sudo ./install-sound
```
E prossiga com a instalação.
Para ter acesso a webcam, execute no terminal droidcam ou droidcam-cli, haverá as opções de usar o celular como webcam plugado na USB ou via Wifi. Para criar um atalho no ambiente GNOME:
```
editor ~/.local/share/applications/droidcam.desktop
```
e então colar:
```
[Desktop Entry]
Version=1.0
Type=Application
Terminal=false
Name=DroidCam
Exec=droidcam
Comment=Use seu celular Android como uma Webcam wireless/USB ou como IP Cam!
Icon=droidcam
Categories=GNOME;GTK;Video;
Name[it]=droidcam
```
Salve e feche o arquivo e a partir de agora encontrará o ícone do DroidCam no seu sistema.

---  

## OBS STUDIO
Este é o melhor programa de studio e studio de streaming. É incrível acreditar que é opensource e extremamente profissional. Para instalar basta ir na loja e escolher OBS Studio.  

---  

## MINDER
Este é o melhor programa para mapas mentais no formato que roda no modo desktop. Ele é muito similar ao femi. Para instalar:  
```
flatpak install --user https://flathub.org/repo/appstream/com.github.phase1geo.minder.flatpakref
flatpak --user update com.github.phase1geo.minder
```
ou se preferir pelo repositório do Ubuntu:
```
sudo apt install minder
```

---  

## HYPNOTIX
Este é o melhor programa de iptv. Ótimo para assistir TVs que são transmitidas via tv. Para instalar:
```
wget https://github.com/linuxmint/hypnotix/releases/download/1.1/hypnotix_1.1_all.deb
sudo dpkg -i hypnotix_1.1_all.deb
sudo apt -f install
```
Algumas fontes de TVs podem estar irregulares, então acaso não queira assistir TVs então poderá desinstalar o programa com o comando:
```
sudo apt remove hypnotix*
```

---  

## INSYNC
Este é o melhor programa cliente de Google Drive, ele simula uma unidade de drive local e comumente é usado para colocar backups no Google Drive sem a necessidade do browser. Para instalar é simples e complicado, simples porque você só tem que dar dois cliques no arquivo e complicado porque se trata dum software proprietário que por não poder ser auditado você terá de confiar no fornecedor. Se você deseja prosseguir assim mesmo então visite a página:  
(https://www.insynchq.com/downloads)

E baixe o instalador e depois dê dois cliques sobre o arquivo baixado e o processo de instalação se iniciará. Se você deseja instalar a partir do repositório, no link acima eles fornecem instruções para serem executados no terminal e você terá atualizações do insink como qualquer outro programa advindo dos repositórios oficiais, neste caso, oficiais do publicador do insink.  

---  

## BLANKET
Programa para exibir sons no ambiente de fundo, geralmente usado para focar no trabalho, com sons ambiente da natureza ou urbanos como de uma cafeteria. Para instalar basta ir na loja e escolhê-lo pelo nome Blanket.  

---  

## HOMESERVER
Este programa serve para compartilhar uma ou varias pastas de uma forma simples, voce inicia o programa, indica as pastas a serem compartilhadas e momentaneamente eles estarão disponíveis para os computadores na mesma rede local através do navegador. Para instalar basta ir na loja e escolhê-lo pelo nome homeserver.  

Quando os clientes copiarem o que queriam, você fecha o aplicativo e o compartilhamento estará encerrado ou pode encerrar pasta a pasta.
Observação: Geralmente se você habilitou o compartilhamento de arquivos, talvez não precise do HOMESERVER.  

---  

## HANDBRAKE
HandBrake é um dos mais poderosos conversores de vídeo. Para instalar basta ir na loja e escolhê-lo pelo nome handbrake.  

---  

## FORMATLAB
FormatLab é um dos mais promissores conversores de vídeo após o HandBrake. Ele faz as mesmas atividades do handbrake, porém é mais simples de operar. Para instalar basta ir na loja e escolhê-lo pelo nome FormatLab.  


---  

## BLENDER
Blender, também conhecido como blender3d, é um programa de código aberto, desenvolvido pela Blender Foundation, para modelagem, animação, texturização, composição, renderização, e edição de vídeo.  Para instalar basta ir na loja e escolhê-lo pelo nome blender. Se você não cria animações, ignore a instalação desse programa.  

---  

## VIDCUTTER
(analogo ao vidcoder para windows)

(http://bluegriffon.org)


---  

## INKSCAPE
Para instalar basta ir na loja e escolhê-lo pelo nome INKSCAPE.  

---  

## OUTROS TÓPICOS INTERESSANTES
* Ambiente de programação JAVA
* Ambiente de programação Python
* Versionamento com o asdf


