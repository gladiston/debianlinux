# INCLUINDO O REPOSITÓRIO DA MICROSOFT
Sim, a Microsoft tem um repositório para distribuições *Debian-like*, o que inclui também as outras derivações como **Ubuntu** e seus sabores, **Linux Mint**, **ZorinOS**, entre outras.  
Não vamos instalar nada de lá ainda; vamos apenas incluir o repositório. E, por mais paradoxal que seja, há um *download* e instalação necessários justamente para que possamos ter esse repositório.  

Descubra a versão do seu Ubuntu executando:
```bash
cat /etc/issue
```
Exemplo de saída:
```
Debian GNU/Linux 13 \n \l
```
No exemplo acima, a versão é o **Debian 13**.  
Agora visite a página:  
[https://packages.microsoft.com/config/](https://packages.microsoft.com/config/)  
Você verá algo assim:  
```
Index of config/
../
alma/                                                                                                 
amazonlinux/                                                                                          
centos/                                                                                               
debian/                                                                                               
fedora/                                                                                               
opensuse/                                                                                             
rhel/                                                                                                 
rocky/                                                                                                
sles/                                                                                                 
ubuntu/                                                                                               
```
Se for Ubuntu ou Debian, acesse a página correspondente a sua distribuição e uma vez que escolhê-la, aparecerá então pastas com as versões suportadas pela distro escolhida e então acesse a pasta correspondente à sua versão, e baixe o arquivo **packages-microsoft-prod.deb**.  
Depois, dê um duplo clique nele para instalá-lo.  

Atualize os repositórios:
```bash
sudo apt update -y
```

Isso criará o arquivo **/etc/apt/sources.list.d/microsoft-prod.list**, que aponta para o repositório oficial da Microsoft.  

Está curioso para saber o que a Microsoft está compartilhando?  
Então execute:
```bash
apt-cache policy | grep packages.microsoft.com
```
Mas antecipando, temos essencialmente as bibliotecas do `mono`.  

É curioso que a atualização do repositório da Microsoft seja mantida por um pacote que precisa ser instalado manualmente — e depois ele mesmo passa a ser atualizado pelo próprio repositório.  
Isso sim é uma implementação diferenciada.  
O time da Microsoft claramente não conhece a oração dos programadores em C/C++:  
> “Salve-nos da recursividade; main().”  - hahahhahahah.





----

[Clique aqui para retornar a página principal](../README.md#incluindo-o-reposit%C3%B3rio-da-microsoft)
