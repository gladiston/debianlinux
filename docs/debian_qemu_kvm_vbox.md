# 🧭 Introdução

O **VirtualBox** é um hipervisor do tipo 2 amplamente utilizado para virtualização em desktops.  
Ele armazena os discos virtuais no formato **VDI (Virtual Disk Image)**, um contêiner eficiente e fácil de expandir, projetado pela Oracle.  
Embora o VDI funcione muito bem dentro do ecossistema do VirtualBox, ele **não é nativamente compatível** com hipervisores baseados em KVM, como o **QEMU**, **virt-manager**, **GNOME Boxes** ou **Cockpit Machines**.

O **QEMU/KVM**, por sua vez, utiliza preferencialmente o formato **QCOW2 (QEMU Copy On Write)**.  
Esse formato é mais avançado, permitindo recursos como:
- snapshots incrementais,
- compactação eficiente,
- encriptação e
- melhor desempenho com drivers VirtIO.

Assim, para migrar uma VM do VirtualBox para o QEMU/KVM, basta converter o disco de **VDI → QCOW2**, e depois criar uma nova VM apontando para o arquivo convertido.

---

## 🧱 Etapa 1 — Identificar o Disco VirtualBox

No VirtualBox, os discos das VMs geralmente ficam armazenados em:
```

~/VirtualBox VMs/<nome-da-vm>/<nome>.vdi

```

Exemplo:
```

~/VirtualBox VMs/win11-dx11/win11-dx11.vdi

````

Confirme o nome do arquivo `.vdi` e **encerre a VM** antes de prosseguir.

---

## 🔄 Etapa 2 — Converter o Arquivo VDI para QCOW2

No Linux, o pacote `qemu-utils` traz a ferramenta **`qemu-img`**, usada para conversão de discos entre diversos formatos.

Instale (se ainda não tiver):
```bash
sudo apt install qemu-utils -y
````

Agora, execute a conversão:

```bash
qemu-img convert -p -O qcow2 -o compat=1.1,cluster_size=1M,lazy_refcounts=on \
  ~/VirtualBox\ VMs/win11-dx11/win11-dx11.vdi ~/libvirt/images/win11-dx11.qcow2
```

### Explicando os parâmetros:

| Parâmetro                   | Função                                 |
| :-------------------------- | :------------------------------------- |
| `-p`                        | Exibe o progresso da conversão         |
| `-O qcow2`                  | Define o formato de saída como QCOW2   |
| `compat=1.1`                | Usa versão moderna e rápida do formato |
| `cluster_size=1M`           | Melhora desempenho de I/O              |
| `lazy_refcounts=on`         | Evita travamentos durante gravação     |
| `~/VirtualBox VMs/...vdi`   | Caminho do disco de origem             |
| `~/libvirt/images/...qcow2` | Caminho de destino do novo disco       |

Após o processo, você terá um arquivo QCOW2 pronto para uso no KVM.

---

## 🔍 Etapa 3 — Validar a Conversão

Execute:

```bash
qemu-img info ~/libvirt/images/win11-dx11.qcow2
```

Saída esperada:

```
file format: qcow2
virtual size: 64G (68719476736 bytes)
disk size: 28G
cluster_size: 1048576
lazy refcounts: true
compat: 1.1
```

---

## 🧹 (Opcional) Etapa 4 — Compactar o Arquivo QCOW2

Para reduzir o tamanho do disco, eliminando blocos vazios, use:

```bash
virt-sparsify --check-tmpdir=warn \
  ~/libvirt/images/win11-dx11.qcow2 ~/libvirt/images/win11-dx11-compact.qcow2
```

O arquivo `*-compact.qcow2` resultante pode substituir o original se desejar.

---

## ⚙️ Etapa 5 — Criar a VM no QEMU/KVM

### Método 1: via Virt-Manager (interface gráfica)

1. Abra o **Virt-Manager**.
2. Clique em **Criar nova máquina virtual**.
3. Escolha **“Importar imagem de disco existente”**.
4. Selecione o arquivo:

   ```
   ~/libvirt/images/win11-dx11.qcow2
   ```
5. Defina o sistema operacional convidado (ex: *Windows 11*).
6. Em **Dispositivo de disco**, selecione **VirtIO** (melhor desempenho).
7. Em **Interface de rede**, use **VirtIO (paravirtualizado)**.
8. Finalize a criação da VM.

---

### Método 2: via linha de comando (`virt-install`)

Se preferir linha de comando, use:

```bash
virt-install \
  --name win11-dx11 \
  --memory 8192 \
  --vcpus 4 \
  --os-variant win11 \
  --disk path=~/libvirt/images/win11-dx11.qcow2,format=qcow2,bus=virtio \
  --network network=default,model=virtio \
  --graphics spice \
  --boot uefi
```

> 💡 O parâmetro `--boot uefi` é importante para sistemas modernos (Windows 10/11).
> Certifique-se de que o pacote `OVMF` esteja instalado (`sudo apt install ovmf -y`).

---

## 🚀 Etapa 6 — Primeiro Boot e Ajustes

Após criar a VM:

* Inicie-a pelo Virt-Manager.
* Se for Windows, instale os **drivers VirtIO** (armazenamento, rede e vídeo).
* Verifique se o disco e rede estão funcionando normalmente.

---

## 🧩 Conclusão

A conversão de discos **VDI → QCOW2** é o caminho mais prático para migrar VMs do VirtualBox para o QEMU/KVM.
Com essa abordagem:

* você mantém todos os dados intactos,
* aproveita o desempenho nativo do KVM,
* e ainda ganha recursos avançados como snapshots e backup incremental.

Essa técnica é ideal tanto para **migrações definitivas** quanto para **testes de performance** em ambientes Linux modernos.

```

