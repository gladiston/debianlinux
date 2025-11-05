# Otimizando imagens de VM (QCOW2) no qemu+kvm
O QCOW2 é um formato copy-on-write com recursos como snapshots, compressão e alocação sob demanda. Esses recursos trazem overhead e, com o tempo, geram fragmentação interna. Mas máquinas Windows são muito mais afetadas do que as demais porque o Windows gera memória virtual, arquivos temporários a todo instante. Então o link a seguir descreve como podemos otimizar e compactar o disco virtual para que o desempenho - especialmente para VMs Windows - fique sempre máximizado.

Em nosso exemplo anterior criamos um arquivo `.qcow2` que simboliza o disco virtual para a nossa VM, ela chama-se:   
```
~/libvirt/images/win2k25.qcow2
```
Usaremos ela como referencia daqui em diante, isto é, o que se aplica a ela, se aplica a qualquer VM que você possua.  
Porém, de maneira geral, aplica-se mais a VMs Windows porque elas desfragmentam e perdem performance com disco mais que outros sistemas operacionais.  

> Objetivo: reduzir espaço em disco, melhorar I/O e manter a integridade dos discos virtuais. Em QCOW2 não há “defrag” online como em sistemas de arquivos; a otimização real combina liberar espaço dentro do guest (TRIM/zerar espaços) e reescrever a imagem (convert/sparsify) para reorganizar clusters e remover fragmentação.

---

## O que é e por que otimizar

- **QCOW2** é um formato copy-on-write com recursos como snapshots, compressão e alocação sob demanda. Esses recursos trazem overhead e, com o tempo, geram fragmentação interna.
- **Por que otimizar**
  - Reduzir o tamanho físico no datastore (aproveitar sparsity/compressão).
  - Reordenar clusters para I/O mais previsível.
  - Remover snapshots e cadeias de backing, simplificando operações.
  - Corrigir metadados (refcounts) e inconsistências.
- **Como otimizar**
  - Dentro do guest: garantir que o espaço “livre” esteja realmente marcado como tal (TRIM/UNMAP ou zerado).
  - No host: converter/sparsificar para reescrever a imagem de forma densa e sequencial, e ajustar parâmetros como cluster size.

---

## Quando aplicar

- Após grandes exclusões de dados no guest (desinstalações, limpeza).
- Depois de criar/remover muitos snapshots.
- Periódico em ambientes com alto churn de dados (trimestral/semestral).
- Antes de migrar para outro armazenamento ou consolidar cadeias de backing.

---

## Pré-requisitos e cuidados

- Faça backup do arquivo QCOW2.
- Desligue a VM antes de operações offline de disco.
- Garanta espaço livre suficiente no FS do host para gerar um arquivo otimizado.
- Exemplo de caminho da sua VM:
  - Arquivo: **~/libvirt/images/win2k25.qcow2**
- Parar a VM pelo libvirt:
  ```bash
  sudo virsh shutdown win2k25
  sudo virsh destroy win2k25  # somente se não desligar graciosamente
  ```

---

## Passo 1 — Dentro do guest: liberar espaço

> A ideia é tornar o espaço realmente “descartável” para que a etapa offline possa compactar.

### Windows (recomendado para a VM “win2k25”)

1. Confirmar TRIM no guest e no disco (virtio-scsi/virtio-blk com discard):
   - No XML do libvirt, habilite discard/detect_zeroes (ver Passo 4).
2. ReTRIM do volume:
   ```powershell
   Optimize-Volume -DriveLetter C -ReTrim -Verbose
   ```
3. Opcional (maximiza compactação offline ao zerar espaços livres):
   ```powershell
   sdelete64 -z C:
   ```

### Linux (se aplicável em outras VMs)

- TRIM imediato:
  ```bash
  sudo fstrim -av
  ```
- TRIM periódico:
  ```bash
  sudo systemctl enable --now fstrim.timer
  ```

> Após TRIM/zeragem, desligue a VM para executar a otimização offline com segurança.

---

## Passo 2 — Verificar integridade da imagem

- Corrigir metadados antes de converter evita propagar problemas:
  ```bash
  qemu-img check -r all /home/gsantana/libvirt/images/win2k25.qcow2
  ```

---

## Passo 3 — Otimização offline (host)

> Estratégias: conversão com reescrita “limpa” e/ou sparsify. A conversão é a forma mais efetiva de “desfragmentar”.

### Opção A: Converter para nova QCOW2 otimizada

- Conversão com compressão e parâmetros recomendados:
  ```bash
  qemu-img convert -p \
    -O qcow2 \
    -c \
    -o compat=1.1,cluster_size=1M,lazy_refcounts=on,preallocation=metadata \
    /home/gsantana/libvirt/images/win2k25.qcow2 \
    /home/gsantana/libvirt/images/win2k25-optimized.qcow2
  ```

