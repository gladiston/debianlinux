Hoje é o seu dia de sorte, o que vou explicar aqui é um material dificil de encontrar na internet.  
Eu tenho um servidor ou desktop Windows, o RDS está ativo nele na porta **3389** e há um programa nele que desejo muito rodar em meu ambiente Linux, e aqui não importa se estamos usando Debian, Ubuntu, Fedora ou que distro seja, como rodar este programa?  
A resposta é por usar um cliente RDP como o Remmina, isso foi fácil de explicar, o que é mais dificil é como rodar este programa no modo conhecido como **SeamLess**, isto é, como rodá-lo como **RemoteApp** onde o comportamento deste programa sejá exatamente como no ambiente do hospedeiro, se eu minimizar o programa ele minimizará dentro do painel do meu Linux e não dentro da janela do Remmina.  
Esse tipo de comportamento é mais complicado de obter via internet, pois depende muito da experiência do profissional com o ambiente Linux e também de como o Windows roda RemoteApps.  

Primeiro, não usaremos o badalado Remmina porque ele não tem a opção de rodar programar no modo **SeamLess**, iremos usar o programa **freerdp3-wayland**, para instalar:  
```bash
sudo apt install -y freerdp3-wayland
```

Agora, eu preciso dos dados de autenticação, em que computador o programa está, e qual programa pretendo rodar, como exemplo, vamos simular este programa chamado `IBExpert` que é um gerenciador de banco de dados para Interbase/FirebirdSQL, ele tem versão apenas para Windows e embora rode via Bootle localmente, eu prefiro rodá-lo diretamente de uma estação Windows em que eu tenha ele instalado porque esta estação de trabalho tem vários programas de Windows instalados onde uma equipe de programadores quando precisam rodar um desses programas, simplesmente vão rodar remotamente e assim centralizo todos as aplicações Windows no mesmo lugar.  
Iremos usar o programa para Linux intitulado `freerdp` e para usá-lo alguns parametros serão obrigatórios:

## 🔑 Parâmetros Essenciais do FreeRDP

| Parâmetro | Descrição | Exemplo de Valor |
| :--- | :--- | :--- | :--- |
| **`/v:`** | **Servidor Remoto** (Endereço IP ou Nome DNS). | `/v:192.168.1.11` |
| **`/u:`** | **Nome de Usuário** para a autenticação no Windows Server. | `/u:gsantana` |
| **`/p:`** | **Senha** do usuário. | `/p:teste123` | 
| **`/app:`** | **Caminho Completo do Executável** no servidor remoto. | `/app:"C:\Program Files (x86)\HK-Software\IBExpert\IBExpert.exe"` | 

Baseado nas informações acima, iremos executar o programa da segunte forma:  

```bash
xfreerdp /v:192.168.1.11 /u:gsantana /p:teste123 /app:"C:\Program Files (x86)\HK-Software\IBExpert\IBExpert.exe"
```

A *flag* **`+seamless`** não é estritamente obrigatória para a **conexão funcionar**, mas é altamente recomendada para garantir que a **janela se integre** corretamente no seu desktop Linux.

