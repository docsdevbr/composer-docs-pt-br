---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/composer-platform-dependencies.md
source_revision: 6198fc1053ede6af9b401a75f4badf0311d2adc1
translation_status: ready

tagline: Fazendo seu pacote depender de versões específicas do Composer
---

# Dependências de plataforma do Composer

## O que são dependências de plataforma

O Composer disponibiliza informações sobre o ambiente em que é executado na
forma de pacotes virtuais.
Isso permite que outros pacotes definam dependências
([require](../04-schema.md#require), [conflict](../04-schema.md#conflict),
[provide](../04-schema.md#provide), [replace](../04-schema.md#replace)) em
relação a diferentes aspectos da plataforma, como PHP, extensões ou bibliotecas
do sistema, incluindo restrições de versão.

Ao declarar uma dependência em um dos pacotes de plataforma, nenhum código é
instalado.
Os números de versão dos pacotes de plataforma são derivados do ambiente onde o
Composer é executado e não podem ser atualizados ou removidos.
No entanto, eles podem ser substituídos para fins de resolução de dependências
por meio de uma [configuração de plataforma](../06-config.md#platform).

**Por exemplo:** se você estiver executando `composer update` com um
interpretador PHP na versão `7.4.42`, o Composer adiciona automaticamente um
pacote chamado `php` ao conjunto de pacotes disponíveis e atribui a ele a versão
`7.4.42`.

É assim que os pacotes podem definir uma dependência em relação à versão do PHP
usada:

```json
{
    "require": {
        "php": ">=7.4"
    }
}
```

O Composer verificará esse requisito em relação à versão do PHP em uso ao
executar o comando do Composer.

### Diferentes tipos de pacotes de plataforma

Existem os seguintes tipos de pacotes de plataforma, dos quais é possível
depender:

1. PHP (`php` e os subtipos: `php-64bit`, `php-ipv6`, `php-zts`, `php-debug`).
2. Extensões do PHP (`ext-*`, ex.: `ext-mbstring`).
3. Bibliotecas do PHP (`lib-*`, ex.: `lib-curl`).
4. Composer (`composer`, `composer-plugin-api`, `composer-runtime-api`).

Para ver a lista completa de pacotes de plataforma disponíveis em seu ambiente,
você pode executar `php composer.phar show --platform` (ou `show -p`, de forma
abreviada).

As diferenças entre os vários pacotes de plataforma do Composer são explicadas
mais adiante neste documento.

## Pacote de plugin `composer-plugin-api`

Você pode modificar o comportamento do Composer com pacotes de
[plugin](plugins.md).
O Composer fornece um conjunto de APIs versionadas para plugins.
Como alterações internas no Composer podem **não** alterar as APIs de plugin, a
versão da API pode não aumentar toda vez que a versão do Composer aumentar.
Por exemplo: na versão `2.3.12` do Composer, a versão do `composer-plugin-api`
ainda poderia ser `2.2.0`.

## Pacote de tempo de execução `composer-runtime-api`

Quando aplicações instaladas com o Composer são executadas (seja via CLI ou por
meio de uma requisição web), elas requerem o arquivo `vendor/autoload.php`,
tipicamente como uma das primeiras linhas de código executado.
Invocações do autoloader do Composer são consideradas o "tempo de execução" da
aplicação.

A partir da versão 2.0, o Composer disponibiliza
[recursos adicionais](../07-runtime.md) (além do registro do autoloader de
classes) para o ambiente do tempo de execução da aplicação.

Assim como ocorre com o `composer-plugin-api`, nem todo lançamento do Composer
adiciona novos recursos de tempo de execução; portanto, a versão do
`composer-runtime-api` também é incrementada independentemente da versão do
Composer.

## Pacote `composer` do Composer

A partir do Composer 2.2.0, está disponível um novo pacote de plataforma chamado
`composer`, que representa a versão exata do Composer que está sendo executada.
Pacotes que dependem desse pacote de plataforma podem, assim, depender de (ou
entrar em conflito com) versões específicas do Composer para cobrir casos
excepcionais em que nem a versão do `composer-runtime-api` nem a do
`composer-plugin-api` foram alteradas.

Como essa opção foi introduzida no Composer 2.2.0, recomenda-se adicionar uma
dependência do `composer-plugin-api` com versão mínima `>=2.2.0` para fornecer
uma mensagem de erro mais clara às pessoas usuárias que executam versões mais
antigas do Composer.

Em geral, depender de `composer-plugin-api` ou `composer-runtime-api` é sempre
recomendado em vez de depender de versões concretas do Composer através do
pacote de plataforma `composer`.
