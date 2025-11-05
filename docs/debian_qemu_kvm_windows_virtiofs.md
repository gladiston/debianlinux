# VIRT-MANAGER - COMPARTILHANDO ARQUIVOS VIA Virtio-FS+WinSFP
Para compartilhar arquivos entre o sistema hospedeiro e convidado, voce pode usar o virtiofs+WinSFP. Esse é o método mais performático que existe.  
Siga as instruções abaixo:  

Desligue a VM. 
Crie um novo *pool*
Vá até as configurações dela, adicione um novo hardware e escolha um novo hardware **Sistema de arquivos(FileSystem)**, tenha certeza de:  
* **Driver**: virtiofs  
* **Source path**: escolha uma pasta de seu sistema como **/home/gsantana/Downloads**, ele não aceitará se **Downloads** não estiver dentro de um **pool**, assim você precirá nesta tela, criar um pool chamado de **home** apontando para a pasta onde a **Download&** está, isto é, **/home/gsantana**, só assim, conseguirá selecionar a pasta de **Downloads** e funcionar. **ADVERTÊNCIA**: possivel selecionar Downloads sem criar um pool, mas não funcionará.       
* **Target path**: escolha um nome para este compartilhamento como **downloads**   
Se quiser impedir da estação windows escrever nesta pasta você pode marcar a opão **Exportar sistema de arquivo como montagem somente leitura**.

Tenha certeza de ter instalado dentro da VM WIndows o programa `WinFsp`.  
Depois de iniciar a VM, acesse a a página WinSFP no link abaixo:  
[https://github.com/winfsp/winfsp/releases](https://github.com/winfsp/winfsp/releases)  
E baixe a ultima versão disponivel:   
![página WinSFP](../img/debian_qemu_kvm_windows59.png)  

Execute `services.msc` e veja se os serviços estão habilitados:  
**VirtIO-FS Service**  
![VirtIO-FS Service](../img/debian_qemu_kvm_windows60.png)   

**WinFsp.Launcher**  
![WinFsp.Launcher Service](../img/debian_qemu_kvm_windows61.png)   

Sem O `WinFsp`, o serviço `Virtio-FS` não inicia, e portanto, a unidade que representa o `Source Path` escolhido não aparecerá. Voce até pode chamar o `services.msc` no menu do Windows e verá que este serviço não inicializa sem o `WinFsp` instalado.  
Isso acontece porque o programa `WinFsp` é convocado assim que o serviço `Virtio-FS`  é iniciado, e então ele lê o `Source Path` e cria então a unidade como Z:, mas temos um problema, ele lê o primeiro `Source Path`, mas não executa os demais se existirem. Caso crie um outro `Source Path` com o nome de `docs` apontando para algum lugar do hospedeiro, essa unidade não será mostrada. Isso corre porque dentro do Windows, o `WinFsp` só consegue executar uma instância de cada vez quando o serviço `Virtio-FS` é iniciado. Assim, caso precise de mais unidades, temos duas soluções alternativas:  

### OPÇÃO 1 - CONSOLIDE TUDO NUM ÚNICO PONTO DE ENTRADA
Consolide todas as pastas necessarias numa unica pasta maior, poderá tentar usar links simbolicos usando a opção de `bind`:
```
mkdir /home/gsantana/share_all
mkdir /home/gsantana/share_all/downloads
mkdir /home/gsantana/share_all/docs

# substitua symlinks por bind mounts:
sudo mount --bind /home/gsantana/work /home/gsantana/share_all/downloads
sudo mount --bind /home/gsantana/docs /home/gsantana/share_all/docs
```
E agora voce exporta o source-path como `/home/gsantana/share_all` e obterá todas as pastas dentro de uma unica que poderá ser usada.  

### OPÇÃO 2 - MAPEIE UNIDADES POR DENTRO DO WINDOWS
Dentro do windows, execute outra instancia manualmente com o comando:  
```
   "C:\Program Files\Virtio-Win\VioFS\virtiofs.exe" docs X:
```
Isso criará a letra X: para Target Path definido como **docs**.  Você terá de criar um `.bat` para mapear cada letra de drive para cada surce-path e depois colocá-lo na auto inicialização do seu perfil, assim não precisará executar estes comandos todas as vezes.


Assim, que estes serviços forem iniciados, olhe novamente para o explorer e notará que as pastas que foram exportadas, em nosso exemplo apenas a pasta `Downloads` serão reconhecidas como unidades:  

![Novas iunidades no Windows](../img/debian_qemu_kvm_windows62.png)   

**DICA:** Não coloque seu home inteiro disponivel para VMs, crie **pools** separadas para as pastas que for usar, eu uso uma subpasta **projetos** onde tenho as pastas que minha VM terá acesso. Além de mais seguro, interrompe programas erroneamente chamados de **telemetria** que acompanham produtos comerciais, mas que não tem diferença para **spywares**.  
