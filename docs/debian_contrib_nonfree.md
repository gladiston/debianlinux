# Estendendo Repositórios Oficiais com `contrib`, `non-free` e `non-free-firmware`

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

Este comando garantirá a instalação de pacotes como `firmware-misc-nonfree`, `firmware-realtek`, e outros populares. Não é possivel saber quais firmwares este meta-pacote atende, mas caso precise instalar algum deles individualmente, veja essa relação:  
```
0xffff - flasher de firmware Fiasco aberto e livre
altos - firmware e utilitários Altus Metrum
dfu-programmer - atualização de firmware de dispositivo (DFU) baseado no programador USB para chips Atmel
dfu-util - programador USB de atualização de firmware de dispositivo (DFU)
esptool - cria e grava arquivos de firmware para chips ESP8266 e ESP32
firmware-linux-free - firmware binário para vários drivers no núcleo Linux
fwupd - daemon de atualização de firmware
fwupd-amd64-signed - ferramentas para gerenciar atualizações de firmware UEFI (assinado)
fxload - download de firmware para dispositivos EZ-USB
grub-firmware-qemu - imagem de firmware do GRUB para o QEMU
ovmf - firmware UEFI para máquinas virtuais x86 64-bit
tp-smapi-dkms - fonte de módulos de acesso de hardware/firmware do ThinkPad - versão dkms
ap51-flash - firmware flasher for ethernet connected routers and access points
arm-trusted-firmware-tools - "secure world" software for ARM SoCs - tools
asahi-fwextract - Asahi Linux firmware extractor
firmware-carl9170 - firmware for AR9170 USB wireless adapters
coreboot-utils - Coreboot firmware utilities
coreboot-utils-doc - Coreboot firmware utilities - documentation
crust-firmware - SCP firmware for Allwinner sunxi boards
depthcharge-tools - Tools for ChromeOS firmware/bootloader integration
dns323-firmware-tools - build and manipulate firmware images for a range of NAS devices
ovmf-ia32 - UEFI firmware for 32-bit x86 virtual machines
qemu-efi-aarch64 - UEFI firmware for 64-bit ARM virtual machines
qemu-efi-arm - UEFI firmware for 32-bit ARM virtual machines
qemu-efi-loongarch64 - UEFI firmware for LoongArch64 virtual machines
qemu-efi-riscv64 - UEFI firmware for RISCV64 virtual machines
fwupd-tests - Test suite for firmware update daemon
gnome-firmware - GTK front end for fwupd
hackrf-firmware - Firmware for HackRF devices
heimdall-flash - tool for flashing firmware on Samsung Galaxy S devices
heimdall-flash-frontend - tool for flashing firmware on Samsung Galaxy S devices - Qt GUI
idevicerestore - command-line application to restore firmware files to iOS devices
ipxe - PXE boot firmware
ipxe-qemu - PXE boot firmware - ROM images for qemu
ffado-tools - FFADO debugging and firmware tools
lm4flash - Command-line firmware flashing tool to communicate with the Stellaris Launchpad
mstflint - Mellanox firmware burning application and diagnostics tools
mstflint-dkms - kernel module for Nvidia (formly Mellanox) firmware buring tool
nmrpflash - firmware flash utility for Netgear devices
nxt-firmware - Improved firmware for LEGO Mindstorms NXT bricks
firmware-ath9k-htc - firmware for AR7010 and AR9271 USB wireless adapters
firm-phoenix-ware - firmware necessary for boxes issued by project PHOENIX
qflipper - Flipper Zero firmware updater
pxfw - Plextor firmware updater
sigrok-firmware-fx2lafw - Firmware for Cypress FX2(LP) based logic analyzers
opal-utils - OPAL firmware utilities
systemd-boot-efi-amd64-signed - Tools to manage UEFI firmware updates (signed)
ubertooth-firmware - Firmware for Ubertooth
ubertooth-firmware-source - Source code for the Ubertooth firmware
uefitool - UEFI firmware image viewer and editor
uefitool-cli - UEFI firmware image viewer and editor - CLI version
python3-virt-firmware - Tools for manipulating edk2 (ovmf/qemu-efi) firmware images
alsa-firmware-loaders - ALSA software loaders for specific hardware
b43-fwcutter - utility for extracting Broadcom 43xx firmware
firmware-b43-installer - firmware installer for the b43 driver
firmware-b43legacy-installer - firmware installer for the b43legacy driver
bladerf-firmware-fx3 - Nuand bladeRF firmware downloader (FX3)
hannah-foo2zjs - Graphical firmware downloader for the foo2zjs package
isight-firmware-tools - tools for dealing with Apple iSight firmware
amd64-microcode - Platform firmware and microcode for AMD CPUs and SoCs
atmel-firmware - Firmware for Atmel at76c50x wireless networking chips.
bluez-firmware - Firmware for Bluetooth devices
dahdi-firmware-nonfree - DAHDI non-free firmware
firmware-ast - Binary firmware for ASpeed Technologies graphics chips
firmware-amd-graphics - Binary firmware for AMD/ATI graphics and NPU chips
firmware-atheros - Binary firmware for Qualcomm Atheros wireless cards
firmware-bnx2 - Binary firmware for Broadcom NetXtremeII
firmware-bnx2x - Binary firmware for Broadcom NetXtreme II 10Gb
firmware-brcm80211 - Binary firmware for Broadcom/Cypress 802.11 wireless cards
firmware-cavium - Binary firmware for Cavium Ethernet adapters
firmware-cirrus - Binary firmware for Cirrus Logic audio chips
firmware-intel-graphics - Binary firmware for Intel iGPUs and IPUs
firmware-intel-misc - Binary firmware for miscellaneous Intel devices and chips
firmware-intel-sound - Binary firmware for Intel sound DSPs
firmware-ipw2x00 - Binary firmware for Intel Pro Wireless 2100, 2200 and 2915
firmware-ivtv - Binary firmware for iTVC15-family MPEG codecs (ivtv and pvrusb2 drivers)
firmware-iwlwifi - Binary firmware for Intel Wireless cards
firmware-libertas - Binary firmware for Marvell wireless cards
firmware-linux - Binary firmware for various drivers in the Linux kernel (metapackage)
firmware-linux-nonfree - Binary firmware for various drivers in the Linux kernel (metapackage)
firmware-marvell-prestera - Binary firmware for Marvell Prestera ASIC devices
firmware-mediatek - Binary firmware for MediaTek and Ralink chips for networking, SoCs and media
firmware-misc-nonfree - Binary firmware for various drivers in the Linux kernel
firmware-myricom - Binary firmware for Myri-10G Ethernet adapters
firmware-netronome - Binary firmware for Netronome network adapters
firmware-netxen - Binary firmware for QLogic Intelligent Ethernet (3000 and 3100 Series)
firmware-nvidia-graphics - Binary firmware for Nvidia GPU chips
firmware-qcom-media - Binary firmware for Qualcomm graphics/video (dummy package)
firmware-qcom-soc - Binary firmware for Qualcomm SoCs
firmware-qlogic - Binary firmware for QLogic HBAs
firmware-realtek - Binary firmware for Realtek network and audio chips
firmware-samsung - Binary firmware for Samsung MFC video codecs
firmware-siano - Binary firmware for Siano MDTV receivers
firmware-ti-connectivity - Binary firmware for TI Connectivity Wi-Fi and BT/FM/GPS adapters
firmware-sof-signed - Intel SOF firmware - signed
intel-microcode - Processor microcode firmware for Intel CPUs
live-task-non-free-firmware-pc - selection of oft-used non-free-firmware shipped on live systems
live-task-non-free-firmware-server - provides firmware for server network and storage devices
midisport-firmware - Firmware loader for M-Audio's MidiSport devices
firmware-nvidia-gsp - NVIDIA GSP firmware
firmware-nvidia-tesla-535-gsp - NVIDIA GSP firmware (Tesla 535 version)
raspi-firmware - Raspberry Pi family GPU firmware and bootloaders
firmware-zd1211 - binary firmware for the zd1211rw wireless driver
```
Mas lembre-se de que `sudo apt install firmware-linux-nonfree` já faria a instalação de alguns deles, por exemplo, se tentar executar:  
Se houver hardware uma placa de vídeo Radeon, então precisará instalar também:  

```bash
sudo apt install firmware-amd-graphics
```
Tomará a seguinte resposta:  
> firmware-amd-graphics já é a versão mais nova (20250410-2).  

Indicando que o mesmo já está instalado.  

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

[Clique aqui para retornar a página principal](../README.md#estendendo-reposit%C3%B3rios-oficiais-com-contrib-non-free-e-non-free-firmware)





