## SUPORTE A NOVAS FONTES
A aparência das fontes influencia diretamente a legibilidade, o conforto visual e até mesmo a produtividade durante o uso do sistema. No Linux, especialmente em distribuições como Debian e Ubuntu, é possível personalizar facilmente o conjunto de fontes disponíveis — seja para o ambiente gráfico, terminais ou IDEs de programação.  
Este tutorial mostra como instalar e gerenciar fontes no sistema, incluindo opções populares entre desenvolvedores e usuários avançados. Também aborda a instalação de fontes de terceiros (como as da Microsoft) e alternativas modernas e livres que oferecem excelente qualidade visual e compatibilidade.  

Normalmente, os pacotes a seguir já vêm instalados, mas precisamos conferir, então execute:

```bash
sudo apt install -y fontconfig fontforge fonttools
```

## INSTALAÇÃO DE FONTE "INTER" e "NOTO"
As fontes Inter e Noto são famílias tipográficas modernas, criadas para oferecer clareza, compatibilidade e cobertura global.  

A Inter foi projetada especialmente para interfaces e textos em tela, com excelente legibilidade e aparência limpa em qualquer tamanho. Já a Noto, desenvolvida pelo Google, tem como principal objetivo suportar todos os idiomas e sistemas de escrita do mundo, mantendo estilo e espaçamento consistentes entre diferentes alfabetos.

Usá-las juntas garante um visual profissional, uniforme e sem “quadradinhos” (tofu) em caracteres internacionais, além de serem gratuitas, leves e otimizadas para a web — perfeitas para projetos modernos e acessíveis. Para instalá-las, execute:

```bash
sudo apt install -y fonts-inter fonts-noto
```

A fonte **fonts-inter** é bonita e versátil, ideal para uso geral, enquanto **fonts-noto-ui** se destaca por sua aparência moderna e legibilidade — uma excelente opção para ser usada na interface de aplicativos.


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
Vamos adicionar um repositório que será útil para acrescentar mais fontes ao sistema:

```bash
sudo apt install -y ttf-mscorefonts-installer
```

O pacote instalado acima complementa as **fontes Microsoft**, que podem ser necessárias para a correta exibição de documentos ou programas portados do Windows.




## INSTALAÇÃO DE FONTE SELAWIK (SIMILAR AO "Segoe UI")
O Linux não possui a fonte **Segoe UI**, idealizada pela Microsoft para o sistema Windows. Essa fonte é muito popular em interfaces gráficas (UI) do Windows. Embora seja possível copiá-la de uma instalação do Windows, isso violaria a licença de uso da Microsoft, portanto não é recomendado. O Linux não tem essa fonte, ela pode ser copiada de uma instalação do Windows, mas isso infringiria leis de copyright. O Linux não tem ela, apesar disso, existe a fonte **SELAWIK**, ela é “metrics-compatible” com a Segoe UI — ou seja, tem as mesmas métricas (altura, espaçamento) para facilitar troca sem quebrar layout. Está sob licença OFL (SIL Open Font License), o que permite uso, modificação e distribuição livremente, inclusive em Linux/web. Sua obtenção depende de cada distro, em algumas distribuições, como o Arch Linux, oferecem a Selawik diretamente nos repositórios (pacote ttf-selawik). Em outras, é necessário instalá-la manualmente, como mostrado abaixo.  

Então vamos ao procedimento de instalação:    
```bash
# Vamos ir para uma pasta de arquivos temporarios
mkdir -p /tmp/selawik
cd /tmp/selawik
wget https://github.com/winjs/winstrap/raw/master/src/fonts/selawk.ttf
wget https://github.com/winjs/winstrap/raw/master/src/fonts/selawkb.ttf
wget https://github.com/winjs/winstrap/raw/master/src/fonts/selawkl.ttf
wget https://github.com/winjs/winstrap/raw/master/src/fonts/selawksb.ttf
wget https://github.com/winjs/winstrap/raw/master/src/fonts/selawksl.ttf


# Crie o diretório de fontes locais:
sudo mkdir -p /usr/share/fonts/truetype/selawik

# Copie os arquivos .ttf:
sudo cp *.ttf /usr/share/fonts/truetype/selawik/

# Atualize o cache de fontes:
sudo fc-cache -f -v
```
Será que esta realmente instalada? Vamos conferir, execute:
Para verificar se o sistema reconheceu:
```bash
$ fc-list | grep Selawik
/usr/share/fonts/truetype/selawik/selawksl.ttf: Selawik,Selawik Semilight:style=Semilight,Regular
/usr/share/fonts/truetype/selawik/selawkb.ttf: Selawik:style=Bold
/usr/share/fonts/truetype/selawik/selawksb.ttf: Selawik,Selawik Semibold:style=Semibold,Regular
/usr/share/fonts/truetype/selawik/selawkl.ttf: Selawik,Selawik Light:style=Light,Regular
/usr/share/fonts/truetype/selawik/selawk.ttf: Selawik:style=Regular
```

