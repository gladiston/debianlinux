# VIRTUALIZAÇÃO NATIVA QEMU+KVM
O Linux é capaz de criar máquinas virtuais e ele mesmo ser o hypervisor. Será um servidor de virtualização nivel 1, o mais rápido possivel, no entanto com algumas ausencia de recursos que facilitam a configuração que existem no VirtualBox e VMWare, por exemplo, criar redes virtuais com vários tipos de topologias,  clipboard e transferencia de arquivos entre host e anfitrião e outras coisas.  

### Vamos fazer o backup?
Antes de prosseguirmos, que tal fazer o backup da sua configuração de rede original?  Isso é importante porque assim que instalar o `libvirt` ele vai criar uma interface NAT para virtualizar e isso vai mudar o ambiente inicial, se quiser saber como fazer isso, siga as instruções no link a seguir:  
[Fazendo um backup das configurações de rede](debian_backup_restore_network.md)  


### Vamos instalar os pacotes principais:  
```bash
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients
sudo apt install -y libguestfs-tools
```
|Pacote|Explicação|
|:--|:--|
|libvirt-daemon-system|Configura o daemon libvirtd para gerenciar VMs via KVM.|  
|libvirt-clients|Ferramentas CLI (, virt-install, etc.).|  
|dnsmasq-bas|Fornece DHCP/NAT automáticos para redes virtuais.|  
|ovmf|Permite boot UEFI em VMs (necessário para Windows modernos).| 
|isc-dhcp-client|Utilitário para cliente DHCP|

### Permitir uso sem root
Adicione seu usuário ao grupo libvirt (e kvm):
Observe se existe  o grupo 'kvm', ele não é necessário em algumas distros, execute:
```bash
getent group kvm
```
Se ele existir, aparecerá algo como:
> kvm:x:993:

Apareceu `kvm`? Entãovamos incluir o nosso usuário no grupo 'kvm', execute:
```bash
sudo usermod -aG kvm $USER
```
Depois, verifique se realmente estou no grupo `kvm`, execute:  
```bash
getent group kvm
```  
Você deve ver:
>kvm:x:993:gsantana    

Se aparecer `kvm` e `gsantana`, então tá tudo certo e podemos prosseguir. 

>**CURIOSIDADE**: Em algumas distros, o grupo 'kvm' não existe porque não é necessário, a distro cuida disto de forma diferente, mas no caso do Debian/Ubuntu e derivações, ele precisa existir.  

Depois disso, então repetimos a operação para observar se o grupo 'libvirt' existe:  
```bash
getent group libvirt
```
É **obrigatório o grupo libvirt existir**, se não existir, algo deu muito errado nos passos anteriores, deverá aparecer algo como:
> libvirt:x:117:   

Apareceu `libvirt`? Então agora que sabemos que ele existe, então incluímos nosso usuário no grupo 'libvirt', execute:
```bash 
sudo usermod -aG libvirt $USER
```
Depois, verifique se realmente estou nestes grupos:  
```bash 
getent group libvirt
```  
Você deve ver:
>libvirt:x:117:gsantana
 
Se aparecer `libvirt` e `gsantana`, então tá tudo certo e podemos prosseguir.  

Esses acessos são dados para que o usuário possa ter acesso a arquivos e pastas que apenas o `libvirt` e `kvm` teriam.   


### VIRTUALIZAÇÃO NATIVA QEMU+KVM - Para uso em Desktops
Por tratar-se de um desktop, faça a instalação mais completa:
```bash
sudo apt install -y virt-manager virtiofsd 
sudo apt install -y spice-vdagent spice-webdavd qemu-guest-agent
#sudo apt install -y davfs2 # requer sudo usermod -aG davfs2 $USER
```
Depois:
```bash
systemctl status spice-vdagentd
```
Se o serviço estiver inativo  `Active: inactive`, então inicie ele:
Depois:
```bash
sudo systemctl start spice-vdagentd
```

|Pacote|Explicação|
|:--|:--| 
|virt-manager|Para uso em desktop ou estação de trabalho, o virt-manager é praticamente indispensável.|   
|virtiofsd|O pacote virtiofsd fornece o daemon do Virtio-FS, que é o método moderno (e mais rápido) para compartilhar pastas entre host e VMs Linux.|  
|spice-vdagent|copiar/colar e ajuste automatico de resolução|  
|spice-webdavd|arrastar e soltar arquivos (SPICE-WebDAV)|  
|qemu-guest-agent|sincronização de tempo|    

