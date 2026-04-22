# OPENTV

O **OpenTV** é um programa desenvolvido para assistir a canais de televisão através do protocolo **IPTV** (Televisão por Protocolo de Internet). Ele permite ao usuário adicionar e gerenciar suas próprias listas de canais (`.m3u` ou outros formatos compatíveis) para consumir conteúdo de TV e *streaming*. Para usar o programa, você precisará de uma lista de canais M3U, como a mantida pelo projeto IPTV-ORG, que agrega canais de acesso livre globalmente.

**É importante notar que o link de referência fornecido (`https://iptv-org.github.io/iptv/index.m3u`) não é pirataria.** Ele compila *streams* de **canais de TV abertos (Free-to-Air - FTA)** de todo o mundo, em diversos idiomas, e **não inclui conteúdo protegido por direitos autorais** que exija pagamento ou assinatura.

Este tutorial demonstra o método de instalação preferido para aplicativos Flatpak de desktop: a instalação na sua área de usuário.

Para realizar a **instalação apenas na sua área de usuário** (sem a necessidade de permissões de administrador), utilize o seguinte comando com a *flag* `--user`:

```bash
flatpak install --user flathub dev.fredol.open-tv
```

O OpenTV estará agora instalado e deve aparecer no menu de seu ambiente gráfico.

-----

### Instruções de Uso: Adicionar a Primeira Fonte

Ao abrir o OpenTV pela primeira vez, ele solicitará que você adicione uma fonte de canais. Baseado na tela de configuração, siga estes passos para adicionar uma lista de canais via URL:

1.  **Selecione o Método:** Clique na opção **`M3U URL`**.
2.  **Nome da Fonte (Source Name):** Digite um nome para identificar a sua lista (ex: `IPTV-ORG`).
3.  **URL da Lista:** No campo abaixo do nome, cole o link M3U que você deseja usar.
      * **Exemplo de URL de Referência:** `https://iptv-org.github.io/iptv/index.m3u`
4.  **Opções:** A caixa **`Use TVG-ID for names when TVG-NAME is unavailable`** deve ser marcada para garantir que a identificação dos canais seja feita corretamente, mesmo que falte o nome.
5.  **Finalizar:** Clique no botão **`Fetch`** (Baixar) para carregar os canais.

Após este processo, seus canais estarão prontos para serem assistidos na interface principal do OpenTV.

