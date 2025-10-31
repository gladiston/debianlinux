# VIRTUALIZAÇÃO NATIVA QEMU+KVM - USANDO VM WINDOWS
A virtualização com QEMU+KVM oferece desempenho quase nativo, aproveitando recursos do processador e integração direta com o kernel Linux. É uma opção poderosa e leve para executar Windows dentro do Linux, especialmente quando usada com o Virt-Manager, que simplifica a criação e o gerenciamento de VMs.

Neste guia, você verá como criar uma máquina virtual Windows otimizada, usando o pacote de drivers virtio-win para melhor desempenho de disco, rede e vídeo, além de boas práticas sobre discos dinâmicos, rede bridge e configurações essenciais.

Agora que você entende o contexto, vamos começar preparando o ambiente com o pacote de drivers VirtIO.

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
        
 

### VIRTUALIZAÇÃO NATIVA QEMU+KVM - Criando conexões bridge
O padrão de rede da VM é usar **NAT**, se você deseja colocar essa VM como cliente de sua rede, troque de **NAT** por **bridge** e forneça a conexão bridge para suas VMs. Algumas formas de compartilhamento de arquivos entre anfitrião e convidado só funcionará se a VM estiver usando a rede em modo bridge.  

Para trabalhos extensos e mais profissionais com VMs é impossivel viver apenas com NAT, então siga o tutorial a seguir para criar uma conexão do tipo bridge em seu sistema:  

[Criando conexões bridge pelo terminal](debian_qemu_kvm_bridge.md)  
  
### VIRTUALIZAÇÃO NATIVA QEMU+KVM - CRIANDO UMA INTERFACE BRIDGE
Para trabalhos extensos e mais profissionais com VMs é impossivel viver apenas com NAT porque na maioria dos ambientes de desenvolvimento ou corporativos uma VM precisa enxergar o anfitrião e também as outras VMs, então siga o tutorial a seguir para criar uma conexão do tipo bridge em seu sistema:  

[Criando conexões bridge pelo terminal](debian_qemu_kvm_bridge.md)   

Se já conseguiu criar uma interface bridge do tipo macvtap, então o que resta agora é configurar sua VM Windows, vá no virt-manager, ao criar ou editar uma VM:  

1. Vá em Interface de rede>Fonte de rede.
2. Escolha Interface Host.
3. Selecione macvtap0 (modo bridge).
4. Clique em Aplicar e salve.

[Tela de configuração da VM](../img/debian_qemu_kvm_bridge1.png)

A VM agora receberá IP direto do servidor DHCP da rede local, sem NAT, podendo se comunicar com outras VMs e dispositivos da LAN.

Agora vamos a outros ajustes:
a) Vá em “Exibir detalhes”>“Vídeo”  
Tipo: QXL (essencial para SPICE).  
Se não existir, adicione “Dispositivo de vídeo >Modelo QXL”.  

b) Vá em “Canal”  
Deve haver um canal “spice” (Principal), se não houver, adicione:  
Adicionar Hardware>Canal>Tipo: spice>Nome: main.0  
Adicione também um “Canal SPICE agent” (isso é o que habilita o clipboard).  

c) Vá em “Display”  
Tipo: Spice (não VNC).  
Habilite “Listen Type: None” (para acesso local via virt-viewer).  
Ative também OpenGL se quiser aceleração.  

