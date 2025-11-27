# MONITORANDO O SISTEMA COM O HTOP
O **htop** é um monitor interativo de processos para Linux, uma versão aprimorada e muito mais amigável do clássico comando `top`.  
Ele exibe em tempo real o uso da CPU, memória, swap, tarefas em execução, processos e suas prioridades, tudo com cores e atalhos intuitivos.  
Ao contrário do `top`, o **htop** permite **navegar com o teclado** pelas listas, **matar processos** com um clique ou tecla e **organizar colunas** conforme sua preferência.

### Instalação
Se ainda não estiver instalado, execute:
```bash
sudo apt install -y htop
```

### Execução
Para abrir o monitor, digite:
```bash
htop
```

Você verá uma tela colorida semelhante a esta:

```
  1  [|||||||||                      20.5%]   Tasks: 123, 1 running
  2  [||||||||||                     24.3%]   Load average: 0.36 0.42 0.41
  Mem[|||||||||||||||||         3.2G/15.5G]   Uptime: 2 days, 3:27
  Swp[|                            0K/4.0G]

  PID USER      PRI  NI  VIRT   RES   SHR S  CPU% MEM%   TIME+  Command
 1587 gsantana   20   0 2316M  176M  110M S   8.5  1.1  0:01.94  firefox
 1733 gsantana   20   0 1584M  214M  140M S   3.2  1.3  0:00.88  code
 1245 root       20   0  302M   40M   25M S   1.2  0.3  1:24.76  Xorg
```

As barras superiores mostram:
- **CPU(s)**: cada núcleo é monitorado individualmente; as cores indicam o tipo de carga (usuário, sistema, I/O etc.).  
- **Memória e Swap**: quanto está sendo usado, quanto está livre e o total disponível.  
- **Load Average**: média de carga do sistema (quanto mais próxima de 1.0 por núcleo, mais equilibrado está).  
- **Uptime**: tempo total desde o último boot.

Abaixo aparecem os processos, com colunas como:
- **PID** — número do processo;  
- **USER** — dono do processo;  
- **%CPU / %MEM** — consumo atual de CPU e memória;  
- **TIME+** — tempo total de CPU usado;  
- **Command** — comando ou programa em execução.

### Atalhos úteis no htop
| Tecla | Função |
|:--|:--|
| **F3** | Buscar por um processo específico (nome ou PID). |
| **F4** | Filtrar por nome. |
| **F5** | Alternar exibição por árvore (mostra relação entre processos). |
| **F6** | Escolher qual coluna usar para ordenar. |
| **F9** | Encerrar (kill) um processo. |
| **F10** | Sair do programa. |

Você também pode usar as **setas do teclado** ou o **mouse** para navegar entre os processos e menus.

### Exemplo prático
Vamos supor que o sistema esteja lento e você queira descobrir **qual programa está consumindo mais CPU**.  
Abra o `htop`:
```bash
htop
```
Pressione **F6** e escolha **%CPU** como coluna de ordenação.  
O processo que aparecer no topo da lista é o que mais consome CPU naquele momento.

Se for um programa que travou e precisa ser encerrado, selecione-o com as setas e pressione `F9` e o `htop` perguntará qual sinal enviar.  
O mais comum é **SIGTERM (15)** para encerrar normalmente, ou **SIGKILL (9)** para forçar o encerramento.

### Executando o htop como administrador
Alguns processos do sistema só são visíveis com privilégios de root.  
Para vê-los, execute:
```bash
sudo htop
```

Assim, você terá uma visão completa de todos os serviços, inclusive os de kernel, rede e drivers.

>Dica extra: Quer monitorar apenas os processos de um usuário específico?
>**htop -u gsantana**
>Ou salvar o uso de CPU e memória num arquivo de log (a cada 2 segundos):
>**htop -d 20 -C > monitor.log**

Com o **htop**, você consegue identificar gargalos de CPU, processos zumbis e programas travados em segundos.  
É uma ferramenta essencial para quem administra sistemas Linux ou desenvolve software e quer entender o comportamento real do sistema.

----

[Clique aqui para retornar a página principal](../README.md#monitorando-o-sistema-com-o-htop)
