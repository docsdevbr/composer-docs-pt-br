---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/versions.md
source_revision: c23beac9c508b701bb481d1c5269e7a2a79e0b60
translation_status: ready

tagline: Explicação das versões.
---

# Versões e restrições

## Versões do Composer vs. versões do VCS

Como o Composer é fortemente voltado para o uso de sistemas de controle de
versão como o Git, o termo "versão" pode ser um pouco ambíguo.
No contexto de um sistema de controle de versão, uma "versão" é um conjunto
específico de arquivos que contém dados específicos.
Na terminologia do Git, isso é uma "ref" (referência) ou um commit específico,
que pode ser representado pelo HEAD de um branch ou por uma tag.
Ao fazer o checkout dessa versão no seu VCS, por exemplo, a tag `v1.1` ou o
commit `e35fa0d`, você está solicitando um conjunto único e conhecido de
arquivos, e sempre obtém os mesmos arquivos de volta.

No Composer, o que é frequentemente chamado de forma casual de versão, isto é, a
string que segue o nome do pacote em uma linha `require` (ex.: `~1.1` ou
`1.2.*`) é, na verdade, mais especificamente uma restrição de versão.
O Composer utiliza restrições de versão para determinar quais refs em um VCS
devem ser baixadas via checkout (ou para verificar se uma determinada biblioteca
é aceitável, no caso de uma biblioteca mantida estaticamente que possua uma
especificação de `version` no arquivo `composer.json`).

## Tags e branches do VCS

*Para a discussão a seguir, vamos assumir o seguinte repositório de biblioteca
de exemplo:*

```shell
~/my-library$ git branch
```
```text
v1
v2
my-feature
another-feature
```

```shell
~/my-library$ git tag
```
```text
v1.0
v1.0.1
v1.0.2
v1.1-BETA
v1.1-RC1
v1.1-RC2
v1.1
v1.1.1
v2.0-BETA
v2.0-RC1
v2.0
v2.0.1
v2.0.2
```

### Tags

