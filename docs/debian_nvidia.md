# **Instalar Drivers NVIDIA no Debian 13+ usando o repositório oficial da NVIDIA**

A seguir você encontrará um guia seguro e atualizado para instalar os drivers proprietários da NVIDIA no Debian 13 usando **o repositório oficial da NVIDIA** em vez dos repositórios Debian.
Esse método fornece drivers mais recentes, compatíveis com Wayland e com suporte pleno ao DKMS.  
Também alguns softwares funcionam melhor com os drivers nativos da NVIDIA, por exemplo, o OBS Studio, DaVinci Resolve, Audacity e seus plugins de processamento baseado em IA, entre outros.  

Sistemas com GPUs Intel ou AMD não precisam desses procedimentos, pois essas empresas fornecem drivers 100% open source já integrados ao kernel e ao Mesa.  
A NVIDIA recentemente integrou parte do seu driver ao kernel, mas essa integração é parcial e não inclui o restante do stack gráfico (OpenGL, Vulkan, CUDA, NVENC etc.), que ainda depende do driver proprietário completo.  

---

## 1. Identificar qual GPU NVIDIA você possui

### Por que este passo é importante?

Saber qual GPU você tem ajuda a confirmar se ela é suportada pelos drivers atuais.
Placas antigas (Legacy) usam drivers diferentes das placas modernas.
Esse passo evita instalar um driver incompatível que pode causar tela preta.

Execute:  
```bash
lspci | grep -i nvidia
```
Será mostrado algo como:
> 01:00.0 VGA compatible controller: NVIDIA Corporation TU117M [GeForce GTX 1650 Mobile / Max-Q] (rev a1)  
> 01:00.1 Audio device: NVIDIA Corporation Device 10fa (rev a1)

Isso indicará que há uma placa NVIDIA no sistema.  

---

## **2. Habilitar contrib e non-free-firmware**

### **Por que isso é necessário?**

Os drivers proprietários da NVIDIA dependem de componentes que não fazem parte do repositório *main* do Debian.
Sem **contrib**, **non-free** e **non-free-firmware**, o sistema não encontrará bibliotecas essenciais para o funcionamento correto do driver.

Como fazer? Edite:  
```bash
sudo editor /etc/apt/sources.list
```

Confirme que as linhas contêm:
```
deb http://deb.debian.org/debian trixie main contrib non-free non-free-firmware
deb http://security.debian.org/debian-security trixie-security main contrib non-free non-free-firmware
```

Atualize:

```bash
sudo apt update
```
Geralmente a parte `non-free` e `non-free-firmware` não está habilitado, se for este o caso, inclua-os.  


---

## 3. Adicionar o repositório oficial da NVIDIA
O Debian 13 inclui drivers da NVIDIA, mas eles costumam atrasar algumas versões em relação ao lançamento oficial da NVIDIA.
O repositório oficial fornece:

* versões mais recentes e otimizadas
* melhor suporte a Wayland
* correções de bugs recentes
* atualizações via APT (com DKMS automaticamente recompilado)

Esse método evita usar o instalador `.run` — que **não deve ser usado no Debian**.

Vamos instalar dependências necessárias:
```bash
sudo apt install curl wget gnupg apt-transport-https dkms
```

---
## 4. Vamos adicionar a chave GPG da NVIDIA:   
Isso é importante porque o APT só instala pacotes de repositórios cuja assinatura digital é válida.
Sem a chave GPG correta, o Debian irá recusar os pacotes do repositório NVIDIA.

```bash
curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/8793F200.pub | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia.gpg
```
**Observação**: Se o comando acima gerar um erro 404 é porque a URL acima mudou, neste caso, acesse a raiz, isto é, https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/ e procure por um arquivo que seja .pub e substitua-o na URL acima a menção de 8793F200.pub.  O método é similar para outras distros Debian-Like, trocando o nome Debian13 pelo nome/release da distro que estiver usando.  

---

## 5. Agora vamos registrar o repositório NVIDIA:  
Vamos dizer ao Debian que estamos acrescentando mais um repositório ao sistema, então crie o arquivo a seguir:   
```bash
sudo editor /etc/apt/sources.list.d/nvidia.list
```

E cole o seguinte conteúdo:  
```
deb [signed-by=/usr/share/keyrings/nvidia.gpg] https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/ /
```

Depois, atualize os índices do repositório:

```bash
sudo apt update
```

