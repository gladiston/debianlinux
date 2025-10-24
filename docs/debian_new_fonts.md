## SUPORTE A NOVAS FONTES
A aparência das fontes influencia diretamente a legibilidade, o conforto visual e até mesmo a produtividade durante o uso do sistema. No Linux, especialmente em distribuições como Debian e Ubuntu, é possível personalizar facilmente o conjunto de fontes disponíveis — seja para o ambiente gráfico, terminais ou IDEs de programação.  
Este tutorial mostra como instalar e gerenciar fontes no sistema, incluindo opções populares entre desenvolvedores e usuários avançados. Também aborda a instalação de fontes de terceiros (como as da Microsoft) e alternativas modernas e livres que oferecem excelente qualidade visual e compatibilidade.  

Normalmente, os pacotes a seguir já vêm instalados, mas precisamos conferir, então execute:

```bash
sudo apt install -y fontconfig fontforge fonttools
```

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


## INSTALAÇÃO DE FONTE SIMILAR AO "Segoe UI"
O Linux não possui a fonte **Segoe UI**, idealizada pela Microsoft para o sistema Windows. Essa fonte é realmente muito bela para ser usada em interfaces gráficas (UI) de aplicativos. O Linux não tem essa fonte, ela pode ser copiada de uma instalação do Windows, mas isso infringiria leis de copyright. O Linux não tem ela, apesar disso, há fontes similares e igualmente elegantes que podemos instalar para configurar em nossos programas, instale:

```bash
sudo apt install -y fonts-inter fonts-noto
```

A fonte **fonts-inter** é bonita e versátil, ideal para uso geral, enquanto **fonts-noto-ui** se destaca por sua aparência moderna e legibilidade — uma excelente opção para ser usada na interface de aplicativos.



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
~/.local/share/fonts/ttf/Hack-BoldItalic.ttf: Hack:style=Bold Italic
```

e outras variações semelhantes, significa que a fonte foi instalada com sucesso.

