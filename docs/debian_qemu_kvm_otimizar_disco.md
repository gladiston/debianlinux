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

### Opção B — Criar cópia **compactada**

```bash
cd ~/libvirt/images/
sudo virt-sparsify --compress win2k25.qcow2 win2k25-optimized.qcow2
```

Ao final, compare tamanhos:

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

---

## Passo 4 — Ajustes no libvirt

Para manter a imagem enxuta continuamente, propague o **DISCARD/UNMAP**:

```xml
<disk type='file' device='disk'>
  <driver name='qemu' type='qcow2' cache='none' io='native'
          discard='unmap' detect_zeroes='unmap'/>
  <source file='/home/gsantana/libvirt/images/win2k25.qcow2'/>
  <target dev='vda' bus='virtio'/>
</disk>
```

**Boas práticas:**

* `cache='none'` e `io='native'` oferecem desempenho previsível.
* Habilite TRIM periódico no guest (fstrim.timer em Linux ou ReTrim no Windows).
* Prefira TRIM agendado, não contínuo, em workloads com alto I/O aleatório.

---

## Passo 5 — Snapshots e cadeias de backing

* Listar snapshots:

  ```bash
  qemu-img snapshot -l ~/libvirt/images/win2k25.qcow2
  ```
* Remover snapshot interno:

  ```bash
  qemu-img snapshot -d <ID> ~/libvirt/images/win2k25.qcow2
  ```
* Flatten (quando há overlay/backing):

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

## Boas práticas operacionais

* Sempre mantenha backups antes de qualquer otimização.
* Execute com a VM desligada.
* Padronize parâmetros QCOW2 conforme o tipo de carga:

  * **cluster_size=1M** → I/O sequencial intenso (DBs, backup).
  * **cluster_size=128K–256K** → I/O aleatório pequeno (sistemas).
* Monitore espaço e fragmentação periodicamente com `qemu-img info`.

---

Quer que eu te entregue esse conteúdo como um arquivo `.md` pronto para ser adicionado ao seu repositório `debianlinux` (com cabeçalho e metadados padrão do projeto)?