---
## 6. Desabilitar o driver Nouveau
É obrigatório desabilitar o driver(módulo) **Nouveau**, ele é o driver open source para placas NVIDIA, ele foi criado para dar alguma vida a essas placas cujo o suporte oficial é sofrível, no entanto, em hardwares modernos que vão além do básico, esse driver não responde bem e oferece apenas o básico e há algo nele que atrapalha a instalação do driver proprietário da **nvidia**: Ele carrega automaticamente no boot e, se estiver ativo, **tem precedência sobre o driver proprietário**, impedindo que o módulo oficial da NVIDIA seja carregado.
Resultado: conflito, tela preta ou fallback para renderização sem aceleração.  

Daí temos de bloqueá-lo para garantir que o kernel use exclusivamente o módulo `nvidia.ko`, proprietário da NVIDIA. Para essa ação, execute:  
```bash
echo 'blacklist nouveau' | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
echo 'options nouveau modeset=0' | sudo tee -a /etc/modprobe.d/blacklist-nouveau.conf
sudo update-initramfs -u
```
Depois disso, reinicie para remover o **noveau** do sistema, pois a instalação do driver da NVIDIA requer que este módulo não esteja na memória:  
```bash
sudo reboot
```

---

## 7. Instalar o driver NVIDIA
No Debian 13 (com Wayland), o driver da NVIDIA já é compatível nativamente, não há necessidade de editar xorg.conf (inclusive o Debian 13 nem usa Xorg por padrão) e assim o driver instalará automaticamente:

* módulo do kernel (via DKMS)
* suporte a aceleradores
* componentes Vulkan
* integração com Wayland (GBM + EGL Streams dependendo da versão)

## Conferencia recomendada:
A NVIDIA é confusa com drivers, atente-se que já houveram casos de atualizações da própria NVIDIA que danificaram GPUs, então a instalação de drivers errados ou extremamente atualizados são pontos que você deve assumir alguma dor de cabeça. Para diminuir os analgésicos, vamos ter que dividir os drivers da nvidia em duas categorias **legacy** para modelos mais antigos, e as **novas** que obviamente tratam de placas mais recentes. Instalar um driver novo, numa placa considerada **legacy** fará com que o driver não rode e seu monitor fique com a tela preta na maioria dos casos.  
Saiba também, que se for uma placa muito, mas muito antiga, a NVIDIA tem histórico de abandoná-las, isto é, deixar de fornecer drivers **legacy**.  
Para tentarmos detectar em qual categoria iremos entrar, instalamos este pacote:
```bash
sudo apt install nvidia-driver-assistant
```
E depois executamos:
```bash
nvidia-driver-assistant
```
E a saida do comando nos dirá o que devemos fazer ou instalar:
```
Detected GPUs:
  GeForce GTX 550 Ti - (pci_id 0x1244)

Detected system:
  Debian GNU/Linux 13

Please copy and paste the following command to install the legacy kernel module flavour:
  sudo apt-get install -Vy cuda-drivers
```
- No exemplo acima, o comando não disse para instalar algo como 'nvidia-driver-******', então para esta placa de video (GeForce GTX 550 Ti) - segundo o **nvidia-driver-assistant** não se trata nem mesmo de uma placa na categoria **legacy** e por isso ele está recomendando apenas a instalação dos drivers CUDA que darão suporte a aceleração e processamento de dados, mas não terei nenhum processamento de saída de vídeo. Neste caso, a resposta oficial é que não tenho drivers nem mesmo para o  **legacy**.  Outros tipos de saída, vão recomendar além do cuda-drivers, também o `nvidia-driver` ou `nvidia-legacy`(os nomes podem mudar), daí então você executa o comando que for recomendado, por exemplo:

Instalação recomendada para placas suportadas(non-legacy):  
```bash
sudo apt install nvidia-driver
```
ou a instalação recomendada para placas legacy:  
```bash
sudo apt install nvidia-legacy
```
Os nomes **nvidia-driver** e **nvidia-legacy** são meta-pacotes que apontam para a ultima versão do driver novo e legacy, mas nem sempre a última versão é a melhor, as vezes precisamos escolher uma versão intermediária, isso é possivel vasculhando as versões existentes com o comando:  
```bash
sudo apt-cache search nvidia-legacy|grep driver # para ver apenas as legacy
# ou para ver apenas as recentes(non-legacy):  
sudo apt-cache search nvidia|grep -v legacy|grep driver
```
E ao ver as versões existentes, então instalar uma versão específica como:  
```bash
sudo apt install nvidia-legacy-390xx-driver
```
A numeração (390xx) pode mudar conforme o repositório liberar novas versões. A curadoria dessas versões é do time da NVIDIA, então não culpe o time do Debian se nenhuma dessas versões funcionar em seu sistema. Tanto o Debian como outras distros estão adotando o Wayland em detrimento do Xorg, e os drivers mais antigos da NVIDIA só funcionam com o Xorg.  
Embora ela tenha melhorado, a NVidia criou muita antipatia pelo tratamento que ela tem dado a seus usuários e comunidade opensource e linux.  

