# WEB-DAV - COMPARTILHANDO ARQUIVOS VIA SPICE WebDAV

O **SPICE** (*Simple Protocol for Independent Computing Environments*) é um protocolo de código aberto desenvolvido para acesso remoto de alta performance a ambientes de desktop virtualizados, sendo amplamente utilizado pelo **QEMU/KVM** e `virt-manager`. Ele lida com a compressão, *encoding* e transporte de dados gráficos, áudio e periféricos (como teclado e mouse) entre o host Linux e o convidado Windows.

O **SPICE WebDAV proxy** é um componente do SPICE que permite o compartilhamento de pastas do sistema hospedeiro (Linux) para o sistema convidado (Windows) utilizando o protocolo de rede **WebDAV** (Web Distributed Authoring and Versioning).

> Embora o **Virtio-FS** seja o método de compartilhamento de arquivos mais **rápido e performático**, ele possui limitações notáveis em ambientes Windows devido à diferença na arquitetura dos sistemas de arquivos (Linux é *case-sensitive* e Windows é *case-insensitive*):
>
> 1.  **Permissões e Segurança (ACLs):** O Virtio-FS mapeia permissões POSIX (Linux) para as ACLs (Windows Access Control List), o que é **incompleto** e pode resultar em problemas como **acesso somente leitura** ou falhas ao tentar modificar ou criar arquivos, a menos que as permissões sejam alteradas manualmente no host (ex: `chmod 777`).
>       * **Nota sobre `chmod 777`:** Embora o uso de `chmod 777` (permissão total para proprietário, grupo e outros) em pastas compartilhadas possa resolver rapidamente problemas de permissão de escrita no Virtio-FS, **cuidado ao aplicá-lo em arquivos**. Arquivos executáveis ou de configuração com permissões tão abertas representam um risco de segurança.
> 2.  **Links Simbólicos (*Symlinks*):** O suporte a *symlinks* em pastas compartilhadas com Virtio-FS no Windows pode ser problemático e, em alguns casos, **causar o *crash*** do driver Virtio-FS no sistema convidado, especialmente com *self-symlinks*.
> 3.  **Sistemas Windows Antigos:** O Virtio-FS é tipicamente suportado apenas a partir do **Windows 10** (e requer a instalação de *drivers* e do WinFsp), enquanto o WebDAV pode funcionar com versões mais antigas (como Windows 7).

Dito isso, o WebDAV via SPICE é uma alternativa estável, embora **mais lenta**, para o compartilhamento de arquivos com VMs Windows.

-----

## 1\. Configuração no Hospedeiro (Linux) com `virt-manager`

1.  **Desligue a VM** Windows.
2.  Vá até as **configurações de *hardware*** da VM.
3.  Adicione um novo hardware, escolhendo o tipo **Canal** (*Channel*).
4.  Defina o **Nome** do dispositivo para `org.spice-space.webdav.0`. Deixe os outros campos como padrão.

> **Nota:** A pasta a ser compartilhada do lado do host será configurada posteriormente através do **`virt-viewer`** (o cliente SPICE).

-----

## 2\. Configuração no Convidado (Windows)

1.  Inicie a VM.
2.  Baixe e instale o **`spice-webdavd`** no Windows convidado. Este pacote contém o serviço necessário para que o Windows se conecte ao canal WebDAV.
      * O instalador pode ser encontrado no repositório de drivers e ferramentas do VirtIO para Windows, tipicamente no pacote **`virtio-win-guest-tools.exe`** ou separadamente:
        [spice-webdavd (arquivos MSI)](https://www.google.com/search?q=https://gitlab.freedesktop.org/spice/win32/spice-webdavd/-/releases)
3.  Após a instalação, verifique se o serviço **`Spice webdav proxy`** está em execução.
      * Execute `services.msc` no Windows.
      * Localize o serviço **`Spice webdav proxy`**.
      * Defina o **Tipo de Inicialização** como **Automático** e inicie o serviço, se necessário.

> **ADVERTÊNCIA**: O serviço `Spice webdav proxy` pode não iniciar corretamente se você estiver usando o console do `virt-manager` diretamente. Para que o recurso funcione, você pode precisar abrir a VM usando o **`virt-viewer`** separadamente.

-----

## 3\. Mapeando a Pasta Compartilhada

O compartilhamento efetivo da pasta do hospedeiro é configurado através do `virt-viewer`:

1.  No `virt-viewer` (a janela que mostra o desktop do Windows):
      * Vá em **Arquivo** (*File*) \> **Preferências** (*Preferences*).
      * Marque a opção **Compartilhar Pasta** (*Share Folder*) e escolha a pasta do seu sistema Linux que você deseja exportar.

Alternativamente, dentro do Windows, você pode executar o *script* de mapeamento de unidade (se instalado com o `spice-webdavd`), que é o atalho para montar o compartilhamento WebDAV como uma letra de *drive*:

```bash
"C:\Program Files\SPICE webdavd\map-drive.bat"
```

### Exemplo do Conteúdo do `map-drive.bat`

Este *script* é um arquivo *batch* simples que utiliza o comando `net use` do Windows para mapear um *drive* (unidade) de rede para o canal WebDAV exposto pelo SPICE:

```bat
@echo off
rem Script para mapear a pasta compartilhada do host como uma unidade de rede no guest.

set HOST_SHARE="\\Spice\\org.spice-space.webdav.0"
set DRIVE_LETTER=W:

rem Verifica se a unidade de rede já existe. Se existir, é removida.
net use %DRIVE_LETTER% /delete >nul 2>&1

rem Mapeia a unidade de rede.
net use %DRIVE_LETTER% %HOST_SHARE%

rem Verifica o sucesso da operação.
if %errorlevel% neq 0 (
    echo.
    echo Nao foi possivel mapear a unidade %DRIVE_LETTER%.
    echo O Servico "Spice webdav proxy" esta em execucao?
    pause
) else (
    echo.
    echo Unidade %DRIVE_LETTER% mapeada com sucesso!
)
```

A pasta escolhida no **`virt-viewer`** aparecerá no Windows geralmente mapeada como a letra de unidade (`W:` no exemplo acima) ou como um **local de rede**.

Você também pode tentar acessar digitando o caminho na barra de endereços do Explorador de Arquivos do Windows:

```
Spice://org.spice-space.webdav.0
```

Isso pode fazer com que o Windows reconheça o compartilhamento.

-----

### Esclarecendo `virt-manager` vs. `virt-viewer`

O **`virt-manager`** é a ferramenta de **gerenciamento** que cria e configura a máquina virtual e seus dispositivos (como o canal WebDAV no Passo 1). O **`virt-viewer`** é o programa **cliente/visualizador** que se conecta à saída gráfica (desktop) da VM. A janela que mostra o sistema Windows em execução é, na verdade, o `virt-viewer`, e é neste programa que você deve ir ao menu **"Arquivo"** para configurar o compartilhamento de pastas.

-----

## SEGURANÇA

1.  Não é recomendado exportar o seu diretório `$HOME` inteiro.
2.  Use o `virt-viewer` para selecionar **apenas a pasta necessária** para a VM convidada.
3.  Lembre-se que o WebDAV via SPICE é mais lento que o Virtio-FS, sendo mais adequado para arquivos pequenos ou acesso esporádico.

-----

[Retornar à página de Virtualização nativa com QAEMU+KVM Usando VM/Windows](debian_qemu_kvm_windows.md)