Esta fonte é mencionada como  solução "cross-plataforma" para as situações onde a Segoe UI não era permitida.   


## INSTALAÇÃO DA FONTE "Segoe UI" (OPCIONAL)
O Linux não possui a fonte **Segoe UI**, e talvez você não goste da **fonts-noto-ui**, e prefere a fonte idealizada pela Microsoft para o sistema Windows. Essa fonte é realmente muito bela para ser usada em interfaces gráficas (UI) de aplicativos.  
Não está claro que a Segoe UI possa ser licenciado para uso independente em Linux ou para redistribuição ou incorporação em aplicações fora do ecossistema Microsoft. Um blog afirma: “Segoe UI … is considered exclusive to Microsoft products and operating systems. … you cannot simply buy a commercial license … to embed it on your website.”, ou seja, para  alguns é claro que o uso dessa fonte é exclusiva para quem tem licença de Windows.  
Mas se você - para uso pessoal - desejar tê-lo em seu Windows para ter compatibilidade total com produtos usando o WINE ou para outros fins, existe uma solução, simplesmente copiá-las apra seu computador.  As instruções a seguir foram tiradas [deste repositório](https://github.com/mrbvrz/segoe-ui-linux) então assuma seu próprio risco, as instruções são:  
```bash
cd /tmp
git clone https://github.com/mrbvrz/segoe-ui-linux
cd segoe-ui-linux
chmod +x install.sh
sudo ./install.sh
```
Uma tela semelhante a seguir lhe será mostrada e então você confirma com "y":
```
                                         _    __            _   
                                        (_)  / _|          | |  
  ___  ___  __ _  ___   ___        _   _ _  | |_ ___  _ __ | |_ 
 / __|/ _ \/ _  |/ _ \ / _ \  __  | | | | | |  _/ _ \|  _ \| __|
 \__ \  __/ (_| | (_) |  __/ (__) | |_| | | | || (_) | | | | |_ 
 |___/\___|\__, |\___/ \___|       \__,_|_| |_| \___/|_| |_|\__|
            __/ |                                               
           |___/              mrbvrz - https://hasansuryaman.com        

 ---------------------------------------------------------------

 [ * ] Checking for internet connection
Testing internet connectivity on interface: lo
 [ X ] Internet Connection on interface lo ➜ OFFLINE!

Testing internet connectivity on interface: enp8s0
 [ ✔ ] Internet Connection on interface enp8s0 ➜ CONNECTED!

 [ * ] Checking for Wget
 [ ✔ ] Wget ➜ INSTALLED

 [ * ] Checking for Segoe-UI Font
 [ X ] Segoe-UI Font ➜ NOT INSTALLED

 Do you want to install Segoe-UI Font? (y)es, (n)o :
```
**Novamente fica o aviso**: Não há uma licença pública da Microsoft que permita livre uso da Segoe UI em sistemas Linux ou para redistribuição / embutimento em aplicações cross-platform sem contato direto/licenciamento especial com a Microsoft ou seus parceiros. Se você fizer isso, há risco de violação de licença.


## INSTALAÇÃO DA FONTE ROBOTO
A fonte **fonts-roboto** é bastante interessante para uso em terminais e IDEs de programação:

```bash
sudo apt install -y fonts-roboto fonts-roboto-fontface fonts-roboto-slab
```

Para conferir se a fonte foi realmente instalada, execute:

```bash
fc-list | grep "roboto"
```


## INSTALAÇÃO DA FONTE HACK
A fonte **Hack** é bastante apropriada para exibir **código-fonte** de programas ou para uso no **terminal**.  
Sua instalação pelo repositório é simples:

```bash
sudo apt install -y fonts-hack-otf fonts-hack-ttf fonts-hack-web
```

Para conferir se a fonte foi realmente instalada, execute:

```bash
fc-list | grep "Hack"
```

Se aparecer algo como:

```
/usr/share/fonts/truetype/hack/Hack-Regular.ttf: Hack:style=Regular
/usr/share/fonts/truetype/hack/Hack-Bold.ttf: Hack:style=Bold
/usr/share/fonts/truetype/hack/Hack-BoldItalic.ttf: Hack:style=Bold Italic
/usr/share/fonts/truetype/hack/Hack-Italic.ttf: Hack:style=Italic
```

e outras variações semelhantes, significa que a fonte foi instalada com sucesso.

