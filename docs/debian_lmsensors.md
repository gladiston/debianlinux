# MONITORANDO TEMPERATURAS COM O LM-SENSORS
O **lm-sensors** mostra as temperaturas, tensões e rotações das ventoinhas da sua placa-mãe e processador.  
É leve, simples e ótimo para acompanhar o aquecimento do sistema.

### Instalação
```bash
sudo apt install -y lm-sensors
```

### Detectando sensores
Antes de usar, detecte os sensores com:
```bash
sudo sensors-detect
```
Boa parte das perguntas já vem com **yes** prédefinido, as outras, você só pode responder com **yes** se tiver os módulos-sensores previamente carregados. Depois de respondidas as questões, poderá executar o comando `sensors` como será mostrado no próximo tópico.   


### Uso prático
Depois disso, basta:

```bash
sensors
```
E vará uma saída como esta:
```
amdgpu-pci-0a00
Adapter: PCI adapter
vddgfx:        1.10 V  
vddnb:       862.00 mV 
edge:         +26.0°C  
PPT:          11.00 W  
sclk:         400 MHz 

k10temp-pci-00c3
Adapter: PCI adapter
Tctl:         +29.0°C  

acpitz-acpi-0
Adapter: ACPI interface
temp1:        +16.8°C  
temp2:        +16.8°C  

gigabyte_wmi-virtual-0
Adapter: Virtual device
temp1:        +25.0°C  
temp2:        +38.0°C  
temp3:        +29.0°C  
temp4:        +16.0°C  
temp5:        +31.0°C  
temp6:        +36.0°C  

nvme-pci-0900
Adapter: PCI adapter
Composite:    +31.9°C  (low  = -273.1°C, high = +89.8°C)
                       (crit = +94.8°C)
Sensor 1:     +31.9°C  (low  = -273.1°C, high = +65261.8°C)
Sensor 2:     +31.9°C  (low  = -273.1°C, high = +65261.8°C)
```

Para acompanhar em tempo real:
```bash
watch -n 2 sensors
```
Assim, o comando atualizará as temperaturas a cada 2 segundos — útil durante compilações ou testes de carga.

### Ativando o display do Water Cooler Rise Mode Aura Ice Black
Se você for um possuidor do Water Cooler Rise Mode Aura Ice Black, boas noticias, você pode ativar o display dele sem muito trabalho, basta seguir as instruções no link a seguir:  
[Ativando o display do modelo  Water Cooler Rise Mode Aura Ice Black](https://github.com/gladiston/water-cooler-hid)  


----

[Clique aqui para retornar a página principal](../README.md#monitorando-temperaturas-com-o-lm-sensors)
