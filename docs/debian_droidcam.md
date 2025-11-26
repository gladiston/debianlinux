# USANDO O CELULAR COMO WEBCAM (DROIDCAM)

O **DroidCam** é uma solução popular que permite transformar seu celular ou tablet Android em uma **webcam de alta qualidade** para seu computador Linux, utilizando a conexão **USB** ou **Wi-Fi**. Isso é útil para quem precisa de melhor qualidade de vídeo do que as webcams embutidas ou para quem não tem uma webcam dedicada.

Este tutorial detalha o processo de instalação e configuração do DroidCam no Linux, focado em distribuições baseadas em **Debian/Ubuntu** que usam o gerenciador de pacotes `apt`.

-----

## Instalação dos Componentes no Linux

Siga os comandos abaixo no terminal para instalar as ferramentas necessárias e o cliente DroidCam:

1.  **Instale as ferramentas ADB (Android Debug Bridge):**

    ```bash
    sudo apt install -y android-tools-adb
    ```

2.  **Baixe o pacote e prepare a instalação do Cliente DroidCam:**

    ```bash
    cd /tmp
    wget https://files.dev47apps.net/linux/droidcam_latest.zip
    unzip droidcam_latest.zip -d droidcam
    cd droidcam
    ```

3.  **Instale o cliente principal:**

    ```bash
    sudo ./install-client
    ```

    *Este comando instala o software base necessário para a comunicação.*

4.  **Acione a parte de vídeo (criação do módulo V4L2):**

    ```bash
    sudo ./install-video
    ```

    *Este comando instala o módulo necessário para que o sistema reconheça o DroidCam como uma webcam de vídeo padrão (V4L2).*

5.  **Acione a parte de áudio (opcional):**

    ```bash
    sudo ./install-sound
    ```

    *Este comando instala os módulos para que o áudio do celular também possa ser usado como microfone no desktop.*

-----

## Uso Básico e Configuração

Após a instalação, é necessário configurar o aplicativo no celular e executar o cliente no desktop:

1.  **No Celular Android:** Instale o aplicativo **DroidCam** (disponível na Google Play Store).
2.  **No Terminal:** Execute o comando `droidcam` ou `droidcam-cli`.
3.  **Configuração:** O programa apresentará opções para usar o celular como webcam **plugado na USB** (via ADB) ou **via Wi-Fi** (usando o endereço IP local do celular).

-----

## Criação de Atalho no GNOME (Opcional)

Para facilitar o acesso ao programa sem usar o terminal, você pode criar um atalho de aplicativo (`.desktop`) no ambiente GNOME (ou outros ambientes compatíveis):

1.  **Crie e abra o arquivo de atalho:**

    ```bash
    editor ~/.local/share/applications/droidcam.desktop
    ```

    *Substitua `editor` pelo seu editor de texto preferido (ex: `nano`, `gedit`, `vim`).*

2.  **Cole o seguinte conteúdo no arquivo:**

    ```ini
    [Desktop Entry]
    Version=1.0
    Type=Application
    Terminal=false
    Name=DroidCam
    Exec=droidcam
    Comment=Use seu celular Android como uma Webcam wireless/USB ou como IP Cam!
    Icon=droidcam
    Categories=GNOME;GTK;Video;
    Name[it]=droidcam
    ```

3.  **Salve e feche o arquivo.** A partir de agora, você encontrará o ícone do **DroidCam** no menu de aplicativos do seu sistema.

---

Para retornar a página inicial, [clique aqui](../README.md)


