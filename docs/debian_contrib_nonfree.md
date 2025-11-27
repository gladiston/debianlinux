# ADICIONANDO OS REPOSITORIOS 'CONTRIB' e 'NON-FREE' NO DEBIAN (SOMENTE PARA DEBIAN)
O Debian 13 é bastante limitado com respeito a programas e módulos(drivers), resumidamente, ele não incluir nada que não tenha licença opensource e seja copy-left e isso pode ser ótimo para servidores, mas se pretende usar um Debian como desktop, essa restrição irá nos limitar muito. Para remover esta restrição ou limitação precisaremos incluir os repositórios `contrub` e `non-free` então siga as instruções abaixo:  

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

----

[Clique aqui para retornar a página principal](../README.md#adicionando-os-repositorios-contrib-e-non-free-no-debian-somente-para-debian)





