# MONITORANDO NOBREAK RAGTECH EASY PRO NO DEBIAN 13

Ter um nobreak (ou **UPS**, do inglês *Uninterruptible Power Supply*) é como ter um seguro de vida para seus dados. Mas atenção: para desktops modernos equipados com fontes de **PFC Ativo**, o UPS **deve ser senoidal**. Isso garante que ele entregue uma onda de energia "limpa", idêntica à da rede elétrica. O **Ragtech Easy Pro** entrega essa onda senoidal pura, sendo a escolha ideal para proteger sua estação Linux.

Embora o fabricante ofereça o software *Supervise Personal*, ele é datado e depende de bibliotecas antigas como **GTK 1.2/2.0**. Por isso, usaremos o **NUT (Network UPS Tools)**, o padrão moderno e nativo de monitoramento no Linux.

---

## O DILEMA DA COMUNICAÇÃO: HID VS. SERIAL

A maioria dos UPS USB usa o padrão **HID** (como um teclado), mas este Ragtech utiliza o padrão **Serial-over-USB (CDC-ACM)**. Isso significa que, embora o cabo seja USB, o sistema o identifica como uma porta serial mapeada no caminho `/dev/ttyACM0`.

---

## PASSO A PASSO DA INSTALAÇÃO

### 1. Instalação dos Pacotes Necessários

O **NUT** é o conjunto de ferramentas que gerencia a comunicação e o desligamento automático. Também instalaremos o **psmisc**, que contém o utilitário `fuser`, fundamental para identificar se algum outro programa está "sequestrando" a porta do nobreak.

```bash
sudo apt update && sudo apt install -y nut nut-client nut-server psmisc
sudo systemctl restart nut-server nut-client
```

### 2. Identificação do Dispositivo

Precisamos confirmar se o Kernel (o núcleo do Linux) criou o arquivo de acesso para a porta serial virtual do nobreak. Vamos listar o dispositivo. Há duas informações imprescindíveis que precisamos obter sobre o hardware: **idVendor** e **idProduct**, e para isso, desplugue o seu UPS por um instante, aguarde uns 5 segundos e plugue-o novamente, agora execute o seguinte comando:

```bash
sudo dmesg -w
```
O comando acima listará as ultimas linhas do log do sistema e continuará a observar, por isso vai precisar dar **Ctrl+C** para sair. Essas ultimas linhas provavelmente conterá o reconhecimento do UPS, algo como:

```text
(...)
usb 1-4: new full-speed USB device number 12 using xhci_hcd
[ 6822.272049] usb 1-4: New USB device found, idVendor=04d8, idProduct=000a, bcdDevice= 1.00
[ 6822.272061] usb 1-4: New USB device strings: Mfr=1, Product=2, SerialNumber=0
[ 6822.272064] usb 1-4: Product: USB Serial Port
[ 6822.272067] usb 1-4: Manufacturer: Ragtech Sistemas de Energia
[ 6822.291809] cdc_acm 1-4:1.0: ttyACM0: USB ACM device

```

Agora, obtivemos a seguinte informação sobre o nosso UPS:

* Fabricante e modelo: **Manufacturer: Ragtech Sistemas de Energia**
* Identificação única: **idVendor=04d8** e **idProduct=000a**
* Porta:  **ttyACM0**, isto é, **/dev/ttyACM0**.

As informações acima são especificas do hardware e é aqui que alguns tutoriais falham quando as pessoas copiam e colam cegamente e não notam que resultados de comandos de reconhecimento modificarão os passos seguintes.

Vamos conferir a presença do dispositivo em nosso sistema, execute:

```bash
ls -l /dev/ttyACM0
```

**Resultado esperado:**

```text
crw-rw---- 1 root dialout 166, 0 mar  5 10:14 /dev/ttyACM0

```
A presença desse dispositivo indica que o kernel pode conversar com o UPS.


### 3. O Teste do "Aperto de Mão" (Handshake)

Antes de editarmos arquivos de configuração, vamos validar se o hardware realmente responde. Este comando envia uma pergunta de status (`Q1`) e aguarda a resposta por 2 segundos:

