# COMPARTILHAMENTO DE ARQUIVOS
O Samba é o componente responsável por permitir que sistemas Linux e Windows troquem arquivos e pastas através da rede local, utilizando o protocolo SMB/CIFS. Ele possibilita tanto o acesso a compartilhamentos de outros computadores quanto a criação de compartilhamentos próprios, tornando a integração entre diferentes sistemas fluida e transparente.

Nas distribuições Debian e Ubuntu, o Samba costuma vir parcialmente instalado, mas requer alguns ajustes para funcionar de forma completa e adequada em ambientes domésticos ou corporativos.
Este tutorial mostrará como instalar os pacotes necessários, ajustar o workgroup ou domínio da rede, evitar que o serviço atue como controlador de domínio e configurar corretamente o compartilhamento de arquivos, garantindo compatibilidade com máquinas Windows e o melhor desempenho possível no seu sistema Linux.

Aparentemente, o **SAMBA** vem pré-instalado, no entanto, foi observado que ele carece de alguns ajustes.

### Instalando o SAMBA
Execute:
```bash
sudo apt install -y plasma-widgets-addons kdenetwork-filesharing 
sudo apt install -y cifs-utils kio-fuse
```

Às vezes, dependendo do perfil de instalação, estes pacotes podem ter sido instalados.

### Ajustando o workgroup ou domínio
Se você possui um domínio em sua rede, é recomendável fazer um pequeno ajuste no arquivo de configuração do **Samba**.  
Edite o arquivo `/etc/samba/smb.conf`:

```bash
sudo editor /etc/samba/smb.conf
```

Procure a linha:
```
WORKGROUP = WORKGROUP
```

E substitua por:
```
WORKGROUP = LOCALDOMAIN.LAN  # ou apenas LOCALDOMAIN
```

O nome do domínio (no exemplo, `LOCALDOMAIN.LAN`) deve ser digitado **em letras maiúsculas** por compatibilidade com o antigo protocolo **WINS**, ainda usado no Windows.  

Salve e feche o arquivo (`Ctrl+O`, `Enter`, `Ctrl+X`).     
Com essa modificação, ao acessar uma pasta compartilhada na rede, o nome `meudominioderedelocal.lan` já aparecerá como padrão na tela de autenticação — poupando alguns toques de teclado e reduzindo o risco de lesão por esforço repetitivo nos dedos.  

No entanto, o serviço **samba-ad-dc** **não deve ser iniciado**, pois ele é destinado a atuar como **controlador de domínio**, e essa não é nossa intenção, portanto, desabilite-o.

### Desativando o controlador de domínio
Em algumas situações, o controlador de domínio pode ter sido instalado, alterando completamente o comportamento do Samba.  
Ele **não deve ser instalado em desktops**. Caso isso tenha acontecido, execute:

```bash
sudo systemctl disable samba-ad-dc
```

Se o comando acima responder:
```
Failed to disable unit: Unit samba-ad-dc.service does not exist
```
Então, parabéns! O controlador de domínio **não está instalado**, e você pode prosseguir.

Mas se aparecer algo como:
```
Synchronizing state of samba-ad-dc.service with SysV service script with /usr/lib/systemd/systemd-sysv-install.
Executing: /usr/lib/systemd/systemd-sysv-install disable samba-ad-dc
Removed '/etc/systemd/system/multi-user.target.wants/samba-ad-dc.service'.
Removed '/etc/systemd/system/samba.service'.
```
Então, é porque você estava com o controlador de domínio instalado — e talvez nem soubesse.  
De qualquer forma, o serviço foi desativado e você pode seguir adiante.

### Ativando o compartilhamento de arquivos
Caso precise **compartilhar arquivos do seu computador com máquinas Windows** reais ou virtuais, habilite os serviços necessários:

```bash
sudo systemctl start smbd nmbd
sudo systemctl enable smbd nmbd
```

Se, por outro lado, você **só precisa acessar** arquivos compartilhados em outros computadores (sem compartilhar os seus), é melhor desativar esses serviços para economizar recursos:

```bash
sudo systemctl stop smbd nmbd
sudo systemctl disable smbd nmbd
```

Isso deixará o sistema mais leve.


