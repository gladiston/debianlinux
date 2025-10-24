# ACESSAR PARTIÇÕES NTFS NO SISTEMA

Se você utiliza uma ou mais partições NTFS(Windows) que não são automaticamente montadas, pode usar o gerenciador de arquivos do KDE ou GNOME para acessá-las.  
No entanto, toda vez que fizer isso provavelmente será solicitada uma senha — e isso cansa até o desenvolvedor mais paciente.
Contudo, você pode ajustar o fstab para melhorar o acesso tanto pela interface gráfica como também pelo **terminal**, é mais prático criar uma pasta de montagem fixa — assim, ao entrar nela, já verá o conteúdo da partição NTFS.  
Se gostou da ideia, vamos implementá-la.  

Primeiro, identifique corretamente qual é o seu disco ou partição NTFS.  
Execute no terminal:
```bash
lsblk -f | grep ntfs
```
E verá algo semelhante a:
```
NAME        FSTYPE FSVER LABEL   UUID                                 MOUNTPOINTS
nvme1n1p2   ntfs   3.1   Windows 389083EB9083AE46                    
nvme1n1p3   ntfs   3.1   DADOS   1EB4CCF2B4CCCE09                    
```

No exemplo acima, o disco desejado tem o **LABEL=DADOS** e o **UUID=1EB4CCF2B4CCCE09**.  
Agora, vamos criar uma pasta vazia (ponto de montagem), de preferência com o nome da label do disco, em `$HOME`, `/media` ou `/mnt`, por exemplo:  
`/home/usuario/label_dados` ou `/media/label_dados`.  

A vantagem de usar `$HOME` é que, com as permissões adequadas, apenas você terá acesso a ela.  
Mas, se quiser disponibilizar a partição para todos os usuários, use:
```bash
sudo mkdir -p /media/label_dados
sudo chmod 2777 /media/label_dados
sudo chmod +s /bin/ntfs-3g
sudo usermod -aG disk $USER
```

Assim, a pasta `/media/label_dados` ficará acessível a todos os usuários.  
Se optar por criá-la dentro de `$HOME`, não há necessidade dos comandos acima — apenas crie a pasta vazia.

Em seguida, edite o arquivo de montagem automática:
```bash
sudo editor /etc/fstab
```

Adicione ao final do arquivo a seguinte linha:
```
# Minha partição NTFS de label “DADOS”
UUID=1EB4CCF2B4CCCE09  /media/label_dados  ntfs-3g  windows_names,nosuid,nodev,nofail,rw,user,gid=users,noauto  0  0
```

| Parâmetro | Explicação |
|:--|:--|
| users | Permite que usuários normais montem e desmontem a partição, não apenas o root. |
| rw | Monta a unidade com leitura e escrita habilitadas. |
| nosuid | Impede a execução de arquivos com bit suid, aumentando a segurança. |
| nodev | Impede a criação de arquivos de dispositivo. |
| file_mode=0777 | Define permissões de arquivos dentro da partição (leitura, gravação e execução para todos). |
| dir_mode=0777 | Define permissões de diretórios dentro da partição. |
| auto | Monta automaticamente durante o boot. |
| noauto | Evita montagem automática no boot. |
| Os dois “0” finais | Desativam *dump* e *fsck* automáticos. |

Outra forma válida de escrever a linha seria:
```
UUID=1EB4CCF2B4CCCE09  /media/label_dados  ntfs  nls-utf8,rw,nosuid,nodev,nofail  0  0
```

A diferença é que, ao usar o driver **ntfs-3g**, você estará utilizando um **driver em espaço de usuário** (precisa do pacote `ntfs-3g` instalado), considerado mais moderno e seguro.  
Já o driver **ntfs** é integrado ao kernel e geralmente recomendado apenas para **modo leitura**, pois seu suporte a gravação é limitado.  

Ambos funcionam, mas o **ntfs-3g** é preferido, pois oferece recursos adicionais — como evitar a criação de nomes de arquivos incompatíveis com o Windows (por exemplo, contendo “:” ou “?”).

### Atenção ao usar LABEL em montagens NTFS
Embora o uso de **LABELs** pareça mais legível e intuitivo que o uso de **UUIDs**, em partições **NTFS** isso pode gerar sérios inconvenientes.  
As etiquetas (LABELs) de volumes NTFS podem **se repetir facilmente** ou, pior, **serem perdidas** após uma manutenção de disco ou atualização do Windows.  

Quando isso acontece, o sistema Linux pode **demorar muito durante o boot** tentando montar a partição configurada no `/etc/fstab`, especialmente em distribuições que exibem tela de inicialização (*splash screen*).  
Durante esse tempo, o usuário pode **achar que o sistema travou**, quando na verdade o Linux está apenas tentando identificar o volume.  

Por esse motivo, **use sempre UUIDs** em vez de LABELs ao montar partições NTFS no `/etc/fstab`.  
Os UUIDs são únicos e não se alteram, garantindo que o sistema encontre a partição correta de forma imediata e confiável.

Salve e feche o arquivo (`Ctrl+O`, `Enter`, `Ctrl+X`).    
**Reinicie o sistema**.  
Após o boot, a pasta indicada servirá como ponto de acesso à partição NTFS.

Se não quiser que a partição seja montada automaticamente no boot, mas apenas sob demanda, altere `auto` para `noauto` e monte manualmente:
```bash
mount /media/label_dados
```

O uso de `noauto` é mais seguro, pois impede que programas ou scripts maliciosos acessem a partição sem sua intervenção.

Se precisar **reparar a unidade NTFS**, use:
```bash
sudo ntfsfix /dev/disk/by-uuid/1EB4CCF2B4CCCE09
```
Ou, se preferir pelo label:
```bash
sudo ntfsfix /dev/disk/by-label/DADOS
```


### Alternativa moderna: AutoFS e systemd.automount
O **AutoFS** continua disponível e funcional nas principais distribuições Linux.  
Ele é um serviço que monta automaticamente pastas, partições ou compartilhamentos de rede **apenas quando são acessados**, o que pode ser útil para dispositivos externos ou unidades de rede.

Apesar de ainda ser amplamente usado, o **systemd** oferece uma alternativa mais moderna e integrada: o **systemd.automount**, que desempenha o mesmo papel de forma nativa, com menos dependências e melhor integração com o processo de inicialização.

Em geral, o uso de `fstab` é mais simples para montagens locais permanentes, enquanto **AutoFS** e **systemd.automount** são indicados para:
- Montagens sob demanda (discos externos, pendrives, unidades de rede);
- Ambientes corporativos com múltiplos compartilhamentos;
- Situações em que economizar tempo de boot é importante.

Se quiser estudar mais sobre o AutoFS:
[https://wiki.archlinux.org/title/Autofs](https://wiki.archlinux.org/title/Autofs)

E sobre systemd.automount:
[https://www.freedesktop.org/software/systemd/man/latest/systemd.mount.html](https://www.freedesktop.org/software/systemd/man/latest/systemd.mount.html)


