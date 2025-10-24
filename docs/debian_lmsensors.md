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
Responda **yes** a todas as perguntas e, ao final, carregue os módulos sugeridos, por exemplo:


### Uso prático
Depois disso, basta:

```
$ sensors
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