Normalmente, o Composer lida com tags, ao contrário de branches (se você não
sabe o que isso significa, informe-se sobre
[sistemas de controle de versão](https://en.wikipedia.org/wiki/Version_control#Common_terminology)).
Ao definir uma restrição de versão, ela pode referenciar uma tag específica (por
exemplo, `1.1`) ou um intervalo válido de tags (por exemplo, `>=1.1 <2.0` ou
`~4.0`).
Para resolver essas restrições, o Composer primeiro solicita ao VCS que liste
todas as tags disponíveis e, em seguida, cria uma lista interna de versões
disponíveis com base nessas tags.
No exemplo acima, a lista interna do Composer inclui as versões `1.0`, `1.0.1`,
`1.0.2`, a versão beta da `1.1`, as primeira e segunda versões candidatas da
`1.1`, a versão final `1.1`, etc.
(Observe que o Composer remove automaticamente o prefixo 'v' do nome da tag
original para obter um número de versão final válido.)

Quando o Composer possui a lista completa de versões disponíveis do seu VCS, ele
identifica a versão mais recente que atende a todas as restrições de versão do
seu projeto (é possível que outros pacotes exijam versões da biblioteca mais
específicas do que a sua; portanto, a versão escolhida nem sempre será a versão
mais recente disponível) e baixa um arquivo zip dessa tag para descompactá-lo no
local correto dentro do seu diretório `vendor`.

### Branches

Se você quiser que o Composer faça o checkout de um branch em vez de uma tag, é
preciso indicá-lo usando o prefixo especial `dev-*` (ou, às vezes, um sufixo;
veja abaixo).
Ao obter um branch, assume-se que você deseja *trabalhar* nele; por isso, o
Composer clona realmente o repositório no local correto dentro do seu diretório
`vendor`.
Já para tags, ele copia apenas os arquivos necessários, sem clonar o repositório
inteiro.
(Você pode alterar esse comportamento com `--prefer-source` e `--prefer-dist`;
consulte as [opções de instalação](../03-cli.md#install).)

No exemplo acima, se você quisesse obter o branch `my-feature`, deveria
especificar `dev-my-feature` como a restrição de versão na cláusula `require`.
Isso faria com que o Composer clonasse o repositório `my-library` no seu
diretório `vendor` e realizasse o checkout do branch `my-feature`.

Quando os nomes dos branches se assemelham a versões, é preciso deixar claro
para o Composer que estamos tentando obter um branch, e não uma tag.
No exemplo acima, temos dois branches de versão: `v1` e `v2`.
Para fazer com que o Composer obtenha um desses branches, você deve especificar
uma restrição de versão como esta: `v1.x-dev`.
O `.x` é uma string arbitrária exigida pelo Composer para indicar que estamos
nos referindo ao branch `v1` e não a uma tag `v1` (alternativamente, você pode
nomear o branch como `v1.x` em vez de `v1`).
No caso de um branch com um nome que lembra uma versão (como `v1`, neste
exemplo), adiciona-se `-dev` como sufixo, em vez de usar `dev-` como prefixo.

### Níveis de estabilidade

O Composer reconhece os seguintes níveis de estabilidade (em ordem de
estabilidade): `dev`, `alpha`, `beta`, `RC` e `stable` (sendo que `RC` significa
release candidate).
A estabilidade de uma versão é definida pelo seu sufixo; por exemplo, a versão
`v1.1-BETA` tem estabilidade `beta` e a `v1.1-RC1` tem estabilidade `RC`.
Se tal sufixo estiver ausente, como na versão `v1.1`, o Composer considera essa
versão como `stable`.
Além disso, o Composer adiciona automaticamente o sufixo `-dev` a todos os
branches numéricos e prefixa todos os outros branches importados de um
repositório VCS com `dev-`.
Em ambos os casos, é atribuída a estabilidade `dev`.

Ter isso em mente ajudará você na próxima seção.

### Estabilidade mínima

Há mais um fator que influencia quais arquivos são extraídos do VCS de uma
biblioteca e adicionados ao seu projeto: o Composer permite definir restrições
de estabilidade para limitar quais tags são consideradas válidas.
No exemplo acima, observe que a biblioteca lançou uma versão beta e duas release
candidates para a versão `1.1` antes do lançamento oficial final.
Para obter essas versões ao executar `composer install` ou `composer update`,
precisamos informar explicitamente ao Composer que aceitamos release candidates
e versões beta (além de versões alpha, caso desejemos).
Isso pode ser feito definindo um valor `minimum-stability` para todo o projeto
no arquivo `composer.json` ou utilizando "flags de estabilidade" nas restrições
de versão.
Leia mais na [página do esquema](../04-schema.md#minimum-stability).

## Definindo restrições de versão

Agora que você tem uma noção de como o Composer interpreta as versões, vamos
falar sobre como definir restrições de versão para as dependências do seu
projeto.

### Restrição de versão exata

Você pode especificar a versão exata de um pacote.
Isso instrui o Composer a instalar apenas essa versão específica.
Se outras dependências exigirem uma versão diferente, o resolvedor de
dependências falhará e interromperá qualquer procedimento de instalação ou
atualização.

Exemplo: `1.0.2`

### Intervalo de versões

Ao utilizar operadores de comparação, você pode especificar intervalos de
versões válidas.
Os operadores válidos são `>`, `>=`, `<`, `<=`, `!=`.

Você pode definir múltiplos intervalos.
Intervalos separados por um espaço (<code>&nbsp;</code>) ou vírgula (`,`) serão
tratados como um **E lógico**.
Uma barra dupla (`||`) será tratada como um **OU lógico**.
O operador E tem precedência sobre o OU.

> **Nota:** Tenha cuidado ao usar intervalos sem limite superior, pois você pode
> acabar instalando inesperadamente versões que quebram a compatibilidade com
> versões anteriores.
> Considere usar o operador [circunflexo](#caret-version-range-) em vez disso,
> por segurança.

> **Nota:** Em versões mais antigas do Composer, a barra simples (`|`) era a
> alternativa recomendada para o **OU lógico** (OR).
> Portanto, para manter a compatibilidade com versões anteriores, a barra
> simples (`|`) ainda será tratada como um **OU lógico**.

Exemplos:

* `>=1.0`
* `>=1.0 <2.0`
* `>=1.0 <1.1 || >=1.2`

### Intervalo de versões com hífen (` - `)

Um intervalo inclusivo de versões.
Versões parciais à direita são completadas com um caractere curinga.
Por exemplo, `1.0 - 2.0` equivale a `>=1.0.0 <2.1`, pois `2.0` torna-se `2.0.*`.
Por outro lado, `1.0.0 - 2.1.0` equivale a `>=1.0.0 <=2.1.0`.

Exemplo: `1.0 - 2.0`

### Intervalo de versões com curinga (`.*`)

Você pode especificar um padrão usando o curinga `*`. `1.0.*` equivale a
`>=1.0 <1.1`.

Exemplo: `1.0.*`

## Operadores de próxima versão significativa

### Operador de intervalo de versão til (`~`)

O operador `~` é melhor explicado por exemplos: `~1.2` é equivalente a
`>=1.2 <2.0.0`, enquanto `~1.2.3` é equivalente a `>=1.2.3 <1.3.0`.
Como você pode ver, ele é especialmente útil para projetos que seguem o
[versionamento semântico](https://semver.org/).
Um uso comum seria definir a versão secundária mínima da qual você depende, como
`~1.2` (o que permite qualquer versão até, mas não incluindo, a 2.0).
Visto que, em teoria, não deve haver quebras de compatibilidade retroativa até a
versão 2.0, isso funciona bem.
Outra forma de entender isso é que o uso de `~` especifica uma versão mínima,
mas permite que o último dígito especificado aumente.

Exemplo: `~1.2`

> **Nota:** Embora `2.0-beta.1` seja estritamente anterior a `2.0`, uma
> restrição de versão como `~1.2` não a instalaria.
> Como mencionado acima, `~1.2` significa apenas que a parte `.2` pode mudar,
> mas a parte `1.` permanece fixa.

> **Nota:** O operador `~` possui uma exceção em seu comportamento em relação ao
> número da versão principal.
> Isso significa, por exemplo, que `~1` é o mesmo que `~1.0`, pois ele não
> permite que o número número da versão principal aumente, buscando manter a
> compatibilidade retroativa.

### Intervalo de versão com circunflexo (`^`)

O operador `^` comporta-se de maneira muito semelhante, mas segue mais
rigorosamente o versionamento semântico, permitindo sempre atualizações que não
quebram a compatibilidade.
Por exemplo, `^1.2.3` é equivalente a `>=1.2.3 <2.0.0`, uma vez que nenhum dos
lançamentos anteriores à versão 2.0 deve quebrar a compatibilidade com versões
anteriores.
Para versões anteriores à 1.0, ele também prioriza a segurança, tratando `^0.3`
como `>=0.3.0 <0.4.0` e `^0.0.3` como `>=0.0.3 <0.0.4`.

Este é o operador recomendado para garantir a máxima interoperabilidade ao
escrever código de bibliotecas.

Exemplo: `^1.2.3`

> **Nota:** Se você estiver usando o PowerShell no Windows, precisará escapar
> os caracteres de circunflexo (`^`) ao utilizá-los como argumento na CLI, por
> exemplo, ao usar o comando `composer require`.
> É necessário usar quatro caracteres de circunflexo consecutivos, como
> `^^^^1.2.3`, para garantir que o operador seja passado corretamente para o
> Composer.

## Restrições de estabilidade

Se você estiver usando uma restrição que não define explicitamente uma
estabilidade, o Composer adotará internamente o padrão `-dev` ou `-stable`,
dependendo dos operadores utilizados.
Isso ocorre de forma transparente.

Se você quiser considerar explicitamente apenas a versão estável na comparação,
adicione o sufixo `-stable`.

Exemplos:

Restrição           | Internamente
------------------- | ------------------------
 `1.2.3`            | `=1.2.3.0-stable`
 `>1.2`             | `>1.2.0.0-stable`
 `>=1.2`            | `>=1.2.0.0-dev`
 `>=1.2-stable`     | `>=1.2.0.0-stable`
 `<1.3`             | `<1.3.0.0-dev`
 `<=1.3`            | `<=1.3.0.0-stable`
 `1 - 2`            | `>=1.0.0.0-dev <3.0.0.0-dev`
 `~1.3`             | `>=1.3.0.0-dev <2.0.0.0-dev`
 `1.4.*`            | `>=1.4.0.0-dev <1.5.0.0-dev`

Para permitir diferentes níveis de estabilidade sem impô-los no nível da
restrição, você pode usar
[flags de estabilidade](../04-schema.md#links-de-pacotes), como `@<stability>`
(por exemplo, `@dev`), para informar ao Composer que um determinado pacote pode
ser instalado com um nível de estabilidade diferente da sua configuração padrão
de `minimum-stability`.
Todas as flags de estabilidade disponíveis estão listadas na seção
`minimum-stability` da [página do esquema](../04-schema.md#minimum-stability).

## Resumo

```jsonc
"require": {
    "vendor/package": "1.3.2", // exactly 1.3.2

    // >, <, >=, <= | specify upper / lower bounds
    "vendor/package": ">=1.3.2", // anything above or equal to 1.3.2
    "vendor/package": "<1.3.2", // anything below 1.3.2

    // * | wildcard
    "vendor/package": "1.3.*", // >=1.3.0 <1.4.0

    // ~ | allows last digit specified to go up
    "vendor/package": "~1.3.2", // >=1.3.2 <1.4.0
    "vendor/package": "~1.3", // >=1.3.0 <2.0.0

    // ^ | doesn't allow breaking changes (major version fixed - following semver)
    "vendor/package": "^1.3.2", // >=1.3.2 <2.0.0
    "vendor/package": "^0.3.2", // >=0.3.2 <0.4.0 // except if major version is 0
}
```

## Testando restrições de versão

Você pode testar restrições de versão usando o
[semver.madewithlove.com](https://semver.madewithlove.com).
Insira o nome de um pacote e a ferramenta preencherá automaticamente a restrição
de versão padrão que o Composer adicionaria ao seu arquivo `composer.json`.
Você pode ajustar a restrição de versão, e a ferramenta destacará todas as
versões lançadas que atendem a esse critério.