- Notas de tuning:
  - **-c**: compressão na conversão; requer que o guest tenha TRIM/zeros.
  - **cluster_size=1M**: ótimo para I/O sequencial e menos metadados; para workloads com muitas gravações pequenas, avalie 128K/256K.
  - **preallocation=metadata**: reduz fragmentação futura (aloca metadados upfront).
  - **lazy_refcounts=on**: melhora write throughput; após falhas abruptas de host, use qemu-img check.

- Trocar o disco no libvirt (com a VM desligada):
  - Atualize o XML para apontar para win2k25-optimized.qcow2 ou renomeie com swap atômico:
    ```bash
    mv /home/gsantana/libvirt/images/win2k25.qcow2 \
       /home/gsantana/libvirt/images/win2k25.qcow2.bak
    mv /home/gsantana/libvirt/images/win2k25-optimized.qcow2 \
       /home/gsantana/libvirt/images/win2k25.qcow2
    ```
  - Suba a VM e valide boot; remova o .bak após validação.

### Opção B: virt-sparsify (libguestfs)

- Gera uma cópia otimizada com compressão:
  ```bash
  virt-sparsify --compress \
    /home/gsantana/libvirt/images/win2k25.qcow2 \
    /home/gsantana/libvirt/images/win2k25-optimized.qcow2
  ```
- In-place (usa espaço temporário; mais rápido de operar, mas cuidado):
  ```bash
  virt-sparsify --in-place --compress /home/gsantana/libvirt/images/win2k25.qcow2
  ```

> Dica: Prefira a conversão (Opção A) quando quiser também “achatar” snapshots e reorganizar clusters de forma mais previsível.

---

## Passo 4 — Ajustes do libvirt para manutenção contínua

> Propagar DISCARD/UNMAP do guest para o host garante que fstrim/Optimize-Volume tenha efeito real.

- Exemplo de disco em virtio com discard:
  ```xml
  <disk type='file' device='disk'>
    <driver name='qemu' type='qcow2' cache='none' io='native' discard='unmap' detect_zeroes='unmap'/>
    <source file='/home/gsantana/libvirt/images/win2k25.qcow2'/>
    <target dev='vda' bus='virtio'/>
  </disk>
  ```
- Boas práticas:
  - **cache='none'** e **io='native'** para coerência e throughput estável sob KVM.
  - Habilite TRIM periódico no guest (Windows ReTrim agendado ou fstrim.timer em Linux).
  - Evite “discard” contínuo na opção de montagem em workloads com I/O aleatório intenso; TRIM periódico costuma ser melhor custo/benefício.

---

## Passo 5 — Snapshots e cadeias de backing

- Listar snapshots internos:
  ```bash
  qemu-img snapshot -l /home/gsantana/libvirt/images/win2k25.qcow2
  ```
- Remover snapshots internos (se existentes):
  ```bash
  qemu-img snapshot -d <ID> /home/gsantana/libvirt/images/win2k25.qcow2
  ```
- “Flatten” (quando há overlay/backing):
  ```bash
  qemu-img convert -p -O qcow2 \
    overlay.qcow2 flattened.qcow2
  ```
- Benefícios:
  - Reduz latência em leituras que atravessariam a cadeia.
  - Elimina dependências e simplifica operações futuras.

---

## Validação e métricas

- Tamanho lógico vs. físico:
  ```bash
  du -h /home/gsantana/libvirt/images/win2k25*.qcow2
  qemu-img info /home/gsantana/libvirt/images/win2k25.qcow2
  ```
- Mapa de alocação (sparsity/fragmentação):
  ```bash
  qemu-img map --output=json /home/gsantana/libvirt/images/win2k25.qcow2 | jq .
  ```
- Benchmark sintético:
  ```bash
  qemu-img bench -c 4k -d 1G -f qcow2 /home/gsantana/libvirt/images/win2k25.qcow2
  ```

---

## Boas práticas operacionais

- Sempre backup antes de conversões/rebases.
- Executar com a VM desligada para consistência de FS.
- Monitorar periodicamente espaço e I/O do datastore.
- Padronizar parâmetros de QCOW2 (compat, cluster_size, preallocation) conforme o perfil de workload:
  - Workloads com grandes sequenciais (backup, ingest, DB com IO grande): cluster_size=1M.
  - Workloads com I/O pequeno/aleatório: cluster_size=128K–256K pode reduzir write amplification.

---

## Checklist rápido (resumo prático)

1. Dentro do guest (Windows):
   - Optimize-Volume -ReTrim
   - Opcional: sdelete64 -z C:
2. Host (VM desligada):
   - qemu-img check -r all win2k25.qcow2
   - qemu-img convert -p -O qcow2 -c -o compat=1.1,cluster_size=1M,lazy_refcounts=on,preallocation=metadata win2k25.qcow2 win2k25-optimized.qcow2
   - Swap do arquivo otimizado e validação de boot
3. Libvirt:
   - discard='unmap', detect_zeroes='unmap', cache='none', io='native'
   - TRIM periódico no guest
