# CRIAR E REMOVER UMA BRIDGE DE REDE COM BRIDGE-UTILS (UBUNTU 25.10 / DEBIAN 13)

## INTRODUÇÃO

Este guia explica, de forma prática e segura, como **criar e remover uma interface de rede do tipo *bridge*** (`br0`) no **Ubuntu 25.10** ou **Debian 13 (Trixie)**.  
O objetivo é permitir que **máquinas virtuais (QEMU/KVM, VirtualBox)** ou **containers** usem a **mesma rede física** do computador anfitrião, recebendo IP diretamente do **servidor DHCP** da rede local — como se fossem dispositivos físicos conectados ao roteador.

A *bridge* atua como uma “ponte virtual” entre a interface de rede física (`enp8s0`) e as interfaces virtuais das VMs.  
Assim, todas compartilham o mesmo segmento de rede, facilitando o acesso entre o host e as VMs e permitindo que elas obtenham IPs automáticos, façam parte do mesmo domínio e respondam a pings normalmente.

> 💡 Este procedimento é compatível tanto com ambientes **gráficos (virt-manager)** quanto **servidores headless (libvirt/QEMU puro)**.

Durante o processo:
- A interface física (`enp8s0`) será **transferida para a bridge**;
- O **IP passará a ser atribuído à `br0`** (e não mais à `enp8s0`);
- Você poderá **usar DHCP ou IP fixo** (exemplo incluso);
- Um **método de reversão** completo é apresentado ao final, com opção de restauração total do backup.

⚠️ **Importante:**  
Execute este guia **diretamente no terminal local** (não via SSH), pois a conexão pode cair temporariamente enquanto a nova bridge é criada.


## ENTENDENDO O AMBIENTE DO EXEMPLO
O ambiente usado neste guia é típico de um sistema limpo com suporte a virtualização (QEMU/KVM, VirtualBox, etc.).

- Distribuição: `Debian 13`
- Gerenciador de rede: NetworkManager (`nmcli`)
- Interface física: `enp8s0`
- Nome da conexão que aponta para Interface fisica: `Wired connection 1`
- Nome da bridge: `br0`
- Diretório de backups: `~/net-backups`
- O passo a passo descreve uma bridge que usa IP via DHCP, mas caso opte por IP Fixo, então o exemplo alternativo usa o IP `192.168.1.50/24`, o gateway será `192.168.1.1` e o DNS da rede será local com o IP `192.168.1.5`.
- Estado do arquivo `/etc/network/interfaces`:
  - Conteúdo atual:
    ```text
    source /etc/network/interfaces.d/*

    # The loopback network interface
    auto lo
    iface lo inet loopback
    ```
  - Interpretação: Apenas o loopback está definido. **Não há blocos para `enp8s0`**, portanto não existe conflito com o NetworkManager. **Nenhuma alteração é necessária nesse arquivo.**

## INSTALAÇÃO
Garante a presença do NetworkManager, utilitários de bridge e ferramentas de rede:  

```bash
sudo apt update
sudo apt install -y network-manager # provavelmente, já instalado
sudo apt install -y bridge-utils dnsmasq-base ovmf isc-dhcp-client iproute2
```

## BACKUP DA CONFIGURAÇÃO ORIGINAL
Vamos ser cautelosos e fazer um backup de nossa configuração de rede atual, assim se algo der errado, restauramos.  

Escolha uma pasta para guardar o backup dessa configuração de rede, minha sugestão é **~/net-backups**, mas você pode escolher outra:  
```bash
export BKP=~/net-backups
```

Depois criamos a pasta:
```bash
mkdir -p $BKP
```

Depois, vamos definir o nome do arquivo de backup com o seu nome:
```bash
export BKPFILE=$BKP/netcfg-bridge-$(date +%F_%H%M%S).tgz 
```

