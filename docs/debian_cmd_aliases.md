## AJUSTANDO ALIASES PARA COMANDOS REPETITIVOS
Aliases não são programas, e sim um recurso presente em praticamente todas as distribuições Linux que permite **abreviar ou simplificar comandos repetitivos**.  
Por exemplo, se você quiser listar os arquivos de uma pasta com cores e tamanhos legíveis (KB, MB, GB), teria que digitar algo assim toda vez:
```bash
ls -lh --color=auto
```

Isso é muito comprido. Que tal digitar apenas `l` e o sistema entender o comando acima?  
É exatamente isso que faremos agora. Edite o arquivo:
```bash
editor ~/.bashrc
```

O arquivo acima é executado automaticamente sempre que você abre um terminal **bash**.  
Acrescente ao final deste arquivo seus **aliases** personalizados, por exemplo:
```bash
alias l='ls -lh --color=auto'
```

Abaixo estão algumas sugestões úteis.  
Ao editar o arquivo `~/.bashrc`, você notará que várias delas já existem (algumas comentadas com `#`).
```
editor ~/.bashrc
```
E então acrescente estas linhas no final do arquivo:  
```
###
### Meus aliases
### 
# Navegação e listagem:
alias l='ls -lh --color=auto'        # Lista detalhada, tamanhos legíveis, com cores (comentada no Debian)
alias la='ls -lha --color=auto'      # Lista tudo, incluindo ocultos (comentada no Debian)
alias ll='ls -lh --color=auto'       # Lista longa, ignora ocultos (comentada no Debian)
alias ls='ls --color=auto'           # Força uso de cores sempre (comentada no Debian)
alias ..='cd ..'                     # Volta um diretório
alias ...='cd ../..'                 # Volta dois diretórios
alias ....='cd ../../..'             # Volta três diretórios
alias c='clear'                      # Limpa o terminal

# Sistema e administração
alias update='sudo apt update && sudo apt upgrade -y'   # Atualiza o sistema
alias install='sudo apt install -y '                    # Instala pacotes rapidamente
alias remove='sudo apt remove '                         # Remove pacotes
alias purge='sudo apt purge '                           # Remove pacotes + configs
alias cls='clear'                                       # Outra forma de limpar tela
alias df='df -h'                                        # Exibe uso de disco em formato legível
alias du='du -h -d 1'                                   # Mostra tamanho de pastas
alias free='free -h'                                    # Memória RAM legível

# Rede
alias ping='ping -c 5'             # Faz 5 pings e para
alias myip='curl ifconfig.me'      # Mostra seu IP público
alias ports='sudo netstat -tulanp' # Lista portas em uso

# Vida longa ao sysadmin
alias grep='grep --color=auto' # Cores no grep
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias h='history'              # Mostra histórico
alias j='jobs -l'              # Lista jobs atuais
alias v='vim'                  # Abre o Vim rapidamente
```

Muito útil ter seus aliases, não é mesmo?  
Agora **salve** o arquivo e feche o editor (`Ctrl+O`, `Enter`, `Ctrl+X`).  

Depois, **reinicie o terminal** e teste um dos aliases:
```bash
l
```

Saída esperada (exemplo):
```
total 0
drwxr-xr-x 1 gsantana gsantana 20 out 10 17:37 'Área de trabalho'
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:37  Documentos
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:41  Downloads
drwxr-xr-x 1 gsantana gsantana 32 out 10 18:48  Imagens
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:37  Modelos
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:37  Músicas
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:37  Público
drwxr-xr-x 1 gsantana gsantana  0 out 10 17:37  Vídeos
```

Pronto — agora você tem **comandos mais breves e práticos** para as tarefas do dia a dia.  
Reinicie a sessão para que todos os aliases entrem em ação permanentemente.

> **Curiosidade histórica**:  
> O uso de aliases e comandos curtos vem dos primeiros sistemas Unix, em que as conexões remotas eram muito lentas — cada caractere digitado economizava tempo e largura de banda. Essa cultura de abreviar comandos (como ls, cp, mv, rm) se manteve até hoje, por eficiência e praticidade.
