# FIREWALL 
Um sistema de firewall geralmente vem instalado por padrão, é chamado de `iptables`, mas ele não vem ativado e nem acompanha um **frontend (aplicativos visuais) para ele**. O motivo de não vir **ativado** é que o usuário comum ficaria frustado em descobrir porque certas coisas não funcionam como ele quer, e a razão disso pode ser um bloqueio de porta, o exemplo mais típico dentro de empresas são as impressoras de rede, cada modelo como HP, EPSON, XEROX,.. usam portas diferentes para serem encontradas na rede, elas geralmente fazem broadcast para que sistemas operacionais as encontrem e se seu sistema tiver exatamente estas portas bloqueadas, tais impressoras não serão encontradas, por isso, use de cautela se desejar realmente instalar o Firewall, se for iniciante, recomendo que não faça isso.  

Mas se é um desenvolvedor ou administrador de rede e entende os impactos de segurança que um Firewall e concordar com eles, então, o primeiro passo é instalá-lo manualmente. Vamos optar pelo **firewalld**, pois ele é o padrão no Fedora, RHEL, AlmaLinux e openSUSE, além de ser totalmente compatível com Debian, Ubuntu e distro derivadas. Interfaces como KDE e GNOME trazem frontend para gerenciar e liberar portas. E também, essa escolha garante comandos consistentes e portabilidade entre diferentes ambientes Linux.  
  
Muitas pessoas argumentam que um firewall é desnecessário em estações de trabalho Linux, e até certo ponto isso é verdade para uso doméstico. Contudo, se você é desenvolvedor ou administrador de sistemas, é essencial que o ambiente de desenvolvimento seja o mais parecido possível com o ambiente de produção — e este quase sempre possui um firewall ativo.  
**Em resumo**: instalar o **firewalld** no seu ambiente desktop não é apenas por segurança, mas por coerência e preparo profissional.  

Instale o **firewalld**:  
```
sudo apt install -y firewalld
```
Em seguida, habilite e inicie o serviço:
```
sudo systemctl enable firewalld
sudo systemctl start firewalld
```
Verifique as portas atualmente liberadas:  
```  
sudo firewall-cmd --list-ports
```
Provavelmente não mostrará nada, mas vamos liberar algumas portas para servir de exemplo.  


## LIBERANDO TEMPORARIAMENTE PORTAS NO FIREWALL
Para liberar portas, precisamos listar os serviços que iremos usar, alguns podem nem ter sido instalados ainda, veja esta lista que exemplifica os serviços/portas que pretendemos liberar:  
|Serviço      |Porta  |Descrição|
|-------------|:-----:|---------|
|HTTP         |80     |Tráfego web padrão (sites sem HTTPS)|
|HTTPS        |443    |Tráfego web seguro (SSL/TLS)|
|SSH          |22     |Acesso remoto seguro a servidores Linux|
|RDP          |3389   |Acesso remoto do Windows (XRDP também usa)|
|MySQL/MariaDB|3306   |Banco de dados |
|PostgreSQL   |5432   |Banco de dados|
|Firebird     |3050   |Banco de dados|

Com base na lista acima, os comandos seriam:  
```
sudo firewall-cmd --add-port=80/tcp
sudo firewall-cmd --add-port=443/tcp
sudo firewall-cmd --add-port=22/tcp
sudo firewall-cmd --add-port=3389/tcp
sudo firewall-cmd --add-port=3306/tcp
sudo firewall-cmd --add-port=5432/tcp
sudo firewall-cmd --add-port=3050/tcp
```
Agora vamos repetir a verificação das portas atualmente liberadas:  
```  
sudo firewall-cmd --list-ports
```
E observe o resultado:  
> 22/tcp 80/tcp 443/tcp 3050/tcp 3306/tcp 3389/tcp 5432/tcp   
  
Isso significa que obtivemos sucesso, no entanto, essas regras são temporarias até reiniciar o firewalld ou o sistema.  

