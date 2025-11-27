# Guia Debian 13: Estendendo Repositórios Oficiais com `contrib`, `non-free` e `non-free-firmware`

O projeto Debian mantém um **compromisso rigoroso** com os princípios do Software Livre, conforme estabelecido pelas **Diretrizes Debian de Software Livre (DFSG)**. Por padrão, o repositório principal do sistema (**`main`**) inclui apenas pacotes que cumprem integralmente esses critérios.

Essa adesão estrita é fundamental para a integridade, segurança e estabilidade do Debian, tornando-o a escolha ideal para ambientes de **servidores** e sistemas onde a liberdade de código é a prioridade máxima.

No entanto, ao configurar o **Debian 13** para uso como **Desktop** ou em hardware moderno, essa restrição pode levar à falta de componentes críticos, como *drivers* ou *firmware* proprietários.

## Componentes do Repositório

Para garantir o **suporte completo ao hardware** e aumentar a disponibilidade de software, é necessário garantir a inclusão dos repositórios complementares. A distinção entre eles é crucial para entender a filosofia do Debian:

| Repositório | Conformidade DFSG | Conteúdo e Propósito |
| :--- | :--- | :--- |
| **`main`** | **Sim** | O repositório central. Contém apenas pacotes que aderem integralmente às DFSG. |
| **`contrib`** | **Sim** | Contém Software Livre, mas que depende de pacotes presentes em `non-free` ou `non-free-firmware` para ser compilado ou executado. |
| **`non-free`** | **Não** | Contém pacotes que **não são Software Livre** e que não se qualificam como *firmware* (Ex: certos codecs e fontes proprietárias, ou drivers legados). |
| **`non-free-firmware`** | **Não** | Contém **firmware proprietário/não-livre** (drivers binários para Wi-Fi, GPUs e outros periféricos). **Incluído por padrão no instalador** desde o Debian 12 e mantido no Debian 13. |

## Procedimento para Adicionar os Repositórios

Para ativar todos os componentes de software, incluindo `contrib`, `non-free` e `non-free-firmware`, é preciso editar o arquivo de configuração de repositórios do APT.

### 1\. Modificar o Arquivo `sources.list`

Abra o arquivo de configuração de repositórios com privilégios de `root`:

```bash
sudo editor /etc/apt/sources.list
```

Localize as linhas de repositório do Debian (referentes a `trixie` ou `stable`) e modifique-as para incluir os componentes **`contrib`**, **`non-free`** e **`non-free-firmware`** no final de cada linha.

**Exemplo da Modificação (adicionando todos os componentes):**

```
deb http://deb.debian.org/debian/ trixie main contrib non-free non-free-firmware
deb http://deb.debian.org/debian/ trixie-updates main contrib non-free non-free-firmware
deb http://security.debian.org/debian-security trixie/updates main contrib non-free non-free-firmware
```

Salve o arquivo (`Ctrl+O`, `Enter`) e saia (`Ctrl+X`).

### 2\. Atualizar a Lista de Pacotes

Para que o APT reconheça os novos repositórios e seus pacotes, é necessário executar o comando de atualização:

```bash
sudo apt update
```

## Instalação e Verificação do Software Non-Free

Com os repositórios ativados, o próximo passo é garantir que o *firmware* essencial para seu hardware seja instalado.

### 1\. Instalação do Pacote `firmware-linux-nonfree`

Execute o seguinte comando para instalar o meta-pacote principal de *firmware* não-livre, que abrange uma ampla gama de dispositivos (gráficos, rede, etc.):

```bash
sudo apt install firmware-linux-nonfree
```

Este comando garantirá a instalação de pacotes como `firmware-misc-nonfree`, `firmware-realtek`, e outros necessários.

### 2\. Verificação de Módulos de Kernel

Após a instalação, é possível verificar se os novos módulos e *firmware* estão sendo reconhecidos pelo kernel.

Liste todos os módulos carregados no sistema:

```bash
lsmod
```

Inspecione os logs do sistema (`dmesg`) para verificar se o kernel está carregando o *firmware* sem erros ou se ainda reporta arquivos faltando:

```bash
sudo dmesg | grep -i firmware
```

Se o *firmware* tiver sido instalado corretamente, o *output* mostrará o kernel carregando os arquivos.

### 3\. Reinicialização do Sistema

Para garantir que todos os drivers e *firmware* sejam inicializados corretamente pelo kernel e aplicados a dispositivos de baixo nível (especialmente placas de vídeo e adaptadores Wi-Fi/rede), uma reinicialização é altamente recomendada:

```bash
sudo reboot
```

Após o reinício, você terá total funcionalidade do seu hardware de desktop, **removendo a limitação** imposta pelo repositório `main` puro.

-----

[Clique aqui para retornar a página principal](../README.md#adicionando-os-repositorios-contrib-e-non-free-no-debian-somente-para-debian)





