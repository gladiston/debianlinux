# Suporte a arquivos comprimidos

Por padrão, o sistema operacional possui suporte limitado a formatos de compressão. É fundamental instalar os pacotes abaixo para garantir a manipulação de formatos amplamente utilizados, como `.zip`, `.rar`, `.7z`, além de vários utilitários de compressão essenciais para o ecossistema Linux.

Execute o comando a seguir para instalar todos os *utilities* necessários:

```bash
sudo apt install -y tar zip unzip p7zip-full p7zip-rar rar unrar lzip lzma xz-utils bzip2 gzip squashfs-tools cabextract
```

A tabela a seguir detalha a função primária de cada pacote instalado:

| Pacote | Função / Formato |
| :--- | :--- |
| `tar` | Criação e extração de arquivos `.tar` e `.tar.gz`. |
| `zip`, `unzip` | Manipulação de arquivos `.zip`. |
| `p7zip-full` | Suporte completo ao formato `.7z` (7-Zip). |
| `p7zip-rar`, `rar`, `unrar` | Suporte para compressão e extração de arquivos `.rar`. |
| `lzip`, `lzma`, `xz-utils`, `bzip2`, `gzip` | Utilitários de compressão amplamente utilizados na distribuição de pacotes Linux. |
| `squashfs-tools` | Criação e extração de arquivos `.squashfs`, comum em pacotes *Snap* e sistemas de arquivos de leitura. |
| `cabextract` | Utilitário para extração de arquivos `.cab` (Microsoft Cabinet). |

## GUI para compactar/descompactar
Ambientes de desktop como KDE, GNOME e outros tem ferramentas próprias para compactar e descompactar arquivos. Essas ferramentas são fáceis e intuitivas, no entando elas são um wrapper para as ferramentas de backend que instalamos acima, por exemplo, o Ark (o gerenciador de arquivos do KDE) não "lê" o arquivo .7z nativamente. Ele faz um `7z l` em background, captura o output de texto, faz o parsing (processamento do texto) e então preenche a lista na interface gráfica e no GNOME, o File-Roller faz a mesma coisa. Nestes ambientes gráficos, cada vez que você clica em uma pasta compactada, ele pode estar reexecutando comandos para atualizar a visão. Então precisamos de um programa gráfico para substituir estes programas e que faça essas operações nativamente como WinRAR ou 7-Zip faz no Windows, e é aqui que recomendo a instalação do `PeaZip`, ele é encontrado na loja Flathub via flatpak, então procure-o por lá. Caso queira instalar via terminal:  
```bash
flatpak install flathub io.github.peazip.PeaZip
```
Caso queira instalar uma versão .deb, [poderá fazer o download aqui](https://peazip.github.io/peazip-linux.html).  
A diferença entre o PeaZip com o Ark(KDE) ou File Roller(GNOME) é da agua para o vinho em velocidade. Me faz pensar que KDE e GNOME deveriam descontinuar suas ferramentas de compactação e descompactação. Sério, é muita diferença de tempo mesmo! Lembre-se de que as ferramentas de terminal são muito rápidas, o problema mesmo é usá-las como frontend do gerenciador de arquivos no KDE/GNOME.  

## Cópia e sincronização de arquivos
Fazer backups, transferencia de arquivos, sincronização e outros tipos de operação é uma tarefa que seria complicada em o uso do `rsync`. Vamos instalá0lo:  
```bash
sudo apt install -y rsync
```
Um outro motivo para instalá-lo é que o `rsync` é backend para muitos outros programas. Se você não instalá-lo, é muito provavel que o instalará mais tarde como dependencia de outro.  

---
## INTEGRAÇÃO COM O SISTEMA DE ARQUIVOS DE SEU SISTEMA
Cuidado ao usar compactar/descompactar pastas usando o gerenciador de arquivos, tanto o GNOME, quanto o KDE usam os utilitários acima para compactar ou descompactar e quando se trata de pastas inteiras não há problemas. Mas quando o objetivo é observar seu conteúdo de um arquivo compactado e/ou extrair apenas conteúdo parcial então vem um problema, neste caso, eles usam os utilitarios acima para listar o conteúdo inteiro para dentro de um arquivo .txt e depois fazer o que chamamos de scrap que é ler o .txt e de acordo com o conteúdo então construir a exibição na tela e isso é MUITO LENTO e fica mais lento ainda quando tal arquivo está numa rede local ou num ponto de montagem lento.   
No Windows, o WinRAR, 7Zip e outros fazem isso nativamente, geralmente arquivos compactados tem um HEADER que tornam a listagem do conteúdo praticamente instantaneo sem precisar varrer o arquivo inteiro. Infelizmente, no Linux, só há um programa que faz acesso a arquivos compactados de forma nativa: PEAZIP, porém nos repositórios do Debian ou Ubuntu, o mesmo não está disponível, a única forma de instalá-lo é por meio de flat-pak.  



----

[Clique aqui para retornar a página principal](../README.md#instalando-compactadoresdescompactadores-de-arquivos)
