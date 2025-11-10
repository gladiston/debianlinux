# WEB-DAV - COMPARTILHANDO ARQUIVOS VIA SPICE WebDAV

Para compartilhar arquivos entre o sistema hospedeiro (host) Linux e o convidado (guest) Windows usando o protocolo **WebDAV**, você pode utilizar o recurso **SPICE WebDAV** (também chamado de *Spice WebDAV proxy*).

Siga as instruções abaixo:

## 1\. Configuração no Hospedeiro (Linux) com `virt-manager` 🐧

1.  **Desligue a VM** Windows.
2.  Vá até as **configurações de *hardware*** da VM.
3.  Adicione um novo hardware, escolhendo o tipo **Canal** (*Channel*).
4.  Defina o **Nome** do dispositivo para `org.spice-space.webdav.0`. Deixe os outros campos como padrão.

> **Nota:** O `virt-manager` configura o canal, mas o **compartilhamento da pasta** em si é geralmente gerenciado através do `virt-viewer`, que é o cliente do SPICE.

## 2\. Configuração no Convidado (Windows) 💻

1.  Inicie a VM.
2.  Baixe e instale o **`spice-webdavd`** no Windows convidado. Você pode encontrar o instalador (geralmente um arquivo `.msi`) pesquisando por "spice-webdavd" ou no repositório do SPICE.
3.  Após a instalação, verifique se o serviço **`Spice webdav proxy`** está em execução.
      * Execute `services.msc` no Windows.
      * Localize o serviço **`Spice webdav proxy`**.
      * Defina o **Tipo de Inicialização** como **Automático** e inicie o serviço, se necessário.

> **ADVERTÊNCIA**: O serviço `Spice webdav proxy` pode não iniciar corretamente se você estiver usando o console do `virt-manager` diretamente. Para que o recurso funcione, você pode precisar abrir a VM usando o `virt-viewer` separadamente, o que pode ser feito abrindo o `virt-viewer` e conectando-se à VM, ou clicando em *Launch remote viewer* (Iniciar visualizador remoto) na aba *Console* do `virt-manager`.

## 3\. Mapeando a Pasta Compartilhada

O compartilhamento efetivo da pasta do hospedeiro é configurado através do `virt-viewer`:

1.  No `virt-viewer` (a janela que mostra o desktop do Windows):
      * Vá em **Arquivo** (*File*) \> **Preferências** (*Preferences*).
      * Marque a opção **Compartilhar Pasta** (*Share Folder*) e escolha a pasta do seu sistema Linux que você deseja exportar.

Alternativamente, dentro do Windows, você pode tentar executar o script de mapeamento de unidade (se instalado com o `spice-webdavd`):

```bash
"C:\Program Files\SPICE webdavd\map-drive.bat"
```

Por padrão, este script tenta mapear a pasta `~/Public` do seu usuário no hospedeiro Linux. Se você configurou via `virt-viewer`, a pasta escolhida aparecerá no Windows geralmente mapeada como uma letra de unidade (por exemplo, `Z:`) ou como um **local de rede**. Você também pode tentar acessar digitando o caminho na barra de endereços do Explorador de Arquivos do Windows:

```
Spice://org.spice-space.webdav.0
```

Isso pode fazer com que o Windows reconheça o compartilhamento.

## SEGURANÇA

1.  Assim como no Virtio-FS, não é recomendado exportar o seu diretório `$HOME` inteiro.
2.  Use o `virt-viewer` para selecionar **apenas a pasta necessária** para a VM convidada.
3.  Observe que o WebDAV via SPICE é conhecido por ser **lento** para transferências de arquivos grandes, sendo o Virtio-FS muito superior em desempenho.

-----

[Retornar à página de Virtualização nativa com QAEMU+KVM Usando VM/Windows](https://www.google.com/search?q=debian_qemu_kvm_windows.md)

-----

Este vídeo demonstra o poder e o uso do **Virtio-FS** no Proxmox, reforçando a performance da tecnologia, embora o WebDAV seja a alternativa que você solicitou [COMPARTILHANDO ARQUIVOS ENTRE VMs NO PROXMOX? VEJA O PODER DO VIRTIO-FS\!](https://www.youtube.com/watch?v=1kGtxAVFIqc).
http://googleusercontent.com/youtube_content/0
zzzzz

# VIRT-MANAGER - COMPARTILHANDO ARQUIVOS VIA SPICE-WEBDAV
Para compartilhar arquivos entre o sistema hospedeiro e convidado, voce pode usar o SPICE-WEBDAV. Esse é o método conhecido por muitos devs no mundo windows.  
Siga as instruções abaixo:  
(em revisão)
# Introdução ao SPICE WebDAV

**SPICE WebDAV** é uma ferramenta que integra protocolo WebDAV (Web Distributed Authoring and Versioning) com suporte a SPICE (Simple Protocol for Independent Computing Environments), permitindo acesso remoto a máquinas virtuais e compartilhamento de arquivos em ambientes de virtualização. Funciona como extensão para gerenciamento de recursos em infraestrutura virtualizada.

Vale instalar em ambientes que utilizam **hypervisores SPICE** (KVM/QEMU, Proxmox, oVirt) onde é necessário compartilhamento de arquivos bidirecional entre host e máquinas virtuais, além de acesso a storage remoto via WebDAV. É particularmente relevante em sua stack de virtualização com **QEMU+KVM**, facilitando deployments ágeis e gerenciamento centralizado de arquivos em ambientes de desenvolvimento e produção virtualizados—reduzindo necessidade de NFS ou Samba para casos de uso específicos.

Nesta seção, o **SPICE WebDAV** permite compartilhar arquivos entre o **sistema hospedeiro (Linux)** e o **convidado (Windows)** sem precisar configurar Samba, FTP ou serviços de rede. Ele é diferente do **WinSFP** que vimos antes, pois o WebDAV seria uma forma universal de troca de arquivos que você poderá ver sendo usado de formas diferentes dentro de organizações.  
O **SPICE WebDAV** usa um **canal SPICE** interno e o protocolo **WebDAV**, exibindo a pasta compartilhada como uma unidade de rede no Windows.  

## Preparar a VM (Canal SPICE WebDAV)
1. Desligue a máquina virtual.
2. Abra o **virt-manager**, selecione a VM, depois **Detalhes**, depois **Adicionar hardware** e então escolha **Canal**.
3. Configure:
   - **Tipo de dispositivo:** `spicevmc`
   - **Nome do dispositivo:** `org.spice-space.webdav.0`
4. Clique em **Concluir** e **salve as alterações**.

![Remova a economia de energia](../img/debian_qemu_kvm_windows56.png)   

Ótimo, agora você tem o canal, vá em Arquivo

(todo)