Depois, geramos um relatorio em formato .txt e usamos o tar para compactar os arquivos de configurações atuais no local de destino:  
```bash
nmcli general status >$BKPFILE.txt
nmcli con show  >>$BKPFILE.txt
sudo tar -C / -czf $BKPFILE   etc/network etc/NetworkManager etc/systemd/network
sudo chown -R $USER:$USER $BKP
```
E então verifique os arquivos que foram gerado:  
```bash
$ ls -lh $BKP/
```
E verá algo similar a isso:
```
-rw-r--r-- 1 gsantana gsantana 2,3K out 23 16:57 netcfg-bridge-2025-10-23_165719.tgz
-rw-rw-r-- 1 gsantana gsantana  399 out 23 17:10 netcfg-bridge-2025-10-23_165719.tgz.txt
```
Como pôde ver, a composição dos arquivos que foram compactados são apenas alguns arquivos textos pequenos, quase nada em termos de espaço. Também criamos uma arquivo `.txt` que armazena os resultados dos comandos que você viu no inicio desse tutorial e mais tarde será possível comparar o antes e o depois. Se você não vê nenhum dos dois arquivos, o `.tgz` e o `.txt` ou estão vazios, então algo deu errado nos passos anteriores e é melhor revisá-los e não prosseguir.

## VERIFICAÇÃO DO AMBIENTE
Comandos úteis de verificação, primeira observamos se `enp8s0` esta presente:
```bash
$ ip -br link | grep -E 'enp|eth'
enp8s0           UP             74:56:3c:f0:18:b4 <BROADCAST,MULTICAST,UP,LOWER_UP> 
```
Depois, execute:
```bash
$ nmcli device status | grep enp8s0
enp8s0  ethernet  conectado               Wired connection 1 
```
Note que acima, está descrito tanto a interface fisica `enp8s0` como também o nome da conexão `Wired connection 1`, fique atento porque você precisará adaptar esses nomes ao seu ambiente que pode ser diferente do meu.  

Após a instalação e a verificação, certifique-se de que o NetworkManager está ativo:
```bash
sudo systemctl enable --now NetworkManager
```
Depois observe o status, execute:
```bash
$ sudo nmcli general status
STATE      CONNECTIVITY  WIFI-HW  WIFI        WWAN-HW  WWAN        METERED          
conectado  completa      missing  habilitado  missing  habilitado  não (adivinhado) 
```
Observe suas conexões atuais:
```bash
$ nmcli con show 
NAME                UUID                                  TYPE      DEVICE 
Wired connection 1  aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee  ethernet  enp8s0 
lo                  bbbbbbbb-cccc-dddd-eeee-ffffffffffff  loopback  lo  
```


## CRIANDO A BRIDGE COM O NETWORK-MANAGER (nmcli)
Procedimento recomendado para criar a bridge `br0` e anexar `enp8s0` como escrava, usando DHCP na bridge.

Vamos desativar a conexão antiga cujo nome da interface é `enp8s0` e cuja conexão tem o nome de `Wired connection 1` :
```bash
sudo nmcli con down "Wired connection 1"
```

Vamos criar a conexão da bridge:
```bash
$ sudo nmcli con add type bridge ifname br0 con-name br0 ipv4.method auto ipv6.method ignore
```
O resultado espera do é:
```
A conexão “br0” (aa0748de-5a0d-4d45-b908-9e9f2c788465) foi adicionada com sucesso.
```

Vamos vincular a interface física `enp8s0` como escrava da bridge:
```bash
sudo nmcli con add type bridge-slave ifname enp8s0 con-name br0-port-enp8s0 master br0
```
O resultado esperado é:
```
A conexão “bridge-slave-enp8s0” (7daddf82-c9cb-4e44-9d26-097e9e9c7604) foi adicionada com sucesso.
```

E depois ativamos a conexão:
```bash
sudo nmcli con up br0
```
O resultado esperado é:
```
Conexão ativada com sucesso (controller waiting for ports) (caminho D-Bus ativo: /org/freedesktop/NetworkManager/ActiveConnection/4)
```

Agora, vamos a verificação, o comando abaixo mostrará tudo relacionado a `br0`:
```bash
nmcli -p con show br0
```
Use "q" para sair da listagem do comando acima, agora vamos ver o IP da nossa `br0`:
```bash
ip -br a show br0
```
Se o resultado for:
```
br0              DOWN    
```
Então significa que a bridge br0 existe, mas está DOWN, ou seja, a interface foi criada, porém ainda não está ativa ou não recebeu IP. Então vamos tentar manualmente dessa vez:
```bash
sudo ip link set br0 up
```
Será que dessa vez esta ativa? Vamos confirmar:
```bash
ip -br a show br0
```
Se estiver ativa (UP), então vamos ver se é capaz de conectar-se a outros computadores:
```bash
ping -c3 1.1.1.1 && ping -c3 8.8.8.8
```

