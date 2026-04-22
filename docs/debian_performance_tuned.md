## INSTALANDO PERFIS DE PERFORMANCE (TUNED)
O **tuned** é um programa que permite trocar, em tempo real, o perfil de desempenho do computador por outro.  
Por exemplo, você pode usar o perfil **balanceado** enquanto navega na internet e, em seguida, alternar para o perfil **realtime** quando quiser maximizar a performance.  

Há outros perfis prontos para uso — voltados para **máquinas virtuais**, **economia de energia**, **servidores** e muito mais.   
O programa oferece uma grande variedade de perfis e é altamente recomendado para quem deseja ajustar o comportamento do sistema conforme a necessidade, vamos à instalação:
```
sudo apt install -y tuned
```
Liste os perfís de otimização existentes:
```
sudo tuned-adm list
```
Observe as opções listadas:  
```
Available profiles:
- accelerator-performance     - Throughput performance based tuning with disabled higher latency STOP states
- atomic-guest                - Optimize virtual guests based on the Atomic variant
- atomic-host                 - Optimize bare metal systems running the Atomic variant
- aws                         - Optimize for aws ec2 instances
- balanced                    - General non-specialized tuned profile
- balanced-battery            - Balanced profile biased towards power savings changes for battery
- cpu-partitioning            - Optimize for CPU partitioning
- cpu-partitioning-powersave  - Optimize for CPU partitioning with additional powersave
- default                     - Legacy default tuned profile
- desktop                     - Optimize for the desktop use-case
- desktop-powersave           - Optmize for the desktop use-case with power saving
- enterprise-storage          - Legacy profile for RHEL6, for RHEL7, please use throughput-performance profile
- hpc-compute                 - Optimize for HPC compute workloads
- intel-sst                   - Configure for Intel Speed Select Base Frequency
- laptop-ac-powersave         - Optimize for laptop with power savings
- laptop-battery-powersave    - Optimize laptop profile with more aggressive power saving
- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
- mssql                       - Optimize for Microsoft SQL Server
- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
- network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
- openshift                   - Optimize systems running OpenShift (parent profile)
- openshift-control-plane     - Optimize systems running OpenShift control plane
- openshift-node              - Optimize systems running OpenShift nodes
- optimize-serial-console     - Optimize for serial console use.
- oracle                      - Optimize for Oracle RDBMS
- postgresql                  - Optimize for PostgreSQL server
- powersave                   - Optimize for low power consumption
- realtime                    - Optimize for realtime workloads
- realtime-virtual-guest      - Optimize for realtime workloads running within a KVM guest
- realtime-virtual-host       - Optimize for KVM guests running realtime workloads
- sap-hana                    - Optimize for SAP HANA
- sap-hana-kvm-guest          - Optimize for running SAP HANA on KVM inside a virtual guest
- sap-netweaver               - Optimize for SAP NetWeaver
- server-powersave            - Optimize for server power savings
- spectrumscale-ece           - Optimized for Spectrum Scale Erasure Code Edition Servers
- spindown-disk               - Optimize for power saving by spinning-down rotational disks
- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
- virtual-guest               - Optimize for running inside a virtual guest
- virtual-host                - Optimize for running KVM guests
Current active profile: balanced
```
No exemplo acima, estou usando o perfil 'balanced', caso ele não apareça, use:
```
sudo tuned-adm active
```
Observe o resultado do comando:  
> Current active profile: balanced  

O modo 'balanceado' é o modo não especializado, se desejar especializar seu computador em algo, por exemplo, para usar o desktop, execute:  
```
sudo tuned-adm profile desktop  
```  
O sistema poderá ser configurado para diferentes finalidades, como por exemplo, otimizado para uso como desktop. É fundamental escolher o perfil correto, pois carregar um perfil inadequado compromete todo o desempenho.
```
sudo tuned-adm profile virtual-guest  
```
No exemplo acima, você estará aplicando um perfil voltado para máquinas virtuais então as VM serão bem mais rápidas, no entanto, caso nenhuma VM esteja sendo usada, isso resultará em uma queda perceptível de performance no ambiente gráfico — seja KDE, GNOME ou outro —, deixando tudo mais lento do que deveria.

Portanto, se optar por usar o tuned, é importante adotar o hábito de alternar o perfil conforme o seu uso atual.
Mais adiante, verifique se o seu ambiente gráfico (como KDE ou GNOME) oferece plugins de integração com o tuned, o que facilita bastante essa troca automática de perfis.

Perfis muito comuns para quem usa laptop:  

