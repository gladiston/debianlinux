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


## INSTALANDO O WINDOWS
Em nosso exemplo, vamos instalar o Windows 2025 Server.  Deixe um `.iso` de instalação deste sistema operacional em **~/libvirt/isos**.  

### CRIANDO DISCO VIRTUAL PARA HOSPEDAR A VM
Voce pode usar o virt-manager para criar suas VMs e na tela que você define o tamanho do disco haverá um problema que detectei nas distros Debian e derivados, em todas as vezes que crio o disco por esse wizard embutido, os discos virtuais criados serão de tamanho fixo, ou seja, se definir alí que sua VM terá um disco de 200GB, ela terá exatamente 200GB ocupados no sistema operacional do hospedeiro. O formato qcow2 aceita usar discos dinamicos, isto é, você cria um disco virtual de 200GB, mas no sistema operacional ele ocupará um tamanho mínimo e crescerá conforme o uso, então como usar discos dinâmicos? Até que corrijam este problema, crie o disco no gerenciador de pools e ele perguntará se deseja um disco de tamanho fixo ou dinâmico e então escolha essa ultima opção e pronto, o arquivo .qcow2 deverá ser criado no pool `default` com tamanho dinâmico. Pessoalmente, acho o Wizard do virt-manager burocrático, então se quiser ser mais rapido, apenas execute no terminal:  
```
sudo virsh vol-create-as default win2k25.qcow2 200G --format qcow2
```

Agora vamos conferir o tamanho:  
```
$ ls -lh ~/libvirt/images
total 196K
-rw------- 1 root kvm 196K Oct 17 14:31 win2k25.qcow2
```
Como pôde ver, um disco de 200GB que ocupa apenas 196K no sistema. É claro que a medida que formos instalar o sistema e todas as demais coisas, este arquivo subirá de tamanho. Na minha modesta opinião, eu criaria discos apenas pelo terminal porque podemos criar vários em sequencia, evitando o wizard burocrático e repetitivo para cada um deles.  

### AJUSTE O VIRT-MANAGER
Carregue o virt-manager, vá em **Editar|Preferencias** na guia **Geral** e ligue as opções:  
1. Habiiutar ícone na bandeja do sistema
2. Habilitar edição de XML

![Habilitar edição de XML](debian_qemu_kvm_windows1.png)  

Depois, na mesma janela, na guia **E/S programada**, faça o seguinte ajuste:  
1. Atualizar o satus a cada 3 segundos
2. Obter o uso da CPU: Ligado
3. Obter E/S de disco: Ligado
4. Obter estatistica de memória: Ligado

![Habilitar edição de XML](debian_qemu_kvm_windows2.png)  

Depois, na mesma janela, na guia **Nova VM**, faça o seguinte ajuste:  
1. Tipo de gráfico: PAdrão do sistema(spice)
2. Formnato de armazenamento: QCOW2
3. CPU Padrão: Padrão do Aplicativo
4. Formware x86: Padrão do sistema
   
![Habilitar edição de XML](debian_qemu_kvm_windows3.png)  


Depois, na mesma janela, na guia **Console**, garanta que o ajuste seja:  
1. Escalonamento de console gráfico: Sempre
2. Redimencionar convidado com janela: Padrão do sistema(desligado)
3. Capturar teclas: Ctrl_L(esquerdo)+ALT_L(esquerdo)
4. Redirecionamento SPICE de USB: Redirecionamento manual apenas
5. Conectar automaticamente ao console: Ligado

![Habilitar edição de XML](debian_qemu_kvm_windows4.png)  


### CRIANDO A VM



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

        
## SERVIÇOS ESSENCIAIS
O sistema hospedeiro precisa ter os serviços abaixo rodando, eles são importantes para que possamos tem alguma integração entre o hospedeiro e máquinas windows, são eles:

### Serviço `spice-vdagentd`
Este é um serviço que lhe permitirá compartilhar a área de clipboard entre hospedeiro e convidado, veja se o serviço está habilitado e rodando, execute:  
```bash
$ sudo systemctl status spice-vdagentd
○ spice-vdagentd.service - Agent daemon for Spice guests
     Loaded: loaded (/usr/lib/systemd/system/spice-vdagentd.service; indirect; preset: enabled)
     Active: inactive (dead)
TriggeredBy: ○ spice-vdagentd.socket
```


