# COMPARTILHAMENTO DE ARQUIVOS
O Samba é o componente responsável por permitir que sistemas Linux e Windows troquem arquivos e pastas através da rede local, utilizando o protocolo SMB/CIFS. Ele possibilita tanto o acesso a compartilhamentos de outros computadores quanto a criação de compartilhamentos próprios, tornando a integração entre diferentes sistemas fluida e transparente.

Nas distribuições Debian e Ubuntu, o Samba costuma vir parcialmente instalado, mas requer alguns ajustes para funcionar de forma completa e adequada em ambientes domésticos ou corporativos.
Este tutorial mostrará como instalar os pacotes necessários, ajustar o workgroup ou domínio da rede, evitar que o serviço atue como controlador de domínio e configurar corretamente o compartilhamento de arquivos, garantindo compatibilidade com máquinas Windows e o melhor desempenho possível no seu sistema Linux.

Aparentemente, o **SAMBA** vem pré-instalado, no entanto, foi observado que ele carece de alguns ajustes.

## Instalando o suporte a arquivos compartilhados
Execute:  
```bash
sudo apt install -y samba cifs-utils 
```
Se estiver usando KDE, recomendo instalar também:  
```bash
sudo apt install plasma-widgets-addons kdenetwork-filesharing kio-fuse
```

Às vezes, dependendo do perfil de instalação, estes pacotes podem ter sido instalados.  

## Ajustando o workgroup ou domínio
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

## Desativando o controlador de domínio
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

## Ativando o compartilhamento de arquivos
Caso precise **compartilhar arquivos do seu computador com máquinas Windows** reais ou virtuais, habilite os serviços necessários:

```bash
sudo systemctl start smbd nmbd
sudo systemctl enable smbd nmbd
```
Se você receber mensagem como:  
> Failed to start smbd.service: Unit smbd.service not found.  
> Failed to start nmbd.service: Unit nmbd.service not found.  

È porque o SAMBA não foi instalado, isso aconteceu com o Ubuntu 25+, então volte aos passos anteriores e inclua a instalação do meta-pacote 'samba'.  

## Compartilhando minhas pastas
O Samba pode ter suas contas integradas ao Active Directory e com isso não é necessário nem mesmo digitar a senha para acessar compartilhamentos externos. E sem a integração, você será forçado a digitar a senha, caso a mesma não seja memorizada.  
Mas e se precisarmos compartilhar os arquivos em nosso computador com outros usuários?  
Neste caso, você deve previamente adicionar os usuários em seu computador, no banco de dados do próprio Samba. Em nosso exemplo, vamos acrescentar a si mesmo no compartilhameto.

### Definir a Senha do Samba:
    Use o comando `smbpasswd -a` para adicionar o usuário `gsantana` ao banco de dados do Samba e definir uma senha de rede.

    ```bash
    sudo smbpasswd -a gsantana
    ```

    (Você será solicitado a digitar e confirmar a nova senha do Samba.)

**Habilitar o Usuário (Garantia)**
Defina uma senha para o usuário recém adicionado:

    ```bash
    sudo smbpasswd -e gsantana
    ```

**Configuração do Compartilhamento `/etc/samba/smb.conf`**

Edite o arquivo principal de configuração para definir o novo recurso de compartilhamento. Primeiro façamos um backup da configuração original:  

    ```bash
    sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
    ```

Depois editamos o arquivo de configuração:  

    ```bash
    sudo editor /etc/samba/smb.conf
    ```