### VIRTUALIZAÇÃO NATIVA QEMU+KVM - CONFERINDO MÓDULOS
Agora, verifique se os módulos do KVM estão carregados no kernel:
```bash
lsmod | grep kvm
```
Uma saída aceitável seria:  
```
kvm_amd               217088  0
kvm                  1396736  1 kvm_amd
irqbypass              12288  1 kvm
ccp                   163840  1 kvm_amd
```
Se constar na lista o módulo *kvm* e kvm_amd|kvm_intel, então tá tudo certo.

Depois, *reinicie o computador*.   

### VIRTUALIZAÇÃO NATIVA QEMU+KVM - ATIVÁ-LO NO BOOT
Se os módulos do 'kvm' aparecem então agora é o momento de prepará-los para iniciar-se como serviço durante o boot, assim, inicie o serviço do libvirtd com:  
```bash
sudo systemctl start libvirtd
```
E para iniciar o serviço durante o boot, execute:
```bash
sudo systemctl enable libvirtd
```
A resposta aguardada é:
```
Synchronizing state of libvirtd.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install enable libvirtd
```
Confira que o serviço esta ativado:
```bash
sudo systemctl status libvirtd
```
Um resultado assim, aparecerá:
```bash
● libvirtd.service - libvirt legacy monolithic daemon
     Loaded: loaded (/usr/lib/systemd/system/libvirtd.service; enabled; preset: enabled)
     Active: active (running) since Wed 2025-10-08 16:26:55 -03; 41s ago
 Invocation: dfc2bb59b8ae4ab3929c9385a657e489
TriggeredBy: ● libvirtd-ro.socket
             ● libvirtd-admin.socket
             ● libvirtd.socket
       Docs: man:libvirtd(8)
             https://libvirt.org/
   Main PID: 3956 (libvirtd)
      Tasks: 21 (limit: 32768)
     Memory: 6.2M (peak: 8M)
        CPU: 199ms
     CGroup: /system.slice/libvirtd.service
             └─3956 /usr/sbin/libvirtd --timeout 120

out 08 16:26:55 ti-01 systemd[1]: Starting libvirtd.service - libvirt legacy monolithic daemon...
out 08 16:26:55 ti-01 systemd[1]: Started libvirtd.service - libvirt legacy monolithic daemon.
```
Se retornou `Active: active` então tá tudo certo.

### VIRTUALIZAÇÃO NATIVA QEMU+KVM - PASTA PARA ARMAZENAR AS VMs e ISOs
Então temos o pool **default** usado pelo libvirt é **/var/lib/libvirt/images**, essa é a localização formal criada pelo próprio virt-manager quando você executá-o pela primeira vez, se estivessemos falando de servidores, a partição **/var** seria uma ótima opção desde que fosse uma partição separada, mas em desktops é muito comum jogarmos /var dentro da partição /(root) o que torna **/var** inviável. Assim recomendo que suas VMs estejam numa partição com mais espaço, por exemplo, o seu /home/$USER/libvirt, assim execute:  
```bash
mkdir -p ~/libvirt/images
```

Vamos aproveitar este momento de criação de pastas e vamos criar uma pasta específica para guardar arquivos .iso, execute:
```bash
mkdir -p ~/libvirt/isos
```

Tanto a pasta de `imagens` como a de `isos` você pode trocar a localização para qualquer outro local, desde que o grupo **libvirt* tenha acesso a ela, por isso, damos permissãoà pasta e subpastas ao libvirt e kvm, execute:  
```bash
sudo chmod g+s ~/libvirt
sudo chmod g+s ~/libvirt/images
sudo chmod g+s ~/libvirt/isos
sudo chown -R libvirt-qemu:kvm ~/libvirt
```



### VIRTUALIZAÇÃO NATIVA QEMU+KVM - POOLS PARA ARMAZENAR AS VMs E ISOs
O sistema trabalha no que chama de 'pools', o conceito é que cada pool tem um nome e aponta para uma pasta ou dispositivo, você pode ter todas as VMs no mesmo lugar, ou criar pools diferentes para cada contexto, vamos ver agora quantos pools temos no sistema, execute:  
```bash
sudo virsh pool-list --all --details
```
E então será exibo algo assim: 
```
 Nome      Estado       Auto-iniciar   Persistente   Capacidade   Alocação    Diposnível
------------------------------------------------------------------------------------------
```

Se não apareceu nada acima, é porque você ainda não executou o `virt-manager` ainda. Quando o `virt-manager` é executado pela primeira vez, ele cria o pool sozinho, porém usando /var como referencia de destino. Então se no seu caso não apareceu nada, então execute:  
```bash
sudo virsh pool-define-as default dir --target /home/gsantana/libvirt/images
sudo virsh pool-autostart default
sudo virsh pool-start default
```
Agora temos um pool `default`, seja criado por nós ou pelo próprio `virt-manager`, seja como for, vamos olhar para onde nosso pool **default**esta apontando, execute:
```bash
sudo virsh pool-dumpxml "default" | grep -oP '(?<=<path>).*(?=</path>)'
```
Se observar que está apontando para o **/var**:
>/var/lib/libvirt/images

