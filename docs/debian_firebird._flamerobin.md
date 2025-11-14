# Guia Completo: Como Instalar o FlameRobin (Versão Recente) Compilando no Linux (Fedora e Debian/Ubuntu)

### 1\. O que é o FlameRobin?

O FlameRobin é um administrador de banco de dados e ferramenta de desenvolvimento open-source, disponível para Mac, Windows e Linux. Com ele, podemos administrar e desenvolver bancos de dados Firebird.

Embora possua uma interface que pode parecer incomum à primeira vista (com janelas soltas), seu fluxo de trabalho é focado em SQL: ele possui diversos assistentes visuais que geram o *statement* SQL base, que você pode então modificar e executar.

Neste artigo, vamos focar em como instalar a versão mais recente do FlameRobin em distribuições Linux, compilando-a diretamente do código-fonte.

-----

### 2\. Por que Compilar? O Problema dos Repositórios

Você provavelmente encontrará o FlameRobin na loja de aplicativos da sua distribuição (Ubuntu Software, Fedora Software, etc.) ou instalando diretamente pelo terminal (ex: `sudo apt install flamerobin`).

**No entanto, este método quase sempre instalará uma versão muito antiga e defasada.**

Para ter melhor compatibilidade, correções de bugs e aproveitar os recursos mais recentes, o método que recomendo é **compilar a versão mais recente** diretamente do código-fonte no GitHub. Abaixo, mostro como preparar o ambiente para isso em distribuições baseadas em Debian e Fedora.

-----

### 3\. Preparando o Ambiente (Pré-compilação)

O segredo para uma compilação fácil é usar o gerenciador de pacotes do sistema para instalar todas as *dependências de compilação* (bibliotecas como wxWidgets, etc.) que o pacote antigo do repositório exigia. Isso quase sempre satisfaz as necessidades do código-fonte mais novo.

#### A. No Fedora (e derivados)

No Fedora, podemos usar o `dnf` para investigar o pacote e depois usar o `dnf builddep` para instalar as dependências.

**1. Investigar o Pacote (Opcional)**

Primeiro, pesquise pelo nome e verifique os detalhes para confirmar a versão:

```bash
dnf search flamerobin
dnf info flamerobin
```

> **Exemplo de Saída:**
> Nome: flamerobin
> Versão: 0.9.9
> Lançamento: 6.fc42
> ...
> Sumário: Graphical client for Firebird

Observe a versão (ex: `0.9.9`). Agora, visite o site oficial de releases no GitHub (`https://github.com/mariuz/flamerobin/releases`) e compare. Se a versão do repositório for antiga, siga para a compilação.

**2. Instalar Dependências**

No Fedora, o comando `dnf builddep` é fornecido pelo pacote `dnf-plugins-core`:

```bash
sudo dnf install dnf-plugins-core
```

Se houver um flamerobin instalado previamente, é melhor desinstalá-lo:

```bash
sudo dnf remove flamerobin
```

Em seguida, puxe todas as dependências de compilação que o pacote do repositório exige:

```bash
sudo dnf builddep flamerobin
```

Por garantia, instale as ferramentas básicas de compilação (caso ainda não estejam no sistema):

```bash
sudo dnf install git cmake gcc-c++
```

#### B. No Debian/Ubuntu (e derivados)

No Debian, Ubuntu e sistemas derivados, o processo é similar com o `apt-get build-dep`.

**1. Habilitando os Repositórios de Código-Fonte (deb-src)**

O comando `build-dep` exige que os repositórios de código-fonte (`deb-src`) estejam habilitados. Verifique seu arquivo `/etc/apt/sources.list` (ou os arquivos dentro de `/etc/apt/sources.list.d/`).

Garanta que as linhas `deb-src` correspondentes às suas fontes `deb` principais estejam ativadas (sem um `#` no início).

*Por exemplo:*

```
deb http://archive.ubuntu.com/ubuntu/ jammy main
deb-src http://archive.ubuntu.com/ubuntu/ jammy main
```

Após editar e salvar o arquivo, atualize sua lista de pacotes:

```bash
sudo apt update
```

**2. Instalando Dependências**

Se houver um flamerobin instalado previamente, desinstale-o:

```bash
sudo apt remove -y flamerobin
```

Execute o comando para baixar todas as dependências de compilação:

```bash
sudo apt-get build-dep flamerobin
```

Por garantia, instale as ferramentas básicas que usaremos para baixar e gerenciar a compilação:

```bash
sudo apt install git cmake build-essential
```

-----

### 4\. Baixando e Compilando o FlameRobin

Agora que o ambiente está pronto em ambos os sistemas, os passos de compilação são os mesmos. Recomendo fazer isso em sua pasta de usuário (`$HOME`), pois é mais seguro e não exige privilégios de administrador até o passo final.

#### Passo 1: Baixar o Código-Fonte (Git)

Ao clonar o repositório padrão (default branch), já pegamos o "nightly-build" mais recente.

