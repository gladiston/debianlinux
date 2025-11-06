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
Essa conversão gerará um arquivo de mesmo tamanho que o original, porém no formato qcow2.  

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

Para verificar integridade, execute:

```bash
sudo qemu-img check -r all ~/libvirt/images/win11-dx11.qcow2
```
Saída esperada:
```
No errors were found on the image.
82174/122880 = 66.87% allocated, 0.00% fragmented, 0.00% compressed clusters
Image end offset: 86170927104
```


Uma vez validado, ou seja **No errors were found on the image.** então podemos obter as informações do disco:  
```bash
qemu-img info ~/libvirt/images/win11-dx11.qcow2
```
Saída esperada:
```
image: win11-dx11.qcow2
file format: qcow2
virtual size: 120 GiB (128849018880 bytes)
disk size: 80.3 GiB
cluster_size: 1048576
Format specific information:
    compat: 1.1
    compression type: zlib
    lazy refcounts: true
    refcount bits: 16
    corrupt: false
    extended l2: false
Child node '/file':
    filename: win11-dx11.qcow2
    protocol type: file
    file length: 80.3 GiB (86170927104 bytes)
    disk size: 80.3 GiB
```
Isso indica que podemos prosseguir.  

---

## 🧹 Etapa 4 — Compactar o Arquivo QCOW2

Para reduzir o tamanho do disco, eliminando blocos vazios, use:

```bash
$ sudo virt-sparsify --in-place ~/libvirt/images/win2k25.qcow2
[   2.6] Trimming /dev/sda1
[   2.7] Trimming /dev/sda2
[   4.0] Trimming /dev/sda3
[   4.1] Sparsify in-place operation completed with no errors
```
O comando realiza uma desfragmentação lógica da imagem QCOW2, consolidando os espaços vazios para o final do arquivo enquanto mantém seu tamanho original. Durante este processo, operações de trimming sinalizam ao formato QCOW2 quais blocos estão realmente vazios, permitindo que o Windows reconheça este espaço como efetivamente disponível para novas alocações de arquivo. Isso otimiza significativamente a performance da VM porque, com os espaços vazios consolidados e sinalizados, o SO convidado pode alocar novos arquivos sem que o QEMU precise realizar custosas operações de growing — o processo onde a imagem QCOW2 precisa ser expandida para armazenar mais dados, consumindo recursos e aumentando latência. Embora o arquivo permaneça no mesmo tamanho, essa otimização de trimming é suficiente para melhorar a performance do Windows, eliminando o overhead desnecessário de expansão de imagem e tornando as operações de I/O mais previsíveis e eficientes.  


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
E prossiga normalmente como faria numa instalação do [Windows](debian_qemu_kvm_windows.md), no entanto, mantenha **Dispositivo de disco** e **Interface de rede** com seus valores padrão. Não é o momento para especificar drivers do **VirtIO** ainda.
---

## 🚀 Etapa 6 — Aprimoramentos

Após o boot do Windows ter iniciado, instale as ferramentas para convidado. Elas incluirão todos os **drivers VirtIO** (armazenamento, rede e vídeo).  
Depois desligue essa VM.
Agora que você tem todos os drivers qemu/kvm necessários, desejar melhorar a performance faça as seguintes modificações nesta VM:
1. Em **Dispositivo de disco**, selecione **VirtIO** (melhor desempenho).
2. Em **Interface de rede**, use **VirtIO (paravirtualizado)**.

Essas alterações estão permonorizadas nos passos anteriores descritos [aqui](debian_qemu_kvm_windows.md)).  
Depois inicie a VM.  
Se não funcionar, reverta as alterações.

## 🧩 Conclusão

A conversão de discos **VDI → QCOW2** é o caminho mais prático para migrar VMs do VirtualBox para o QEMU/KVM.
Com essa abordagem:

* você mantém todos os dados intactos,
* aproveita o desempenho nativo do KVM,
* e ainda ganha recursos avançados como snapshots e backup incremental.

Essa técnica é ideal tanto para **migrações definitivas** quanto para **testes de performance** em ambientes Linux modernos.

```

