# INSTALANDO O CURSOR
O **Cursor** é um editor de código inovador, construído sobre a base do VS Code, mas redesenhado para colocar a **Inteligência Artificial** no centro do fluxo de trabalho. 

Ele combina a experiência familiar do VS Code com recursos avançados de IA, como **edição preditiva**, **chat contextual com o projeto** e a capacidade de **gerar código** instantaneamente, suportando inclusive a importação de todas as suas extensões habituais.

O Cursor disponibiliza um arquivo .deb para Debian e derivados. Visite a página oficial:  

[Site oficial para download do Cursor](https://www.cursor.com/)  
E faça o download da versão Linux (.deb):  
![Tela de Download](../img/debian_cursor1.png)  

Depois, apenas dê um duplo clique no arquivo e siga as instruções na tela. Ao final do processo, o Cursor estará instalado e disponível no seu menu de aplicativos.

## EXTENSÕES PARA SQL E GERENCIAMENTO DE BANCOS DE DADOS
Como o Cursor utiliza a mesma base de extensões, execute no terminal:  

```bash
cursor --install-extension mtxr.sqltools \
       --install-extension mtxr.sqltools-driver-mysql \
       --install-extension mtxr.sqltools-driver-pg \
       --install-extension mtxr.sqltools-driver-sqlite \
       --install-extension mtxr.sqltools-driver-firebird \
       --install-extension adpyke.vscode-sql-formatter \
       --install-extension cweijan.vscode-database-client2
