---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/plugins.md
source_revision: 6ffc1177404d0c50119c22dde6564a380f4a82c9
translation_status: ready

tagline: Modifique e estenda a funcionalidade do Composer
---

# Configurando e usando plugins

## Visão geral

Você pode querer alterar ou expandir a funcionalidade do Composer com suas
próprias implementações.
Por exemplo, se o seu ambiente impuser requisitos específicos ao comportamento
do Composer que não se aplicam à maioria das pessoas usuárias, ou se você quiser
realizar algo com o Composer de uma maneira que não seja desejada pela maioria
delas.

Nesses casos, você pode considerar a criação de um plugin para lidar com sua
lógica específica.

## Criando um plugin

Um plugin é um pacote Composer comum que inclui seu código como parte do pacote
e também pode depender de outros pacotes.

### Pacote de plugin

O arquivo do pacote é igual a qualquer outro arquivo de pacote, mas com os
seguintes requisitos:

1. O atributo [type][1] deve ser `composer-plugin`.
2. O atributo [extra][2] deve conter um elemento `class` definindo o nome da
   classe do plugin (incluindo o namespace).
   Se um pacote contiver múltiplos plugins, este pode ser um array de nomes de
   classes.
3. Você deve declarar como dependência o pacote especial chamado
   `composer-plugin-api` para definir com quais versões da API de Plugins o seu
   plugin é compatível.
   Declarar esse pacote não adiciona, de fato, nenhuma dependência extra; apenas
   especifica qual versão da API de Plugins deve ser utilizada.

> **Nota:** Ao desenvolver um plugin, embora não seja obrigatório, é útil
> adicionar uma dependência `require-dev` para `composer/composer` a fim de
> obter autocompletar da IDE para as classes do Composer.

A versão exigida do `composer-plugin-api` segue as mesmas [regras][7] aplicadas
a pacotes comuns.

A versão atual da API de Plugins do Composer é `2.9.0`.

Um exemplo de arquivo `composer.json` válido para um plugin (com a parte de
autoloading omitida e uma dependência `require-dev` opcional para
`composer/composer`, visando o autocompletar da IDE):

```json
{
    "name": "my/plugin-package",
    "type": "composer-plugin",
    "require": {
        "composer-plugin-api": "^2.0"
    },
    "require-dev": {
        "composer/composer": "^2.0"
    },
    "extra": {
        "class": "My\\Plugin"
    }
}
```

### Classe do plugin

Todo plugin deve fornecer uma classe que implemente a interface
[`Composer\Plugin\PluginInterface`][3].
O método `activate()` do plugin é chamado após o carregamento do plugin e recebe
uma instância de [`Composer\Composer`][4], bem como uma instância de
[`Composer\IO\IOInterface`][5].
Com esses dois objetos, é possível ler toda a configuração e manipular todos os
objetos e estados internos conforme desejado.

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

## Manipulador de eventos

Além disso, os plugins podem implementar a interface
[`Composer\EventDispatcher\EventSubscriberInterface`][6] para que seus
manipuladores de eventos sejam registrados automaticamente no `EventDispatcher`
quando o plugin for carregado.