**laptop-ac-powersave** - “Optimize for laptop with power savings”  
*Uso*: notebook ligado na tomada (AC).  
*Objetivo*: manter bom desempenho, mas ainda economizar energia onde possível.   
*Ajustes típicos*: Habilita CPU frequency scaling (a CPU reduz clock quando ociosa). Mantém turbo boost ativado para tarefas pesadas. Reduz brilho de tela e consumo de periféricos em idle.   
Mantém discos e interfaces de rede em modo balanceado.     
*Resumo*: bom equilíbrio entre desempenho e economia.  Ideal para uso diário com o notebook conectado à energia.  

**laptop-battery-powersave** - “Optimize laptop profile with more aggressive power saving”
*Uso*: notebook usando bateria.  
*Objetivo*: maximizar autonomia, mesmo sacrificando desempenho.  
*Ajustes típicos*: CPU limitada a clocks mais baixos. Desativa turbo boost e núcleos ociosos. Reduz brilho e tempo de suspensão automática. Interfaces Wi-Fi e Bluetooth podem entrar em modos de economia agressiva. Discos mecânicos são parados rapidamente quando inativos.  
*Resumo*: desempenho menor, mas maior duração de bateria.  
Ideal para uso em viagens, reuniões ou campo.  

**latency-performance** - “Optimize for deterministic performance at the cost of increased power consumption”  
*Uso*: servidores ou estações de trabalho que exigem baixa latência e previsibilidade (mas não tempo real).  
*Objetivo*: garantir resposta consistente, mesmo com maior consumo de energia.  
*Ajustes típicos*: Desativa CPU frequency scaling → clock fixo máximo. Desativa C-states profundos e economias de energia. Ajusta IRQs e afinidade de CPU para reduzir jitter. Mantém memória e dispositivos em estado ativo constante.  
*Resumo*: máxima estabilidade e latência previsível, sem foco em economia.  
Ideal para bancos de dados, servidores de aplicações ou jogos que exigem resposta constante.  

Outros perfis muito uteis para desenvolvedores são:
**realtime** - “Optimize for realtime workloads”  
*Uso*: sistemas físicos (bare metal) com necessidades críticas de tempo real.  
*O que faz*: Reduz a latência ao máximo. Ajusta o escalonador de CPU para favorecer tarefas de tempo real. Desativa power saving features (como C-states e turbo boost). Fixa frequências da CPU em nível máximo. Ajusta IRQs e prioridade de processos.  
Exemplo de uso: estações de áudio profissional (JACK), robótica, processamento de sinais, sistemas de controle industrial.  

**realtime-virtual-guest** - “Optimize for realtime workloads running within a KVM guest”  
*Uso*: máquinas virtuais (guests) executando workloads de tempo real dentro de um host KVM.  
*O que faz*: Aplica otimizações similares ao perfil realtime, mas levando em conta que o controle de hardware é mediado pelo hipervisor. Ajusta parâmetros de temporização (clocksource, tickless, etc.) para sincronizar com o host. Minimiza interferência do kernel convidado.  
*Exemplo*: uma VM rodando um sistema de controle de robô industrial, ou processamento de áudio, dentro de um servidor KVM.  

**realtime-virtual-host** - “Optimize for KVM guests running realtime workloads”
*Uso*: no host KVM que executa VMs que, por sua vez, têm workloads de tempo real.
*O que faz*: Garante que as VMs de tempo real recebam CPU e I/O com mínima latência. Usa CPU pinning e isolcpus para isolar núcleos destinados às VMs RT. Minimiza a interferência do host em threads de tempo real. 
*Exemplo*: servidor KVM que hospeda várias VMs RT, como sistemas de automação ou simulações científicas críticas.  

Claro! Aqui está o **conteúdo em formato Markdown** que você pode **copiar e colar no seu arquivo `debian_performance_tuned.md`**, com o tópico novo **“Usando o tuned no ambiente gráfico”** e as informações que conversamos:

## Selecionando perfis do tuned no ambiente gráfico

Embora o *tuned* seja tradicionalmente usado via linha de comando (por exemplo, `sudo tuned-adm active` e `sudo tuned-adm profile <perfil>`), existe como tornar a troca de perfis bem mais amigável em um ambiente gráfico, especialmente em desktops como KDE Plasma:

### Tuned Switcher (Flatpak)

Uma opção prática é instalar o **Tuned Switcher**, um utilitário que adiciona um ícone na bandeja do sistema (system tray) permitindo:
- visualizar o perfil ativo;
- trocar perfis com cliques, sem precisar digitar comandos no terminal;
- suporte a notificações e modo de widget.

O Tuned Switcher está disponível no **Flathub** como Flatpak, se você ainda não instalou, siga as instruções em:  
[Instalando e habilitando o suporte a flatpak](debian_flatpak.md)  

Você pode instalá-lo através da loja de aplicativos do KDE ou GNOME, mas se quiser, também pode via terminal:   
```bash
flatpak install flathub org.easycoding.TunedSwitcher
```

