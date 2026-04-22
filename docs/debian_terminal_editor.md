# EDITOR DE TEXTO PADRÃO PARA O TERMINAL
Por padrão, Debian e Ubuntu (e muitas distros derivadas) definem o **nano** como editor de texto padrão do terminal. Embora o **nano** seja simples e intuitivo, muitos administradores e desenvolvedores preferem editores mais avançados, como o **Vim**, **Neovim** ou **Micro**, que oferecem recursos adicionais — realce de sintaxe, atalhos poderosos e suporte a múltiplos modos de edição. Sempre que o sistema precisar abrir um editor — por exemplo, ao executar comandos como `crontab -e`, `visudo` ou `git commit` — ele usará o editor definido na variável de ambiente **EDITOR** (ou **VISUAL**).  

### Verificando o editor padrão atual
Para descobrir qual editor o sistema usa por padrão, execute:
```bash
echo $EDITOR
```

Se não houver saída, significa que nenhum editor está definido explicitamente e normalmente é usado o **nano**.

### EDITOR DE TEXTO PADRÃO PARA O TERMINAL - Alterando o editor padrão
Você pode trocar o editor padrão de forma temporária (válida apenas para a sessão atual do terminal) ou de forma permanente.

Para definir o editor padrão de forma permanente para todo o sistema, use:
```bash
sudo update-alternatives --config editor
```
Você verá uma lista semelhante a:
```
Há 3 escolhas como alternativas de editor. (fornecendo /usr/bin/editor).

  Seleção    Caminho             Prioridade Estado
------------------------------------------------------------
* 0            /bin/nano            40        modo automático
  1            /bin/nano            40        modo manual
  2            /usr/bin/vim.basic   30        modo manual
  3            /usr/bin/vim.tiny    15        modo manual

Pressione <enter> para manter o padrão[*] ou digite o número da seleção:
```
Digite o número correspondente ao editor que deseja usar e pressione `Enter`.  
Por exemplo, para definir o **Vim** como editor padrão, escolha o número referente a `/usr/bin/vim.basic` (modo texto com recursos usuais, colorização, macros) ou `/usr/bin/vim.tiny`(mínimo possível, usado como “emergency editor”).

**Dica:** se você é iniciante e prefere um ambiente mais amigável, mantenha o **Nano**.  
Mas se já se sente à vontade no terminal e quer produtividade, o **Vim** é uma excelente escolha.

----

[Clique aqui para retornar a página principal](../README.md#editor-de-texto-padr%C3%A3o-para-o-terminal)
