# Instalação de Ferramentas Essenciais para Compilação

Os pacotes a seguir são mandatórios para qualquer usuário que pretenda compilar programas, *drivers* (módulos de kernel) ou bibliotecas a partir do código-fonte no ambiente Linux. Eles são necessários para procedimentos subsequentes deste guia.

Execute os comandos a seguir para instalar o conjunto completo de ferramentas de desenvolvimento:

1.  **Ferramentas de Compilação Fundamentais (`build-essential`):**
    ```bash
    sudo apt install -y build-essential
    ```
2.  **Utilitários de Empacotamento Debian e Meta-dados:**
    ```bash
    sudo apt install -y dh-make exuberant-ctags dpkg-dev debhelper fakeroot
    ```
3.  **Utilitários Adicionais de Kernel e Segurança:**
    ```bash
    sudo apt install -y exuberant-ctags module-assistant dkms patch libssl-dev
    ```
4.  **Ferramentas de Desenvolvimento e Visualização:**
    ```bash
    sudo apt install -y libncurses-dev ack fontconfig imagemagick git meson sassc tree
    ```

Estes pacotes fornecem desde o compilador GNU (GCC) até utilitários de empacotamento (`dpkg-dev`, `fakeroot`) e dependências de bibliotecas de desenvolvimento (`libssl-dev`), preparando o sistema para qualquer tarefa de compilação.


----

[Clique aqui para retornar a página principal](../README.md#instalando-compactadoresdescompactadores-de-arquivos)
