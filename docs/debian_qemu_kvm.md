# VIRTUALIZAÇÃO NATIVA QEMU+KVM
O Linux é capaz de criar máquinas virtuais e ele mesmo ser o hypervisor. Será um servidor de virtualização nivel 1, o mais rápido possivel, no entanto com algumas ausencia de recursos que facilitam a configuração que existem no VirtualBox e VMWare, por exemplo, criar redes virtuais com vários tipos de topologias,  clipboard e transferencia de arquivos entre host e anfitrião e outras coisas.  

### Vamos fazer o backup?
Antes de prosseguirmos, que tal fazer o backup da sua configuração de rede original?  Isso é importante porque assim que instalar o `libvirt` ele vai criar uma interface NAT para virtualizar e isso vai mudar o ambiente inicial, se quiser saber como fazer isso, siga as instruções no link a seguir:  
[Fazendo um backup das configurações de rede](debian_backup_restore_network.md)  


### Vamos instalar os pacotes principais:  
```bash
sudo apt install -y qemu-kvm libvirt-daemon-system libvirt-clients
sudo apt install -y bridge-utils dnsmasq-base ovmf isc-dhcp-client
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
```

|Pacote|Explicação|
|:--|:--| 
|virt-manager|Para uso em desktop ou estação de trabalho, o virt-manager é praticamente indispensável.|  
|virtiofsd|O pacote virtiofsd fornece o daemon do Virtio-FS, que é o método moderno (e mais rápido) para compartilhar pastas entre host e VMs Linux.|  

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

### VIRTUALIZAÇÃO NATIVA QEMU+KVM - PASTA PARA ARMAZENAR AS VMs
O sistema trabalha no que chama de 'pools', o conceito é que cada pool tem um nome e aponta para uma pasta ou dispositivo, você pode ter todas as VMs no mesmo lugar, ou criar pools diferentes para cada contexto, vamos ver agora quantos pools temos no sistema, execute:  
```bash
sudo virsh pool-list --all --details
```
E então será exibo algo assim: 
```
 Nome      Estado       Auto-iniciar   Persistente   Capacidade   Alocação    Diposnível
------------------------------------------------------------------------------------------
 default   executando   sim            sim           114,79 GiB   11,67 GiB   103,12 GiB
```

Caso ele não apareça nada na listagem, é porque você ainda não executou o `virt-manager` pela primeira vez. Quando ele é executado pela primeira vez, ele cria o pool sozinho, porém usando /var que tem muito mesmo espaço que o nosso $HOME. Mas se não apareceu nada, então execute:  
```bash
sudo virsh pool-define-as default dir --target /var/lib/libvirt/images
sudo virsh pool-autostart default
sudo virsh pool-start default
```

Agora que confirmamos a existencia do pool `default` Como poderá observar, há apenas 1 pool, chamado 'default' que se estiver apontando para /var tem muito menos espaço disponível do que teria se fosse nosso $HOME, por isso confira novamemte:, execute:
```bash
sudo virsh pool-dumpxml "default" | grep -oP '(?<=<path>).*(?=</path>)'
```
Se ver algo assim, isto é, indicando o **/var**:
>/var/lib/libvirt/images

Então temos o pool **default** que aponta para **/var/lib/libvirt/images**, essa é a localização formal criada pelo próprio virt-manager, se estivessemos falando de servidores, a partição **/var** seria uma subpartição ou partição separada. Mas em desktops é muito comum jogarmos /var dentro da partição /(root) que normalmente tem capacidade menor de espaço, e se você seguiu o tutorial até aqui é bem provavel que esteja assim também no seu computador. Assim recomendo que suas VMs estejam numa partição com mais espaço, por exemplo, o seu /home/$USER/libvirt, assim execute:  
```bash
mkdir -p ~/libvirt/images
```
Você pode trocar a localização para qualquer outro local, desde que o grupo **libvirt* tenha acesso a ela, por isso, damos permissãoà pasta e subpastas ao libvirt e kvm, execute:  
```bash
sudo chmod g+s ~/libvirt/images
sudo chown -R libvirt-qemu:kvm ~/libvirt
```

Agora que a pasta foi criada com sucesso e tem as permissões corretas, então vamos definir o pool de imagens para lá, primeiro vamos desativar o pool 'default', execute:
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

Se vocÊ já copiou arquivos para os pools, então seria importante executar também:  
```bash
sudo find ~/libvirt -type f -exec chmod 666 {} \; -o -type d -exec chmod 777 {} \;
```