Vamos eliminar os atributos que somente o Linux enxergaria, isso mesmo, a incompatibilidade na visualização de pastas ou arquivos dentro do Windows ocorre frequentemente quando o Samba encontra **links simbólicos** que apontam para fora do diretório compartilhado. Por padrão, o Samba tenta aplicar as extensões e atributos de segurança do UNIX (permissões, proprietário, grupo) à conexão SMB/CIFS, o que pode confundir clientes Windows.     
Para garantir que links simbólicos funcionem e que o Windows consiga interpretar corretamente os atributos das pastas e arquivos:, adicione a diretiva `unix extensions = no` na seção `[global]` do arquivo `/etc/samba/smb.conf`. Esta linha desabilita a tentativa do Samba de usar atributos de arquivo UNIX, melhorando a compatibilidade com o Windows, especialmente ao lidar com links simbólicos:  
```Ini, TOML
    [global]
        (...)
        workgroup = WORKGROUP
        unix extensions = no    ; <<< Adicionar esta linha
        (...)
```  
Se você tiver um dominio em sua rede, troque **WORKGROUP** pelo nome do seu dominio, ex **LOCALDOMAIN**. Isso acelera nosso trabalho porque ao mapear unidades não precisamos informar o usuário desse jeito **localdomain\gsantana**, apenas **gsantana** será suficiente.      

Agora, vamos ao compartilhamento em si mesmo, adicione a seção a seguir ao **final** do arquivo. Ela restringe o acesso ao usuário `gsantana` e permite leitura/escrita.

```ini
[work]
    comment = Pasta de Trabalho do gsantana
    path = /home/gsantana/work
    browseable = yes
    read only = no
    valid users = gsantana
    public = no
    writable = yes
    follow symlinks = yes
    wide links = yes

    # Máscaras de permissão
    create mask = 0644
    directory mask = 0755

    # Configurações de segurança para mapeamento de usuário
    force user = gsantana
    force group = gsantana
```
A pasta e o nome do compartilhamnento você pode ficar a vontade para modificar.  
Depois salve o arquivo e saia do editor.  

## Verificação de Permissões no Linux

Confirme se o usuário `gsantana` possui as permissões corretas no sistema de arquivos para a pasta a ser compartilhada.
Define gsantana como dono (se necessário):  
```bash
sudo chown -R gsantana:gsantana /home/gsantana/work
sudo setfacl -R -m d:u:gsantana:rwx /home/gsantana/work
```

Se precisar que mais pessoas tenham acesso, recomendo:  
```
sudo chmod -R 777 /home/gsantana/work
```

### Reinício e Teste do Serviço

Verificar a Sintaxe da Configuração:   
```bash
testparm
```
Confirme que o `smb.conf` foi carregado sem erros. 

Reiniciar o Serviço SMB:  
```bash
sudo systemctl restart smbd
```

Inicir o Serviço SMB após o boot:  
```bash
sudo systemctl enable smbd
```
    
Para testar o acesso localmente (Opcional):  
```bash
smbclient //localhost/work -U gsantana
```
Você terá de fornecerr a senha do `gsantana` que criamos nos passos anteriores e se estiver correta, o prompt `smb: \>` confirmará a conexão bem-sucedida e se quiser usar alguns comandos, tente o `ls` para listar arquivos e depois o `quit` para sair dele.  
    

## Acesso e Mapeamento no Cliente Windows

Após a configuração no Debian, o compartilhamento pode ser acessado em qualquer máquina Windows na mesma rede.

Obtenha o **Endereço IP** do seu servidor Debian/Ubuntu (e.g., usando `ip a` no terminal Linux), neste teste, não use o nome do host ainda, afinal queremos apenas testar o compartilhamento do jeito mais crús que existir, e se funcionar então podemos testar com o nome do host. No Windows, pressione **`Win + R`** para abrir o **Executar** e depois no Explorer, o caminho UNC do compartilhamento:
```
\\192.168.1.50\work
```
Quando questionado, forneça seu nome de usuário `gsantana` e a senha que criamos para ele e também marque a opção para lembrar das credenciais.

## Não quero compartilhar meus arquivos
Se, por outro lado, você **só precisa acessar** arquivos compartilhados em outros computadores (sem compartilhar os seus), é melhor desativar esses serviços para economizar recursos:

```bash
sudo systemctl stop smbd nmbd
sudo systemctl disable smbd nmbd
```

Sem nenhum compartilhamento, deixará o sistema mais leve.


----

[Clique aqui para retornar a página principal](../README.md)  

