---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/04-schema.md
source_revision: 8fbad13554162188f900ff149454932500d508a8
translation_status: ready
---

# O esquema do composer.json

Este capítulo explicará todos os campos disponíveis no `composer.json`.

## Esquema JSON

Nós temos um [esquema JSON](https://json-schema.org) que documenta o formato e
também pode ser usado para validar seu `composer.json`.
De fato, ele é usado pelo comando `validate`.
Você pode encontrá-lo no
[site do Composer](https://getcomposer.org/schema.json).

## Pacote raiz

O pacote raiz é o pacote definido pelo `composer.json` na raiz do seu projeto.
É o `composer.json` principal que define os requisitos do seu projeto.

Certos campos se aplicam apenas no contexto do pacote raiz.
Um exemplo disto é o campo `config`.
Somente o pacote raiz pode definir a configuração.
O campo `config` das dependências é ignorado.
Isto faz do campo `config` um campo `root-only`.

> **Nota:** Um pacote pode ser o pacote raiz ou não, dependendo do contexto.
> Por exemplo, se seu projeto depende da biblioteca `monolog`, seu projeto é o
> pacote raiz.
> No entanto, se você clonar o `monolog` no GitHub para corrigir um erro, então
> o `monolog` é o pacote raiz.

## Propriedades

### name

O nome do pacote.
Consiste no nome do vendor e no nome do projeto, separados por `/`.
Exemplos:

* monolog/monolog
* igorw/event-source

O nome deve estar em letras minúsculas e consistir em palavras separadas por
`-`, `.` ou `_`.
O nome completo deve corresponder a
`^[a-z0-9]([_.-]?[a-z0-9]+)*/[a-z0-9](([_.]|-{1,2})?[a-z0-9]+)*$`.

A propriedade `name` é obrigatória para pacotes publicados (bibliotecas).

> **Nota:** Antes da versão 2.0 do Composer, um nome podia conter qualquer
> caractere, inclusive espaços em branco.

### description

Uma breve descrição do pacote.
Normalmente, tem apenas uma linha de comprimento.

Obrigatório para pacotes publicados (bibliotecas).

### version

A versão do pacote.
Geralmente, não é necessária e deve ser omitida (consulte abaixo).

Ela deve seguir o formato `X.Y.Z` ou `vX.Y.Z` com um sufixo opcional `-dev`,
`-patch` (`-p`), `-alpha` (`-a`), `-beta` (`-b`) ou `-RC`.
Os sufixos patch, alpha, beta e RC podem ser seguidos por um número.

Exemplos:

- 1.0.0
- 1.0.2
- 1.1.0
- 0.2.5
- 1.0.0-dev
- 1.0.0-alpha3
- 1.0.0-beta2
- 1.0.0-RC5
- v2.0.4-p1

Opcional se o repositório do pacote puder inferir a versão de algum lugar, como
o nome da tag no repositório VCS.
Neste caso, também é recomendável omiti-la.

> **Nota:** O Packagist usa repositórios VCS, portanto, a declaração acima
> também é verdadeira para o Packagist.
> Especificar a versão por conta própria provavelmente criará problemas em algum
> momento devido a erro humano.

### type

O tipo do pacote.
O padrão é `library`.

Os tipos de pacote são usados para lógica de instalação personalizada.
Se você tiver um pacote que precise de alguma lógica especial, você pode definir
um tipo personalizado.
Pode ser, por exemplo, `symfony-bundle`, `wordpress-plugin` ou
`typo3-cms-extension`.
Estes tipos serão específicos para determinados projetos e precisarão fornecer
um instalador capaz de instalar pacotes deste tipo.

Por padrão, o Composer oferece suporte a quatro tipos:

- **library:** este é o padrão.
  Ele simplesmente copiará os arquivos para `vendor`.
- **project:** denota um projeto em vez de uma biblioteca.
  Por exemplo, shells de aplicações como a
  [Edição Padrão do Symfony](https://github.com/symfony/symfony-standard), CMSs
  como o
  [instalador do SilverStripe](https://github.com/silverstripe/silverstripe-installer)
  ou aplicações completas distribuídas como pacotes.
  Isto pode ser usado, por exemplo, pelas IDEs para fornecer listagens de
  projetos a serem inicializados ao criar um workspace.
- **metapackage:** um pacote vazio que contém requisitos e acionará suas
  instalações, mas não contém nenhum arquivo e não gravará nada no sistema de
  arquivos.
  Sendo assim, não requer uma chave `dist` ou `source` para ser instalável.
- **composer-plugin:** um pacote do tipo `composer-plugin` pode fornecer um
  instalador para outros pacotes que possuem um tipo personalizado.
  Leia mais no [artigo dedicado](articles/custom-installers.md).
- **php-ext** e **php-ext-zend**: estes nomes são reservados para pacotes de
  extensão do PHP escritos em C.
  Não use esses tipos para pacotes escritos em PHP.

Use um tipo personalizado somente se precisar de lógica personalizada durante a
instalação.
É recomendável omitir este campo e usar o padrão `library`.

### keywords

Um array de palavras-chave às quais o pacote está relacionado.
Elas podem ser usadas para pesquisa e filtragem.

Exemplos:

- logging
- events
- database
- redis
- templating

> **Nota**: Algumas palavras-chave especiais acionam o `composer require` sem a
> opção `--dev`, perguntando às pessoas usuárias se desejam adicionar esses
> pacotes à seção `require-dev` em vez de `require`.
> São elas: `dev`, `testing`, `static analysis`.

> **Nota**: O conjunto de caracteres permitidos na string restringe-se a letras
> ou números Unicode, espaço `" "`, ponto `.`, sublinhado `_` e hífen `-`.
> (Regex: `'{^[\p{N}\p{L} ._-]+$}u'`)
> O uso de outros caracteres gerará um aviso ao executar o `composer validate` e
> fará com que a atualização do pacote falhe no Packagist.org.

Opcional.

### homepage

Uma URL para o site do projeto.

Opcional.

### readme

Um caminho relativo para o documento README.
O padrão é `README.md`.

Isso é útil principalmente para pacotes que não estão no GitHub, pois para
pacotes do GitHub, o Packagist.org usará a API do README para buscar o arquivo
detectado pelo GitHub.

Opcional.

### time

Data de lançamento da versão.

Deve estar no formato `AAAA-MM-DD` ou `AAAA-MM-DD HH:MM:SS` no fuso horário UTC.

Opcional.

### license

A licença do pacote.
Pode ser uma string ou um array de strings.

A notação recomendada para as licenças mais comuns é (em ordem alfabética):

- Apache-2.0
- BSD-2-Clause
- BSD-3-Clause
- BSD-4-Clause
- GPL-2.0-only / GPL-2.0-or-later
- GPL-3.0-only / GPL-3.0-or-later
- LGPL-2.1-only / LGPL-2.1-or-later
- LGPL-3.0-only / LGPL-3.0-or-later
- MIT

Opcional, mas é altamente recomendável fornecê-la.
Mais identificadores estão listados no
[Registro de Licenças de Código Aberto SPDX](https://spdx.org/licenses/).

> **Nota:** Para software de código fechado, você pode usar `"proprietary"` como
> identificador de licença.

Um exemplo:

```json
{
    "license": "MIT"
}
```

Para um pacote, quando há uma escolha entre as licenças ("licenças
disjuntivas"), várias podem ser especificadas como array.

Um exemplo usando licenças disjuntivas:

```json
{
    "license": [
       "LGPL-2.1-only",
       "GPL-3.0-or-later"
    ]
}
```

Alternativamente, elas podem ser separadas por `or` e colocadas entre
parênteses;

```json
{
    "license": "(LGPL-2.1-only or GPL-3.0-or-later)"
}
```

Da mesma forma, quando várias licenças precisam ser aplicadas ("licenças
conjuntivas"), elas devem ser separadas por `and` e colocadas entre parênteses;

### authors

As pessoas autoras do pacote.
Trata-se de um array de objetos.

Cada objeto de pessoa autora pode ter as seguintes propriedades:

* **name:** o nome da pessoa.
  Geralmente o nome verdadeiro.
* **email:** o endereço de e-mail da pessoa.
* **homepage:** um URL para o site da pessoa.
* **role:** a função da pessoa no projeto (por exemplo, desenvolvedora ou
  tradutora).

Um exemplo:

```json
{
    "authors": [
        {
            "name": "Nils Adermann",
            "email": "naderman@naderman.de",
            "homepage": "http://www.naderman.de",
            "role": "Developer"
        },
        {
            "name": "Jordi Boggiano",
            "email": "j.boggiano@seld.be",
            "homepage": "https://seld.be",
            "role": "Developer"
        }
    ]
}
```

Opcional, mas altamente recomendada.

### support

Várias informações para obter suporte para o projeto.

As informações de suporte incluem as seguintes:

* **email:** endereço de e-mail para suporte.
* **issues:** URL do sistema para acompanhamento de issues.
* **forum:** URL do fórum.
* **wiki:** URL da wiki.
* **irc:** canal IRC para suporte, como `irc://servidor/canal`.
* **source:** URL para pesquisar ou baixar o código-fonte.
* **docs:** URL da documentação.
* **rss:** URL para o feed RSS.
* **chat:** URL para o canal de chat.
* **security:** URL para a política de divulgação de vulnerabilidades (VDP).

Um exemplo:

```json
{
    "support": {
        "email": "suporte@example.org",
        "irc": "irc://irc.freenode.org/composer"
    }
}
```

Opcional.

### funding

Uma lista de URLs para fornecer financiamento às pessoas autoras do pacote para
manutenção e desenvolvimento de novas funcionalidades.

Cada entrada consiste no seguinte:

* **type:** o tipo de financiamento ou a plataforma através da qual o
  financiamento pode ser fornecido, por exemplo: patreon, opencollective,
  tidelift ou github.
* **url:** URL para um site com detalhes e uma forma de financiar o pacote.

Um exemplo:

```json
{
    "funding": [
        {
            "type": "patreon",
            "url": "https://www.patreon.com/phpdoctrine"
        },
        {
            "type": "tidelift",
            "url": "https://tidelift.com/subscription/pkg/packagist-doctrine_doctrine-bundle"
        },
        {
            "type": "other",
            "url": "https://www.doctrine-project.org/sponsorship.html"
        }
    ]
}
```

Opcional.

### Links de pacotes

Todos os itens a seguir recebem um objeto que mapeia nomes de pacotes para
versões do pacote por meio de restrições de versão.
Leia mais sobre versões [aqui](articles/versions.md).

Exemplo:

```json
{
    "require": {
        "monolog/monolog": "1.0.*"
    }
}
```

Todos os links são campos opcionais.

`require` e `require-dev` também oferecem suporte a _flags de estabilidade_
([root-only](#root-package)).
Elas assumem a forma "_restrição_@_flag de estabilidade_".
Elas permitem restringir ou expandir ainda mais a estabilidade de um pacote além
do escopo da configuração [minimum-stability](#minimum-stability).
Você pode aplicá-las a uma restrição ou aplicá-las a uma restrição vazia, se
desejar permitir pacotes instáveis de uma dependência, por exemplo.

Exemplo:

```json
{
    "require": {
        "monolog/monolog": "1.0.*@beta",
        "acme/foo": "@dev"
    }
}
```

Se uma de suas dependências depender de um pacote instável, você também
precisará requisitá-lo explicitamente, juntamente com a flag de estabilidade
necessária.

Exemplo:

Assumindo que `doctrine/doctrine-fixtures-bundle` requer
`"doctrine/data-fixtures": "dev-master"`, então dentro do `composer.json`raiz,
você precisará adicionar a segunda linha abaixo para permitir versões de
desenvolvimento do pacote `doctrine/data-fixtures`:

```json
{
    "require": {
        "doctrine/doctrine-fixtures-bundle": "dev-master",
        "doctrine/data-fixtures": "@dev"
    }
}
```

As seções `require` e `require-dev` também suportam referências explícitas (ou
seja, commits) para versões de desenvolvimento, garantindo que elas fiquem
fixadas em um estado específico, mesmo ao executar uma atualização.
Isso só funciona se você requisitar explicitamente uma versão de desenvolvimento
e acrescentar a referência usando `#<ref>`.
Esse recurso é exclusivo do [pacote raiz](#root-package) e será ignorado em
dependências.

Exemplo:

```json
{
    "require": {
        "monolog/monolog": "dev-master#2eb0c0978d290a1c45346a1955188929cb4e5db7",
        "acme/foo": "1.0.x-dev#abc123"
    }
}
```

> **Nota:** Este recurso possui limitações técnicas severas.
> A referência altera apenas de qual commit foi feito o check out; ela é
> aplicada no momento da instalação como uma substituição de baixo nível que o
> resolvedor de dependências nunca chega a ver.
> Como resultado:
>
> - Os metadados do `composer.json` do pacote (suas próprias entradas `require`,
>   regras de autoload, etc.) são lidos a partir do branch especificado, em seu
>   estado atual, e não do commit fixado.
>   Portanto, as dependências que o Composer resolve podem diferir daquelas
>   declaradas naquele commit.
> - A substituição é confiável apenas para instalações a partir do código-fonte.
>   Uma instalação de distribuição só consegue respeitar a referência quando o
>   pacote é obtido de uma fonte capaz de gerar um arquivo compactado para um
>   commit arbitrário (por exemplo, o GitHub); caso contrário, o commit fixado é
>   ignorado durante o download dos arquivos.
>
> Portanto, você deve usar isso apenas como uma solução temporária durante o
> desenvolvimento para contornar problemas passageiros, até que possa migrar
> para versões com tag.
> O time do Composer não oferece suporte ativo a este recurso e não aceitará
> relatórios de erros relacionados a ele.

Também é possível criar um alias em linha de uma restrição de pacote, para que
ela corresponda a uma restrição que de outra forma não corresponderia.
Para obter mais informações,
[consulte o artigo sobre aliases](articles/aliases.md).

`require` e `require-dev` também suportam referências a versões específicas do
PHP e de extensões PHP que seu projeto precisa para executar com sucesso.

Exemplo:

```json
{
    "require": {
        "php": ">=7.4",
        "ext-mbstring": "*"
    }
}
```

> **Nota:** É importante listar as extensões PHP que seu projeto requer.
> Nem todas as instalações PHP são criadas da mesma forma: algumas podem não
> possuir extensões que você pode considerar como padrão (como `ext-mysqli`, que
> não é instalada por padrão nas instalações mínimas dos sistemas
> Fedora/CentOS).
> Não listar as extensões PHP necessárias pode levar a uma experiência ruim da
> pessoa usuária: o Composer instalará seu pacote sem erros, mas ele falhará em
> tempo de execução.
> O comando `composer show --platform` lista todas as extensões PHP disponíveis
> no seu sistema.
> Você pode usá-lo para te ajudar a compilar a lista de extensões que você usa e
> precisa.
> Como alternativa, você pode usar ferramentas de terceiros para analisar seu
> projeto para obter a lista de extensões usadas.

#### require

Mapa de pacotes exigidos por este pacote.
O pacote não será instalado a menos que esses requisitos possam ser atendidos.

#### require-dev <span>([root-only](#root-package))</span>

Mapa de pacotes necessários para desenvolver este pacote, executar testes, etc.
As dependências de desenvolvimento do pacote raiz são instaladas por padrão.
Tanto o comando `install` quanto o `update` suportam a opção `--no-dev`, que
impede a instalação de dependências de desenvolvimento.

#### conflict

Mapa de pacotes incompatíveis com esta versão deste pacote.
A instalação deles em conjunto com o seu pacote não será permitida.

Observe que, ao especificar intervalos como `<1.0 >=1.1` em uma relação de
conflito (`conflict`), isso indicará um conflito com todas as versões que
sejam simultaneamente menores que 1.0 *e* iguais ou mais recentes que 1.1,
o que provavelmente não é o que você deseja.
Nesse caso, você provavelmente deveria usar `<1.0 || >=1.1`.

#### replace

Mapa dos pacotes substituídos por este pacote.
Isso permite que você crie um fork de um pacote, publique-o com um nome
diferente com seus próprios números de versão, enquanto os pacotes que exigem o
pacote original continuam funcionando com o seu fork, pois ele substitui o
pacote original.

Isso também é útil para pacotes que contêm subpacotes; por exemplo, o pacote
principal `symfony/symfony` contém todos os Componentes do Symfony que também
estão disponíveis como pacotes individuais.
Se você exigir o pacote principal, ele atenderá automaticamente a qualquer
requisito de um dos componentes individuais, já que os substitui.

Recomenda-se cuidado ao usar `replace` para a finalidade de subpacote explicada
acima.
Normalmente, você deve substituir apenas usando `self.version` como uma
restrição de versão, para garantir que o pacote principal substitua apenas os
subpacotes daquela versão específica, e não de qualquer outra versão, o que
seria incorreto.

#### provide

Mapa dos pacotes fornecidos por este pacote.
Isso é especialmente útil para implementações de interfaces comuns.
Um pacote pode depender de um pacote virtual, por exemplo,
`psr/log-implementation`.
Qualquer biblioteca que implemente essa interface de logger a listaria em
`provide`.
Os implementadores podem então ser
[encontrados no Packagist.org](https://packagist.org/providers/psr/log-implementation).

Usar `provide` com o nome de um pacote real em vez de um virtual implica que o
código desse pacote também é distribuído, caso em que `replace` geralmente é uma
escolha melhor.
Uma convenção comum para pacotes que fornecem uma interface e dependem de outros
pacotes para fornecer uma implementação (por exemplo, as interfaces PSR) é usar
o sufixo `-implementation` para o nome do pacote virtual correspondente ao
pacote da interface.

#### suggest

Pacotes sugeridos que podem aprimorar ou funcionar bem com este pacote.
Eles são informativos e são exibidos após a instalação do pacote, para dar às
pessoas uma dica de que elas podem adicionar mais pacotes, mesmo que não sejam
estritamente necessários.

O formato é semelhante aos links de pacotes acima, exceto que os valores são
texto livre e não restrições de versão.

Exemplo:

```json
{
    "suggest": {
        "monolog/monolog": "Permite o registro mais avançado de logging do fluxo da aplicação",
        "ext-xml": "Necessária para suportar o formato XML na classe Foo"
    }
}
```

### autoload

Mapeamento de autoloading para um autoloader PHP.

Os autoloadings [`PSR-4`][php-psr4] e [`PSR-0`][php-psr0], a geração de
`classmap` e a inclusão de `files` são suportados.

A PSR-4 é a maneira recomendada, pois oferece maior facilidade de uso (não há
necessidade de regenerar o autoloader ao adicionar classes).

#### PSR-4

Na chave `psr-4`, você define um mapeamento de namespaces para caminhos
relativos à raiz do pacote.
Ao fazer o autoloading de uma classe como `Foo\\Bar\\Baz`, um prefixo de
namespace `Foo\\` apontando para um diretório `src/` significa que o autoloader
procurará por um arquivo chamado `src/Bar/Baz.php` e o incluirá, se ele existir.
Observe que, ao contrário do antigo estilo PSR-0, o prefixo (`Foo\\`) **não**
está presente no caminho do arquivo.

Os prefixos de namespace devem terminar em `\\` para evitar conflitos entre
prefixos semelhantes.
Por exemplo, `Foo` corresponderia às classes no namespace `FooBar`, por isso as
barras invertidas finais resolvem o problema: `Foo\\` e `FooBar\\` são
distintos.

As referências PSR-4 são todas combinadas, durante a instalação/atualização, em
um único array associativo, que pode ser encontrado no arquivo
`vendor/composer/autoload_psr4.php` gerado.

Exemplo:

```json
{
    "autoload": {
        "psr-4": {
            "Monolog\\": "src/",
            "Vendor\\Namespace\\": ""
        }
    }
}
```

Se você precisar pesquisar o mesmo prefixo em vários diretórios, poderá
especificá-los como um array, assim:

```json
{
    "autoload": {
        "psr-4": { "Monolog\\": ["src/", "lib/"] }
    }
}
```

Se você quiser ter um diretório alternativo onde qualquer namespace será
procurado, use um prefixo vazio, como:

```json
{
    "autoload": {
        "psr-4": { "": "src/" }
    }
}
```

#### PSR-0

Na chave `psr-0`, você define um mapeamento de namespaces para caminhos
relativos à raiz do pacote.
Observe que ele também suporta a convenção sem namespaces do estilo PEAR.

Observe que as declarações de namespaces devem terminar em `\\` para garantir
que o autoloader responda corretamente.
Por exemplo, `Foo` corresponderia a `FooBar`, portanto, as barras invertidas
finais resolvem o problema: `Foo\\` e `FooBar\\` são distintos.

As referências PSR-0 são todas combinadas, durante a instalação/atualização, em
um único array associativo, que pode ser encontrado no arquivo
`vendor/composer/autoload_namespaces.php` gerado.

Exemplo:

```json
{
    "autoload": {
        "psr-0": {
            "Monolog\\": "src/",
            "Vendor\\Namespace\\": "src/",
            "Vendor_Namespace_": "src/"
        }
    }
}
```

Se você precisar pesquisar um mesmo prefixo em vários diretórios, poderá
especificá-los como um array, assim:

```json
{
    "autoload": {
        "psr-0": { "Monolog\\": ["src/", "lib/"] }
    }
}
```

O estilo PSR-0 não se limita apenas às declarações de namespace, mas pode ser
especificado até o nível da classe.
Isso pode ser útil para bibliotecas com apenas uma classe no namespace global.
Se o arquivo fonte PHP também estiver localizado na raiz do pacote, por exemplo,
ele pode ser declarado assim:

```json
{
    "autoload": {
        "psr-0": { "ClasseGlobalUnica": "" }
    }
}
```

Se você quiser ter um diretório alternativo onde qualquer namespace será
procurado, use um prefixo vazio, como:

```json
{
    "autoload": {
        "psr-0": { "": "src/" }
    }
}
```

#### Classmap

As referências em `classmap` são todas combinadas, durante a
instalação/atualização, em um único array associativo, que pode ser encontrado
no arquivo `vendor/composer/autoload_classmap.php` gerado.
Esse mapa é construído através da busca por classes em todos os arquivos `.php`
e `.inc` nos diretórios/arquivos especificados.

Você pode usar o suporte à geração de mapa de classes para definir o autoloading
para todas as bibliotecas que não seguem as PSR-0/4.
Para configurar isso, especifique todos os diretórios ou arquivos nos quais as
classes devem ser procuradas.

Exemplo:

```json
{
    "autoload": {
        "classmap": ["src/", "lib/", "AlgumaCoisa.php"]
    }
}
```

Caracteres curinga (`*`) também são suportados em caminhos de classmap e se
expandem para corresponder a qualquer nome de diretório:

Exemplo:

```json
{
    "autoload": {
        "classmap": ["src/addons/*/lib/", "3rd-party/*", "AlgumaCoisa.php"]
    }
}
```

#### Files

Se você quiser carregar explicitamente determinados arquivos em todas as
requisições, pode usar o mecanismo de autoloading `files`.
Ele é útil caso o seu pacote inclua funções PHP que não podem ser carregadas
automaticamente pelo PHP.

Exemplo:

```json
{
    "autoload": {
        "files": ["src/MinhaBiblioteca/funcoes.php"]
    }
}
```

As regras de autoloading de arquivos são incluídas sempre que
`vendor/autoload.php` é incluído, logo após o registro do autoloader.
A ordem de inclusão depende das dependências dos pacotes; assim, se o pacote A
depende do pacote B, os arquivos do pacote B serão incluídos primeiro para
garantir que o pacote B esteja totalmente inicializado e pronto para uso quando
os arquivos do pacote A forem incluídos.

Se dois pacotes tiverem a mesma quantidade de dependentes ou nenhuma
dependência, a ordem será alfabética.

Os arquivos do pacote raiz são sempre carregados por último, e você não pode
usar o autoloading de arquivos para sobrescrever funções de suas dependências.
Se quiser fazer isso, recomendamos incluir suas próprias funções *antes* de
incluir o arquivo `vendor/autoload.php` do Composer.

#### Excluir arquivos do mapa de classes

Se você deseja excluir alguns arquivos ou pastas do mapa de classes, use a
propriedade `exclude-from-classmap`.
Isso pode ser útil para excluir as classes de teste em seu ambiente ativo, por
exemplo, pois elas serão omitidas do mapa de classes, até mesmo ao criar um
autoloader otimizado.

O gerador de mapa de classes ignorará todos os arquivos nos caminhos
configurados aqui.
Os caminhos são absolutos no diretório raiz do pacote (ou seja, o local do
`composer.json`) e suportam `*` para corresponder a qualquer coisa, exceto uma
barra, e `**` para corresponder a qualquer coisa.
`**` é incluído implicitamente ao final dos caminhos.

Exemplo:

```json
{
    "autoload": {
        "exclude-from-classmap": ["/Tests/", "/test/", "/tests/"]
    }
}
```

#### Otimizando o autoloader

O autoloader pode ter um impacto bastante significativo no tempo de
processamento da requisição (50-100ms por requisição em grandes frameworks que
usam muitas classes).
Consulte o
[artigo sobre otimização do autoloader](articles/autoloader-optimization.md)
para mais detalhes sobre como reduzir esse impacto.

### autoload-dev <span>([root-only](#root-package))</span>

Esta seção permite definir regras de autoloading para fins de desenvolvimento.

Classes necessárias para executar a suíte de testes não devem ser incluídas nas
regras principais de autoloading, para evitar poluir o autoloader em produção e
quando outras pessoas usarem seu pacote como dependência.

Portanto, é recomendável usar um diretório dedicado para seus testes unitários
e adicioná-lo à seção `autoload-dev`.

Exemplo:

```json
{
    "autoload": {
        "psr-4": { "MinhaBiblioteca\\": "src/" }
    },
    "autoload-dev": {
        "psr-4": { "MinhaBiblioteca\\Tests\\": "tests/" }
    }
}
```

### include-path

> **OBSOLETA**: Esta propriedade está presente apenas para dar suporte a
> projetos legados, e todo código novo deve usar preferencialmente o
> autoloading.
> Como tal, é uma prática desaprovada, mas o recurso em si provavelmente não
> desaparecerá do Composer.

Uma lista de caminhos que devem ser anexados ao `include_path` do PHP.

Exemplo:

```json
{
    "include-path": ["lib/"]
}
```

Opcional.

### target-dir

> **OBSOLETA**: Esta propriedade está presente apenas para dar suporte ao
> autoloading no estilo PSR-0 legado, e todo código novo deve preferencialmente
> usar a PSR-4 sem `target-dir` e os projetos usando a PSR-0 com namespaces PHP
> são encorajados a migrar para a PSR-4.

Define o destino da instalação.

Caso a raiz do pacote esteja abaixo da declaração do namespace, você não poderá
fazer o autoloading corretamente.
`target-dir` resolve este problema.

Um exemplo é o Symfony.
Existem pacotes individuais para os componentes.
O componente Yaml está em `Symfony\Component\Yaml`.
A raiz do pacote é esse diretório `Yaml`.
Para tornar o autoloading possível, precisamos garantir que ele não esteja
instalado em `vendor/symfony/yaml`, mas sim em
`vendor/symfony/yaml/Symfony/Component/Yaml`, para que o autoloader possa
carregá-lo a partir de `vendor/symfony/yaml`.

Para fazer isso, `autoload` e `target-dir` são definidas da seguinte maneira:

```json
{
    "autoload": {
        "psr-0": { "Symfony\\Component\\Yaml\\": "" }
    },
    "target-dir": "Symfony/Component/Yaml"
}
```

Opcional.

### minimum-stability <span>([root-only](#root-package))</span>

Isso define o comportamento padrão para filtrar pacotes por estabilidade.
O padrão é `stable`, portanto, se você depender de um pacote `dev`,
especifique-o em seu arquivo para evitar surpresas.

Todas as versões de cada pacote são verificadas quanto à estabilidade, e aquelas
que são menos estáveis do que a configuração `minimum-stability` serão ignoradas
ao resolver as dependências do projeto.
(Observe que você também pode especificar requisitos de estabilidade para cada
pacote individualmente usando flags de estabilidade nas restrições de versão que
você especifica em um bloco `require` (consulte [package links](#package-links)
para obter mais detalhes).

As opções disponíveis (em ordem de estabilidade) são `dev`, `alpha`, `beta`,
`RC` e `stable`.

### prefer-stable <span>([root-only](#root-package))</span>

Quando esta opção está habilitada, o Composer dará preferência a pacotes mais
estáveis em relação aos instáveis sempre que for possível encontrar pacotes
estáveis compatíveis.
Se você precisar de uma versão de desenvolvimento ou se apenas versões alpha
estiverem disponíveis para um pacote, elas ainda serão selecionadas, desde que
`minimum-stability` permita.

Use `"prefer-stable": true` para habilitar.

### repositories <span>([root-only](#root-package))</span>

Repositórios de pacotes personalizados a serem usados.

Por padrão, o Composer usa apenas o repositório Packagist.
Ao especificar repositórios, você pode obter pacotes de outros locais.

Os repositórios não são resolvidos recursivamente.
Você pode adicioná-los apenas ao seu arquivo `composer.json` principal.
As declarações de repositórios dos arquivos `composer.json` das dependências são
ignoradas.

Os seguintes tipos de repositórios são suportados:

* **composer:** um repositório Composer é um arquivo `packages.json` servido via
  rede (HTTP, FTP, SSH), que contém uma lista de objetos `composer.json` com
  informações adicionais sobre `dist` e/ou `source`.
  O arquivo `packages.json` é carregado usando um stream PHP.
  Você pode definir opções extras para esse stream usando o parâmetro `options`.
* **vcs:** o repositório do sistema de controle de versão pode buscar pacotes de
  repositórios git, svn, fossil e hg.
* **package:** se você depende de um projeto que não tem nenhum suporte para o
  Composer, você pode definir o pacote em linha usando um repositório `package`.
  Basicamente você adiciona o objeto `composer.json` em linha.

Para obter mais informações sobre qualquer um deles, consulte
[Repositórios](05-repositories.md).

Exemplo:

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "http://packages.example.com"
        },
        {
            "type": "composer",
            "url": "https://packages.example.com",
            "options": {
                "ssl": {
                    "verify_peer": "true"
                }
            }
        },
        {
            "type": "vcs",
            "url": "https://github.com/Seldaek/monolog"
        },
        {
            "type": "package",
            "package": {
                "name": "smarty/smarty",
                "version": "3.1.7",
                "dist": {
                    "url": "https://www.smarty.net/files/Smarty-3.1.7.zip",
                    "type": "zip"
                },
                "source": {
                    "url": "https://smarty-php.googlecode.com/svn/",
                    "type": "svn",
                    "reference": "tags/Smarty_3_1_7/distribution/"
                }
            }
        }
    ]
}
```

> **Nota:** A ordem é importante aqui.
> Ao procurar um pacote, o Composer procurará do primeiro repositório ao último
> e escolherá a primeira correspondência.
> Por padrão, o Packagist é adicionado por último, o que significa que
> repositórios personalizados podem substituir pacotes dele.

Também é possível usar a notação de objeto JSON.
No entanto, pares de chave/valor JSON devem ser considerados ignorando a ordem,
portanto, um comportamento consistente não pode ser garantido e está obsoleto.

```json
{
    "repositories": {
        "foo": {
            "type": "composer",
            "url": "http://packages.foo.com"
        }
    }
}
```

Será substituído pela propriedade `name`:

```json
{
    "repositories": [
        {
            "name": "foo",
            "type": "composer",
            "url": "http://packages.foo.com"
        }
    ]
}
```

### config <span>([root-only](#root-package))</span>

Um conjunto de opções de configuração.
É usado apenas para projetos.
Consulte [Config](06-config.md) para obter uma descrição de cada opção
individual.

### scripts <span>([root-only](#root-package))</span>

O Composer permite que você interaja com várias partes do processo de instalação
por meio do uso de scripts.

Consulte [Scripts](articles/scripts.md) para obter detalhes e exemplos de
eventos.

### extra

Dados extras arbitrários para consumo por `scripts`.

Pode ser praticamente qualquer coisa.
Para acessá-los de um manipulador de eventos de script, você pode fazer:

```php
$extra = $event->getComposer()->getPackage()->getExtra();
```

Opcional.

### bin

Um conjunto de arquivos que devem ser tratados como binários e disponibilizados
no diretório `bin-dir` (da configuração).

Consulte [Binários dos fornecedores](articles/vendor-binaries.md) para obter
mais detalhes.

Opcional.

### archive

Um conjunto de opções para criar arquivos de pacotes.

As seguintes opções são suportadas:

* **name:** permite configurar o nome base para o arquivo.
  Por padrão (se não configurado e `--file` não for passado como argumento da
  linha de comando), `preg_replace('#[^a-z0-9-_]#i', '-', name)` é usado.

  Exemplo:

  ```json
  {
      "name": "org/nomeEstranho",
      "archive": {
          "name": "Nome_estranho"
      }
  }
  ```

* **exclude:** permite configurar uma lista de padrões para caminhos excluídos.
  A sintaxe do padrão corresponde aos arquivos `.gitignore`.
  Um ponto de exclamação (`!`) inicial fará com que quaisquer arquivos
  correspondentes sejam incluídos, mesmo que um padrão anterior os tenha
  excluído.
  Uma barra inicial corresponderá apenas no início do caminho relativo do
  projeto.
  Um asterisco não será expandido para um separador de diretório.

  Exemplo:

  ```json
  {
      "archive": {
          "exclude": ["/foo/bar", "baz", "/*.test", "!/foo/bar/baz"]
      }
  }
  ```

  O exemplo incluirá `/dir/foo/bar/arquivo`, `/foo/bar/baz`, `/arquivo.php`,
  `/foo/meu.test`, mas excluirá `/foo/bar/qualquer`, `/foo/baz` e `/meu.test`.

  Opcional.

### abandoned

Indica se este pacote foi abandonado.

Pode ser um valor booleano ou um nome/URL de um pacote que aponta para uma
alternativa recomendada.

Exemplos:

Use `"abandoned": true` para indicar que este pacote foi abandonado.
Use `"abandoned": "monolog/monolog"` para indicar que este pacote foi abandonado
e a alternativa recomendada é `monolog/monolog`.

O padrão é `false`.

Opcional.

### _comment

Chave de nível superior usada para armazenar comentários (pode ser uma string ou
um array de strings).

```json
{
    "_comment": [
        "O pacote foo/bar era necessário para a lógica de negócios",
        "Remova o pacote foo/baz ao remover foo/bar"
    ]
}
```

O padrão é vazio.

Opcional.

### non-feature-branches

Uma lista de padrões de expressões regulares de nomes de branches não numéricos
(por exemplo, "latest" ou algo semelhante), que NÃO serão tratados como feature
branches.
É um array de strings.

Se você tiver nomes de branches não numéricos, por exemplo, como "latest",
"current", "latest-stable" ou algo semelhante, que não se parecem com um número
de versão, o Composer os tratará como feature branches.
Isso significa que ele busca branches pai que se parecem com uma versão ou que
terminam em branches especiais (como `master`), e o número da versão do pacote
raiz se torna a versão do branch pai ou, pelo menos, `master` ou algo similar.

Para tratar branches com nomes não numéricos como versões, em vez de buscar um
branch pai com uma versão válida ou um nome de branch especial como `master`,
você pode definir padrões para nomes de branches que devem ser tratados como
branches de versões de desenvolvimento.

Isso é muito útil quando você tem dependências usando `self.version`, para não
ser `dev-master`, mas o mesmo branch seja instalado (no exemplo:
`latest-testing`).

Exemplo:

Se você tiver um branch `testing`, que recebe manutenção intensiva durante a
fase de testes e é implantado em seu ambiente de staging, normalmente
`composer show -s` retornará `versions : * dev-master`.

Se você configurar `latest-.*` como um padrão para `non-feature-branches`, como
neste exemplo:

```json
{
    "non-feature-branches": ["latest-.*"]
}
```

Então `composer show -s` retornará `versions : * dev-latest-testing`.

Opcional.

&larr; [Interface de linha de comando](03-cli.md) | [Repositórios](05-repositories.md) &rarr;