### Serviço `spice-webdavd`
Este é um serviço que lhe permitirá compartilhar arquivos entre hospedeiro e convidado, veja se o serviço está habilitado e rodando, execute:  
```bash
$ sudo systemctl status spice-webdavd
○ spice-vdagentd.service - Agent daemon for Spice guests
     Loaded: loaded (/usr/lib/systemd/system/spice-vdagentd.service; indirect; preset: enabled)
     Active: inactive (dead)
TriggeredBy: ○ spice-vdagentd.socket
```
Se estes serviços não estiverem rodando, avalie os passos anteriores porque prosseguir sem eles torna a sua VM Windows muito limitada.   

## VIRTUALIZAÇÃO NATIVA QEMU+KVM - Criando conexões bridge
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

## OTIMIZAÇÃO DA VM WINDOWS
O Windows depois de instalado está carregado de coisas que roubam performance, vamos tentar melhorar.  


### Otimizando o Windows - Papel de parede e gadgets
Remova o papel de parede e use uma cor solida como preto. Antes que pergunte, sim, isso faz muito a diferença.  
No painel de menu, remova os recursos que não precisa, como caixa de pesquisa, telas virtuais, etc...  
Lembre-se de que qualquer coisa que consuma ciclos de CPU e não são úteis, devem ser desativados.  

### Otimizando o Windows - Programas dispensáveis
Se você não usa os serviços Microsoft 365 nesta VM, não instale o onedrive e afins, só vão lhe roubar recursos.  

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
  bthserv
  SCardSvr
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
Aproveite para remover remova os nomes de serviços que na sua definição lhe são útes, afinal, a lista acima desativa todos os serviços que detalhei na tabela. Eu por exemplo, removo da lista os serviços a serem desativados: **Print Spooler** e **Secondary Logon** porque eles me são uteis, por isso eles estão fora da lista do arquivo .cmd acima.    
Salve o conteúdo acima como `otimizar-vm.cmd`, então clique com o botão direito sobre ele e **“Executar como Administrador”**.    

Caso se arrependa de ter desativado algum serviço em particular, execute `servces.msc` e ative-o.

### Otimizando o Windows - Recursos Visuais
A configuração de vídeo é um aspecto muito importante porque não importa o quanto a VM seja rápida para processar, o aspecto mais valorizado é a responsividade. As vezes você pode achar a VM lenta, mas quando roda um processo, o processo roda rápido, mas a impressão que se tem é de lerdeza ao operar a VM, isto é a responsividade.  
Se você tiver um notebook que tem uma placa de vídeo Intel e outra NVIDIA, parabens você pode configurar sua máquina virtual para passthrough, isto é, deixar o sistema hospedeiro ficar 100% com uma placa de vídeo(Intel) enquanto a VM fica 100% com a outra placa de vídeo(NVIDIA) por meio de passthrough e poderá inclusive jogar nessa VM com desempenho similar sem virtualização.  
Mas voltando ao assunto, este guia passo a passo foi feito para mortais que usufruem apenas de uma placa de vídeo e como ela fica com o hospedeiro, as VMs "emulam" uma placa de vídeo que usa um driver QXL que é apenas um quebra-galho aceitando apenas a parte 2D, por isso dentro do Windows vocÊ precisa urgentemente desligar todos os efeitos visuais que puder, vá em **Configurações>ConfiguraçõesaAvançadas do sistema>Desempenho** e clique em **Configurações** e deixe selecionado apenas a opção **Usar fontes de tela com cantos arredondados** porque nossos olhos não precisam sangrar também:
!(Desempenho)[../img/debian_qemu_kvm_windows1.png]   

### Otimizando o Windows - Agendador de tarefas
Depois de instalar dentro da VM todos os programas de que precisa, vá no agendador de tarefas e desative os agendamentos de atualizações que estes programas gostam de deixar lá, por exemplo, o Oracle Java e Adobe Reader deixam no Agendador de tarefas programas para atualização de seus produtos. Normalmente ficam programados para conferir se há atualizações de seu produtos quando o computador esta ocioso e diariamente, e isso é horrivel para a nossa VM.  


