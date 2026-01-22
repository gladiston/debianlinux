# INSTALANDO E MELHORANDO O GIMP (Flatpak)

O **GIMP** (GNU Image Manipulation Program) é um **editor de imagens** de código aberto, gratuito e poderoso. Ele é a principal alternativa ao Adobe Photoshop no ambiente Linux e outras plataformas.

## 1\. Instalação Base do GIMP

Para instalar a versão base do GIMP através do Flatpak, execute o seguinte comando no seu terminal, garantindo a instalação na sua área de usuário (`--user`):

```bash
flatpak install org.gimp.GIMP
```

-----

## 2\. Plugins Essenciais via Flatpak (Extensões)

Muitos plugins importantes foram empacotados como extensões para a versão Flatpak do GIMP. Você pode instalá-los separadamente com o comando `flatpak install --user` e o respectivo ID da extensão:

### **Resynthesizer** (Remoção Inteligente de Objetos)

O Resynthesizer é famoso por sua funcionalidade de "Content-Aware Fill" (Preenchimento Sensível ao Conteúdo), permitindo remover objetos, manchas ou imperfeições de forma inteligente, preenchendo a área selecionada com base na textura ao redor.

  * **Comando de Instalação:**

    ```bash
    flatpak install org.gimp.GIMP.Plugin.Resynthesizer
    ```

  * **Vídeo Tutorial (Exemplo da Função):**
    [GIMP Resynthesizer Remove Objects](https://www.google.com/search?q=https://www.youtube.com/watch%3Fv%3DFjI1J4pTq08)

-----

### **G'MIC** (A Maior Coleção de Filtros)

O G'MIC (GREYC's Magic for Image Computing) é uma biblioteca de filtros e efeitos de imagem avançados, muitas vezes usados para **substituir a funcionalidade de plugins menos comuns**. Ele inclui ferramentas para manipulação de imagem, arte e retoque.

  * **Comando de Instalação:**

    ```bash
    flatpak install org.gimp.GIMP.Plugin.GMic
    ```

  * **Vídeo Tutorial (Exemplo da Função):**
    [How to Install G'MIC on GIMP 2.10 and Use It](https://www.google.com/search?q=https://www.youtube.com/watch%3Fv%3Dk521P6kM75o)

-----

## Plugins Suportados (Informação Complementar)

Os plugins originais que você mencionou têm as seguintes equivalências ou status dentro do ambiente Flatpak mais atual:

| Plugin Original | Função | Status / Equivalência no Flatpak |
| :--- | :--- | :--- |
| **BIMP** | Operações em *batch* (lote). | Geralmente, requer instalação externa ou uso da biblioteca **G'MIC** para automação de algumas tarefas. |
| **FocusBlur** | Efeito de Profundidade. | **Incluído** na maioria das versões GIMP 2.10.x. Não é necessário instalar à parte. |
| **LiquidRescale** | Redimensionamento consciente de conteúdo. | Geralmente, requer instalação externa. |
| **Gimp Lens Fans** | Correção de lentes. | Muitos desses filtros são agora integrados no **GIMP base** ou disponíveis via **G'MIC**. |
| **Gimp Transformação Fourier** | Remoção de padrões (*Moire*). | Esta é uma funcionalidade que pode ser replicada com filtros avançados no **G'MIC**. |


----

[Clique aqui para retornar a página principal](../README.md#instalando-o-gimp)
