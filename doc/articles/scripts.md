---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/scripts.md
source_revision: fee2383ed5bb0dfd95315997cd789fa8d37f2f47
translation_status: ready

tagline: >-
  Scripts são funções de retorno invocadas antes ou depois da instalação de
  pacotes.
---

# Scripts

## O que é um script?

Um script, na terminologia do Composer, pode ser uma função de retorno PHP
(definida como um método estático) ou qualquer comando executável via linha de
comando.
Scripts são úteis para executar código personalizado de um pacote ou comandos
específicos do pacote durante o processo de execução do Composer.

A partir do Composer 2.5, scripts também podem ser classes de comando do Symfony
Console, o que permite executá-los facilmente, inclusive passando opções.
No entanto, isso não é recomendado para lidar com eventos.

> **Nota:** Apenas scripts definidos no arquivo `composer.json` do pacote raiz
> são executados.
> Se uma dependência do pacote raiz especificar seus próprios scripts, o
> Composer não executará esses scripts adicionais.

## Nomes de eventos

O Composer dispara os seguintes eventos nomeados durante seu processo de
execução:

### Eventos de comando

- **pre-install-cmd**: ocorre antes da execução do comando `install`, quando um
  arquivo de bloqueio está presente.
- **post-install-cmd**: ocorre após a execução do comando `install`, quando um
  arquivo de bloqueio está presente.
- **pre-update-cmd**: ocorre antes da execução do comando `update`, ou antes da
  execução do comando `install` quando não há um arquivo de bloqueio presente.
- **post-update-cmd**: ocorre após a execução do comando `update`, ou após a
  execução do comando `install` quando não há um arquivo de bloqueio presente.
- **pre-status-cmd**: ocorre antes da execução do comando `status`.
- **post-status-cmd**: ocorre após a execução do comando `status`.
- **pre-archive-cmd**: ocorre antes da execução do comando `archive`.
- **post-archive-cmd**: ocorre após a execução do comando `archive`.
- **pre-autoload-dump**: ocorre antes da geração do autoloader, seja durante
  `install`/`update` ou via comando `dump-autoload`.
- **post-autoload-dump**: ocorre após a geração do autoloader, seja durante
  `install`/`update` ou via comando `dump-autoload`.
- **post-root-package-install**: ocorre após a instalação do pacote raiz
  durante o comando `create-project` (mas antes da instalação de suas
  dependências).
- **post-create-project-cmd**: ocorre após a execução do comando
  `create-project`.

### Eventos do instalador

- **pre-operations-exec**: ocorre antes da execução das operações de
  instalação/atualização/etc. ao instalar um arquivo de bloqueio.
  Plugins que precisem interagir com este evento devem ser instalados
  globalmente para poderem ser utilizados; caso contrário, não seriam carregados
  a tempo durante uma instalação inicial do projeto.

### Eventos de pacote

- **pre-package-install**: ocorre antes da instalação de um pacote.
- **post-package-install**: ocorre após a instalação de um pacote.
- **pre-package-update**: ocorre antes da atualização de um pacote.
- **post-package-update**: ocorre após a atualização de um pacote.
- **pre-package-uninstall**: ocorre antes da desinstalação de um pacote.
- **post-package-uninstall**: ocorre após a desinstalação de um pacote.

### Eventos de plugin

- **init**: ocorre após a conclusão da inicialização de uma instância do
  Composer.
- **command**: ocorre antes da execução de qualquer comando do Composer na CLI.
  Ele fornece acesso aos objetos de entrada e saída do programa.
- **pre-file-download**: ocorre antes do download de arquivos e permite
  manipular o objeto `HttpDownloader` antes de baixar os arquivos, com base na
  URL que será baixada.
- **post-file-download**: ocorre após o download dos arquivos de distribuição do
  pacote e permite realizar verificações adicionais no arquivo, se necessário.
- **pre-command-run**: ocorre antes da execução de um comando e permite
  manipular as opções e argumentos do objeto `InputInterface` para ajustar o
  comportamento do comando.
- **pre-pool-create**: ocorre antes da criação do Pool de pacotes e permite
  filtrar a lista de pacotes que será processada pelo Solver.

> **Nota:** O Composer não faz suposições sobre o estado das suas dependências
> antes da execução de `install` ou `update`.
> Portanto, você não deve especificar scripts que exijam dependências
> gerenciadas pelo Composer nos hooks de evento `pre-update-cmd` ou
> `pre-install-cmd`.
> Se precisar executar scripts antes de `install` ou `update`, certifique-se de
> que eles sejam autossuficientes dentro do seu pacote raiz.

## Definindo scripts

