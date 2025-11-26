# ESPELHAMENTO DE CELULAR (SCRCPY)

O **scrcpy** (Screen Copy) é uma aplicação de código aberto que permite espelhar e controlar dispositivos Android (celular ou tablet) em um desktop Linux, macOS ou Windows. O programa foca em **desempenho** e **baixa latência**, fornecendo controle total do dispositivo via teclado e mouse sem a necessidade de *root*.

Este tutorial demonstra a instalação e uso no Linux, focando em distribuições baseadas em **Debian** e **Ubuntu** (que utilizam o gerenciador de pacotes `apt`):

### Instalação Essencial

Para usar o scrcpy, você precisa primeiro instalar as ferramentas **ADB** (Android Debug Bridge) e, em seguida, o próprio scrcpy:

1.  **Instale as ferramentas ADB e o scrcpy:**
    ```bash
    sudo apt install -y android-tools-adb scrcpy
    ```

## Pré-requisitos e Uso

Antes de executar o scrcpy, você deve **ativar o modo de depuração USB** em seu celular ou tablet Android. Este modo é essencial para que o ADB e o scrcpy possam se comunicar com o dispositivo.

  * **Ativação da Depuração:**
    O processo geralmente envolve ir para "Configurações" \> "Sobre o telefone" e tocar várias vezes no "Número da build" para desbloquear o menu "Opções do desenvolvedor", onde a depuração USB pode ser ativada.

  * **Uso Básico:**
    Após conectar o celular via USB e ativar a depuração, execute no terminal:

    ```bash
    scrcpy
    ```

## Interface Gráfica (GUI)

Para usuários que preferem uma interface gráfica para simplificar as operações e configurações do scrcpy, existe uma GUI (Interface Gráfica do Usuário) disponível:

  * **Instalação da GUI (Guiscrcpy):**
    A interface **`guiscrcpy`** pode ser instalada diretamente pela loja de aplicativos ou, se disponível no seu repositório ou via Flatpak (verifique o nome exato do pacote na sua distribuição).
    ```bash
    guiscrcpy
    ```

O scrcpy estará agora instalado e pronto para espelhar seu celular no ambiente desktop.
