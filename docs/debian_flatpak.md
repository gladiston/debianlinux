# ATIVE O SUPORTE A FLATPAK CENTRAL
Flatpak é um sistema de empacotamento e distribuição de aplicativos para Linux que permite instalar programas de forma isolada do restante do sistema, em um sandbox. Isso garante maior segurança e compatibilidade entre diferentes distribuições (como Debian, Fedora, Ubuntu, etc.), já que o aplicativo traz junto todas as suas dependências e Flathub é o repositório oficial e mais popular de aplicativos distribuídos via Flatpak.
Não são todas as distros que habilitam este repositório, especialmente o Ubuntu.   

Para habilitá-lo, siga as instruções abaixo, execute:

```bash
sudo apt install -y flatpak
```

Para o ambiente **GNOME**, instale também:
```bash
sudo apt install -y gnome-software-plugin-flatpak
```

Para o ambiente **KDE**, instale também:
```bash
sudo apt install -y plasma-discover-backend-flatpak
```

Esses pacotes adicionais melhoram a integração do Flatpak com os ambientes GNOME e KDE, permitindo que o **Flathub** apareça como fonte de aplicativos nas lojas gráficas.

Depois disso, adicione o repositório **Flathub**:
```bash
sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

Após a instalação, **reinicie sua sessão** (faça logout e login novamente) para que os repositórios do Flatpak funcionem corretamente.

### IMPORTANTE
Não é uma boa ideia instalar programas do **Flathub** que já são fornecidos pelos desenvolvedores originais por outros meios, por exemplo:
- **Google Chrome**: o próprio Google o fornece diretamente em seu site oficial.

Isso acontece porque os pacotes no Flathub são muitas vezes empacotados pela comunidade, e podem conter variações, plugins adicionais ou pequenas modificações.  
Essas versões podem carecer de revisão ou assinatura oficial, o que traz riscos potenciais.

Por outro lado, há desenvolvedores que publicam **eles mesmos** seus aplicativos no Flathub — e, nesses casos, são totalmente confiáveis, por exemplo:
- **Mozilla Firefox**: publicado pela própria Mozilla.  
- **Telegram**: publicado pela própria equipe do Telegram.

**Diferença entre pacotes tradicionais e do Flathub:**  
Os pacotes tradicionais são inspecionados e mantidos pela própria distribuição, sendo instalados em diretórios padrão do sistema.  
Já os pacotes obtidos via Flathub são empacotados pela comunidade e executados dentro de **contêineres**, o que traz uma camada extra de isolamento e segurança.  
Esses aplicativos geralmente armazenam seus dados em:
```
~/.var/app
```

Alguns programas do Flathub podem solicitar acesso ao seu `$HOME` ou a outros diretórios restritos — o sistema pedirá sua autorização nesses casos.  
Mesmo rodando sob contêiner, o isolamento **nem sempre é uma vantagem**, dependendo do tipo de aplicativo.  
Por exemplo, um navegador instalado via Flathub ainda pode ter acesso ao seu histórico e senhas salvas.  

Portanto, use com moderação e **verifique sempre se o desenvolvedor e o publicador são a mesma entidade**, especialmente em aplicativos de terceiros.

### Utilizando backups de aplicativos flat-pak
Se você deseja aproveitar instalações anteriores já configuradas anteriormente, basta ir até a maquina ou instalação antiga e ir na pasta:
```
~/.var/app
```
E ali copiar as pastas de aplicativos que você deseja, por exemplo:
```
[perfil da instalação antiga]/.var/app/org.remmina.Remmina
[perfil da instalação antiga]/.var/app/org.telegram.desktop
```
E colar as pastas acima em uma nova instalação em:  
```
~/.var/app/org.remmina.Remmina
~/.var/app/org.telegram.desktop
```
E quando instalar o Remmina e o Telegram pelo FlatPak, eles já estarão configurados.  

**Curiosidade**: Na pasta ~/.var/app, há pastas com nomes bem sugestivos que parecem ser, mas não são, **com.google.Chrome**, **com.google.ChromeDev** e **io.github.ungoogled_software.ungoogled_chromium** não estão relacionados ao navegador Google Chrome que instalamos, de fato, ainda é uma incognita porque estas pastas passam a existir assim que instalamos o flat-pak.

----

[Clique aqui para retornar a página principal](../README.md#ative-o-suporte-a-flatpak-central)
