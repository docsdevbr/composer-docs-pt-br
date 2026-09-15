---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/custom-installers.md
source_revision: d76813b309c2ad2e992f4f72dc697017c374f1be
translation_status: ready

tagline: Modifique a maneira como certos tipos de pacotes são instalados.
---

# Configurando e utilizando instaladores personalizados

## Visão geral

Às vezes, pode ser necessário que um pacote exija ações adicionais durante a
instalação, como instalar pacotes fora da biblioteca `vendor` padrão.

Nesses casos, você pode considerar a criação de um instalador personalizado para
lidar com sua lógica específica.

## Alternativa a instaladores personalizados com o Composer 2.1+

A partir do Composer 2.1, a classe `Composer\InstalledVersions` conta com o
método
[`getInstalledPackagesByType`](../07-runtime.md#saber-quais-pacotes-de-um-determinado-tipo-estão-instalados),
que permite identificar, em tempo de execução, quais plugins, módulos ou
extensões estão instalados.

Se você estiver desenvolvendo uma nova aplicação, recomenda-se fortemente
utilizar essa abordagem em vez de criar novos instaladores personalizados.
Isso oferece a vantagem de manter todo o código de terceiros no diretório
`vendor`, dispensando a necessidade de código para instaladores personalizados.

## Chamando um instalador personalizado

Suponha que seu projeto já possua um instalador personalizado para módulos
específicos; nesse caso, invocar esse instalador é uma questão de definir o
[tipo][1] correto no arquivo do seu pacote.

> _Consulte o próximo capítulo para obter instruções sobre como criar
> instaladores personalizados._

Todo instalador personalizado define qual string de [tipo][1] ele reconhecerá.
Uma vez reconhecido, ele substituirá completamente o instalador padrão e
aplicará apenas a sua própria lógica.

Um exemplo de caso de uso seria:

> O phpDocumentor possui Templates que precisam ser instalados fora da estrutura
> de pastas padrão `/vendor`.
> Por isso, optaram por adotar o [tipo][1] `phpdocumentor-template` e criar um
> plugin que fornece o instalador personalizado para enviar esses templates para
> a pasta correta.

Um exemplo de `composer.json` para tal pacote de template seria:

```json
{
    "name": "phpdocumentor/template-responsive",
    "type": "phpdocumentor-template",
    "require": {
        "phpdocumentor/template-installer-plugin": "*"
    }
}
```

> **IMPORTANTE**: para garantir que o instalador de template esteja presente no
> momento em que o pacote de template for instalado, os pacotes de template
> devem declarar o pacote do plugin como dependência.

## Criando um instalador

Um instalador personalizado é definido como uma classe que implementa a
interface [`Composer\Installer\InstallerInterface`][4] e é geralmente
distribuído em um plugin do Composer.

Um plugin de instalador básico seria composto, portanto, por três arquivos:

1. O arquivo do pacote: `composer.json`.
2. A classe do plugin, ex.: `My\Project\Composer\Plugin.php`, contendo uma
   classe que implementa `Composer\Plugin\PluginInterface`.
3. A classe do instalador, ex.: `My\Project\Composer\Installer.php`, contendo
   uma classe que implementa `Composer\Installer\InstallerInterface`.

### composer.json

O arquivo do pacote é igual a qualquer outro arquivo de pacote, mas com os
seguintes requisitos:

1. O atributo [type][1] deve ser `composer-plugin`.
2. O atributo [extra][2] deve conter um elemento `class` definindo o nome da
   classe do plugin (incluindo o namespace).
   Se um pacote contiver múltiplos plugins, este pode ser um array de nomes de
   classes.

Exemplo:

```json
{
    "name": "phpdocumentor/template-installer-plugin",
    "type": "composer-plugin",
    "license": "MIT",
    "autoload": {
        "psr-0": {"phpDocumentor\\Composer": "src/"}
    },
    "extra": {
        "class": "phpDocumentor\\Composer\\TemplateInstallerPlugin"
    },
    "require": {
        "composer-plugin-api": "^1.0"
    },
    "require-dev": {
        "composer/composer": "^1.3"
    }
}
```

O exemplo acima inclui o próprio Composer em seu `require-dev`, o que permite
utilizar as classes do Composer em sua suíte de testes, por exemplo.

### A classe do plugin

A classe que define o plugin do Composer deve implementar a interface
[`Composer\Plugin\PluginInterface`][3].
Ela pode, então, registrar o instalador personalizado em seu método
`activate()`.

A classe pode estar localizada em qualquer lugar e ter qualquer nome, desde que
seja acessível via autoload e corresponda ao elemento `extra.class` na
definição do pacote.

Exemplo:

```php
<?php

namespace phpDocumentor\Composer;

use Composer\Composer;
use Composer\IO\IOInterface;
use Composer\Plugin\PluginInterface;

class TemplateInstallerPlugin implements PluginInterface
{
    public function activate(Composer $composer, IOInterface $io)
    {
        $installer = new TemplateInstaller($io, $composer);
        $composer->getInstallationManager()->addInstaller($installer);
    }
}
```

### A classe do instalador personalizado

A classe que executa a instalação personalizada deve implementar a interface
[`Composer\Installer\InstallerInterface`][4] (ou estender outro instalador que
implemente essa interface).
Ela define, no método `supports()`, a string de [tipo][1] pela qual será
reconhecida pelos pacotes que utilizarão este instalador.

> **NOTA**: _escolha o nome do seu [tipo][1] com cuidado; recomenda-se seguir
> o formato: `vendor-type`_.
> Por exemplo: `phpdocumentor-template`.

A interface `InstallerInterface` define os seguintes métodos (consulte o
código-fonte para ver a assinatura exata):

* **supports()**, aqui você verifica se o [tipo][1] fornecido corresponde ao
  nome que você declarou para este instalador (veja o exemplo).
* **isInstalled()**, determina se um pacote suportado está instalado ou não.
* **install()**, aqui você pode determinar as ações que precisam ser executadas
  durante a instalação.
* **update()**, aqui você define o comportamento necessário quando o Composer é
  invocado com o argumento `update`.
* **uninstall()**, aqui você pode determinar as ações que precisam ser
  executadas quando o pacote precisar ser removido.
* **getInstallPath()**, este método deve retornar o caminho absoluto onde o
  pacote será instalado.
  O caminho _não deve terminar com uma barra._

Exemplo:

```php
<?php

namespace phpDocumentor\Composer;

use Composer\Package\PackageInterface;
use Composer\Installer\LibraryInstaller;

class TemplateInstaller extends LibraryInstaller
{
    /**
     * @inheritDoc
     */
    public function getInstallPath(PackageInterface $package)
    {
        $prefix = substr($package->getPrettyName(), 0, 23);
        if ('phpdocumentor/template-' !== $prefix) {
            throw new \InvalidArgumentException(
                'Unable to install template, phpdocumentor templates '
                .'should always start their package name with '
                .'"phpdocumentor/template-"'
            );
        }

        return 'data/templates/'.substr($package->getPrettyName(), 23);
    }

    /**
     * @inheritDoc
     */
    public function supports($packageType)
    {
        return 'phpdocumentor-template' === $packageType;
    }
}
```

O exemplo demonstra que é possível estender a classe
[`Composer\Installer\LibraryInstaller`][5] para remover um prefixo
(`phpdocumentor/template-`) e utilizar a parte restante para montar um caminho
de instalação completamente diferente.

> _Em vez de ser instalado em `/vendor`, qualquer pacote instalado utilizando
> este instalador será colocado na pasta `/data/templates/<nome_sem_prefixo>`._

[1]: ../04-schema.md#type
[2]: ../04-schema.md#extra
[3]: https://github.com/composer/composer/blob/main/src/Composer/Plugin/PluginInterface.php
[4]: https://github.com/composer/composer/blob/main/src/Composer/Installer/InstallerInterface.php
[5]: https://github.com/composer/composer/blob/main/src/Composer/Installer/LibraryInstaller.php
