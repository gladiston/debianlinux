# **Instalar Drivers NVIDIA no Debian 13 (Trixie) usando o repositório oficial da NVIDIA**

A seguir você encontrará um guia seguro e atualizado para instalar os drivers proprietários da NVIDIA no Debian 13 usando **o repositório oficial da NVIDIA** em vez dos repositórios Debian.
Esse método fornece drivers mais recentes, compatíveis com Wayland e com suporte pleno ao DKMS.

---

# **1. Identificar qual GPU NVIDIA você possui**

### **Por que este passo é importante?**

Saber qual GPU você tem ajuda a confirmar se ela é suportada pelos drivers atuais.
Placas antigas (Legacy) usam drivers diferentes das placas modernas.
Esse passo evita instalar um driver incompatível que pode causar tela preta.

### **Comando:**

```bash
lspci | grep -i nvidia
```
Será mostrado algo como:
> 01:00.0 VGA compatible controller: NVIDIA Corporation TU117M [GeForce GTX 1650 Mobile / Max-Q] (rev a1)
> 01:00.1 Audio device: NVIDIA Corporation Device 10fa (rev a1)

Isso indicará que há uma placa NVIDIA no sistema.  

---

# **2. Habilitar contrib e non-free-firmware**

### **Por que isso é necessário?**

Os drivers proprietários da NVIDIA dependem de componentes que não fazem parte do repositório *main* do Debian.
Sem **contrib**, **non-free** e **non-free-firmware**, o sistema não encontrará bibliotecas essenciais para o funcionamento correto do driver.

### **Como fazer:**

Edite:

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
Geralmente a parte `non-free non-free-firmware` não está habilitado, se for este o caso, inclua-os.  


---

# **3. Adicionar o repositório oficial da NVIDIA**

### **Por que usar o repositório da NVIDIA?**

O Debian 13 inclui drivers da NVIDIA, mas eles costumam atrasar algumas versões em relação ao lançamento oficial da NVIDIA.
O repositório oficial fornece:

* versões mais recentes e otimizadas
* melhor suporte a Wayland
* correções de bugs recentes
* atualizações via APT (com DKMS automaticamente recompilado)

Esse método evita usar o instalador `.run` — que **não deve ser usado no Debian**.

---

## **3.1 Instalar dependências necessárias**

```bash
sudo apt install curl wget gnupg apt-transport-https dkms
```

---

## **3.2 Adicionar a chave GPG da NVIDIA**

### **Por que isso é importante?**

O APT só instala pacotes de repositórios cuja assinatura digital é válida.
Sem a chave GPG correta, o Debian irá recusar os pacotes do repositório NVIDIA.

```bash
curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/3bf863cc.pub | \
  sudo gpg --dearmor -o /usr/share/keyrings/nvidia.gpg
```

---

## **3.3 Registrar o repositório NVIDIA**

Crie:

```bash
sudo editor /etc/apt/sources.list.d/nvidia.list
```

Conteúdo:

```
deb [signed-by=/usr/share/keyrings/nvidia.gpg] https://developer.download.nvidia.com/compute/cuda/repos/debian13/x86_64/ /
```

Atualize índices:

```bash
sudo apt update
```

---

# **4. Instalar o driver NVIDIA**

### **Por que este passo funciona no Debian 13?**

No Debian 13 (com Wayland), o driver da NVIDIA já é compatível nativamente.
Não há necessidade de editar xorg.conf (inclusive o Debian 13 nem usa Xorg por padrão).
O driver instalará automaticamente:

* módulo do kernel (via DKMS)
* suporte a aceleradores
* componentes Vulkan
* integração com Wayland (GBM + EGL Streams dependendo da versão)

### **Instalação recomendada:**

```bash
sudo apt install nvidia-driver
```

Ou, se quiser uma versão mais nova:

```bash
sudo apt install nvidia-driver-560
```

*(A numeração pode mudar conforme o repositório liberar novas versões.)*

---

# **5. Desabilitar o driver Nouveau**

### **Por que isso é obrigatório?**

O **Nouveau** é o driver open source para placas NVIDIA.
Ele carrega automaticamente no boot e, se estiver ativo, **tem precedência sobre o driver proprietário**, impedindo que o módulo oficial da NVIDIA seja carregado.
Resultado: conflito, tela preta ou fallback para renderização sem aceleração.

Bloqueá-lo garante que o kernel use exclusivamente o módulo `nvidia.ko`.

### **Como bloquear o Nouveau:**

```bash
echo 'blacklist nouveau' | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
echo 'options nouveau modeset=0' | sudo tee -a /etc/modprobe.d/blacklist-nouveau.conf
sudo update-initramfs -u
```

---

# **6. Reiniciar**

Reinicie para aplicar o novo kernel + módulos:

```bash
sudo reboot
```

---

# **7. Confirmar que a instalação funcionou**

### **Por que testar?**

Para garantir que:

* o módulo da NVIDIA carregou
* o Wayland está usando a aceleração da NVIDIA
* o DKMS recompilou corretamente

### **Teste principal:**

```bash
nvidia-smi
```

Se aparecer a tabela com informações da GPU, tudo está funcionando.

### **Verificar se Wayland está ativo**

```bash
echo $XDG_SESSION_TYPE
```

Saída esperada:

```
wayland
```

---

# **8. Ajustes opcionais**

### **Ferramenta de controle**

```bash
sudo apt install nvidia-settings
```

### **Vulkan**

```bash
sudo apt install nvidia-vulkan-icd
```

### **Modo híbrido (Intel + NVIDIA — notebooks)**

```bash
sudo apt install nvidia-prime
sudo prime-select nvidia
```

---

# **9. Como reverter caso dê problema**

### **Por que isso pode ser necessário?**

Se você instalou uma versão muito nova que apresenta bugs, pode voltar para o Nouveau ou para uma versão do Debian.

Para remover completamente:

```bash
sudo apt purge nvidia-driver nvidia-*
sudo rm /etc/modprobe.d/blacklist-nouveau.conf
sudo update-initramfs -u
sudo reboot
```

---

# **10. Conclusão**

Este é o método mais seguro e moderno para instalar NVIDIA no Debian 13:

* total compatibilidade com Wayland
* drivers sempre atualizados
* sem instaladores problemáticos
* suporte a DKMS
* sem configurações manuais de Xorg
