# INSTALANDO IDE DE PROGRAMAÇÃO PASCAL COM A IDE LAZARUS
O Lazarus é uma IDE completa para desenvolvimento em Object Pascal, baseada no compilador Free Pascal (FPC).
Ela é multiplataforma (Linux, Windows e macOS) e oferece uma experiência muito próxima do antigo Delphi, com formulários visuais, depurador integrado e suporte a componentes visuais compatíveis com diversos sistemas gráficos (GTK, Qt, Win32, Cocoa etc.).   

Este tutorial mostra como instalar o Lazarus de forma limpa e moderna usando o fpcupdeluxe, uma ferramenta que automatiza a obtenção do código-fonte mais recente diretamente do repositório oficial e compila tudo localmente, sem interferir no sistema. O método é especialmente útil para quem deseja ter múltiplas versões do Lazarus, fazer cross-compile ou manter um ambiente isolado por usuário.   

Antes de começar, é importante garantir que o sistema esteja atualizado e que todas as dependências necessárias para o ambiente gráfico (Qt ou GTK) estejam instaladas. Nos passos a seguir, você verá como preparar o ambiente e instalar o Lazarus sem precisar de permissões administrativas (homeuser install), ideal para desenvolvedores que preferem manter o sistema limpo e facilmente reversível.   

Primeiro, atualize o repositorio
```bash
sudo apt update -y
```

As ferramentas básicas de build precisam ser instaladas:
```bash
sudo apt install -y build-essential pkg-config
sudo apt install -y global exuberant-ctags python3-pygments
```

Se você usa KDE ou ambientes baseados em QT, também precisará:
```bash
sudo apt install -y libqt5pas-dev libqt5pas1 qtbase5-dev qtbase5-dev-tools libqt5x11extras5-dev
```

Se você usa GNOME ou ambientes baseados em GTK, também precisará:
```bash
# GTK2 (legado, mas amplamente usado pelo Lazarus)
sudo apt install -y libgtk2.0-dev libcanberra-gtk-module 
# ou GTK3
sudo apt install -y libgtk-3-dev libcanberra-gtk3-module
```

## FPCUPDELUXE
Agora que o ambiente já está preparado, podemos iniciar a instalação do Lazarus usando o fpcupdeluxe, um excelente instalador que automatiza todo o processo.
Ele baixa o código-fonte diretamente do repositório oficial (gitlab.net) e compila tanto o Free Pascal Compiler (FPC) quanto o Lazarus IDE, permitindo personalizações avançadas — incluindo suporte a cross-compile para outras plataformas.   

