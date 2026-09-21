---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/vendor-binaries.md
source_revision: 1c6ba6af4746d425213568e367af686b60df05b3
translation_status: ready

tagline: Exponha scripts de linha de comando de pacotes
---

# Binários de fornecedor e o diretório `vendor/bin`

## O que é um binário de fornecedor?

Qualquer script de linha de comando que um pacote do Composer deseje
disponibilizar à pessoa usuária que instalar o pacote deve ser listado como um
binário de fornecedor.

Se um pacote contiver outros scripts que não sejam necessários às pessoas
usuárias do pacote (como scripts de construção ou compilação), esse código não
deve ser listado como um binário de fornecedor.

## Como ele é definido?

Ele é definido adicionando a chave `bin` ao arquivo `composer.json` do projeto.
É especificado como um array de arquivos, permitindo adicionar múltiplos
binários a um determinado projeto.

```json
{
    "bin": ["bin/my-script", "bin/my-other-script"]
}
```

## O que definir um binário de fornecedor no `composer.json` faz?

Ela instrui o Composer a instalar os binários do pacote em `vendor/bin` para
qualquer projeto que **dependa** desse projeto.

Essa é uma maneira conveniente de disponibilizar scripts úteis que, de outra
forma, ficariam ocultos nas profundezas do diretório `vendor/`.

## O que acontece quando o Composer é executado em um `composer.json` que define binários de fornecedor?

Para os binários que um pacote define diretamente, nada acontece.

## O que acontece quando o Composer é executado em um `composer.json` que possui dependências com binários de fornecedor listados?

O Composer busca os binários definidos em todas as dependências.
Um arquivo proxy (ou dois, no Windows/WSL) é criado em `vendor/bin` para os
binários de cada dependência.

Suponha que o pacote `my-vendor/project-a` tenha binários configurados assim:

```json
{
    "name": "my-vendor/project-a",
    "bin": ["bin/project-a-bin"]
}
```

Executar `composer install` para este `composer.json` não fará nada com
`bin/project-a-bin`.

Suponha que o projeto `my-vendor/project-b` tenha as dependências configuradas
desta forma:

```json
{
    "name": "my-vendor/project-b",
    "require": {
        "my-vendor/project-a": "*"
    }
}
```

Ao executar `composer install` para este `composer.json`, o Composer analisará
todos os binários do `project-a` e os instalará em `vendor/bin`.

Nesse caso, o Composer disponibilizará
`vendor/my-vendor/project-a/bin/project-a-bin` como `vendor/bin/project-a-bin`.

## Localizando o autoloader do Composer a partir de um binário

A partir do Composer 2.2, uma nova variável global `$_composer_autoload_path`
é definida pelo arquivo proxy do binário; assim, quando o binário for executado,
ele poderá utilizá-la para localizar facilmente o autoloader do projeto.

No entanto, essa variável global não estará disponível ao executar binários
definidos pelo próprio pacote raiz; portanto, é necessário implementar uma
alternativa de contingência.

Um exemplo de como isso pode ser feito é:

```php
<?php

include $_composer_autoload_path ?? __DIR__ . '/../vendor/autoload.php';
```

Se você quiser depender disso em seu pacote, no entanto, certifique-se de também
exigir `"composer-runtime-api": "^2.2"` para garantir que o pacote seja
instalado com uma versão do Composer que suporte esse recurso.

## Localizando o diretório `bin-dir` do Composer a partir de um binário

A partir do Composer 2.2.2, uma nova variável global `$_composer_bin_dir` é
definida pelo arquivo proxy de binários; assim, quando seu binário for
executado, ele poderá utilizá-la para localizar facilmente o diretório de
binários do Composer no projeto.

Para binários que não são PHP, a partir do Composer 2.2.6, o proxy de binários
define uma variável de ambiente chamada `COMPOSER_RUNTIME_BIN_DIR`.

No entanto, essa variável global não estará disponível ao executar binários
definidos pelo próprio pacote raiz; portanto, é necessário implementar uma
alternativa de contingência.

Um exemplo de como fazer isso seria:

```php
<?php

$binDir = $_composer_bin_dir ?? __DIR__ . '/../vendor/bin';
```

```php
#!/bin/bash

if [[ -z "$COMPOSER_RUNTIME_BIN_DIR" ]]; then
  BIN_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"
else
  BIN_DIR="$COMPOSER_RUNTIME_BIN_DIR"
fi
```

Se você quiser depender disso em seu pacote, no entanto, certifique-se de também
exigir `"composer-runtime-api": "^2.2.2"` para garantir que o pacote seja
instalado com uma versão do Composer que suporte esse recurso.

## E quanto ao Windows e arquivos .bat?

Pacotes gerenciados inteiramente pelo Composer não *precisam* conter arquivos
`.bat` para compatibilidade com o Windows.
O Composer lida com a instalação de binários de uma maneira especial quando
executado em um ambiente Windows:

* Um arquivo `.bat` é gerado automaticamente para referenciar o binário.
* Um arquivo proxy no estilo Unix com o mesmo nome do binário também é gerado, o
  que é útil para WSL, VMs Linux, etc.

Pacotes que precisam oferecer suporte a fluxos de trabalho que podem não incluir
o Composer podem manter arquivos `.bat` personalizados.
Nesse caso, o pacote **não** deve listar o arquivo `.bat` como um binário, pois
isso não é necessário.

## Binários do fornecedor podem ser instalados em outro local que não seja `vendor/bin`?

Sim, existem duas maneiras de especificar um local alternativo para binários do vendor:

1. Definindo a configuração `bin-dir` no `composer.json`.
1. Definindo a variável de ambiente `COMPOSER_BIN_DIR`.

Um exemplo da primeira opção é assim:

```json
{
    "config": {
        "bin-dir": "scripts"
    }
}
```

Executar `composer install` para este `composer.json` fará com que todos os
binários de dependências sejam instalados em `scripts/` em vez de `vendor/bin/`.

Você pode definir `bin-dir` como `./` para colocar os binários na raiz do seu
projeto.