---


## 8. Reiniciar

Reinicie para aplicar o novo kernel + módulos:

```bash
sudo reboot
```

---

## 9. Confirmar que a instalação funcionou

Agora precisamos testar para garantir que:

* o módulo da NVIDIA carregou
* o Wayland está usando a aceleração da NVIDIA
* o DKMS recompilou corretamente

Vamos ao teste principal, execute:  
```bash
nvidia-smi
```
E então é esperado mostrar algo como:  
```
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.105.08             Driver Version: 580.105.08     CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|                                         |                        |               MIG M. |
|=========================================+========================+======================|
|   0  NVIDIA GeForce GTX 1650        Off |   00000000:01:00.0  On |                  N/A |
| N/A   36C    P8              1W /   50W |     100MiB /   4096MiB |      0%      Default |
|                                         |                        |                  N/A |
+-----------------------------------------+------------------------+----------------------+

+-----------------------------------------------------------------------------------------+
| Processes:                                                                              |
|  GPU   GI   CI              PID   Type   Process name                        GPU Memory |
|        ID   ID                                                               Usage      |
|=========================================================================================|
|    0   N/A  N/A            2211      G   /usr/bin/kwin_wayland                     1MiB |
+-----------------------------------------------------------------------------------------+
```
Se aparecer a tabela com informações da GPU, tudo está funcionando.  

Agora, vamos verificar se Wayland está ativo, execute:  
```bash
echo $XDG_SESSION_TYPE
```

Saída esperada:  
> wayland  
  
---

## 10. NVIDIA-SETTINGS
Há um programa que nem sempre é instalado por padrão via meta-pacote, ele chama-se `nvidia-settings` e com ele podemos ver a situação da placa de vídeo como temperatura, fans e processamento. Para instalar, execute:
```bash
sudo apt install nvidia-settings
```

---
## 11. API VULKAN
Uma coisa interessante sobre o Vulkan é que ele também requer componentes instalados para ele, mas diferentes de drivers de sistemas, são softwares que fazem a ligação da placa de rede com a principal API usada para aceleração de gráficos via hardware chamada **Vulkan**. Então vamos instalar este componente, execute:
```bash
sudo apt install nvidia-vulkan-icd
```
---
## 12. NVIDIA-PRIME
Se seu computador é um notebook hibrido, isto é, vem com uma APU Intel e uma GPU NVIDIA então parabens, você é um usuário da plataforma **Prime** que é uma grande sacada para notebooks terem uma longetividade maior de bateria, usando a APU da Intel para o trabalho, mas ter a opção de executar programas usando a GPU da NVIDIA. Se é este o seu caso, execute também:  
```bash
sudo apt install nvidia-prime
```
Na plataforma prime também é possivel abdicar da Intel e usar somente NVIDIA, basta executar:  
```bash
sudo prime-select nvidia
```
Em ambientes GNOME e KDE, quando habilitado o **Prime**, o seletor de programas inclui a opção de **Executar com NVIDIA** e assim, você pode executar especialmente jogos diretamente com a GPU NVDIA.  

---
## 13. Como reverter caso dê problema
Se você instalou uma versão muito nova que apresenta bugs, pode voltar para o Nouveau ou para uma versão do Debian.

Para remover completamente:

```bash
sudo apt purge nvidia-driver nvidia-*
```
E então remova o arquivo `/etc/modprobe.d/blacklist-nouveau.conf`:  
```bash
sudo rm -f /etc/modprobe.d/blacklist-nouveau.conf
```
Depois atualizamos nosso boot, execute:  
```bash
sudo update-initramfs -u 
sudo reboot
```
Apenas estes passos retornarão o sistema ao original.  

---

## Conclusão

Este é o método mais seguro e moderno para instalar NVIDIA no Debian 13:

* total compatibilidade com Wayland
* drivers sempre atualizados
* sem instaladores problemáticos
* suporte a DKMS
* sem configurações manuais de Xorg


----

[Clique aqui para retornar a página principal](../README.md#nvidia-no-debian)
