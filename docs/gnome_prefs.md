# Guia de Otimização: GNOME Produtivo para Desenvolvedores

## Introdução

O GNOME é conhecido por sua interface limpa e minimalista, mas para o fluxo de trabalho de desenvolvimento, essa "limpeza" pode resultar em cliques excessivos e perda de agilidade. Este artigo detalha como implementar quatro pilares fundamentais de usabilidade: **gestão de área de transferência**, **otimização de espaço de tela**, **mosaico de janelas inteligente** e **transparência no sistema de arquivos**.

Ao utilizar essas ferramentas no Debian 13 ou derivados, você reduz a carga cognitiva e mantém o foco no código, aproveitando a estabilidade da base Debian com a modernidade das extensões atuais.

---

## Gerenciador de extensões
O GNOME em ambientes como o Debian é tão Vanilla que nem mesmo o gerenciador de extensões o acompanha, então vamos instalá-lo:  
```bash
sudo apt install gnome-shell-extension-manager
```

### Instalando algumas extensões  
Abra o Extension Manager, clique na pequena "lupa" e busque pelos nomes a seguir e clique em **Instalar..** :  
   * **Dash to Panel**: Um painel para minimizar/restaurar programas em uso e outras funcionalidades. Se usa o Ubuntu, não é muito diferente daquela barra lateral;     
   * **Clipboard history**: Para ter um histórico da clipboard;  
   * **Lock Keys**: apenas se voce usa teclado sem fio ou algum modelo de teclado que não tem indicador de caps lock, num lock e scroll lock;  
   * **Tile assistent**: para usar a comodidate de ajuste automático de janelas.
   * **User Themes**: Libera o uso de temas personalizados.
   * **Force Quit**: Quem trabalha com programação sabe o quanto é ruim quando um programa trava, então essa extensão permite abortá-la quando não está mais respondendo.  
A seguir, alguns ajustes finos que executo em cada uma dessas extensões...  

### Dash to Panel
Consolida status do sistema, relógio e aplicativos abertos em uma única barra (estilo clássico, porém moderno).  
Se usa o Ubuntu, ele não precisa ser instalado, afinal a barra lateral do Ubuntu é praticamente o mesmo dash.  
Algo que costuma me dar produtividade é ir em configurações|Dash To Panel e então na guia **Position**:  
* **Panel monitor position**: a maioria prefere manter a dock no lado inferior, mas em monitores wide screen, colocá-lo no lado esquerdo(ou direito) aproveita melhor o espaço.
* **Panel length**: o padrão é 100%, mas se você usar `Dynamic` ele se ajustará sozinho conforme a necessidade.
Na guia **Style**:  
* **Border radius**: Se quiser cantos arrendodados troque de `0` para `4`px.  
* **Animate hovering app icons**: Deixe `habilitado` para destacar melhor os ícones quando o ponteiro do mouse passar por cima deles.   

Existem muitos outros ajustes, mas estes eu considero essencais.  

### Clipboard history
O desenvolvedor médio copia e cola centenas de trechos de código, comandos e URLs diariamente. Um histórico de clipboard não é um luxo, é uma necessidade.
Um applet no painel aparecerá e ali aparecerá todos os seus `ctrl+c` (copiar) para que possa acessar o histórico e recuperar qualquer um deles. Recomendo que use a tecla de atalho dele, `Ctrl+Super+V`. Voce até pode trocar por outro atalho, mas não deve usar o `Super+V` que é o padrão do Windows porque essa combinação já está em uso pelo painel superior do GNOME.

### Lock Keys
Alguns teclados sem fio e de notebook podem não ter os leds indicadores de `Caps Lock`,`Num Lock` e `Scroll Lock`  então nessa situação, esta extesão ela é importante porque mostra estado dessas teclas diretamente no painel.
Se seu teclado já mostra isso, essa extensão é desnecessária. Porém, teclados sem fio e teclados de notebook podem não ter esses leds.  


### Gerenciamento de Janelas (Tiling Assistant)