5) Ajuste para IP estático (opcional):
```bash
sudo nmcli con modify br0 ipv4.method manual   ipv4.addresses "192.168.1.50/24"   ipv4.gateway "192.168.1.1"   ipv4.dns "192.168.1.5"

sudo nmcli con up br0
```

## VERIFICAÇÃO PÓS-BRIDGE (teste de funcionalidade)
Após reiniciar o sistema, confirme que a bridge está ativa e funcional:

Vamos listar as conexões gerenciadas pelo NetworkManager:
```bash
nmcli con show
```

Vamos verificar o estado da bridge e da interface física:
```bash
nmcli device status
```
Saída esperada:
```
DEVICE   TYPE      STATE      CONNECTION
br0      bridge    conectado  br0
enp8s0   ethernet  conectado  bridge-slave-enp8s0
```

Vamos confirmar a associação física entre bridge e interface:
```bash
ip link show br0
```

Saída esperada:
```
3: br0: <BROADCAST,MULTICAST,UP,LOWER_UP> ...
    bridge-slave enp8s0
```

Vamos validar endereço IP e gateway:
```bash
ip -br addr show br0
ip route | grep default
```

Agora, o teste de conectividade:
```bash
ping -c3 1.1.1.1
ping -c3 deb.debian.org
```

### Verificar via bridge-utils (opcional):
```bash
sudo brctl show
```
Saída esperada:
```
bridge name bridge id        STP enabled interfaces
br0         8000.74563cf018b4 no          enp8s0
```

Se todos os testes retornarem resultados positivos, a bridge `br0` está ativa e totalmente operacional.

## REVERSÃO SIMPLESMENTE COM O NETWORK-MANAGER (se necessário)
Para desfazer a bridge e voltar a usar a `enp8s0` diretamente, para dar inicio ao processo, vamos desativar a interface `br0`:
```bash
$ sudo nmcli con down br0 
Conexão “br0” desativada com sucesso (caminho D-Bus ativo: /org/freedesktop/NetworkManager/ActiveConnection/6)
```

Depois vamos apagá-la:
```bash
$ sudo nmcli con delete br0 
A conexão “br0” (aa0748de-5a0d-4d45-b908-9e9f2c788465) foi excluída com sucesso.
```
Em seguida, remova apenas as conexões relacionadas à bridge `br0`, evitando apagar todas as bridges do sistema, digamos por exemplo que haja uma bridge criada com o nome `br0-port-enp8s0`, então repetimos o processo:
O comando abaixo irá desativar a porta recem criada:
```bash
sudo nmcli con down br0-port-enp8s0
sudo nmcli con delete br0-port-enp8s0 
```

> Se seu desejo for apagar todas as conexões de tipo bridge-slave do NetworkManager:  
> nmcli -t -f NAME,TYPE con show | awk -F: '$2=="bridge-slave"{print $1}' | xargs -r -I{} sudo nmcli con delete "{}"  
> **IMPORTANTE**: Este comando apaga todos sem distinção, só use se souber exatamente o que esta fazendo.

Vamos listar as conexão que temos com o nome da original `xxx`, execute:
```bash
$ nmcli con show | grep "Wired connection 1"
Wired connection 1     f35944fc-aec3-4d0e-9ded-76c199678b97  ethernet  enp8s0 
Wired connection 1     55f6aebb-2d5e-46a5-8562-560211ea3a11  ethernet  --     
Wired connection 1     23471efa-cdf7-4d97-9d90-c822445d7c96  ethernet  --     
```
Perceba que apenas a primeira tem um vinculo com `enp8s0`, as demais não deveriam existir, vamos excluí-las:
Excluindo a conexão com id `55f6aebb-2d5e-46a5-8562-560211ea3a11`:
```bash
$ sudo nmcli con delete 55f6aebb-2d5e-46a5-8562-560211ea3a11
A conexão “Wired connection 1” (55f6aebb-2d5e-46a5-8562-560211ea3a11) foi excluída com sucesso.
```
Excluindo a conexão com id `23471efa-cdf7-4d97-9d90-c822445d7c96`:
```bash
$ sudo nmcli con delete 23471efa-cdf7-4d97-9d90-c822445d7c96
A conexão “Wired connection 1” (23471efa-cdf7-4d97-9d90-c822445d7c96) foi excluída com sucesso.
```