Depois da instalação, é possível abrir o `Tuned Switcher` pelo menu de aplicativos e fixar o ícone na bandeja para facilitar o uso diário.  
Se vocÊ gostar, poderá acrescentá-lo no inicializador de sessão basta executar:  
```bash
mkdir -p ~/.config/autostart/
cp /var/lib/flatpak/exports/share/applications/org.easycoding.TunedSwitcher.desktop ~/.config/autostart/
```
Estes comandos irão copiar o atalho do programa TunedSwitcher para autoinicialização junto com a sessão.  


### Alternativa usando script com tecla de atalho
Se você preferir algo ainda mais integrado ao seu ambiente, pode criar um pequeno script que usa um seletor de menus (por exemplo, `zenity`, `rofi` etc.) para listar perfis disponíveis e permitir a escolha visual, como:
```bash
sudo touch /usr/local/bin/menu-tuned.sh
sudo chmod a+x /usr/local/bin/menu-tuned.sh
sudo touch /usr/share/applications/tuned-gui.desktop
sudo chmod a+x /usr/share/applications/tuned-gui.desktop
``
Estes arquivos foram criados vazios, vamos agora dar vida para eles, execute:
```bash
sudo editor /usr/local/bin/menu-tuned.sh
``
E depois cole este conteúdo
```bash
#!/bin/bash

# =================================================================
# Script: menu-tuned.sh
# Local: /usr/local/bin/menu-tuned.sh
# Descrição: Interface gráfica para troca de perfis do Tuned
# Requisitos: zenity, tuned, policykit-1
# =================================================================

# 1. Verifica se o Zenity está instalado
if ! command -v zenity &> /dev/null; then
    # Como o script pode ser rodado via menu, tentamos avisar via terminal ou log
    echo "Zenity não encontrado. Por favor, instale com: sudo apt install zenity"
    exit 1
fi

# 2. Captura a lista de perfis e descrições
# O awk remove o prefixo "- ", separa Nome e Descrição e os coloca em linhas
# individuais para que o Zenity preencha as colunas corretamente.
LISTA_PERFIS=$(tuned-adm list | awk -F ' - ' '/^- / {sub(/^- /, ""); print $1; print $2}')

# 3. Exibe a interface gráfica (Zenity)
# Redirecionamos o stderr para /dev/null para ignorar warnings de GTK 4.0
SELECAO=$(echo "$LISTA_PERFIS" | zenity --list \
    --title="Otimização de Sistema (Tuned)" \
    --window-icon="preferences-system-performance" \
    --width=750 --height=450 \
    --text="Selecione o perfil de performance desejado.\n(Para KVM, utilize perfis virtual-host ou virtual-guest)" \
    --column="Perfil" --column="Descrição" 2>/dev/null)

# 4. Processa a escolha do usuário
if [ -n "$SELECAO" ]; then
    # pkexec abre a caixa de diálogo nativa do GNOME/KDE para pedir senha de admin
    if pkexec tuned-adm profile "$SELECAO"; then
        zenity --info \
            --title="Sucesso" \
            --text="O perfil <b>$SELECAO</b> foi aplicado com sucesso!" \
            --timeout=3 2>/dev/null
    else
        # Caso o usuário cancele a senha ou ocorra erro
        zenity --error \
            --title="Falha" \
            --text="Não foi possível aplicar o perfil. Verifique a senha ou o status do Tuned." 2>/dev/null
    fi
else
    # Usuário fechou a janela ou clicou em Cancelar
    exit 0
fi
```
Salve e saia do editor.  
Agora você já pode executar o menu de perfil de performance, mas apenas no terminal, então vamos criar uma entrada para o menu de seu desktop, execute:
```bash
sudo editor /usr/share/applications/tuned-gui.desktop
```
E depois cole este conteúdo
```
[Desktop Entry]
Version=1.0
Type=Application
Name=Otimização Tuned
Comment=Gerenciar perfis de performance (KVM/Sistema)
Exec=/usr/local/bin/menu-tuned.sh
Icon=utilities-terminal
Terminal=false
Categories=System;Settings;
Keywords=performance;tuned;kvm;debian;
StartupNotify=true
```
Salve e saia do editor.
Agora você pode ir no menu do seu sistema e procurar por `tuned` e aparecerá o `Otimização Tuned` para selecionarmos nosso perfil de carga de trabalho, caso ele não apareça, dê uma forçada executando:
```bash
sudo update-desktop-database /usr/share/applications
```
Finalmente, agora você poderá ver o `Otimização Tuned` no menu do seu sistema.


----

[Clique aqui para retornar a página principal](../README.md#instalando-perfis-de-performance-tuned)