O GNOME 47+ melhorou o suporte a janelas, mas o **Tiling Assistant** eleva isso ao nível de um Tiling Window Manager (como i3 ou Sway) sem perder a facilidade do GNOME.  
Se você usa o Ubuntu, não precisa instalá-lo porque o Ubuntu teu outra extensão para **Tilling** que é bem similar.  

**Principais Recursos:**

* **Snap Groups:** Ao redimensionar uma janela, a vizinha se ajusta automaticamente.
* **Layouts Favoritos:** Salve layouts específicos (ex: IDE à esquerda, Terminal e Navegador à direita).

---

## Otimização do Arquivos (Nautilus)

O gerenciador de arquivos padrão esconde metadados cruciais para quem gerencia permissões de servidores ou backups de bancos de dados.

**Como habilitar as colunas detalhadas:**

1. Abra o **Nautilus** (Arquivos).
2. Clique no ícone de "Hambúrguer" (menu) no canto superior direito.
3. Vá em **Preferências**.
4. Procure pela seção **Colunas Visíveis**.
5. Marque as opções:
* **Data de Modificação** (Essencial para saber qual log é o mais recente).
* **Tamanho** (Para identificar rapidamente dumps de DB pesados).
* **Proprietário** (Crucial para debugar problemas de permissão em volumes KVM/Docker).
* **Grupo** (Opcional, mas útil em ambientes multi-usuário).



---

## Otimização da área de trabalho

Uma Dock embaixo(Dash to Panel) ou do lado (Ubuntu Dock) pode ocupar um espaço razoável se você ainda tem no topo o painel do GNOME mostrando canto de tarefas, data/hora e demais opções.  
Esse painel no topo em algumas versões do Debian foi removido ficando apenas a Dock embaixo, pessoalmente eu gostei muito.  
Mas olha a situação do Ubuntu, a Dock fica ao lado e no topo o painel do GNOME e ainda por cima da largura do monitor apenas para mostrar hora e alguns botões estreitos! Se você achar isso um desperdicio de espaço, vamos mudar isso.
Execute o gerenciador de extensões, vá em mnavegar e procure e instale as extensções:  
* **App Icons TaskBar**:  Ela traz os seus favoritos e taskbar para o painel superior.   
* **AppsMenu**: Ele traz um menu para voce encontrar as aplicações de que precisa sem precisar usar a tecla Super.  
* **Move date menu to the right**: Como o próprio nome sugere, muda a posição de data/hora para o lado direito.  

### Ubuntu
Depois, no caso do Ubuntu, você deve ir na guia de **Instalados** e desativar as extensões:  
* **Ubuntu AppIndicators**:  Ela traz os seus favoritos e taskbar para o painel superior.   
* **Ubuntu Dock**: Ele traz um menu para voce encontrar as aplicações de que precisa sem precisar usar a tecla Super.  

### Debian
Caso voce tenha o painel do GNOME lá em cima, a extensão **Dash to Panel** perde sua necessidade porque a extensão **App Icons TaskBar** fará a mesma coisa no painel do GNOME, então sugiro desativá-la. Mas se quiser mantê-la por causa da aparência, ótimo, também tá valendo.  

---

## O que deve evitar
Evite instalar extensões que monitoram seu sistema, é legal você ter algo assim para monitorar tempo de bateria, mas muitas dessas extensões de monitoramente que pegam dados de CPU, Memória, Disco e rede a cada segundo é desperdiçar os recursos do sistema, afinal, o tempo de de monitoramento também roubará ciclos de processamento de seu computador. O meio termo de uso ideal de uma extensão assim seria deixá-lo desabilitado até que pretenda realmente observar um comportamento anômolo.  
Mas, sendo sincero, se pretende monitorar algo, use o **Gerenciador de Tarefas** do GNOME, ele é personalizável e pode lhe entregar muito mais informaõesd do que essas extensões mostrariam.  


---
## Conclusão

A filosofia do Debian 13 é oferecer uma base sólida. Ao adicionar essas camadas de personalização, você não quebra a estabilidade do sistema, mas remove os atritos do GNOME "seco". O resultado é um ambiente onde o histórico de cópia é acessível, as janelas se organizam sozinhas e as informações de sistema estão sempre visíveis, permitindo que você gaste mais tempo codando e menos tempo gerenciando sua interface.