Vinculamos um nome de conexão `Wired connection 1` a interface `enp8s0`:
```bash
$ sudo nmcli con add type ethernet ifname enp8s0 con-name "Wired connection 1" ipv4.method auto
A conexão “enp8s0” (55f6aebb-2d5e-46a5-8562-560211ea3a11) foi adicionada com sucesso.
```
Onde está `Wired connection 1` é o nome original, mas você pode usar outro nome como `conexao1` para ficar mais simples.

E finalizamos com a ativação da interface `enp8s0`:
```bash
sudo nmcli con up enp8s0
Conexão ativada com sucesso (caminho D-Bus ativo: /org/freedesktop/NetworkManager/ActiveConnection/11)
```

## RESTAURAÇÃO BACKUP DA CONFIGURAÇÃO DE REDE
Se está lendo isso é porque algo deu muito errado e reversão usando o NETWORK-MANAGER não funcionou e agora precisa voltar a configuração de rede original, não é mesmo?
Sem problemas, foi por isso que fizemos o backup antes de dar inicio aos procedimentos.  

Primeiro vamos apontar a variavel para a localização de seus backups:  
```bash
export BKP=~/net-backups
```

Depois liste todos os seus backups e escoha o que for mais apropriado:  
```bash
$ ls -1 $BKP/
netcfg-2025-10-22_165719.tgz
netcfg-bridge-2025-10-23_165719.tgz
```

Determinado qual o arquivo de backup desejado, execute:
```bash
sudo tar -C / -xzf $BKP/netcfg-bridge-2025-10-23_165719.tgz
```

Os arquivos de configuração serão substituidos pelo backup, porém o gerenciador que os controla precisa ser reiniciado:    
```bash
sudo systemctl restart NetworkManager 
```

Para confirmar que a rede foi restaurada ao original, execute:
```bash
nmcli general status
nmcli con show 
ip -br a 
```
E compare seus resultados com o que havia no backup lendo o arquivo `netcfg-bridge-2025-10-23_165719.tgz.txt`.  
Se os resultados dos comandos acima deram baterem com os resultados do arquivo `netcfg-bridge-2025-10-23_165719.tgz.txt`, então você restaurou sua rede com sucesso.  

## CONCLUSÃO

Ao concluir este guia, você criou uma **bridge de rede funcional (`br0`)** totalmente integrada ao **NetworkManager**, permitindo que suas **máquinas virtuais ou containers** utilizem a mesma rede física do host.  
Esse método garante que cada VM possa obter **endereços IP reais da sua LAN**, facilitando comunicações diretas, compartilhamento de arquivos, e acesso a serviços como se fossem máquinas físicas.

Você também aprendeu a:
- Fazer **backup das configurações de rede** antes de qualquer modificação;
- Criar e vincular uma **interface física (`enp8s0`)** à bridge;
- Usar tanto **DHCP quanto IP estático** na `br0`;
- **Verificar o estado da bridge** e testar conectividade;
- **Reverter** as alterações com segurança caso algo dê errado.

> 💡 **Dica final:**  
> Mantenha o arquivo de backup em um local externo (como um pendrive ou pasta de rede).  
> Assim, caso perca a conectividade, será mais fácil restaurar sua configuração original.

Com a bridge configurada corretamente, seu ambiente está pronto para hospedar **máquinas virtuais com desempenho de rede real**, ideal para testes de servidores, clusters e laboratórios de virtualização.  
Agora você pode seguir adiante e integrar a `br0` em suas VMs pelo **virt-manager**, **virsh** ou **QEMU** diretamente.
