# 'SUDO' - PERSONALIZANDO AS OPÇÕES
O sudoers e seu utilitário de linha de comando chamado `sudo` é o responsável por elevar as permissões do usuário para que ele consiga executar comandos que apenas o **root** teria acesso.  Se você é desenvolver ou administrador de sistemas, tem duas opções que flexibiizam o uso do comando `sudo`, e vou demonstrar cada uma delas.

Diferentemente do Debian, no Ubuntu o usuário comum já é membro do grupo `sudo`, então ele pode usar o comando `sudo` livremente logo depois que o sistema foi instalado completamente. Mas se você usa o Debian, tenha certeza de ter aplicado [estas instruções](debian_sudoers_user.md).   
Mas qualquer que seja sua distro, caso seja um desenvolvedor e pretenda relaxar um pouco o uso do 'SUDO' então podemos alterar o comportamento do `sudo` em relação a solicitação de senha, execute: 
```bash
sudo visudo
```

No editor procure por uma linha parecida com:

```
%sudo   ALL=(ALL:ALL) ALL
```
E então a comente, isto é, coloque "#" no início e adicione as linhas de uma das opções abaixo.  

### Opção simples (sem senha para comandos específicos)
A opção simples é fazer o `sudo` permitir que certos comandos sejam executados sem senha, enquanto outros comandos lhe será solicitado senha. Isso é muito util para comandos repetitivos do dia a dia. Mas para isso funcionar direito, os comandos precisam ser informados o seu caminho absoluto, por isso, recomendo testar os comandos primeiro com o uso de `command -v <comando>` (ex.: `command -v apt`) — é uma regra do sudoers exigir o caminho correto ou o comando não será reconhecido.  

Acrescente logo após a linha comentada, as linhas:  
```
Cmnd_Alias BASIC = /usr/bin/mount, /usr/bin/umount, /usr/bin/mkdir, /bin/rm, /bin/cp, /bin/chmod, /bin/chown, /usr/bin/touch, /usr/bin/apt, /sbin/reboot, /sbin/poweroff

# Permite executar os comandos do alias BASIC sem senha
%sudo   ALL=(ALL:ALL) NOPASSWD: BASIC

# Para todos os demais comandos, continua exigindo senha
%sudo   ALL=(ALL:ALL) ALL
```
Essas linhas liberam o `sudo` sem senha apenas alguns comandos, os demais precisarão da digitação da senha.
Salve e feche o arquivo (`Ctrl+O`, `Enter`, `Ctrl+X`).    

**Como funciona:** membros do grupo `sudo` poderão executar os comandos listados em `BASIC` sem digitar a senha. Para qualquer outro comando, continuará sendo exigida a senha.

### Opção permissiva (sem senha para qualquer comando)
Se quiser liberar qualquer comando sem senha (não recomendado em estações com risco), use:
```
# Permite qualquer comando sem senha — use com cautela
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```

> **IMPORTANTE:** liberar `NOPASSWD: ALL` representa um risco de segurança. Faça isso **apenas** se tiver certeza do contexto: a estação não contém chaves privadas, credenciais sensíveis ou arquivos valiosos e está em ambiente controlado. Mesmo com `NOPASSWD` para uma lista, confirme os caminhos absolutos dos comandos (use `command -v`) para evitar entradas inválidas no `sudoers`.

### Observações práticas
- Sempre edite o arquivo com `visudo` — ele valida a sintaxe antes de salvar.  
- Verifique caminhos com `command -v <comando>` e substitua os caminhos no `Cmnd_Alias` se necessário.  
- Para desfazer, remova as linhas adicionadas ou restaure a linha original `%sudo   ALL=(ALL:ALL) ALL`.

Salve e feche o arquivo (`Ctrl+O`, `Enter`, `Ctrl+X`).    