```bash
echo -e "Q1\r" | sudo tee /dev/ttyACM0 > /dev/null && sudo timeout 2s cat /dev/ttyACM0 | od -x


```

**Resultado esperado:**

```text
0000000 00ca
0000001
```

**O que este resultado significa?**

* **O valor `00ca**`: Em hexadecimal, `ca` representa o número **202** em decimal. No protocolo desses aparelhos, isso indica que o nobreak respondeu com um dado real. Receber esses números é a prova cabal de que a comunicação física está perfeita!

### 4. Ajuste de Permissões e Regra UDEV (Definitivo)
No Linux, portas seriais como **/dev/ttyACM0** são restritas e suas permissões voláteis, ou seja, se eu der um `chmod 666 **/dev/ttyACM0` para que qualquer usuário tenha acesso a ele, o que acontece se eu despluar? O device **/dev/ttyACM0** deixa de existir e quando plugo novamente, **/dev/ttyACM0** será recriado, mas não com as permissões antigas(666), mas com as permissões default. Para que o NUT funcione sempre (mesmo após reiniciar), precisamos de uma regra de **udev**. O udev é o gerente de dispositivos que aplicará as permissões -666- automaticamente assim que o hardware é reconhecido.

Primeiro, vamos colocar o usuário **'nut'** nos grupos de comunicação:

```bash
sudo usermod -aG dialout nut
sudo usermod -aG tty nut
```

Para criar uma regra 'udev' precisaremos de duas informações sobre o hardware  **idVendor** e **idProduct**, e graças ao passo anterior com `sudo dmesg -w` sabemos que o **idVendor=04d8** e **idProduct=000a**, também que a porta é **/dev/ttyACM0**. Então de posse desses dados vamos criar o arquivo de regra personalizada, execute:

```bash
sudo editor /etc/udev/rules.d/99-ragtech-ups.rules

```

Cole o seguinte conteúdo dentro do arquivo:

```text
SUBSYSTEM=="tty", ATTRS{idVendor}=="04d8", ATTRS{idProduct}=="000a", MODE="0666", GROUP="nut"
```

Para que o sistema reconheça a nova regra imediatamente sem precisar reiniciar, execute:

```bash
sudo udevadm control --reload-rules && sudo udevadm trigger

```

### 5. Configuração do Driver

O **Driver** é o tradutor que entende o "idioma" do UPS. Vamos configurar o arquivo de drivers do NUT:

```bash
sudo editor /etc/nut/ups.conf

```

Este arquivo geralmente está todo comentado e não há linhas configuradas. então cole no final deste arquivo o conteúdo abaixo:

```text
[meunobreak]
    driver = nutdrv_qx
    port = /dev/ttyACM0
    protocol = q1
    desc = "Ragtech Easy Pro Senoidal"


```

### 6. Configurando o Modo e o Monitoramento

Avise ao NUT que ele operará em modo **Standalone** (uso local) no arquivo `/etc/nut/nut.conf`:
```bash
sudo editor  /etc/nut/nut.conf
```
E então procure a linha `MODE=`, que talvez esteja assim:
```text
MODE=none
```
Comente a linha acima acrescentando "#" no inicio da linha, e acrescente abaixo dela:
```text
MODE=standalone
```
Talvez a linha acima esteja comentada, neste caso, apenas acrescente abaixo dela, o conteúdo acima.  

Agora, vamos editar o arquivo `/etc/nut/upsmon.conf`, execute:

```bash
sudo editor /etc/nut/upsmon.conf
```

E acrescente esta linha ao final do arquivo, mas atente-se para adaptá-la conforme a explicação seguinte:

```text
MONITOR meunobreak@localhost 1 monuser sua_senha_secreta master
```

**Entenda os parâmetros:**

* **meunobreak@localhost**: Nome do UPS definido no passo anterior.
* **1**: Indica que monitoramos apenas 1 nobreak.
* **monuser**: Usuário interno do serviço.
* **sua_senha_secreta**: Senha de comunicação interna. **Troque para uma de sua preferência!**
* **master**: Define que este computador controla o desligamento.

---

## INICIALIZAÇÃO E VALIDAÇÃO

Reinicie os serviços para carregar as configurações:

