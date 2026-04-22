# KDE – Ajustes básicos no Dolphin

## Introdução
Este artigo descreve configurações do gerenciador de arquivos **Dolphin** que contribuem para evitar problemas recorrentes no ambiente KDE Plasma, além de apresentar ajustes recomendados para usuários avançados que necessitam de maior estabilidade e previsibilidade no uso do sistema.

---

## Dolphin – Desligar “Recentes”

### Por que isso é indicado
Em ambientes KDE Plasma, especialmente em versões recentes como o Kubuntu 25.10 e Plasma 6.x, é relativamente comum que aplicações baseadas em Electron (como editores e IDEs) apresentem travamentos ao abrir o diálogo **Arquivo → Abrir pasta**.  
O sintoma mais frequente é a exibição de uma janela de diálogo vazia, sem listagem de arquivos, acompanhada da indicação de que a aplicação “não está respondendo”.

Na prática, esse comportamento está quase sempre relacionado a **caminhos inexistentes ou indisponíveis** (por exemplo, `/mnt`, NFS, discos externos ou diretórios removidos) que permanecem registrados no histórico do sistema. O diálogo de arquivos tenta resolver automaticamente esses locais, o que pode resultar no bloqueio da interface.

A funcionalidade **Recentes** mantém referências a arquivos e pastas acessados anteriormente, incluindo locais de rede, dispositivos removíveis e pontos de montagem temporários.  
Quando algum desses caminhos deixa de existir ou não responde, o diálogo de arquivos pode permanecer aguardando indefinidamente, ocasionando o travamento observado.  
Aplicações desenvolvidas com Electron, como as IDEs VS Code e Cursor, são particularmente sensíveis a esse comportamento.

A desativação dessa funcionalidade é especialmente recomendada para:
- usuários que utilizam mounts temporários (`/mnt/backup`, `/mnt/nfs-*`);
- ambientes com NFS ou compartilhamentos que nem sempre estão disponíveis;
- estações de trabalho técnicas, servidores ou máquinas de desenvolvimento;
- uso frequente de aplicações baseadas em Electron, como VS Code e Cursor.

O principal benefício dessa configuração é o aumento da **estabilidade e previsibilidade** do sistema, em detrimento de um recurso voltado principalmente ao uso doméstico.

### Como configurar (KDE Plasma 6.4.5)

A partir do KDE Plasma 6, o controle da funcionalidade **Recentes** deixou de ser realizado no Dolphin e passou a ser uma configuração **global do sistema**.

1. Abra **Configurações do Sistema**
2. Acesse **Espaço (Área) de Trabalho → Pesquisa do Plasma**
3. No painel direito, utilize a barra de pesquisa e procure por **recentes**
4. Desative:
   - **Arquivos recentes**
5. Aplique as alterações
6. Em seguida, vá em **Sistema → Sessão → Sessão da área de trabalho**
7. Em **Restauração da sessão**, caso esteja selecionada a opção *Na última saída*, altere para **Iniciar com uma sessão vazia**
8. Aplique as alterações
9. Abra o **Dolphin**
10. Na barra superior, no canto direito, acesse o menu em forma de *hamburger*, depois **Mais → Configurar → Configurar Dolphin**.  
    Na seção **Interface**, em **Mostrar ao iniciar**, altere a opção de *Pastas, abas e estado das janelas da última vez* para a pasta utilizada com mais frequência, normalmente o diretório `$HOME` (`/home/fulano`), e confirme em **OK**
11. Encerre a sessão do usuário (logout) e efetue novo login

Essas alterações afetam todos os diálogos de arquivos do sistema, não apenas o Dolphin, incluindo aplicações baseadas em Electron.

---

## Dolphin – Lista detalhada de arquivos

### Por que isso é indicado para usuários avançados
Ao configurar o Dolphin para utilizar a visualização **Lista detalhada**, o usuário obtém maior controle e previsibilidade na navegação entre diretórios.  
Esse modo reduz a dependência de metadados adicionais (como miniaturas, pré-visualizações e consultas extras), tornando o carregamento das pastas mais simples e menos suscetível a bloqueios quando existem caminhos lentos ou problemáticos.

Para usuários que trabalham com:
- `/mnt`, NFS, CIFS ou dispositivos removíveis;
- grandes hierarquias de diretórios;
- pastas que nem sempre estão disponíveis;

essa configuração diminui significativamente a possibilidade de o diálogo de arquivos permanecer bloqueado aguardando respostas de locais inexistentes.

### Como configurar

1. Abra o **Dolphin**
2. Na barra superior, no canto direito, acesse o menu em forma de *hamburger* e selecione **Mais → Exibir → Ajustar o estilo de exibição**.  
   Na opção **Modo de Exibição**, selecione **Detalhes**. Caso deseje, ajuste também a ordenação dos arquivos
3. Em **Informações adicionais**, marque as colunas que deseja exibir na listagem de arquivos, tais como:
   - Tamanho
   - Modificado
   - Tipo
   - Permissões (no final da lista)
   - Proprietário (no final da lista)
   - Grupo do usuário (no final da lista)
4. Clique em **OK**
5. Essa alteração será aplicada apenas à pasta atual. Para torná-la o padrão, retorne ao menu em forma de *hamburger*, selecione **Mais → Configurar → Configurar Dolphin** e, na seção **Exibir**, marque as opções:
   - Usar estilo de exibição comum para todas as pastas
   - Abrir arquivos compactados como pastas
   - Abrir pastas durante as operações de arrastar  
   Em seguida, clique em **Aplicar** e depois em **OK**

Essa configuração passa a ser utilizada automaticamente também nos diálogos de abrir e salvar arquivos do sistema.

---

## Conclusão
Travamentos no diálogo **Abrir pasta** raramente decorrem de falhas na aplicação em si. Na maioria dos casos, o problema está associado ao histórico de caminhos que o sistema tenta resolver automaticamente, incluindo diretórios inexistentes ou temporariamente indisponíveis.

Ao:
- utilizar a visualização em **lista detalhada**;
- desativar a funcionalidade **Recentes**;

o ambiente KDE passa a apresentar um comportamento mais robusto e previsível, especialmente em cenários avançados que envolvem mounts, rede e armazenamento externo.

Essas configurações simples eliminam uma das principais causas de travamentos em aplicações Electron no KDE e são altamente recomendadas para ambientes de desenvolvimento e uso técnico. Os demais ajustes mencionados são de caráter cosmético e podem ser adotados conforme a necessidade de cada usuário.

----

[Clique aqui para retornar à página principal](../README.md#kde---ajustes-básicos-no-dolphin)
