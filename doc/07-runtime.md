---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/07-runtime.md
source_revision: 31c7474cde1abe5bb5ea5bc9623c399797ba2f8e
translation_status: ready
---

# Utilitários do tempo de execução do Composer

Embora o Composer seja usado principalmente para instalar as dependências do
seu projeto, existem alguns recursos disponibilizados para uso em tempo de
execução.

Caso precise usar algum deles em uma versão específica, você pode declarar o
pacote `composer-runtime-api` como dependência.

## Autoloading

O autoloader é o recurso mais usado e já foi abordado em nosso
[guia de uso básico](01-basic-usage.md#autoloading).
Ele está disponível em todas as versões do Composer.

## Versões instaladas

O `composer-runtime-api` `2.0` introduziu uma nova classe
`Composer\InstalledVersions` que oferece alguns métodos estáticos para verificar
quais versões estão instaladas atualmente.
Isso fica disponível automaticamente para o seu código, desde que você inclua o
autoloader do Composer.

Os principais casos de uso para essa classe são os seguintes:

### Saber se o pacote X (ou pacote virtual) está presente

```php
\Composer\InstalledVersions::isInstalled('fornecedor/pacote'); // retorna booleano
\Composer\InstalledVersions::isInstalled('psr/log-implementation'); // retorna booleano
```

A partir do Composer 2.1, você também pode verificar se algo foi instalado via
`require-dev` ou não, passando `false` como segundo argumento:

```php
\Composer\InstalledVersions::isInstalled('fornecedor/pacote'); // retorna true, assumindo que este pacote esteja instalado
\Composer\InstalledVersions::isInstalled('fornecedor/pacote', false); // retorna true se fornecedor/pacote estiver em require, false se estiver em require-dev
```

Observe que isso não pode ser usado para verificar se pacotes de plataforma
estão instalados.

### Saber se o pacote X está instalado na versão Y

> **Nota:** Para usar isso, seu pacote deve exigir `"composer/semver": "^3.0"`.

```php
use Composer\Semver\VersionParser;

\Composer\InstalledVersions::satisfies(new VersionParser, 'fornecedor/pacote', '2.0.*');
\Composer\InstalledVersions::satisfies(new VersionParser, 'psr/log-implementation', '^1.0');
```

Isso retornará `true` se, por exemplo, `fornecedor/pacote` estiver instalado em
uma versão que corresponda a `2.0.*`, mas também se o nome do pacote fornecido
for substituído ou fornecido por algum outro pacote.

### Saber qual a versão do pacote X

> **Nota:** Isso retornará `null` se o nome do pacote requisitado não estiver
> instalado diretamente, mas apenas for fornecido ou substituído por outro
> pacote.
> Portanto, recomendamos usar `satisfies()` pelo menos em códigos de
> bibliotecas.
> Em códigos de aplicações, você tem um pouco mais de controle e isso é menos
> importante.

```php
// retorna uma versão normalizada (ex.: 1.2.3.0) se fornecedor/pacote estiver instalado,
// ou null se ele for fornecido/substituído,
// ou lança uma OutOfBoundsException se o pacote não estiver instalado de forma alguma
\Composer\InstalledVersions::getVersion('fornecedor/pacote');
```

```php
// retorna a versão original (ex.: v1.2.3) se fornecedor/pacote estiver instalado,
// ou null se ele for fornecido/substituído,
// ou lança uma OutOfBoundsException se o pacote não estiver instalado de forma alguma
\Composer\InstalledVersions::getPrettyVersion('fornecedor/pacote');
```

```php
// retorna a referência de distribuição ou fonte do pacote (ex.: um hash de commit do git) se fornecedor/pacote estiver instalado,
// ou null se ele for fornecido/substituído,
// ou lança uma OutOfBoundsException se o pacote não estiver instalado de forma alguma
\Composer\InstalledVersions::getReference('fornecedor/pacote');
```

### Saber qual a versão instalada do próprio pacote

Se você quiser apenas obter a versão do próprio pacote, por exemplo, no
código-fonte de `acme/foo`, você quer saber qual versão de `acme/foo` está sendo
executada atualmente para exibi-la à pessoa usuária, então é aceitável usar
`getVersion`, `getPrettyVersion` ou `getReference`.

O alerta da seção anterior não se aplica neste caso, pois, se o seu código está
sendo executado, você tem certeza de que o pacote está presente e não está sendo
substituído.

Ainda assim, é uma boa ideia garantir que você trate o valor de retorno `null`
da maneira mais elegante possível, por segurança.

----

Alguns outros métodos estão disponíveis para usos mais complexos; consulte o
código-fonte ou os docblocks da
[própria classe](https://github.com/composer/composer/blob/main/src/Composer/InstalledVersions.php).

### Saber qual o caminho em que um pacote está instalado

O método `getInstallPath` recupera o caminho absoluto de instalação de um
pacote.

> **Nota:** Embora absoluto, o caminho pode conter `../` ou links simbólicos.
> Não há garantia de que ele seja equivalente a um `realpath()`; portanto, você
> deve aplicar `realpath()` a ele caso isso seja importante para o seu caso de
> uso.

```php
// retorna o caminho absoluto para o local de instalação do pacote se fornecedor/pacote estiver instalado,
// ou null se ele for fornecido/substituído ou se o pacote for um metapacote,
// ou lança uma OutOfBoundsException se o pacote não estiver instalado
\Composer\InstalledVersions::getInstallPath('fornecedor/pacote');
```

> Disponível a partir do Composer 2.1 (ou seja, `composer-runtime-api ^2.1`).

### Saber quais pacotes de um determinado tipo estão instalados

O método `getInstalledPackagesByType` aceita um tipo de pacote (por exemplo,
`foo-plugin`) e lista os pacotes desse tipo que estão instalados.
Você pode então usar os métodos acima para obter mais informações sobre cada
pacote, se necessário.

Esse método deve eliminar a necessidade de instaladores personalizados que
colocam plugins em um caminho específico em vez de deixá-los no diretório
`vendor`.
Assim, você pode localizar plugins para inicialização em tempo de execução via
`InstalledVersions`, incluindo seus caminhos por meio de `getInstallPath`, se
preciso.

```php
\Composer\InstalledVersions::getInstalledPackagesByType('foo-plugin');
```

> Disponível a partir do Composer 2.1 (ou seja, `composer-runtime-api ^2.1`).

## Verificação de plataforma

A `composer-runtime-api` `2.0` introduziu um novo arquivo
`vendor/composer/platform_check.php`, que é incluído automaticamente quando você
inclui o autoloader do Composer.

Ele verifica se os requisitos de plataforma (ou seja, PHP e extensões do PHP)
são atendidos pelo processo PHP em execução no momento.
Se os requisitos não forem atendidos, o script exibe um alerta com os requisitos
ausentes e encerra a execução com o código `104`.

Para evitar uma inesperada "tela branca da morte" com algum alerta obscuro de
extensão PHP em produção, você pode executar `composer check-platform-reqs` como
parte do seu processo de implantação/construção; se isso retornar um código
diferente de zero, você deve abortar.

O valor padrão é `php-only`, que verifica apenas a versão do PHP.

Se, por algum motivo, você não quiser usar essa verificação de segurança e
preferir arriscar erros de tempo de execução quando seu código for executado,
você pode desabilitá-la definindo a opção de configuração
[`platform-check`](06-config.md#platform-check) como `false`.

Se você quiser que a verificação inclua a presença de extensões do PHP, defina a
opção de configuração como `true`.
Os requisitos `ext-*` serão então verificados, mas, por motivos de desempenho, o
Composer verifica apenas se a extensão está presente, e não a sua versão exata.

Requisitos `lib-*` nunca são suportados/verificados pelo recurso de verificação
de plataforma.

## Caminho do autoloader em binários

A `composer-runtime-api` `2.2` introduziu uma nova variável global
`$_composer_autoload_path`, definida ao executar binários instalados com o
Composer.
Leia mais sobre isso
[na documentação de binários de fornecedor](articles/vendor-binaries.md#localizando-o-autoloader-do-composer-a-partir-de-um-binário).

Ela é definida pelo proxy do binário e, portanto, não é disponibilizada aos
projetos pelo arquivo `vendor/autoload.php` do Composer, o que seria inútil,
pois apontaria de volta para si mesma.

## Caminho dos binários (`bin-dir`) em binários

A `composer-runtime-api` `2.2.2` introduziu uma nova variável global
`$_composer_bin_dir`, definida ao executar binários instalados com o Composer.
Leia mais sobre isso
[na documentação de binários de fornecedor](articles/vendor-binaries.md#localizando-o-diretório-bin-dir-do-composer-a-partir-de-um-binário).

Isso é definido pelo proxy de binário e, portanto, não é disponibilizado aos
projetos pelo arquivo `vendor/autoload.php` do Composer.

&larr; [Config](06-config.md) | [Comunidade](08-community.md) &rarr;
