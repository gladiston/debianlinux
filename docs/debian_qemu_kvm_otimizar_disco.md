# Otimizando imagens de VM (QCOW2) no qemu+kvm

O **QCOW2** é um formato *copy-on-write* que oferece recursos como snapshots, compressão e alocação sob demanda. Com o tempo, esses recursos introduzem fragmentação interna e perda de desempenho. Máquinas Windows são especialmente afetadas, pois criam e apagam arquivos temporários e de paginação continuamente.

Este guia mostra como **otimizar e compactar** discos QCOW2, mantendo desempenho máximo e reduzindo o espaço ocupado — **sem precisar executar comandos dentro da VM**.

> **Objetivo:** reduzir o tamanho em disco, melhorar I/O e preservar a integridade das imagens virtuais.

---

> ⚠️ **Nota sobre o termo “compactar”**
>
> Neste artigo, as palavras **“compactar”** ou **“comprimir”** não se referem à compressão de dados como `zip`, `gzip` ou `bzip2`.
> Aqui, elas indicam o **processo de redução do tamanho físico ocupado por uma imagem QCOW2** após a remoção de blocos não utilizados e reorganização dos dados no disco virtual.
> Em outras palavras, “compactar” significa **otimizar o espaço interno** da imagem, e não aplicar compressão de dados com perda ou custo de CPU.

---

## O que é e por que otimizar

* **Por que otimizar**

  * Reduz o tamanho físico no datastore (melhor aproveitamento de espaço).
  * Reorganiza clusters e metadados, melhorando desempenho de I/O.
  * Remove snapshots antigos e cadeias de backing desnecessárias.
  * Corrige metadados e inconsistências de refcount.

* **Como otimizar**

  * No host, usando ferramentas como `qemu-img` e `virt-sparsify` para reescrever ou enxugar a imagem.
  * Ajustando parâmetros (compat, cluster_size, lazy_refcounts) para equilibrar desempenho e espaço.

---

## Quando aplicar

* Após exclusões grandes de dados no guest.
* Depois de criar/remover vários snapshots.
* Periodicamente em ambientes com muita escrita.
* Antes de migrar ou consolidar VMs em outro storage.

---

## Ajustes no libvirt (via virt-manager)

Os ajustes mencionados a seguir foram feitos nos passos anteriores deste guia, no entanto, caso tenha caído nesta página por qualquer outra razão, o que iremos fazer é conferir se o nosso disco virtual foi ajustado para ter máxima performance com o Windows, esses ajustes garantem que operações de “descartar blocos” (TRIM/UNMAP) do convidado cheguem até o hospedeiro, permitindo que futuras exclusões dentro da VM liberem espaço real.

### 🔧 Pelo virt-manager (GUI)

1. **Abra o virt-manager** e selecione a VM desejada.
2. Clique no ícone **⚙️ “i” (Mostrar detalhes da máquina virtual)**.
3. No painel esquerdo, clique em **VirtIO Disk (vda)** — ou o nome do disco principal.
4. Expanda **Advanced options** (Opções avançadas).
5. Configure:

   * **Cache mode:** `none`
   * **IO mode:** `native`
   * **Discard:** `unmap`
   * **Detect zeroes:** `unmap` *(ou `on`, se `unmap` não estiver disponível)*
6. Clique em **Aplicar** e **OK**.

> Se sua interface do virt-manager não mostrar as opções *Discard* ou *Detect zeroes*, use a edição em XML conforme abaixo.

### 🧩 Editando o XML manualmente

1. Ainda na tela de **Detalhes da VM**, clique em **Overview** → **XML** (alternador no canto inferior).
2. Localize o bloco `<disk …>` e provavelmente estará assim:
```xml
<disk type="file" device="disk">
  <driver name="qemu" type="qcow2" cache="none" discard="unmap"/>
  (...)
</disk>
```
Onde vocÊ vê:  
> <driver name="qemu" type="qcow2" cache="none" discard="unmap"/>

Sugere-se acrescentar também `io='native' detect_zeroes='unmap'`, ficando assim:  
> <driver name='qemu' type='qcow2' cache='none' discard="unmap"  **io='native' detect_zeroes='unmap'**/>

3. **Salve** as alterações.
4. Inicie a VM normalmente — as novas flags serão aplicadas no próximo boot, caso elas não não funcionem, reverta as alterações. Essas alterações são especificas para disco usando api "VirtIO" e possivelmente você não as utilizou quando criou sua VM.   


## Pré-requisitos e cuidados

* Faça **backup** da imagem QCOW2.
* Certifique-se de que a **VM esteja desligada**.
* Garanta espaço suficiente no filesystem do host.

Exemplo de caminho da VM:

```
~/libvirt/images/win2k25.qcow2
```

Parar a VM:

```bash
sudo virsh shutdown win2k25
sudo virsh destroy win2k25  # se necessário
```

---

## Passo 1 — Verificar tamanho da imagem

```bash
cd ~/libvirt/images/
ls -lh *.qcow2
```

Exemplo:

```
-rw------- 1 root kvm 28G nov  5 14:38 win2k25.qcow2
```

---

## Passo 2 — Verificar integridade

```bash
sudo qemu-img check -r all ~/libvirt/images/win2k25.qcow2
```

Saída típica:

