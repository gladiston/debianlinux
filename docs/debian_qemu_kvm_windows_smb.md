# Compartilhamento de Pastas com Samba no Debian 13 e Acesso via Windows

Este documento fornece um guia completo, passo a passo, para configurar um compartilhamento de rede (SMB/CIFS) seguro usando o Samba no Debian 13 (Trixie) e acessá-lo a partir de um cliente Windows, utilizando um link simbólico local.

## 1\. Configuração do Servidor Debian/Ubuntu (Samba)

As etapas a seguir devem ser executadas no seu servidor ou máquina Linux Debian/Ubuntu para configurar o compartilhamento da pasta `/home/gsantana/work`.

### 1.1. Instalação do Samba

Instale os pacotes necessários para o servidor Samba e as ferramentas de cliente para testes.

```bash
sudo apt update
sudo apt install -y samba smbclient
```

### 1.2. Criação e Configuração do Usuário Samba

O Samba gerencia sua própria base de dados de senhas para acesso à rede. O usuário do sistema (`gsantana`) deve ser adicionado a esta base de dados.

1.  **Definir a Senha do Samba:**
    Use o comando `smbpasswd -a` para adicionar o usuário `gsantana` ao banco de dados do Samba e definir uma senha de rede.

    ```bash
    sudo smbpasswd -a gsantana
    ```

    (Você será solicitado a digitar e confirmar a nova senha do Samba.)

2.  **Habilitar o Usuário (Garantia):**

    ```bash
    sudo smbpasswd -e gsantana
    ```

### 1.3. Configuração do Compartilhamento (`/etc/samba/smb.conf`)

Edite o arquivo principal de configuração para definir o novo recurso de compartilhamento.

1.  **Backup da Configuração Original:**

    ```bash
    sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
    ```

2.  **Editar o Arquivo de Configuração:**

    ```bash
    sudo editor /etc/samba/smb.conf
    ```

3.  **Solução de Compatibilidade para Links Simbólicos (Symlinks)**  
    A incompatibilidade na visualização de pastas ou arquivos dentro do Windows ocorre frequentemente quando o Samba encontra **links simbólicos** que apontam para fora do diretório compartilhado. Por padrão, o Samba tenta aplicar as extensões e atributos de segurança do UNIX (permissões, proprietário, grupo) à conexão SMB/CIFS, o que pode confundir clientes Windows.  
    
    Para garantir que links simbólicos funcionem e que o Windows consiga interpretar corretamente os atributos das pastas e arquivos:  
    
    Adicione a diretiva `unix extensions = no` na seção `[global]` do arquivo `/etc/samba/smb.conf`. Esta linha desabilita a tentativa do Samba de usar atributos de arquivo UNIX, melhorando a compatibilidade com o Windows, especialmente ao lidar com links simbólicos:  
```Ini, TOML
    [global]
        (...)
        workgroup = WORKGROUP
        unix extensions = no    ; <<< Adicionar esta linha
        (...)
```  
    
4.  **Adicionar o Novo Compartilhamento:**
    Adicione a seção a seguir ao **final** do arquivo. Ela restringe o acesso ao usuário `gsantana` e permite leitura/escrita.

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

5.  **Salvar e Sair** do editor.

### 1.4. Verificação de Permissões no Linux

Confirme se o usuário `gsantana` possui as permissões corretas no sistema de arquivos para a pasta a ser compartilhada.
Define gsantana como dono (se necessário):  
```bash
sudo chown -R gsantana:gsantana /home/gsantana/work
```

Garante permissões rwx (7) ao proprietário:  
```
sudo chmod -R 755 /home/gsantana/work
```

### 1.5. Reinício e Teste do Serviço

1.  **Verificar a Sintaxe da Configuração:**

    ```bash
    testparm
    ```

    Confirme que o `smb.conf` foi carregado sem erros.

2.  **Reiniciar o Serviço SMB:**

    ```bash
    sudo systemctl restart smbd
    ```

3.  **Testar o Acesso Localmente (Opcional):**

    ```bash
    smbclient //localhost/work -U gsantana
    ```

    Digite a senha do Samba. O prompt `smb: \>` confirma a conexão bem-sucedida.

-----

## 2\. Acesso e Mapeamento no Cliente Windows

Após a configuração no Debian, o compartilhamento pode ser acessado em qualquer máquina Windows na mesma rede.

### 2.1. Teste de Conexão Inicial

1.  Obtenha o **Endereço IP** do seu servidor Debian/Ubuntu (e.g., usando `ip a` no terminal Linux). Mapear usando o nome não funciona muito bem com VMs usando NAT(provavelmente nosso caso).  
2.  No Windows, pressione **`Win + R`** para abrir o **Executar**.
3.  Digite o caminho UNC do compartilhamento:
    ```
    \\IP_DO_SEU_LINUX\work
    ```
