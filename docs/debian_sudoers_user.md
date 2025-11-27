# 'SUDO' - SOMENTE PARA O DEBIAN 

O sudoers e seu utilitário de linha de comando chamado `sudo` é o responsável por elevar as permissões do usuário para que ele consiga executar comandos que apenas o **root** teria acesso.  
Por padrão, o usuario comum não é do mesmo grupo do "sudo", isso impossibilitará de usar o comando `sudo`, vamos resolver essa situação, execute:  

```bash
su root  
```

No comando acima, precisará digitar a senha, e então depois disso, execute:  
```bash
sudo usermod -aG sudo $USER
```

E confirme se realmente o gsantana esta no grupo 'sudo', execute:
```bash
groups $USER
```

O resultado deve demonstrar que o usuário **gsantana** esta incluso no grupo **sudo**:  
>**gsantana** : gsantana cdrom floppy **sudo** audio dip video plugdev users netdev scanner bluetooth lpadmin

Depois vamos reiniciar o sistema para que as alterações acima entrem em vigor:  
```bash
/sbin/reboot
```

Após o boot, o `sudoers` estará funcionando para os membros do grupo `sudo`.  

----

[Clique aqui para retornar a página principal](../README.md#sudo---somente-para-o-debian)
