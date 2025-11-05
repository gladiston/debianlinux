# HABILITANDO AREA DE TRABALHO REMOTA
Vez ou outra precisaremos acessar nossa area de trabalho, as mais experientes recomendarão usar o 'ssh -x' ou usar 'xserver' e logar-se no ip de nosso desktop, no entanto, isso não é tão simples para novos usuários do linux e também não permite o acesso onde a origem é um desktop Windows. Portanto, minha recomendação é instalar o xrdp, um protocolo de compartilhamento de sessões compativel com o 'rdp' da Microsoft e assim poderemos acessar nosso terminal Linux até mesmo de um Windows através do programa 'Remote Deskop'. Para instalar:  
```  
sudo apt install -y xrdp 
```
Verifique se o grupo 'ssl-cert' existe, execute:
```  
getent group ssl-cert
```
Observe o resultado indicando a existencia:  
```
ssl-cert:x:105:  
```
Se a resposta for afirmativa como acima, então agora verifique se o usuário 'xrdp' também exista. execute:
```  
getent passwd xrdp  
```
E observe o resultado indicando a existencia:  
```
xrdp:x:111:116::/run/xrdp:/usr/sbin/nologin  
```

Agora que sabemos que o grupo 'ssl-cert' existe, e o usuário 'xrdp' também, então adicione o usuário 'xrdp' ao grupo 'ssl-cert' para permitir que o serviço xrdp acesse as chaves SSL corretamente, execute:
```  
sudo usermod -aG ssl-cert xrdp
```
Agora precisaremos atualizar as permissões imediatamente, execute:
```
sudo systemctl daemon-reload
```
Ou, se preferir, reinicie o sistema (o login de grupo é aplicado na próxima sessão).  
Agora o serviço xrdp pode acessar os certificados em /etc/ssl/private/, permitindo conexões seguras via TLS sem erros.  

### HABILITANDO AREA DE TRABALHO REMOTA - CRIANDO UM CERTIFICADO PARA O XRDP
1. Gere o certificado e a chave privada
Use o openssl para criar um certificado autoassinado válido por 10 anos (ajuste conforme quiser):
```
sudo openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -nodes \
  -keyout /etc/ssl/private/xrdp.key \
  -out /etc/ssl/certs/xrdp.crt \
  -subj "/CN=$(hostname)"
```
Explicação:  
* -x509 → cria um certificado autoassinado
* -newkey rsa:4096 → gera uma nova chave RSA de 4096 bits
* -nodes → não criptografa a chave com senha (requerida para uso automático)
* -subj → define o nome comum (CN) com o hostname do servidor

2. Ajuste as permissões corretamente, somente root e membros do grupo ssl-cert devem ter acesso:
```
sudo chown root:ssl-cert /etc/ssl/private/xrdp.key
sudo chmod 640 /etc/ssl/private/xrdp.key
sudo chmod 644 /etc/ssl/certs/xrdp.crt
```

3. Configure o XRDP para usar o novo certificado, edite o arquivo /etc/xrdp/xrdp.ini:
```
sudo editor /etc/xrdp/xrdp.ini
```
E inclua o seguinte conteúdo:  
```  
certificate=/etc/ssl/certs/xrdp.crt
key_file=/etc/ssl/private/xrdp.key
```
4. Reinicie o serviço XRDP, execute:
```
sudo systemctl daemon-reload
sudo systemctl restart xrdp
sudo systemctl status xrdp
```
Agora, ao se conectar via RDP (Windows, Remmina, etc.), o XRDP usará seu certificado SSL personalizado. Mas vamos validar o certificado manualmente, execute:  
```  
openssl x509 -in /etc/ssl/certs/xrdp.crt -text -noout
```
Pronto, agora o resultado esperado é:  
* Conexão RDP criptografada com TLS
* Nenhum erro relacionado a certificado inválido ou permissão negada  
* Logs limpos em /var/log/xrdp-sesman.log e /var/log/xrdp.log  

Este certificado local funcionará em sua rede local, mas se for para um acesso externo, precisará do 'certbot' que crescente uma seção opcional com integração via certificado Let’s Encrypt (SSL público e renovável) para uso remoto pela internet.

### COMO TESTAR
Você pode testar usando o Remmina(Linux) ou Remote Desktop(Windows) e apontando para o IP do seu Linux.  