4.  Insira o nome de usuário (`gsantana`) e a **senha do Samba**. Marque a opção para lembrar credenciais.

### 2.2. Recomendação: Mapeamento Local com Link Simbólico (Symlink)

Uma vez que você manteve sua senha "lembrada", o Windows guardará suas credenciais para que uma próxima vez que for mapeá-los não precise mais fornecer senha. Isso é muito bom, mas pode ficar ainda melhor, para um acesso mais integrado e transparente, crie um link simbólico que aponta o caminho de rede para um diretório local.  
Iremos criar uma pasta C:\mnt e dentro dela links simbólicos que apontam para nosso compartilhamento e com isso, voce não precisa mais usar letras de drives para acessá-los.  

**Requer: Prompt de Comando executado como Administrador.**

1.  **Abrir o Prompt de Comando:**
    Procure por `cmd`, clique com o botão direito e selecione **"Executar como administrador"**.

2.  **Criar o Diretório de Montagem Local:**
    Crie a pasta que servirá de destino para o link simbólico.

    ```cmd
    mkdir C:\mnt   # ou \@mnt, caso queira que seja listado no seu explorer como primeira pasta
    ```

3.  **Criar o Link Simbólico:**
    Use o comando `MKLINK` com a opção `/D` (para diretórios), apontando o caminho local (`C:\Mnt\work`) para o caminho de rede (UNC).

    Substitua `IP_DO_SEU_DEBIAN` pelo endereço real do seu servidor.

    ```cmd
    mklink /D C:\Mnt\work \\IP_DO_SEU_LINUX\work
    ```

4.  **Resultado:**
    A pasta compartilhada do Linux agora é acessível diretamente no seu sistema Windows através do caminho local **`C:\Mnt\work`**.

-----
## 3\. Configuração do Firewall (UFW) no Debian/Ubuntu
Se você tem o firewall instalado, então vai precisar liberar o protocolo SMB para que outros computadores e suas VMs possam te acessar.  
O protocolo SMB (Samba) utiliza portas específicas para comunicação. Se o firewall Uncomplicated Firewall (UFW) estiver ativo no seu Debian/Ubuntu, você precisará liberar essas portas para permitir o acesso da rede local ao compartilhamento.

### 3.1. Verificar o Status do UFW

Primeiro, verifique se o UFW está ativo no seu sistema:

```bash
sudo ufw status
```

Se o status for `inactive`, você pode pular esta seção. Se o status for `active`, continue com as regras.

### 3.2. Regras de Firewall para Samba (SMB/CIFS)

O Samba utiliza as portas TCP e UDP para os serviços NetBIOS e SMB. Você deve liberá-las para a sua rede local.

**1. Liberar Portas Essenciais do Samba:**

O Samba geralmente requer as seguintes portas:

  * **Portas NetBIOS:** UDP 137, UDP 138
  * **Porta SMB/CIFS:** TCP 139 (para compatibilidade legada)
  * **Porta Net Logon/Replication:** TCP 445 (a porta SMB moderna e mais comum)

Você pode liberar todas elas usando o nome do serviço `samba` no UFW, se a regra estiver definida:

```bash
sudo ufw allow samba
```

ou, de forma mais específica, liberando as portas diretamente:

```bash
# Permite NetBIOS (name/datagram service) e compatibilidade com SMB
sudo ufw allow 137,138/udp
sudo ufw allow 139/tcp

# Permite SMB/CIFS (o tráfego mais moderno)
sudo ufw allow 445/tcp
```

**2. Restringir o Acesso à Sub-rede Local (Melhor Prática de Segurança):**

Se você quiser liberar o acesso **somente** para a sua rede local (por exemplo, `192.168.1.0/24`), e não para a Internet, a sintaxe de segurança é a seguinte. Substitua `192.168.1.0/24` pelo bloco CIDR da sua rede:

```bash
# Permite SMB/CIFS (Porta 445 TCP) apenas da sub-rede local
sudo ufw allow from 192.168.1.0/24 to any port 445 proto tcp

# Permite a porta TCP 139 apenas da sub-rede local
sudo ufw allow from 192.168.1.0/24 to any port 139 proto tcp
```

### 3.3. Aplicar e Verificar as Alterações

Recarregue o UFW para que as novas regras entrem em vigor:

```bash
sudo ufw reload
```

Em seguida, verifique o status do firewall para confirmar que as regras do Samba estão ativas:

```bash
sudo ufw status verbose
```

O Samba agora deve estar acessível por outros dispositivos na sua rede, respeitando as regras de acesso do seu firewall.
