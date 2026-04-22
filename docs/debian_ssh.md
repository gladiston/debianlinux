# Habilitando acesso remoto por SSH

## Introdução
O SSH (*Secure Shell*) é uma ferramenta indispensável para a administração remota de sistemas Linux. Ele permite a execução de comandos, transferência de arquivos e até o redirecionamento de interfaces gráficas de forma criptografada. No ecossistema Debian 13 e Ubuntu, o servidor padrão é o `OpenSSH`, conhecido por sua robustez e conformidade com os padrões de segurança atuais do Kernel Linux.

## Passo a Passo

### 1. Instalação do Servidor SSH
Para permitir que outros computadores acessem sua máquina, é necessário instalar o pacote `openssh-server`.

```bash
sudo apt update
sudo apt install openssh-server

```

Após a instalação, verifique se o serviço está ativo:

```bash
sudo systemctl status ssh

```

### 2. Habilitar Acesso à Conta Root (Opcional)

Por padrão, o Debian e o Ubuntu desativam o login remoto diretamente como root por motivos de segurança. 
Caso seu fluxo de trabalho exija isso, siga estes passos:

1. Edite o arquivo de configuração do SSH:
```bash
sudo editor /etc/ssh/sshd_config

```


2. Localize a linha `#PermitRootLogin prohibit-password` e altere para:
```text
PermitRootLogin yes

```


3. Salve (Ctrl+O, Enter) e saia (Ctrl+X).
4. Reinicie o serviço:
```bash
sudo systemctl restart ssh
```

*Nota: Recomenda-se o uso de chaves SSH em vez de senhas ao habilitar o acesso root.*

### 3. Habilitar X11 Forwarding (Interface Gráfica Remota)

O X11 Forwarding permite que você execute um aplicativo gráfico no servidor e visualize a janela no seu computador local.

1. No servidor, certifique-se de que a opção está ativa no `/etc/ssh/sshd_config`:
```text
X11Forwarding yes
```

2. Reinicie o serviço SSH se houver alterações.
3. Para conectar-se usando o X11, utilize a flag `-X` no cliente:
```bash
ssh -X usuario@ip-do-servidor
```


4. Agora, ao abrir um programa (ex: `x11-apps` ou `virt-manager`), a janela aparecerá na sua máquina local.

## Conclusão

A configuração do SSH no Debian 13 e Ubuntu é um processo direto, mas que exige atenção às permissões de acesso. Ao habilitar o encaminhamento X11, o administrador ganha a flexibilidade de gerenciar ferramentas complexas remotamente com a mesma facilidade de uma sessão local. Para diretrizes avançadas de segurança e virtualização, sempre consulte a [documentação oficial do Debian](https://www.debian.org/doc/) e o [portal de documentação do Ubuntu Server](https://documentation.ubuntu.com/server/how-to/).

----

[Clique aqui para retornar a página principal](../README.md#habilitando-area-de-trabalho-remota-via-ssh)
