---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/repository-priorities.md
source_revision: 239638e687e8e67a2f1dadf225731fcd6309e8c4
translation_status: ready

tagline: Configure quais pacotes são encontrados em quais repositórios
---

# Prioridade dos repositórios

## Repositórios canônicos

Quando o Composer resolve dependências, ele procura um determinado pacote no
repositório de maior prioridade (o primeiro da lista).
Se esse repositório não contiver o pacote, ele passa para o próximo, até que um
repositório o contenha e o processo termine.

Repositórios canônicos são preferíveis por alguns motivos:

- Em termos de desempenho, é mais eficiente parar de procurar um pacote assim
  que ele é encontrado em algum lugar.
  Isso também evita carregar pacotes duplicados caso o mesmo pacote esteja
  presente em vários dos seus repositórios.
- Em termos de segurança, é mais seguro tratá-los como canônicos, pois isso
  garante que pacotes que você espera que venham de seus repositórios mais
  importantes nunca sejam carregados de outro repositório.
  Digamos que você tenha um repositório privado que não seja canônico e precise
  do seu pacote privado `foo/bar ^2.0`, por exemplo.
  Se alguém publicar `foo/bar 2.999` no packagist.org, o Composer escolherá esse
  pacote, pois ele tem uma versão superior à sua versão mais recente (digamos,
  2.4.3), e você acaba instalando algo que talvez não pretendesse.
  No entanto, se o repositório privado for canônico, essa versão 2.999 do
  packagist.org não será considerada de forma alguma.

Existem, no entanto, alguns casos em que você pode querer carregar
especificamente alguns pacotes de um determinado repositório, mas não todos.
Ou você pode querer que um determinado repositório não seja canônico, sendo
preferido apenas se contiver versões de pacotes superiores às dos repositórios
definidos abaixo dele.

## Comportamento padrão

Por padrão, no Composer 2.x, todos os repositórios são canônicos.
O Composer 1.x tratava todos os repositórios como não canônicos.

Outro comportamento padrão é que o repositório packagist.org é sempre adicionado
implicitamente como o último repositório, a menos que você o
[desabilite](../05-repositories.md#desabilitando-o-packagist.org).

## Tornando repositórios não canônicos

Você pode adicionar a opção `canonical` a qualquer repositório para desabilitar
esse comportamento padrão e garantir que o Composer continue procurando em
outros repositórios, mesmo que aquele repositório contenha um determinado
pacote.

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://example.org",
            "canonical": false
        }
    ]
}
```

## Filtrando pacotes

Você também pode filtrar os pacotes que um repositório poderá carregar, seja
selecionando aqueles que deseja ou excluindo aqueles que não deseja.

Por exemplo, aqui queremos selecionar apenas o pacote `foo/bar` e todos os
pacotes de `some-vendor/` deste repositório Composer.

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://example.org",
            "only": ["foo/bar", "some-vendor/*"]
        }
    ]
}
```

E neste outro exemplo, excluímos `toy/package` de um repositório, pois talvez
não queiramos carregá-lo neste projeto.

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://example.org",
            "exclude": ["toy/package"]
        }
    ]
}
```

Tanto `only` quanto `exclude` devem ser arrays de nomes de pacotes, que também
podem conter caracteres curinga (`*`), os quais correspondem a qualquer
caractere.
