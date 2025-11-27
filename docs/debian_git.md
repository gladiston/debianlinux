# GIT
O Git é um dos sistemas de controle de versão mais utilizados no mundo do desenvolvimento de software. Ele permite gerenciar projetos de forma colaborativa, acompanhar alterações no código e garantir segurança e rastreabilidade em cada modificação.

No Linux, especialmente nas distribuições Debian e Ubuntu, a instalação e configuração do Git são simples, mas recentes mudanças no GitHub exigem ajustes adicionais para autenticação segura. Este tutorial mostrará como preparar seu ambiente corretamente, compilar o suporte ao libsecret (necessário para armazenar credenciais com segurança) e configurar o Git para utilizar tokens de acesso pessoal, substituindo o antigo método de login por senha.

Vamos ajustar nosso ambiente com o GIT com os comandos:
```
git config --global user.name "Seu nome completo"
git config --global user.email "seu.email@dominio.com"
```

Recentemente, o github fez alterações em seu sistema onde a instrução:
```
git config --global credential.helper 'cache --timeout=28800'
```

Será ignorada completamente ou terminará em erro e a tentativa de login resultará neste erro:
**Fatal Authentication Failed for: site.com.br**

Para solucionar o problema, precisará de mais alguns pacotes:
```
sudo apt install -y libsecret-1-0 libsecret-tools libsecret-1-dev build-essential
```
Agora vamos até o código fonte:  
```
cd /usr/share/doc/git/contrib/credential/libsecret
```
E depois compilar:
```
sudo make
```
Após, o git só precisará dessa configuração adicional:
```
git config --global credential.helper /usr/share/doc/git/contrib/credential/libsecret/git-credential-libsecret
```
Agora, você precisa saber que o método de autenticação mudou, você não usa mais o “username+ senha” do seu usuario git, mas “username+token”. O token é criado na página do github, no menu de sua profile->Settings->Developer settings->Personal access tokens->Tokens(classic) e então criar um token. Este token será o substituto de sua senha git no terminal.



----

[Clique aqui para retornar a página principal](../README.md#git)