Pronto, novas VMs serão criadas no diretório desejado.  

Se você aponto o pool `default` para um local que usa partição Btrfs, então você terá uma perda de performance se não aplicar as instruções no link abaixo:   

[Virtualização nativa do qemu+kvm usando Btrfs](debian_qemu_kvm_btrfs.md)


### VIRTUALIZAÇÃO NATIVA QEMU+KVM - Localização das ISOs  
Também precisaremos de um repositório para guardar nossas isos - arquivos de instalação de sistemas operacionais - escolha o diretorio que desejar, mas o mais bacana é não ter arquivos .iso dentro de unidades caras e rápidas como ssd, o interessante é armazená-las em discos mecânicos que são mais baratos, mas isso é apenas uma sugestão, caso você use o sistema de virtualização apenas para máquinas Windows e terá poucos isos, talvez não faça diferença onde criar este _pool_ porque apagará estes .iso depois de terminada a instalação e é este exemplo que faço abaixo, execute:  

```
sudo -p ~/libvirt/isos
sudo chown -R libvirt-qemu:kvm ~/libvirt
```

Agora que a pasta com as permissões foram criadas, então criamos o pool 'isos', execute:
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

### VIRTUALIZAÇÃO NATIVA QEMU+KVM - Windows
O `virtio-win.iso` é um pacote de drivers e utilitários da Red Hat criado para melhorar o desempenho e a compatibilidade de máquinas virtuais Windows executadas em hipervisores baseados em KVM/QEMU (como Virt-Manager, Proxmox, Xen, etc.).
Se pretende virtualizar máquinas windows precisará dessa .iso em seu sistema. Em nosso exemplo anterior, o pool de arquivos `.iso` é a pasta de **~/libvirt/isos**, então vamos baixá-lo lá, execute:  
```
cd ~/libvirt/isos
wget -vc https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso
```

