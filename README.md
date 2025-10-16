# INSTALAÇÃO DO DEBIAN E PREPARAÇÃO DO AMBIENTE
Este guia documenta o pós-instalação do Debian com foco em desenvolvedores, administradores de sistemas e entusiastas Linux. Por que Debian? Porque ele é base de várias distros populares (Ubuntu, Linux Mint, Zorin OS, etc.), então as instruções tendem a funcionar nelas com pouca ou nenhuma adaptação. O objetivo não é explorar cada detalhe da instalação, e sim padronizar o que fazer depois para ter um ambiente estável e produtivo.  
Escopo:  
* Foco: configuração do sistema após a instalação (pacotes, serviços, rede, segurança, dev tools).  
* Público-alvo: quem precisa de um passo a passo repetível em máquinas novas ou reinstaladas.  
* Portabilidade: comandos priorizam Debian; quando houver diferença para Ubuntu/Fedora, aponto variantes.  

### Sobre o particionamento (Btrfs vs ext4)
Se o seu foco for virtualização e você pretende usar snapshots (recurso em que o Btrfs brilha), o Btrfs pode ser excelente — mas há nuances para VMs (desempenho, CoW, layout de subvolumes) que exigem atenção.
Se você não precisa de snapshots ou prefere o caminho mais simples, ext4 é uma escolha direta e estável. No tópico específico de Btrfs explico quando e por que usá-lo (e como ajustar para VMs).
Se possivel, todas as partições que contêm dados importantes devem ter um *label* como #disco1, #disco2, #home e assim por diante, sempre sendo fáceis de serem identificados quando executarmos o comando **lsblk -f**. Colocar labels em disco é vida!  

### Como usar este guia
Siga até o fim, mas pule seções que não se aplicam ao seu cenário.  
Ao concluir, você terá um ambiente coerente, reduzindo a necessidade de “formatar/reinstalar” como no Windows — a ideia é evoluir o sistema, não recomeçar do zero.

### Resultado esperado
Um sistema previsível e repetível, com configurações documentadas, pronto para trabalho diário, testes e virtualização.

### Os padrões usados neste HowTo
Para o correto entendimento deste HowTo, usarei alguns padrões:  
**Nome do host**: ti-01  
**Nome do usuário**: gsantana  
**Nome do dominio local**: localdomain.lan
**Debian-Like**: É o termo que uso para distro Linux baseadas em Debian que pode se referir a Ubuntu, Linux Mint, Zorin OS,...

As vezes, comandos que precisam ser executados no terminal são mesclados com o texto da saída do comando, quando isso acontecer, para que você diferencie, qual que é o comando e qual é a saída de texto dele, os comandos serão precedidos de "$", por exemplo:  

|Os comandos que serão seguidos por textos serão precedidos de "$", exemplo:"|  
|:--|
|$ sudo apt update -y|
|Atingido:1 http://security.debian.org/debian-security trixie-security InRelease|
|Atingido:2 http://deb.debian.org/debian trixie InRelease|
|Atingido:3 http://deb.debian.org/debian trixie-updates InRelease|
|Atingido:4 https://dl.google.com/linux/chrome/deb stable InRelease|
|Todos os pacotes estão atualizados.|  

No exemplo acima, apenas a linha #1 é um comando e o restante são textos resultantes do comando.  

### NOTEBOOKs DA LINHA ACER NITRO
Se tiver um ACER NITRO ou outro computador similar com “Secure Boot”, siga essas instruções:  

