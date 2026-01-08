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

----

[Clique aqui para retornar a página principal](../README.md#instalando-compactadoresdescompactadores-de-arquivos)