Para registrar um método em um evento, implemente o método
`getSubscribedEvents()` e faça com que ele retorne um array.
A chave do array deve ser o [nome do evento](./scripts.md#nomes-de-eventos) e o
valor deve ser o nome do método a ser chamado nesta classe.

> **Nota:** Se você não souber qual evento monitorar, pode executar um comando
> do Composer com a variável de ambiente `COMPOSER_DEBUG_EVENTS=1` definida;
> isso pode ajudar a identificar o evento que você procura.

```php
public static function getSubscribedEvents()
{
    return [
        'post-autoload-dump' => 'methodToBeCalled',
        // ^ event name ^         ^ method name ^
    ];
}
```

Por padrão, a prioridade de um manipulador de eventos é definida como 0.
A prioridade pode ser alterada associando-se uma tupla na qual o primeiro valor
é o nome do método, como anteriormente, e o segundo valor é um número inteiro
que representa a prioridade.
Valores inteiros mais altos representam prioridades mais altas.
A prioridade 2 é executada antes da prioridade 1, e assim por diante.

```php
public static function getSubscribedEvents()
{
    return [
        // Will be called before events with priority 0
        'post-autoload-dump' => ['methodToBeCalled', 1]
    ];
}
```

Se vários métodos precisarem ser chamados, um array de tuplas pode ser anexado a
cada evento.
As tuplas não precisam incluir a prioridade.
Se ela for omitida, o valor padrão será 0.

```php
public static function getSubscribedEvents()
{
    return [
        'post-autoload-dump' => [
            ['methodToBeCalled'      ], // Priority defaults to 0
            ['someOtherMethodName', 1], // This fires first
        ]
    ];
}
```

Aqui está um exemplo completo:

```php
<?php

namespace Naderman\Composer\AWS;

use Composer\Composer;
use Composer\EventDispatcher\EventSubscriberInterface;
use Composer\IO\IOInterface;
use Composer\Plugin\PluginInterface;
use Composer\Plugin\PluginEvents;
use Composer\Plugin\PreFileDownloadEvent;

class AwsPlugin implements PluginInterface, EventSubscriberInterface
{
    protected $composer;
    protected $io;

    public function activate(Composer $composer, IOInterface $io)
    {
        $this->composer = $composer;
        $this->io = $io;
    }

    public function deactivate(Composer $composer, IOInterface $io)
    {
    }

    public function uninstall(Composer $composer, IOInterface $io)
    {
    }

    public static function getSubscribedEvents()
    {
        return [
            PluginEvents::PRE_FILE_DOWNLOAD => [
                ['onPreFileDownload', 0]
            ],
        ];
    }

    public function onPreFileDownload(PreFileDownloadEvent $event)
    {
        $protocol = parse_url($event->getProcessedUrl(), PHP_URL_SCHEME);

        if ($protocol === 's3') {
            // ...
        }
    }
}
```

## Capacidades de plugins

O Composer define um conjunto padrão de capacidades que podem ser implementadas
por plugins.
O objetivo é tornar o ecossistema de plugins mais estável, reduzindo a
necessidade de manipular o estado interno da classe [`Composer\Composer`][4] ao
fornecer pontos de extensão explícitos para requisitos comuns de plugins.

Classes de plugins que utilizam capacidades devem implementar a interface
[`Composer\Plugin\Capable`][8] e declarar suas capacidades no método
`getCapabilities()`.
Esse método deve retornar um array, tendo como _chave_ o nome da classe de
capacidade do Composer e como _valor_ o nome da classe de implementação da
referida capacidade fornecida pelo próprio plugin:

```php
<?php

namespace My\Composer;

use Composer\Composer;
use Composer\IO\IOInterface;
use Composer\Plugin\PluginInterface;
use Composer\Plugin\Capable;

class Plugin implements PluginInterface, Capable
{
    public function activate(Composer $composer, IOInterface $io)
    {
    }

    public function getCapabilities()
    {
        return [
            'Composer\Plugin\Capability\CommandProvider' => 'My\Composer\CommandProvider',
        ];
    }
}
```

### Provedor de comandos

A funcionalidade [`Composer\Plugin\Capability\CommandProvider`][9] permite
registrar comandos adicionais para o Composer:

```php
<?php

namespace My\Composer;

use Composer\Plugin\Capability\CommandProvider as CommandProviderCapability;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Composer\Command\BaseCommand;

class CommandProvider implements CommandProviderCapability
{
    public function getCommands()
    {
        return [new Command];
    }
}

class Command extends BaseCommand
{
    protected function configure(): void
    {
        $this->setName('custom-plugin-command');
    }

    protected function execute(InputInterface $input, OutputInterface $output): int
    {
        $output->writeln('Executing');

        return 0;
    }
}
```

Agora, o comando `custom-plugin-command` está disponível juntamente com os
comandos do Composer.

> _Os comandos do Composer baseiam-se no [Componente Console do Symfony][10]._

## Executando plugins manualmente

Plugins associados a um evento podem ser executados manualmente por meio do
comando `run-script`.
Isso funciona da mesma maneira que
[executar scripts manualmente](scripts.md#executando-scripts-manualmente).

Se for outro tipo de plugin, a melhor forma de testá-lo é, provavelmente,
utilizar um [repositório do tipo path](../05-repositories.md#path) para incluir
o plugin como dependência em um projeto de teste.
Se você estiver desenvolvendo localmente e quiser realizar testes frequentes,
pode configurar o repositório do tipo path para usar links simbólicos
(symlinks), garantindo assim que as alterações sejam atualizadas imediatamente.
Caso contrário, será necessário executar `rm -rf vendor && composer update`
sempre que quiser instalá-lo ou executá-lo novamente.

## Utilizando plugins

Pacotes de plugins são carregados automaticamente assim que são instalados e
serão carregados na inicialização do Composer caso constem na lista de pacotes
instalados do projeto atual.
Além disso, todos os pacotes de plugins instalados no diretório `COMPOSER_HOME`
por meio do comando global do Composer são carregados antes dos plugins locais
do projeto.

> Você pode utilizar a opção `--no-plugins` nos comandos do Composer para
> desabilitar todos os plugins instalados.
> Isso pode ser particularmente útil caso algum dos plugins cause erros e você
> deseje atualizá-lo ou desinstalá-lo.

## Auxiliares de plugins

A partir do Composer 2, porque a `DownloaderInterface` pode, às vezes, retornar
*Promises* e ter sido dividida em mais etapas do que anteriormente,
disponibilizamos um [SyncHelper][11] para facilitar o download e a instalação de
pacotes.

## Atributos extras de plugins

Algumas funcionalidades especiais de plugins podem ser habilitadas utilizando
atributos extras no arquivo `composer.json` do plugin.

### class

[Veja acima](#pacote-de-plugin) a explicação sobre o atributo `class` e como ele
funciona.

### plugin-modifies-downloads

Alguns plugins especiais precisam atualizar as URLs de download dos pacotes
antes que eles sejam baixados.

A partir do Composer 2.0, todos os pacotes são baixados antes de serem
instalados.
Isso significa que, na primeira instalação, seu plugin ainda não está instalado
quando o download ocorre, e ele não tem a oportunidade de atualizar as URLs a
tempo.

Especificar `{"extra": {"plugin-modifies-downloads": true}}` no seu
`composer.json` indicará ao Composer que o plugin deve ser instalado
separadamente antes de prosseguir com o restante dos downloads de pacotes.
No entanto, isso torna o processo geral de instalação ligeiramente mais lento;
portanto, não utilize essa opção em plugins que não a exijam absolutamente.

### plugin-modifies-install-path

Alguns plugins especiais modificam o caminho de instalação dos pacotes.

A partir do Composer 2.2.9, você pode especificar
`{"extra": {"plugin-modifies-install-path": true}}` no seu `composer.json` para
indicar ao Composer que o plugin deve ser ativado o mais rápido possível,
evitando efeitos colaterais indesejados decorrentes de o Composer presumir que
os pacotes estão instalados em um local diferente daquele onde realmente estão.

### plugin-optional

Como os plugins do Composer podem ser usados para realizar ações necessárias
para a instalação de uma aplicação funcional, como modificar o caminho onde os
arquivos são armazenados, ignorar plugins necessários inadvertidamente pode
resultar em aplicações que não funcionam.
Por isso, no modo não interativo, o Composer falhará se um novo plugin não
estiver listado em ["allow-plugins"](../06-config.md#allow-plugins); isso força
as pessoas usuárias a decidirem se desejam executar o plugin, evitando falhas
silenciosas.

A partir do Composer 2.5.3, você pode usar a configuração
`{"extra": {"plugin-optional": true}}` em seu plugin para informar ao Composer
que pular o plugin não acarreta consequências catastróficas, e que ele pode ser
desativado com segurança no modo não interativo caso ainda não esteja listado
em "allow-plugins".
A próxima execução interativa do Composer ainda solicitará que as pessoas
usuárias escolham se desejam ativar ou desativar o plugin.

## Autoloading de plugins

Como os plugins são carregados pelo Composer em tempo de execução, e para
garantir que plugins que dependem de outros pacotes funcionem corretamente, um
autoloader de tempo de execução é criado sempre que um plugin é carregado.
Esse autoloader é configurado apenas para carregar as dependências do plugin;
portanto, você pode não ter acesso a todos os pacotes instalados.

## Suporte a análise estática

A partir do Composer 2.3.7, disponibilizamos um arquivo de configuração
`phpstan/rules.neon` para o PHPStan, que oferece verificação adicional de erros
ao desenvolver plugins para o Composer.

### Uso com o [PHPStan Extension Installer][13]

Os arquivos de configuração necessários são carregados automaticamente caso o
projeto do seu plugin declare uma dependência do `phpstan/extension-installer`.

### Instalação manual alternativa

Para utilizá-lo, o projeto do seu plugin Composer precisa de um
[arquivo de configuração do PHPStan][12] que inclua o arquivo
`phpstan/rules.neon`:

```neon
includes:
	- vendor/composer/composer/phpstan/rules.neon

// your remaining config..
```

[1]: ../04-schema.md#type
[2]: ../04-schema.md#extra
[3]: https://github.com/composer/composer/blob/main/src/Composer/Plugin/PluginInterface.php
[4]: https://github.com/composer/composer/blob/main/src/Composer/Composer.php
[5]: https://github.com/composer/composer/blob/main/src/Composer/IO/IOInterface.php
[6]: https://github.com/composer/composer/blob/main/src/Composer/EventDispatcher/EventSubscriberInterface.php
[7]: ../01-basic-usage.md#restrições-de-versão-do-pacote
[8]: https://github.com/composer/composer/blob/main/src/Composer/Plugin/Capable.php
[9]: https://github.com/composer/composer/blob/main/src/Composer/Plugin/Capability/CommandProvider.php
[10]: https://symfony.com/doc/current/components/console.html
[11]: https://github.com/composer/composer/blob/main/src/Composer/Util/SyncHelper.php
[12]: https://phpstan.org/config-reference#multiple-files
[13]: https://github.com/phpstan/extension-installer#usage