O objeto JSON raiz no arquivo `composer.json` deve conter uma propriedade
chamada `"scripts"`, que contém pares de nomes de eventos e os scripts
correspondentes a cada evento.
Os scripts de um evento podem ser definidos como uma string (apenas para um
único script) ou como um array (para um ou múltiplos scripts).

Para qualquer evento:

- Os scripts são executados na ordem definida quando o evento correspondente é
  disparado.
- Um array de scripts associado a um único evento pode conter tanto funções de
  retorno PHP quanto comandos executáveis via linha de comando.
- Classes PHP e comandos que contenham funções de retorno definidas devem ser
  carregáveis automaticamente por meio da funcionalidade de autoload do
  Composer.
- Funções de retorno só podem carregar automaticamente classes a partir de
  definições PSR-0, PSR-4 e mapa de classes.
  Se uma função de retorno definida depender de funções definidas fora de uma
  classe, a própria função de retorno é responsável por carregar o arquivo que
  contém essas funções.

Exemplo de definição de script:

```json
{
    "scripts": {
        "post-update-cmd": "MyVendor\\MyClass::postUpdate",
        "post-package-install": [
            "MyVendor\\MyClass::postPackageInstall"
        ],
        "post-install-cmd": [
            "MyVendor\\MyClass::warmCache",
            "phpunit -c app/"
        ],
        "post-autoload-dump": [
            "MyVendor\\MyClass::postAutoloadDump"
        ],
        "post-create-project-cmd": [
            "php -r \"copy('config/local-example.php', 'config/local.php');\""
        ]
    }
}
```

Usando o exemplo de definição anterior, aqui está a classe `MyVendor\MyClass`
que pode ser usada para executar as funções de retorno PHP:

```php
<?php

namespace MyVendor;

use Composer\Script\Event;
use Composer\Installer\PackageEvent;

class MyClass
{
    public static function postUpdate(Event $event)
    {
        $composer = $event->getComposer();
        // do stuff
    }

    public static function postAutoloadDump(Event $event)
    {
        $vendorDir = $event->getComposer()->getConfig()->get('vendor-dir');
        require $vendorDir . '/autoload.php';

        some_function_from_an_autoloaded_file();
    }

    public static function postPackageInstall(PackageEvent $event)
    {
        $installedPackage = $event->getOperation()->getPackage();
        // do stuff
    }

    public static function warmCache(Event $event)
    {
        // make cache toasty
    }
}
```

> **Nota:** Durante a execução de um comando `install` ou `update` do Composer,
> uma variável chamada `COMPOSER_DEV_MODE` será adicionada ao ambiente.
> Se o comando tiver sido executado com a flag `--no-dev`, essa variável será
> definida como 0; caso contrário, será definida como 1.
> A variável também fica disponível durante a execução do `dump-autoload`,
> mantendo o mesmo valor definido na última execução de `install` ou `update`.

## Classes de eventos

Quando um evento é disparado, sua função de retorno PHP recebe, como primeiro
argumento, um objeto `Composer\EventDispatcher\Event`.
Esse objeto possui um método `getName()` que permite obter o nome do evento.