Então terá de trocar a localização para qualquer outro local, preferencialmente **~/libvirt/images**, primeiro vamos desativar o pool 'default', execute:
```
sudo virsh pool-destroy default
```
Esse nome "destroy" dá a ideia de apagar, mas não apaga, apenas desativa, uma vez desativado o pool, podemos agora modificá-lo, execute:  
```
sudo virsh pool-edit default
```
Agora, procure pelo caminho **path** no arquivo XML, algo como:  
```
    <path>/var/lib/libvirt/images</path>
```
E então troque pelo caminho desejado:
```
    <path>/home/gsantana/libvirt/images</path>
```
Salve e feche o arquivo (`Ctrl+O`, `Enter`, `Ctrl+X`).    

Agora, construa, inicie o pool e ajuste-o para autoinicio após o boot, execute:  
```
sudo virsh pool-build default
sudo virsh pool-start default
sudo virsh pool-autostart default
```

Agora vamos repetir o comando abaixo e conferir se as modificações que implantamos estão corretas:  
```bash
sudo virsh pool-list --all --details
```
E então será exibo algo assim: 
```
 Nome      Estado       Auto-iniciar   Persistente   Capacidade   Alocação    Diposnível
------------------------------------------------------------------------------------------
 default   executando   sim            sim           821,56 GiB   19,33 GiB   802,22 GiB
```
Como poderá observar, a alocação agora está mais generosa, parece que realmente não está mais usando o `/var`, vamos conferir? Execute:
```bash
sudo virsh pool-dumpxml "default" | grep -oP '(?<=<path>).*(?=</path>)'
```
E então será exibo algo assim: 
>/home/gsantana/libvirt/images   

Isso confirma que realmente mudamos o pool 'default' de lugar.  

Se você já copiou arquivos para os pools, então seria importante executar também:  
```bash
sudo find ~/libvirt -type f -exec chmod 666 {} \; -o -type d -exec chmod 777 {} \;
```

Pronto, novas VMs serão criadas no diretório desejado.  

Se você aponto o pool `default` para um local que usa partição Btrfs, então você terá uma perda de performance se não aplicar as instruções no link abaixo:   

[Virtualização nativa do qemu+kvm usando Btrfs](debian_qemu_kvm_btrfs.md)


### VIRTUALIZAÇÃO NATIVA QEMU+KVM - Localização das ISOs  
Também precisaremos de um repositório para guardar nossas isos - arquivos de instalação de sistemas operacionais - escolha o diretorio que desejar, mas o mais bacana é não ter arquivos .iso dentro de unidades caras e rápidas como ssd, o interessante é armazená-las em discos mecânicos que são mais baratos, mas isso é apenas uma sugestão, caso você use o sistema de virtualização apenas para máquinas Windows e terá poucos isos, talvez não faça diferença onde criar este _pool_ porque apagará estes .iso depois de terminada a instalação e é este exemplo que faço abaixo.   

Nos passos anteriores, assumimos que nossas ISOs serão gravadas em:  
```
~/libvirt/isos
```
Então criamos o pool 'isos', execute:
```
sudo virsh pool-define-as isos dir - - - - "/home/gsantana/libvirt/isos"
```

O pool 'isos' está definido, mas precisa ser construído e iniciado(também autoiniciado após o boot), então execute:  
```
sudo virsh pool-build isos
sudo virsh pool-start isos
sudo virsh pool-autostart isos
```

Agora, vamos conferir, execute:  
```bash
sudo virsh pool-list --all
```
E será exibido algo como:
```
 Nome      Estado   Auto-iniciar
----------------------------------
 default   ativo    sim
 isos      ativo    sim
```
Ótimo, o pool **isos** está pronto, vamos ver mais detalhes dele, execute:
```bash
sudo virsh pool-dumpxml "isos" | grep -oP '(?<=<path>).*(?=</path>)'
```
E verá algo como:
```
/home/gsantana/libvirt/isos
```


Se no futuro quiser mudar o lugar desse pool, você pode excluir e criar de novo, assim:
```
virsh pool-list --all # para listar todos e ter certeza do nome a excluir
sudo virsh pool-destroy isos  # parar com o uso do pool
sudo virsh pool-undefine isos # remover a definição do pool
```
Essa remoção do pool não mexe com os arquivos que estavam na pasta, e caso precise disso, remova-os manualmente.  