```bash
sudo systemctl restart nut-server nut-client
```

Consulte os dados reais do aparelho:

```bash
upsc meunobreak@localhost
```
O resultado esperado é uma lista de informações técnica como:
```text
battery.charge: Porcentagem da bateria.
input.voltage: Tensão da tomada (nossos ~127V ou ~220V).
ups.status: OL (Online/Na rede) ou OB (On Battery/Na bateria).
```

No entanto, pode aparecer um erro.

---

## E SE O COMANDO UPSC FALHAR? (DEPURAÇÃO)

Se ao executar `upsc meunobreak@localhost` você recebeu erros como "**Driver not connected**" ou "**Connection failure: Connection refused**", precisamos rodar o driver manualmente para ver o que está travando a comunicação.

**Primeiro, pare os serviços para liberar a porta:**

```bash
sudo systemctl stop nut-server nut-client

```

**Agora, inicie o driver manualmente em modo de depuração profunda:**

```bash
sudo /lib/nut/nutdrv_qx -u root -a meunobreak -DDD

```

**Resultado esperado (caso haja erro):**

```text
0.000220 [D1] upsdrv_initups...
0.001605 /dev/ttyACM0 is locked by another process

```

**O que fazer se o erro for `/dev/ttyACM0 is locked by another process`?**
Isso significa que outro programa está usando o nobreak. Para descobrir quem é o "culpado", usamos o comando `fuser` (do pacote `psmisc` que instalamos no início):

```bash
sudo fuser -v /dev/ttyACM0

```

Se aparecer um processo (como o **ModemManager**), finalize-o (kill -9 PID) e tente rodar o driver novamente. Quando ele conectar e mostrar os dados da bateria, aperte **Ctrl+C** e reinicie os serviços normalmente. Se você não precisa do **ModemManager** ou os bloqueios forem constantes feito por ele, simplesmente desative-o:

```bash
sudo systemctl stop ModemManager && sudo systemctl disable ModemManager

```

**O que fazer se o erro for `Error: Driver not connected`?**
Isso significa que o serviço principal do servidor NUT (`upsd`) está rodando, mas não consegue "conversar" com o driver (o nosso tradutor). Geralmente, isso acontece se o driver "morreu" logo após tentar iniciar porque não conseguiu ler a porta.

1. Verifique se o cabo USB está bem conectado.
2. Certifique-se de que a regra **udev** do Passo 4 foi aplicada corretamente (verifique as permissões com `ls -l /dev/ttyACM0`).
3. Tente rodar o driver em modo de depuração (comando logo acima) e procure por linhas como `send: 'Q1'` seguidas de `read: timeout (0)`. Se houver timeout contínuo, o driver está gritando no vazio e o nobreak não está respondendo. Neste caso, verifique se inseriu o `idVendor` e `idProduct` corretamente na sua regra.

**O que fazer se o erro for `upsnotify: failed to notify about state 4: no notification tech defined, will not spam more about it`?**
Não entre em pânico! Apesar da palavra "failed", esta mensagem é apenas um aviso inofensivo. O sistema de depuração do NUT está basicamente dizendo: *"Eu queria mandar um alerta para a sua área de trabalho avisando sobre uma mudança de status, mas não sei como fazer isso"*. Ela não impede o funcionamento do nobreak e pode ser sumariamente ignorada. O que realmente importa é se o comando de depuração conseguiu ler os dados da bateria.

**CONCLUSÃO**
Configurar um nobreak que se comunica via "Serial sobre USB" no Linux pode parecer um desafio digno de um hacker de cinema no início. No entanto, ao fugirmos de softwares proprietários obsoletos e abraçarmos o **NUT** junto com as regras definitivas do **udev**, criamos uma solução robusta, nativa e à prova de reinicializações. Agora o seu Debian 13 tem a inteligência e a autonomia para monitorar a rede elétrica e desligar o sistema com elegância antes que a bateria se esgote, protegendo o seu hardware e a integridade dos seus arquivos. Pode dormir tranquilo, o `upsc` está de vigia!

---

[Clique aqui para retornar a página principal](https://www.google.com/search?q=../README.md%23monitorando-nobreak-ragtech-easy-pro)
