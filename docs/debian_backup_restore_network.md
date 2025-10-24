# BACKUP/RESTORE DA CONFIGURAÇÃO ORIGINAL DE REDE
Vamos ser cautelosos e fazer um backup de nossa configuração de rede atual, assim se algo der errado, restauramos.
Veja como está sua configuração de rede atual, execute:
Depois observe o status, execute:
```bash
nmcli general status
```
E observe o resultado:  
```
STATE      CONNECTIVITY  WIFI-HW  WIFI        WWAN-HW  WWAN        METERED          
conectado  completa      missing  habilitado  missing  habilitado  não (adivinhado) 
```
Observe suas conexões atuais:  
```bash
$ nmcli con show
```
E observe os resultados:  
```
NAME                UUID                                  TYPE      DEVICE 
Wired connection 1  aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee  ethernet  enp8s0 
lo                  bbbbbbbb-cccc-dddd-eeee-ffffffffffff  loopback  lo  
```

Agora que viu sua configuração de rede original, escolha uma pasta para guardar o backup dessa configuração de rede, minha sugestão é **~/net-backups**, mas você pode escolher outra. Primeiro vamos apontar a variavel para a localização de seus backups:  
```bash
export BKP=~/net-backups
```

Depois criamos a pasta:
```bash
mkdir -p $BKP
```

Depois, vamos definir o nome do arquivo de backup com o seu nome:
```bash
export BKPFILE=$BKP/netcfg-$(date +%F_%H%M%S).tgz 
```

Depois, geramos um relatorio em formato .txt para posterior consulta e usamos o tar para compactar os arquivos de configurações atuais no local de destino:  
```bash
nmcli general status >$BKPFILE.txt
nmcli con show >>$BKPFILE.txt
ip -br a >>$BKPFILE.txt
sudo tar -C / -czf $BKPFILE   etc/network etc/NetworkManager etc/systemd/network
sudo chown -R $USER:$USER $BKP
```

Agora, verifique o arquivo que foi gerado:
```bash
$ ls -lh $BKP/
total 4,0K
-rw-r--r-- 1 gsantana gsantana 2,3K out 23 16:57 netcfg-2025-10-22_165719.tgz
```
Como pôde ver, são apenas alguns arquivos textos pequenos, e se o backup não foi gerado, observe novamente a sintaxe dos comandos que executou, mas não prossiga este tutorial sem um backup. Se tudo deu certo, então podemos prosseguir. 

### RESTAURAÇÃO BACKUP
Se mais tarde ocorrer alguma pane e precisar restaurar a configuração de rede, faça o seguinte, primeiro vamos apontar a variavel `BKP` para a localização de seus backups:  
```bash
export BKP=~/net-backups
```

Depois liste todos os seus backups e escoha o que for apropriado:  
```bash
$ ls -1 $BKP/
netcfg-2025-10-22_165719.tgz
```

Determinado qual o arquivo de backup desejado, execute:
```bash
sudo tar -C / -xzf $BKP/netcfg-2025-10-22_165719.tgz
```

Os arquivos de configuração serão substituidos pelo backup, porém o gerenciador que os controla precisa ser reiniciado:   

Vamos reiniciar, execute:  
```bash
sudo systemctl restart NetworkManager 
```