Se você já copiou arquivos para os pools, então seria importante executar também:  
```bash
sudo find ~/libvirt -type f -exec chmod 666 {} \; -o -type d -exec chmod 777 {} \;
```

### VIRTUALIZAÇÃO NATIVA QEMU+KVM - CRIANDO UMA INTERFACE BRIDGE
Para trabalhos extensos e mais profissionais com VMs é impossivel viver apenas com NAT porque na maioria dos ambientes de desenvolvimento ou corporativos uma VM precisa enxergar o anfitrião e também as outras VMs, então siga o tutorial a seguir para criar uma conexão do tipo bridge em seu sistema:  

[Criando conexões bridge pelo terminal](debian_qemu_kvm_bridge.md)   

É claro que se você está lendo este tutorial e apenas quer usar o Windows com o NAT, não precisará de uma conexão de bridge.  
  

### VIRTUALIZAÇÃO NATIVA QEMU+KVM - Windows
A virtualização com QEMU+KVM oferece desempenho quase nativo, aproveitando recursos do processador e integração direta com o kernel Linux. É uma opção poderosa e leve para executar Windows dentro do Linux, especialmente quando usada com o Virt-Manager, que simplifica a criação e o gerenciamento de VMs.  

Neste guia, você verá como criar uma máquina virtual Windows otimizada, usando o pacote de drivers virtio-win para melhor desempenho de disco, rede e vídeo, além de boas práticas sobre discos dinâmicos, rede bridge e configurações essenciais.  

Agora que você entende o contexto, vamos começar preparando o ambiente com o pacote de drivers VirtIO. Siga as instruções no link abaixo:  

[Criando e ajustando VM Windows](debian_qemu_kvm_windows.md)


### VIRTUALIZAÇÃO NATIVA QEMU+KVM - Criando máquinas virtuais pelo Virt-Manager
Instruções de como usar o virt-manager encontra-se na página:  
[Criando máquinas virtuais pelo Virt-Manager](https://sempreupdate.com.br/como-configurar-e-usar-o-virt-manager-para-kvm-no-fedora-ubuntu-debian-e-derivados/#google_vignette)   


### VIRTUALIZAÇÃO NATIVA QEMU+KVM - Permissões
A cada copia de arquivos para a pasta **~/libvirt** é bom consertar as permissões, execute:
```bash
sudo find ~/libvirt -type f -exec chmod 666 {} \; -o -type d -exec chmod 777 {} \;
sudo chown -R libvirt-qemu:kvm ~/libvirt
```
É bom que faça isso porque o dono (owner) dos arquivos que estão sendo copiados podem ser diferentes de **libvirt-qemu:kvm** e isso pode efetar a execução da VM. Isso não é necessário para VMs que são criadas ali.  

### OTIMIZANDO O DISCO QCOW2
O QCOW2 é um formato copy-on-write com recursos como snapshots, compressão e alocação sob demanda. Esses recursos trazem overhead e, com o tempo, geram fragmentação interna. Mas máquinas Windows são muito mais afetadas do que as demais porque o Windows gera memória virtual, arquivos temporários a todo instante. Então o link a seguir descreve como podemos otimizar e compactar o disco virtual para que o desempenho - especialmente para VMs Windows - fique sempre máximizado.  

[Instruções para melhorar o desempenho](debian_qemu_kvm_otimizar_disco.md)

### Backup de Máquinas Virtuais em QEMU+KVM
**Backup de máquina virtual** é a replicação sistemática dos arquivos de disco (imagens QCOW2, RAW, VDI, etc.) e metadados de configuração (arquivos XML do libvirt) para um local independente, garantindo recuperação em caso de corrupção, falha de hardware, exclusão acidental ou desastre. Diferencia-se de snapshots, que são pontos de restauração locais; backups são cópias isoladas em mídia ou storage separado.  

Para entender melhor e aprender como fazer siga o link:  
[Realizando backups de maquinas virtuais](debian_qemu_kvm_backup.md)  


### Convertendo máquinas virtuais do VirtualBox para QEMU+KVM
O VirtualBox é um hipervisor do tipo 2 amplamente utilizado para virtualização em desktops. As vezes é preciso migrar máquinas virtuais desta plataforma para o QEMU+KVM. A conversão funciona, mas levamos junto o lixo da instalação anterior e nem sempre temos os melhores resultados, no entanto, é melhor do que criar tudo do zero. Siga o link abaixo se precisar migrar máquinas virtuais do VirtualBox:  

[Convertendo máquinas virtuais do VirtualBox para QEMU/KVM](debian_qemu_kvm_vbox.md)  


----

[Clique aqui para retornar a página principal](../README.md#virtualiza%C3%A7%C3%A3o-nativa-qemukvm)
