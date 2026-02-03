# INSTALANDO O MICROSOFT EDGE
Algumas pessoas são apaixonados pelo navegador da Microsoft, se este é o seu caso, o navegador Microsoft Edge também está disponível para Linux, os procedimentos de instalação são similares ao Google Chrome, o que muda é basicamente o link para download, então vamos reprisar a instalação.  

[Acesse a página de download dos pacotes do navegador Edge](https://www.microsoft.com/pt-br/edge/download) e role a tela até encontrar a mensagem quase escondida:
> O Microsoft Edge já está disponível no Linux. Baixar para Linux (.deb)  

Clique então em **Baixar para Linux (.deb)**.  
Após o download, dê duplo clique no arquivo baixado. O sistema iniciará o instalador automaticamente; a partir daí, basta seguir as instruções exibidas na tela.  

Similar ao navegador do Google, a Microsoft também adiciona o repositório dela no sistema e as atualizações deste navegador serão automáticas. Mas você poderá encontrar um **Aviso** que pode te assustar quando tiver que atualizar seu repositório, veja:  
```bash
$ sudo apt update
Atingido:1 http://security.debian.org/debian-security trixie-security InRelease
Atingido:2 http://deb.debian.org/debian trixie InRelease                                          
Atingido:3 http://deb.debian.org/debian trixie-updates InRelease
Obter:4 https://packages.microsoft.com/repos/edge stable InRelease [3.590 B]                                                        
Atingido:5 https://dl.google.com/linux/chrome/deb stable InRelease
(...)
Obter:7 https://packages.microsoft.com/repos/edge stable/main amd64 Packages [24,4 kB]
Obtidos 28,0 kB em 1s (53,1 kB/s)   
Todos os pacotes estão atualizados.         
Aviso: https://packages.microsoft.com/repos/edge/dists/stable/InRelease: Policy will reject signature within a year, see --audit for details
```
O aviso é esse:  
>**Aviso: https://packages.microsoft.com/repos/edge/dists/stable/InRelease: Policy will reject signature within a year, see --audit for details**

Não precisa se preocupar — essa mensagem não indica erro, apenas um aviso de validade futura da assinatura GPG do repositório da Microsoft. O apt está informando que a chave usada para assinar o repositório do Edge vai expirar dentro de um ano. Isso é apenas uma precaução de segurança para lembrar que o repositório ainda é confiável e válido agora, mas o sistema está antecipando o aviso.



### Antes de executar pela primeira vez
Se você tiver uma cópia do Microsoft Edge em algum outro computador, você o copia de:
```
~/.config/microsoft-edge
```
do computador antigo para o novo, exatamente o mesmo lugar.   
Funciona? Sim, desde que sejam a mesma versão e o mesmo perfil, mas requeirerá as autenticações, aparentemente os cookies são invalidados.   
Este navegador quando é executado pela primeira vez, se apercebe da instalação anterior e faz upgrade, depois disso, não adianta mais fazer a cópia substituindo a configuração anterior, há alguma inteligência que bloqueia repetir o processo várias vezes, então a primeira cópia sempre funciona, as demais, nem sempre.  

----

[Clique aqui para retornar a página principal](../README.md#instalando-o-microsoft-edge)
