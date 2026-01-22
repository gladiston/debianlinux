# Travamento do diálogo “Abrir pasta” em aplicações Electron no KDE

## Introdução

Em ambientes KDE Plasma, especialmente em versões recentes como o Kubuntu 25.10, é relativamente comum que aplicações baseadas em Electron (como editores e IDEs) apresentem travamentos ao abrir o diálogo **Arquivo → Abrir pasta**.  
O sintoma típico é a janela de diálogo aparecer vazia, sem listar arquivos, acompanhada da mensagem “não está respondendo”.

Na prática, isso quase sempre está relacionado a **caminhos inexistentes ou indisponíveis** (por exemplo, `/mnt`, NFS, discos externos ou pastas removidas) que ficam salvos no histórico do sistema. O diálogo de arquivos tenta resolver esses locais automaticamente e acaba bloqueando a interface.

Este artigo descreve duas configurações no **Dolphin** que ajudam a evitar esse tipo de problema, especialmente para usuários avançados.

---

## Dolphin – Desligar “Recentes”

### Por que isso é indicado

A funcionalidade **Recentes** mantém referências a arquivos e pastas acessados anteriormente, inclusive locais de rede, discos removíveis e pontos de montagem temporários.  
Quando algum desses caminhos deixa de existir ou não responde, o diálogo de arquivos pode ficar bloqueado tentando acessá-lo.  
Programas feitos em electron como as IDEs vscode e cursor sofrem desse problema.  

Desligar essa funcionalidade é especialmente indicado para:
- usuários que usam mounts temporários (`/mnt/backup`, `/mnt/nfs-*`)
- ambientes com NFS ou shares que nem sempre estão ativos
- estações de trabalho técnicas, servidores ou máquinas de desenvolvimento
- uso de programas feitos em electron como vscode e cursor.
  

O ganho aqui é **estabilidade e previsibilidade**, em troca da perda de um recurso mais voltado a uso doméstico.

### Como configurar

1. Abra o **Dolphin**
2. Vá em **Configurações → Configurar o Dolphin**
3. Na seção **Geral**, desmarque:
   - Mostrar arquivos recentes
   - Mostrar locais recentes
4. Confirme e feche as configurações

Essa mudança afeta todos os diálogos de arquivos do sistema, não apenas o Dolphin.

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

Essas duas configurações simples eliminam uma das principais causas de travamentos em aplicações Electron no KDE e são altamente recomendadas para ambientes de desenvolvimento e uso técnico.