Dependendo dos [tipos de script](#nomes-de-eventos), você obterá várias
subclasses de evento contendo diversos métodos getter com dados relevantes e
objetos associados:

- Classe base:
  [`Composer\EventDispatcher\Event`](https://github.com/composer/composer/blob/main/src/Composer/EventDispatcher/Event.php)
- Eventos de comando:
  [`Composer\Script\Event`](https://github.com/composer/composer/blob/main/src/Composer/Script/Event.php)
- Eventos do instalador:
  [`Composer\Installer\InstallerEvent`](https://github.com/composer/composer/blob/main/src/Composer/Installer/InstallerEvent.php)
- Eventos de pacote:
  [`Composer\Installer\PackageEvent`](https://github.com/composer/composer/blob/main/src/Composer/Installer/PackageEvent.php)
- Eventos de plugin:
  - init:
    [`Composer\EventDispatcher\Event`](https://github.com/composer/composer/blob/main/src/Composer/EventDispatcher/Event.php)
  - command:
    [`Composer\Plugin\CommandEvent`](https://github.com/composer/composer/blob/main/src/Composer/Plugin/CommandEvent.php)
  - pre-file-download:
    [`Composer\Plugin\PreFileDownloadEvent`](https://github.com/composer/composer/blob/main/src/Composer/Plugin/PreFileDownloadEvent.php)
  - post-file-download:
    [`Composer\Plugin\PostFileDownloadEvent`](https://github.com/composer/composer/blob/main/src/Composer/Plugin/PostFileDownloadEvent.php)

## Executando scripts manualmente

Se você quiser executar manualmente os scripts de um evento, a sintaxe é:

```shell
php composer.phar run-script [--dev] [--no-dev] script
```

Por exemplo, `composer run-script post-install-cmd` executará quaisquer scripts
e [plugins](plugins.md) definidos para o evento **post-install-cmd**.

Você também pode fornecer argumentos adicionais ao manipulador do script
adicionando `--` seguido pelos argumentos do manipulador.
Por exemplo: `composer run-script post-install-cmd -- --check` passará `--check`
para o manipulador do script.
Esses argumentos são recebidos como argumentos de CLI por manipuladores de CLI
e podem ser recuperados como um array via `$event->getArguments()` por
manipuladores PHP.

## Criando comandos personalizados

Se você adicionar scripts personalizados que não se enquadram em nenhum dos
nomes de eventos predefinidos acima, você pode executá-los com `run-script` ou
como comandos nativos do Composer.
Por exemplo, o manipulador definido abaixo pode ser executado chamando
`composer test`:

```json
{
    "scripts": {
        "test": "phpunit",
        "do-something": "MyVendor\\MyClass::doSomething",
        "my-cmd": "MyVendor\\MyCommand"
    }
}
```

Assim como no comando `run-script`, você pode passar argumentos adicionais para
os scripts; por exemplo, `composer test -- --filter <pattern>` repassará
`--filter <pattern>` para o script `phpunit`.

Invocar um método PHP via `composer do-something arg` permite executar uma
`static function doSomething(\Composer\Script\Event $event)`, tornando o
argumento `arg` disponível em `$event->getArguments()`.
No entanto, isso não facilita a passagem de opções personalizadas na forma de
`--flags`.

Ao utilizar uma classe `Command` do
[symfony/console](https://packagist.org/packages/symfony/console), você pode
descrever seu script e definir ou acessar argumentos e opções com mais
facilidade.

Por exemplo, com o comando abaixo, você pode simplesmente chamar
`composer my-cmd --arbitrary-flag` sem sequer precisar do separador `--`.
Para serem detectadas como comandos do symfony/console, as classes devem
terminar com o sufixo `Command` e estender a classe `Command` do Symfony.
Observe também que a execução ocorrerá utilizando a versão do symfony/console
integrada ao Composer; essa versão pode diferir daquela definida em seu projeto
e pode mudar entre versões menores do Composer.
Se precisar de mais garantias de estabilidade, é preferível utilizar seu próprio
arquivo binário para executar sua versão específica do symfony/console de forma
isolada, em seu próprio processo.

Os nomes e descrições de scripts definidos em uma classe `Command` substituirão
os detalhes presentes no `composer.json`: a chave da entrada em `scripts`
(utilizada como o comando passado ao `run-script`) será substituída por
`$defaultName` ou pelo valor definido em `setName()`; uma substituição
semelhante ocorrerá com qualquer informação incluída em `scripts-descriptions`
para essa classe de script.

```php
<?php

namespace MyVendor;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Input\InputOption;
use Symfony\Component\Console\Output\OutputInterface;

class MyCommand extends Command
{
    protected function configure(): void
    {
        $this
//          ->setName('custom-cmd') //if this gets included, it would execute with `composer custom-cmd` instead
            ->setDescription('Custom description for this command')
            ->setDefinition([
                new InputOption('arbitrary-flag', null, InputOption::VALUE_NONE, 'Example flag'),
                new InputArgument('foo', InputArgument::OPTIONAL, 'Optional arg'),
            ])
            ->setHelp(
                "Here you can define a long description for your command\n".
                "This would be visible with composer my-cmd --help"
            );
    }

    public function execute(InputInterface $input, OutputInterface $output): int
    {
        if ($input->getOption('arbitrary-flag')) {
            $output->writeln('The flag was used');
        }

        return 0;
    }
}
```

> **Nota:** Antes de executar scripts, o diretório de binários (`bin-dir`) do
> Composer é temporariamente adicionado ao início da variável de ambiente
> `PATH`, de modo que os binários das dependências fiquem diretamente
> acessíveis.
> Neste exemplo, independentemente de o binário `phpunit` estar localizado em `vendor/bin/phpunit` ou em `bin/phpunit`, ele será encontrado e executado.

## Gerenciando o tempo limite do processo

Embora o Composer não tenha como objetivo gerenciar processos de longa duração
ou outros aspectos semelhantes de projetos PHP, às vezes pode ser útil desativar
o tempo limite do processo em comandos personalizados.
Esse tempo limite é de 300 segundos por padrão e pode ser alterado de várias
maneiras, dependendo do efeito desejado:

- Desabilite-o para todos os comandos usando a chave de configuração
  `process-timeout`.
- Desabilite-o para invocações atuais ou futuras do Composer usando a variável
  de ambiente `COMPOSER_PROCESS_TIMEOUT`.
- Desabilite-o para uma invocação específica, usando a flag `--timeout` do
  comando `run-script`.
- Desabilite-o usando um método auxiliar estático para scripts específicos.

Para desabilitar o tempo limite de scripts específicos usando o método auxiliar
estático diretamente no `composer.json`:

```json
{
    "scripts": {
        "test": [
            "Composer\\Config::disableProcessTimeout",
            "phpunit"
        ]
    }
}
```

Para desativar o tempo limite para todos os scripts de um determinado projeto,
você pode usar a configuração do `composer.json`:

```json
{
    "config": {
        "process-timeout": 0
    }
}
```

Também é possível definir a variável de ambiente global para desativar o tempo
limite de todos os scripts subsequentes no ambiente do terminal atual:

```shell
export COMPOSER_PROCESS_TIMEOUT=0
```

Para desativar o tempo limite de uma chamada de script específica, você deve
usar o comando `run-script` do Composer e especificar o parâmetro `--timeout`:

```shell
php composer.phar run-script --timeout=0 test
```

## Referenciando scripts

Para permitir a reutilização de scripts e evitar duplicatas, você pode chamar um
script a partir de outro prefixando o nome do comando com `@`:

```json
{
    "scripts": {
        "test": [
            "@clearCache",
            "phpunit"
        ],
        "clearCache": "rm -rf cache/*"
    }
}
```

Você também pode referenciar um script e passar novos argumentos para ele:

```json
{
    "scripts": {
        "tests": "phpunit",
        "testsVerbose": "@tests -vvv"
    }
}
```

## Chamando comandos do Composer

Para chamar comandos do Composer, você pode usar `@composer`, que será resolvido automaticamente para o `composer.phar` que estiver sendo utilizado no momento:

```json
{
    "scripts": {
        "test": [
            "@composer install",
            "phpunit"
        ]
    }
}
```

Uma limitação disso é que você não pode executar vários comandos do Composer em
sequência, como `@composer install && @composer foo`.
Você deve separá-los em um array JSON de comandos.

## Executando scripts PHP

Para executar scripts PHP, você pode usar `@php`, que será automaticamente
resolvido para o processo PHP que estiver sendo utilizado no momento:

```json
{
    "scripts": {
        "test": [
            "@php script.php",
            "phpunit"
        ]
    }
}
```

Uma limitação disso é que você não pode chamar vários comandos em sequência,
como `@php install && @php foo`.
Você deve separá-los em um array JSON de comandos.

Você também pode chamar um script shell/bash, que terá o caminho para o
executável do PHP disponível como uma variável de ambiente `PHP_BINARY`.

## Controlando argumentos adicionais

A partir do Composer 2.8, você pode controlar como argumentos adicionais são
passados para comandos de script.

Ao executar scripts como `composer script-name arg arg2` ou
`composer script-name -- --option`, o Composer, por padrão, anexará `arg`,
`arg2` e `--option` ao comando do script.

Se você não quiser esses argumentos em um determinado comando, pode incluir
`@no_additional_args` em qualquer parte dele; isso desativa o comportamento
padrão e a própria flag será removida antes da execução do comando.

Se você quiser que os argumentos sejam adicionados em outro lugar que não seja o
final, pode usar `@additional_args` para escolher exatamente onde eles devem
ficar.

Por exemplo, ao executar `composer run-commands ARG` com a configuração abaixo:

```json
{
    "scripts": {
        "run-commands": [
            "echo hello @no_additional_args",
            "command-with-args @additional_args && do-something-without-args --here"
        ]
    }
}
```

Acabaria executando estes comandos:

```
echo hello
command-with-args ARG && do-something-without-args --here
```

## Definindo variáveis de ambiente

Para definir uma variável de ambiente de forma multiplataforma, você pode usar
`@putenv`:

```json
{
    "scripts": {
        "install-phpstan": [
            "@putenv COMPOSER=phpstan-composer.json",
            "@composer install --prefer-dist"
        ]
    }
}
```

## Descrições personalizadas

Você pode definir descrições personalizadas para scripts usando o seguinte no
seu `composer.json`:

```json
{
    "scripts-descriptions": {
        "test": "Run all tests!"
    }
}
```

As descrições são utilizadas nos comandos `composer list` ou `composer run -l`
para descrever o que os scripts fazem quando o comando é executado.

> **Nota:** Você só pode definir descrições personalizadas para comandos
> personalizados.

## Aliases personalizados

A partir do Composer 2.7, você pode definir aliases personalizados para scripts
utilizando o seguinte no seu `composer.json`:

```json
{
    "scripts-aliases": {
        "phpstan": ["stan", "analyze"]
    }
}
```

Os aliases fornecem nomes de comando alternativos.

> **Nota:** Você só pode definir aliases personalizados para comandos
> personalizados.