[Se tiver um ACER NITRO, siga as instruções em:  ](https://github.com/gladiston/fedorainstallonacernitro)


## INSTALAÇÃO
Não há nada muito especial na instalação, o ponto mais critico é mesmo o particionamento. Esta é uma recomendaçao baseada na minha experiência:   

|sistema|Ponto de montagem|rotulo |Tamanho   |   
|:-----:|:----------------|:------|:--------:|
|fat32  |/boot/efi        |Nenhum |1GB       |
|swap   |Nenhum           |Nenhum |mem.atual |
|ext4   |/boot            |#boot  |1GB       |
|ext4   |/                |#root  |100GB     |
|ext4   |/home            |#disco1|max       |  

Caso prefira usar Btrfs, "/" e "/home" serão uma única partição geral como "/" formado com o tamanho restante que sobrou do particionamento do disco. Subvolumes para "/" e "/home" são recomendados, mas o instalador do Debian não faz isso. Este sistema de arquivos é uma mão na roda para programadores porque usando snaphots é facil recupera qualquer arquivo apagado ou sobreescrito sem recorrer a backups, além disso a compactação é muito eficiente. 
>**ALERTA:** Partições Btrfs não podem ter mais de 80% ocupados senão a performance cai por causa do Copy-on-Write(CoW).   

Caso pretenda usar ext4, se possível, mantenha / e /home em partições separadas. Os peritos em virtualização também recomendam /var em separado, no entanto, neste HowTo, as VMs estarão no $HOME.  


**Swap**: Memória SWAP é uma memória de fuga, é para onde os programas correm quando ficam sem memória, afinal se a memória esgotar tanto o programa como o sistema inteiro pode parar de funcionar. Toda vez que programas usam o SWAP, espera-se que seja apenas um pico e que logo volte a não precisar mais dela. Se o SWAP estiver sempre em uso, recomenda-se que compre mais memória, afinal levar uma "vida _loca_" é desperdicio de tempo e compromete os resultados.  
O tamanho de swap para uma partição Linux não pode ser inferior a memória atual de seu equipamento senão ele não será capaz de hibernar. Mas quanto? Digamos que seu equipamento tenha 16GB de RAM, a pergunta que faço é: "De quanta memória seu computador precisa para uma rota de fuga decente?" e se a resposta for "16GB tá bom" então o ideal de swap é 16GB mesmo, mas se a resposta for 32GB de RAM no geral então voce precisa acrescentar mais 16GB de swap totalizando 32GB de SWAP. Mas, e se o seu computador não irá hibernar? Então a regra de ter o minimo do tamanho da RAM não se aplica mais, apenas o quanto de memória seu computador precisaria ter de RAM como plano de fuga.  


## SUDO
Por padrão, o usuario comum não é do mesmo grupo do "sudo", isso impossibilitará usar o comando "sudo", vamos resolver essa situação, execute:

```
su root  
(precisará digitar a senha)  
sudo usermod -aG sudo gsantana  
exit  
```  
Se estiver usando uma maquina de desenvolvimento e deseja relaxar um pouco o 'sudo' então poderá inibir ele de ficar perguntando a senha a todo instante, vá no terminal e execute:  
```  
$ su root  
(precisará digitar a senha)  
$ sudo visudo    
```
e então procure por:    
```
root    ALL=(ALL:ALL) ALL  
```  
Comente essa linha acrescentando "#" no inicio dela e escolha uma dessas opções para colocar na linha abaixo dela:  
```
gsantana    ALL=(ALL) NOPASSWD: /bin/mount, /bin/umount, /bin/mkdir, /bin/rm, /bin/cp, /bin/chmod, /bin/chown, /bin/touch, /bin/apt, /sbin/reboot, /sbin/poweroff
gsantana    ALL=(ALL) NOPASSWD: ALL # precisa de senha para todos, exceto a lista acima  
```
As linhas acima, liberam o sudo sem senha apenas alguns comandos, os demais precisarão da digitação da senha.
A outra opção, abaixo, libera qualquer comando sem o uso da senha:
```
gsantana   ALL=(ALL:ALL) NOPASSWD: ALL # libera qualquer comando sem usar senha  
```
Salve o arquivo e saida do editor.  

> **IMPORTANTE**: A linha acima pode ser um risco ou não a depender do contexto em que você estiver inserido, se achar apropriado fazer isso na sua estação de trabalho então poderá fazê-lo, mas tenha certeza de que seu computador é um **zé-roela** que não oferece nenhum risco, isto é, não tem chaves de segurança que possam ser roubadas ou arquivos valiosos.  


## BLOQUEIO DE TELA AUTOMÁTICO
O sistema normalmente é ajustado automaticamente para bloquear após 5 minutos de atividade, mas ‘falta de atividade’ é um termo incorreto, o correto seria ‘tempo sem interatividade’, isto é, o tempo que você fica sem ter que interagir com o computador. Às vezes estamos processando algo demorado e temos de esperar ou acompanhar a movimentação de log de status e o computador durante este tempo estará tendo muito trabalho, porém com pouca interatividade, a tela será bloqueada. Então precisamos saber quanto tempo precisamos nas tarefas do dia a dia ou então desligá-la.

### GNOME
Vá em configurações->Privacidade->Tela de bloqueio:
![Opção de privacidade e tela de bloqueio](https://github.com/gladiston/fedoralinux/blob/main/gnome_tela_bloqueio01.png)

E então ajuste o tempo:
![Opção de privacidade e tela de bloqueio](https://github.com/gladiston/fedoralinux/blob/main/gnome_tela_bloqueio02.png)

### KDE
![tela de bloqueio](kde_tela_bloqueio01.png)

Não importa se é GNOME ou KDE, voce também tem a possibilidade de desligá-lo. 

## INSTALANDO O GOOGLE CHROME
O Debian acompanha o navegador de Internet Konqueror, ou dependendo do perfil escolhido de isntalação ou distro baseada em Debian, o Firefox.  
No entanto, o Google Chrome é muito popular e deveras alguns sites só funcionam bem com o motor dele.   
[Acesse a página de download dos pacotes do navegador Chrome](https://www.google.com/chrome/?platform=linux) e clique em Fazer o download do Google Chrome. Irá aparecer várias versões para Linux, escolha o pacote .deb de 64 bits para as plataformas Debian e Ubuntu, que serve na realidade para todas as distros derivadas do Debian.  
Após o Download, dê duplo clique nele e o sistema irá dar inicio a instalação e daí, apenas siga as instruções em tela.  


## ADICIONANDO OS REPOSITORIOS 'CONTRIB' e 'NON-FREE'

Primeiro, vamos fazer um backup do arquivo original sources.list:
```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.ori
```
Depois edite o arquivo sources.list:
```
sudo nano /etc/apt/sources.list
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


## INCLUINDO O REPOSITÓRIO DO VSCODE
O Visual Studio Code (VS Code) é uma IDE leve, poderosa e multiplataforma desenvolvida pela Microsoft.  
Não vamos instalá-lo agora, vamos apenas incluir seu repositório, execute os procedimentos abaixo:  

Atualiza a lista de pacotes:
```
sudo apt update
```

Adiciona a chave pública da Microsoft:
```
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | \
  sudo tee /usr/share/keyrings/microsoft.gpg > /dev/null
```
Adiciona o repositório do VS Code:  
```
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | \
  sudo tee /etc/apt/sources.list.d/vscode.list
```

Atualiza os repositórios e instala o VS Code:  
```
sudo apt update
```  


## INCLUINDO O REPOSITÓRIO DA MICROSOFT
Sim, a Microsoft tem um repositório para distribuições Debian-Like.
Não vamos instalar nada de lá ainda, vamos apenas incluir seu repositório e por mais paradoxo que seja, há um download e instalação para que tenhamos tal repositório, execute os procedimentos abaixo:  

Baixa o pacote de configuração da Microsoft
```
wget https://packages.microsoft.com/config/debian/13/packages-microsoft-prod.deb -O /tmp/packages-microsoft-prod.deb
```
Instala o pacote, que adiciona automaticamente o repositório oficial:  
```
sudo dpkg -i  /tmp/packages-microsoft-prod.deb
```
Atualiza os repositórios:  
```
sudo apt update
```  
Isso produzirá um arquivo em /etc/apt/sources.list.d/microsoft-prod.list que apontará para o repositório oficial da Microsoft.
Está curioso para saber o que a Microsoft está compartilhando? Então execute:
```  
apt-cache policy | grep packages.microsoft.com
```  
E então, observe o resultado:  
> 500 https://packages.microsoft.com/repos/code stable/main amd64 Packages  
>     origin packages.microsoft.com  
> 500 https://packages.microsoft.com/debian/13/prod trixie/main all Packages  
>     origin packages.microsoft.com  

Isso confirma que o repositório foi reconhecido, agora vamos listar o que tem lá, execute:  
```  
 apt list -a | grep microsoft
```  
E então, observe o resultado:  
>(...)  
>libmono-microsoft-build-engine4.0-cil/stable 6.12.0.199+dfsg-6 all  
>libmono-microsoft-build-framework4.0-cil/stable 6.12.0.199+dfsg-6 all  
>libmono-microsoft-build-tasks-v4.0-4.0-cil/stable 6.12.0.199+dfsg-6 all  
>libmono-microsoft-build-utilities-v4.0-4.0-cil/stable 6.12.0.199+dfsg-6 all  
>libmono-microsoft-build4.0-cil/stable 6.12.0.199+dfsg-6 all  
>libmono-microsoft-csharp4.0-cil/stable 6.12.0.199+dfsg-6 all  
>libmono-microsoft-visualc10.0-cil/stable 6.12.0.199+dfsg-6 all  
>libmono-microsoft-web-infrastructure1.0-cil/stable 6.12.0.199+dfsg-6 all  
>libmono-system-json-microsoft4.0-cil/stable 6.12.0.199+dfsg-6 all  
>lomiri-online-accounts-plugin-microsoft/stable 0.20-1 all  
>packages-microsoft-prod/trixie,now 1.1-debian13 all [instalado]  
>php-symfony-microsoft-teams-notifier/stable 6.4.21+dfsg-2 all  

É curioso que a atualização do repositório da Microsoft é mantido por um pacote que precisa ser instalado manualmente e depois ele mesmo será atualizado pelo próprio repositório, isso que é uma implementação diferenciada. O time da Microsoft não conheçe a oração dos programadores em C/C++ 'salve-nos da recursividade; main()'. hahahhahahah.


## ATUALIZAÇÃO DE REPOSITÓRIO
Vamos atualizar o repositório de programas:  
```  
sudo apt -y update
```  

Agora, vamos atualizar o sistema:  
```  
sudo apt -y upgrade
```
E então observe o resultado:  

>Atualizando:  
>  google-chrome-stable  
>  
>Resumo:  
>  Atualizando: 1, Instalando: 0, Removendo: 0, Não atualizando: 0   
>  Tamanho de download: 121 MB  
>  Espaço necessário: 7.168 B / 997 GB disponível    
>    
>Obter:1 https://dl.google.com/linux/chrome/deb stable/main amd64 google-chrome-stable amd64 141.0.7390.65-1 [121 MB]  
>Obtidos 121 MB em 5s (23,1 MB/s)  
>apt-listchanges: Lendo logs de mudanças...  
>(Lendo banco de dados ... 202444 arquivos e diretórios atualmente instalados).  
>Preparando para desempacotar .../google-chrome-stable_141.0.7390.65-1_amd64.deb ...  
>Desempacotando google-chrome-stable (141.0.7390.65-1) sobre (141.0.7390.54-1) ...  
>Configurando google-chrome-stable (141.0.7390.65-1) ...  
>Processando gatilhos para mailcap (3.74) ...  
>Processando gatilhos para man-db (2.13.1-1) ...  

No exemplo acima, apenas o google-chrome requer atualização.


## INSTALANDO CODECS
Agora que habilitamos repositórios considerados 'non-free' e 'contrib' poderemos instalar alguns pacotes importantes que liberarão codecs e players de vídeo/musica em nosso sistema:
```
sudo apt install -y libavcodec-extra ffmpeg vlc
```

## INSTALANDO O HTOP, LMSENSORS e STRACE
Os comandos htop, lm-sensors e strace não vêm instalados por padrão, mas são muito úteis para gerenciar e diagnosticar o sistema diretamente pelo terminal. Eles servem para:  
* htop: gerencia tarefas no terminal, semelhante ao top, porém com interface mais amigável e interativa.
* lm-sensors: lê e exibe dados de sensores de temperatura, voltagem e ventoinhas disponíveis no hardware.
* strace: monitora chamadas de sistema e sinais usados por um processo — útil, por exemplo, para descobrir qual programa está acessando ou bloqueando um arquivo como arquivo.docx.

Gostou deles? Então execute:  
```  
sudo apt install -y htop lm-sensors  strace
```

## NOTIFY-SEND
O notify-send é um utilitario geralmente usado para enviar mensagens de um usuário para outro no mesmo sistema, ele é analogo ao comando 'wall' que serve para uso em terminal. Muitos scripts usam ele para entregar notificações em interfaces gráficas, então vamos instalá-lo:
```
sudo apt install -y libnotify-bin
```


## INSTALANDO O SILVERSEARCH-AG(ag)
O Silversearcher-ag, também conhecido apenas como ag, é uma ferramenta de busca extremamente rápida para código-fonte e arquivos de texto.
Ele é similar ao comando grep, porém muito mais veloz e prático, sendo ideal para desenvolvedores e administradores que precisam localizar trechos de texto em grandes projetos.
```
sudo apt install -y silversearcher-ag
```


## INSTALANDO ADICIONAIS PARA O APT
O programa 'apt' está instalado, mas para algumas operações ele precisa de alguns extras, eles são obrigatórios em minha opinão:  
```
sudo apt install -y apt-transport-https gpg
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
São instalados poucos formatos, por isso, é recomendável instalar os pacotes abaixo para garantir suporte aos formatos mais comuns e também outros que embora pouco usados por usuários comuns, desenvolvedores costumam usar, por exemplo, o formato RAR.
```
sudo apt install -y tar zip unzip p7zip-full p7zip-rar rar unrar lzip lzma xz-utils bzip2 gzip squashfs-tools cabextract
```

Pacote|Função / Formato
|:--|:--|
tar|Criação e extração de arquivos .tar e .tar.gz
zip, unzip|Manipulação de arquivos .zip
p7zip-full|Suporte a arquivos .7z (formato 7-Zip)
p7zip-rar, rar, unrar|Suporte a arquivos .rar
lzip, lzma, xz-utils, bzip2, gzip|Compactações livres amplamente usadas em pacotes Linux
squashfs-tools|Criação e extração de arquivos .squashfs


## INSTALANDO O GERENCIADOR DE FONTES
Normalmente, eles já vem instalados em algumas distros, mas no Debian, geralmente não. Vamos executar:  
```
sudo apt install -y  fontconfig fontforge fonttools
```


## INSTALANDO PROGRAMAS BASICOS PARA COMPILAÇÃO DE FONTES
Os pacotes a seguir servem para quem pretende compilar algo no ambiente Linux. Se você pretende instalar o driver proprietário da NVIDIA fornecido pela NVIDIA você também precisará deles:
```
sudo apt install -y build-essential
sudo apt install -y dh-make exuberant-ctags dpkg-dev debhelper fakeroot
sudo apt install -y exuberant-ctags module-assistant dkms patch libssl-dev
sudo apt install -y libncurses-dev ack fontconfig imagemagick git meson sassc 
```

## ATIVE O SUPORTE A FLATPAK CENTRAL
O flatpak não está instalado ou habilitado em nosso sistema, para hablitá-lo, basta rodar o seguinte comando no terminal:
```  
sudo apt install -y flatpak
```  
Para o ambiente GNOME, instale também:
```  
sudo apt install gnome-software-plugin-flatpak
```  
Para o ambiente KDE, instale também:
```  
sudo apt install plasma-discover-backend-flatpak
```
Os pacotes alternativos para GNOME ou KDE são para maior integração dessas DE's ao flathub.

Caso as instrções acima não funcionem, visite a página com informações atualizadas:   
[https://flatpak.org/setup/Debian](https://flatpak.org/setup/Debian)  
O link acima também é possivel ter outros repositórios mais específicos como o Ubuntu.  


Depois disso, adicionamos enfim, o repositório:
```  
sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```
Provavelmente aparecerá a seguinte mensagem de texto:  
>Note que os diretórios   
>  
>'/var/lib/flatpak/exports/share'  
>'/home/gsantana/.local/share/flatpak/exports/share'  
>  
> não estão no caminho de pesquisa definido pela variável de ambiente XDG_DATA_DIRS, portanto, os aplicativos instalados pelo Flatpak podem não aparecer em sua área de trabalho até que a sessão seja reiniciada.  

Essa advertência nos instrui a incluir as pastas citadas na variavel de ambiente XDG_DATA_DIRS. E se executar o comando:  
```
echo $XDG_DATA_DIRS
```
Não aparecerá saída de texto nenhuma, e realmente parece que tem uma falha a ser corrigida, mas calma lá, acontece que os caminhos indicados só serão vistos depois que você *reiniciar sua sessão* e refazer o login, então faça isso.  

Depois de logar-se, vamos testar novamente:
```
echo $XDG_DATA_DIRS
```
Agora enxergaremos:
```
/home/gsantana/.local/share/flatpak/exports/share:/var/lib/flatpak/exports/share:/usr/local/share:/usr/share
```

### IMPORTANTE
Não é uma boa ideia instalar programas do flathub que são fornecidos pelos desenvolvedores originais em local diferente, por exemplo:  
* Google Chrome, o próprio Google os fornece diretamente em seu site oficial  
Isto acontece porque geralmente os pacotes fornecidos pelo flathub são feitos pela comunidade cujo resultado final pode diferir do autor original, por exemplo, com alguma mistura de plugins adicionais ou modificações. Essas adaptações carecem de verificação pela comunidade e até que isso aconteça, pode ser inseguro.   

Mas também há desenvolvedores que publicam seu próprio programa no flathub,e neste caso, são confiáveis, por exemplo:    
* Mozilla Firefox, a própria Mozilla publica seu software no flathub  
* Telegram, a própria Telegram publica seu software no flathub  
 
Qual a diferença de um pacote tradicional para um pacote advindo do flathub? Os pacotes tradicionais são inspecionados pela própria distribuição e instalados de maneira tradicional em seus respectivos diretorios. Já os pacotes adquiridos do flathub são produzidos pela própria comunidade e considerados mais seguros porque rodam sob container, isto é, estão limitados a pastas como: 
```
~/.var/app
```
Alguns programas adquiridos do flathub pedem autorização para acessar seu $HOME ou algo restrito do seu sistema, quando isso acontece, geralmente eles perguntam para o usuário, que pode conceder o acesso ou não. Mas independentemente de rodarem sob container, dependendo do programa escolhido, o uso de containeres não é nenhuma vantagem, por exemplo, um navegador baixado do flathub teria acesso ao seu histórico, senhas, etc... então tudo precisa de moderação da sua parte e em descobrir o que é viável instalar do flathub e o que não é, em especial, duvide de aplicativos onde o desenvolvedor e o publicador não são a mesma entidade.  



## INSTALANDO O VSCODE
O Visual Studio Code (VS Code) é uma IDE leve, poderosa e multiplataforma desenvolvida pela Microsoft.  
Ele combina a simplicidade de um editor de texto com recursos avançados de programação, como autocompletar inteligente (IntelliSense), debug integrado, controle de versão com Git, e uma ampla variedade de extensões para praticamente qualquer linguagem. O VS Code não está nos repositórios padrão de nenhuma distro e por isso incluímos ele nos passos anteriores, dessa forma ficará fácil instalar e receber atualizações, execute:  
```
sudo apt install -y code
```  

**EXTENSÕES SUGERIDAS:**
**NODE.JS**  
É preciso ter a linguagem previamente instalada para prosseguir com as instruções abaixo:  
```
code --install-extension waderyan.nodejs-extension-pack \
     --install-extension dbaeumer.vscode-eslint \
     --install-extension christian-kohler.npm-intellisense \
     --install-extension christian-kohler.path-intellisense \
     --install-extension ms-vscode.node-debug2
```
**JAVA**  
É preciso ter a linguagem previamente instalada para prosseguir com as instruções abaixo:  
```
code --install-extension vscjava.vscode-java-pack \
     --install-extension redhat.java \
     --install-extension vscjava.vscode-java-debug \
     --install-extension vscjava.vscode-java-test \
     --install-extension vscjava.vscode-maven
```
**FREE PASCAL E DELPHI**  
É preciso ter a linguagem previamente instalada para prosseguir com as instruções abaixo, isso também inclui o Lazarus, IDE para programação usando freepascal:  
```
sudo apt install -y global exuberant-ctags python3-pygments
```
Depois:  
```
$ code --install-extension Wosi.omnipascal \
       --install-extension alefragnani.pascal \
       --install-extension alefragnani.pascal-formatter
```
> 💡 Dica: no Debian, é mandatório instalar o FreePascal Compiler (fpc) usufruir dessas extensões. Cada uma dessas extensões carecem de configuração, vejá-os:  
> [Instruções para Omini Pascal](https://www.omnipascal.com)     
> [Instruções para a Linguagem Pascal](https://github.com/alefragnani/vscode-language-pascal)     
> [Instruções para Pascal Formatter](https://github.com/alefragnani/vscode-pascal-formatter)     

Caso queira uma outra IDE (e mais completa) para FreePascal, recomendo o [Lazarus](https://lazarus-ide.org).  


**EXTENSÕES PARA HTML, CSS E JAVASCRIPT**  
```
code --install-extension ecmel.vscode-html-css \
     --install-extension esbenp.prettier-vscode \
     --install-extension ritwickdey.LiveServer \
     --install-extension formulahendry.auto-rename-tag \
     --install-extension xabikos.JavaScriptSnippets
```
**EXTENSÕES PARA PYTHON**   
É preciso ter a linguagem previamente instalada para prosseguir com as instruções abaixo:  
```
code --install-extension ms-python.python \
     --install-extension ms-python.vscode-pylance \
     --install-extension ms-toolsai.jupyter \
     --install-extension ms-python.debugpy
```
**EXTENSÕES PARA SQL E GERENCIAMENTO DE BANCOS DE DADOS**  
```
code --install-extension mtxr.sqltools \
     --install-extension mtxr.sqltools-driver-mysql \
     --install-extension mtxr.sqltools-driver-pg \
     --install-extension mtxr.sqltools-driver-sqlite \
     --install-extension mtxr.sqltools-driver-firebird \
     --install-extension adpyke.vscode-sql-formatter \
     --install-extension cweijan.vscode-database-client2
```
>**COMO USAR**:   
>Após a instalação, abra o Painel SQLTools (Ctrl+Shift+P → “SQLTools: Show Connections”).   
>Clique em + New Connection e configure o banco desejado (MySQL, PostgreSQL, Firebird etc.).    
>Execute consultas com Ctrl+Alt+E ou usando o menu de contexto “Run Query”.     
>Para múltiplos bancos, o Database Client (de Cweijan) exibe uma interface visual de fácil navegação, inclusive com editor gráfico de tabelas.     
>**Dica**: Se for usar o Firebird, certifique-se de que o cliente isql e o driver libfbclient.so estão instalados no sistema.    

**EXTENSÕES PARA BASH SCRIPT E TERMINAL**  
```
code --install-extension mads-hartmann.bash-ide-vscode \
     --install-extension timonwong.shellcheck \
     --install-extension foxundermoon.shell-format \
     --install-extension formulahendry.code-runner \
     --install-extension jeff-hykin.better-shellscript-syntax \
     --install-extension formulahendry.terminal
```
**CONFIGURAÇÕES RECOMENDADAS**:  
Após instalar as extensões, adicione estas configurações no arquivo ~/.config/Code/User/settings.json (ou use Ctrl + , → Abrir Configurações JSON):
```
{
  "editor.formatOnSave": true,
  "[shellscript]": {
    "editor.defaultFormatter": "foxundermoon.shell-format"
  },
  "code-runner.executorMap": {
    "bash": "bash"
  },
  "shellformat.flag": "-i 2"
}
```
Essas opções ativam:
* Formatação automática ao salvar
* Execução direta de scripts (Ctrl+Alt+N)
* Indentação de 2 espaços padrão


## OBTENHA O KDE COMPLETO (OPCIONAL)  
O KDE que acompanha o Debian é uma versão leve e personalizada para o Debian(wallpapers, logos, etc...), sem todos os módulos e personalizações idealizados pelo time do KDE, se desejar a versão idealizada pelo time do KDE, execute:
```  
sudo apt install -y kde-full
```
Depois disso, *recomendo que reinicie o computador*.


## PRELOAD
Se estiver usando discos mecanicos, provavelmente sente muita latencia para carregar certos progrmas. Numa situação assim, é bom instalar um serviço chamado 'preload', ele monitora os programas que você mais utiliza e durante o boot já os carrega para você. A vantagem é a velocidadade para carregá-los da primeira vez, no entanto, tem o lado negativo, tais programas SEMPRE ESTARÃO NA MEMÓRIA logo após o boot e com isso, o tamanho da sua memória principal após o boot será menor porque esse grupo de programas já estarão na memória. Então a minha recomendação do uso do préload é de apenas usar se (1) usa discos mecânicos e (2) e se tem um fluxo de trabalho consistente e repetitivo com o mesmo grupo de programas. Se depois de avaliar, decidir que o 'preload' é para você, então execute::
```
sudo apt install -y preload
```
Quando instalar, ele se ativará sozinho como serviço e nada mais precisa ser feito, mesmo assim é bom conferir:
```
sudo systemctl status preload
```
Depois disso, o serviço estiver desativado, então:
```
sudo systemctl start preload
```
E para iniciar o serviço durante o boot, execute:
```
sudo systemctl enable preload
```
Se achar que não houve vantagens, poderá desinstalá-lo pela interface KDE ou GNOME.

## INSTALANDO PERFIS DE USO (TUNED)
O tunned é um programa que permite trocar em tempo real o perfil de desempenho do compuador, por exemplo, posso usar o perfil de desempenho balanceado quando quero navegar na internet e de um momento para outro trocar o perfil de desempenho para 'realtime' quando quero maximimizar a performance. Há outros perfis prontos para usar maquinas virtuais, economia de energia, etc... O programa tem muitos perfís e é altamente recomendado, vamos a instalação:
```
sudo apt install -y tuned
```
Liste os perfís de otimização existentes:
```
sudo tuned-adm list
```
Observe as opções listadas:  
```
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
sudo tuned-adm active
```
Observe o resultado do comando:  
> Current active profile: balanced  

O modo 'balanceado' é o modo não especializado, se desejar especializar seu computador em algo, por exemplo, para usar o desktop, execute:  
```
sudo tuned-adm profile desktop  
```  
E então o sistema será especializado ou otimizado para ser usado como 'desktop'. É importante saber que carregar o perfil errado, torna tudo errado, por exemplo, se executar:
```
sudo tuned-adm profile virtual-guest  
```
Ou seja, especializando-se em uso de VMs, mas não usufruindo de VMs, seu ambiente de destkop terá uma performance rebaixada, em poucas palavras, o que quer irá usar no KDE, GNOME,...terá performance menor do que deveria ter. Então se pretende usar este programa, deverá entrar no seu fluxo de trabalho ficar trocando de perfil quando troca de um momento para outro. Mais tarde, veja se seu ambiente gráfico KDE ou GNOME possuem plugins para usufruir do 'tuned' e tornar mais fácil essas trocas.

Perfis muito comuns para quem usa laptop:  

**laptop-ac-powersave** - “Optimize for laptop with power savings”  
*Uso*: notebook ligado na tomada (AC).  
*Objetivo*: manter bom desempenho, mas ainda economizar energia onde possível.   
*Ajustes típicos*: Habilita CPU frequency scaling (a CPU reduz clock quando ociosa). Mantém turbo boost ativado para tarefas pesadas. Reduz brilho de tela e consumo de periféricos em idle.   
Mantém discos e interfaces de rede em modo balanceado.     
*Resumo*: bom equilíbrio entre desempenho e economia.  Ideal para uso diário com o notebook conectado à energia.  

**laptop-battery-powersave** - “Optimize laptop profile with more aggressive power saving”
*Uso*: notebook usando bateria.  
*Objetivo*: maximizar autonomia, mesmo sacrificando desempenho.  
*Ajustes típicos*: CPU limitada a clocks mais baixos. Desativa turbo boost e núcleos ociosos. Reduz brilho e tempo de suspensão automática. Interfaces Wi-Fi e Bluetooth podem entrar em modos de economia agressiva. Discos mecânicos são parados rapidamente quando inativos.  
*Resumo*: desempenho menor, mas maior duração de bateria.  
Ideal para uso em viagens, reuniões ou campo.  

**latency-performance** - “Optimize for deterministic performance at the cost of increased power consumption”  
*Uso*: servidores ou estações de trabalho que exigem baixa latência e previsibilidade (mas não tempo real).  
*Objetivo*: garantir resposta consistente, mesmo com maior consumo de energia.  
*Ajustes típicos*: Desativa CPU frequency scaling → clock fixo máximo. Desativa C-states profundos e economias de energia. Ajusta IRQs e afinidade de CPU para reduzir jitter. Mantém memória e dispositivos em estado ativo constante.  
*Resumo*: máxima estabilidade e latência previsível, sem foco em economia.  
Ideal para bancos de dados, servidores de aplicações ou jogos que exigem resposta constante.  

Outros perfis muito uteis para desenvolvedores são:
**realtime** - “Optimize for realtime workloads”  
*Uso*: sistemas físicos (bare metal) com necessidades críticas de tempo real.  
*O que faz*: Reduz a latência ao máximo. Ajusta o escalonador de CPU para favorecer tarefas de tempo real. Desativa power saving features (como C-states e turbo boost). Fixa frequências da CPU em nível máximo. Ajusta IRQs e prioridade de processos.  
Exemplo de uso: estações de áudio profissional (JACK), robótica, processamento de sinais, sistemas de controle industrial.  

**realtime-virtual-guest** - “Optimize for realtime workloads running within a KVM guest”  
*Uso*: máquinas virtuais (guests) executando workloads de tempo real dentro de um host KVM.  
*O que faz*: Aplica otimizações similares ao perfil realtime, mas levando em conta que o controle de hardware é mediado pelo hipervisor. Ajusta parâmetros de temporização (clocksource, tickless, etc.) para sincronizar com o host. Minimiza interferência do kernel convidado.  
*Exemplo*: uma VM rodando um sistema de controle de robô industrial, ou processamento de áudio, dentro de um servidor KVM.  

**realtime-virtual-host** - “Optimize for KVM guests running realtime workloads”
*Uso*: no host KVM que executa VMs que, por sua vez, têm workloads de tempo real.
*O que faz*: Garante que as VMs de tempo real recebam CPU e I/O com mínima latência. Usa CPU pinning e isolcpus para isolar núcleos destinados às VMs RT. Minimiza a interferência do host em threads de tempo real. 
*Exemplo*: servidor KVM que hospeda várias VMs RT, como sistemas de automação ou simulações científicas críticas.  


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

## INSTALANDO A FONTE "CONSOLAS"
A fonte “consolas” é uma interessante fonte para ser usada tanto em desenvolvimento de aplicativos como também no ambiente de terminal. Ela é de propriedade de terceiros e por isso não vem acompanhada dentro das distribuições Linux, mas é possível instalá-las. Para instalar siga as instruções:
```
cd /tmp
mkdir -p ~/.local/share/fonts
wget -O /tmp/YaHei.Consolas.1.12.zip https://storage.googleapis.com/google-code-archive-downloads/v2/code.google.com/uigroupcode/YaHei.Consolas.1.12.zip
unzip YaHei.Consolas.1.12.zip -d ~/.local/share/fonts/ 
```
Para conferir se foi realmente instalada, execute agora:  
```
fc-list | grep "Consolas"
```
Se aparacer algo como abaixo, então foi um sucesso:  
```
/home/gsantana/.local/share/fonts/YaHei.Consolas.1.12.ttf: YaHei Consolas Hybrid:style=YaHei Consolas Hybrid Regular,Normal,obyčejné,Standard,Κανονικά,Normaali,Normál,Normale,Standaard,Normalny,Обычный,Normálne,Navadno,Arrunta
```


## INSTALAÇÃO DE FONTES MICROSOFT
Vamos adicionar um repositório que nos será util para acrescentar mais fontes ao sistema:
```
sudo apt install -y ttf-mscorefonts-installer
```
O pacote instalado acima complementará as fontes Microsoft de que alguns programas portados do Windows talvez precisem.

## INSTALAÇÃO DA FONTE ROBOTO
A fonte ‘fonts-roboto’ é bastante interessante para uso em terminal e IDEs de programação:
```
sudo apt install -y fonts-roboto fonts-roboto-fontface  fonts-roboto-slab
```
Para conferir se a fonte foi realmente instalada, executamos:
```
fc-list | grep "roboto"
```

## INSTALAÇÃO DA FONTE HACK
A fonte Hack é bastante apropriada para ser usada para listar codigo fonte de programas ou utilizar o terminal, sua instalação pelo repositório é simples:
```
sudo apt install -y fonts-hack-otf fonts-hack-ttf fonts-hack-web
```

Para conferir se a fonte foi realmente instalada, executamos:
```
fc-list | grep "Hack"
```
Se aparecer o nome da fonte em ~/.local/share/fonts/ttf/Hack-BoldItalic.ttf: Hack:style=Bold Italic e assim por diante é porque a fonte foi instalada com sucesso.

## GIT
Vamos ajustar nosso ambiente com o GIT com os comandos:
```
git config --global user.name "Seu nome completo"
git config --global user.email "seu.email@dominio.com"
```

Recentemente, o github fez alterações em seu sistema onde a instrução:
```
git config --global credential.helper 'cache --timeout=28800'
```

Será ignorada completamente ou terminará em erro e a tentativa de login resultará neste erro:
**Fatal Authentication Failed for: site.com.br**

Para solucionar o problema, precisará de mais alguns pacotes:
```
sudo apt install -y libsecret-1-0 libsecret-tools libsecret-1-dev build-essential
```
Agora vamos até o código fonte:  
```
cd /usr/share/doc/git/contrib/credential/libsecret
```
E depois compilar:
```
sudo make
```
Após, o git só precisará dessa configuração adicional:
```
git config --global credential.helper /usr/share/doc/git/contrib/credential/libsecret/git-credential-libsecret
```
Agora, você precisa saber que o método de autenticação mudou, você não usa mais o “username+ senha” do seu usuario git, mas “username+token”. O token é criado na página do github, no menu de sua profile->Settings->Developer settings->Personal access tokens->Tokens(classic) e então criar um token. Este token será o substituto de sua senha git no terminal.


## MUDANDO O NOME DO HOST  
Durante a instalação do Debian, você provavelmente definiu um nome para o seu computador (hostname).
Entretanto, caso queira modificá-lo depois, é possível fazer isso facilmente.  
Pelo ambiente gráfico (KDE ou GNOME), abra o aplicativo Configurações do Sistema, e na barra de pesquisa, digite “host”, "sistema" ou algo similar e essas informações serão exibidas e passiveis de modificações. A cada versão do KDE e GNOME, essas opções mudam de lugar ou são traduzidas de forma diferente o que impede de trazer um screenshot. Mas pelo terminal, é bem mais eficiente, basta executar:  
``` 
sudo hostnamectl set-hostname novo-nome
``` 


## COMPARTILHAMENTO DE ARQUIVOS
Aparentemente, o SAMBA vem pré instalado no Debian, no entanto, foi observado que carece de alguns ajustes.

### Instalando o SAMBA
Execute:
```  
sudo apt install -y plasma-widgets-addons kdenetwork-filesharing samba
sudo apt install -y cifs-utils kio-fuse
```
As vezes, dependendo do perfil de instação, ele pode já ter sido instalado.

### Ajustando workgroup ou dominio
Algo que também é eficiente, caso você tenha um dominio em sua rede é fazer um pequeno ajuste no arquivo de configuração do 'samba', edite o arquivo */etc/samba/smb.conf*:
```  
sudo nano /etc/samba/smb.conf
```  

e vá até a linha:    
```  
WORKGROUP = WORKGROUP
```  
e troque por:  
```  
WORKGROUP = LOCALDOMAIN.LAN
```  
O nome do dominio (LOCALDOMAIN.LAN, mas use o nome de seu dominio local) deve ser digitado em maiuscula por causa do antigo protocolo WINS ainda em uso no Windows, depois disso salve o arquivo e saia do editor. 
Com essa modificação, quando acessar uma pasta compartilhada na rede, o nome 'meudominioderedelocal.lan' já aparecerá como padrão na tela de autenticação de usuário e retardará problemas futuros de lesão por esforços repetitivos em seus dedos.  

no entanto, o serviço 'samba-ad-dc' não deve ser iniciado, pois ele é destinado a servir como controlador de dominio e essa não é nossa intenção, então desabilite tal serviço:

### Desativando o controlador de dominio
Em algumas situações, o controlador de dominio foi instalado e daí o comportamento do SAMBA é totalmente diferente. Ele não deve ser instalado em desktops, caso isso tenha acontecido, execute:  
```  
sudo systemctl disable samba-ad-dc
```
Se o comando acima responder:
```
Failed to disable unit: Unit samba-ad-dc.service does not exist
```
Então parabens! O controlador de dominio não esta instalado e poderá prosseguir, mas se mostrar:  
```
Synchronizing state of samba-ad-dc.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install disable samba-ad-dc
Removed '/etc/systemd/system/multi-user.target.wants/samba-ad-dc.service'.
Removed '/etc/systemd/system/samba.service'.
```
Entao é porque você estava com o controlador de dominio instalado e nem fazia ideia. De qualquer forma, desativamos e poderá prosseguir.

### Ativando o compartilhamento de arquivos
Para usar apenas o compartilhamento de arquivos, iniciaremos apenas estes serviços:  
```  
sudo systemctl enable smbd nmbd
sudo systemctl start smbd nmbd
```


## CRONTAB
O crontab é o agendador de tarefas do Linux. Ele permite que comandos ou scripts sejam executados automaticamente em horários ou intervalos definidos, sem a necessidade de intervenção do usuário.
É extremamente útil para tarefas recorrentes, como backups, limpeza de arquivos temporários, sincronização de dados, atualizações de sistema, entre outras.

O cron daemon (crond) é o serviço que fica em execução em segundo plano e verifica, minuto a minuto, se há alguma tarefa programada a ser executada.
Cada usuário pode ter seu próprio arquivo de crontab, e o sistema também possui um crontab global em /etc/crontab.

Para editar o agendamento do seu usuário, use:
```
crontab -e
```
Vamos a um exemplo mais prático, vamos editar o agendamento do seu sistema e para isso repetimos o mesmo comando, porém usando o 'sudo':
```
sudo crontab -e
```
Mesmo que não use o 'crontab' neste momento, recomendo que cole este cabecalho:  
```
SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
MAILTO=""
# Exemplo de definição de tarefa (job):
# .---------------- minuto (0 - 59)
# |  .------------- hora (0 - 23)
# |  |  .---------- dia do mês (1 - 31)
# |  |  |  .------- mês (1 - 12) OU jan,fev,mar,abr ...
# |  |  |  |  .---- dia da semana (0 - 6) (domingo=0 ou 7) OU dom,seg,ter,qua,qui,sex,sab
# |  |  |  |  |
# *  *  *  *  *  nome-usuario  comando a ser executado
#
# Exemplo: executar um backup todos os dias às 2h30 da manhã
# Também é uma boa prática redirecionar saídas de erro para um log
# 30 2 * * * root /usr/local/bin/backup-diario.sh >> /var/log/backup.log 2>&1
```
Salve e feche o arquivo (Ctrl+O, Enter, Ctrl+X).  
Porque deixar as linhas acima? Para que quando você for executar o 'sudo crontab -e' possa lembrar do formato do agendamento.  
O parametro **SHELL** indica que shell iremos usar, podemos escolher o 'bash' que é mais comum, mas ao usar o 'sh' evitamos a carga do perfil com variaiveis de ambiente, path e outras coisas fazendo com que o agendamento não sofra nenhuma interferencia.  
O parametro **PATH** é porque ao usar o shell 'sh', ele não tem PATH nenhum e boa parte dos comandos não funcionariam.  
Em servidores com algum MTA instalado, incluiriamos também **MAILTO** com algum e-mail destacado e assim qualquer tarefa agendada, seu resultado seria enviado para o e-mail indicado, mas não funciona sem um MTA ou SMTP instalado no sistema.  

Vez ou outra irá surgir a necessidade de listar os agendamentos programados então execute:
```
sudo crontab -l # para listar agendamentos globais ou
crontab -l # para listar seus agendamentos
```
Use agendamentos globais para tarefas que envolvam o sistema, tendo ou não usuários conectados, e que possivelmente use comandos que só o root possa executar, por exemplo, desligar o computador as 02h00 da manhã caso eu tenha-o deixado ligado após o expediente:  
```
# Desligar às 02:00
0 2  * * * root /usr/sbin/shutdown -h now
```
E use agendamentos pessoais que só se aplicam quando você estiver conectado ao computador, por exemplo, deixar um lembre de se levantar a cada 2h para beber água, mas só vale das 10h até as 18h:
```
# Enviar lembrete para beber água das 10h às 18h, a cada 2 horas
#0 10-18/2 * * * wall "Lembrete: Levante-se um pouco e beba água!"  # terminal texto
0 10-18/2 * * * export DISPLAY=:0 && notify-send "Lembrete: Levante-se um pouco e beba água!" # KDE, GNOME, etc...
```
O uso do **crontab** que foi mencionado aqui vale para todas as distribuições Linux, mesmo seu computador sendo um desktop, faça uso dele. O GNOME, KDE e outras DE's(Desktop Enviroment) tem utilitários para agendamento de maneira visual que tornam os agendamentos mais simples, mas lembre-se que nesse HowTO, não vamos entregar essas facilidades porque elas variam de DE para DE e o uso do terminal é a maneira mais consistente de explicar.  


## EDITOR DE TEXTO VIM
O Vim (Vi IMproved) é um editor de texto poderoso e altamente configurável, baseado no clássico Vi, presente em praticamente todas as distribuições Unix e Linux. É amplamente utilizado por administradores de sistema e desenvolvedores por ser leve, rápido e disponível mesmo em ambientes sem interface gráfica.  

Ele oferece atalhos de teclado eficientes, realce de sintaxe, e modos de operação distintos (comando, inserção e visualização), que tornam a edição ágil e precisa. Também pode ser expandido com plugins e temas, transformando-o em um ambiente completo para programação.  

No Debian 13, o Vim não vem instalado por padrão, mas pode ser adicionado facilmente com o apt:  
```  
sudo apt install -y vim
```  

Para confirmar a instalação e ver a versão instalada:
```  
vim --version
```
Algo que pode ser um pouco irritante ao usar o Vim é que o mouse também é controlado por ele. Assim, ao abrir o terminal dentro do KDE ou GNOME, os comandos de copiar/colar do terminal podem não funcionar, pois o Vim captura os cliques do mouse.

Se isso te incomoda, basta executar dentro do Vim o comando (tecle ":"):
```
set mouse=
```  
Para tornar essa configuração permanente, edite (ou crie) o arquivo de configuração do Vim:
```  
nano ~/.vimrc
```  
E adicione a linha:
```  
set mouse=
```  
Salve e feche o arquivo (Ctrl+O, Enter, Ctrl+X).  
Pronto — agora o mouse não interferirá mais ao usar o Vim.

## PERMISSÃO AO JOURNAL
O journal é o mecanismo de logs do systemd. Ele registra praticamente tudo o que ocorre no sistema — mensagens do kernel, inicialização de serviços, eventos de segurança, entre outros.
Antigamente, esses registros eram armazenados em simples arquivos texto (como /var/log/syslog), acessíveis a qualquer usuário. Hoje, o journal é um serviço binário centralizado com restrições de acesso.  
Essa restrição afeta alguns comandos como o 'systemctrl status \[serviço\]', veja este exemplo:  
```  
systemctl status systemd-journald
```
você poderá ver um aviso como:
```  
Warning: some journal files were not opened due to insufficient permissions.
```
Para eliminar esse *warning* e permitir acesso completo aos logs, adicione seu usuário atual ao grupo systemd-journal:
```  
sudo usermod -aG systemd-journal "$USER"
```
Em seguida, atualize sua sessão para que a mudança tenha efeito:  
```  
newgrp systemd-journal  # ou faça logout/login
```
Essa alteração só concede acesso de leitura aos logs do sistema. Ela é segura e recomendada para administradores que precisam analisar mensagens de serviços sem usar sudo o tempo todo.

## FIREWALL 
Um sistema de firewall geralmente não vem instalado por padrão em muitas distribuições voltadas para desktop. Por isso, o primeiro passo é instalá-lo manualmente. Vamos optar pelo Firewalld, pois ele é o padrão no Fedora, RHEL, CentOS e openSUSE, além de ser totalmente compatível com Debian e Ubuntu. Essa escolha garante comandos consistentes e portabilidade entre diferentes ambientes Linux.  
  
Muitas pessoas argumentam que um firewall é desnecessário em estações de trabalho Linux, e até certo ponto isso é verdade para uso doméstico. Contudo, se você é desenvolvedor ou administrador de sistemas, é essencial que o ambiente de desenvolvimento seja o mais parecido possível com o ambiente de produção — e este quase sempre possui um firewall ativo.  
Em resumo: instalar o Firewalld no seu ambiente desktop não é apenas por segurança, mas por coerência e preparo profissional.  

Instale o Firewalld:  
```
sudo apt install -y firewalld
```
Em seguida, habilite e inicie o serviço:
```
sudo systemctl enable firewalld
sudo systemctl start firewalld
```
Verifique as portas atualmente liberadas:  
```  
sudo firewall-cmd --list-ports
```
Provavelmente não mostrará nada, mas vamos liberar algumas portas para servir de exemplo.  


### LIBERANDO TEMPORARIAMENTE PORTAS NO FIREWALL
Para liberar portas, precisamos listar os serviços que iremos usar, alguns podem nem ter sido instalados ainda, veja esta lista que exemplifica os serviços/portas que pretendemos liberar:  
|Serviço      |Porta  |Descrição|
|-------------|:-----:|---------|
|HTTP         |80     |Tráfego web padrão (sites sem HTTPS)|
|HTTPS        |443    |Tráfego web seguro (SSL/TLS)|
|SSH          |22     |Acesso remoto seguro a servidores Linux|
|RDP          |3389   |Acesso remoto do Windows (XRDP também usa)|
|MySQL/MariaDB|3306   |Banco de dados |
|PostgreSQL   |5432   |Banco de dados|
|Firebird     |3050   |Banco de dados|

Com base na lista acima, os comandos seriam:  
```
sudo firewall-cmd --add-port=80/tcp
sudo firewall-cmd --add-port=443/tcp
sudo firewall-cmd --add-port=22/tcp
sudo firewall-cmd --add-port=3389/tcp
sudo firewall-cmd --add-port=3306/tcp
sudo firewall-cmd --add-port=5432/tcp
sudo firewall-cmd --add-port=3050/tcp
```
Agora vamos repetir a verificação das portas atualmente liberadas:  
```  
sudo firewall-cmd --list-ports
```
E observe o resultado:  
> 22/tcp 80/tcp 443/tcp 3050/tcp 3306/tcp 3389/tcp 5432/tcp  
  
Isso significa que obtivemos sucesso, no entanto, essas regras são temporarias até reiniciar o firewalld ou o sistema.  

### LIBERANDO PERMANENTEMENTE PORTAS NO FIREWALL
Como vimos no passo anterior, as regras são temporarias, elas valem apenas até reiniciar o firewalld ou o sistema. Para torná-las permanentes, execute:  
```
$ sudo firewall-cmd --runtime-to-permanent
success
```
Agora, vamos reiniciar o firewall:  
```
sudo firewall-cmd --reload
```
Agora vamos repetir a verificação das portas atualmente liberadas:  
```  
$ sudo firewall-cmd --list-ports
22/tcp 80/tcp 443/tcp 3050/tcp 3306/tcp 3389/tcp 5432/tcp 
```
Como pode observar acima, as regras não sumiram. Então, quando precisar de regras permanentes faça isso.  

### LIBERANDO PERMANENTEMENTE PORTAS NO FIREWALL POR PERFIL
Assim como outros sistemas de firewall, o Firewalld trabalha com o conceito de perfis, chamados de zonas (zones).  
A ideia é simples: cada zona representa um conjunto de regras de segurança.  
  
Por exemplo, você pode ter uma zona para virtualização, outra para desenvolvimento e outra para uso pessoal.  
Quando muda de zona, o Firewalld desativa as regras da anterior e aplica as novas, permitindo perfis de rede específicos para cada tipo de tarefa.  
  
Por padrão, o Firewalld traz apenas uma zona ativa chamada public, que não possui regras liberadas inicialmente.  
No entanto, essa zona é herdada por todas as outras, ou seja, qualquer regra adicionada a public se aplicará às demais zonas também.  
  
Isso é bastante útil — por exemplo, se você quiser liberar a porta 3389 (RDP) para acesso remoto, basta adicioná-la à zona public e ela valerá para todos os perfis.  No exemplo abaixo vamos acrescentar a porta **3389** a zona 'public' de forma permanente:    
```
sudo firewall-cmd --zone=public --add-port=3389/tcp --permanent
sudo firewall-cmd --reload
```

Aproveite este momento para identificar quais portas precisam ser liberadas e aplique-as conforme sua necessidade.  
As portas que forem de uso comum a todos os perfis (zonas) devem ser adicionadas à zona public, garantindo que estejam disponíveis independentemente do perfil ativo.  

>⚠️ IMPORTANTE:
>Embora muitos considerem o uso de um firewall opcional em ambientes Linux de desktop, eu discordo totalmente. Mesmo em estações de trabalho, é fundamental manter um firewall ativo e configurado, pois ele protege serviços locais e deixa seu ambiente mais próximo do ambiente de produção, onde o firewall quase sempre está habilitado.  No entanto, nunca desative o firewall permanentemente ou ignore políticas básicas de segurança — isso elimina uma camada essencial de proteção que o Linux oferece por padrão.  


## AJUSTANDO ALIASES PARA COMANDOS REPETITIVOS
Aliases não são programas, e sim um recurso presente em praticamente todas as distribuições Linux que permite abreviar ou simplificar comandos repetitivos. 
Por exemplo, se você quiser listar os arquivos de uma pasta com cores e tamanhos em formato legível (KB, MB, GB), precisaria digitar algo assim toda vez:
```  
ls -lh --color=auto'
```  

Isso é muito comprido, então que tal apenas digitar 'l' e o sistema dar o comando acima? É isso que faremos agora, execute:
```  
nano ~/.bashrc
```
O arquivo acima é um arquivo de autoexecução que é rodado sempre que você acessa o terminal bash, acrescente ao final deste arquivo, seus aliases, por exemplo:
```  
alias l='ls -lh --color=auto'
```
Vamos a algumas sugestões minhas, algumas delas ao editar o ~/bashrc verá que sua distro já os tem ou estão comentadas.  
```
###
### Meus aliases
### 
# Navegação e listagem:
alias l='ls -lh --color=auto'        # Lista detalhada, tamanhos legíveis, com cores (comentaada no debian)
alias la='ls -lha --color=auto'      # Lista tudo, incluindo arquivos ocultos  (comentaada no debian)
alias ll='ls -lh --color=auto'       # Lista longa, mas ignora ocultos (comentaada no debian)
alias ls='ls --color=auto'           # Força o uso de cores sempre (comentaada no debian)
alias ..='cd ..'                     # Volta um diretório
alias ...='cd ../..'                 # Volta dois diretórios
alias ....='cd ../../..'             # Volta três diretórios
alias c='clear'                      # Limpa o terminal

# Sistema e administração
alias update='sudo apt update && sudo apt upgrade -y'   # Atualiza o sistema
alias install='sudo apt install -y '                    # Instala pacotes rapidamente
alias remove='sudo apt remove '                         # Remove pacotes
alias purge='sudo apt purge '                           # Remove pacotes + configs
alias cls='clear'                                       # Outra forma de limpar tela
alias df='df -h'                                        # Exibe uso de disco em formato legível
alias du='du -h -d 1'                                   # Mostra tamanho de pastas
alias free='free -h'                                    # Memória RAM legível

# Rede
alias ping='ping -c 5'             # Faz 5 pings e para
alias myip='curl ifconfig.me'      # Mostra seu IP público
alias ports='sudo netstat -tulanp' # Lista portas em uso

# Vida longa ao sysadmin
alias grep='grep --color=auto' # Cores no grep
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias h='history'              # Mostra histórico
alias j='jobs -l'              # Lista jobs atuais
alias v='vim'                  # Abre o Vim rapidamente
```
Muito util ter seus aliases, não é mesmo?
Agora *salve* o arquivo e feche o editor (Ctrl+O, Enter, Ctrl+X).  

Depois reinicie o seu terminal e para testar um dos aliases, execute:  
```  
$ l
total 0
drwxr-xr-x 1 gsantana gsantana 20 out 10 17:37 'Área de trabalho'
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:37  Documentos
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:41  Downloads
drwxr-xr-x 1 gsantana gsantana 32 out 10 18:48  Imagens
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:37  Modelos
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:37  Músicas
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:37  Público
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:37  Vídeos
```  
Pronto — agora voce tem comandos mais *breves* para as atividades mais costumeiras.  

> 💡 Curiosidade histórica:  
> O uso de aliases e comandos curtos vem dos primeiros sistemas Unix, em que as conexões remotas eram muito lentas — cada caractere digitado economizava tempo e largura de banda. Essa cultura de abreviar comandos (como ls, cp, mv, rm) se manteve até hoje, por eficiência e praticidade.



## AJUSTANDO O PROMPT NO TERMINAL
Às vezes o prompt do terminal pode incomodar alguns, por exemplo, é justo que ao logarmos em servidores o terminal revele no prompt seu username e nome do computador:
![Prompt normal](https://github.com/gladiston/fedoralinux/blob/main/mudando_prompt01.png)

porém ao usarmos o desktop sabemos quem somos e que computador é, então vamos ajustar o terminal para não mostrar essas duas informações. A variável de ambiente que gostaríamos de modificar que faz o prompt refletir o que desejamos chama-se PS1 e podemos ajustá-la assim::
```
export PS1='${debian_chroot:+($debian_chroot)}\[\033[32;40m\]\w:\[\033[00m\] ' 
```
Ele deixará nosso prompt "oldschool", colorido:
![Novo prompt](https://github.com/gladiston/fedoralinux/blob/main/mudando_prompt02.png)

Note na imagem acima, nosso prompt deixou de ser o que era antes para ser uma forma verde “oldschool” com o nome da pasta onde estamos, agora não esta mais exibindo username ou computername.

Se você não gosta de exibir o caminho completo do diretório onde você está porque prefere diminuí-lo, então você deve trocar o código \w por \W, a diferença entre eles é que o W maiúsculo mostra apenas o nome do diretório que você está sem o caminho completo. Muitas pessoas preferem desse jeito e caso queiram saber o caminho apenas executam o comando **pwd**.  Vamos mostrar um prompt classico com dois pontos, porém na cor azul:

```
export PS1='${debian_chroot:+($debian_chroot)}\[\033[01;34m\]\w:\[\033[00m\] ' 
```
![Novo prompt](https://github.com/gladiston/fedoralinux/blob/main/mudando_prompt03.png)

Ou se quiser o mesmo prompt acima, mas na classica cor verde:
```
export PS1='${debian_chroot:+($debian_chroot)}\[\033[32;40m\]\w:\[\033[00m\] '
```
![Novo prompt](https://github.com/gladiston/fedoralinux/blob/main/mudando_prompt04.png)

Os “:” no meio da sentença pode ser trocado por um caractere unicode mais bacana:
```
export PS1="\e[32;40m\w➤\e[00m "
```
![Novo prompt](https://github.com/gladiston/fedoralinux/blob/main/mudando_prompt05.png)

Muito melhor, não é mesmo? Há uma página que descreve várias formas de como deixar seu prompt:  
https://www.ibm.com/developerworks/linux/library/l-tip-prompt/ 
Experimente todas as opções que puder e ao final determine o prompt que deseja usar. Mas tem um problema, ao definir o tipo de prompt que desejamos, não queremos executar “export PS1=...”  todas as vezes que formos usar o terminal, isso nos daria muito trabalho. Precisamos de um jeito de automatizar isso, então o primeiro passo é descobrir o tipo de terminal que usamos, execute: 
```
echo $TERM
```
Observe o resultado:  
>xterm-256color  

Agora que sabemos que o nome é xterm-256color então edite o arquivo ~/.bashrc (o til representa a localização da sua pasta $HOME), então editamos este arquivo:  
```
nano ~/.bashrc
```
E ao final do arquivo ou numa localidade melhor, pois alguns .bashrc as vezes tem IFs que discriminam terminal colorido de não colorido vou acrescentar:
```
# Meu ajuste de terminal ao estilo old school
export PS1='${debian_chroot:+($debian_chroot)}\[\033[32;40m\]\w➤\[\033[00m\] '
```

Ficando mais ou menos assim nosso arquivo:  
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
![Novo prompt](https://github.com/gladiston/fedoralinux/blob/main/mudando_prompt06.png)  

Muito bacana, hein?

## ACESSAR PARTIÇÕES LINUX NO SISTEMA
Se utiliza uma ou mais partições Linux que não estão automaticamente montadas você pode usar o gerenciador de arquivos do KDE ou GNOME para acessá-la, mas toda vez que fizer isso, provavelmente lhe será pedido uma senha e isso cansa a vida do desenvolvedor. Minha recomendação é deixar essas partições já montadas e disponiveis imediatamente após o boot. Para conseguir isso, vamos a um exemplo:
```
lsblk -f
```
E então verá algo parecido com isso:  
```
NAME        FSTYPE FSVER LABEL   UUID                                 FSAVAIL FSUSE% MOUNTPOINTS
sda                                                                                  
└─sda1      ext4   1.0   #disco2 b2154643-7b94-42a1-8146-267bb88ba833                
sdb                                                                                  
nvme0n1                                                                              
├─nvme0n1p1 vfat   FAT32         CF05-E144                             943,3M     1% /boot/efi
├─nvme0n1p2 swap   1             14ef5e32-fbfe-4fbe-a10a-25df502a6039                [SWAP]
├─nvme0n1p3 ext4   1.0   #boot   c279ec54-2e8c-4534-a9de-eeefdbd285c3  684,7M    19% /boot
└─nvme0n1p4 btrfs        #disco1 7f257ca3-213c-4423-a0b9-8cac39089205  921,7G     1% /
```
Veja que minhas partições tem etiquetas (label), assim fica muito mais fácil de identificá-las para montagem do que se guiar por nomes como: sda1, sda2, etc...   
Além da partição NVME onde tem meu sistema inteira instalado, há um disco adicional em /dev/sda1, cujo label é 'ti-01-disco2' e o UUID é 'b2154643-7b94-42a1-8146-267bb88ba833'.   

Primeiro, vamos criar uma pasta vazia para montagem:  
```
sudo mkdir -p /mnt/disco2
sudo chown -R $USER:$USER /mnt/disco2
sudo chmod -R 2777 /mnt/disco2
```
Os comandos acima garantirão pleno acesso ao conteúdo do que for montado. Depois vamos editar o arquivo /etc/fstab:
```
sudo nano /etc/fstab
```
E acrescentamos a seguinte linha usando como exemplo a etiqueta(label) do disco:    
```
# Meu disco#2
LABEL=#disco2  /mnt/disco2  ext4  defaults  0  0
```
Usar etiquetas(LABEL) é interessante, mas o nome da etiqueta pode ser trocado a qualquer instante, mas digamos que o disco seja para ser usado como destino de backup e você não deseja que a troca da etiqueta afete seus scripts de backup? Sua solução neste caso é usar UUID, veja este exemplo:  
```
UUID=b2154643-7b94-42a1-8146-267bb88ba833  /mnt/disco2  ext4  rw,user,exec,auto,umask=000  0  0
```
Salve o arquivo, saia do editor.  
Toda vez que modificar o arquivo 'fstab', precisará executar um comando para que o sistema reconheça as mudanças, execute então:
```
sudo systemctl daemon-reload
```
Alguns vão sugerir 'trocar' o 'defaults' por 'rw,user,exec,auto,umask=000', mas isso nem sempre funciona porque pode variar do tipo de partição que irá usar. Melhor deixar 'defaults' e usar ACLs de permissão de acesso com os comandos chown/chmod.  

Parametro|Explicação
|:--|:--|
ext2,ext3,ext4...|Tipo de partição que pretende montar, aceita-se muitos tipos de partições incluindo as de windows como vfat e ntfs. Dependendo do tipo partição, as outras opções de montagem podem variar.
users|Permite que usuários normais montem e desmontem o compartilhamento, não apenas o superusuário (root).  
rw|Especifica que o compartilhamento deve ser montado com permissões de leitura e escrita.  
user,exec,umask=000|Monta com permissões abertas (qualquer usuário pode ler/gravar/executar).  
nosuid|Impede a execução de arquivos com permissões suid (Set User ID), o que pode ser um risco de segurança em compartilhamentos de rede.  
nodev|Impede a criação de arquivos de dispositivo no compartilhamento montado.  
file_mode=0777|Define as permissões para arquivos dentro do compartilhamento montado como 0777, o que concede permissões totais (read/write/execute) para todos os usuários.  
dir_mode=0777|Define as permissões para diretórios dentro do compartilhamento montado como 0777, também concedendo permissões totais para todos os usuários.  
auto|Faz a montagem diretamente no boot  
noauto|Não faz a montagem automática durante o boot  
zero e zero no final da linha|Desativa dump e fsck automático.  


Reinicie o sistema, depois de fazer o login, observe agora seu gerenciador de arquivos:  
![Gerenciador de disco mostrando as etiquetas fornecidas](debian-gerenciador-discos-montados.png)


### Diferença entre /mnt e /media  
Será que tem diferença entre usar /mnt ou /media para suas montagens de disco?  

|Diretório|Uso recomendado|
|:--|:--|
| **`/mnt`**   | Montagens **manuais ou permanentes** administradas pelo usuário ou pelo sistema.| Ideal para discos fixos, partições internas, volumes que ficam sempre disponíveis. |
| **`/media`** | Montagens **automáticas e removíveis**, geralmente gerenciadas pelo ambiente gráfico (ex: pendrives, HDs USB, DVDs). | Ideal para mídias removíveis, montadas automaticamente pelo udisks/udev. Também uso ela para unidades de rede. Na prática, tudo que pode ser ejetado, incluindo unidades de rede, eu uso /media|

### Sem acesso aos discos montados?
Se perceber que não tem acesso a modificação ao conteúdo disco montado, provavelmente é porque haviam permissões pré-existentes indicando outro usuário, isso pode ser corrigido repetindo o comando:  
```
sudo chown -R $USER:$USER /mnt/disco2
sudo chmod -R 2777 /mnt/disco2
```
Claro, existem outras formas de dar permissão, incluindo ACLs que já são conhecidas no Windows, mas isso é assunto para outro tópico.  


## ACESSAR PARTIÇÕES NTFS NO SISTEMA
Se utiliza uma partição Windows (NTFS) para gravar seus arquivos e dados a partir do Linux, você pode simplesmente não fazer nada e usar o gerenciador de arquivos do GNOME, KDE e afins para entrar e sair do disco NTFS quando quiser. Contudo, se você tem que ir para o terminal e acessá-lo dali então lhe seria conveniente criar uma pasta vazia que ao entrar nela você já observasse o conteúdo dessa partição, se você gostou da idéia então vamos implementá-la.  
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
sudo chmod 2777 /media/label_dados
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
|Parametro|Explicação
|:--|:--|
|users|Permite que usuários normais montem e desmontem o compartilhamento, não apenas o superusuário (root).|
|rw|Especifica que o compartilhamento deve ser montado com permissões de leitura e escrita.|
|nosuid|Impede a execução de arquivos com permissões suid (Set User ID), o que pode ser um risco de segurança em compartilhamentos de rede.|
|nodev|Impede a criação de arquivos de dispositivo no compartilhamento montado.|
|file_mode=0777|Define as permissões para arquivos dentro do compartilhamento montado como 0777, o que concede permissões totais (read/write/execute) para todos os usuários.|
|dir_mode=0777|Define as permissões para diretórios dentro do compartilhamento montado como 0777, também concedendo permissões totais para todos os usuários.|
|auto|Faz a montagem diretamente no boot|
|noauto|Não faz a montagem automática durante o boot|
|zero e zero no final da linha|Desativa dump e fsck automático.|


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
Alternativas: Existe um serviço chamado AutoFS, ele implementa uma solução onde você indica pastas e apenas quando você acessá-las, ele as monta.  
Serve para discos externos, partições internas e também para compartilhamentos remotos. Esta última, é o motivo pelo qual é mais usado visto que auto-montar pastas que já estão em nosso domínio é mais fácil usando o fstab. AutoFS é um pouco mais complicado que usar /etc/fstab, mas nem tanto depois que você entende como ele funciona.  Eu tenho receio de utilizá-lo em ambientes com pouco controle porque se houver programas que vasculhem discos eles irão montar todas as pastas que encontrarem na configuração para auto montar, talvez  voce pense na situação de vírus de computador, mas ocorreria algo similar em softwares de backups que podem erroneamente incluir pastas que não deveriam. Se quiser estudar o AutoFS:

https://devconnected.com/how-to-install-autofs-on-linux/

## ACESSAR COMPARTILHAMENTOS NA REDE
Os gerenciadores de arquivos para linux acessam redes remotas, não importa o tipo, através de prefixos no inicio de URI, por exemplo, compartilhamentos smb/cifs:  
```
smb://nas01/pub # ou smb://gsantana[:senha]@nas01/pub
```
E não seria diferente se fosse um outro protocolo como ftp, sftp(ftp_ssh) ou nfs, exemplo:
```
sftp://nas01/mnt/po_nas01/pub # ou sftp://gsantana[:senha]@nas01/mnt/po_nas01/pub
```
Mas em algumas oportunidades, queremos acessar isso pelo terminal, o que fazer?
Simplesmente ir para o terminal e criar a pasta:  
```
sudo mkdir -p /media/pub
```
E depois concedemos acesso a pasta para nós mesmos e com um bitstick "2" para que que subpastas herdem as permissões da pasta-pai:  
```
sudo chown $USER:$USER /media/pub 
sudo chmod 2777 /media/pub
```
Enfim, Montando a pasta:  
```
sudo mount -t cifs //nas01/pub /media/pub -o username=gsantana,password=suasenha,domain=localdomain.lan,users,rw,nosuid,nodev,file_mode=0777,dir_mode=0777
```
Mas esse linguição ser executados todas as vezes não é uma boa ideia quando a pasta é recorrente e pelo terminal, então vamos precisar editar o arquivo /etc/fstab e supondo que desejemos incluir um compartilhamento usando o protocolo smb/cifs, então a solução seria incluir:
|/etc/fstab|
|:--|
|//nas01/pub /media/pub cifs -o username=gsantana,password=suasenha,domain=localdomain.lan,users,rw,auto,nosuid,nodev,file_mode=0777,dir_mode=0777|  


Você olha para a linha acima e já vê o problema, usuário e senha ficam expostos, então vamos tentar de outra forma, vamos incluir a linha acima da seguinte forma:
```
# Montando pasta pub
//nas01/pub /media/pub cifs credentials=/etc/cifs-credentials.gsantana.localdomain.lan,users,rw,auto,nosuid,nodev,file_mode=0777,dir_mode=0777,auto 0 0
```
Salve o arquivo fstab e saia do editor. 
Novamente, toda vez que modificar o arquivo 'fstab', precisará executar um comando para que o sistema reconheça as mudanças, execute então:  
```
sudo systemctl daemon-reload
```

Mas como poderá notar no arquivo acima, no lugar de usuario+senha informamos um arquivo, será este arquivo que fornecerá as autenticações necessárias. Então vamos editar e/ou criar o arquivo /etc/cifs-credentials.gsantana.localdomain.lan:
```
sudo nano /etc/cifs-credentials.gsantana.localdomain.lan
```
E acrescente as linhas a este arquivo as linhas:  
```
username=gsantana
password=suasenha
domain=localdomain
```
E mudamos a permissão do arquivo acima para que outras pessoas não consigam ler este arquivo e descobrir nossas credenciais:
```
sudo chmod 600 /etc/cifs-credentials.gsantana.localdomain.lan
```
Agora vamos testar a montagem nas pastas recém criadas:  
```
sudo mount -t cifs //nas01/pub /media/pub -o credentials=/etc/cifs-credentials.gsantana.localdomain.lan,rw,nosuid,nodev,file_mode=0777,dir_mode=0777
```
Funcionou? Espero que sim, mas tome muito cuidado com o arquivo /etc/cifs-credentials.gsantana.localdomain.lan, se houver mensagens como:
```
mount error(13): Permission denied
Refer to the mount.cifs(8) manual page (e.g. man mount.cifs) and kernel log messages (dmesg)
```
Então significa que o usuário e/ou senha e/ou dominio estão errados. Para uso do dominio, não se usa o nome inteiro qualificado como 'localdomain.lan' apenas 'localdomain' será o suficiente. Em alguns casos, seria bom que os nomes de hosts utilizados também estejam discriminados em /etc/hosts.   

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
|zero e zero no final da linha|Desativa dump e fsck automático.|


## BANCO DE DADOS FIREBIRD
O FirebirdSQL não é empacotado para Debian, RedHat ou outras distros, isso já aconteceu no passado, mas atualmente o FirebirdSQL inclui seu próprio instalador, mas antes de prosseguir com a instalação dele, vamos instalar a lib 'libtommath' que é uma dependencia, execute:
```
sudo apt install-y libtommath-dev
```
Agora vá até a [página oficial do FirebirdSQL](https://firebirdsql.org/downloads) e baixe a ultima versão para Linux:  

![Download do Firebird](firebird-download.png)

Digamos que tenha baixado em ~/Downloads, vamos descompactá-lo:

```
tar zxvf Firebird-5.0.3.1683-0-linux-x64.tar.gz
```
E então alguns arquivos serão extraídos:  
>Firebird-5.0.3.1683-0-linux-x64/
>Firebird-5.0.3.1683-0-linux-x64/manifest.txt
>Firebird-5.0.3.1683-0-linux-x64/buildroot.tar.gz
>Firebird-5.0.3.1683-0-linux-x64/install.sh

A descompressão irá criar uma subpasta, vamos entrar nela e executar o instalador:  
```
cd Firebird-5.0.3.1683-0-linux-x64/
```
E executo o instalador:  
```
sudo ./install.sh
```
Durante a instalação lhe será questionado qual será a senha do SYSDBA, informe o que desejar, inclusive 'masterkey' se for usá-lo como desenvolvimento.
A instalação será feita em /opt/firebird e já será iniciado por padrão, mas se preferir conferir, execute:
```
sudo systemctl status firebird
```
Caso o serviço, por alguma razão estranha não tenha sido iniciado, recomendo os comandos que habilitem e iniciem o serviço:  
```
sudo systemctl enable firebird
sudo systemctl start firebird
```
E pronto, agora repita o comando 'sudo systemctl status firebird' e notará que o serviço já esta habilitado.   

### BANCO DE DADOS FIREBIRD - GRUPO FIREBIRD
O serviço de banco de dados FirebirdSQL é mantido por usuário criado com permissões restritas chamado 'firebird', isso é uma medida de segurança em sistemas posix para impedir que um hacker do mal aproveite-se de alguma falha neste serviço e tente escalar permissões maiores. Isso funciona muito bem, porém impede que outras pessoas se conectem localmente (não confundir com acesso ao localhost) a qualquer banco de dados porque apenas o usuário/grupo 'firebird' tem acesso a eles. Para que você possa contornar esta situação, seu login precisa estar no grupo 'firebird', então execute:
```  
sudo usermod -aG firebird "$USER"
newgrp firebird
```
Agora, para conferir, execute:
```  
groups
```
Observe o resultado do comando:  
> **firebird** cdrom floppy audio dip video plugdev users netdev scanner bluetooth lpadmin gsantana

Se o nome 'firebird' aparecer na relação então é um indicativo que a operação foi realizada com sucesso.   

### BANCO DE DADOS FIREBIRD - PERMISSÕES
Se for copiar arquivos de banco de dados - geralmente .fdb - para seu sistema, não os copie para seu $HOME, pois o firebird não tem acesso lá, ele até poderia se ajustado de acordo, mas isso violaria a parte de segurança. Então sempre que for copiar arquivos .fdb para este servidor, localize uma pasta externa ao $HOME, por exemplo, /var/banco, se ele não existir, crie-o:  
```  
sudo mkdir -p /var/banco
```  
Vamos dar permissão com bitstick para que subpastas herdem a pasta do pai, mas permissões não incluem "outros", apenas o firebird, execute:
```  
sudo chmod -R 2660 /var/banco
sudo chown -R firebird:firebird /var/banco
```  
Pronto, agora você pode copiar arquivos .fdb para essa pasta, em alguns casos, depois que copiar, é muito provavel que também precise repetir este comando:
```  
sudo chown -R firebird:firebird /var/banco
```  
Pois o arquivo copiado pode ter outro 'dono' e este mantêm-se ao invés de ter como dono, o usuário 'firebird'.  

### BANCO DE DADOS FIREBIRD - ALIASES PARA ESCONDER OS BANCOS
Ao conectar-se ao banco de dados, geralmente usamos algo como:
|String de conexão:|
|:--|
|localhost/3050:/var/banco/banco.fdb|
 
Qual o problema disso? Essa string revela de forma cruel onde nosso banco de dados está localizado dentro do servidor. Qual a maneira correta? A maneira correta é criar um alias e usá-lo na string de conexão do banco no lugar do caminho do banco. 
Edite o arquivo /opt/firebird/databases.conf, execute:  
```  
sudo nano /opt/firebird/databases.conf
```  
E acrescente ao final do arquivo:  
```  
banco.link = /var/banco/banco.fdb 
```  
Agora, poderá usar a seguinte string de conexão:  
|String de conexão:|
|:--|
|localhost/3050:banco.link|

E assim, nenhum caminho para nossos arquivos de dados serão revelados, isso serve muito bem para o ambiente de produção como também o de desenvolvimento porque geralmente desenvolvimento espelha a forma de produção. Caso seu banco de dados precise de parametros de ajustes associados ao banco, o /opt/firebird/databases.conf deverá ser modificado como o exemplo abaixo:  
```  
banco.link = /var/banco/banco.fdb 
{
  RemoteAccess = true
  DefaultDbCachePages = 131072
  LockMemSize = 30M
  TempCacheLimit  = 512M  
  StatementTimeout = 0 # 0=ilimitado
}
```  
E assim, cada banco de dados, além de possuir seu alias, terá também sua parametrização.  

### BANCO DE DADOS FIREBIRD - LIBERANDO PERMANENTEMENTE PORTAS NO FIREWALL
Caso tenha instalado o firewalld, sistema de firewalling, então precisaremos liberar a porta usada por este serviço, mas qual porta? Paa descobrir qual porta o Firebird usa, execute:  
```
sudo cat /opt/firebird/firebird.conf |ag RemoteServicePort
```
E observe o resultado:   
>\# found in the 'services.' file), then the 'RemoteServicePort'.  
>>RemoteServicePort = 3050  

No exemplo acima, a nossa porta é **3050**, se for um desktop de desenvolvimento, sugiro que troque por **8050**, isso evita acidentes onde o sujeito esqueceu de mudar o nome de host e aplicou o que não deveria em servidor de produção achando que era desenvolvimento, siga esta dica, não use as mesmas portas que um servidor de produção usaria. Muito bem, agora que sabemos o numero da porta, e neste exemplo, vamos usar tanto a **3050** como também a **8050**, execute:

```
sudo firewall-cmd --add-port=3050/tcp
sudo firewall-cmd --add-port=8050/tcp
```
Agora vamos repetir a verificação das portas atualmente liberadas:  
```  
sudo firewall-cmd --list-ports
```
E observe o resultado:  
> 22/tcp 80/tcp 443/tcp 3050/tcp 3306/tcp 3389/tcp 5432/tcp 8050/tcp  

Esses comandos aplicam as regras apenas no modo temporário até reiniciar o firewalld ou o sistema. Para que elas fiquem permanentes, execute:  
```
sudo firewall-cmd --runtime-to-permanent
```
Agora, vamos reiniciar o firewall:  
```
sudo firewall-cmd --reload
```
Agora vamos repetir a verificação das portas atualmente liberadas:  
```  
sudo firewall-cmd --list-ports
```
E observe o resultado:  
> 22/tcp 80/tcp 443/tcp 3050/tcp 3306/tcp 3389/tcp 5432/tcp 8050/tcp

Como pode observar acima, as regras não sumiram. Então, este procedimento foi concluido com sucesso.   


### BANCO DE DADOS FIREBIRD - VARIAVEIS DE AMBIENTE
Essas variaveis serão usadas para ao inves de logar-se digitando a usuário e senha, elas sejam suprimidas, você pode achar que isso é um risco, mas o vamos colocá-la no nosso perfil onde somente nós mesmos e o root tem acesso, então não há riscos. Edite o arquivo ~/.bash_profile:
```  
sudo nano ~/.bash_profile
```
E acrescente as linhas:  
```  
export FIREBIRD_MSG=/opt/firebird
export ISC_USER=SYSDBA
export ISC_PASSWORD=masterkey
```  
Agora *salve* o arquivo e feche o editor (Ctrl+O, Enter, Ctrl+X) e estará pronto, toda vez que abrir o terminal 'bash' as variaveis acima já estarão prontas.  
Inclusive muitos serviços de rest/api são iniciados dessa maneira, usando variaveis de ambiente ao inves de agendar a execução no inicio do boot e os seus parametros porque esses comandos com o uso de uma 'ps auxwww' seriam revelá-los e se houverem parametros de usuário e senha, então seriam revelados.  
Para testar, execute:  
```  
echo $ISC_USER
```
Observe o resultado:  
> SYSDBA  
Indicando que a variavel já está com cnoteúdo.  

### BANCO DE DADOS FIREBIRD - VARIAVEIS DE AMBIENTE GLOBAIS
Também podemos criar variaveis de ambiente globalmente, neste caso, todos os usuários se beneficiam dessas variaiveis, faça isso quando todos os usuários e/ou serviços precisam se beneficiar dessas variaveis. Edite o arquivo /etc/environment.d/999-firebird.conf:
```  
sudo nano /etc/environment.d/999-firebird.conf 
```
A pasta /etc/environment.d contêm arquivos `.conf` que o sistema lerá durante o processo de boot, novamente colocamos o prefixo "999" porque a lista de arquivos é lida alfabeticamente e desejamos que nosso arquivo fique por ultimo. Depois acrescente as linhas:  
```  
FIREBIRD_MSG=/opt/firebird
ISC_USER=SYSDBA
ISC_PASSWORD=masterkey
```  
Note que não precisamos do comando "export" porque isso não é um script, mas um arquivo de configuração que criará as variaveis de ambiente para nós durante o boot.
Agora *salve* o arquivo e feche o editor (Ctrl+O, Enter, Ctrl+X) e estará pronto, basta apenas reiniciar o sistema para que as modificações entrem em vigor, depois disso, se quiser testar, execute:

Para testar, execute:
```  
echo $ISC_USER
```
Observe o resultado:  
> SYSDBA  
Indicando que a variavel já está com cnoteúdo.  


### BANCO DE DADOS FIREBIRD - AJUSTANDO PATH
Vez ou outra, alguns programas são instalados em diretórios não convencionais, e isso inclui seus binários executáveis.
Um exemplo típico é o FirebirdSQL, que é instalado em /opt/firebird, e seus utilitários (como isql e gbak) ficam em /opt/firebird/bin.

O problema é que, ao tentar executar um desses utilitários, você precisa digitar o caminho completo e as vezes usando 'sudo', por exemplo:
```
sudo /opt/firebird/bin/gbak (...)
```
Como bons programadores, sabemos que digitar caminhos longos repetidamente não é nada prático.  
Para resolver isso, existem duas abordagens:  
1. Criar links simbólicos dos executáveis em /usr/bin; ou  
2. Acrescentar o diretório dos binários ao $PATH do sistema.  

A segunda opção é a mais limpa e flexível, pois o caminho será reconhecido automaticamente por todos os usuários e sessões.
Para configurá-la, crie (ou edite) um script Bash em: 
```  
sudo nano /etc/profile.d/999-firebird-path.sh
```
E insira o seguinte conteúdo:
```  
# Adiciona o Firebird ao PATH do sistema
# Torna o caminho acessível a todos os usuários de login
if [ -d /opt/firebird/bin ]; then
  PATH="$PATH:/opt/firebird/bin"
  export PATH
fi
```
Agora *salve* o arquivo e feche o editor (Ctrl+O, Enter, Ctrl+X).  
Então, dê permissão de leitura global:
```  
sudo chmod 644 /etc/profile.d/999-firebird-path.sh
```  
Note que a permissão "644" não dá poder de execução, e a ideia é essa mesma, o próprio sistema se encarregará de carregá-lo por uma sucessão de scripts, não precisando de "alguém" para executá-lo. Para uma outra pessoa poder executar manualmente, seria:
```  
source /etc/profile.d/999-firebird-path.sh
```
Agora vamos conferir o PATH, execute:  
```  
echo $PATH
```
E observe o resultado:  
>/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:**/opt/firebird/bin**

Se '/opt/firebird/bin' apareceu então este tópico foi concluido com sucesso.


## HABILITANDO AREA DE TRABALHO REMOTA
Vez ou outra precisaremos acessar nossa area de trabalho, as mais experientes recomendarão usar o 'ssh -x' ou usar 'xserver' e logar-se no ip de nosso desktop, no entanto, isso não é tão simples para novos usuários do linux e também não permite o acesso onde a origem é um desktop Windows. Portanto, minha recomendação é instalar o xrdp, um protocolo de compartilhamento de sessões compativel com o 'rdp' da Microsoft e assim poderemos acessar nosso terminal Linux até mesmo de um Windows através do programa 'Remote Deskop'. Para instalar:  
```  
sudo apt install -y xrdp remmina
```
Verifique se o grupo 'ssl-cert' existe, execute:
```  
getent group ssl-cert
```
Observe o resultado indicando a existencia:  
```
ssl-cert:x:105:  
```
Se a resposta for afirmativa como acima, então agora verifique se o usuário 'xrdp' também exista. execute:
```  
$ getent passwd xrdp  
```
E observe o resultado indicando a existencia:  
```
xrdp:x:111:116::/run/xrdp:/usr/sbin/nologin  
```

Agora que sabemos que o grupo 'ssl-cert' existe, e o usuário 'xrdp' também, então adicione o usuário 'xrdp' ao grupo 'ssl-cert' para permitir que o serviço xrdp acesse as chaves SSL corretamente, execute:
```  
sudo usermod -aG ssl-cert xrdp
```
Agora precisaremos atualizar as permissões imediatamente, execute:
```
sudo systemctl restart xrdp
```
Ou, se preferir, reinicie o sistema (o login de grupo é aplicado na próxima sessão).  
Agora o serviço xrdp pode acessar os certificados em /etc/ssl/private/, permitindo conexões seguras via TLS sem erros.  

### HABILITANDO AREA DE TRABALHO REMOTA - CRIANDO UM CERTIFICADO SSK OARA I XRDP
1. Gere o certificado e a chave privada
Use o openssl para criar um certificado autoassinado válido por 10 anos (ajuste conforme quiser):
```
sudo openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes \
  -keyout /etc/ssl/private/xrdp.key \
  -out /etc/ssl/certs/xrdp.crt \
  -subj "/CN=$(hostname)"
```
Explicação:  
* -x509 → cria um certificado autoassinado
* -newkey rsa:4096 → gera uma nova chave RSA de 4096 bits
* -nodes → não criptografa a chave com senha (requerida para uso automático)
* -subj → define o nome comum (CN) com o hostname do servidor

2. Ajuste as permissões corretamente, somente root e membros do grupo ssl-cert devem ter acesso:
```
sudo chown root:ssl-cert /etc/ssl/private/xrdp.key
sudo chmod 640 /etc/ssl/private/xrdp.key
sudo chmod 644 /etc/ssl/certs/xrdp.crt
```

3. Configure o XRDP para usar o novo certificado, edite o arquivo /etc/xrdp/xrdp.ini:
```
sudo nano /etc/xrdp/xrdp.ini
```
E inclua o seguinte conteúdo:  
```  
certificate=/etc/ssl/certs/xrdp.crt
key_file=/etc/ssl/private/xrdp.key
```
4. Reinicie o serviço XRDP, execute:
```  
sudo systemctl restart xrdp
sudo systemctl status xrdp
```
Agora, ao se conectar via RDP (Windows, Remmina, etc.), o XRDP usará seu certificado SSL personalizado. Mas vamos validar o certificado manualmente, execute:  
```  
openssl x509 -in /etc/ssl/certs/xrdp.crt -text -noout
```
Pronto, agora o resultado esperado é:  
* Conexão RDP criptografada com TLS
* Nenhum erro relacionado a certificado inválido ou permissão negada  
* Logs limpos em /var/log/xrdp-sesman.log e /var/log/xrdp.log  

Este certificado local funcionará em sua rede local, mas se for para um acesso externo, precisará do 'certbot' que crescente uma seção opcional com integração via certificado Let’s Encrypt (SSL público e renovável) para uso remoto pela internet.

## VIRTUALIZAÇÃO NATIVA QEMU+KVM
O Linux é capaz de criar máquinas virtuais e ele mesmo ser o hypervisor. Será um servidor de virtualização nivel 1, o mais rápido possivel, no entanto com algumas ausencia de recursos que facilitam a configuração que existem no VirtualBox e VMWare, por exemplo, criar redes virtuais com vários tipos de topologias,  clipboard e transferencia de arquivos entre host e anfitrião e outras coisas.  
### Vamos instalar os pacotes principais:  
```
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils dnsmasq-base ovmf
```
Pacote|Explicação
|:--|:--|
libvirt-daemon-system|Configura o daemon libvirtd para gerenciar VMs via KVM.  
libvirt-clients|Ferramentas CLI (virsh, virt-install, etc.).  
dnsmasq-bas|Fornece DHCP/NAT automáticos para redes virtuais.  
ovmf|Permite boot UEFI em VMs (necessário para Windows/modernos).  

### Permitir uso sem root
Adicione seu usuário ao grupo libvirt (e kvm):
```  
sudo usermod -aG kvm $USER
sudo usermod -aG libvirt $USER
```  

### Para uso em Desktops
Por tratar-se de um desktop, faça a instalação mais completa:
```  
sudo apt install -y virt-manager virtiofsd
```

Pacote|Explicação
|:--|:--| 
virt-manager|Para uso em desktop ou estação de trabalho, o virt-manager é praticamente indispensável.  
virtiofsd|O pacote virtiofsd fornece o daemon do Virtio-FS, que é o método moderno (e mais rápido) para compartilhar pastas entre host e VMs Linux.  

3. Conferindo o KVM
Agora, verifique se os módulos do KVM estão carregados no kernel:
```
lsmod | grep kvm
```
Uma saída aceitável seria:  
```
kvm_amd               217088  0
kvm                  1396736  1 kvm_amd
irqbypass              12288  1 kvm
ccp                   163840  1 kvm_amd
```
Se constar na lista o módulo ‘kvm’, então tá tudo certo.

Depois, *reinicie o computador*.
Depois do login, verifique se realmente estou nestes grupos:
```  
groups
```  
Você deve ver:
>**gsantana** (...) **libvirt** **kvm** (...)


### Programando o início do serviço
Se os módulos acima aparecem então agora é o momento de prepará-los para iniciar-se como serviço durante o boot, assim, inicie o serviço do libvirtd com:  
```
sudo systemctl start libvirtd
```
E para iniciar o serviço durante o boot, execute:
```
$ sudo systemctl enable libvirtd
Synchronizing state of libvirtd.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable libvirtd
```
Confira que o serviço esta ativado:
```
$ sudo systemctl status libvirtd
● libvirtd.service - libvirt legacy monolithic daemon
     Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-10-08 16:26:55 -03; 41s ago
 Invocation: dfc2bb59b8ae4ab3929c9385a657e489
TriggeredBy: ● libvirtd-ro.socket
             ● libvirtd-admin.socket
             ● libvirtd.socket
       Docs: man:libvirtd(8)
             https://libvirt.org/
   Main PID: 3956 (libvirtd)
      Tasks: 21 (limit: 32768)
     Memory: 6.2M (peak: 8M)
        CPU: 199ms
     CGroup: /system.slice/libvirtd.service
             └─3956 /usr/sbin/libvirtd --timeout 120

out 08 16:26:55 ti-01 systemd[1]: Starting libvirtd.service - libvirt legacy monolithic daemon...
out 08 16:26:55 ti-01 systemd[1]: Started libvirtd.service - libvirt legacy monolithic daemon.
```
Se retornou 'Active: active' então tá tudo certo.

### Localização das VMs
Por padrão a localização a localização das máquinas virtuais fica em:
```
/var/lib/libvirt/images
```
Pessoalmente, se você tem /var junto a partição /(root), este não é o local mais adequado, assim recomendo que suas VMs estejam numa partição com mais espaço, por exemplo, o seu $HOME em:  
```
/home/$USER/libvirt
```
Se estiver usando uma partição Btrfs, isso não se aplica e /var/lib/libvirt/images é um bom local, então ignore este subtópico.
Mas caso não use Btrfs, vamos mudar a localização original das VMs para nsso $HOME, primeiramente precisaremos incluir um novo poll:  
```
mkdir -p /home/$USER/libvirt/images
```
Agora que a pasta foi criada com sucesso, então redirecionar o pool de imagens para lá:  
```
virsh pool-define-as vm dir - - - - "/home/$USER/libvirt/images"
```
Pronto, novas VMs serão criadas no diretório acima.

### Localização das VMs numa partição Btrfs
Se a pasta acima estiver num tipo de partição Btrfs, este tipo de partição faz uma série de operações no disco e algumas delas são anti-performaticas para carregamento de VMs, são elas:
* CoW: O Copy-on-Write(CoW) é um recurso do Btrfs que (1) quando um arquivo é modificado, ele não é alterado diretamente e (2) o sistema cria uma nova cópia dos blocos modificados e só depois descarta os antigos e isso protege contra corrupção e permite snapshots, mas também significa que cada gravação cria fragmentação e sobrecarga de I/O. Uma coisa interessante é que o CoW pode ser desligado por pastas, então a pasta que armazena as VMs podemos desligar o CoW.
  Desabilitando CoW:
```
chattr +C /var/lib/libvirt/images
```
* Compressão de dados: No seu tempo ocioso, o Btrfs vai compactar seus arquivos, mas em maquinas virtuais que são arquivos grandes e são modificados a todo instante, não parece ser uma boa ideia e para piorar ainda mais, esse recurso não pode ser desligado por pasta, apenas para [sub]volumes inteiros. A solução é (1) você configurar no virtualizador que crie arquivos seguimentados, ao inves de uma única VM de 50GB, separá-los em vários arquivos menores, por exemplo, a medida que um arquivo enche (exemplo) 2GB, cria um arquivo seguinte e vai repetindo o processo e assim a compressão não irá atrapalhar porque o virtualizador nunca sobreescreve os arquivos seguimentados anteriores. A outra solução, (2) é desabilitando a compressão, e nesse caso, vamos pelo jeito mais simples, quando você for mais experiente, crie volumes separados para VMs para não ter que desligar a compressão para a partição/disco inteiro como faremos agora, edite o arquivo /etc/fstab e procure pela representação do seu disco/partição Btrfs, no meu exemplo, esta assim:
```
UUID=c045fd1f-7c4f-4ec3-84d9-ec79f8859adf /               btrfs   defaults,subvol=@rootfs 0       0
```
Agora, junto com as opções 'default', você acrescenta ',compress=no', ficando assim:
```
UUID=c045fd1f-7c4f-4ec3-84d9-ec79f8859adf /               btrfs   defaults,subvol=@rootfs,compress=no 0       0
```
Salve e feche o editor, então execute:
```
sudo systemctl daemon-reload
```
Note que agora, a compressão zstd para a unidade inteira esta desligada, significando que todos os arquivos ocuparão mais espaços.
Recomendo que reinicie o computador antes de prosseguir.  

Depois de reiniciar o computador, abra o terminal e execute:
```
$ sudo btrfs filesystem df /
Data, single: total=19.01GiB, used=15.59GiB
System, DUP: total=8.00MiB, used=16.00KiB
Metadata, DUP: total=2.00GiB, used=333.94MiB
GlobalReserve, single: total=35.06MiB, used=0.00B
```
Se não aparecer a palavra “*Compressed*”, significa que nenhum dado comprimido está sendo escrito — a compressão está efetivamente desativada.  


Algo também muito recomendado é a desfragmentação da pasta, pois desligamos algumas propriedades do btrfs e as imagens de VMs costumam ser grandes.   Isso pode ser feito com o comando:  
```
sudo btrfs filesystem defragment -r "/home/$USER/libvirt/images"
```
Se for possivel, use o agendador de tarefsa do Linux para rodá-lo num horário programado, execute o comando 'sudo crontab -e' e adicione a linha:
```
0 12 * * * btrfs filesystem defragment -r  "/home/gsantana/libvirt/images"
```
O comando acima, no horário 12:00 (almoço) fará a desfragmentação da pasta mencionada.

### Localização das ISOs
Também precisaremos de um repositório para guardar nossas isos, escolha o diretorio que desejar:  
```
mkdir -p /home/$USER/WinSrv/isos
virsh pool-define-as isos dir - - - - "/home/$USER/WinSrv/isos"
```

### Virtualização de Windows
Se pretende virtualizar máquinas windows precisará dessa .iso em seu sistema, eles contêm drivers de sistema convidado:  
```
cd /home/$USER/WinSrv/isos
wget -vc https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
```

Outras instruções e explicações do porque precisamos desses drivers podem ser obtidas aqui:
https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md

### Criando máquinas virtuais pelo Virt-Manager
Instruções de como usar o virt-manager encontra-se na página:
[Criando máquinas virtuais pelo Virt-Manager](https://sempreupdate.com.br/como-configurar-e-usar-o-virt-manager-para-kvm-no-fedora-ubuntu-debian-e-derivados/#google_vignette)

## VIRTUALIZAÇÃO NATIVA QEMU+KVM JUNTO COM O VIRTUALBOX
Usar o QEMU+KVM junto ou simultaneamente com o VirtualBox não é possivel, mas é possivel chavear o uso, como assim? É possivel usar o VirtualBox enquanto não usar QEMU+KVM. Funciona assim, quando você dá boot no sistema, um dos modulos do kernel é requisitado pelo QEMU+KVM e este módulo carregado interompe a virtuaização de VMs pelo VirtualBox, então o que precisa fazer é, antes de chamar o virtualbox, descarregar este modulo da memória, execute:  
```
sudo systemctl stop libvirtd # para o serviço libvirtd
#sudo systemctl disable libvirtd # desabilitar durante o boot
sudo modprobe -r kvm kvm_amd 
```
Se for Intel, use kvm_intel em vez de kvm_amd, como fiz acima. Depois, confirme que o módulo saiu:  
```
lsmod | grep kvm
```
Se irá usar o VirtualBox por um certo tempo é chato ficar executando os comandos acima todas as vezes, então neste caso, edite o arquivo blacklist-kvm.conf, execute:
```
sudo nano /etc/modprobe.d/blacklist-kvm.conf
```
E acrescente as linhas:
```
# Impede o carregamento automático do KVM para uso do VirtualBox
blacklist kvm
blacklist kvm_amd
# Para processadores Intel, troque por:
# blacklist kvm_intel
```
Salve (Ctrl+O, Enter, Ctrl+X) e depois atualize o initramfs:
```
sudo update-initramfs -u
```
Depois poderá reiniciar o sistema com 'sudo reboot' e notará que o VirtualBox funcionará de primeira.
Se quiser reverter, apenas comente as linhas no arquivo 'blacklist-kvm.conf' e repita 'sudo update-initramfs -u' e a seguir o kvm se ligará novamente ao qemu.  


## VIRTUALBOX
O VirtualBox é outro virtualizador, ele é do tipo "2" e isto significa que é um pouco inferior em performance ao qemu+kvm, no entanto, ele tem muito mais habilidades para desktop do que o virtualizador nativo, por exemplo, o SEAMLESS que permite puxar um aplicativo Windows dentro da VM para o sistema hospedeiro, causando a impressão que está rodando uma aplicação Linux nativa.  
No entanto, ele enfrenta alguns bugs chatos desde que os ambientes Linux estão saindo do Xorg para o Wayland(Debian 13). Alguns são problemas grandes, o SEAMLESS não funciona mais, e outros são problemas pequenos aleatórios e irritantes como o conteúdo da área de clipboard entre hospedeiro e convidado deixar de funcionar, cursor do mouse que deixa de funcionar e coisas do tipo, ainda estou enumerando-os. Espero que as versões recentes corrijam isso, é um bom virtualizador e tem uma opção que qemu+kvm não tem: transportar a VM para outros sistemas operacionais, isto é, você pode copiar a VM criado no Linux para rodar num hospedeiro Windows ou Mac OS.  

Para instalar é fácil, similar ao Google Chrome, você precisa acessar a página de Downloads que começa no link abaixo:  
(https://www.virtualbox.org/)

Você irá baixar a versão .deb, e dar um duplo clique no arquivo e seguir as instruções na tela.  
Depois de instalado, você volta a página de download e procura por "VirtualBox Extension Pack", baixe ele:
![VirtualBox Extension Pack](virtualbox-extension-pack.png)  

Depois dê duplo clique nele e o próprio VirtualBox o instalará.  
O "VirtualBox Extension Pack" é um pacote adicional oficial da Oracle que amplia as funcionalidades do VirtualBox, adicionando recursos que não vêm na instalação padrão, por exemplo, a VM acessa dispositivos USB mais recentes (pen drives, HDs externos, impressoras, etc.), acesso remoto via VRDP que é similar ao RDP da Microsoft, acesso a WebCAM do hospedeiro, encriptação de disco e tem outras coisas também, mas você terá de ler diretamente no site.  

O "VirtualBox Extension Pack" é gratuito para uso pessoal e educacional, mas tem uma licença diferente chamada de PUEL – Personal Use and Evaluation License) o que impede das distros empacotarem ele ou até remover completamente o VirtualBox de seus repositórios. Se você usar o virtualBox dentro de uma empresa, **uso pessoal** poderá ser questionável.  

Uma vez que tanto o **VirtualBox** como também o **Extension Pack** estão instalados, agora vamos fazer alguns ajustes.  

>**IMPORTANTE**: O principal concorrente do VirtualBox é o **VMWare WorkStation** que recentemente também tornou-se gratuito para alguns fins, se você considera instalá-lo, preciso te alertar, também sofre de alguns bugs nesta transição de Xorg para Wayland. Além disso, é fácil de instalar e complicado de manter já que a cada atualização de kernel, seus módulos precisam ser recompilados e nem sempre funcionam na versão recente. Em distros como 'Debian' que tem pouca atualização de kernel - apenas patches - é até um mundo tranquilo para usá-lo, mas em distros _blending-edge_ como o Fedora, é um inferno.  



### VIRTUALBOX - ACESSO AO GRUPO 'VBOXUSERS'
Você precisa adicionar o usuário ao grupo ‘vboxusers’ para que ele tenha permissão de acessar dispositivos USB, configurar redes em modo bridge e usar outros recursos específicos do VirtualBox. Sem isso, o VirtualBox não consegue gerenciar esses recursos de forma segura e controlada. Assim precisaremos executar:  
```
sudo usermod -a -G vboxusers $USER
```
Para verificar se você mesmo foi incluído no grupo ‘vboxusers’, execute:
```
groups $USER
```
E então, veja o resultado:  
> gsantana : gsantana cdrom floppy audio dip video plugdev users systemd-journal netdev scanner bluetooth lpadmin firebird vboxusers

Se aparecer seu login(gsantana), depois do comando acima, então tá tudo certo, vamos prosseguir.

### VIRTUALBOX - HABILITAR O SERVIÇO 'VBOXDRV'
Para garantir o desempenho consistente do VirtualBox, é essencial iniciar o serviço vboxdrv e configurá-lo para iniciar automaticamente sempre que o sistema inicializar. Isso garante que o VirtualBox esteja sempre pronto para operar sempre que você precisar. Execute o seguinte comando para conseguir isso:  
```
sudo systemctl enable vboxdrv --now
```
Depois disso, vamos conferir:  
```
$ systemctl status vboxdrv
● vboxdrv.service - VirtualBox Linux kernel module
     Loaded: loaded (/usr/lib/virtualbox/vboxdrv.sh; enabled; preset: enabled)
     Active: active (exited) since Wed 2025-10-15 08:50:37 -03; 7h ago
 Invocation: 5bcba9dd3cdc4396a6ab37f00a45c68b
   Mem peak: 26.9M
        CPU: 359ms
```
Se mostrar 'active' então tá tudo certo.  

### VIRTUALBOX - PERMISSÕES DE ACESSO A DISPOSITIVO USB
Sob algumas circunstâncias, o VirtualBox as vezes reclama de não ter acesso aos dispositivos USBs, o 'Extension Pack' deveria resolver isso, mas nem sempre.  Se você sentir este problema, algo que faço nessas situações é acrescentar uma regra **udev** ajustando as permissões do dispositivo reconhecido, crie o seguinte arquivo:  
```
sudo nano /etc/udev/rules.d/60-vboxusb.rules
```
E insira:  
```
SUBSYSTEM=="usb_device", GROUP="vboxusers", MODE="0664"
SUBSYSTEM=="usb", GROUP="vboxusers", MODE="0664"
```
E depois recarregue as regras:  
```
sudo udevadm control --reload-rules
sudo udevadm trigger
```
Para testar, plugue um pen drive>abra o menu Dispositivos>USB dentro da janela da VM>selecione seu dispositivo.
Ele deve desaparecer do sistema host e aparecer dentro da VM.

### VIRTUALBOX - CONFLITO COM A INTEGRAÇÃO DO MOUSE
Pois é, talvez por causa da transição do Xorg para o Wayland, algumas coisas ainda possam apresentar pequenos travamentos ou falhas de integração.
Um problema comum é a perda da integração do mouse dentro da máquina virtual — o ponteiro pode ficar travado, desaparecer ou não responder corretamente, especialmente quando a VM está em tela cheia (Full Screen).  

Quando isso acontecer, uma boa solução é desativar a MiniBarra de Ferramentas do VirtualBox, para isso:   
1. Selecione a sua máquina virtual no VirtualBox.  
2. Vá em Configurações → Interface do Usuário.  
3. Localize a opção MiniBarra de Ferramentas e desative.  
Com isso, a interação entre o mouse e a VM tende a voltar ao normal, mesmo em tela cheia.  

Outro ajuste que ajuda bastante é desativar o recurso de Arrastar e Soltar (Drag and Drop) da VM. Para isso, vá em Configurações>Geral>Avançado e, na opção Arrastar e Soltar, selecione **Desligado**.

Com essa configuração, junto com a MiniBarra de Ferramentas desativada, o problema de travamento do mouse praticamente desaparece — mesmo em modo Tela Cheia (Full Screen).


### VIRTUALBOX - CONFLITO COM O KVM
Se ao executar uma VM, aparecer a mensagem:  
> VirtualBox can't operate in VMX root mode.
> Please disable the KVM kernel extension, recompile your kernel and reboot (VERR_VMX_IN_VMX_ROOT_MODE).  
ou:
> VirtualBox can't enable the AMD-V extension. Please disable the KVM kernel extension, recompile your kernel and reboot (VERR_SVM_IN_USE). Código de Resultado: NS_ERROR_FAILURE (0x80004005)

Esse erro acontece porque você está tentando rodar uma VM no VirtualBox, mas o sistema já está usando a virtualização via KVM, que "ocupa" o recurso de virtualização de hardware (Intel VT-x ou AMD-V), impedindo o VirtualBox de usá-lo.  

Para resolver a situação, feche o VirtualBox e desabilite o módulo KVM temporariamente (sessão atual), se seu computador usar processadores INTEL então execute:  
```
sudo modprobe -r kvm_intel
```
Se for um processador AMD, então execute:  
```
sudo modprobe -r kvm_amd
```

Isso desativa o KVM e libera o uso da virtualização para o VirtualBox até o próximo reboot, depois voltará a mesma mensagem de erro. Para manter a solução acima em definitivo, é preciso colocar o modulo kvm numa blacklist, impedindo-a de ser carregada durante o boot. Vamos precisar editar o arquivo /etc/modprobe.d/blacklist-kvm.conf, execute:
```
sudo nano /etc/modprobe.d/blacklist-kvm.conf
```
E então adicione a seguinte linha se seu computador for Intel:
```
blacklist kvm_intel
```
Ou se for processador AMD, adicione:
```
blacklist kvm_amd
```
Salve e reinicie o computador, notará que agora é possivel iniciar as VMs dentro do VirtualBox.  


---
## DAQUI EM DIANTE SÃO PROGRAMAS RECOMENDADOS, QUE VOCE PODE OU NAO QUERER USAR
---

## SOFTWARE PARA TREINAMENTO
Para criar material de treinamento que incluirá vídeo é sugerível instalar a seguinte extensão Draw On Your Screen cuja instrução para instalação se encontra em:
https://codeberg.org/som/DrawOnYourScreen

```
git clone https://codeberg.org/som/DrawOnYourScreen --depth=1 --single-branch --branch face ~/.local/share/gnome-shell/extensions/draw-on-your-screen@som.codeberg.org
```
Depois vá até .local/share/gnome-shell/extensions e abra o arquivo metadata.json e adicione "41" e então reinicie o gnome-shell.


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


## LEITOR DE CERTIFICADO DIGITAL
Cada leitor e modelo pode ter instruções diferentes, é melhor procurar um howto na internet apropriado.



## MICROSOFT OFFICE (web apps)
Visite a página a seguir usando um navegador Google Chrome(ou compatível com webapps):
[Site do office.com](office.com)   
E autentique-se com uma conta live da Microsoft.
Estando na home da página, precisará de usar o navegador para transformar a página em um aplicativo WEB. Usando o Google Chrome você iria nos 3 pontinhos no canto superior direito>Transmitir, salvar e compartilhar>Instalar página como app...   

![Instalação do app MSOFFICE](app-msoffice-web.png)  

Na realidade, para qualquer aplicativo WEB, você seguiria estas mesmas instruções.  


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
nano ~/.local/share/applications/droidcam.desktop
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
(https://www.insynchq.com/downloads)

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
(https://www.youtube.com/watch?v=tQY5IHOnK9E)
Dá para recuperar tanto arquivos quanto o sistema operacional.



## HANDBRAKE
HandBrake é um dos mais poderosos conversores de vídeo. Para instalar basta ir na loja e escolhê-lo pelo nome handbrake.  


## FORMATLAB
FormatLab é um dos mais promissores conversores de vídeo após o HandBrake. Ele faz as mesmas atividades do handbrake, porém é mais simples de operar. Para instalar basta ir na loja e escolhê-lo pelo nome FormatLab.  



## GPARTED
GParted é um particionador gráfico para Linux, com ele podemos criar e manipular partições de discos que tenham os mais diversos sistemas operacionais. Muito intuitivo e fácil, torna operações complexas bem mais simples e por isso é importante ter muita cautela. Ele tem um método onde você planeja o que vai fazer, varias tarefas seguidamente mas só o aplica quando você confirmar. Isso é importante porque antes de você executar o procedimento você poderá cancelar a operação, pode parecer simples, mas a maioria dos particionados fazem apenas um passo de cada vez e não tem volta, então o gparted é muito eficiente e fácil. Para instalar basta ir na loja e escolhê-lo pelo nome gparted.  


## BLENDER
Blender, também conhecido como blender3d, é um programa de código aberto, desenvolvido pela Blender Foundation, para modelagem, animação, texturização, composição, renderização, e edição de vídeo.  Para instalar basta ir na loja e escolhê-lo pelo nome blender. Se você não cria animações, ignore a instalação desse programa.  

## VIDCUTTER
(analogo ao vidcoder para windows)

(http://bluegriffon.org)


## INKSCAPE
Para instalar basta ir na loja e escolhê-lo pelo nome INKSCAPE.  


## OUTROS TÓPICOS INTERESSANTES
* Ambiente de programação FreePascal/Lazarus
* Ambiente de programação JAVA
* Ambiente de programação Python
* Versionamento com o asdf

