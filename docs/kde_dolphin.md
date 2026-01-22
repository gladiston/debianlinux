# KDE - Ajustes básicos no Dolphin

## Introdução
Este artigo descreve algumas configurações no gerenciador de arquivos **Dolphin** que ajudam a evitar alguns problemas conhecidos, e também mostrar como fazer ajustes para usuários avançados.

---

## Dolphin – Desligar “Recentes”

### Por que isso é indicado
Em ambientes KDE Plasma, especialmente em versões recentes como o Kubuntu 25.10 e Plasma 6.x, é relativamente comum que aplicações baseadas em Electron (como editores e IDEs) apresentem travamentos ao abrir o diálogo **Arquivo → Abrir pasta**.  
O sintoma típico é a janela de diálogo aparecer vazia, sem listar arquivos, acompanhada da mensagem “não está respondendo”.

Na prática, isso quase sempre está relacionado a **caminhos inexistentes ou indisponíveis** (por exemplo, `/mnt`, NFS, discos externos ou pastas removidas) que ficam salvos no histórico do sistema. O diálogo de arquivos tenta resolver esses locais automaticamente e acaba bloqueando a interface.

A funcionalidade **Recentes** mantém referências a arquivos e pastas acessados anteriormente, inclusive locais de rede, discos removíveis e pontos de montagem temporários.  
Quando algum desses caminhos deixa de existir ou não responde, o diálogo de arquivos pode ficar bloqueado tentando acessá-lo.  
Programas feitos em Electron como as IDEs VS Code e Cursor sofrem com esse problema.

Desligar essa funcionalidade é especialmente indicado para:
- usuários que usam mounts temporários (`/mnt/backup`, `/mnt/nfs-*`)
- ambientes com NFS ou shares que nem sempre estão ativos
- estações de trabalho técnicas, servidores ou máquinas de desenvolvimento
- uso de programas feitos em Electron como VS Code e Cursor

O ganho aqui é **estabilidade e previsibilidade**, em troca da perda de um recurso mais voltado a uso doméstico.

### Como configurar (KDE Plasma 6.4.5)

A partir do KDE Plasma 6, o controle de **Recentes** deixou de ser feito no Dolphin e passou a ser uma configuração **global do sistema**.

1. Abra **Configurações do Sistema**
2. Vá em **Área de Trabalho → Pesquisa**
3. Acesse a seção **Histórico**
4. Desative:
   - **Lembrar arquivos e locais recentes**
   - **Mostrar arquivos recentes**
5. Aplique as alterações
6. Feche e reabra o Dolphin ou faça logout/login

Essa mudança afeta todos os diálogos de arquivos do sistema, não apenas o Dolphin, incluindo aplicações baseadas em Electron.

---

## Dolphin – Lista detalhes dos arquivos

### Por que isso é indicado para usuários avançados
Ao configurar o Dolphin para usar a visualização **Lista detalhada**, o usuário passa a ter maior controle e previsibilidade sobre a navegação de diretórios.  
Essa visualização reduz dependências de metadados extras (como miniaturas, prévias e consultas adicionais), tornando o carregamento de pastas mais simples e menos sujeito a bloqueios quando existem caminhos lentos ou problemáticos.

Para quem trabalha com:
- `/mnt`, NFS, CIFS ou discos removíveis
- grandes árvores de diretórios
- pastas que podem não estar sempre disponíveis  

essa configuração diminui a chance de o diálogo de arquivos ficar aguardando respostas de locais que não existem mais.

### Como configurar

1. Abra o **Dolphin**  
2. Vá em **Menu → Exibir → Modo de exibição → Lista detalhada**  
3. (Opcional, mas recomendado) Ajuste as colunas visíveis para exibir apenas o necessário, como:
   - Nome
   - Tamanho
   - Data de modificação  

Essa configuração passa a ser reutilizada automaticamente nos diálogos de abrir/salvar arquivos do sistema.

---

## Conclusão
Travamentos no diálogo “Abrir pasta” raramente são defeitos da aplicação em si. Na maioria dos casos, o problema está no histórico de caminhos que o sistema tenta resolver automaticamente, incluindo pastas que não existem mais ou que estão temporariamente indisponíveis.

Ao:
- usar a visualização em **lista detalhada**
- desativar o uso de **Recentes**

o ambiente KDE passa a se comportar de forma mais robusta e previsível, especialmente em cenários avançados com mounts, rede e armazenamento externo.

Essas duas configurações simples eliminam uma das principais causas de travamentos em aplicações Electron no KDE e são altamente recomendadas para ambientes de desenvolvimento e uso técnico. Os demais ajustes citados são cosméticos e você decidiu fazer ou não cada um deles.

----

[Clique aqui para retornar a página principal](../README.md#kde---ajustes-básicos-no-dolphin)