As orientações a seguir foram baseadas na documentação oficial:   
[Wiki da página do fpcupdeluxe](https://wiki.lazarus.freepascal.org/fpcupdeluxe)  

>**Dica**: Com o tempo, os pacotes do sistema e o Lazarus evoluem. Caso este tutorial fique desatualizado, sempre consulte a wiki acima para verificar eventuais mudanças nas dependências ou nas opções do instalador.


Para começar, acesse o repositório oficial e baixe a versão mais recente para sua plataforma:   
[Download da versão recente do fpcupdeluxe](https://github.com/newpascal/fpcupdeluxe/releases/latest)   

Veja o exemplo que serve para GNOME, mas tem para KDE/QT também:    
![Baixando o fpcupdeluxe](../img/instalacao_linux_fpcupdeluge1.png)  

Digamos que tenha baixado a versão para GNOME, agora vamos executá-lo:   
```bash
chmod +x fpcupdeluxe-x86_64-linux
./fpcupdeluxe-x86_64-linux
```
Não é necessário usar sudo, pois esta será uma instalação homeuser, ou seja, ficará isolada no diretório do seu usuário e não exigirá privilégios administrativos.    
Na tela inicial, selecione:   
* FPC Version: fixes   
* Lazarus Version: fixes   
Em seguida, clique em `Setup+` para abrir as opções adicionais.    
[Clique em Setup++ para ajustar alguns parametros](../img/instalacao_linux_fpcupdeluge2.png)

Depois, ajuste as configurações conforme a plataforma que pretende compilar:   
[Marque as opções que achar apropriado](../img/instalacao_linux_instalador1.png)     

Marque apenas o que for realmente necessário. A opção `Use System FPC for Lazarus` deve ser usada somente se você já tiver o compilador Free Pascal instalado pelos repositórios — isso economiza tempo e evita downloads desnecessários, geralmente deixo desmarcado e o próprio instalador fará um download e instalação local do **fpc**.  

Você pode fazer algumas outras marcações como definir a plataforma em que pretende compilar:    
[Algumas outras opções também podem ser interessantes](../img/instalacao_linux_fpcupdeluge3.png)  

Algumas outras opções também podem ser interessantes, explore as opções do ambiente.  

Se você é fã das janelas integradas ao estilo "Delphi" talvez queira ligar essa opção **Docked Lazarus IDE**:    
[Habilitando janelas docadas na IDE](instalacao_linux_fpcupdeluge5.png)   
Pessoalmente, eu gosto de instalar sem a docagem completa, mas depois de instalado, ir na `IDE>Packages>Install Packages` e então instalar as packages `AnchorDocking` e `AnchorDockingDsgn`, o motivo é tudo fica docado com exceção do formulário de design e isso me permitequando preciso desenhar no formulário ter uma área bem maior do que fosse docado na IDE.  

Após finalizar os ajustes, clique em `OK` e depois em `Install/Update FPC+Laz` para iniciar a instalação:     
[Prossiga com a instalação](instalacao_linux_fpcupdeluge4.png)     


**Atenção**: o processo pode demorar bastante, pois o fpcupdeluxe compila tudo a partir do código-fonte.  
Ao concluir com sucesso, será criada uma entrada no menu do sistema e um arquivo de inicialização chamado Lazarus_fpcupdeluxe no seu diretório home.  
Para iniciar o Lazarus, use esse script ou os atalhos criados automaticamente — executar o binário diretamente não funcionará corretamente.   


### POSSÍVEIS ERROS E SOLUÇÕES
Durante a compilação pelo fpcupdeluxe, é comum aparecerem alguns erros de dependência.    
A seguir estão os mais frequentes e suas respectivas soluções.   
  
>Erro: cannot find -lX11

 
Embora a maioria das distros estejam migrando para Wayland, alguns programas ou bibliotecas podem ter depêndencias das libs do Xorg, então **É POSSIVEL OU NÃO** que precisemos de certas libs, caso a instalação do **fpcupdeluxe** falhe mostrando a mensagem acima, provavelmente você precisará também:    
```bash
sudo apt install -y libx11-dev libxext-dev libxtst-dev libxi-dev libxrandr-dev libxinerama-dev libxrender-dev libxt-dev
```
Mas não se apresse em instalar as libs acima primeiro, veja se **fpcupdeluxe** falha ao compilar a IDE e daí sim, você instala elas.   

## VSCODE 
O **Visual Studio Code (VS Code)** é uma IDE leve, poderosa e multiplataforma desenvolvida pela Microsoft.  
Ele combina a simplicidade de um editor de texto com recursos avançados de programação, como **autocompletar inteligente (IntelliSense)**, **depuração integrada**, **controle de versão com Git** e uma ampla variedade de extensões para praticamente qualquer linguagem.  
Uma vez que tenhamos o compilador em nosso sistema e vscode instalado, agora precisamos apenas de seus plugins:    

```
$ code --install-extension Wosi.omnipascal \
       --install-extension alefragnani.pascal \
       --install-extension alefragnani.pascal-formatter
```
> 💡 Dica: no Debian, Ubuntu e derivamos, é mandatório instalar o FreePascal Compiler (fpc) usufruir dessas extensões. Cada uma dessas extensões carecem de configuração, vejá-os:  
> [Instruções para Omini Pascal](https://www.omnipascal.com)     
> [Instruções para a Linguagem Pascal](https://github.com/alefragnani/vscode-language-pascal)     
> [Instruções para Pascal Formatter](https://github.com/alefragnani/vscode-pascal-formatter)     

Caso queira uma outra IDE (e mais completa) para FreePascal, recomendo o [Lazarus](https://lazarus-ide.org).  
