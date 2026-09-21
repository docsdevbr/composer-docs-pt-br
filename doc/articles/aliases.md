---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/aliases.md
source_revision: bd4fd2cf94b9e7c72417bba963afec25130c7ce0
translation_status: ready

tagline: Associe nomes de branches a versões usando aliases
---

# Aliases

## Por que usar aliases?

Ao usar um repositório VCS, você só obterá versões comparáveis para branches que
se parecem com versões, como `2.0` ou `2.0.x`.
Para o seu branch `main`, você obterá uma versão `dev-main`.
Para o seu branch `bugfix`, você obterá uma versão `dev-bugfix`.

Se o seu branch `main` for usado para criar tags de lançamento da linha de
desenvolvimento `1.0`, isto é, `1.0.1`, `1.0.2`, `1.0.3`, etc., qualquer pacote
que dependa dele exigirá provavelmente a versão `1.0.*`.

Se alguém quiser exigir a versão `dev-main` mais recente, terá um problema:
outros pacotes podem exigir `1.0.*`, portanto, exigir essa versão de
desenvolvimento causará conflitos, já que `dev-main` não satisfaz a restrição
`1.0.*`.

É aí que entram os aliases (apelidos).

## Alias de branch

O branch `dev-main` existe no seu repositório VCS principal.
É bastante comum que alguém queira a versão de desenvolvimento mais recente do
branch principal.
Assim, o Composer permite que você crie um alias do seu branch `dev-main` para
uma versão `1.0.x-dev`.
Isso é feito especificando um campo `branch-alias` dentro de`extra` no arquivo
`composer.json`:

```json
{
    "extra": {
        "branch-alias": {
            "dev-main": "1.0.x-dev"
        }
    }
}
```

Se você criar um alias para uma versão não comparável (como `dev-develop`), o
prefixo `dev-` deve preceder o nome do branch.
Você também pode criar um alias para uma versão comparável (ou seja, que comece
com números e termine com `.x-dev`), mas apenas como uma versão mais específica.
Por exemplo, um branch `1.x` ou `1.x-dev` poderia ter seu alias alterado de
`1.x-dev` para `1.2.x-dev`, por ser mais específico.

O alias deve ser uma versão de desenvolvimento comparável (você não pode definir
um alias de `dev-main` para `dev-master`, por exemplo), e o `branch-alias` deve
estar presente no branch à qual ele faz referência.
Para criar um alias para `dev-main`, você precisa defini-lo e fazer o commit na
branch `main`.

Como resultado, qualquer pessoa pode agora exigir a versão `1.0.*` e o sistema
instalará automaticamente a `dev-main`.

Para usar aliases de branch, você deve ser a pessoa proprietária do repositório
do pacote que está recebendo o alias.
Se você quiser criar um alias para um pacote de terceiros sem manter um fork
dele, use aliases em linha, conforme descrito abaixo.

## Alias em linha no require

Aliases de branch são excelentes para criar aliases de linhas principais de
desenvolvimento.
No entanto, para usá-los, é necessário ter controle sobre o repositório de
origem e fazer o commit das alterações no controle de versão.

Isso não é nada prático quando você quer testar a correção de uma falha em uma
biblioteca que é dependência do seu projeto local.

Por isso, você pode definir aliases para pacotes diretamente nos campos
`require` e `require-dev`.
Suponha que você tenha encontrado uma falha no pacote `monolog/monolog`.
Você clonou o [Monolog](https://github.com/Seldaek/monolog) no GitHub e corrigiu
o problema em um branch chamado `bugfix`.
Agora, você deseja instalar essa versão do Monolog no seu projeto local.

Você está usando o `symfony/monolog-bundle`, que exige a versão `1.*` do
`monolog/monolog`.
Portanto, é necessário que a sua versão `dev-bugfix` atenda a essa restrição.

Adicione isto ao arquivo `composer.json` na raiz do seu projeto:

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/<você>/monolog"
        }
    ],
    "require": {
        "symfony/monolog-bundle": "2.0",
        "monolog/monolog": "dev-bugfix as 1.0.x-dev"
    }
}
```

Ou deixe o Composer adicionar isso para você com:

```shell
php composer.phar require "monolog/monolog:dev-bugfix as 1.0.x-dev"
```

Isso buscará a versão `dev-bugfix` do `monolog/monolog` no seu GitHub e criará
um alias para `1.0.x-dev`.

> **Nota:** O alias em linha é um recurso exclusivo do arquivo raiz.
> Se um pacote com aliases em linha for requerido, o alias (à direita de `as`)
> é usado como a restrição de versão.
> A parte à esquerda de `as` é descartada.
> Consequentemente, se A requer B e B requer `monolog/monolog` na versão
> `dev-bugfix as 1.0.x-dev`, a instalação de A fará com que B requeira
> `1.0.x-dev`, que pode existir como um alias de branch ou como um branch `1.0`
> real.
> Se não existir, o alias em linha deverá ser definido novamente no
> `composer.json` de A.

> **Nota:** O uso de aliases em linha deve ser evitado, especialmente para
> pacotes ou bibliotecas publicados.
> Se você encontrou uma falha, tente fazer com que sua correção seja integrada
> ao projeto original.
> Isso ajuda a evitar problemas para as pessoas usuárias do seu pacote.
