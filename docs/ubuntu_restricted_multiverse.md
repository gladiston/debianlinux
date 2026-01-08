# Habilitando Repositórios `restricted` e `multiverse` (Apenas para Ubuntu e Derivados)

Diferentemente do Debian, as distribuições baseadas em Ubuntu gerenciam os componentes de software não-livre por meio de repositórios específicos: **`restricted`** e **`multiverse`**. Se você optou por uma instalação mínima ou desmarcou a opção de incluir *software* proprietário durante a configuração inicial, é essencial habilitar esses repositórios para obter suporte completo a multimídia e *hardware*.

Esses repositórios **estendem o repertório de pacotes e programas** disponíveis no sistema:

  * **`restricted`**: Contém *drivers* proprietários (não livres) que são fornecidos por fabricantes e são cruciais para o funcionamento de componentes de *hardware*, como placas de vídeo NVIDIA ou AMD.
  * **`multiverse`**: Contém *software* que não é livre e que não é *driver*, incluindo codecs multimídia, fontes proprietárias e alguns utilitários de terceiros.

### Procedimento Gráfico para Habilitação

Embora a edição manual do arquivo `/etc/apt/sources.list` seja possível, o método mais recomendado para usuários de desktop é via interface gráfica:

1.  Acesse o menu de configurações do seu ambiente de trabalho.
2.  Na maioria dos ambientes baseados em GTK, como GNOME, procure pelo aplicativo **Programas e Atualizações**, mas se utiliza o KDE, esta opção na seção de **Configurações do Sistema->Gerenciador de Drivers** e então selecione a guia **Software Ubuntu**:
    [Programas e Atualizações](../img/ubuntu_restricted_multiverse01.png)
    
4.  Dentro da seção apropriada, geralmente rotulada como **"Outro Software"** ou **"Software Ubuntu"**, localize e **habilite as caixas de seleção** correspondentes aos repositórios **`restricted`** e **`multiverse`**.
5.  O sistema solicitará a atualização dos seus índices de pacotes (o equivalente a rodar `apt update`). Confirme a atualização para aplicar as mudanças imediatamente.

Com os repositórios adicionais ativados, seu sistema estará pronto para instalar codecs e *players* de áudio e vídeo, além de *drivers* proprietários.

-----

## Atualização Completa do Sistema

Após a modificação e habilitação de novos repositórios, é crucial sincronizar os índices de pacotes locais e aplicar quaisquer atualizações pendentes para garantir a estabilidade e segurança do sistema.

### Comandos de Terminal

Execute os seguintes comandos no terminal:

1.  **Atualização dos Índices de Pacotes (`update`):**
    Sincroniza a lista de pacotes disponíveis com os repositórios recentemente habilitados.

    ```bash
    sudo apt update -y
    ```

2.  **Atualização dos Pacotes Instalados (`upgrade`):**
    Aplica todas as atualizações de pacotes instalados no sistema para suas versões mais recentes, resolvendo dependências automaticamente.

    ```bash
    sudo apt upgrade -y
    ```

## Análise do Resultado

Após a execução do `upgrade`, observe o sumário fornecido pelo APT. O resultado pode variar:

> **Exemplo de Saída (Sem Atualizações):**
> Resumo: Atualizando: 0, Instalando: 0, Removendo: 0, Não atualizando: 0

Se o *output* mostrar números diferentes de zero (por exemplo, `Atualizando: 5`), significa que o sistema encontrou atualizações de segurança ou correções de *bugs* que foram aplicadas.

----

[Clique aqui para retornar a página principal](../README.md#reposit%C3%B3tios-restricted-e-multiverse---apenas-para-ubuntu-e-derivados)