Se estiver pensando em ambiente de desenvolvimento, a ISO do Windows Server é melhor, tem menor footprint de consumo de memória e CPU, e também, geralmente as atualizações dele não quebram o sistema como as atualizações da versão Workstation. Embora não exista uma `.iso` em português para o Windows Server, fique tranquilo, desde a edição do Windows Server 2012 é possivel modificar o idioma após a instalação. O link para download é:  
[Site oficial da Microsoft para baixar o Windows Server](https://www.microsoft.com/pt-br/evalcenter/download-windows-server-2025)  


**ALERTA**: Voce pode usar o virt-manager para criar suas VMs e na tela que você define o tamanho do disco haverá um pseudo-problema, em todas as vezes que crio o disco por esse wizard, os discos virtuais criados serão de tamanho fixo, ou seja, se definir alí que sua VM terá um disco de 200GB, ela terá exatamente 200GB ocupados no sistema operacional. O formato qcow2 aceita usar discos dinamicos, isto é, você cria um disco virtual de 200GB, mas no sistema operacional ele ocupará um tamanho mínimo e crescerá conforme o uso, então como usar discos dinâmicos? Simples, crie primeiro o disco no gerenciador de pools e ele perguntará se deseja um disco de tamanho fixo ou dinâmico e então escolha essa ultima opção e pronto, o arquivo .qcow2 será criado no pool 'default' com tamanho dinâmico. Depois disso, crie sua VM e associe o disco recém-criado com ela. Se houver algum **bug** nisso, poderá também criar o disco virtual pelo terminal, neste exemplo crio um disco de 200GB no pool 'default', veja:  
```
sudo virsh vol-create-as default win2k25.qcow2 200G --format qcow2
```
Agora vamos conferir o tamanho:  
```
$ ls -lh ~/libvirt/images
total 196K
-rw------- 1 root kvm 196K Oct 17 14:31 win2k25.qcow2
```
Como pôde ver, um disco de 200GB que ocupa apenas 196K no sistema. É claro que a medida que formos instalar o sistema e todas as demais coisas, este arquivo subirá de tamanho.  
Na minha modesta opinião, eu criaria discos apenas pelo terminal porque podemos criar vários em sequencia, evitando o wizard repetitivo para cada um deles.  

Outras instruções e explicações do porque precisamos desses drivers podem ser obtidas aqui:   
https://github.com/virtio-win/virtio-win-pkg-scripts/blob/master/README.md   

No vídeo a seguir, há uma playlist, cada uma com explicação sobre máquinas virtuais:  
[Produtividade com máquinas virtuais](https://youtu.be/8swg8mDQ9SA?si=HZC7vKnrx7ZxmCfE)  

**NÃO ESQUEÇA AO CRIAR UMA VM WINDOWS**
Depois de instalar os drivers de convidado(virtio) numa VM Windows e reiniciar a VM, não esqueça de:  
1. Na configuração da VM, trocar a Placa de rede (NIC) para **virtio**, isso melhorará o desempenho.  
2. Na configuração da VM, trocar a placa de vídeo em **Vídeo QXL** para usar o modelo **virtio**, isso melhorará o desempenho.  
3. Na configuração da VM, na seção **Memória** se você ajustar a memória minima para algo diferente do máximo, o Windows buga, não faça isso.  
4. Na configuração da VM, na seção **Memória** marque a opção "**Habilitar memória compartilhada** se tiver muitas VMs com o mesmo SO, isso economizará RAM. Essa opção também é necessária caso tenha que compartilhar arquivos entre host e convidado, falaremos disso noutra ocasião.  
5. Na configuração da VM, na seção **Exibição Spide** marque o tipo como **Servidor Spice**, tipo de escula como **Endereço** e endereço como **Todas as interfaces** e se achar que precisa de aceleração marque a opção **OpenGL**, o qual não recomendo. O SPICE é um protocolo de acesso remoto, mas com um detalhe importante — ele também é usado localmente pelo virt-manager para exibir a tela da VM, então mesmo que você não esteja acessando “remotamente”, o virt-manager está se conectando via SPICE por baixo dos panos.  
6. Dentro da VM Windows, crie uma conta com poderes de admin para seu dia a dia, e evite a conta **Administrador**.  
7. Dentro da VM Windows, instale um programa de [autologon](https://learn.microsoft.com/pt-br/sysinternals/downloads/autologon).  
8. Ser for Windows Server, não esqueça de ir em "Language" e mudar o idioma de inglês para o português/brasil, regionalidade, dicionários, etc... e no final, copiar para todos os usuários e tela de boas vindas.  
9. Para compartilhar arquivos/pastas entre anfitrião e convidado sem precisar de compartilhar via rede veja este [link aqui](https://www.debugpoint.com/share-folder-virt-manager/) e [este outro aqui](https://www.debugpoint.com/kvm-share-folder-windows-guest/#google_vignette).
        
### VIRTUALIZAÇÃO NATIVA QEMU+KVM - Permissões
A cada copia de arquivos para a pasta **~/libvirt** é bom consertar as permissões, execute:
```bash
sudo find ~/libvirt -type f -exec chmod 666 {} \; -o -type d -exec chmod 777 {} \;
sudo chown -R libvirt-qemu:kvm ~/libvirt
```
É bom que faça isso porque o dono (owner) dos arquivos que estão sendo copiados podem ser diferentes de **libvirt-qemu:kvm** e isso pode efetar a execução da VM. Isso não é necessário para VMs que são criadas ali.   

### VIRTUALIZAÇÃO NATIVA QEMU+KVM - Criando conexões bridge
O padrão de rede da VM é usar **NAT**, se você deseja colocar essa VM como cliente de sua rede, troque de **NAT** por **bridge** e forneça a conexão bridge para suas VMs. Algumas formas de compartilhamento de arquivos entre anfitrião e convidado só funcionará se a VM estiver usando a rede em modo bridge.  

Para trabalhos extensos e mais profissionais com VMs é impossivel viver apenas com NAT, então siga o tutorial a seguir para criar uma conexão do tipo bridge em seu sistema:  

[Criando conexões bridge pelo terminal](debian_qemu_kvm_bridge.md)  
  

Se desejar criar uma conexão bridge usando a interface do KDE, há vários vídeos no YouTube ensinando como fazer, [recomendo este aqui.](https://youtu.be/jZEN5jFn8rY?si=m9vVPFLATud_DtOE&t=99) ou este [daqui](https://youtu.be/_9zg37IZDsk?si=M66Wad4W21qZ816f). 


### VIRTUALIZAÇÃO NATIVA QEMU+KVM - Criando máquinas virtuais pelo Virt-Manager
Instruções de como usar o virt-manager encontra-se na página:  
[Criando máquinas virtuais pelo Virt-Manager](https://sempreupdate.com.br/como-configurar-e-usar-o-virt-manager-para-kvm-no-fedora-ubuntu-debian-e-derivados/#google_vignette)   

