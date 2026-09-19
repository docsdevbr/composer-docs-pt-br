---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/02-libraries.md
source_revision: 83212118cbaf7ab44b51f8afcd45a7540275e639
translation_status: ready
---

# Bibliotecas

Este capítulo te ensinará como tornar sua biblioteca instalável através do
Composer.

## Todo projeto é um pacote

Assim que você tiver um arquivo `composer.json` em um diretório, esse diretório
será um pacote.
Quando você adiciona um [`require`](04-schema.md#require) a um projeto, você
está criando um pacote que depende de outros pacotes.
A única diferença entre seu projeto e uma biblioteca é que seu projeto é um
pacote sem nome.

Para tornar esse pacote instalável, você precisa dar um nome a ele.
Você faz isso adicionando a propriedade [`name`](04-schema.md#name) ao
`composer.json`:

```json
{
    "name": "acme/ola-mundo",
    "require": {
        "monolog/monolog": "1.0.*"
    }
}
```

Neste caso, o nome do projeto é `acme/ola-mundo`, onde `acme` é o nome do
fornecedor.
Fornecer um nome de fornecedor é obrigatório.

> **Nota:** Se você não sabe o que usar como nome de fornecedor, seu nome de
> usuário do GitHub geralmente é uma boa aposta.
> Os nomes dos pacotes devem estar em letras minúsculas e a convenção é usar
> traços para separação de palavras.

## Versionamento de biblioteca

Na grande maioria dos casos, você manterá sua biblioteca usando algum tipo de
sistema de controle de versão como git, svn, hg ou fossil.
Nesses casos, o Composer infere as versões a partir do seu VCS e você **não
deve** especificar uma versão no seu arquivo `composer.json`.
(Consulte o [artigo sobre versões](articles/versions.md) para saber como o
Composer usa branches e tags do VCS para resolver restrições de versão.)

Se você estiver mantendo pacotes manualmente (ou seja, sem um VCS), precisará
especificar a versão explicitamente, adicionando uma propriedade `version` no
seu arquivo `composer.json`:

```json
{
    "version": "1.0.0"
}
```

> **Nota:** Quando você adiciona uma versão fixa no código ao VCS, a versão
> entrará em conflito com os nomes das tags.
> O Composer não poderá determinar o número da versão.

### Versionamento do VCS

O Composer usa as tags e branches do VCS para resolver as restrições de versão
que você especifica no campo [`require`](04-schema.md#require) para conjuntos
específicos de arquivos.
Ao determinar as versões válidas disponíveis, o Composer examina todas as suas
tags e branches, e converte os seus nomes para uma lista interna de opções que
ele então compara com a restrição de versão que você forneceu.

Para mais informações sobre como o Composer trata tags e branches e como ele
resolve restrições de versão de pacote, leia o artigo sobre
[versões](articles/versions.md).

## Arquivo de lock

Para sua biblioteca, você pode fazer o commit do arquivo `composer.lock`, se
quiser.
Isso pode ajudar o seu time a testar sempre com as mesmas versões das
dependências.
No entanto, esse arquivo de lock não terá nenhum efeito em outros projetos que
dependem da sua biblioteca.
Ele só tem efeito no projeto principal.

Se você não quiser fazer o commit do arquivo de lock e estiver usando o git,
adicione-o ao `.gitignore`.

## Publicando em um VCS

Após ter um repositório VCS (sistema de controle de versão, por exemplo, git)
contendo um arquivo `composer.json`, sua biblioteca já pode ser instalada pelo
Composer.
Neste exemplo, publicaremos a biblioteca `acme/ola-mundo` no GitHub em
`github.com/<usuário>/ola-mundo`.

Agora, para testar a instalação do pacote `acme/ola-mundo`, criamos um projeto
localmente.
Vamos chamá-lo de `acme/blog`.
Este blog dependerá de `acme/ola-mundo`, que por sua vez depende de
`monolog/monolog`.
Podemos fazer isso criando um diretório `blog` em algum lugar, contendo um
`composer.json`:

```json
{
    "name": "acme/blog",
    "require": {
        "acme/ola-mundo": "dev-master"
    }
}
```

O nome não é necessário neste caso, pois não queremos publicar o blog como uma
biblioteca.
Ele é adicionado aqui para esclarecer qual `composer.json` está sendo descrito.

Agora precisamos informar à aplicação do blog onde encontrar a dependência
`ola-mundo`.
Fazemos isso adicionando uma especificação de repositório de pacotes ao
`composer.json` do blog:

```json
{
    "name": "acme/blog",
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/<usuário>/ola-mundo"
        }
    ],
    "require": {
        "acme/ola-mundo": "dev-master"
    }
}
```

Para mais detalhes sobre como os repositórios de pacotes funcionam e quais
outros tipos estão disponíveis, consulte [Repositórios](05-repositories.md).

Isso é tudo. Agora você pode instalar as dependências executando o comando
[`install`](03-cli.md#install) do Composer!

**Recapitulando:** qualquer repositório git/svn/hg/fossil que contenha um
`composer.json` pode ser adicionado ao seu projeto especificando o repositório
do pacote e declarando a dependência no campo [`require`](04-schema.md#require).

## Publicando no Packagist

Tudo bem, agora você pode publicar pacotes.
Mas especificar o repositório VCS o tempo todo é trabalhoso.
Você não quer forçar todas as suas pessoas usuárias a fazer isso.

A outra coisa que você deve ter notado é que não especificamos um repositório de
pacotes para o `monolog/monolog`.
Como isso funcionou?
A resposta é Packagist.

O [Packagist](https://packagist.org/) é o principal repositório de pacotes do
Composer e está habilitado por padrão.
Tudo o que é publicado no Packagist fica disponível automaticamente através do
Composer.
Como o
[Monolog está no Packagist](https://packagist.org/packages/monolog/monolog),
podemos depender dele sem precisar especificar nenhum repositório adicional.

Se quiséssemos compartilhar o `ola-mundo` com o mundo, também iríamos publicá-lo
no Packagist.

Você acessa o [Packagist](https://packagist.org) e clica no botão "Submit".
O sistema solicitará que você faça um cadastro, caso ainda não o tenha feito, e
permitirá que você envie a URL do seu repositório VCS, momento no qual o
Packagist começará a processá-lo.
Feito isso, seu pacote estará disponível para qualquer pessoa!

## Pacotes de distribuição leves

Algumas informações inúteis como o diretório `.github`, ou grandes exemplos,
dados de teste, etc., normalmente não devem ser incluídas em pacotes
distribuídos.

O arquivo `.gitattributes` é um arquivo específico do git, que assim como
`.gitignore`, fica no diretório raiz da sua biblioteca.
Ele substitui a configuração local e global (`.git/config` e `~/.gitconfig`,
respectivamente) quando presente e rastreado pelo git.

Use `.gitattributes` para evitar que arquivos indesejados inchem os pacotes de
distribuição zip.

```text
// .gitattributes
/demo export-ignore
phpunit.xml.dist export-ignore
/.github/ export-ignore
```

Teste inspecionando o arquivo zip gerado manualmente:

```shell
git archive <branch> --format zip -o arquivo.zip
```

> **Nota:** os arquivos ainda seriam rastreados pelo git, mas não incluídos na
> distribuição zip.
> Isso só funciona para pacotes instalados de `dist` (ou seja, lançamentos com
> tag) vindos do GitHub, GitLab ou Bitbucket.

&larr; [Uso básico](01-basic-usage.md) | [Interface de Linha de Comando](03-cli.md) &rarr;