### Dentro do Windows Guest
Baixe o instalador oficial SPICE Guest Tools (assinatura Red Hat):
[https://www.spice-space.org/download.html](https://www.spice-space.org/download.html)

Procure o pacote `spice-guest-tools-latest.exe` e instale (basta “Next, Next, Finish”) e reinicie o Windows. Ele instala:
* spice-vdagent
* qxl driver de vídeo
* drivers para mouse, clipboard e redimensionamento dinâmico

### Otimizando o Windows - Serviçõs dispensáveis
Alguns serviços o Windows sao dispensáveis, execute `services.msc` e desative alguns desses(ou todos eles):  
## ⚙️ Serviços seguros para desativar em VMs do Windows Server / Windows 2025

| Nome exibido no `services.msc` | Nome interno (`sc config ...`) | Função | Pode desativar? |
|--------------------------------|-------------------------------|---------|------------------|
| **Windows Search** | `WSearch` | Indexação de arquivos e e-mails | ✅ |
| **SysMain** *(antigo Superfetch)* | `SysMain` | Otimiza inicialização e cache de aplicativos | ✅ |
| **Optimize Drives (Desfragmentador)** | `defragsvc` | Desfragmenta discos mecânicos | ✅ |
| **Windows Error Reporting Service** | `WerSvc` | Envia relatórios de erro para a Microsoft | ✅ |
| **Diagnostic Policy Service** | `DPS` | Detecta e tenta corrigir problemas de rede e hardware | ✅ |
| **Connected User Experiences and Telemetry** | `DiagTrack` | Coleta telemetria e estatísticas de uso | ✅ |
| **Windows Update Medic Service** | `WaaSMedicSvc` | Reativa o Windows Update automaticamente | ✅ |
| **Remote Registry** | `RemoteRegistry` | Permite editar o registro remotamente | ✅ |
| **Fax** | `Fax` | Suporte a envio de fax | ✅ |
| **Print Spooler** | `Spooler` | Gera fila de impressão | ✅ *(a não ser que use impressoras)* |
| **Bluetooth Support Service** | `bthserv` | Gerencia dispositivos Bluetooth | ✅ |
| **Smart Card** | `SCardSvr` | Gerencia cartões inteligentes | ✅ |
| **Secondary Logon** | `seclogon` | Permite “Executar como outro usuário” | ⚠️ *Opcional* |
| **Windows Defender Antivirus Service** | `WinDefend` | Proteção antivírus | ⚠️ *Somente se VM isolada* |
| **Offline Files** | `CscService` | Sincroniza arquivos offline | ✅ |
| **Program Compatibility Assistant Service** | `PcaSvc` | Detecta compatibilidade de programas antigos | ✅ |
| **Security Center** | `wscsvc` | Central de segurança (alertas) | ✅ |

Algo que pode agilizar é criar um script `agilizar_vm.bat` com o seguinte cnteúdo:  

```cmd
@echo off
echo === Otimizando VM Windows para melhor desempenho ===
echo.

for %%S in (
  WSearch
  SysMain
  defragsvc
  WerSvc
  DPS
  DiagTrack
  WaaSMedicSvc
  RemoteRegistry
  Fax
  Spooler
  bthserv
  SCardSvr
  seclogon
  WinDefend
  CscService
  PcaSvc
  wscsvc
) do (
  echo Desativando %%S ...
  sc stop "%%S" >nul 2>&1
  sc config "%%S" start= disabled >nul 2>&1
)

echo.
echo === Concluido! Reinicie o Windows para aplicar todas as alteracoes. ===
pause
```
Aproveite para remover remova os nomes de serviços que na sua definição lhe são útes, afinal, a lista acima desativa todos os serviços que detalhei na tabela.    
Salve o conteúdo acima como `otimizar-vm.cmd`, então clique com o botão direito sobre ele e **“Executar como Administrador”**.  


### Teste o copiar/colar

Abra a VM no virt-manager (janela SPICE) e:
* Copie um texto no host Linux>Ctrl + V dentro da VM Windows   
* Copie algo no Windows>Ctrl + Shift + V (ou normal, dependendo do app) no host   

Se funcionar em um sentido mas não no outro, reinicie tanto o serviço no host:
```bash
sudo systemctl restart spice-vdagentd
```
quanto o processo spice-vdagent.exe dentro do Windows (ele inicia automaticamente com o login).  

### COPIA DE ARQUIVOS ENTRE ANFITRIÃO E CONVIDADO
Para copiar arquivos, não só texto:  
* Instale e ative o spice-webdavd no host;
*  Dentro da VM, abra o Explorador>“Rede”>aparecerá um drive WebDAV (Spice), onde dá pra arrastar arquivos.  

Resumo rápido:   
|:---|:---|
|Função	|Onde configurar|
|Copiar/colar texto|spice-vdagent (host + guest)|
|Ajuste de resolução|QXL + spice-vdagent|
|Copiar/colar arquivos|spice-webdavd|
|Canal de comunicação|virt-manager>Canal SPICE agent|


### VÍDEOS NO YOUTUBE RECOMENDADOS
Não se trata mais de criar VMs, as informações que obteve até aqui cobriram essa etapa e algumas foram além disso. Então os links a seguir são para "tunar" suas estações Windows. Esses vídeos incluiem coisas que mencionei e vão além disso, há algumas coisas que são possiveis fazer, mas é dificil, explicar com palavras "como". Então os vídeos a seguir vão estender ainda mais as funcionalidades de computadores com o Windows.

* (How To PROPERLY Install Windows 11 on KVM)[https://youtu.be/7tqKBy9r9b4?si=xmM6vPTbu68kVxPr]  

* [Migrando pro Linux - migre Windows 11 do VirtualBox pro Virt-Manager e Ative ACELERAÇÃO GRÁFICA](https://youtu.be/WZ16GsFHSjM?si=LJoLVhI3c9iBqPWI)

* [Qemu: Data Exchange Between Windows and Linux - Share Folders](https://youtu.be/0hZU3vltZVM?si=wHv3id8czwD4u957)

* [This is a basic tutorial on how to virtualize Any Operating System using QEMU in PlayList](https://www.youtube.com/watch?v=cE6X2IrTzgU&list=PLmsony4NVQpxYb6B51t-uWWuGkph5rmf1)
