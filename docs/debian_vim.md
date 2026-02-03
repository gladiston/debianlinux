## EDITOR DE TEXTO VIM
O **Vim (Vi IMproved)** é um editor de texto poderoso e altamente configurável, baseado no clássico **Vi**, presente em praticamente todas as distribuições Unix e Linux.  
É amplamente utilizado por administradores de sistema e desenvolvedores por ser leve, rápido e disponível mesmo em ambientes sem interface gráfica.  

Ele oferece atalhos de teclado eficientes, realce de sintaxe e modos de operação distintos (comando, inserção e visualização), que tornam a edição ágil e precisa.  
Também pode ser expandido com plugins e temas, transformando-o em um ambiente completo para programação.  

Em algumas distribuições, o Vim não vem instalado por padrão, por isso vamos instalá-lo (caso já esteja instalado, o comando será apenas ignorado):

```bash
sudo apt install -y vim vim vim-doc  vim-scripts universal-ctags
```

Para confirmar a instalação e verificar a versão instalada:
```bash
vim --version
```

Algo que pode ser um pouco irritante ao usar o Vim é que ele também assume o controle do **mouse**.  
Assim, ao abrir o terminal dentro do KDE ou GNOME, os comandos de copiar/colar do terminal podem não funcionar, pois o Vim captura os cliques do mouse.

Se isso te incomoda, basta executar dentro do Vim o comando (tecle `:`):
```
set mouse=
```

Para tornar essa configuração permanente, edite (ou crie) o arquivo de configuração do Vim:
```bash
editor ~/.vimrc
```

E adicione a linha:
```
set mouse=
```

Salve e feche o arquivo (`Ctrl+O`, `Enter`, `Ctrl+X`).  
Pronto — agora o mouse não interferirá mais ao usar o Vim.

----

[Clique aqui para retornar a página principal](../README.md#editor-de-texto-vim)
