---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/00-intro.md
source_revision: ceb88f194775cb689a3792a9d856f311b47febea
translation_status: ready
---

# Introdução

O Composer é uma ferramenta para gerenciamento de dependências em PHP.
Ele permite que você declare as bibliotecas das quais seu projeto depende e ele
as gerenciará (instalará/atualizará) para você.

## Gerenciamento de dependências

O Composer **não** é um gerenciador de pacotes no mesmo sentido que o Yum ou o
Apt.
Sim, ele lida com "pacotes" ou bibliotecas, mas os gerencia separadamente
por projeto, instalando-os em um diretório (por exemplo, `vendor`) dentro do seu
projeto.
Por padrão, ele não instala nada globalmente.
Portanto, ele é um gerenciador de dependências.
No entanto, ele oferece suporte a um projeto "global" por conveniência através
do comando [global](03-cli.md#global).

Essa ideia não é nova e o Composer é fortemente inspirado pelo
[npm](https://www.npmjs.com/) do Node.js e pelo [bundler](https://bundler.io/)
do Ruby.

Suponha que:

1. Você tem um projeto que depende de várias bibliotecas.
2. Algumas dessas bibliotecas dependem de outras bibliotecas.

O Composer:

1. Permite que você declare as bibliotecas das quais depende.
2. Descobre quais versões de quais pacotes podem e precisam ser instaladas e as
   instala (ou seja, as baixa para o seu projeto).
5. Você pode atualizar todas as suas dependências com um único comando.

Consulte o capítulo [Uso básico](01-basic-usage.md) para obter mais detalhes
sobre como declarar dependências.

## Requisitos do sistema

A versão mais recente do Composer requer o PHP 7.2.5 para funcionar.
Uma versão com suporte de longo prazo (2.2.x) ainda oferece suporte ao PHP
5.3.2+, caso você esteja usando uma versão antiga do PHP.
Algumas configurações sensíveis do PHP e flags de compilação também são
necessárias, mas ao usar o instalador, você saberá de quaisquer
incompatibilidades.

O Composer precisa de várias aplicações auxiliares para funcionar de forma
eficaz, tornando o processo de gerenciamento de dependências de pacotes mais
eficiente.
Para descompactar arquivos, o Composer usa ferramentas como `7z` (ou `7zz`),
`gzip`, `tar`, `unrar`, `unzip` e `xz`.
Quanto aos sistemas de controle de versão, o Composer se integra perfeitamente
com Fossil, Git, Mercurial, Perforce e Subversion, garantindo assim o bom
funcionamento da aplicação e o gerenciamento dos repositórios de bibliotecas.
Antes de usar o Composer, certifique-se de que essas dependências estejam
instaladas corretamente no seu sistema.

O Composer é multiplataforma e nos esforçamos para que ele funcione igualmente
bem no Windows, Linux e macOS.

## Instalação - Linux / Unix / macOS

### Baixando o executável do Composer

O Composer oferece um instalador prático que você pode executar diretamente da
linha de comando.
Sinta-se à vontade para [baixar este arquivo](https://getcomposer.org/installer)
ou revisá-lo no
[GitHub](https://github.com/composer/getcomposer.org/blob/main/web/installer) se
quiser saber mais sobre o funcionamento interno do instalador.
O código-fonte é PHP puro.

Existem, resumidamente, duas formas de instalar o Composer.
Localmente, como parte do seu projeto, ou globalmente, como um executável
disponível em todo o sistema.

#### Localmente

Para instalar o Composer localmente, execute o instalador no diretório do seu
projeto.
Consulte [a página de Download](https://getcomposer.org/download/) para obter
instruções.

O instalador verificará algumas configurações do PHP e, em seguida, baixará o
arquivo `composer.phar` para o seu diretório de trabalho.
Este arquivo é o binário do Composer.
É um arquivo PHAR (PHP archive), um formato de arquivo para PHP que pode ser
executado na linha de comando, entre outras coisas.

Agora execute `php composer.phar` para executar o Composer.

Você pode instalar o Composer em um diretório específico usando a opção
`--install-dir` e também renomeá-lo usando a opção `--filename`.
Ao executar o instalador seguindo
[as instruções da página de download](https://getcomposer.org/download/),
adicione os seguintes parâmetros:

```shell
php composer-setup.php --install-dir=bin --filename=composer
```

Agora execute `php bin/composer` para executar o Composer.

#### Globalmente

Você pode colocar o arquivo PHAR do Composer em qualquer lugar que desejar.
Se você o colocar em um diretório que faça parte da variável de ambiente `PATH`,
poderá acessá-lo globalmente.
Em sistemas Unix, você pode até mesmo torná-lo executável e invocá-lo sem usar
diretamente o interpretador `php`.

Após executar o instalador seguindo
[as instruções da página de download](https://getcomposer.org/download/), você
pode executar o seguinte comando para mover o composer.phar para um diretório
que esteja na variável `PATH`:

```shell
mv composer.phar /usr/local/bin/composer
```

Se você preferir instalá-lo apenas para o seu usuário e evitar a necessidade de
permissões de root, você pode usar `~/.local/bin`, que está disponível por
padrão em algumas distribuições Linux.

> **Nota:** Se o comando acima falhar devido a permissões, você pode precisar
> executá-lo novamente com `sudo`.

> **Nota:** Em algumas versões do macOS, o diretório `/usr` não existe por
> padrão.
> Se ocorrer o erro `/usr/local/bin/composer: No such file or directory`, o
> diretório deverá ser criado manualmente antes de continuar:
> `mkdir -p /usr/local/bin`.

> **Nota:** Para obter informações sobre como alterar a variável `PATH`, leia o
> [artigo da Wikipédia](https://en.wikipedia.org/wiki/PATH_(variable)) e/ou use
> seu mecanismo de busca preferido.

Agora execute `composer` para executar o Composer em vez de `php composer.phar`.

## Instalação - Windows

### Usando o instalador

Esta é a maneira mais fácil de configurar o Composer na sua máquina.

Baixe e execute
[Composer-Setup.exe](https://getcomposer.org/Composer-Setup.exe).
Ele instalará a versão mais recente do Composer e configurará a variável `PATH`
para que você possa executar o `composer` de qualquer diretório na sua linha de
comando.

> **Nota:** Feche o terminal atual.
> Teste o uso em um novo terminal: isso é importante, pois a variável `PATH` só
> é carregada quando o terminal é iniciado.

### Instalação Manual

Acesse um diretório que esteja na variável `PATH` e execute o instalador
seguindo
[as instruções da página de download](https://getcomposer.org/download/) para
baixar o arquivo `composer.phar`.

Crie um arquivo `composer.bat` junto ao `composer.phar`:

Usando o `cmd.exe`:

```shell
C:\bin> echo @php "%~dp0composer.phar" %*>composer.bat
```

Usando o PowerShell:

```shell
PS C:\bin> Set-Content composer.bat '@php "%~dp0composer.phar" %*'
```

Adicione o diretório à sua variável de ambiente `PATH`, caso ainda não tenha
adicionado.
Para obter informações sobre como alterar sua variável `PATH`, consulte
[este artigo](https://www.computerhope.com/issues/ch000549.htm) e/ou
use seu mecanismo de busca preferido.

Feche o terminal atual.
Teste o uso em um novo terminal:

```shell
C:\Users\username>composer -V
```

```text
Composer version 2.4.0 2022-08-16 16:10:48
```

## Imagem Docker

O Composer é publicado como uma imagem Docker em alguns locais.
Consulte a lista no
[README do composer/docker](https://github.com/composer/docker).

Exemplo de uso:

```shell
docker pull composer/composer
docker run --rm -it -v "$(pwd):/app" composer/composer install
```

Para adicionar o Composer a um **Dockerfile** existente, basta copiar o arquivo
binário de imagens pré-construídas e de tamanho reduzido:

```Dockerfile
# Versão mais recente
COPY --from=composer/composer:latest-bin /composer /usr/bin/composer

# Versão específica
COPY --from=composer/composer:2-bin /composer /usr/bin/composer
```

**Nota:** você precisa instalar manualmente outras dependências de tempo de
execução dentro da sua imagem ao usar este método; consulte também
https://github.com/composer/composer/blob/main/README.md#binary-dependencies.

Leia a [descrição da imagem](https://hub.docker.com/r/composer/composer) para
obter mais informações de uso.

> **Nota:** Problemas específicos do Docker devem ser relatados
> [no repositório composer/docker](https://github.com/composer/docker/issues).

> **Nota:** Você também pode usar `composer` em vez de `composer/composer` como
> nome da imagem acima.
> É mais curto e é uma imagem oficial do Docker, mas não é publicada diretamente
> por nós e, portanto, geralmente recebe novas versões com alguns dias de
> atraso.

> **Importante**: Imagens com apelidos curtos não têm equivalentes apenas com
> binários, portanto, para a abordagem `COPY --from` é melhor usar as imagens
> `composer/composer`.

## Usando o Composer

Agora que você instalou o Composer, já pode usá-lo!
Vá para o próximo capítulo para uma breve demonstração.

[Uso básico](01-basic-usage.md) &rarr;