### Teste o copiar/colar
Uma vez que tenha instalado o programa cliente dentro da VM Windows, o recurso de copiar/colar da área de clipboard funcionará perfeitamente.  

Se não estiver funcionando, talvez seja necessário seguir os passos anteriores.  
O teste é simplesmente, abra a VM no virt-manager (janela SPICE) e:
* Copie um texto no host Linux>Ctrl + V dentro da VM Windows.     
* Copie algo no Windows>Ctrl + Shift + V (ou normal, dependendo do app) no host.   

Se não estiver funcionando, confirme se o host está com o serviço funcionando:  

1. Serviço `spice-vdagentd`, este é um serviço que lhe permitirá compartilhar a área de clipboard entre hospedeiro e convidado:
```bash
$ sudo systemctl status spice-vdagentd
○ spice-vdagentd.service - Agent daemon for Spice guests
     Loaded: loaded (/usr/lib/systemd/system/spice-vdagentd.service; indirect; preset: enabled)
     Active: inactive (dead)
TriggeredBy: ○ spice-vdagentd.socket
```
2. Serviço `spice-webdavd`, este é um serviço que lhe permitirá compartilhar arquivos entre hospedeiro e convidado:
```bash
$ sudo systemctl status spice-webdavd
○ spice-vdagentd.service - Agent daemon for Spice guests
     Loaded: loaded (/usr/lib/systemd/system/spice-vdagentd.service; indirect; preset: enabled)
     Active: inactive (dead)
TriggeredBy: ○ spice-vdagentd.socket



E se o serviço no Windows está habilitado e rodando:  
* QEMU Guest Agent
* Spice Agent
Se não estiverem, os recursos entre hospedeiro e convidado não funcionarão.

### COPIA DE ARQUIVOS ENTRE ANFITRIÃO E CONVIDADO
Para copiar arquivos, não só texto:  
* Instale e ative o spice-webdavd no host;
*  Dentro da VM, abra o Explorador>“Rede”>aparecerá um drive WebDAV (Spice), onde dá pra arrastar arquivos.  

## CRIANDO A VM VIA YOUTUBE
Sem falsa modéstia, mas este guia passo a passo é mais completo que a maioria dos vídeos no YouTube que mostram como criar VMs Windows.
Mas 'ler como fazer' e 'assistir como fazer' são coisas diferentes, e pessoas diferentes podem preferir cada qual, o seu método.
Por isso, depois deste guia pronto, fui fuçar alguns videos e vou recomendar alguns, são eles:  

* (How To PROPERLY Install Windows 11 on KVM)[https://youtu.be/7tqKBy9r9b4?si=xmM6vPTbu68kVxPr]  
* [Migrando pro Linux - migre Windows 11 do VirtualBox pro Virt-Manager e Ative ACELERAÇÃO GRÁFICA](https://youtu.be/WZ16GsFHSjM?si=LJoLVhI3c9iBqPWI)
* [Qemu: Data Exchange Between Windows and Linux - Share Folders](https://youtu.be/0hZU3vltZVM?si=wHv3id8czwD4u957)
* [This is a basic tutorial on how to virtualize Any Operating System using QEMU in PlayList](https://www.youtube.com/watch?v=cE6X2IrTzgU&list=PLmsony4NVQpxYb6B51t-uWWuGkph5rmf1)

Esses vídeos incluiem coisas que mencionei e alguns deles vão além disso, por exemplo, há algumas coisas que são possiveis fazer, mas é dificil explicar com palavras "como fazer", mas vão estender ainda mais as funcionalidades de computadores virtualizados com o Windows, um dos exemplos é usufruir de uma GPU dedicada por meio de passtrough.


## CONCLUSÃO
Não se trata mais de criar VMs, as informações que obteve até aqui cobriram essa etapa e algumas foram além disso. Então os links a seguir são para "tunar" suas estações Windows.

Resumo rápido:   
|:---|:---|
|Função	|Onde configurar|
|Copiar/colar texto|spice-vdagent (host + guest)|
|Ajuste de resolução|QXL + spice-vdagent|
|Copiar/colar arquivos|spice-webdavd|
|Canal de comunicação|virt-manager>Canal SPICE agent|

