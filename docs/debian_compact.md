# Suporte Abrangente a Arquivos Comprimidos

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

-----

## Instalação de Ferramentas Essenciais para Compilação

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

[Clique aqui para retornar a página principal](../README.md#xxxxxx)
