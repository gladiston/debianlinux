# MINDER

O **Minder** é um excelente programa de código aberto dedicado à criação de **mapas mentais** (mind maps) no formato de aplicação desktop nativa. Ele oferece recursos robustos para estruturar ideias, notas e conceitos de forma hierárquica e visual, sendo muito similar a outras ferramentas de mapeamento mental.

Este tutorial apresenta duas formas de instalação: via Flatpak (recomendado para isolamento e uso no `$HOME`) e via repositório APT do Ubuntu (recomendado para integração mais profunda com o sistema).

### Opção 1: Instalação via Flatpak (Recomendada)

O Flatpak garante que o programa seja instalado de forma isolada, apenas para o seu usuário (`--user`), e facilita a obtenção da versão mais recente do software:

1.  **Instale o Minder:**

    ```bash
    flatpak install --user https://flathub.org/repo/appstream/com.github.phase1geo.minder.flatpakref
    ```

2.  **Atualize o programa (opcional, mas recomendado):**

    ```bash
    flatpak --user update com.github.phase1geo.minder
    ```

### Opção 2: Instalação via Repositório APT (Ubuntu e Derivados)

Se você utiliza Ubuntu ou uma distribuição derivada e prefere a integração tradicional do sistema de pacotes, use o APT:

1.  **Instale o Minder:**
    ```bash
    sudo apt install minder
    ```

O Minder estará agora instalado e deve aparecer no menu de seu ambiente gráfico.


----

[Clique aqui para retornar a página principal](../README.md#minder)