```bash
# Vá para sua pasta de usuário
cd ~

# Clone o repositório
git clone https://github.com/mariuz/flamerobin.git

# Entre na pasta que foi criada
cd flamerobin
```

#### Passo 2: Configurar a Compilação (CMake)

Criaremos um diretório `build` separado para não misturar os arquivos de compilação com o código-fonte.

```bash
# Crie e entre no diretório de compilação
mkdir build
cd build
```

Agora, configure o projeto no modo **Release** (recomendado), que gera um binário otimizado:

```bash
cmake -DCMAKE_BUILD_TYPE=Release ..
```

#### Passo 3: Compilar (Make)

Com tudo configurado, execute o `make`. Isso pode demorar alguns minutos.

```bash
make
```

-----

### 5\. Executando e Instalando (Pós-compilação)

Neste ponto, o programa está compilado\! Você tem duas opções:

#### Opção A: Rodar Localmente (Sem Instalar)

Se você quer apenas testar ou rodar o programa sem "instalá-lo" oficialmente no sistema, pode usar o script de execução fornecido:

```bash
# (Estando no diretório 'build', volte um nível)
cd ..

# Execute o script
./run_flamerobin.sh
```

#### Opção B: Instalar no Sistema (Recomendado)

Se você quiser que o FlameRobin seja instalado permanentemente (em `/usr/local/bin`) e apareça no menu de aplicativos, use o `make install`. Este é o único passo que exige `sudo`.

```bash
# (Certifique-se de estar no diretório 'build')
# cd ~/flamerobin/build

# Execute a instalação
sudo make install
```

-----

### 6\. Confgurações do FlameRobin
As configurações do FlameRobin são armazenadas na seguinte pasta: 
```
~/.flamerobin
```
Então caso precise aproveitar configurações advindas de outros computadores, basta copiá-las para a pasta acima.

-----

### 7\. Criando uma Entrada de Menu (Atalho)

Se o `sudo make install` não tiver criado um atalho de menu, ou se você escolheu a "Opção A" (Rodar Localmente), você pode criar um atalho manual.

Abra um editor de texto para criar o arquivo `.desktop`:

```bash
editor ~/.local/share/applications/flamerobin.desktop
```

Cole o seguinte conteúdo dentro do editor:

```ini
[Desktop Entry]
Version=1.0
Type=Application
Terminal=false
Name=FlameRobin
Exec=flamerobin
Comment=Gerenciador de banco de dados FirebirdSQL
Icon=flamerobin
Categories=GNOME;GTK;Develop;
```

**⚠️ Atenção à linha `Exec=`:**

1.  **Se você usou `sudo make install` (Opção B):** O valor `Exec=flamerobin` está **correto**.
2.  **Se você usou a "Opção A" (Rodar Localmente):** O valor `Exec=flamerobin` **não vai funcionar**. Você deve usar o caminho completo para o script:
    `Exec=/home/SEU_USUARIO/flamerobin/run_flamerobin.sh`
    *(Lembre-se de trocar `SEU_USUARIO` pelo seu nome de usuário\!)*

Salve e feche o arquivo. A partir de agora, você encontrará o ícone do FlameRobin no menu de aplicativos do seu sistema.

-----

### 8\. Minhas Considerações Pessoais sobre o FlameRobin

Agora que está instalado, quero compartilhar minha análise pessoal sobre a ferramenta, baseada na minha experiência de uso (incluindo as versões recentes de desenvolvimento):

O fluxo de trabalho dele é focado em SQL. Iniciantes preguiçosos vão odiá-lo, mas pessoas mais pacientes em aprender os comandos ou os mais experientes vão gostar desse método. Especialmente porque ele não te prende a uma versão particular do Firebird; funciona com qualquer versão em que o *statement* é reconhecido.

No entanto, notei vários pontos de atenção:

  * **DDL não confiável:** Gera um DDL que nem sempre é confiável. Um exemplo claro são as *triggers*.
  * **Campos Calculados:** Tabelas com campos calculados "somem" e a estrutura DDL gerada não é limpa. Ao invés de fazer o `CREATE` e depois o `ALTER` para adicionar PK/FK e campos calculados, ele tenta fazer tudo no `CREATE TABLE`.
  * **Extração de Dados:** Ao extrair dados para script (`INSERT`/`UPDATE`), ele inclui os campos calculados, o que, evidentemente, falhará na execução.
  * **Domínios:** Não entende domínios do Firebird. Se você gerar DDLs, ele os tratará como tipos normais (ex: `INTEGER` ao invés de `T_MEU_DOMINIO_INT`).
  * **Scripts:** Não roda scripts SQL complexos.
  * **Debug:** Não faz debug de PSQL (procedures e triggers).
  * **Wizards Antigos:** Embora funcione perfeitamente com as versões mais recentes do Firebird, os *wizards* (assistentes) mostram claramente que foram idealizados para oferecer apenas tipos de dados usados até o Firebird 3.

 
