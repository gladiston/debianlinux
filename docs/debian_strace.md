# INSTALANDO O STRACE
O **strace** mostra as chamadas de sistema (útil para ver onde o erro ocorre) e também detectar recursos que estão sendo usados por outros programas. Instale:  
```bash
sudo apt install -y strace
```

### Usando o `strace` para descobrir por que a unidade não desmonta
Um exemplo comum é tentar desmontar uma unidade e receber algo como:  
```
umount: /media/rede: target is busy.
```
Isso significa que **algum processo ainda está usando arquivos ou subpastas dentro desse ponto de montagem**.  
Podemos usar o `strace` para descobrir quem é o culpado, execute:

```bash
sudo strace -f -e trace=open,openat,stat,statx,access,close,read,write umount /media/rede 2>&1 | less
```

Observe as últimas linhas da saída: elas normalmente mostram o arquivo, diretório ou descritor que está sendo acessado quando ocorre o erro `EBUSY` (Device or resource busy).

Um exemplo de saída típica:
```
umount("/media/rede")          = -1 EBUSY (Device or resource busy)
write(2, "umount: /media/rede: target is busy.\n", 39) = 39
```

Isso indica que algo ainda está usando `/media/rede`.

Agora, para descobrir **quem** está mantendo o ponto ocupado, combine o `strace` com o comando `lsof`:
```bash
sudo lsof +D /media/rede
```
ou
```bash
sudo fuser -mv /media/rede
```

Esses comandos listarão os processos e programas que ainda têm arquivos abertos na unidade.  
Por exemplo:
```
COMMAND   PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
bash     1085 user  cwd    DIR  0,43      4096    2 /media/rede
nautilus 1202 user  15u   DIR  0,43      4096 4097 /media/rede/projetos
```

Neste caso, **o terminal atual** (bash) e o **gerenciador de arquivos (nautilus)** ainda estão dentro de `/media/rede`.  
Basta fechar as janelas ou mudar de diretório:
```bash
cd ~
```
e tentar novamente:
```bash
sudo umount /media/rede
```
> **Dica:**
> Se quiser confirmar em tempo real **quem tenta acessar a pasta enquanto desmonta**, use:
>**sudo strace -p $(pidof umount)**
> ou rastreie o uso direto da pasta:
>**sudo strace -f -p $(pidof bash) -e trace=open,openat 2>&1 | grep "/media/rede"**
Esses comandos mostram quais processos ainda estão “segurando” o ponto de montagem.

----

[Clique aqui para retornar a página principal](../README.md#instalando-o-strace)