## LIBERANDO PERMANENTEMENTE PORTAS NO FIREWALL
Como vimos no passo anterior, as regras são temporarias, elas valem apenas até reiniciar o firewalld ou o sistema. Para torná-las permanentes, execute:  
```
$ sudo firewall-cmd --runtime-to-permanent
success
```
Agora, vamos reiniciar o firewall:  
```
sudo firewall-cmd --reload
```
Agora vamos repetir a verificação das portas atualmente liberadas:  
```  
sudo firewall-cmd --list-ports
```
E observe o resultado:  
> 22/tcp 80/tcp 443/tcp 3050/tcp 3306/tcp 3389/tcp 5432/tcp  

Como pode observar acima, as regras não sumiram. Então, quando precisar de regras permanentes faça isso.  

## LIBERANDO PERMANENTEMENTE PORTAS NO FIREWALL POR PERFIL
Assim como outros sistemas de firewall, o Firewalld trabalha com o conceito de perfis, chamados de zonas (zones).  
A ideia é simples: cada zona representa um conjunto de regras de segurança.  
  
Por exemplo, você pode ter uma zona para virtualização, outra para desenvolvimento e outra para uso pessoal.  
Quando muda de zona, o Firewalld desativa as regras da anterior e aplica as novas, permitindo perfis de rede específicos para cada tipo de tarefa.  
  
Por padrão, o Firewalld traz apenas uma zona ativa chamada public, que não possui regras liberadas inicialmente.  
No entanto, essa zona é herdada por todas as outras, ou seja, qualquer regra adicionada a public se aplicará às demais zonas também.  
  
Isso é bastante útil — por exemplo, se você quiser liberar as portas usadas pelo CUPS e certas impressoras de rede, basta adicioná-la à zona public e ela valerá para todos os perfis, execute:    

**IPP e IPPS (CUPS, impressão e status):**  
```bash
sudo firewall-cmd --zone=public --add-port=631/tcp --permanent
sudo firewall-cmd --zone=public --add-port=631/udp --permanent
sudo firewall-cmd --zone=public --add-port=443/tcp --permanent
```

**Impressão via Windows/SMB:**  
```bash
sudo firewall-cmd --zone=public --add-port=139/tcp --permanent
sudo firewall-cmd --zone=public --add-port=445/tcp --permanent
sudo firewall-cmd --zone=public --add-service=samba-client --permanent
```

**Impressão LPD/LPR:**  
```bash
sudo firewall-cmd --zone=public --add-port=515/tcp --permanent
```

**Impressão RAW (JetDirect/AppSocket) – HP, Brother, Canon, Epson:**  
```bash
sudo firewall-cmd --zone=public --add-port=9100-9200/tcp --permanent
```

**Descoberta de impressoras (Bonjour/mDNS):**  
```bash
sudo firewall-cmd --zone=public --add-port=5353/udp --permanent
sudo firewall-cmd --zone=public --add-service=mdns --permanent
```

**Monitoramento SNMP (opcional, para status de impressoras de rede)**  
```bash
sudo firewall-cmd --zone=public --add-port=161/udp --permanent
```  
**Recarregando o Firewall:**
```bash
sudo firewall-cmd --reload
```

Aproveite este momento para identificar quais portas precisam ser liberadas e aplique-as conforme sua necessidade.  
As portas que forem de uso comum a todos os perfis (zonas) devem ser adicionadas à zona public, garantindo que estejam disponíveis independentemente do perfil ativo.  

>**IMPORTANTE**:
>Embora muitos considerem o uso de um firewall opcional em ambientes Linux de desktop, eu discordo totalmente. Mesmo em estações de trabalho, é fundamental manter um firewall ativo e configurado, pois ele protege serviços locais e deixa seu ambiente mais próximo do ambiente de produção, onde o firewall quase sempre está habilitado.  No entanto, nunca desative o firewall permanentemente ou ignore políticas básicas de segurança — isso elimina uma camada essencial de proteção que o Linux oferece por padrão.  


----

[Clique aqui para retornar a página principal](../README.md#firewall)
