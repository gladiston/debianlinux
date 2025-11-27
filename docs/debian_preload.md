# PRELOAD (OPCIONAL)
Se estiver usando **discos mecânicos**, provavelmente sente muita **latência** ao carregar certos programas.  
Numa situação assim, é bom instalar um serviço chamado **preload**: ele monitora os programas que você mais utiliza e, durante o boot, já os carrega antecipadamente.  

A vantagem é a **velocidade** para abrir esses programas pela primeira vez.  
Por outro lado, há uma desvantagem: esses programas **sempre estarão na memória** logo após o boot, reduzindo a quantidade de RAM disponível imediatamente após a inicialização.  

Portanto, a recomendação de uso do **preload** é:
1. Somente se você utiliza **discos mecânicos**, e  
2. Se tem um **fluxo de trabalho consistente e repetitivo**, com o mesmo grupo de programas.  

Se, após avaliar, decidir que o *preload* é útil para o seu caso, instale-o com:

```bash
sudo apt install -y preload
```

Quando instalado, ele é ativado automaticamente como serviço.  
Mesmo assim, é bom conferir seu status:

```bash
sudo systemctl status preload
```

Exemplo de saída:
```
● preload.service - LSB: Adaptive readahead daemon
     Loaded: loaded (/etc/init.d/preload; generated)
     Active: active (running) since Thu 2025-10-16 16:15:49 -03; 44s ago
 Invocation: 43dd115e9acd4d12bc9dea6af1aa2f6a
       Docs: man:systemd-sysv-generator(8)
    Process: 3634 ExecStart=/etc/init.d/preload start (code=exited, status=0/SUCCESS)
      Tasks: 1 (limit: 37542)
     Memory: 3.5M (peak: 3.5M)
        CPU: 114ms
     CGroup: /system.slice/preload.service
             └─3639 /usr/sbin/preload -s /var/lib/preload/preload.state
```

Como podemos observar no exemplo acima, o serviço está **active**, indicando que está em execução.  
Caso seu sistema indique o contrário, inicie-o manualmente com:

```bash
sudo systemctl start preload
```

E para que ele seja iniciado automaticamente durante o boot, execute:

```bash
sudo systemctl enable preload
```

Se achar que o **preload** não trouxe vantagens perceptíveis, você pode removê-lo pela interface gráfica do KDE ou GNOME, ou via terminal:

```bash
sudo apt remove -y --purge preload
```

----

[Clique aqui para retornar a página principal](../README.md#preload-opcional)
