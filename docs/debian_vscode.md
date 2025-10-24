# INSTALANDO O VSCODE
O **Visual Studio Code (VS Code)** é uma IDE leve, poderosa e multiplataforma desenvolvida pela Microsoft.  
Ele combina a simplicidade de um editor de texto com recursos avançados de programação, como **autocompletar inteligente (IntelliSense)**, **depuração integrada**, **controle de versão com Git** e uma ampla variedade de extensões para praticamente qualquer linguagem.  

O VS Code não está nos repositórios padrão de nenhuma distribuição, e por isso vamos precisar habilitar o repositório oficial do vs code.  

## INCLUINDO O REPOSITÓRIO DO VSCODE
O **Visual Studio Code (VS Code)** é uma IDE leve, poderosa e multiplataforma desenvolvida pela Microsoft.  
Não iremos instalá-lo agora — apenas incluir o repositório oficial.  
Vamos adicionar a chave pública da Microsoft:  
```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | \
  sudo tee /usr/share/keyrings/microsoft.gpg > /dev/null
```
Agora, vamos incluir o repositório:  
```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | \
  sudo tee /etc/apt/sources.list.d/vscode.list
```
Agora, atualizamos os nossos repositórios:  
```
sudo apt update -y
```

Pronto, o repositório do **VS Code** está configurado corretamente.   


## INCLUINDO O REPOSITÓRIO DO VSCODE
Agora que temos o repositório do vscode instalado, podemos prosseguir, execute:  
```bash
sudo apt install -y code
```
A seguir vamos a algumas extensões sugeridas.  

## NODE.JS  
É preciso ter a linguagem previamente instalada para prosseguir com as instruções abaixo:  
```
code --install-extension waderyan.nodejs-extension-pack \
     --install-extension dbaeumer.vscode-eslint \
     --install-extension christian-kohler.npm-intellisense \
     --install-extension christian-kohler.path-intellisense \
     --install-extension ms-vscode.node-debug2
```
## JAVA
É preciso ter a linguagem previamente instalada para prosseguir com as instruções abaixo:  
```
code --install-extension vscjava.vscode-java-pack \
     --install-extension redhat.java \
     --install-extension vscjava.vscode-java-debug \
     --install-extension vscjava.vscode-java-test \
     --install-extension vscjava.vscode-maven
```

## FREE PASCAL E DELPHI
É preciso ter a linguagem previamente instalada para prosseguir com as instruções abaixo, isso também inclui o Lazarus, IDE para programação usando freepascal:  
```
sudo apt install -y global exuberant-ctags python3-pygments
sudo apt install -y libqt5pas-dev
sudo apt install -y libqt5pas1
```
Depois:  
```
$ code --install-extension Wosi.omnipascal \
       --install-extension alefragnani.pascal \
       --install-extension alefragnani.pascal-formatter
```
> 💡 Dica: no Debian, Ubuntu e derivamos, é mandatório instalar o FreePascal Compiler (fpc) usufruir dessas extensões. Cada uma dessas extensões carecem de configuração, vejá-os:  
> [Instruções para Omini Pascal](https://www.omnipascal.com)     
> [Instruções para a Linguagem Pascal](https://github.com/alefragnani/vscode-language-pascal)     
> [Instruções para Pascal Formatter](https://github.com/alefragnani/vscode-pascal-formatter)     

Caso queira uma outra IDE (e mais completa) para FreePascal, recomendo o [Lazarus](https://lazarus-ide.org).  


## EXTENSÕES PARA HTML, CSS E JAVASCRIPT  
```
code --install-extension ecmel.vscode-html-css \
     --install-extension esbenp.prettier-vscode \
     --install-extension ritwickdey.LiveServer \
     --install-extension formulahendry.auto-rename-tag \
     --install-extension xabikos.JavaScriptSnippets
```

## EXTENSÕES PARA PYTHON   
É preciso ter a linguagem previamente instalada para prosseguir com as instruções abaixo:  
```
code --install-extension ms-python.python \
     --install-extension ms-python.vscode-pylance \
     --install-extension ms-toolsai.jupyter \
     --install-extension ms-python.debugpy
```

## EXTENSÕES PARA SQL E GERENCIAMENTO DE BANCOS DE DADOS
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

## EXTENSÕES PARA BASH SCRIPT E TERMINAL
```
code --install-extension mads-hartmann.bash-ide-vscode \
     --install-extension timonwong.shellcheck \
     --install-extension foxundermoon.shell-format \
     --install-extension formulahendry.code-runner \
     --install-extension jeff-hykin.better-shellscript-syntax \
     --install-extension formulahendry.terminal
```

## CONFIGURAÇÕES RECOMENDADAS  
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

