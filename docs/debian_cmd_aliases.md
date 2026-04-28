## AJUSTANDO ALIASES PARA COMANDOS REPETITIVOS

Aliases não são programas, e sim um recurso presente em praticamente todas as distribuições Linux que permite **abreviar ou simplificar comandos repetitivos**.  
Por exemplo, se você quiser listar os arquivos de uma pasta com cores e tamanhos legíveis (KB, MB, GB), teria que digitar algo assim toda vez:

```bash
ls -lh --color=auto
```

Isso é muito comprido. Que tal digitar apenas `l` e o sistema entender o comando acima?  
É exatamente isso que faremos agora.

### Aliases globais (para todos os usuários)

A forma mais simples e “padrão de Debian” para tornar aliases globais é criar um arquivo em `/etc/profile.d/`.  
Tudo que você colocar ali será carregado por shells “de login” (e também por muitas sessões gráficas que inicializam o ambiente via `profile`).

```bash
sudo editor /etc/profile.d/aliases.sh
```

Cole o conteúdo abaixo (pode ajustar à vontade):

```text
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
alias ping='ping -c 5'             # Faz 5 pings ao inves de indefinido
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
Agora **salve** o arquivo e feche o editor.

Depois, **abra um novo terminal** (recomendado) ou recarregue a sessão atual:

```bash
source /etc/profile
```

E faça o teste com um dos aliases:
```bash
c
```

E ele limpará a tela, porque `c` é um alias para `clear`.

Pronto — agora você tem **comandos mais breves e práticos** para as tarefas do dia a dia.  
Reinicie a sessão para que todos os aliases entrem em ação permanentemente.

### Observações importantes

- **Bash não-login (muito comum)**: em alguns fluxos, o Bash pode não carregar `/etc/profile`. Se você notar que os aliases globais não entram em vigor em terminais “normais”, é bom adicionar esta linha ao final de `/etc/bash.bashrc` que em algumas distros como Debian e Ubuntu já vem comentadas e neste caso, basta descomentá-las:

```bash
if [ -d /etc/profile.d ]; then
  for _i in /etc/profile.d/*.sh; do
    if [ -r $_i ]; then
      . $_i
    fi
  done
  unset _i
fi
```

- **Zsh**: se você usa Zsh, verifique se a sua sessão carrega `/etc/profile` (normalmente carrega). Se não carregar, você pode fazer o mesmo ajuste no arquivo global apropriado (ex.: `/etc/zsh/zshrc`) para “sourcear” o `/etc/profile`.

> **Curiosidade histórica**:  
> O uso de aliases e comandos curtos vem dos primeiros sistemas Unix, em que as conexões remotas eram muito lentas — cada caractere digitado economizava tempo e largura de banda. Essa cultura de abreviar comandos (como ls, cp, mv, rm) se manteve até hoje, por eficiência e praticidade.

----

[Clique aqui para retornar a página principal](../README.md#ajustando-aliases-para-comandos-repetitivos)
