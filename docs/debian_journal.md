# PERMISSÃO AO JOURNAL 
O journal é o mecanismo de logs do systemd. Ele registra praticamente tudo o que ocorre no sistema — mensagens do kernel, inicialização de serviços, eventos de segurança, entre outros. Antigamente, esses registros eram armazenados em simples arquivos texto (como /var/log/syslog), acessíveis a qualquer usuário. Hoje, o journal é um serviço binário centralizado com restrições de acesso.
Essa restrição afeta alguns comandos como o **systemctrl status [serviço]**, veja este exemplo:  
```bash
systemctl status systemd-journald
```
Se você observar um aviso como este:
> Warning: some journal files were not opened due to insufficient permissions.

Significará que você não tem permissões suficientes para observar os logs.  
Algumas distros removem este aviso, dando a entender que talvez você tenha a permissão necessária, para investigar isso, temos de ver se seu login tem acesso ao grupo **systemd-journal**, então execute:
```bash
groups
```

E na lista de grupos que tem acesso, observe se aparece ou não o grupo **systemd-journal**:  
>gsantana cdrom floppy sudo audio dip video plugdev users netdev scanner bluetooth lpadmin

Caso não apareça o grupo **systemd-journal** na listagem, então execute: 
```bash
sudo usermod -aG systemd-journal "$USER"
```
Em seguida, atualize sua sessão para que a mudança tenha efeito:
```bash
newgrp systemd-journal  # ou faça logout/login
```
Essa alteração só concede acesso de leitura aos logs do sistema. Ela é segura e recomendada para administradores que precisam analisar mensagens de serviços sem usar sudo o tempo todo.
