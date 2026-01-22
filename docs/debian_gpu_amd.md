# PLACAS DE VIDEO AMD
Algumas placas de video no Linux entram numa "zona cinzenta" onde existe tanto o driver antigo como também o novo são funcionais, 
porém o driver novo recebe atualizações e o antigo é apenas legado e não recebe nem correções ou novos recursos.
Você verá no exemplo `Radeon R7` exatamente isso, ele carrega o driver legado que funciona, mas ao precisar de aceleração grafica 
como no navegador Google Chrome uma pixação estranha e renderização toda desengonçada e as vezes, o sistema trava.  
Sim, "trava". Estamos lidando com drivers de modulo do kernel, erros aqui podem levar a máquina a bancarrota.  
Se você passa por uma situação assim, talvez as instruções a seguir sirvam para você como serviram para mim.  


## Identifique sua placa
Execute:
```bash
lspci -k |grep -A 3 -E "VGA|3D"
```
E veja o resultado:
```
06:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Oland PRO [Radeon R7 240/340 / Radeon 520]
        Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Device 0000
        Kernel driver in use: radeon
        Kernel modules: radeon, amdgpu
```
Agora que sabemos o modelo podemos resolver o problema.  

## Radeon R7 240/340 / Radeon 520
Execute:
```bash
lspci -k |grep -A 3 -E "VGA|3D"
```
E veja o resultado:
```
06:00.0 VGA compatible controller: Advanced Micro Devices, Inc. [AMD/ATI] Oland PRO [Radeon R7 240/340 / Radeon 520]
        Subsystem: Advanced Micro Devices, Inc. [AMD/ATI] Device 0000
        Kernel driver in use: radeon
        Kernel modules: radeon, amdgpu
```
Note a frase:  
> Kernel driver in use: radeon
> Kernel modules: radeon, amdgpu  

Isso significa que há dois modulos, porém o que o kernel decidiu usar foi o `radeon` que não é muito recomendado para aceleração de hardware, precisaremos trocar para `amdgpu`.  
Precisaremos dizer ao sistema para não carregar o suporte antigo para essa arquitetura chamada Southrn Islands/Sea Islands.  
1. No terminal, execute:
```bash
sudo editor /etc/default/grub
```
Procure pela linha que tenha `GRUB_CMDLINE_LINUX_DEFAULT`, exemplo:
> GRUB_CMDLINE_LINUX_DEFAULT="quiet"

Comente essa linha, e depois a duplique ficando a primeira linha comentada e a segunda(exatamente igual) não comentada.

Agora, dentro das aspas, após a ultima instrução - em nosso exemplo "quiet" - , acrescente as seguintes instruções na mesma linha até fechar aspas:
> radeon.si_support=0  
> radeon.cik_support=0
> amdgpu.si_support=1
> amdgpu.cik_support=1

Ficando assim:
```
#GRUB_CMDLINE_LINUX_DEFAULT="quiet"
GRUB_CMDLINE_LINUX_DEFAULT="quiet radeon.si_support=0 radeon.cik_support=0 amdgpu.si_support=1 amdgpu.cik_support=1"  
```
Salve o arquivo e volte ao prompt.  
Para efetivar as mudanças, execute: 
```bash
sudo update-grub
```
E depois reinicie o computador, execute:  
```bash
sudo reboot
```
A placa de vídeo mencionada Radeon R7 240/340/550 e similares - tem também a Oland PRO entram numa "zona cinzenta" do Linux. 
O driver `readon` é o padrão por questões históricas, mas o `amdgpu` é o que recebe atualizações de performance. 
Ao desativar o suporte no `radeon` e ativar no `amdgpu`, nós obrigamos o sistema a usar o driver moderno.  


----

[Clique aqui para retornar a página principal](../README.md#vga-radeon-antigas)


