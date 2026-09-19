---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/05-repositories.md
source_revision: 49d88f15080f3724a4eef066e3cf66dc9e15f116
translation_status: ready
---

# Repositórios

Este capítulo explicará o conceito de pacotes e repositórios, os tipos de
repositórios disponíveis e como eles funcionam.

## Conceitos

Antes de analisarmos os diferentes tipos de repositórios existentes, precisamos
entender alguns conceitos básicos sobre os quais o Composer foi construído.

### Pacote

O Composer é um gerenciador de dependências.
Ele instala pacotes localmente.
Um pacote é essencialmente um diretório contendo algo.
Neste caso, é código PHP, mas em teoria poderia ser qualquer coisa.
E contém uma descrição do pacote que possui um nome e uma versão.
O nome e a versão são usados para identificar o pacote.

Na verdade, internamente, o Composer vê cada versão como um pacote separado.
Embora essa distinção não importe quando você está usando o Composer, ela é
muito importante quando você quiser alterar o pacote.

Além do nome e da versão, existem metadados úteis.
A informação mais relevante para a instalação é a definição da fonte, que
descreve onde obter o conteúdo do pacote.
Os dados do pacote apontam para o conteúdo do pacote.
E existem duas opções aqui: `dist` e `source`.

**Dist:** é uma versão empacotada dos dados do pacote.
Geralmente é uma versão lançada, normalmente é uma versão estável.

**Source:** é usada para desenvolvimento.
Ela geralmente se origina de um repositório de código-fonte, como o git.
Você pode obter o código-fonte quando quiser modificar o pacote baixado.

Os pacotes podem fornecer uma dessas opções ou até mesmo as duas.
Dependendo de certos fatores, como as opções fornecidas pela pessoa e a
estabilidade do pacote, uma delas terá a preferência.

### Repositório

Um repositório é uma fonte de pacotes.
É uma lista de pacotes/versões.
O Composer buscará em todos os seus repositórios os pacotes que seu projeto
requer.

Por padrão, apenas o repositório Packagist.org está registrado no Composer.
Você pode adicionar mais repositórios ao seu projeto declarando-os no
`composer.json`.

Os repositórios estão disponíveis apenas para o pacote raiz e os repositórios
definidos em suas dependências não serão carregados.
Leia a
[FAQ](faqs/why-cant-composer-load-repositories-recursively.md) se quiser saber o
porquê.
Ao resolver dependências, os pacotes são procurados nos repositórios de cima
para baixo e, por padrão, assim que um pacote é encontrado em um repositório, o
Composer para de procurar em outros.
Leia o artigo sobre
[prioridades de repositórios](articles/repository-priorities.md) para obter mais
detalhes e saber como alterar esse comportamento.

## Tipos

### composer

O principal tipo de repositório é o repositório `composer`.
Ele usa um único arquivo `packages.json` que contém todos os metadados dos
pacotes.

