# 'SUDO' - PERSONALIZANDO AS OPÇÕES
O sudoers e seu utilitário de linha de comando chamado `sudo` é o responsável por elevar as permissões do usuário para que ele consiga executar comandos que apenas o **root** teria acesso.  Se você é desenvolver ou administrador de sistemas, tem duas opções que flexibiizam o uso do comando `sudo`, e vou demonstrar cada uma delas.

Diferentemente do Debian, no Ubuntu o usuário comum já é membro do grupo `sudo`, então ele pode usar o comando `sudo` livremente logo depois que o sistema foi instalado completamente. Mas se você usa o Debian, tenha certeza de ter aplicado [estas instruções](debian_sudoers_user.md).  

Mas qualquer que seja sua distro, caso seja um desenvolvedor e pretenda relaxar um pouco o uso do 'SUDO' então podemos alterar o comportamento do `sudo` em relação a solicitação de senha. Se você usou Debian ou Ubuntu no passado, você talvez saiba que antigamentealterávamos o arquivo principal `/etc/sudoers` (ou sudo visudo), no entanto, hoje não é mais assim, alias até pode funcionar numa distro ou outra, mas o jeito certo agora é criar arquivos separados para cada ajuste e salvos em `/etc/sudoers.d/`. Essa mudança tem uma vantagem, caso aconteça alguma atualização do **sudoers** que sobreponha o arquivo de configuração principal, não perderá as modificações feitas. Então risca do seu hábito anterior o comando ~sudo visudo~, agora é:  

```bash
sudo visudo -f /etc/sudoers.d/00-admin-permissive
```
E então você coloca o que deseja, as regras recentes substituem as antigas, por exemplo, no `/etc/sudoers` tem a linha:  
```
%sudo   ALL=(ALL:ALL) ALL
```
Mas se em nosso `/etc/sudoers.d/00-admin-permissive` colocarmos:
```
# Permite qualquer comando sem senha — use com cautela
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
# No Ubuntu tem também:
%admin  ALL=(ALL:ALL) NOPASSWD: ALL
```
Isso substituirá a regra anterior.  
Nosso exemplo acima é de uma regra permissiva, isto é, não pedirá senha. Você deve usar apenas se for desenvolvedor e sabe o que esta fazendo no computador.  

Em servidores, muitas vezes precisamos executar comandos sem senha, mas quando precisamos fazer isso, precisamos determinar exatamente como o comando será executado, incluindo o seu path e parametros e então podemos criar algo assim:  
```bash
sudo visudo -f /etc/sudoers.d/10-admin-basic
```
E colar o conteúdo assim:  
```
Cmnd_Alias BASIC = /usr/bin/mount, /usr/bin/umount, /usr/bin/mkdir, /bin/rm, /bin/cp, /bin/chmod, /bin/chown, /usr/bin/touch, /usr/bin/apt, /sbin/reboot, /sbin/poweroff

# Permite executar os comandos do alias BASIC sem senha
%sudo   ALL=(ALL:ALL) NOPASSWD: BASIC

# Para todos os demais comandos, continua exigindo senha
%sudo   ALL=(ALL:ALL) ALL
```
Em desktops, dificilmente usariamos um arquivo como o acima, mas em servidores isso é comum, imagine a instalação de um servidor cujo menu de administração seja uma página web onde os cgi façam isso de comandos no prompt e isso acontece muito, por exemplo, um servidor de proxy squid e você quer colocar uma página web para que o administrador possa reiniciar o computador e também o serviço então seria mais ou menos assim:
```bash
sudo visudo -f /etc/sudoers.d/10-admin-squid-server
```
Com o conteúdo de comandos que os cgi's fariam uso no computador:  
```
ALL ALL=NOPASSWD: /usr/sbin/squid -k reconfigure
ALL ALL=NOPASSWD: /usr/sbin/systemctl reload apache2
ALL ALL=NOPASSWD: /sbin/shutdown -rF now
ALL ALL=NOPASSWD: /sbin/shutdown -h now
ALL ALL=NOPASSWD: /usr/bin/systemctl status squid
ALL ALL=NOPASSWD: /usr/bin/systemctl start squid
ALL ALL=NOPASSWD: /usr/bin/systemctl stop squid
ALL ALL=NOPASSWD: /usr/bin/systemctl restart squid
```
Apenas um ponto de atenção, se a linha é `/usr/bin/systemctl restart squid`, não funcionará `systemctl restart squid`, mesmo que o comando `systemctl` esteja no $PATH. Isso é uma questão de segurança. Então para saber se um comando funcionará ou não usamos `command -v <comando>` (ex.: `command -v /usr/bin/systemctl restart squid`) . Geralmente o `visudo` faz este teste, mas um bom administrador que lida com muitas distros diferentes não tem como saber se a versão do sudoers de uma tem o mesmo comportamento da outra então ele faz o teste com `command -v <comando>`  sempre.  

As alterações em aquivos do **sudoers** passam a valer imediatamente.  

### Observações práticas  
- Sempre edite o arquivo com `visudo` — ele valida a sintaxe antes de salvar.  
- Verifique caminhos com `command -v <comando>` e substitua os caminhos no `Cmnd_Alias` se necessário.   
- Para desfazer, remova as linhas adicionadas ou restaure a linha original `%sudo   ALL=(ALL:ALL) ALL`.  
- Não use `sudo visudo` ou edite diretamente `/etc/sudoers`, prefira arquivos externos.  

Salve e feche o arquivo (`Ctrl+O`, `Enter`, `Ctrl+X`).    



----

[Clique aqui para retornar a página principal](../README.md#sudo---personalizando-op%C3%A7%C3%B5es)
