# INSTALAÇÃO DO DEBIAN E PREPARAÇÃO DO AMBIENTE
Instalação do Debian e preparação do ambiente de desenvolvimento.  

Serão muitas instruções a seguir, mas vale a pena ir até o final, mas fique liguado para pular as opções que podem não ser necessárias para você. E quando finalmente chegar ao final você será recompensado por não precisar ficar reinstalado o sistema todas as vezes como no Windows e tudo sempre funcionará exatamente como planejado.

Para o correto entendimento, usarei alguns padrões:  
Nome do host: ti-01  
Nome do usuário: gsantana  
IP: 192.168.1.5  
Nome do dominio local: local.lan  


## INSTALAÇÃO
Nada de especial aqui, mas recomendo que tenha a partição / (root) em BTRFS. Este sistema de arquivos é uma mão na roda para programadores porque poderá ser habilitado snaphots no sistema de boot e arquivos importantes que você crie.


## Notebooks da série ACER NITRO
Se tiver um ACER NITRO ou outro computador similar com “Secure Boot”, siga essas instruções:  

[Se tiver um ACER NITRO, siga as instruções em:  ](https://github.com/gladiston/fedorainstallonacernitro)


## SUDO
Por padrão, o usuario comum não é do mesmo grupo "sudo", isso impossibilitará usar o comando "sudo", vamos resolver essa situação, execute:

```
su root  
(precisará digitar a senha)  
sudo usermod -aG sudo gsantana  
exit  
```  
Se estiver usando uma maquina de desenvolvimento e deseja relaxar um pouco o 'sudo' então poderá inibir ele de ficar perguntando a senha a todo instante, vá no terminal e execute:  
```  
su root  
(precisará digitar a senha)  
sudo visudo    
```
e então procure por:    
```
root    ALL=(ALL:ALL) ALL  
```  
E acrescente a seguinte linha abaixo:  
```
gsantana    ALL=(ALL) NOPASSWD: /bin/mount, /bin/umount, /bin/mkdir, /bin/rm, /bin/cp, /bin/chmod, /bin/chown, /bin/touch  
gsantana    ALL=(ALL) NOPASSWD: ALL # precisa de senha para todos, exceto a lista acima  
#gsantana   ALL=(ALL:ALL) NOPASSWD: ALL # libera qualquer comando sem usar senha  
(Salve o arquivo e saia do editor)  
exit  
```
Para testar, tente:  
```
sudo touch /tmp/teste.txt    
```
O comando acima só vai criar um arquivo vazio como /tmp/teste.txt, mas note que ele não perguntou a senha porque este comando estava na relação dos comandos permitidos sem uso de senha, isto é: /bin/mount, /bin/umount, /bin/mkdir, /bin/rm, /bin/cp, /bin/chmod, /bin/chown, /bin/touch...porém qualquer outro comando, ele perguntará a senha, veja:  
```
sudo apt update    
```
Não faça isso em computadores de produção, mesmo em desenvolvimento dependendo do comando que foi liberado, pode ser perigoso. Se achar que po risco não vale a pena, simplesmente remova a linha:  
```  
gsantana    ALL=(ALL) NOPASSWD: /bin/mount, /bin/umount, /bin/mkdir, /bin/rm, /bin/cp, /bin/chmod, /bin/chown, /bin/touch
```
E então qualquer comando precisará de senha quando o 'sudo' estiver envolvido.  
A linha abaixo esta comentada porque se for liberada, nenhum comando 'sudo' precisará mais de senha:  
```  
#gsantana   ALL=(ALL:ALL) NOPASSWD: ALL # libera qualquer comando sem usar senha  
```
Então isso pode ser um risco ou não a depender do contexto em que você estiver inserido, se achar apropriado fazer isso na sua estação de trabalho, poderá fazê-lo, mas tenha certeza de que seu   computador é um computador zé-roela que não oferece nenhum risco.  

## ADICIONANDO OS REPOSITORIOS 'CONTRIB' e 'NON-FREE'
Execute:
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.ori
nano /etc/apt/sources.list
```
A depender do repositório que escolheu durante a instalação, o sources.list estará parecido como este:
```
deb http://mirrors.ic.unicamp.br/debian/ trixie main non-free-firmware
deb-src http://mirrors.ic.unicamp.br/debian/ trixie main non-free-firmware

deb http://security.debian.org/debian-security trixie-security main non-free-firmware
deb-src http://security.debian.org/debian-security trixie-security main non-free-firmware

# trixie-updates, to get updates before a point release is made;
# see https://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_updates_and_backports
deb http://mirrors.ic.unicamp.br/debian/ trixie-updates main non-free-firmware
deb-src http://mirrors.ic.unicamp.br/debian/ trixie-updates main non-free-firmware
```

Como poderá notar, cada linha cita 'main non-free-firmware', precisamos acrescenter 'contrib' e 'non-free' à essas linhas ficando assim:
```
deb http://mirrors.ic.unicamp.br/debian/ trixie main non-free-firmware contrib non-free  
deb-src http://mirrors.ic.unicamp.br/debian/ trixie main non-free-firmware contrib non-free   

deb http://security.debian.org/debian-security trixie-security main non-free-firmware contrib non-free  
deb-src http://security.debian.org/debian-security trixie-security main non-free-firmware contrib non-free  

# trixie-updates, to get updates before a point release is made;
# see https://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_updates_and_backports
deb http://mirrors.ic.unicamp.br/debian/ trixie-updates main non-free-firmware contrib non-free  
deb-src http://mirrors.ic.unicamp.br/debian/ trixie-updates main non-free-firmware contrib non-free  
```

Agora, temos um repositório mais amplo e por isso precisaremos atualizá-lo:  
```
$ sudo apt update -y
Atingido:1 http://security.debian.org/debian-security trixie-security InRelease
Atingido:2 http://mirrors.ic.unicamp.br/debian trixie InRelease               
Atingido:3 http://mirrors.ic.unicamp.br/debian trixie-updates InRelease       
Obter:4 http://security.debian.org/debian-security trixie-security/main Sources [56,0 kB]
Obter:5 http://mirrors.ic.unicamp.br/debian trixie/non-free Sources [75,8 kB]   
Obter:6 http://mirrors.ic.unicamp.br/debian trixie/contrib Sources [52,3 kB]                          
Obter:7 http://mirrors.ic.unicamp.br/debian trixie/main Sources [10,5 MB]      
Atingido:8 https://dl.google.com/linux/chrome/deb stable InRelease                                   
Obter:9 http://mirrors.ic.unicamp.br/debian trixie/non-free-firmware Sources [6.536 B]
Obtidos 10,7 MB em 1s (12,0 MB/s)
Todos os pacotes estão atualizados.  
```


## INSTALANDO CODECS
Agora que habilitamos repositórios considerados 'non-free' e 'contrib' poderemos instalar alguns pacotes importantes que liberarão codecs e players de vídeo/musica em nosso sistema:
```
$ sudo apt install libavcodec-extra ffmpeg vlc -y
```

## INSTALANDO GOOGLE CHROME
O Google Chrome é muito popular e deveras alguns sites só funcionam bem com o motor dele. 
[Acesse a página de download dos pacotes do navegador Chrome](https://www.google.com/chrome/?platform=linux) e clique em Fazer o download do Google Chrome. Irá aparecer várias versões para Linux, escolha o pacote .deb de 64 bits para as plataformas Debian e Ubuntu.
Após o Download, dê duplo clique nele e o sistema irá dar inicio a instalação e daí, apenas siga as instruções em tela.


## INSTALANDO O HTOP, LMSENSORS e STRACE
Os comandos htop, lm-sensors e strace não vêm instalados por padrão, mas são muito úteis para gerenciar e diagnosticar o sistema diretamente pelo terminal. Eles servem para:  
* htop: gerencia tarefas no terminal, semelhante ao top, porém com interface mais amigável e interativa.
* lm-sensors: lê e exibe dados de sensores de temperatura, voltagem e ventoinhas disponíveis no hardware.
* strace: monitora chamadas de sistema e sinais usados por um processo — útil, por exemplo, para descobrir qual programa está acessando ou bloqueando um arquivo como arquivo.docx.

Gostou deles? Então execute:  
```  
sudo apt install -y htop lm-sensors  strace
```


## ATUALIZAÇÃO DE REPOSITÓRIO
Vamos atualizar o repositório de programas:  
```  
$ sudo apt -y update
```  

Agora, vamos atualizar o sistema:  
```  
$ sudo apt -y upgrade
Atualizando:                                
  google-chrome-stable

Resumo:
  Atualizando: 1, Instalando: 0, Removendo: 0, Não atualizando: 0
  Tamanho de download: 121 MB
  Espaço necessário: 7.168 B / 997 GB disponível

Obter:1 https://dl.google.com/linux/chrome/deb stable/main amd64 google-chrome-stable amd64 141.0.7390.65-1 [121 MB]
Obtidos 121 MB em 5s (23,1 MB/s)               
apt-listchanges: Lendo logs de mudanças...
(Lendo banco de dados ... 202444 arquivos e diretórios atualmente instalados).
Preparando para desempacotar .../google-chrome-stable_141.0.7390.65-1_amd64.deb ...
Desempacotando google-chrome-stable (141.0.7390.65-1) sobre (141.0.7390.54-1) ...
Configurando google-chrome-stable (141.0.7390.65-1) ...
Processando gatilhos para mailcap (3.74) ...
Processando gatilhos para man-db (2.13.1-1) ...
```  
No exemplo acima, apenas o google-chrome requer atualização.


## INSTALANDO O SILVERSEARCH-AG(ag)
O Silversearcher-ag, também conhecido apenas como ag, é uma ferramenta de busca extremamente rápida para código-fonte e arquivos de texto.
Ele é similar ao comando grep, porém muito mais veloz e prático, sendo ideal para desenvolvedores e administradores que precisam localizar trechos de texto em grandes projetos.
```
sudo apt install -y silversearcher-ag
```


## INSTALAÇÃO DE FERRAMENTAS DE DOWNLOAD (WGET E CURL)
O comando abaixo instala duas ferramentas essenciais para realizar downloads e requisições web diretamente pelo terminal Linux:    
```
sudo apt install -y wget curl 
```
Descrição dos componentes:
* WGET: Utilitário simples e confiável para baixar arquivos via HTTP, HTTPS e FTP.
* CURL: Ferramenta mais avançada para transferir dados ou interagir com APIs usando diversos protocolos (HTTP, HTTPS, FTP, SCP, etc.).
Esses programas são amplamente usados em scripts, automações e testes de conectividade.


## INSTALANDO COMPACTADORES/DESCOMPACTADORES DE ARQUIVOS
Os utilitários de compactação e descompactação não vêm todos instalados por padrão em uma instalação mínima do Debian.
Por isso, é recomendável instalar os pacotes abaixo para garantir suporte aos formatos mais comuns.
```
sudo apt install -y tar zip unzip p7zip-full p7zip-rar rar unrar lzip lzma xz-utils bzip2 gzip squashfs-tools
```

Pacote|Função / Formato
|:--|:--|
tar|Criação e extração de arquivos .tar e .tar.gz
zip, unzip|Manipulação de arquivos .zip
p7zip-full|Suporte a arquivos .7z (formato 7-Zip)
p7zip-rar, rar, unrar|Suporte a arquivos .rar
lzip, lzma, xz-utils, bzip2, gzip|Compactações livres amplamente usadas em pacotes Linux
squashfs-tools|Criação e extração de arquivos .squashfs


## INSTALANDO PROGRAMAS BASICOS PARA RECOMPILAR
Os pacotes a seguir servem para quem pretende compilar algo no ambiente Linux. Se você pretende instalar o driver proprietário da NVIDIA fornecido pela NVIDIA você também precisará deles:
```
sudo apt install -y build-essential
sudo apt install -y dh-make exuberant-ctags dpkg-dev debhelper fakeroot
sudo apt install -y exuberant-ctags module-assistant dkms patch libssl-dev
sudo apt install -y libncurses-dev ack fontconfig imagemagick git meson sassc 
```

## OBTENHA O KDE COMPLETO
O KDE que acompanha o Debian é uma versão leve, sem todos os módulos do KDE, por exemplo, não acompanha o modulo de compartilhamento de arquivos que muitas vezes pode ser necessário para desenvolvedores e administradores de sistemas, então se desejar um KDE mais cheio de funções, execute:
```  
sudo apt install -y kde-full
```
Depois disso, *recomendo que reinicie o computador*.


## ATIVE O SUPORTE A FLATPAK CENTRAL
O flatpak não está instalado ou habilitado em nosso sistema, para hablitá-lo, basta rodar o seguinte comando no terminal:
```  
sudo apt install -y flatpak
# para GNOME, instale também:
sudo apt install gnome-software-plugin-flatpak
# para KDE, instale também:
sudo apt install plasma-discover-backend-flatpak
```
Os pacotes alternativos para GNOME ou KDE são para maior integração dessas DE's ao flathub.

Caso as instrções acima não funcionem, visite a página com informações atualizadas:   
[https://flatpak.org/setup/Debian](https://flatpak.org/setup/Debian)  

Depois disso, adicionamos enfim, o repositório:
```
$ flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
Note que os diretórios 

'/var/lib/flatpak/exports/share'
'/home/gsantana/.local/share/flatpak/exports/share'

não estão no caminho de pesquisa definido pela variável de ambiente
XDG_DATA_DIRS, portanto, os aplicativos instalados pelo Flatpak podem não
aparecer em sua área de trabalho até que a sessão seja reiniciada.
```
Note a advertência, precisamos incluir as pastas citadas na variavel de ambiente XDG_DATA_DIRS, para testar, execute:
```
$ echo $XDG_DATA_DIRS


```
Mas calma, isso já foi feito, acontece que os caminhos indicados só serão vistos depois que você fizer o logout e login. Então faça isso.  

Depois de encerrar a sessão e fazer um novo login, vamos testar novamente:
```
$ echo $XDG_DATA_DIRS
/home/gsantana/.local/share/flatpak/exports/share:/var/lib/flatpak/exports/share:/usr/local/share:/usr/share
```
Ai está, tudo certo! Mas digamos que continua a retornar vazio, então vamos corrigir isso, execute:
```
nano ~/.bash_profile
```  
e acrescente as seguintes linhas no final:
```
export XDG_DATA_DIRS="/var/lib/flatpak/exports/share:$HOME/.local/share/flatpak/exports/share:$XDG_DATA_DIRS"  
```
Salve o arquivo e saia do editor.

Depois, encerre a sua sessão e faça um novo login, notará que agora, esses caminhos estão atualizados:
```
$ echo $XDG_DATA_DIRS
/home/gsantana/.local/share/flatpak/exports/share:/var/lib/flatpak/exports/share:/usr/local/share:/usr/share
```
### IMPORTANTE
Não é uma boa ideia instalar programas do flathub que são fornecidos por também pelos desenvolvedores originais, por exemplo:  
* Google Chrome, o próprio Google os fornece diretamente  
Isto acontece porque geralmente os pacotes fornecidos pelo flathub são feitos pela comunidade que pode diferir do autor original com alguma mistura de plugins adicionais ou modificações que carecem de verificação. 
Mas também há desenvolvedores que publicam seu próprio programa no flathub, exemplo:  
* Mozilla Firefox, a própria Mozilla publica seu software no flathub
* Telegram, a própria Telegram publica seu software no flathub
 

Se não houver desconfiança sob quem publica os programas no flathub, geralmente tais programas são mais seguros porque rodam sob container, isto é, estão limitados a pastas como: 
```
~/.local  
~/.var
```  
E esses programas geralmente não tem acesso ao seu $HOME a menos que você os conceda, e neste caso um link simbolico nas pastas acima irão apontar para seu $HOME para que o aplicativo tenha acesso a ele.  


## DOMINIO DE REDE LOCAL  
Se tiver um dominio de rede, edite o arquivo */etc/samba/smb.conf* e vá até a linha:    
```  
WORKGROUP = WORKGROUP
```  
e troque por:  
```  
WORKGROUP = meudominioderedelocal.lan
```  
Salve e saia do editor.  
Com essa modificação, quando acessar uma pasta compartilhada na rede, o nome 'meudominioderedelocal.lan' já aparecerá como padrão na tela de autenticação de usuário e retardará problemas futuros de lesão por esforços repetitivos.  


## MUDANDO O NOME DO HOST  
Durante a instalação do Debian, você provavelmente definiu um nome para o seu computador (hostname).
Entretanto, caso queira modificá-lo depois, é possível fazer isso facilmente.  
Pelo ambiente gráfico (KDE ou GNOME), abra o aplicativo Configurações do Sistema, e na barra de pesquisa, digite “host”, "sistema" ou algo similar e essas informações serão exibidas e passiveis de modificações. A cada versão do KDE e GNOME, essas opções mudam de lugar ou são traduzidas de forma diferente o que impede de trazer um screenshot. Mas pelo terminal, é bem mais eficiente, basta executar:  
``` 
sudo hostnamectl set-hostname novo-nome
``` 


## HABILITANDO O MÓDULO “COMPARTILHAMENTO” NO KDE  
Por padrão, o Debian com KDE (Plasma) não exibe a opção “Compartilhamento” nas Configurações do Sistema, diferentemente do Fedora, onde ela vem habilitada. Isso ocorre porque o módulo depende de pacotes adicionais que não são instalados automaticamente, para habilitar o módulo, instale os pacotes abaixo:
``` 
sudo apt install -y plasma-widgets-addons kdenetwork-filesharing samba 

sudo systemctl start samba
``` 
Depois, encerre a sessão e a inicie novamnete.  
Agora vá em Configurações do Sistema>Compartilhamento>Compartilhamento de Arquivos, o módulo agora estará disponível e exibirá também o campo “Nome do Computador (Hostname)”, permitindo alterar o nome da máquina de forma gráfica:  
![Habilitando o compartilhamento de arquivos](kde-ativar-compatilhamento.png)  


## HABILITANDO AREA DE TRABALHO REMOTA
Ainda no programa “Configurações”, então procure por “host” ou “compartilhamento” e então encontre a seção “Compartilhamento”. Neste momento queremos ativar o compartilhamento arquivos:

![Habilitando area de trabalho remota](./area_trabalho_remota.png)

Quando precisar acessar sua estação de trabalho remota você poderá usar aplicativos como o remmina. Bastará fornecer a URL que indica seu computador como mostrado acima em destaque e então fornecer as credenciais fornecidas.  
💡 DICA:  
Mesmo que você não pretenda ativar o compartilhamento de arquivos, é seguro instalar esses pacotes apenas para obter acesso ao painel de configuração e alterar o hostname de forma visual.  


## COMPARTILHAMENTO DE MULTIMEDIA
Ainda no programa “Configurações”, então procure por “host” ou “compartilhamento” e então encontre a seção “Compartilhamento”. Neste momento queremos ativar o compartilhamento de multimedia, que na verdade é uma seção para escolher o que devo compartilhar, enquanto a seção HABILITANDO O COMPARTILHAMENTO DE ARQUIVOS habilita o compartilhamento de arquivos, a seção COMPARTILHAMENTO DE MULTIMEDIA determina o que deve ser compartilhado:

![Escolhendo o que devo compartilhar](./compartilhar_arquivos_selecionar_pastas.png)

Note na imagem que também é possivel acrescentar outras pastas.
Normalmente, estes compartilhamento são visiveis a partir de outras estações linux e também estações windows. A partir de estações Windows basta chamar como \\computador e os compartilhamentos que houverem no computador serão exibidos.

## HABILITANDO SESSÃO REMOTA
Carregue o programa “Configurações”, então procure por “host” ou “compartilhamento” e então encontre a seção “Compartilhamento”. Queremos no momento ativar o compartilhamento de sessão remota, isto é, permitir que outras pessoas na rede possam abrir uma sessão via ssh ou display grafico (ssh -X) neste computador:

![Habilitando sessão remota](./sessao_remota_ativar.png)

Note que na imagem é mostrado um exemplo de como posso usar o ssh para abrir uma sessão.

## PRELOAD
Se estiver usando discos mecanicos, provavelmente sente muita latencia para carregar certos progrmas. Numa situação assim, é bom instalar um serviço chamado 'preload', ele monitora os programas que você mais utiliza e durante o boot já os carrega para você. A vantagem é a velocidadade para carregá-los na memória, note porém que tais programas SEMPRE ESTARÃO NA MEMÓRIA e com isso, o tamanho da sua memória irá abaixar, por isso, só recomendo que use este programa com moderação e com discos mecânicos que são lentos, não há vantagens em discos SSD ou NVME. Para instalar:
```
sudo apt install -y preload
```

## INSTALANDO PERFIS DE USO (TUNED)
O tunned é um programa que permite trocar em tempo real o perfil de desempenho do compuador, por exemplo, posso usar o perfil de desempenho balanceado quando quero navegar na internet e de um momento para outro trocar o perfil de desempenho para 'realtime' quando quero maximimizar a performance. Há outros perfis prontos para usar maquinas virtuais, economia de energia, etc... O programa tem muitos perfís e é altamente recomendado, vamos a instalação:
```
sudo apt install -y tuned
```
Liste os perfís de otimização existentes:
```
$ sudo tuned-adm list
Available profiles:
- accelerator-performance     - Throughput performance based tuning with disabled higher latency STOP states
- atomic-guest                - Optimize virtual guests based on the Atomic variant
- atomic-host                 - Optimize bare metal systems running the Atomic variant
- aws                         - Optimize for aws ec2 instances
- balanced                    - General non-specialized tuned profile
- balanced-battery            - Balanced profile biased towards power savings changes for battery
- cpu-partitioning            - Optimize for CPU partitioning
- cpu-partitioning-powersave  - Optimize for CPU partitioning with additional powersave
- default                     - Legacy default tuned profile
- desktop                     - Optimize for the desktop use-case
- desktop-powersave           - Optmize for the desktop use-case with power saving
- enterprise-storage          - Legacy profile for RHEL6, for RHEL7, please use throughput-performance profile
- hpc-compute                 - Optimize for HPC compute workloads
- intel-sst                   - Configure for Intel Speed Select Base Frequency
- laptop-ac-powersave         - Optimize for laptop with power savings
- laptop-battery-powersave    - Optimize laptop profile with more aggressive power saving
- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
- mssql                       - Optimize for Microsoft SQL Server
- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
- network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
- openshift                   - Optimize systems running OpenShift (parent profile)
- openshift-control-plane     - Optimize systems running OpenShift control plane
- openshift-node              - Optimize systems running OpenShift nodes
- optimize-serial-console     - Optimize for serial console use.
- oracle                      - Optimize for Oracle RDBMS
- postgresql                  - Optimize for PostgreSQL server
- powersave                   - Optimize for low power consumption
- realtime                    - Optimize for realtime workloads
- realtime-virtual-guest      - Optimize for realtime workloads running within a KVM guest
- realtime-virtual-host       - Optimize for KVM guests running realtime workloads
- sap-hana                    - Optimize for SAP HANA
- sap-hana-kvm-guest          - Optimize for running SAP HANA on KVM inside a virtual guest
- sap-netweaver               - Optimize for SAP NetWeaver
- server-powersave            - Optimize for server power savings
- spectrumscale-ece           - Optimized for Spectrum Scale Erasure Code Edition Servers
- spindown-disk               - Optimize for power saving by spinning-down rotational disks
- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
- virtual-guest               - Optimize for running inside a virtual guest
- virtual-host                - Optimize for running KVM guests
Current active profile: balanced
```
No exemplo acima, estou usando o perfil 'balanced', caso ele não apareça, use:
```
$sudo tuned-adm active
Current active profile: balanced
```
O modo 'balanceado' é o modo não especializado, se desejar especializar seu computador em algo, por exemplo, para usar o desktop, execute:  
```
sudo tuned-adm profile desktop  
```  
E então o sistema será especializado ou otimizado para ser usado como 'desktop'.   

Perfis muito comuns para quem usa laptop:  

🔌 laptop-ac-powersave - “Optimize for laptop with power savings”    
Uso: notebook ligado na tomada (AC).  
Objetivo: manter bom desempenho, mas ainda economizar energia onde possível.  
Ajustes típicos: Habilita CPU frequency scaling (a CPU reduz clock quando ociosa). Mantém turbo boost ativado para tarefas pesadas. Reduz brilho de tela e consumo de periféricos em idle.  
Mantém discos e interfaces de rede em modo balanceado.    
Resumo: bom equilíbrio entre desempenho e economia. 
📘 Ideal para uso diário com o notebook conectado à energia.  

🔋 laptop-battery-powersave - “Optimize laptop profile with more aggressive power saving”
Uso: notebook usando bateria.  
Objetivo: maximizar autonomia, mesmo sacrificando desempenho.  
Ajustes típicos: CPU limitada a clocks mais baixos. Desativa turbo boost e núcleos ociosos. Reduz brilho e tempo de suspensão automática. Interfaces Wi-Fi e Bluetooth podem entrar em modos de economia agressiva. Discos mecânicos são parados rapidamente quando inativos.  
Resumo: desempenho menor, mas maior duração de bateria.  
📘 Ideal para uso em viagens, reuniões ou campo.

⚙️ latency-performance - “Optimize for deterministic performance at the cost of increased power consumption”
Uso: servidores ou estações de trabalho que exigem baixa latência e previsibilidade (mas não tempo real).
Objetivo: garantir resposta consistente, mesmo com maior consumo de energia.
Ajustes típicos: Desativa CPU frequency scaling → clock fixo máximo. Desativa C-states profundos e economias de energia. Ajusta IRQs e afinidade de CPU para reduzir jitter. Mantém memória e dispositivos em estado ativo constante.
Resumo: máxima estabilidade e latência previsível, sem foco em economia.
📘 Ideal para bancos de dados, servidores de aplicações ou jogos que exigem resposta constante.

Outros perfis muito uteis para desenvolvedores são:
🕒 realtime - “Optimize for realtime workloads”
Uso: sistemas físicos (bare metal) com necessidades críticas de tempo real.
O que faz: Reduz a latência ao máximo. Ajusta o escalonador de CPU para favorecer tarefas de tempo real. Desativa power saving features (como C-states e turbo boost). Fixa frequências da CPU em nível máximo. Ajusta IRQs e prioridade de processos.
Exemplo de uso: estações de áudio profissional (JACK), robótica, processamento de sinais, sistemas de controle industrial.

🧩 realtime-virtual-guest - “Optimize for realtime workloads running within a KVM guest”
Uso: máquinas virtuais (guests) executando workloads de tempo real dentro de um host KVM.
O que faz: Aplica otimizações similares ao perfil realtime, mas levando em conta que o controle de hardware é mediado pelo hipervisor. Ajusta parâmetros de temporização (clocksource, tickless, etc.) para sincronizar com o host. Minimiza interferência do kernel convidado.
Exemplo: uma VM rodando um sistema de controle de robô industrial, ou processamento de áudio, dentro de um servidor KVM.

🖥️ realtime-virtual-host - “Optimize for KVM guests running realtime workloads”
Uso: no host KVM que executa VMs que, por sua vez, têm workloads de tempo real.
O que faz: Garante que as VMs de tempo real recebam CPU e I/O com mínima latência. Usa CPU pinning e isolcpus para isolar núcleos destinados às VMs RT. Minimiza a interferência do host em threads de tempo real. 
Exemplo: servidor KVM que hospeda várias VMs RT, como sistemas de automação ou simulações científicas críticas.


## COMPLETANDO O IDIOMA PORTUGUÊS
O idioma português-brasil não está completamente instalado, para isso execute o programa “system-config-language”, porém ele não está instalado por padrão, execute:
```
sudo apt install locales task-laptop task-portuguese task-portuguese-desktop
```
💡 O pacote locales fornece os idiomas do sistema; os pacotes task-* completam tradução de menus, ajuda e aplicativos do ambiente gráfico.

Se usa KDE, GNOME, XFCE etc., vá em Configurações do sistema>Região e Idioma>Idioma>Português (Brasil):  
![Mudando ou atualizando o idioma](debian-regiao-idioma.png)  
Você pode aproveitar o momento e remover os idiomas desnecessários, mas não remova o inglês, poderemos requerer ele em algum momento, por exemplo, verificação ortográfica de um texto escrito em inglês.  

Depois disso, vá em Configurações do sistema>Região e Idioma>Idioma>Verificação ortográfica e:  
* Idioma padrão: Português(Brasil)  
* Ativar detecção automática de idioma: Ligado  
* Verificação ortográfica automática ativada por padrão: Ligado  
* Ignorar palavras com toda as letras em maiúsculo: Desligado  
* Ignorar palavras coladas: Ligado  
Como visto nesta imagem:

![Ativando correção ortográfica(debian-regiao-idioma-ortografica.png)  

Depois, encerre e entre novamente na sessão.  

## GIT
Vamos ajustar nosso ambiente com o GIT com os comandos:
```
git config --global user.name "Seu nome completo"
git config --global user.email "seu.email@dominio.com"
```

Recentemente, o github fez alterações em seu sistema onde a instrução:
git config credential.helper 'cache --timeout=28800' 
Será ignorada completamente ou terminará em erro e a tentativa de login resultará neste erro:
**Fatal Authentication Failed for: site.com.br**

Para solucionar o problema, precisará instalar e configurar o ‘Git Credential Manager’ (GCM):
```
sudo dnf install -y seahorse
sudo dnf install -y git-credential-libsecret
```
Após, o git só precisará dessa configuração adicional:
```
git config --global credential.helper /usr/libexec/git-core/git-credential-libsecret
```
Agora, você precisa saber que o método de autenticação mudou, você não usa mais o “username+ senha” do seu usuario git, mas “username+token”. O token é criado na página do github, no menu de sua profile->Settings->Developer settings->Personal access tokens->Tokens(classic) e então criar um token. Este token será o substituto de sua senha git no terminal.


## PARTIÇÕES NTFS NO SISTEMA
Se utiliza uma partição Windows (NTFS) para gravar seus arquivos e dados a partir do Linux, você pode simplesmente não fazer nada e usar o gerenciador de arquivos do GNOME para entrar e sair do disco NTFS quando quiser. Contudo, se você tem que ir para o terminal e acessá-lo dali então lhe seria conveniente criar uma pasta vazia que ao entrar nela você já observasse o conteúdo dessa partição, se você gostou da idéia então vamos implementá-la.  
Primeiro, identifique corretamente qual é o seu disco/partição com NTFS, execute no terminal:  
```
sudo blkid |grep ntfs
/dev/nvme1n1p2: LABEL="Windows" BLOCK_SIZE="512" UUID="389083EB9083AE46" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="f8ff4bca-3ef8-4ba7-b1b1-6e0b00689aab"
/dev/nvme1n1p3: LABEL="DADOS" BLOCK_SIZE="512" UUID="1EB4CCF2B4CCCE09" TYPE="ntfs" PARTLABEL="Basic data partition" PARTUUID="0acfc054-4ac1-46f6-83e3-c90bb5e79f12"
```
No exemplo anterior, o disco desejado tem o LABEL=DADOS e UUID="1EB4CCF2B4CCCE09" , então vamos criar uma pasta vazia, preferencialmente com o nome da label do disco na $HOME ou /media (ou /mnt), ex:  
/home/usuario/label_dados ou /media/label_dados  
A vantagem de ficar em $HOME é que com os cuidados necessários apenas você terá acesso a ela, mas se prefere disponibilizar a partição para todos então tente:
```
sudo mkdir -p /media/label_dados
sudo chmod 1777 /media/label_dados
sudo chmod +s /bin/ntfs-3g
sudo usermod -aG disk $USER 
```

Estamos deixando claro que /media/label_dados estará disponível a todos os usuários. Se escolher criar uma pasta em $HOME, não há a necessidade das linhas acima, apenas criar a pasta vazia.

Execute no terminal:
```
sudo nano /etc/fstab
```
Agora vamos editar o arquivo /etc/fstab e então acrescentar após a última linha:
```
# Minha partição NTFS de label “DADOS”
UUID="1EB4CCF2B4CCCE09" /media/label_dados ntfs-3g windows_names,nosuid,nodev,nofail,rw,user,gid=users,noauto 0 0
```
Explicando os demais parâmetros:
nosuid: Bloqueia a operação de bits suid e sgid
nodev: Não interpreta dispositivos especiais de bloco no sistema de arquivos.
nofail: faz com que o dispositivo seja montado mesmo que tenha sido marcado com erro. Um drive é marcado com ‘erro’ quando o computador é desligado abruptamente ou quando qualquer unidade listada no fstab não está presente ou falhou na montagem.
zero e zero no final da linha: todo


Uma outra forma de escrever essa linha no fstab seria:
```
UUID="1EB4CCF2B4CCCE09" /media/label_dados ntfs nls-utf8,rw,nosuid,nodev,nofail 0 0
```
A diferença é que ao usar driver “ntfs-3g” você estará usando um driver do tipo userspace(pacote ntfs-3g precisa estar instalado) considerado o método mais moderno e seguro, enquanto “ntfs” está ligado a um módulo diretamente no kernel do linux que geralmente recomendamos usar apenas no modo de leitura que impede qualquer gravação na unidade. Mas pode-se habilitar o modo de leitura e escrita se desejar, já testei ambos no modo leitura/escrita e prefiro o “ntfs-3g” que além de ser mais seguro, possui outros parâmetros que nos ajudam a evitar nomes de arquivos que podemos criar usando Linux, mas que o Windows terá problemas com eles, por exemplo o uso de caracteres como  “:” e “?” em nomes de arquivos.

Salve e após o boot, a pasta indicada servirá de acesso a partição NTFS.
Se você não quiser auto montá-la no boot, mas mantê-la apenas quando executar no terminal:
```
mount /media/label_dados
```
Então troque “auto” para “noauto”. O “noauto” é mais seguro por impedir que programas instalados ou scripts maliciosos tenham acesso a esta partição ou disco.
Caso precise reparar a unidade NTFS, execute:
```
sudo ntfsfix /dev/disk/by-uuid/34F84B57F84B1690
```
ou se souber o label do disco:
```
sudo ntfsfix /dev/disk/by-label/DADOS
```
Alternativas: Existe um serviço chamado AutoFS, ele implementa uma solução onde você indica pastas e apenas quando você acessá-las, ele as monta. Serve para discos externos, partições internas e também para compartilhamentos remotos. Esta última, é o motivo pelo qual é mais usado visto que auto-montar pastas que já estão em nosso domínio é mais fácil usando o fstab. AutoFS é um pouco mais complicado que usar /etc/fstab, mas nem tanto depois que você entende como ele funciona. Eu tenho receio de utilizá-lo em ambientes com pouco controle porque se houver programas que vasculhem discos eles irão montar todas as pastas que encontrarem na configuração para auto montar, talvez  voce pense na situação de vírus de computador, mas ocorreria algo similar em softwares de backups que podem erroneamente incluir pastas que não deveriam. Se quiser estudar o AutoFS:

https://devconnected.com/how-to-install-autofs-on-linux/

## HABILITANDO OS ÍCONES DA BANDEJA NO GNOME
O GNOME nas suas últimas opções não vem mais com a bandeja do sistema(tray) e por isso algumas programas que antes usavam ela para ancorar opções podem deixar de funcionar. Para tê-la de volta será necessário instalar extensões do GNOME. Essas extensões na maior parte das vezes são instaladas diretamente do site oficial, mas por algumas serem tão populares também estão nos repositórios e pré-instaladas, então você já tem elas instaladas.
Visite o site(será preciso abrir uma conta):
https://addons.mozilla.org/en-US/firefox/addon/gnome-shell-integration/
Se achar a tela abaixo, então a mesma já está em seu sistema, caso contrário instale-a:
![Figura](./fig.png)

Outra extensão altamente recomendada é essa:
https://extensions.gnome.org/extension/2890/tray-icons-reloaded/
https://extensions.gnome.org/extension/615/appindicator-support/

## BLOQUEIO DE TELA AUTOMÁTICO
O sistema normalmente é ajustado automaticamente para bloquear após 5 minutos de atividade, mas ‘falta de atividade’ é um termo incorreto, o correto seria ‘tempo sem interatividade’, isto é, o tempo que você fica sem ter que interagir com o computador. Às vezes estamos processando algo demorado e temos de esperar ou acompanhar a movimentação de log de status e o computador durante este tempo estará tendo muito trabalho, porém com pouca interatividade a tela será bloqueada. Então precisamos saber quanto tempo precisamos nas tarefas do dia a dia ou então desligá-la.

Vá em configurações->Privacidade->Tela de bloqueio:
![Opção de privacidade e tela de bloqueio](./gnome_tela_bloqueio01.png)

E então ajuste o tempo:
![Opção de privacidade e tela de bloqueio](./gnome_tela_bloqueio02.png)

Ou até mesmo desligue-o se isso for preciso. Note que na imagem acima, també, foi desligado as notificações, isso é especialmente util quando você deixa o local e outras pessoas podem passar na frente de sua tela e ler suas notificações que seriam exibidas mesmo com a sessão bloqueada.

## AJUSTANDO O PROMPT NO TERMINAL
Às vezes o prompt do terminal pode incomodar alguns, por exemplo, é justo que ao logarmos em servidores o terminal revele no prompt seu username e nome do computador:
![Prompt normal](./mudando_prompt01.png)
porém ao usarmos o desktop sabemos quem somos e que computador é, então vamos ajustar o terminal para não mostrar essas duas informações. A variável de ambiente que gostaríamos de modificar que faz o prompt refletir o que desejamos chama-se PS1 e podemos ajustá-la assim::
```
export PS1='${debian_chroot:+($debian_chroot)}\[\033[32;40m\]\w:\[\033[00m\] ' 
```
Ele deixará nosso prompt "oldschool", colorido:
![Novo prompt](./mudando_prompt02.png)

Note na imagem acima, nosso prompt deixou de ser o que era antes para ser uma forma verde “oldschool” com o nome da pasta onde estamos, agora não esta mais exibindo username ou computername.

Se você não gosta de exibir o caminho completo do diretório onde você está porque prefere diminuí-lo, então você deve trocar o código \w por \W, a diferença entre eles é que o W maiúsculo mostra apenas o nome do diretório que você está sem o caminho completo. Muitas pessoas preferem desse jeito e caso queiram saber o caminho apenas executam o comando **pwd**.  Vamos mostrar um prompt classico com dois pontos, porém na cor azul:

```
export PS1='${debian_chroot:+($debian_chroot)}\[\033[01;34m\]\w:\[\033[00m\] ' 
```
![Novo prompt](./mudando_prompt03.png)

Ou se quiser o mesmo prompt acima, mas na classica cor verde:
```
export PS1='${debian_chroot:+($debian_chroot)}\[\033[32;40m\]\w:\[\033[00m\] '
```
![Novo prompt](./mudando_prompt04.png)

Os “:” no meio da sentença pode ser trocado por um caractere unicode mais bacana:
```
export PS1="\e[32;40m\w➤\e[00m "
```
![Novo prompt](./mudando_prompt05.png)

Muito melhor, não é mesmo? Há uma página que descreve várias formas de como deixar seu prompt:  
https://www.ibm.com/developerworks/linux/library/l-tip-prompt/ 
Experimente todas as opções que puder e ao final determine o prompt que deseja usar. Mas tem um problema, ao definir o tipo de prompt que desejamos, não queremos executar “export PS1=...”  todas as vezes que formos usar o terminal, isso nos daria muito trabalho. Precisamos de um jeito de automatizar isso, então o primeiro passo é descobrir o tipo de terminal que usamos, execute: 
```
echo $TERM
xterm-256color
```
Agora que sabemos que o nome é xterm-256color então edite o arquivo ~/.bashrc (o til representa a localização da sua pasta $HOME), então editamos este arquivo:  
```
gnome-text-editor ~/.bashrc
```
E ao final do arquivo ou numa localidade melhor, pois alguns .bashrc as vezes tem IFs que discriminam terminal colorido de não colorido vou acrescentar:
# Meu ajuste de terminal ao estilo old school
export PS1='${debian_chroot:+($debian_chroot)}\[\033[32;40m\]\w➤\[\033[00m\] '
Ficando mais ou menos assim nosso arquivo
```
(...)  
# User specific aliases and functions  
if [ -d ~/.bashrc.d ]; then  
    for rc in ~/.bashrc.d/*; do  
        if [ -f "$rc" ]; then  
            . "$rc"  
        fi  
    done  
fi  
  
# Meu ajuste de terminal ao estilo old school  
export PS1='${debian_chroot:+($debian_chroot)}\[\033[32;40m\]\w➤\[\033[00m\] '
  
unset rc
```
A partir de agora, quando abrir o terminal, seu prompt será assim:  
![Novo prompt](./mudando_prompt06.png)  
Muito bacana, hein?


## INTEGRAÇÃO DO GNOME SHELL COM O FIREFOX
As integrações dos navegadores com o ambiente GNOME é permitida através de extensões, com elas instaladas você pode fazer mudanças no ambiente gnome shell como alterar o papel de parede do gnome ao visitar a página gnome-look.org ou instalar extensões em extensions.gnome.org. 
Instale a extensão para o Firefox:
https://addons.mozilla.org/pt-BR/firefox/addon/gnome-shell-integration/

## ALGUMAS EXTENSÕES ÚTEIS
As extensões a seguir lhe serão úteis
```
sudo dnf install -y gnome-extensions-app gnome-tweaks
~sudo dnf install -y gnome-shell-extension-appindicator~
```

## PARA TREINAMENTO
Para criar material de treinamento que incluirá vídeo é sugerível instalar a seguinte extensão Draw On Your Screen cuja instrução para instalação se encontra em:
https://codeberg.org/som/DrawOnYourScreen

```
git clone https://codeberg.org/som/DrawOnYourScreen --depth=1 --single-branch --branch face ~/.local/share/gnome-shell/extensions/draw-on-your-screen@som.codeberg.org
```
Depois vá até .local/share/gnome-shell/extensions e abra o arquivo metadata.json e adicione "41" e então reinicie o gnome-shell.


## GNOME DASH TO DOCK
O GNOME não possui uma dock, isto é, um local para repousar os aplicativos favoritos ou atualmente carregados. O time do GNOME espera que você use o botão super ou ALT+TAB para listá-los ou carregá-los de seu painel de programas. Porém, muitos estão acostumados a uma dock, se você é uma delas então instale a extensão dash to dock:

https://extensions.gnome.org/extension/307/dash-to-dock/


## HABILITANDO OS ÍCONES DA BANDEJA NO GNOME
Outra extensão altamente recomendada é essa:
https://extensions.gnome.org/extension/2890/tray-icons-reloaded/
https://extensions.gnome.org/extension/615/appindi	cator-support/


## GERENCIANDO A CLIPBOARD
(todo)


## GNOME-TWEAKS
(todo)


## INSTALAÇÃO DE FONTES DE CARACTERES ADICIONAIS
Vamos adicionar um repositório que nos será util para acrescentar mais fontes ao sistema:
```
sudo dnf install -y curl cabextract xorg-x11-font-utils fontconfig
sudo rpm -i https://downloads.sourceforge.net/project/mscorefonts2/rpms/msttcore-fonts-installer-2.6-1.noarch.rpm
```
O pacote instalado acima complementará as fontes microsoft de que alguns programas portados do Windows talvez precisem.

A fonte ‘fonts-roboto’ é bastante interessante para uso em terminal e IDEs de programação:
sudo dnf install -y google-roboto-fonts google-roboto-condensed-fonts google-roboto-mono-fonts google-roboto-slab-fonts

A fonte Hack é bastante apropriada para ser usada para listar codigo fonte de programas ou utilizar o terminal, sua instalação pelo repositório é simples:
```
sudo dnf copr enable zawertun/hack-fonts
sudo dnf install -y hack-fonts
```
Mas recomendo sua instalação manual, pois se instalada em $HOME a mesma poderá ser reaproveitada em futuras reinstalações, visite a página:
https://github.com/source-foundry/Hack
e siga as instruções, a saber:
```
cd /tmp
wget -vc https://github.com/source-foundry/Hack/releases/download/v3.003/Hack-v3.003-ttf.zip
unzip Hack-v3.003-ttf.zip -d ~/.local/share/fonts/
fc-cache -f -v
```
Para conferir se a fonte foi realmente instalada, executamos:
```
fc-list | grep "Hack"
```
Se aparecer o nome da fonte em ~/.local/share/fonts/ttf/Hack-BoldItalic.ttf: Hack:style=Bold Italic e assim por diante é porque a fonte foi instalada com sucesso.


## Fonte "CONSOLAS"
A fonte “consolas” é uma interessante fonte para ser usada tanto em desenvolvimento e aplicativos como também no ambiente gráfico. Ela é de propriedade de terceiros e por isso não vem acompanhada dentro das distribuições Linux, mas é possível instalá-las. Para instalar siga as instruções:
```
cd /tmp
wget -O /tmp/YaHei.Consolas.1.12.zip https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/uigroupcode/YaHei.Consolas.1.12.zip
unzip YaHei.Consolas.1.12.zip -d ~/.local/share/fonts/ 
fc-cache -f -v
fc-list | grep "Consolas"
```


## EXTENSÕES DO GNOME SHELL
Extensões do GNOME SHELL acrescentam funcionalidades ou mudam comportamento. A partir do GNOME 41 muitas extensões listadas abaixo deixaram de funcionar, mas deixo elas listadas porque em algum momento poderão voltar a funcionar. Alguns são considerados necessários, outros opcionais e ainda outros são bem específicos. Coloquei aqui algumas que considero necessários e opcionais.
Extensões necessárias:
Para áudio (escolha apenas uma):
~https://extensions.gnome.org/exten sion/906/sound-output-device-chooser/~
**Outras**
https://extensions.gnome.org/extension/19/user-themes/
https://extensions.gnome.org/extension/36/lock-keys/
https://extensions.gnome.org/extension/779/clipboard-indicator/
https://extensions.gnome.org/extension/1460/vitals/
https://extensions.gnome.org/extension/3902/eye-and-mouse-extended/

**Outras que são opcionalmente úteis:**
https://extensions.gnome.org/extension/690/easyscreencast/
https://extensions.gnome.org/extension/945/cpu-power-manager/ (testar melhor)
https://extensions.gnome.org/extension/750/openweather/
https://extensions.gnome.org/extension/1238/time/

IMPORTANTE: Se por acaso alguma extensão se comportar mal, use o programa  “gnome-extensions-app” para removê-la ou configurá-la de forma diferente.


## USANDO O THEMAS VARIADOS DO GNOME SHELL
O GNome Shell usa um conjunto de critérios para focar no que você está fazendo e evitar distrações, mas nada disso adianta se não nos comprometermos em saber como as coisas funcionam. Por isso recomendo que leiam e vejam os vídeos contidos na página:
https://material-shell.com/

Se você usa muito o computador, recomendo que dê atenção às teclas de atalho:
https://material-shell.com/#hotkeys

## USANDO O TERMINAIS VIRTUAIS DO GNOME SHELL
Quem vem do ambiente Windows parece odiar o GNOME justamente porque é muito diferente do Windows, afinal cadê o minimizar, maximizar, painel inferior, menu iniciar, bandeja, etc… porém, se você se habituar a utilizar algumas janelas lado a lado ou sempre maximizados então terá de se acostumar a alguns atalhos:

Super+Shift+PageDown[PageUp] para descer[ou sumir] a aplicação para a workspace abaixo[ou acima]
Super+PageDown[PageUp] para descer ou subir entre as workspaces
Super+AWSD permite movimentar-se entre as áreas virtuais. AWSD podem ser substituídas pelas setas, quem usa notebook com teclados que combinam as setas com Fn ficará mais à vontade em usar AWSD.
Não uso tanto, mas são poucos conhecidos.
Print Screen copia a tela inteira e grava-a na pasta Imagens
Alt+Print Screen copia a janela do aplicativo que estiver ativo grava-o na pasta Imagens
Shft+Print Screen permite selecionar a área que deseja gravar e  grava-o na pasta Imagens



## INSTALAR O GOOGLE CHROME
Apesar do Firefox ser um excelente navegador, muitas vezes precisaremos do Google Chrome, para baixá-lo use a loja de software (Programas) e procure por “Google Chrome” e instale-o.



## INTEGRAÇÃO DO GNOME SHELL COM O CHROMIUM 
As integrações dos navegadores com o ambiente GNOME é permitida através de extensões, com elas instaladas você pode fazer mudanças no ambiente gnome shell como alterar o papel de parede do gnome ao visitar a página gnome-look.org ou instalar extensões em extensions.gnome.org. 
Também aplica-se a variações, incluindo o Google Chrome, execute no terminal:
```
sudo dnf install chrome-gnome-shell
```
## Visual Studio Code (repositório e instalação)

Para instalar visite:
https://code.visualstudio.com/docs/setup/linux

E siga as instruções, mas basicamente é:

Para acrescentar o repositório oficial do VS Code:
```
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
sudo sh -c 'echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" > /etc/yum.repos.d/vscode.repo'
sudo dnf check-update
sudo dnf install -y code
```
Relata-se um problema ao sincronizar configurações e extensões com o vscode:
**“Visual Studio Code is unable to watch for file changes in this large workspace” (error ENOSPC)**
Na internet encontrei o seguinte workaround que resolve tal problema:
```
sudo -i
echo 'fs.inotify.max_user_watches = 524288' >> /etc/sysctl.conf
sysctl -p
```
Se esta for sua primeira instalação visite novamente a página:
https://code.visualstudio.com/download

Nessa página há uma seção intitulada ‘Top Extensions’, guie-se por ela para instalar os plugins mais efetivos para cada linguagem que for utilizar.


## ZOOM CLOUD MEETINGS
Para baixá-lo use a loja de software (Programas) e procure por “Zoom” e instale-o.

## IMPRESSORA PDF
(todo)

## INSTALANDO A IMPRESSORA EPSON L355 LOCALIZADA NA REDE
(todo)

## INSTALANDO O SCANNER EPSON L355
(todo)


## INSTALANDO O LEITOR OCR
(todo)


## CRIANDO ATALHOS PARA PROGRAMAS CONHECIDOS
(todo)


## AUTO-CARREGAR PROGRAMAS NO INICIO DA SESSÃO
(todo)


## LIBERANDO PORTAS NO FIREWALL
Get a list of allowed ports in the current zone:
```
$ firewall-cmd --list-ports
```
Add a port to the allowed ports to open it for incoming traffic:
```
$ sudo firewall-cmd --add-port=port-number/port-type
```
Make the new settings persistent:
```
$ sudo firewall-cmd --runtime-to-permanent
```


## VIRTUALBOX
O VirtualBox é essencial para o pleno funcionamento do ambiente de desenvolvimento para instalar, precisaremos incluir o repositório oficial do Vitualbox. O virtualbox pode ser obtido diretamente no site:
https://www.virtualbox.org/
Sua instalação é simples, basta baixar o arquivo com a extensão .rpm e dar duplo clique nele.
Também é necessário abaixar “VirtualBox Oracle VM VirtualBox Extension Pack” que entrega alguns recursos extras.



## LEITOR DE CERTIFICADO DIGITAL
Cada leitor e modelo pode ter instruções diferentes, é melhor procurar um howto na internet apropriado.



## MICROSOFT OFFICE (web apps)

Visite a página a seguir usando um navegador Google Chrome(ou compatível com webapps):
office.com 
E autentique-se com uma conta live da Microsoft.
Estando na home da página vá no menu do navegador


## INSTALANDO O GIMP
Vá no app  Software e procure por GIMP no repositório do ‘Flathub’:

Clique nas propriedades dele e encontrará alguns plugins(complementos) que também poderá instalar, são eles:

BIMP - Realizar operações em batch com vários arquivos
https://www.youtube.com/watch?v=CaeTkgPNkkg

FocusBlur - Capacidade de efeito de profundidade
https://www.youtube.com/watch?v=u-YB-KipWzk

Gimp transformação Fourier
Técnica para remover ou manipular padrões em fotos, geralmente as antigas
https://www.youtube.com/watch?v=se9I3uGITR0

Gimp Lens Fans
Aplicar e corrigir efeitos por lentes
https://www.youtube.com/watch?v=FQGDgBT1tWk

G’MIC
GRAYC’s Efeitos
https://www.youtube.com/watch?v=kZnEpkNsDK0
https://www.youtube.com/watch?v=VOPHbSgJUSI

LiquidRescale
Permite remover um elemento e redimensionar uma imagem como se o elemento nunca estivesse existido.
https://www.youtube.com/watch?v=hhFVWKJA76U

Resynthesizer
Retire manchas ou outros defeitos de imagens
https://www.youtube.com/watch?v=n76owcpShqw

## ESPELHAMENTO DE CELULAR
Usaremos um programa chama scrcpy, para instalar executamos:
```
sudo dnf install -y adb android-tools 
sudo dnf install -y scrcpy scrcpy-server
```
Para usá-lo terá de ativar o modo de depuração de seu celular ou tablet:
https://developer.android.com/studio/command-line/adb#Enabling
E em alguns aparelhos também:
https://github.com/Genymobile/scrcpy/issues/70#issuecomment-373286323

Existe também uma GUI que para alguns simplifica algumas operações, faça a instalação a partir da loja de aplicativos:
```
guiscrcpy
```


## USANDO O CELULAR COMO WEBCAM
Instruções: https://www.dev47apps.com/droidcam/linux/

No celular Android instale o app DroidCam.
No terminal, execute:
```
sudo dnf install -y adb android-tools
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
gnome-text-editor ~/.local/share/applications/droidcam.desktop
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

## OBS STUDIO
Este é o melhor programa de studio e studio de streaming. É incrível acreditar que é opensource e extremamente profissional. Para instalar basta ir na loja e escolher OBS Studio.


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

## INSYNC
Este é o melhor programa cliente de Google Drive, ele simula uma unidade de drive local e comumente é usado para colocar backups no Google Drive sem a necessidade do browser. Para instalar é simples e complicado, simples porque você só tem que dar dois cliques no arquivo e complicado porque se trata dum software proprietário que por não poder ser auditado você terá de confiar no fornecedor. Se você deseja prosseguir assim mesmo então visite a página:
https://www.insynchq.com/downloads

E baixe o instalador e depois dê dois cliques sobre o arquivo baixado e o processo de instalação se iniciará. Se você deseja instalar a partir do repositório, no link acima eles fornecem instruções para serem executados no terminal e você terá atualizações do insink como qualquer outro programa advindo dos repositórios oficiais, neste caso, oficiais do publicador do insink.



## BLANKET
Programa para exibir sons no ambiente de fundo, geralmente usado para focar no trabalho, com sons ambiente da natureza ou urbanos como de uma cafeteria. Para instalar basta ir na loja e escolhê-lo pelo nome Blanket.


## HOMESERVER
Este programa serve para compartilhar uma ou varias pastas de uma forma simples, voce inicia o programa, indica as pastas a serem compartilhadas e momentaneamente eles estarão disponíveis para os computadores na mesma rede local através do navegador. Para instalar basta ir na loja e escolhê-lo pelo nome homeserver.

Quando os clientes copiarem o que queriam, você fecha o aplicativo e o compartilhamento estará encerrado ou pode encerrar pasta a pasta.
Observação: Geralmente se você habilitou o compartilhamento de arquivos, talvez não precise do HOMESERVER.


## TIMESHIFT
Este programa serve para backups, especialmente backups incrementais. Para instalar:
```
sudo dnf install -y  timeshift
```
É melhor procurar no youtube em como utilizá-lo:
https://www.youtube.com/watch?v=tQY5IHOnK9E
Dá para recuperar tanto arquivos quanto o sistema operacional.



## HANDBRAKE
HandBrake é um dos mais poderosos conversores de vídeo. Para instalar basta ir na loja e escolhê-lo pelo nome handbrake.


## FormatLab
FormatLab é um dos mais promissores conversores de vídeo após o HandBrake. Ele faz as mesmas atividades do handbrake, porém é mais simples de operar. Para instalar basta ir na loja e escolhê-lo pelo nome FormatLab.



## GPARTED
GParted é um particionador gráfico para Linux, com ele podemos criar e manipular partições de discos que tenham os mais diversos sistemas operacionais. Muito intuitivo e fácil, torna operações complexas bem mais simples e por isso é importante ter muita cautela. Ele tem um método onde você planeja o que vai fazer, varias tarefas seguidamente mas só o aplica quando você confirmar. Isso é importante porque antes de você executar o procedimento você poderá cancelar a operação, pode parecer simples, mas a maioria dos particionados fazem apenas um passo de cada vez e não tem volta, então o gparted é muito eficiente e fácil. Para instalar basta ir na loja e escolhê-lo pelo nome gparted.


## BLENDER
Blender, também conhecido como blender3d, é um programa de código aberto, desenvolvido pela Blender Foundation, para modelagem, animação, texturização, composição, renderização, e edição de vídeo.  Para instalar basta ir na loja e escolhê-lo pelo nome blender. Se você não cria animações, ignore a instalação desse programa.

## VidCutter
(analogo ao vidcoder para windows)

http://bluegriffon.org/


## INKSCAPE
Para instalar basta ir na loja e escolhê-lo pelo nome INKSCAPE.


## OUTROS TÓPICOS INTERESSANTES
* Virtualização
* Instalando o banco de dados FirebirdSQL
* Ambiente de programação FreePascal/Lazarus
* Ambiente de programação JAVA
* Ambiente de programação Python
* Versionamento com o asdf