Este também é o tipo de repositório usado pelo Packagist.
Para referenciar um repositório `composer`, forneça o caminho antes do arquivo
`packages.json`.
No caso do Packagist, esse arquivo está localizado em `/packages.json`,
portanto, a URL do repositório seria `repo.packagist.org`.
Para `example.org/packages.json`, a URL do repositório seria `example.org`.

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://example.org"
        }
    ]
}
```

#### packages

O único campo obrigatório é `packages`.
A estrutura JSON é a seguinte:

```json
{
    "packages": {
        "vendor/nome-do-pacote": {
            "dev-master": { @composer.json },
            "1.0.x-dev": { @composer.json },
            "0.0.1": { @composer.json },
            "1.0.0": { @composer.json }
        }
    }
}
```

O marcador `@composer.json` seria o conteúdo do arquivo `composer.json` daquela
versão do pacote, incluindo, no mínimo:

* `name`
* `version`
* `dist` ou `source`

Aqui está uma definição mínima de pacote:

```json
{
    "name": "smarty/smarty",
    "version": "3.1.7",
    "dist": {
        "url": "https://www.smarty.net/files/Smarty-3.1.7.zip",
        "type": "zip"
    }
}
```

Pode incluir qualquer um dos outros campos especificados no
[esquema](04-schema.md).

#### notify-batch

O campo `notify-batch` permite especificar uma URL que será chamada sempre que
uma pessoa usuária instalar um pacote.
A URL pode ser um caminho absoluto (que usará o mesmo domínio do repositório) ou
uma URL totalmente qualificada.

Um exemplo de valor:

```json
{
    "notify-batch": "/downloads/"
}
```

Para o arquivo `example.org/packages.json` que contém o pacote
`monolog/monolog`, isso enviaria uma requisição `POST` para
`example.org/downloads/` com o seguinte corpo da requisição JSON:

```json
{
    "downloads": [
        {"name": "monolog/monolog", "version": "1.2.1.0"}
    ]
}
```

O campo `version` conterá a representação normalizada do número da versão.

Este campo é opcional.

#### metadata-url, available-packages e available-package-patterns

O campo `metadata-url` permite que você forneça um modelo de URL para servir
todos os pacotes presentes no repositório.
Ele deve conter o marcador `%package%`.

Este campo é novo no Composer v2 e tem prioridade sobre os campos
`provider-includes` e `providers-url` se ambos estiverem presentes.

Para compatibilidade com o Composer v1 e v2, o ideal é fornecer ambos.
Novas implementações de repositório podem precisar suportar apenas a versão 2.

Um exemplo:

```json
{
    "metadata-url": "/p2/%package%.json"
}
```

Sempre que o Composer procurar um pacote, ele substituirá `%package%` pelo nome
do pacote e buscará essa URL.
Se a estabilidade `dev` for permitida para o pacote, ele também carregará a URL
novamente com `$packageName~dev` (por exemplo, `/p2/foo/bar~dev.json` para
procurar as versões de desenvolvimento de `foo/bar`).

Os arquivos `foo/bar.json` e `foo/bar~dev.json` que contêm as versões do pacote
DEVEM conter apenas as versões do pacote foo/bar, como
`{"packages":{"foo/bar":[ ... versões aqui ... ]}}`.

O cache é feito por meio do cabeçalho If-Modified-Since, portanto, certifique-se
de retornar os cabeçalhos Last-Modified e de que eles estejam corretos.

A lista de versões também pode ser opcionalmente minificada usando
`Composer\MetadataMinifier\MetadataMinifier::minify()` do
[composer/metadata-minifier](https://packagist.org/packages/composer/metadata-minifier).

Se você fizer isso, adicione a chave `"minified": "composer/2.0"` no nível
superior para indicar ao Composer que ele deve expandir a lista de versões de
volta aos dados originais.
Consulte https://repo.packagist.org/p2/monolog/monolog.json para um exemplo.

Qualquer pacote requisitado que não exista DEVE retornar um código de status
404, o que indicará ao Composer que este pacote não existe em seu repositório.
Certifique-se de que a resposta 404 seja rápida para evitar o bloqueio do
Composer.

Evite redirecionamentos para páginas 404 alternativas.

Se o seu repositório tiver apenas um pequeno número de pacotes e você quiser
evitar requisições 404, você também pode especificar uma chave
`"available-packages"` no arquivo `packages.json`, que deve ser um array com
todos os nomes de pacotes que seu repositório contém.
Como alternativa, você pode especificar uma chave `available-package-patterns`,
que é um array de padrões de nomes de pacotes (com `*` correspondendo a qualquer
string, por exemplo, `vendor/*` faria com que o Composer pesquisasse todos os
nomes de pacotes correspondentes neste repositório).

Este campo é opcional.

#### providers-api

O campo `providers-api` permite que você forneça um modelo de URL para exibir
todos os pacotes que fornecem um determinado nome de pacote, mas não o pacote
que possui esse nome, mesmo que ele exista.
Ele deve conter o marcador `%package%`.

Por exemplo, https://packagist.org/providers/psr/log-implementation.json lista
alguns pacotes que possuem uma regra `provide` para `psr/log-implementation`.

```json
{
    "providers-api": "https://packagist.org/providers/%package%.json",
}
```

Este campo é opcional.

### list

O campo `list` permite que você retorne os nomes dos pacotes que correspondem a
um filtro específico (ou todos os nomes se nenhum filtro for aplicado).
Deve aceitar um parâmetro de consulta opcional `?filter=xx`, que pode conter `*`
como curingas que correspondam a qualquer substring.

Regras `replace`/`provide` não devem ser consideradas aqui.

Deve retornar um array de nomes de pacotes:

```json
{
    "packageNames": [
        "a/b",
        "c/d"
    ]
}
```

Consulte
[https://packagist.org/packages/list.json?filter=composer/*](https://packagist.org/packages/list.json?filter=composer/*)
como exemplo.

Este campo é opcional.

#### provider-includes e providers-url

O campo `provider-includes` permite listar um conjunto de arquivos que listam
nomes de pacotes fornecidos por este repositório.
O hash deve ser um sha256 dos arquivos neste caso.

O campo `providers-url` descreve como os arquivos do provedor são encontrados no
servidor.
É um caminho absoluto a partir da raiz do repositório.
Deve conter os placeholders: `%package%` e `%hash%`.

Esses campos são usados pelo Composer v1 ou caso seu repositório não tenha o
campo `metadata-url` definido.

Exemplo:

```json
{
    "provider-includes": {
        "providers-a.json": {
            "sha256": "f5b4bc0b354108ef08614e569c1ed01a2782e67641744864a74e788982886f4c"
        },
        "providers-b.json": {
            "sha256": "b38372163fac0573053536f5b8ef11b86f804ea8b016d239e706191203f6efac"
        }
    },
    "providers-url": "/p/%package%$%hash%.json"
}
```

Esses arquivos contêm listas de nomes de pacotes e hashes para verificar a
integridade do arquivo, por exemplo:

```json
{
    "providers": {
        "acme/foo": {
            "sha256": "38968de1305c2e17f4de33aea164515bc787c42c7e2d6e25948539a14268bb82"
        },
        "acme/bar": {
            "sha256": "4dd24c930bd6e1103251306d6336ac813b563a220d9ca14f4743c032fb047233"
        }
    }
}
```

O arquivo acima declara que `acme/foo` e `acme/bar` podem ser encontrados neste
repositório, carregando o arquivo referenciado por `providers-url`, substituindo
`%package%` pelo nome do pacote com namespace do fornecedor e `%hash%` pelo
campo `sha256`.
Esses arquivos contêm definições de pacotes conforme descrito
[acima](#pacotes).

Esses campos são opcionais.
Provavelmente você não precisará deles para seu próprio repositório
personalizado.

#### Opções de cURL ou stream

O repositório é acessado usando cURL (Composer 2 com ext-curl habilitado) ou
streams PHP.
Você pode definir opções extras usando o parâmetro `options`.
Para streams PHP, você pode definir qualquer opção de contexto de stream PHP
válida.
Consulte
[Opções e parâmetros de contexto](https://php.net/manual/pt_BR/context.php) para
obter mais informações.
Quando o cURL é usado, apenas um conjunto limitado de opções `http` e `ssl` pode
ser configurado.

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://example.org",
            "options": {
                "http": {
                    "timeout": 60
                }
            }
        }
    ],
    "require": {
        "acme/package": "^1.0"
    }
}
```

#### filter

Um repositório Composer pode anunciar suporte a listas de filtros para clientes
incluindo um objeto `filter` em sua resposta `packages.json`.
Isso informa ao Composer que o repositório fornece dados de listas de filtros e
descreve quais listas estão disponíveis.

```json
{
    "metadata-url": "/p2/%package%.json",
    "filter": {
        "metadata": true,
        "lists": {
            "malware": { "enabled": true },
            "typosquatting": { "enabled": true }
        },
        "summary-url": "/p2/filter-summary.json"
    }
}
```

- **`metadata`** (obrigatório, booleano): defina como `true` para indicar que os
  arquivos de metadados por pacote (servidos via `metadata-url`) contêm dados de
  listas de filtros.
  O Composer buscará os metadados para cada pacote relevante e procurará uma
  chave `filter` contendo entradas de lista.
- **`lists`** (obrigatório, objeto): os nomes de todas as listas de filtros
  fornecidas por este repositório, mapeados para um objeto que descreve a lista.
  O objeto atualmente possui uma flag `enabled` (definida como `true` quando a
  lista está disponível) e está estruturado desta forma para que metadados por
  lista possam ser adicionados posteriormente sem quebrar o formato de
  transmissão.
- **`summary-url`** (opcional, string): uma URL (absoluta ou relativa à raiz)
  que retorna um mapeamento compacto de nome da lista → nome do pacote →
  restrição de versão.
  Quando configurado, o Composer busca este endpoint durante `composer install`,
  `composer update` e `composer audit` e o usa para ignorar buscas de metadados
  por pacote para pacotes que não correspondem a nenhuma lista ativa.

  O endpoint deve retornar JSON no formato:

  ```json
  {
      "filter": {
          "malware": {
              "fornecedor/pacote": ">=1.0.0,<1.2.0",
              "outro/pacote": "*"
          }
      }
  }
  ```

  O Composer revalida o resumo em cache por meio de `If-Modified-Since` a cada
  `install`/`update`/`audit`, usando o mesmo cache de repositório em disco
  que os metadados do pacote.
- **`api-url`** (opcional, string): uma URL (absoluta ou relativa à raiz) que
  aceita uma requisição POST com as PURLs relevantes do pacote e os nomes de
  lista configurados, e responde com as entradas de filtro correspondentes
  diretamente.
  Quando definido, o Composer usa `api-url` em vez de `summary-url` e metadados
  por pacote para fins de filtragem; isso é útil quando o resumo seria muito
  grande para ser exibido na íntegra.
  Se `summary-url` e `api-url` forem anunciados, `api-url` terá precedência e
  `summary-url` será ignorado.
  As pessoas implementadoras devem estar cientes de que grandes quantidades
  (algumas centenas seriam normais) de nomes de pacotes podem ser enviadas pelo
  Composer.
  Você pode optar por limitar essa lista e retornar apenas os primeiros 1000 ou
  quantos itens você decidir, mas certifique-se de limitar isso de alguma forma.

  O endpoint recebe um corpo JSON no seguinte formato:

  ```json
  {
      "packages": ["pkg://composer/fornecedor/pacote", "pkg://composer/outro/pacote"],
      "lists": ["malware"]
  }
  ```

  e deve retornar um JSON no seguinte formato:

  ```json
  {
      "filter": {
          "malware": [
              {
                  "package": "fornecedor/pacote",
                  "constraint": ">=1.0.0,<1.2.0",
                  "url": "https://example.org/filters/123",
                  "reason": "Malware",
                  "id": "PKFE-xxxx-xxxx-xxxx"
              }
          ]
      }
  }
  ```

  Cada entrada tem o mesmo formato que uma entrada de filtro de metadados por
  pacote, com a adição do campo `package` (já que uma resposta abrange vários
  pacotes).
  O POST `api-url` não é armazenado em cache no lado do cliente porque cada
  corpo de requisição é diferente.

Os arquivos de metadados por pacote devem incluir uma chave `filter` cujo valor
é um objeto que mapeia nomes de listas para arrays de entradas de filtro:

```json
{
    "packages": {
        "fornecedor/pacote": [{ ... }]
    },
    "filter": {
        "malware": [
            {
                "constraint": ">=1.0.0,<1.2.0",
                "url": "https://example.org/filters/123",
                "reason": "Malware",
                "id": "PKFE-xxxx-xxxx-xxxx"
            }
        ]
    }
}
```

Em um arquivo `composer.json`, a chave `filter` em uma definição de repositório
controla quais listas anunciadas por esse repositório são consideradas para
relatórios de auditoria e bloqueio de versão.

Defina `filter: false` para optar por não participar de todas as listas
anunciadas por este repositório:

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://example.org",
            "filter": false
        }
    ]
}
```

Defina `filter` como um objeto listando os nomes das listas anunciadas que devem
ser ignoradas neste repositório.

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://example.org",
            "filter": {
                "untrusted-list": false
            }
        }
    ]
}
```

O filtro de nível de repositório apenas restringe as contribuições deste
repositório; ele não pode habilitar uma lista que não esteja configurada
globalmente em [`config.policy`](06-config.md#policy).

### VCS

VCS significa sistema de controle de versão.
Isso inclui sistemas de versionamento como git, svn, fossil ou hg.
O Composer possui um tipo de repositório para instalar pacotes desses sistemas.

#### Carregando um pacote de um repositório VCS

Existem alguns casos de uso para isso.
O mais comum é manter seu próprio fork de uma biblioteca de terceiros.
Se você estiver usando uma determinada biblioteca para o seu projeto e decidir
alterar algo nela, você vai querer que seu projeto use a versão alterada.
Se a biblioteca estiver no GitHub (o que acontece na maioria das vezes), você
pode criar um fork e enviar suas alterações para o seu fork.
Depois disso, você atualiza o `composer.json` do projeto.
Tudo o que você precisa fazer é adicionar seu fork como um repositório e
atualizar a restrição de versão para apontar para seu branch personalizado.
Somente no `composer.json`, você deve prefixar o nome do seu branch
personalizado com `"dev-"` (sem incluí-lo no nome do branch).
Para convenções de nomenclatura de restrições de versão, consulte
[Bibliotecas](02-libraries.md) para obter mais informações.

Exemplo, supondo que você tenha aplicado um patch no monolog para corrigir uma
falha no branch `bugfix`:

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "https://github.com/igorw/monolog"
        }
    ],
    "require": {
        "monolog/monolog": "dev-bugfix"
    }
}
```

Ao executar `php composer.phar update`, você deverá obter sua versão modificada
do pacote `monolog/monolog` em vez da versão do Packagist.

Observe que você não deve renomear o pacote a menos que realmente pretenda criar
um fork a longo prazo e abandonar completamente o pacote original.
O Composer escolherá corretamente seu pacote em vez do original, já que o
repositório personalizado tem prioridade sobre o Packagist.
Se você quiser renomear o pacote, faça isso no branch padrão (geralmente
`master`) e não em um feature branch, pois o nome do pacote é obtido do branch
padrão.

Observe também que a substituição não funcionará se você alterar a propriedade
`name` no arquivo `composer.json` do seu repositório onde foi criado o fork,
pois ela precisa corresponder ao original para que a substituição funcione.

Se outras dependências dependerem do pacote que você criou o fork, é possível
criar um alias em linha para que ele corresponda a uma restrição que, de outra
forma, não corresponderia.
Para mais informações [consulte o artigo sobre aliases](articles/aliases.md).

#### Usando repositórios privados

A mesma solução permite que você trabalhe com seus repositórios privados no
GitHub e no Bitbucket:

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url":  "git@bitbucket.org:vendor/my-private-repo.git"
        }
    ],
    "require": {
        "vendor/my-private-repo": "dev-master"
    }
}
```

O único requisito é a instalação de chaves SSH para um cliente Git.

#### Alternativas ao Git

O Git não é o único sistema de controle de versão suportado pelo repositório
VCS.
Os seguintes são suportados:

* **Git:** [git-scm.com](https://git-scm.com)
* **Subversion:** [subversion.apache.org](https://subversion.apache.org)
* **Mercurial:** [mercurial-scm.org](https://www.mercurial-scm.org)
* **Fossil**: [fossil-scm.org](https://www.fossil-scm.org/)

Para obter pacotes desses sistemas, você precisa ter seus respectivos clientes
instalados.
Isso pode ser inconveniente.
Por isso, há suporte especial para GitHub e Bitbucket, que usa as APIs
fornecidas por esses sites para buscar os pacotes sem precisar instalar o
sistema de controle de versão.
O repositório VCS fornece arquivos `dist` para eles, que baixam os pacotes como
arquivos zip.

* **GitHub:** [github.com](https://github.com) (Git)
* **Bitbucket:** [bitbucket.org](https://bitbucket.org) (Git)

O driver VCS a ser usado é detectado automaticamente com base na URL.
No entanto, caso precise especificar um por algum motivo, você pode usar
`bitbucket`, `github`, `gitlab`, `perforce`, `fossil`, `git`, `svn` ou `hg` como
o tipo de repositório em vez de `vcs`.

Se você definir a chave `no-api` como `true` em um repositório do GitHub, ele
clonará o repositório como faria com qualquer outro repositório Git, em vez de
usar a API do GitHub.
Mas, diferentemente do uso direto do driver `git`, o Composer ainda tentará usar
os arquivos zip do GitHub.

Note que:

* **Para permitir que o Composer escolha qual driver usar**, o tipo de
  repositório precisa ser definido como `vcs`.
* **Se você já usou um repositório privado**, isso significa que o Composer já
  deve tê-lo clonado em cache.
  Se você quiser instalar o mesmo pacote com drivers, lembre-se de executar o
  comando `composer clearcache` seguido do comando `composer update` para
  atualizar o cache do Composer e instalar o pacote a partir de `dist`.
* O driver VCS `git-bitbucket` está obsoleto e foi substituído por `bitbucket`.

#### Configuração do driver do BitBucket

> **Observe que o endpoint do repositório para o Bitbucket precisa ser https em
> vez de git.**

Após configurar seu repositório Bitbucket, você também precisará
[configurar a autenticação](articles/authentication-for-private-packages.md#bitbucket-oauth).

#### Opções do Subversion

Como o Subversion não possui um conceito nativo de branches e tags, o Composer
assume por padrão que o código está localizado em `$url/trunk`, `$url/branches`
e `$url/tags`.
Se o seu repositório tiver um layout diferente, você pode alterar esses valores.
Por exemplo, se você usa nomes com a primeira letra maiúscula, pode configurar o
repositório assim:

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "http://svn.example.org/projeto-a/",
            "trunk-path": "Trunk",
            "branches-path": "Branches",
            "tags-path": "Tags"
        }
    ]
}
```

Se você não tiver um diretório `branches` ou `tags`, poderá desabilitá-los
completamente definindo `branches-path` ou `tags-path` como `false`.

Se o pacote estiver em um subdiretório, por exemplo,
`/trunk/foo/bar/composer.json` e `/tags/1.0/foo/bar/composer.json`, você poderá
fazer com que o Composer acesse-o definindo a opção `"package-path"` para o
subdiretório; neste exemplo, seria `"package-path": "foo/bar/"`.

Se você tiver um repositório Subversion privado, poderá salvar as credenciais na
seção `http-basic` da sua configuração (consulte
[Config](06-config.md#http-basic)):

```json
{
    "http-basic": {
        "svn.example.org": {
            "username": "usuário",
            "password": "senha"
        }
    }
}
```

Se o seu cliente Subversion estiver configurado para armazenar credenciais por
padrão, essas credenciais serão salvas para o usuário atual e as credenciais
salvas existentes para este servidor serão sobrescritas.
Para alterar esse comportamento, defina a opção `"svn-cache-credentials"` na
configuração do seu repositório:

```json
{
    "repositories": [
        {
            "type": "vcs",
            "url": "http://svn.example.org/projeto-a/",
            "svn-cache-credentials": false
        }
    ]
}
```

### Package

Se você quiser usar um projeto que não oferece suporte ao Composer por nenhum
dos métodos acima, ainda é possível definir o pacote você mesmo usando um
repositório `package`.

Basicamente, você define as mesmas informações que estão incluídas no
`packages.json` do repositório `composer`, mas apenas para um único pacote.
Novamente, os campos mínimos obrigatórios são `name`, `version` e `dist` ou
`source`.

Aqui está um exemplo para o mecanismo de templates Smarty:

```json
{
    "repositories": [
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
                    "url": "http://smarty-php.googlecode.com/svn/",
                    "type": "svn",
                    "reference": "tags/Smarty_3_1_7/distribution/"
                },
                "autoload": {
                    "classmap": ["libs/"]
                }
            }
        }
    ],
    "require": {
        "smarty/smarty": "3.1.*"
    }
}
```

Normalmente, você omitiria a parte `source`, pois ela não é realmente
necessária.

Se uma chave `source` for incluída, o campo `reference` deve ser uma referência
à versão que será instalada.
Quando o campo `type` for `git`, este será o ID do commit, o nome do branch ou
da tag.

> **Nota**: Não é recomendável usar o nome de um branch do Git para o campo
> `reference`.
> Embora isso seja válido, já que é suportado pelo `git checkout`, os nomes de
> branch são mutáveis e, portanto, não podem ser travados.

Quando o campo `type` for `svn`, o campo `reference` deve conter a referência
anexada à URL ao executar `svn co`.

> **Nota**: Esse tipo de repositório possui algumas limitações e deve ser
> evitado sempre que possível:
>
> - O Composer não atualizará o pacote, a menos que você altere o campo
>   `version`.
> - O Composer não atualizará as referências dos commits, portanto, se você usar
>   `master` como referência, terá que excluir o pacote para forçar uma
>   atualização e terá que lidar com um arquivo de lock instável.

A chave `package` em um repositório `package` pode ser definida como um array
para definir várias versões de um pacote:

```json
{
    "repositories": [
        {
            "type": "package",
            "package": [
                {
                    "name": "foo/bar",
                    "version": "1.0.0",
                    ...
                },
                {
                    "name": "foo/bar",
                    "version": "2.0.0",
                    ...
                }
            ]
        }
    ]
}
```

## Hospedando seu próprio repositório

Embora você provavelmente queira colocar seus pacotes no Packagist na maioria
das vezes, existem alguns casos de uso para hospedar seu próprio repositório.

* **Pacotes de empresas privadas:** se você faz parte de uma empresa que usa o
  Composer para seus pacotes internamente, talvez queira manter esses pacotes
  privados.

* **Ecossistema separado:** se você tem um projeto com seu próprio ecossistema,
  e os pacotes não são facilmente reutilizáveis pela comunidade PHP em geral,
  talvez queira mantê-los separados do Packagist.
  Um exemplo disso seriam plugins do WordPress.

Para hospedar seus próprios pacotes, um repositório nativo do tipo `composer` é
recomendado, pois oferece o melhor desempenho.

Existem algumas ferramentas que podem ajudá-lo a criar um repositório
`composer`.

### Private Packagist

O [Private Packagist](https://packagist.com/) é uma aplicação hospedada ou
auto-hospedada que oferece hospedagem privada de pacotes, bem como espelhamento
do GitHub, Packagist.org e outros repositórios de pacotes.

Confira [Packagist.com](https://packagist.com/) para obter mais informações.

### Satis

O Satis é um gerador de repositórios `composer` estáticos.
É como uma versão ultraleve e baseada em arquivos estáticos do Packagist.

Você fornece um arquivo `composer.json` contendo repositórios, normalmente
definições de VCS e repositórios de pacotes.
Ele buscará todos os pacotes que são `require` e gerará um arquivo
`packages.json` que é o seu repositório `composer`.

Consulte [o repositório GitHub do Satis](https://github.com/composer/satis) e o
[artigo sobre como lidar com pacotes privados](articles/handling-private-packages.md)
para obter mais informações.

### Artifact

Existem casos em que não é possível ter um dos tipos de repositório mencionados
anteriormente online, nem mesmo o VCS.
Um exemplo típico seria a troca de bibliotecas entre organizações por meio de
artefatos de compilação.
Obviamente, na maioria das vezes, esses artefatos são privados.
Para usar esses arquivos como estão, pode-se usar um repositório do tipo
`artifact` com uma pasta contendo arquivos ZIP ou TAR desses pacotes privados:

```json
{
    "repositories": [
        {
            "type": "artifact",
            "url": "caminho/para/o/diretorio/com/zips/"
        }
    ],
    "require": {
        "fornecedor-privado-um/core": "15.6.2",
        "fornecedor-privado-dois/connectivity": "*",
        "acme-corp/parser": "10.3.5"
    }
}
```

Cada artefato zip é um arquivo ZIP com o arquivo `composer.json` no diretório
raiz:

```shell
unzip -l acme-corp-parser-10.3.5.zip
```

```text
composer.json
...
```

Se houver dois arquivos com versões diferentes de um pacote, ambos serão
importados.
Quando um arquivo com uma versão mais recente for adicionado à pasta de
artefatos, e você executar `update`, essa versão também será importada e o
Composer atualizará para a versão mais recente.

### Path

Além do repositório `artifact`, você pode usar o repositório `path`, que
permite depender de um diretório local, seja absoluto ou relativo.
Isso pode ser especialmente útil ao lidar com repositórios monolíticos.

Por exemplo, se você tiver a seguinte estrutura de diretórios em seu
repositório:

```text
...
├── apps
│   └── minha-aplicacao
│       └── composer.json
├── pacotes
│   └── meu-pacote
│       └── composer.json
...
```

Em seguida, para adicionar o pacote `meu/pacote` como uma dependência, no seu
arquivo `apps/minha-aplicacao/composer.json`, você pode usar a seguinte
configuração:

```json
{
    "repositories": [
        {
            "type": "path",
            "url": "../../pacotes/meu-pacote"
        }
    ],
    "require": {
        "meu/pacote": "*"
    }
}
```

Se o pacote for um repositório VCS local, a versão pode ser inferida pelo branch
ou tag que está atualmente em uso.
Caso contrário, a versão deve ser definida explicitamente no arquivo
`composer.json` do pacote.
Se a versão não puder ser resolvida por esses meios, assume-se que seja
`dev-master`.

Quando a versão não puder ser inferida do repositório VCS local, ou quando você
quiser sobrescrever a versão, você pode usar a opção `versions` ao declarar
o repositório:

```json
{
    "repositories": [
        {
            "type": "path",
            "url": "../../pacotes/meu-pacote",
            "options": {
                "versions": {
                    "meu/pacote": "4.2-dev"
                }
            }
        }
    ]
}
```

O pacote local será vinculado simbolicamente, se possível.
Nesse caso, a saída no console exibirá
`Symlinking from ../../pacotes/meu-pacote`.
Se a criação de link simbólico _não_ for possível, o pacote será copiado.
Nesse caso, o console exibirá `Mirrored from ../../pacotes/meu-pacote`.

Em vez da estratégia de fallback padrão, você pode forçar o uso de links
simbólicos com a opção `"symlink": true` ou espelhamento com a opção
`"symlink": false`.
Forçar o espelhamento pode ser útil ao implantar ou gerar pacotes a partir de um
repositório monolítico.

> **Nota:** No Windows, os links simbólicos de diretório são implementados
> usando junções NTFS, pois podem ser criados por usuários sem privilégios de
> administrador.
> O espelhamento sempre será usado em versões anteriores ao Windows 7 ou se
> `proc_open` estiver desabilitado.

```json
{
    "repositories": [
        {
            "type": "path",
            "url": "../../pacotes/*",
            "options": {
                "symlink": false
            }
        }
    ]
}
```

Os tils iniciais são expandidos para a pasta inicial do usuário atual e as
variáveis de ambiente são analisadas nas notações do Windows e do Linux/Mac.
Por exemplo, `~/git/meu-pacote` carregará automaticamente o clone de
`meu-pacote` de `/home/<usuário>/git/meu-pacote`, equivalente a
`$HOME/git/meu-pacote` ou `%USERPROFILE%/git/meu-pacote`.

> **Nota:** Os caminhos do repositório também podem conter curingas como `*` e
> `?`.
> Para obter detalhes, consulte a
> [função glob do PHP](https://www.php.net/manual/pt_BR/function.glob.php).

Você pode configurar como a referência `dist` do pacote (que aparece no arquivo
`composer.lock`) é construída.

Existem os seguintes modos:
- `none`: a referência será sempre nula.
  Isso pode ajudar a reduzir conflitos no arquivo de lock mas reduz a clareza
  sobre quando ocorreu a última atualização e se o pacote está no estado mais
  recente.
- `config`: a referência é construída com base em um hash do `composer.json` do
  pacote e da configuração do repositório.
- `auto` (usado por padrão): a referência é construída com base no hash, como em
  `config`, mas se a pasta do pacote contiver um repositório git, o hash do
  commit HEAD será usado como referência.

```json
{
    "repositories": [
        {
            "type": "path",
            "url": "../../pacotes/*",
            "options": {
                "reference": "config"
            }
        }
    ]
}
```

## Desabilitando o Packagist.org

Você pode desabilitar o repositório padrão do Packagist.org adicionando o
seguinte ao seu `composer.json`:

```json
{
    "repositories": [
        {
            "packagist.org": false
        }
    ]
}
```

Você pode desabilitar o Packagist.org globalmente usando a flag de configuração
global:

```bash
composer config -g repo.packagist false
```

&larr; [Esquema](04-schema.md) | [Config](06-config.md) &rarr;