```
No errors were found on the image.
335077/3276800 = 10.23% allocated, 17.39% fragmented, 0.00% compressed clusters
Image end offset: 29463674880
```

---

## Passo 3 — Otimização com virt-sparsify

A ferramenta `virt-sparsify` do pacote `libguestfs-tools` remove blocos não utilizados e pode reduzir o tamanho físico da imagem.

### Opção A — Otimização **in-place** (mantém o mesmo arquivo)

```bash
cd ~/libvirt/images/
sudo virt-sparsify --in-place win2k25.qcow2
```

> Modifica o arquivo existente, liberando espaço não usado.  
> Não cria cópia nova, é mais rápida, mas requer espaço temporário proporcional.
> Essa é a melhor opção para mim, ela não diminui o arquivo, mas fez a optimização de `Trimming` que é suficiente para a performance do Windows.  

### Opção B — Criar cópia **compactada**

```bash
cd ~/libvirt/images/
sudo virt-sparsify --compress win2k25.qcow2 win2k25-optimized.qcow2
```
virt-sparsify: aviso: Pode não haver espaço livre suficiente em /tmp.
Você talvez precise definir a variável TMPDIR para apontar para um diretório com mais espaço disponível.

Máximo necessário: 200,0 GB. Livre: 15,4 GB. Pode ser necessário mais 184,6 GB.

Observe que isso é uma superestimativa. Se o disco do sistema convidado estiver cheio de dados, provavelmente não será necessário tanto espaço livre.

Você pode ignorar este aviso ou transformá-lo em uma falha obrigatória usando a opção:
--check-tmpdir=(ignore|continue|warn|fail)
Consulte o manual virt-sparsify(1) para mais detalhes.
```
Neste caso, ele está mostrando que irá recriar o disco e que precisará de 200GB! E se você não tiver isso em `/tmp` então não conseguirá completar o processo. Ele irá mostrar uma barra de progresso e uma estimativa de tempo. Essa opção **"B"** é praticamente inviável para mim por causa do tempo, o virt-sparsify é tão esperto em otimizar e conhecer o sistema de arquivos que faz muitas operações demoradas e com isso demora bastante. Com ele,agendá-lo em dias/horários que você não precise usar o máquinas virtuais, quicá o comuputador.  Mas compensa na redução de tamanho, compare:
```bash
ls -lh *.qcow2
```

Exemplo:

```
-rw-r--r-- 1 root kvm 16G nov  5 14:52 win2k25-optimized.qcow2
-rw------- 1 root kvm 28G nov  5 14:38 win2k25.qcow2
```

Substitua a imagem original com segurança (swap atômico):

```bash
mv win2k25.qcow2 win2k25.qcow2.bak
mv win2k25-optimized.qcow2 win2k25.qcow2
```

Depois de validar o boot, remova o `.bak`.

### Opção C — Criar cópia **compactada** usando o 'qemu-img'
O utilitário `qemu-img` é algo mais burro que o `virt-sparsify` copiando os dados de um disco velho para o novo e com isso ignora dados vazios e fará a mesma coisa coisa que a opção "B", porém em 6 minutos, veja como funciona: 
```bash
cd ~/libvirt/images/
qemu-img convert -p \
  -O qcow2 \
  -c \
  -o compat=1.1,cluster_size=1M,lazy_refcounts=on,preallocation=metadata \
  win2k25.qcow2 win2k25-optimized.qcow2
```
Ele vai gerar uma novo arquivo `win2k25-optimized.qcow2` otimizado e compactado. Daí repetimos o swap atômico:  
```bash
mv win2k25.qcow2 win2k25.qcow2.bak
mv win2k25-optimized.qcow2 win2k25.qcow2
```
E novamente, depois de validar o boot, remova o `.bak`.
  
---

## Passo 4 — Snapshots e cadeias de backing

* Listar snapshots:

  ```bash
  qemu-img snapshot -l ~/libvirt/images/win2k25.qcow2
  ```
* Remover snapshot interno:

  ```bash
  qemu-img snapshot -d <ID> ~/libvirt/images/win2k25.qcow2
  ```
* “Flatten” (quando há overlay/backing):

  ```bash
  qemu-img convert -p -O qcow2 overlay.qcow2 flattened.qcow2
  ```

---

## Validação e métricas

```bash
du -h ~/libvirt/images/win2k25*.qcow2
qemu-img info ~/libvirt/images/win2k25.qcow2
```

Ver mapa de alocação:

```bash
qemu-img map --output=json ~/libvirt/images/win2k25.qcow2 | jq .
```

Benchmark rápido:

```bash
qemu-img bench -c 4k -d 1G -f qcow2 ~/libvirt/images/win2k25.qcow2
```

---
## Permissões nos arquivos
Se fez ajustes e criou novos arquivos, então é razoável conferir se as permissões estão corretas, execute:  
```bash
sudo find ~/libvirt -type f -exec chmod 666 {} \; -o -type d -exec chmod 777 {} \;
```

## Boas práticas operacionais

* Sempre mantenha backups antes de qualquer otimização.
* Execute com a VM desligada.
* Padronize parâmetros QCOW2 conforme o tipo de carga:

  * **cluster_size=1M** → I/O sequencial intenso (DBs, backup).
  * **cluster_size=128K–256K** → I/O aleatório pequeno (sistemas).
* Monitore espaço e fragmentação periodicamente com `qemu-img info`.

