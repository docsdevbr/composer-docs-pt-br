---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/03-cli.md
source_revision: d63e06ab5f6593d1c50859359aabd478865407bd
translation_status: ready
---

# Interface de linha de comando / Comandos

Você já aprendeu como usar a interface de linha de comando para fazer algumas
coisas.
Este capítulo documenta todos os comandos disponíveis.

Para obter ajuda na linha de comando, basta executar `composer` ou
`composer list` para consultar a lista completa de comandos e, em seguida,
`--help` combinado com qualquer um deles para fornecer mais informações.

Como o Composer usa o [symfony/console](https://github.com/symfony/console),
você pode executar os comandos usando os nomes abreviados, se não forem
ambíguos.

```shell
php composer.phar dump
```

executa `composer dump-autoload`.

## Autocompletar para Bash

Para instalar o recurso de autocompletar para Bash, você pode executar
`composer completion bash > completion.bash`.
Isso criará um arquivo `completion.bash` no diretório atual.

Em seguida, execute `source completion.bash` para ativá-lo na sessão atual do
terminal.

Mova e renomeie o arquivo `completion.bash` para
`/etc/bash_completion.d/composer` para que ele seja carregado automaticamente em
novos terminais.

## Opções globais

As seguintes opções estão disponíveis em todos os comandos:

* **--verbose (-v):** aumenta a verbosidade das mensagens.
* **--help (-h):** exibe informações de ajuda.
* **--quiet (-q):** não gera nenhuma mensagem.
* **--no-interaction (-n):** não faz nenhuma pergunta interativa.
* **--no-plugins:** desabilita os plugins.
* **--no-scripts:** pula a execução de scripts definidos no `composer.json`.
* **--no-cache:** desabilita o uso do diretório de cache.
  O mesmo que definir a variável de ambiente `COMPOSER_CACHE_DIR` como
  `/dev/null` (ou `NUL` no Windows).
* **--working-dir (-d):** se especificada, usa o diretório fornecido como
  diretório de trabalho.
* **--profile:** exibe informações sobre tempo de execução e uso da memória.
* **--ansi:** força a saída ANSI.
* **--no-ansi:** desabilita a saída ANSI.
* **--version (-V):** exibe a versão desta aplicação.

## Códigos de saída do processo

* **0:** OK
* **1:** código de erro genérico/desconhecido
* **2:** código de erro de resolução de dependências

## init

No capítulo [Bibliotecas](02-libraries.md), vimos como criar um `composer.json`
manualmente.
Também existe um comando `init` disponível para isso.

Ao executar o comando, ele solicitará interativamente que você preencha os
campos, enquanto usa alguns padrões inteligentes.

```shell
php composer.phar init
```

### Opções

* **--name:** nome do pacote.
* **--description:** descrição do pacote.
* **--author:** nome da pessoa autora do pacote.
* **--type:** tipo do pacote.
* **--homepage:** página do pacote.
* **--require:** pacote requerido com uma restrição de versão.
  Deve estar no formato `foo/bar:1.0.0`.
* **--require-dev:** requisitos de desenvolvimento, consulte **--require**.
* **--stability (-s):** valor para o campo `minimum-stability`.
* **--license (-l):** licença do pacote.
* **--repository:** fornece um ou mais repositórios personalizados.
  Eles serão armazenados no `composer.json` gerado e usados para o preenchimento
  automático ao solicitar a lista de requisitos.
  Cada repositório pode ser um URL HTTP apontando para um repositório do
  `composer` ou uma string JSON semelhante à string aceita pela chave
  [`repositories`](04-schema.md#repositories).
* **--autoload (-a):** adiciona um mapeamento de autoloading PSR-4 ao
  `composer.json`.
  Mapeia automaticamente o namespace do seu pacote para o diretório fornecido.
  (Espera um caminho relativo, ex.: `src/`.)
  Consulte também o [autoloading PSR-4](04-schema.md#psr-4).

## install / i

O comando `install` lê o arquivo `composer.json` presente no diretório atual,
resolve as dependências e as instala em `vendor`.

```shell
php composer.phar install
```

Se houver um arquivo `composer.lock` no diretório atual, ele usará as versões
exatas desse arquivo em vez de resolvê-las.
Isso garante que todas as pessoas usando a biblioteca obtenham as mesmas versões
das dependências.

Se não houver um arquivo `composer.lock`, o Composer criará um após a resolução
das dependências.

### Opções

* **--prefer-install:** existem duas formas de baixar um pacote: `source` e
  `dist`.
  O Composer usa `dist` por padrão.
  Se você usar `--prefer-install=source` (ou `--prefer-source`), o Composer fará
  a instalação a partir do `source`, caso ele esteja disponível.
  Isso é útil se você quiser corrigir uma falha em um projeto e obter
  diretamente um clone local da dependência.
  Para obter o comportamento legado, no qual o Composer usa `source`
  automaticamente para versões de desenvolvimento de pacotes, use
  `--prefer-install=auto`.
  Consulte também [config.preferred-install](06-config.md#preferred-install).
  O uso desta flag substituirá o valor definido na configuração.
* **--dry-run:** se você quiser executar uma instalação sem realmente instalar
  um pacote, pode usar `--dry-run`.
  Isso simulará a instalação e mostrará o que aconteceria.
* **--download-only:** apenas baixa, não instala pacotes.
* **--dev:** instala os pacotes listados em `require-dev` (este é o
  comportamento padrão).
* **--no-dev:** ignora a instalação dos pacotes listados em `require-dev`.
  A geração do autoloader ignora as regras em `autoload-dev`.
  Consulte também [COMPOSER_NO_DEV](#composer-no-dev).
* **--no-autoloader:** ignora a geração do autoloader.
* **--no-progress:** remove a exibição de progresso que pode interferir em
  alguns terminais ou scripts que não tratam caracteres de backspace.
* **--audit:** executa uma auditoria após a conclusão da instalação.
* **--audit-format:** formato de saída da auditoria.
  Deve ser "table", "plain", "json" ou "summary" (padrão).
* **--no-security-blocking:** OBSOLETO; use `--no-blocking` em vez disso.
  Permite instalar pacotes com avisos de segurança ou que estejam abandonados.
  Consulte também
  [COMPOSER_NO_SECURITY_BLOCKING](#composer-no-security-blocking).
* **--no-blocking:** desabilita todo o bloqueio de dependências baseado em
  políticas durante a execução deste comando.
  Consulte também [COMPOSER_NO_BLOCKING](#composer-no-blocking).
* **--optimize-autoloader (-o):** converte o autoloading PSR-0/4 em um mapa de
  classes para obter um autoloader mais rápido.
  Isso é recomendado especialmente em produção, mas pode demorar um pouco para
  ser executado, portanto, atualmente não é feito por padrão.
* **--classmap-authoritative (-a):** faz o autoloading apenas das classes do
  mapa de classes.
  Habilita implicitamente `--optimize-autoloader`.
* **--strict-psr-autoloader:** retorna um código de saída de falha (6) se houver
  erros de mapeamento PSR-4 ou PSR-0 no projeto atual (excluindo dependências).
  Requer `--optimize-autoloader` para funcionar.
* **--apcu-autoloader:** usa a APCu para armazenar em cache as classes
  encontradas/não encontradas.
* **--apcu-autoloader-prefix:** usa um prefixo personalizado para o cache do
  autoloader da APCu.
  Habilita implicitamente `--apcu-autoloader`.
* **--ignore-platform-reqs:** ignora todos os requisitos de plataforma (`php`,
  `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina local não
  atenda a eles.
  Consulte também a opção de configuração [`platform`](06-config.md#platform).
* **--ignore-platform-req:** ignora um requisito de plataforma específico
  (`php`, `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina
  local não atenda a ele.
  É possível ignorar múltiplos requisitos usando curingas.
  Acrescentar um `+` faz com que apenas o limite superior dos requisitos seja
  ignorado.
  Por exemplo, se um pacote exige `php: ^7`, a opção
  `--ignore-platform-req=php+` permitiria a instalação no PHP 8, mas a
  instalação no PHP 5.6 ainda falharia.

## update / u / upgrade

Para obter as versões mais recentes das dependências e atualizar o arquivo
`composer.lock`, você deve usar o comando `update`.
Este comando também tem o apelido `upgrade`, já que ele faz o mesmo que
`upgrade`, se você estiver pensando no `apt-get` ou em gerenciadores de pacotes
similares.

```shell
php composer.phar update
```

Isso resolverá todas as dependências do projeto e gravará as versões exatas no
`composer.lock`.

Se você quiser atualizar apenas alguns pacotes e não todos, pode listá-los da
seguinte forma:

```shell
php composer.phar update vendor/pacote vendor/pacote2
```

Você também pode usar curingas para atualizar vários pacotes de uma vez:

```shell
php composer.phar update "vendor/*"
```

Se você quiser fazer o downgrade de um pacote para uma versão específica sem
alterar o seu `composer.json`, você pode usar `--with` e fornecer uma restrição
de versão personalizada:

```shell
php composer.phar update --with vendor/pacote:2.0.1
```

Observe que, com o comando acima, todos os pacotes serão atualizados.
Se você quiser atualizar apenas os pacotes para os quais definiu restrições
personalizadas usando `--with`, pode dispensar o `--with` e, em vez disso, usar
restrições com a sintaxe de atualização parcial:

```shell
php composer.phar update vendor/pacote:2.0.1 vendor/pacote2:3.0.*
```

> **Nota:** Para pacotes que também são exigidos no seu `composer.json`, a
> restrição personalizada deve ser um subconjunto da restrição existente.
> As restrições do `composer.json` continuam valendo, e o `composer.json` não é
> modificado por essas restrições de atualização temporárias.

### Opções

* **--prefer-install:** existem duas formas de baixar um pacote: `source` e
  `dist`.
  O Composer usa `dist` por padrão.
  Se você usar `--prefer-install=source` (ou `--prefer-source`), o Composer fará
  a instalação a partir do `source`, caso ele esteja disponível.
  Isso é útil se você quiser corrigir uma falha em um projeto e obter
  diretamente um clone local da dependência.
  Para obter o comportamento legado, no qual o Composer usa `source`
  automaticamente para versões de desenvolvimento de pacotes, use
  `--prefer-install=auto`.
  Consulte também [config.preferred-install](06-config.md#preferred-install).
  O uso desta flag substituirá o valor definido na configuração.
* **--dry-run:** simula o comando sem realmente fazer nada.
* **--dev:** instala os pacotes listados em `require-dev` (este é o
  comportamento padrão).
* **--no-dev:** ignora a instalação dos pacotes listados em `require-dev`.
  A geração do autoloader ignora as regras em `autoload-dev`.
  Consulte também [COMPOSER_NO_DEV](#composer-no-dev).
* **--no-install:** não executa a etapa de instalação após atualizar o arquivo
  `composer.lock`.
* **--no-audit:** não executa as etapas de auditoria após atualizar o arquivo
  `composer.lock`.
  Consulte também [COMPOSER_NO_AUDIT](#composer-no-audit).
* **--audit-format:** formato de saída da auditoria.
  Deve ser "table", "plain", "json" ou "summary" (padrão).
* **--no-security-blocking:** OBSOLETO; use `--no-blocking` em vez disso.
  Permite instalar pacotes com avisos de segurança ou que estejam abandonados.
  Consulte também
  [COMPOSER_NO_SECURITY_BLOCKING](#composer-no-security-blocking).
* **--no-blocking:** desabilita todo o bloqueio de dependências baseado em
  políticas durante a execução deste comando.
  Consulte também [COMPOSER_NO_BLOCKING](#composer-no-blocking).
* **--lock:** sobrescreve o hash do arquivo de bloqueio para suprimir o alerta
  de que o arquivo de bloqueio está desatualizado, sem atualizar as versões dos
  pacotes.
  Metadados dos pacotes, como mirrors e URLs, são atualizados caso tenham
  sofrido alterações.
* **--with:** restrição de versão temporária para adicionar, por exemplo,
  `foo/bar:1.0.0` ou `foo/bar=1.0.0`.
* **--no-autoloader:** ignora a geração do autoloader.
* **--no-progress:** remove a exibição de progresso que pode interferir em
  alguns terminais ou scripts que não tratam caracteres de backspace.
* **--with-dependencies (-w):** também atualiza as dependências dos pacotes da
  lista de argumentos, exceto aquelas que são requisitos raiz.
  Também pode ser definida por meio da variável de ambiente
  `COMPOSER_WITH_DEPENDENCIES=1`.
* **--with-all-dependencies (-W):** também atualiza as dependências dos pacotes
  da lista de argumentos, incluindo aquelas que são requisitos raiz.
  Também pode ser definida por meio da variável de ambiente
  `COMPOSER_WITH_ALL_DEPENDENCIES=1`.
* **--optimize-autoloader (-o):** converte o autoloading PSR-0/4 em um mapa de
  classes para obter um autoloader mais rápido.
  Isso é recomendado especialmente em produção, mas pode demorar um pouco para
  ser executado, portanto, atualmente não é feito por padrão.
* **--classmap-authoritative (-a):** faz o autoloading apenas das classes do
  mapa de classes.
  Habilita implicitamente `--optimize-autoloader`.
* **--strict-psr-autoloader:** retorna um código de saída de falha (6) se houver
  erros de mapeamento PSR-4 ou PSR-0 no projeto atual (excluindo dependências).
  Requer `--optimize-autoloader` para funcionar.
* **--apcu-autoloader:** usa a APCu para armazenar em cache as classes
  encontradas/não encontradas.
* **--apcu-autoloader-prefix:** usa um prefixo personalizado para o cache do
  autoloader da APCu.
  Habilita implicitamente `--apcu-autoloader`.
* **--ignore-platform-reqs:** ignora todos os requisitos de plataforma (`php`,
  `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina local não
  atenda a eles.
  Consulte também a opção de configuração [`platform`](06-config.md#platform).
* **--ignore-platform-req:** ignora um requisito de plataforma específico
  (`php`, `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina
  local não atenda a ele.
  É possível ignorar múltiplos requisitos usando curingas.
  Acrescentar um `+` faz com que apenas o limite superior dos requisitos seja
  ignorado.
  Por exemplo, se um pacote exige `php: ^7`, a opção
  `--ignore-platform-req=php+` permitiria a instalação no PHP 8, mas a
  instalação no PHP 5.6 ainda falharia.
* **--prefer-stable:** prefere versões estáveis das dependências.
  Também pode ser definida por meio da variável de ambiente
  `COMPOSER_PREFER_STABLE=1`.
* **--prefer-lowest:** prefere as versões mais antigas das dependências.
  Útil para testar versões mínimas de requisitos, geralmente usada com
  `--prefer-stable`.
  Também pode ser definida por meio da variável de ambiente
  `COMPOSER_PREFER_LOWEST=1`.
* **--minimal-changes (-m):** realiza apenas as alterações absolutamente
  necessárias nas dependências.
  Se os pacotes não puderem ser mantidos na versão atualmente fixada, eles são
  atualizados.
  Para atualizações parciais, os pacotes incluídos na lista de permissões são
  sempre atualizados completamente.
  Também pode ser definida por meio da variável de ambiente
  `COMPOSER_MINIMAL_CHANGES=1`.
* **--patch-only:** permite apenas atualizações de versão de correção para as
  dependências atualmente instaladas.
* **--interactive:** interface interativa com preenchimento automático para
  selecionar os pacotes a serem atualizados.
* **--root-reqs:** restringe a atualização às dependências de primeiro grau.
* **--bump-after-update:** executa o `bump` após realizar a atualização.
  Defina como `dev` ou `no-dev` para atualizar a versão apenas dessas
  dependências.

Especificar uma das palavras `mirrors`, `lock` ou `nothing` como um argumento
tem o mesmo efeito que especificar a opção `--lock`, por exemplo,
`composer update mirrors` é o mesmo que `composer update --lock`.

## require / r

O comando `require` adiciona pacotes ao arquivo `composer.json` presente no
diretório atual.
Se nenhum arquivo existir, um arquivo será criado durante a execução do comando.

Se você não especificar um pacote, o Composer solicitará que você procure por um
e, com base nos resultados, apresentará uma lista de correspondências para você
adicionar como dependência.

```shell
php composer.phar require
```

Após adicionar/alterar os requisitos, os requisitos modificados serão instalados
ou atualizados.

Se você não quiser escolher os requisitos interativamente, pode passá-los para
o comando:

```shell
php composer.phar require vendor/pacote:2.* vendor/pacote2:dev-master
```

Se você não especificar uma restrição de versão, o Composer escolherá uma
adequada com base nas versões de pacote disponíveis.

```shell
php composer.phar require vendor/pacote vendor/pacote2
```

Se você não quiser instalar as novas dependências imediatamente, pode executá-lo
com `--no-update`.

### Opções

* **--dev:** adiciona pacotes a `require-dev`.
* **--dry-run:** simula o comando sem realmente fazer nada.
* **--prefer-install:** existem duas formas de baixar um pacote: `source` e
  `dist`.
  O Composer usa `dist` por padrão.
  Se você usar `--prefer-install=source` (ou `--prefer-source`), o Composer fará
  a instalação a partir do `source`, caso ele esteja disponível.
  Isso é útil se você quiser corrigir uma falha em um projeto e obter
  diretamente um clone local da dependência.
  Para obter o comportamento legado, no qual o Composer usa `source`
  automaticamente para versões de desenvolvimento de pacotes, use
  `--prefer-install=auto`.
  Consulte também [config.preferred-install](06-config.md#preferred-install).
  O uso desta flag substituirá o valor definido na configuração.
* **--no-progress:** remove a exibição de progresso que pode interferir em
  alguns terminais ou scripts que não tratam caracteres de backspace.
* **--no-update:** desabilita a atualização automática das dependências (implica
  `--no-install`).
* **--no-install:** não executa a etapa de instalação após atualizar o arquivo
  `composer.lock`.
* **--no-audit:** não executa as etapas de auditoria após atualizar o arquivo
  `composer.lock`.
  Consulte também [COMPOSER_NO_AUDIT](#composer-no-audit).
* **--audit-format:** formato de saída da auditoria.
  Deve ser "table", "plain", "json" ou "summary" (padrão).
* **--no-security-blocking:** OBSOLETO; use `--no-blocking` em vez disso.
  Permite instalar pacotes com avisos de segurança ou que estejam abandonados.
  Consulte também
  [COMPOSER_NO_SECURITY_BLOCKING](#composer-no-security-blocking).
* **--no-blocking:** desabilita todo o bloqueio de dependências baseado em
  políticas durante a execução deste comando.
  Consulte também [COMPOSER_NO_BLOCKING](#composer-no-blocking).
* **--update-no-dev:** executa a atualização de dependências com a opção
  `--no-dev`.
  Consulte também [COMPOSER_NO_DEV](#composer-no-dev).
* **--update-with-dependencies (-w):** também atualiza as dependências dos novos
  pacotes requeridos, exceto aquelas que são requisitos raiz.
  Também pode ser definida por meio da variável de ambiente
  `COMPOSER_WITH_DEPENDENCIES=1`.
* **--update-with-all-dependencies (-W):** também atualiza as dependências dos
  novos pacotes requeridos, incluindo aquelas que são requisitos raiz.
  Também pode ser definida por meio da variável de ambiente
  `COMPOSER_WITH_ALL_DEPENDENCIES=1`.
* **--ignore-platform-reqs:** ignora todos os requisitos de plataforma (`php`,
  `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina local não
  atenda a eles.
  Consulte também a opção de configuração [`platform`](06-config.md#platform).
* **--ignore-platform-req:** ignora um requisito de plataforma específico
  (`php`, `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina
  local não atenda a ele.
  É possível ignorar múltiplos requisitos usando curingas.
* **--prefer-stable:** prefere versões estáveis das dependências.
  Também pode ser definida por meio da variável de ambiente
  `COMPOSER_PREFER_STABLE=1`.
* **--prefer-lowest:** prefere as versões mais antigas das dependências.
  Útil para testar versões mínimas de requisitos, geralmente usada com
  `--prefer-stable`.
  Também pode ser definida por meio da variável de ambiente
  `COMPOSER_PREFER_LOWEST=1`.
* **--minimal-changes (-m):** durante uma atualização com `-w`/`-W`, realiza
  apenas as alterações absolutamente necessárias nas dependências transitivas.
  Também pode ser definido por meio da variável de ambiente
  `COMPOSER_MINIMAL_CHANGES=1`.
* **--sort-packages:** mantém os pacotes ordenados no `composer.json`.
* **--optimize-autoloader (-o):** converte o autoloading PSR-0/4 em um mapa de
  classes para obter um autoloader mais rápido.
  Isso é recomendado especialmente em produção, mas pode demorar um pouco para
  ser executado, portanto, atualmente não é feito por padrão.
* **--classmap-authoritative (-a):** faz o autoloading apenas das classes do
  mapa de classes.
  Habilita implicitamente `--optimize-autoloader`.
* **--apcu-autoloader:** usa a APCu para armazenar em cache as classes
  encontradas/não encontradas.
* **--apcu-autoloader-prefix:** usa um prefixo personalizado para o cache do
  autoloader da APCu.
  Habilita implicitamente `--apcu-autoloader`.

## remove / rm / uninstall

O comando `remove` remove pacotes do arquivo `composer.json` presente no
diretório atual.

```shell
php composer.phar remove vendor/pacote vendor/pacote2
```

Após remover os requisitos, os requisitos modificados serão desinstalados.

### Opções

* **--unused:** remove pacotes não usados que não são (mais) uma dependência
  direta ou indireta.
* **--dev:** remove pacotes de `require-dev`.
* **--dry-run:** simula o comando sem realmente fazer nada.
* **--no-progress:** remove a exibição de progresso que pode interferir em
  alguns terminais ou scripts que não tratam caracteres de backspace.
* **--no-update:** desabilita a atualização automática das dependências (implica
  `--no-install`).
* **--no-install:** não executa a etapa de instalação após atualizar o arquivo
  `composer.lock`.
* **--no-audit:** não executa as etapas de auditoria após atualizar o arquivo
  `composer.lock`.
  Consulte também [COMPOSER_NO_AUDIT](#composer-no-audit).
* **--audit-format:** formato de saída da auditoria.
  Deve ser "table", "plain", "json" ou "summary" (padrão).
* **--no-security-blocking:** OBSOLETO; use `--no-blocking` em vez disso.
  Permite instalar pacotes com avisos de segurança ou que estejam abandonados.
  Consulte também
  [COMPOSER_NO_SECURITY_BLOCKING](#composer-no-security-blocking).
* **--no-blocking:** desabilita todo o bloqueio de dependências baseado em
  políticas durante a execução deste comando.
  Consulte também [COMPOSER_NO_BLOCKING](#composer-no-blocking).
* **--update-no-dev:** executa a atualização de dependências com a opção
  `--no-dev`.
  Consulte também [COMPOSER_NO_DEV](#composer-no-dev).
* **--update-with-dependencies (-w):** também atualiza as dependências dos
  pacotes removidos.
  Também pode ser definida por meio da variável de ambiente
  `COMPOSER_WITH_DEPENDENCIES=1`.
  (Descontinuada, agora é o comportamento padrão.)
* **--update-with-all-dependencies (-W):** permite que todas as dependências
  herdadas sejam atualizadas, incluindo aquelas que são requisitos raiz.
  Também pode ser definida por meio da variável de ambiente
  `COMPOSER_WITH_ALL_DEPENDENCIES=1`.
* **--minimal-changes (-m):** durante uma atualização com `-w`/`-W`, realiza
  apenas as alterações absolutamente necessárias nas dependências transitivas.
  Também pode ser definido por meio da variável de ambiente
  `COMPOSER_MINIMAL_CHANGES=1`.
* **--ignore-platform-reqs:** ignora todos os requisitos de plataforma (`php`,
  `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina local não
  atenda a eles.
  Consulte também a opção de configuração [`platform`](06-config.md#platform).
* **--ignore-platform-req:** ignora um requisito de plataforma específico
  (`php`, `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina
  local não atenda a ele.
  É possível ignorar múltiplos requisitos usando curingas.
* **--optimize-autoloader (-o):** converte o autoloading PSR-0/4 em um mapa de
  classes para obter um autoloader mais rápido.
  Isso é recomendado especialmente em produção, mas pode demorar um pouco para
  ser executado, portanto, atualmente não é feito por padrão.
* **--classmap-authoritative (-a):** faz o autoloading apenas das classes do
  mapa de classes.
  Habilita implicitamente `--optimize-autoloader`.
* **--apcu-autoloader:** usa a APCu para armazenar em cache as classes
  encontradas/não encontradas.
* **--apcu-autoloader-prefix:** usa um prefixo personalizado para o cache do
  autoloader da APCu.
  Habilita implicitamente `--apcu-autoloader`.

## bump

O comando `bump` atualiza o limite inferior das suas dependências no
`composer.json` para as versões atualmente instaladas.
Isso ajuda a garantir que suas dependências não sejam revertidas para versões
anteriores acidentalmente devido a algum conflito, além de poder melhorar
ligeiramente o desempenho da resolução de dependências, pois limita a quantidade
de versões de pacotes que o Composer precisa analisar.

Executar esse comando indiscriminadamente em bibliotecas **NÃO** é recomendado,
pois isso restringe as dependências permitidas, o que pode causar o chamado
"inferno de dependências" para as pessoas usuárias.
No entanto, executá-lo com a opção `--dev-only` em bibliotecas pode ser
aceitável, já que as dependências de desenvolvimento são locais à biblioteca e
não afetam quem consome o pacote.

### Opções

* **--dev-only:** atualiza apenas as dependências listadas em "require-dev".
* **--no-dev-only:** atualiza apenas as dependências listadas em "require".
* **--dry-run:** exibe os pacotes que seriam atualizados, mas não executa
  nenhuma ação.

## reinstall

O comando `reinstall` localiza pacotes instalados pelo nome, desinstala-os e os
reinstala.
Isso permite realizar uma instalação limpa de um pacote caso você tenha
modificado seus arquivos ou deseje alterar o tipo de instalação usando
`--prefer-install`.

```shell
php composer.phar reinstall acme/foo acme/bar
```

Você pode especificar mais de um nome de pacote para reinstalar ou usar um
caractere curinga (*wildcard*) para selecionar vários pacotes de uma só vez:

```shell
php composer.phar reinstall "acme/*"
```

### Opções

* **--prefer-install:** existem duas formas de baixar um pacote: `source` e
  `dist`.
  O Composer usa `dist` por padrão.
  Se você usar `--prefer-install=source` (ou `--prefer-source`), o Composer fará
  a instalação a partir do `source`, caso ele esteja disponível.
  Isso é útil se você quiser corrigir uma falha em um projeto e obter
  diretamente um clone local da dependência.
  Para obter o comportamento legado, no qual o Composer usa `source`
  automaticamente para versões de desenvolvimento de pacotes, use
  `--prefer-install=auto`.
  Consulte também [config.preferred-install](06-config.md#preferred-install).
  O uso desta flag substituirá o valor definido na configuração.
* **--no-autoloader:** ignora a geração do autoloader.
* **--no-progress:** remove a exibição de progresso que pode interferir em
  alguns terminais ou scripts que não tratam caracteres de backspace.
* **--optimize-autoloader (-o):** converte o autoloading PSR-0/4 em um mapa de
  classes para obter um autoloader mais rápido.
  Isso é recomendado especialmente em produção, mas pode demorar um pouco para
  ser executado, portanto, atualmente não é feito por padrão.
* **--classmap-authoritative (-a):** faz o autoloading apenas das classes do
  mapa de classes.
  Habilita implicitamente `--optimize-autoloader`.
* **--apcu-autoloader:** usa a APCu para armazenar em cache as classes
  encontradas/não encontradas.
* **--apcu-autoloader-prefix:** usa um prefixo personalizado para o cache do
  autoloader da APCu.
  Habilita implicitamente `--apcu-autoloader`.
* **--ignore-platform-reqs:** ignora todos os requisitos de plataforma.
  Isso só tem efeito no contexto da geração do autoloader para o comando
  `reinstall`.
* **--ignore-platform-req:** ignora um requisito de plataforma específico.
  Isso só tem efeito no contexto da geração do autoloader para o comando
  `reinstall`.
  Múltiplos requisitos podem ser ignorados usando curingas.

## check-platform-reqs

O comando `check-platform-reqs` verifica se as suas versões do PHP e das
extensões correspondem aos requisitos de plataforma dos pacotes instalados.
Isso pode ser usado para verificar se um servidor de produção possui todas as
extensões necessárias para executar um projeto após a instalação, por exemplo.

Diferente de `update`/`install`, este comando ignorará as configurações em
`config.platform` e verificará os pacotes reais da plataforma para garantir que
você tenha as dependências de plataforma necessárias.

### Opções

* **--lock:** verifica as dependências apenas a partir do arquivo de bloqueio, e
  não dos pacotes instalados.
* **--no-dev:** desabilita a verificação das dependências dos pacotes listados
  em `require-dev`.
* **--format (-f):** formato da saída: `text` (padrão) ou `json`.

## global

O comando `global` permite executar outros comandos, como `install`, `remove`,
`require` ou `update`, como se você os estivesse executando a partir do
diretório [COMPOSER_HOME](#composer-home).

Este é apenas um auxiliar para gerenciar um projeto armazenado em um local
central que pode conter ferramentas CLI ou plugins do Composer que você deseja
ter disponíveis em qualquer lugar.

Ele pode ser usado para instalar utilitários CLI globalmente.
Aqui está um exemplo:

```shell
php composer.phar global require friendsofphp/php-cs-fixer
```

Agora, o binário `php-cs-fixer` está disponível globalmente.
Certifique-se de que o diretório global dos
[binários dos fornecedores](articles/vendor-binaries.md) esteja na sua variável
de ambiente `PATH`.
Você pode obter a sua localização com o seguinte comando:

```shell
php composer.phar global config bin-dir --absolute
```

Se você quiser atualizar o binário mais tarde, pode executar uma atualização
global:

```shell
php composer.phar global update
```

## search

O comando `search` permite pesquisar nos repositórios de pacotes do projeto
atual.
Geralmente, será o Packagist.
Você simplesmente passa os termos que deseja pesquisar.

```shell
php composer.phar search monolog
```

Você também pode pesquisar mais de um termo passando vários argumentos.

### Opções

* **--only-name (-N):** pesquisa apenas nos nomes dos pacotes.
* **--only-vendor (-O):** pesquisa apenas por nomes de
  fornecedores/organizações; retorna apenas o "vendor" como resultado.
* **--type (-t):** pesquisa por um tipo de pacote específico.
* **--format (-f):** permite escolher entre o formato de saída texto (padrão) ou
  JSON.
  Observe que, no JSON, apenas as chaves `name` e `description` têm presença
  garantida.
  As demais (`url`, `repository`, `downloads` e `favers`) estão disponíveis
  para resultados de pesquisa do Packagist.org, e outros repositórios podem
  retornar mais ou menos dados.

## show / info

Para listar todos os pacotes disponíveis, você pode usar o comando `show`.

```shell
php composer.phar show
```

Para filtrar a lista, você pode passar uma máscara de pacote usando curingas.

```shell
php composer.phar show monolog/*
```

```text
monolog/monolog 2.4.0 Sends your logs to files, sockets, inboxes, databases and various web services
```

Se você quiser consultar os detalhes de um determinado pacote, pode passar o
nome do pacote.

```shell
php composer.phar show monolog/monolog
```

```text
name     : monolog/monolog
descrip. : Sends your logs to files, sockets, inboxes, databases and various web services
keywords : log, logging, psr-3
versions : * 1.27.1
type     : library
license  : MIT License (MIT) (OSI approved) https://spdx.org/licenses/MIT.html#licenseText
homepage : http://github.com/Seldaek/monolog
source   : [git] https://github.com/Seldaek/monolog.git 904713c5929655dc9b97288b69cfeedad610c9a1
dist     : [zip] https://api.github.com/repos/Seldaek/monolog/zipball/904713c5929655dc9b97288b69cfeedad610c9a1 904713c5929655dc9b97288b69cfeedad610c9a1
names    : monolog/monolog, psr/log-implementation

support
issues : https://github.com/Seldaek/monolog/issues
source : https://github.com/Seldaek/monolog/tree/1.27.1

autoload
psr-4
Monolog\ => src/Monolog

requires
php >=5.3.0
psr/log ~1.0
```

Você pode até mesmo passar a versão do pacote, o que informará os detalhes
daquela versão específica.

```shell
php composer.phar show monolog/monolog 1.0.2
```

### Opções

* **--all:** lista todos os pacotes disponíveis em todos os seus repositórios.
* **--installed (-i):** lista os pacotes que estão instalados (esta opção está
  habilitada por padrão e se tornou obsoleta).
* **--locked:** lista os pacotes fixados do `composer.lock`.
* **--platform (-p):** lista apenas pacotes de plataforma (PHP e extensões).
* **--available (-a):** lista apenas os pacotes disponíveis.
* **--self (-s):** lista as informações do pacote raiz.
* **--name-only (-N):** lista apenas os nomes dos pacotes.
* **--path (-P):** lista os caminhos dos pacotes.
* **--tree (-t):** lista as dependências como uma árvore.
  Se você passar um nome de pacote, essa opção exibirá a árvore de dependências
  desse pacote.
* **--latest (-l):** lista todos os pacotes instalados, incluindo a sua versão
  mais recente.
* **--outdated (-o):** implica `--latest`, mas lista *apenas* os pacotes que têm
  uma versão mais recente disponível.
* **--ignore:** ignora o(s) pacote(s) especificado(s).
  Pode conter caracteres curinga (`*`).
  Use-o com a opção `--outdated` se não quiser ver informações sobre novas
  versões de alguns pacotes.
* **--no-dev:** filtra as dependências de desenvolvimento da lista de pacotes.
* **--major-only (-M):** use com `--latest` ou `--outdated`.
  Exibe apenas pacotes que possuem atualizações de versão principal compatíveis
  com o SemVer.
* **--minor-only (-m):** use com `--latest` ou `--outdated`.
  Exibe apenas os pacotes que possuem atualizações menores compatíveis com o
  SemVer.
* **--patch-only:** use com `--latest` ou `--outdated`.
  Exibe apenas pacotes que possuem atualizações de nível de patch compatíveis
  com o SemVer.
* **--sort-by-age (-A):** exibe a idade da versão instalada e classifica os
  pacotes, começando pelos mais antigos.
  Use com a opção `--latest` ou `--outdated`.
* **--direct (-D):** restringe a lista de pacotes às suas dependências diretas.
* **--strict:** retorna um código de saída diferente de zero quando há pacotes
  desatualizados.
* **--format (-f):** permite escolher entre o formato de saída de texto (padrão)
  ou JSON.
* **--ignore-platform-reqs:** ignora todos os requisitos de plataforma (`php`,
  `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina local não
  atenda a eles.
  Use com a opção `--outdated`.
* **--ignore-platform-req:** ignora um requisito de plataforma específico
  (`php`, `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina
  local não atenda a ele.
  É possível ignorar múltiplos requisitos usando curingas.
  Use com a opção `--outdated`.

## outdated

O comando `outdated` exibe uma lista de pacotes instalados que têm atualizações
disponíveis, incluindo as suas versões atuais e mais recentes.
Ele é basicamente um apelido para `composer show -lo`.

O código de cores é o seguinte:

* **verde (=)**: a dependência está na versão mais recente e atualizada.
* **amarelo (~)**: a dependência possui uma nova versão disponível, que inclui
  quebra de compatibilidade com versões anteriores conforme o SemVer, então
  atualize quando puder, mas isso pode envolver algum trabalho.
* **vermelho (!)**: a dependência possui uma nova versão compatível com o SemVer
  e você deveria atualizá-la.

### Opções

* **--all (-a):** exibe todos os pacotes, não apenas os desatualizados (apelido
  para `composer show --latest`).
* **--direct (-D):** restringe a lista de pacotes às suas dependências diretas.
* **--strict:** retorna um código de saída diferente de zero quando há pacotes
  desatualizados.
* **--ignore:** ignora o(s) pacote(s) especificado(s).
  Pode conter caracteres curinga (`*`).
  Use isso se não quiser ver informações sobre novas versões de alguns pacotes.
  Use-o se não quiser ser notificado sobre novas versões de alguns pacotes.
* **--major-only (-M):** exibe apenas pacotes que possuem atualizações de versão
  principal compatíveis com o SemVer.
* **--minor-only (-m):** exibe apenas os pacotes que possuem atualizações
  menores compatíveis com o SemVer.
* **--patch-only (-p):** exibe apenas pacotes que possuem atualizações de nível
  de patch compatíveis com o SemVer.
* **--sort-by-age (-A):** exibe a idade da versão instalada e classifica os
  pacotes, começando pelos mais antigos.
* **--format (-f):** permite escolher entre o formato de saída de texto (padrão)
  ou JSON.
* **--no-dev:** não exibe dependências de desenvolvimento desatualizadas.
* **--locked:** exibe as atualizações dos pacotes do arquivo de bloqueio,
  independentemente do que está atualmente no diretório `vendor`.
* **--ignore-platform-reqs:** ignora todos os requisitos de plataforma (`php`,
  `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina local não
  atenda a eles.
* **--ignore-platform-req:** ignora um requisito de plataforma específico
  (`php`, `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina
  local não atenda a ele.
  É possível ignorar múltiplos requisitos usando curingas.

## browse / home

O comando `browse` (ou o apelido `home`) abre a URL do repositório ou a página
do pacote no navegador.

### Opções

* **--homepage (-H):** abre a página do pacote em vez da URL do repositório.
* **--show (-s):** apenas exibe a página ou a URL do repositório.

## suggests

Lista todos os pacotes sugeridos pelo conjunto de pacotes atualmente instalado.
Você pode, opcionalmente, passar um ou vários nomes de pacotes no formato
`vendor/pacote` para limitar a saída apenas às sugestões feitas por esses
pacotes.

Use as flags `--by-package` (padrão) ou `--by-suggestion` para agrupar a saída
pelo pacote que oferece as sugestões ou pelos pacotes sugeridos,
respectivamente.

Se você quiser apenas uma lista de nomes de pacotes sugeridos, use `--list`.

### Opções

* **--by-package:** agrupa a saída por pacote que oferece a sugestão (padrão).
* **--by-suggestion:** agrupa a saída por pacote sugerido.
* **--all:** exibe sugestões de todas as dependências, incluindo as transitivas
  (por padrão, apenas as sugestões das dependências diretas são exibidas).
* **--list:** exibe apenas a lista dos nomes dos pacotes sugeridos.
* **--no-dev:** exclui as sugestões dos pacotes de `require-dev`.

## fund

Descobre como ajudar a financiar a manutenção das suas dependências.
Este comando lista todos os links de financiamento das dependências instaladas.
Use `--format=json` para obter uma saída legível por máquina.

### Opções

* **--format (-f):** permite escolher entre o formato de saída de texto (padrão)
  ou JSON.

## depends / why

O comando `depends` informa quais outros pacotes dependem de um determinado
pacote.
Assim como na instalação, os relacionamentos em `require-dev` são considerados
apenas para o pacote raiz.

```shell
php composer.phar depends doctrine/lexer
```

```text
doctrine/annotations  1.13.3 requires doctrine/lexer (1.*)
doctrine/common       2.13.3 requires doctrine/lexer (^1.0)
```

Opcionalmente, você pode especificar uma restrição de versão após o pacote para
limitar a pesquisa.

Adicione a flag `--tree` ou `-t` para mostrar uma árvore recursiva do motivo da
dependência do pacote, por exemplo:


```shell
php composer.phar depends psr/log -t
```

```text
psr/log 1.1.4 Common interface for logging libraries
├──composer/composer 2.4.x-dev (requires psr/log ^1.0 || ^2.0 || ^3.0)
├──composer/composer dev-main (requires psr/log ^1.0 || ^2.0 || ^3.0)
├──composer/xdebug-handler 3.0.3 (requires psr/log ^1 || ^2 || ^3)
│  ├──composer/composer 2.4.x-dev (requires composer/xdebug-handler ^2.0.2 || ^3.0.3)
│  └──composer/composer dev-main (requires composer/xdebug-handler ^2.0.2 || ^3.0.3)
└──symfony/console v5.4.11 (conflicts psr/log >=3) (circular dependency aborted here)
```

### Opções

* **--recursive (-r):** resolve recursivamente até o pacote raiz.
* **--tree (-t):** exibe os resultados como uma árvore aninhada, implica `-r`.

## prohibits / why-not

O comando `prohibits` informa quais pacotes estão impedindo a instalação de um
determinado pacote.
Especifique uma restrição de versão para verificar se as atualizações podem ser
executadas no seu projeto e, se não, por que não.
Observe o seguinte exemplo:

```shell
php composer.phar prohibits symfony/symfony 3.1
```

```text
laravel/framework v5.2.16 requires symfony/var-dumper (2.8.*|3.0.*)
```

Observe que você também pode especificar os requisitos de plataforma, por
exemplo, para verificar se você pode atualizar seu servidor para o PHP 8.0:

```shell
php composer.phar prohibits php 8
```

```text
doctrine/cache        v1.6.0 requires php (~5.5|~7.0)
doctrine/common       v2.6.1 requires php (~5.5|~7.0)
doctrine/instantiator 1.0.5  requires php (>=5.3,<8.0-DEV)
```

Assim como `depends`, você pode solicitar uma pesquisa recursiva, que listará
todos os pacotes que dependem dos pacotes que causam o conflito.

### Opções

* **--recursive (-r):** resolve recursivamente até o pacote raiz.
* **--tree (-t):** exibe os resultados como uma árvore aninhada, implica `-r`.

## validate

Você sempre deve executar o comando `validate` antes de fazer o commit do
arquivo `composer.json` (e do `composer.lock`
[se aplicável](01-basic-usage.md#faça-o-commit-do-arquivo-composer.lock-para-o-controle-de-versão))
e antes de criar a tag de uma versão.

Ele verificará se o seu `composer.json` é válido.
Se um arquivo `composer.lock` existir, ele também verificará se este está
atualizado em relação ao `composer.json`.

```shell
php composer.phar validate
```

### Opções

* **--no-check-all:** não emite um alerta se os requisitos do `composer.json`
  usarem restrições de versão não associadas ou excessivamente rígidas.
* **--no-check-lock:** não emite um erro se o `composer.lock` existir e não
  estiver atualizado.
* **--check-lock** verifica se o arquivo de bloqueio está atualizado (mesmo
  quando [config.lock](06-config.md#lock) é falso).
* **--no-check-publish:** não emite um erro se o `composer.json` não for
  adequado para publicação como um pacote no Packagist, mas for válido.
* **--no-check-version:** não emite um erro se o campo de versão estiver
  presente.
* **--with-dependencies:** também valida o `composer.json` de todas as
  dependências instaladas.
* **--strict:** retorna um código de saída diferente de zero para os alertas e
  erros.

## status

Se você precisar modificar frequentemente o código das suas dependências, e elas
são instaladas a partir do código-fonte, o comando `status` permitirá verificar
se há alterações locais em alguma delas.

```shell
php composer.phar status
```

Com a opção `--verbose`, você obtém mais informações sobre o que foi alterado:

```shell
php composer.phar status -v
```

```text
You have changes in the following dependencies:
vendor/seld/jsonlint:
    M README.mdown
```

## self-update / selfupdate

Para atualizar o próprio Composer para a versão mais recente, execute o comando
`self-update`.
Ele substituirá o seu `composer.phar` pela versão mais recente.

```shell
php composer.phar self-update
```

Se, em vez disso, você quiser atualizar para uma versão específica,
especifique-a:

```shell
php composer.phar self-update 2.4.0-RC1
```

Se você instalou o Composer para todo o sistema (consulte a
[instalação global](00-intro.md#globalmente)), pode ser necessário executar o
comando com privilégios de `root`.

```shell
sudo -H composer self-update
```

Se o Composer não foi instalado como um arquivo PHAR, este comando não estará
disponível.
(Isso às vezes ocorre quando o Composer foi instalado por um gerenciador de
pacotes do sistema operacional.)

Ao atualizar, o Composer avisa se a versão que você está usando ou para a qual
está atualizando chegou ao fim de sua vida útil, ou se é uma versão antiga
próxima do fim de sua vida útil que recebe apenas correções de segurança
críticas, e indica a versão estável mais recente.
Se existir uma versão mais recente do Composer, mas que exija uma versão do PHP
mais recente do que a que você está usando, o Composer informa qual versão do
PHP você precisa e que seu ambiente está usando a uma versão mais antiga.

Os backups criados para `--rollback` são armazenados no
[`data-dir`](06-config.md#data-dir), e as chaves públicas usadas para verificar
os downloads são armazenadas em `COMPOSER_HOME`.
Ambos os diretórios devem ser graváveis apenas pelo usuário proprietário da
instalação do Composer e devem ser tratados como confiáveis: um diretório
gravável por outros usuários poderia ser usado para plantar um arquivo
`composer.phar` malicioso que seria instalado posteriormente por meio de um
`self-update --rollback` com privilégios elevados.

### Opções

* **--rollback (-r):** reverte para a última versão que você instalou.
* **--clean-backups:** exclui os backups antigos durante uma atualização.
  Isso torna a versão atual do Composer o único backup disponível após a
  atualização.
* **--no-progress:** não exibe o progresso do download.
* **--update-keys:** solicita uma atualização de chave.
* **--stable:** força uma atualização para o canal estável.
* **--preview:** força uma atualização para o canal preview.
* **--snapshot:** força uma atualização para o canal snapshot.
* **--1:** força uma atualização para o canal estável, mas usa apenas as versões
  1.x.
* **--2:** força uma atualização para o canal estável, mas usa apenas as versões
  2.x.
* **--set-channel-only:** define apenas o canal como padrão e, em seguida,
  encerra a execução.

## config

O comando `config` permite editar as configurações e repositórios do Composer
tanto no arquivo local `composer.json` quanto no arquivo global `config.json`.

Além disso, permite editar a maioria das propriedades no `composer.json` local.

```shell
php composer.phar config --list
```

### Uso

`config [opções] [nome-da-configuração] [valor-da-configuração1] ... [valor-da-configuraçãoN]`

`nome-da-configuração` é um nome de opção de configuração e
`valor-da-configuração1` é um valor de configuração.
Para configurações que podem receber uma lista de valores (como
`github-protocols`), mais de um argumento `valor-da-configuração` é permitido.

Você também pode editar os valores das seguintes propriedades:

`description`, `homepage`, `keywords`, `license`, `minimum-stability`, `name`,
`prefer-stable`, `type` e `version`.

Consulte o capítulo [Config](06-config.md) para conhecer as opções de
configuração válidas.

### Opções

* **--global (-g):** opera no arquivo de configuração global localizado em
  `$COMPOSER_HOME/config.json` por padrão.
  Sem esta opção, este comando afeta o arquivo `composer.json` local ou um
  arquivo especificado por `--file`.
* **--editor (-e):** abre o arquivo `composer.json` local usando um editor de
  texto conforme definido pela variável de ambiente `EDITOR`.
  Com a opção `--global`, abre o arquivo de configuração global.
* **--auth (-a):** afeta o arquivo de configuração de autenticação (usada apenas
  para `--editor`).
* **--unset:** remove o elemento de configuração nomeado por
  `nome-da-configuração`.
* **--list (-l):** exibe a lista das variáveis de configuração atuais.
  Com a opção `--global`, lista apenas as configurações globais.
* **--file="..." (-f):** opera em um arquivo específico em vez do
  `composer.json`.
  Observe que isso não pode ser usado em conjunto com a opção `--global`.
* **--absolute:** retorna caminhos absolutos em vez de caminhos relativos ao
  buscar valores de configuração `*-dir`.
* **--json:** decodifica o valor da configuração como JSON, para uso com chaves
  `extra.*`.
* **--merge:** mescla o valor da configuração com o valor atual, para uso com
  chaves `extra.*` em conjunto com `--json`.
* **--append:** ao adicionar um repositório, anexa-o ao final da lista (menor
  prioridade) em vez de inseri-lo no início (maior prioridade).
* **--source:** exibe a origem de onde o valor de configuração foi carregado.

### Modificando repositórios

Além de modificar a seção `config`, o comando `config` também suporta alterações
na seção `repositories`, usando-o da seguinte maneira:

```shell
php composer.phar config repositories.foo vcs https://github.com/foo/bar
```

Se o seu repositório exigir mais opções de configuração, você poderá passar a
sua representação JSON:

```shell
php composer.phar config repositories.foo '{"type": "vcs", "url": "http://svn.example.org/meu-projeto/", "trunk-path": "master"}'
```

### Modificando valores extras

Além de modificar a seção `config`, o comando `config` também suporta alterações
na seção `extra`, usando-o da seguinte maneira:

```shell
php composer.phar config extra.foo.bar valor
```

Os pontos indicam aninhamento de arrays, embora seja permitida uma profundidade
máxima de 3 níveis.
O comando acima definiria
`"extra": { "foo": { "bar": "valor" } }`.

Se você tiver um valor complexo para adicionar/modificar, poderá usar as flags
`--json` e `--merge` para editar os campos extras como JSON:

```shell
php composer.phar config --json extra.foo.bar '{"baz": true, "qux": []}'
```

## repository / repo

O comando `repo` permite gerenciar repositórios no seu `composer.json`.
Ele é mais poderoso e recomendado em relação ao uso de
`composer config repositories.*` para manipular a configuração de repositórios.
Consulte a documentação sobre [Repositórios](05-repositories.md) para obter
detalhes sobre os tipos disponíveis e as opções de configuração.

### Uso

```shell
repo [opções] list
repo [opções] add [nome-do-repositório] [tipo-de-repositório] [url]
repo [opções] add [nome-do-repositório] [definição-json-do-repositório]
repo [opções] remove [nome-do-repositório]
repo [opções] set-url [nome-do-repositório] [url]
repo [opções] get-url [nome-do-repositório]
repo [opções] enable packagist.org
repo [opções] disable packagist.org
```

### Opções

* **--global (-g):** para modificar o arquivo global
  `$COMPOSER_HOME/config.json`.
* **--file (-f):** para modificar um arquivo específico em vez do
  `composer.json`.
* **--append:** para adicionar um repositório com prioridade mais baixa (por
  padrão, os repositórios são inseridos no início da lista e, portanto, têm
  prioridade maior do que os existentes).
* **--before [nome]:** para inserir o novo repositório antes de um repositório
  existente chamado `[nome]`.
* **--after [nome]:** para inserir o novo repositório após um repositório
  existente chamado `[nome]`.
  O `[nome]` deve corresponder ao nome de um repositório existente.

### Exemplos

```shell
php composer.phar repo list
php composer.phar repo add foo vcs https://github.com/acme/foo
php composer.phar repo add bar composer https://repo.packagist.com/bar
php composer.phar repo add zips '{"type":"artifact","url":"/caminho/para/o/diretório/com/zips"}'
php composer.phar repo add baz vcs https://example.org --before foo
php composer.phar repo add qux vcs https://example.org --after bar
php composer.phar repo remove foo
php composer.phar repo set-url foo https://git.example.org/acme/foo
php composer.phar repo get-url foo
php composer.phar repo disable packagist.org
php composer.phar repo enable packagist.org
```

## policy

O comando `policy` permite gerenciar políticas personalizadas de dependência e
suas fontes no seu `composer.json`, na seção `config.policy`.
Uma fonte aponta o Composer para uma URL remota que fornece o conjunto de
versões de pacotes às quais a política se aplica.
Adicionar uma fonte para uma política de dependência que ainda não existe criará
a política automaticamente.

Políticas de dependência nativas (`advisories`, `malware`, `abandoned`) não
aceitam fontes e são rejeitadas.
Para alterar suas configurações, use `composer config policy.<policy>.<field>` —
consulte a documentação de configuração de [policy](06-config.md#policy).

### Uso

```shell
policy [opções] add-source [nome-da-política] [tipo-de-fonte] [url]
policy [opções] add-source [nome-da-política] [definição-json-da-fonte]
```

Atualmente, apenas `url` é suportado como `tipo-de-fonte`, e as URLs devem
começar com `https://`.

### Opções

* **--global (-g):** para modificar o arquivo global
  `$COMPOSER_HOME/config.json`.
* **--file (-f):** para modificar um arquivo específico em vez do composer.json.

### Exemplos

```shell
php composer.phar policy add-source minha-lista url https://example.org/list.json
php composer.phar policy add-source minha-lista '{"type":"url","url":"https://example.org/list.json"}'
```

## create-project

Você pode usar o Composer para criar projetos a partir de um pacote existente.
Isso é o equivalente a executar um `git clone` ou um `svn checkout` seguido por
um `composer install` dos fornecedores.

Existem várias aplicações para isso:

1. Você pode implantar pacotes de aplicações.
2. Você pode baixar qualquer pacote e começar a desenvolver patches, por
   exemplo.
3. Projetos com várias pessoas desenvolvedoras podem usar esse recurso para
   inicializar a aplicação inicial para desenvolvimento.

Para criar um projeto usando o Composer, você pode usar o comando
`create-project`.
Passe o nome de um pacote e o diretório no qual criará o projeto.
Você também pode fornecer uma versão como terceiro argumento, caso contrário, a
versão mais recente será usada.

Se o diretório não existir, ele será criado durante a instalação.

```shell
php composer.phar create-project composer/hello-world meu-projeto
```

Também é possível executar o comando sem parâmetros em um diretório com um
arquivo `composer.json` existente para inicializar um projeto.

Por padrão, o comando procura por pacotes no [Packagist](https://packagist.org).

### Opções

* **--stability (-s):** estabilidade mínima do pacote.
  O padrão é `stable`.
* **--prefer-install:** existem duas formas de baixar um pacote: `source` e
  `dist`.
  O Composer usa `dist` por padrão.
  Se você usar `--prefer-install=source` (ou `--prefer-source`), o Composer fará
  a instalação a partir do `source`, caso ele esteja disponível.
  Isso é útil se você quiser corrigir uma falha em um projeto e obter
  diretamente um clone local da dependência.
  Para obter o comportamento legado, no qual o Composer usa `source`
  automaticamente para versões de desenvolvimento de pacotes, use
  `--prefer-install=auto`.
  Consulte também [config.preferred-install](06-config.md#preferred-install).
  O uso desta flag substituirá o valor definido na configuração.
* **--repository:** fornece um repositório personalizado para pesquisar o
  pacote, que será usado no lugar do Packagist.
  Pode ser uma URL HTTP apontando para um repositório do `composer`, um caminho
  para um arquivo `packages.json` local ou uma string JSON semelhante à string
  aceita pela chave [repositories](04-schema.md#repositories).
  Você pode usar esta opção várias vezes para configurar vários repositórios.
* **--add-repository:** adiciona um repositório personalizado ao
  `composer.json`.
  Se um arquivo de bloqueio estiver presente, ele será excluído e uma
  atualização será executada, ao invés de uma instalação.
* **--dev:** instala os pacotes listados em `require-dev`.
* **--no-dev:** ignora a instalação dos pacotes listados em `require-dev`.
* **--no-scripts:** ignora a execução dos scripts definidos no pacote raiz.
* **--no-progress:** remove a exibição de progresso que pode interferir em
  alguns terminais ou scripts que não tratam caracteres de backspace.
* **--no-secure-http:** desabilita a opção de configuração `secure-http`
  temporariamente ao instalar o pacote raiz.
  Use por sua conta e risco.
  Usar essa flag é uma má ideia.
* **--keep-vcs:** ignora a exclusão dos metadados do VCS para o projeto criado.
  Isto é útil principalmente se você executar o comando em modo não interativo.
* **--remove-vcs:** força a remoção dos metadados do VCS sem pedir confirmação.
* **--no-install:** desabilita a instalação dos fornecedores.
* **--no-audit:** não executa as etapas de auditoria após a conclusão da
  instalação.
  Consulte também [COMPOSER_NO_AUDIT](#composer-no-audit).
* **--audit-format:** formato de saída da auditoria.
  Deve ser "table", "plain", "json" ou "summary" (padrão).
* **--no-security-blocking:** OBSOLETO; use `--no-blocking` em vez disso.
  Permite instalar pacotes com avisos de segurança ou que estejam abandonados.
  Consulte também
  [COMPOSER_NO_SECURITY_BLOCKING](#composer-no-security-blocking).
* **--no-blocking:** desabilita todo o bloqueio de dependências baseado em
  políticas durante a execução deste comando.
  Consulte também [COMPOSER_NO_BLOCKING](#composer-no-blocking).
* **--ignore-platform-reqs:** ignora todos os requisitos de plataforma (`php`,
  `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina local não
  atenda a eles.
  Consulte também a opção de configuração [`platform`](06-config.md#platform).
* **--ignore-platform-req:** ignora um requisito de plataforma específico
  (`php`, `hhvm`, `lib-*` e `ext-*`) e força a instalação, mesmo que a máquina
  local não atenda a ele.
  É possível ignorar múltiplos requisitos usando curingas.
* **--require:** exige que pacotes adicionais sejam incluídos no `composer.json`
  após a instalação do projeto.
  Se um arquivo de bloqueio estiver presente, ele será excluído e uma
  atualização será executada em vez de uma instalação.
  Pode ser especificado várias vezes para múltiplos pacotes.
  Deve seguir o formato `foo/bar:1.0.0` caso você queira especificar uma
  restrição de versão.
* **--ask:** solicita o diretório de destino para o novo projeto.

## dump-autoload / dumpautoload

Se você precisar atualizar o autoloader devido a novas classes em um pacote de
mapa de classes, por exemplo, poderá usar `dump-autoload` para fazer isso sem
ter que passar por uma instalação ou atualização.

Além disso, ele pode gerar um autoloader otimizado que converte pacotes PSR-0/4
em pacotes de mapa de classes por motivos de desempenho.
Em aplicações grandes com muitas classes, o autoloader pode ocupar uma porção
substancial do tempo de cada requisição.
O uso de mapas de classes para tudo é menos conveniente durante o
desenvolvimento, mas, usando esta opção, você ainda pode usar PSR-0/4 por
conveniência e mapas de classes por desempenho.

### Opções

* **--optimize (-o):** converte o autoloading PSR-0/4 em um mapa de classes para
  obter um autoloader mais rápido.
  Isso é recomendado especialmente em produção, mas pode demorar um pouco para
  ser executado, portanto, atualmente não é feito por padrão.
* **--classmap-authoritative (-a):** faz o autoloading apenas das classes do
  mapa de classes.
  Implicitamente habilita `--optimize`.
* **--apcu:** usa a APCu para armazenar em cache as classes encontradas/não
  encontradas.
* **--apcu-prefix:** usa um prefixo personalizado para o cache do autoloader da
  APCu.
  Habilita implicitamente `--apcu`.
* **--dry-run:** exibe as operações, mas não executa nada.
* **--no-dev:** desabilita as regras de `autoload-dev`.
  Por padrão, o Composer deduzirá isso automaticamente com base no estado
  `--no-dev` da última operação de `install` ou `update`.
* **--dev:** habilita as regras de `autoload-dev`.
  Por padrão, o Composer deduzirá isso automaticamente com base no estado
  `--no-dev` da última operação de `install` ou `update`.
* **--ignore-platform-reqs:** ignora todos os requisitos `php`, `hhvm`, `lib-*`
  e `ext-*` e pula a [verificação de plataforma](07-runtime.md#platform-check)
  para eles.
  Consulte também a opção de configuração [`platform`](06-config.md#platform).
* **--ignore-platform-req:** ignora um requisito de plataforma específico
  (`php`, `hhvm`, `lib-*` e `ext-*`) e pula a
  [verificação de plataforma](07-runtime.md#platform-check) para ele.
  Múltiplos requisitos podem ser ignorados usando curingas.
* **--strict-psr:** retorna um código de saída de falha (1) se houver erros de
  mapeamento PSR-4 ou PSR-0 no projeto atual (excluindo dependências).
  Requer `--optimize` para funcionar.
* **--strict-ambiguous:** retorna um código de saída de falha (2) se a mesma
  classe for encontrada em múltiplos arquivos.
  Requer `--optimize` para funcionar.

## clear-cache / clearcache / cc

Exclui todo o conteúdo dos diretórios de cache do Composer.

### Opções

* **--gc:** executa apenas a coleta de lixo, não uma limpeza completa do cache.

## licenses

Lista o nome, a versão e a licença de cada pacote instalado.
Use `--format=json` para obter uma saída legível para máquinas.

### Opções

* **--locked:** lista as licenças do arquivo de bloqueio, independentemente do
  conteúdo atual do diretório `vendor`.
* **--format:** formato da saída: `text`, `json` ou `summary` (padrão: `text`).
* **--no-dev:** remove as dependências de desenvolvimento da saída.

## run-script / run

Para executar [scripts](articles/scripts.md) manualmente, você pode usar este
comando, fornecendo o nome do script e, opcionalmente, quaisquer argumentos
necessários.

### Opções

* **--timeout:** define o tempo limite do script em segundos ou 0 para
  desabilitar o tempo limite.
* **--dev:** habilita o modo de desenvolvimento.
* **--no-dev:** desabilita o modo de desenvolvimento.
* **--list (-l):** lista os scripts definidos pela pessoa usuária.

## exec

Executa um binário ou script de um fornecedor.
Você pode executar qualquer comando e `exec` garantirá que o diretório `bin-dir`
do Composer seja adicionado à variável `PATH` antes da execução do comando.

### Opções

* **--list (-l):** lista os binários disponíveis no Composer.

## diagnose

Se você achar que encontrou uma falha, ou se algo está se comportando de forma
estranha, convém executar o comando `diagnose` para realizar verificações
automatizadas de muitos problemas comuns.

```shell
php composer.phar diagnose
```

## archive

Este comando é usado para gerar um arquivo compactado `zip`/`tar` para um
determinado pacote numa determinada versão.
Também pode ser usado para arquivar o seu projeto inteiro sem os arquivos
excluídos/ignorados.

```shell
php composer.phar archive vendor/pacote 2.0.21 --format=zip
```

### Opções

* **--format (-f):** formato do arquivo compactado resultante: `tar`, `tar.gz`,
  `tar.bz2` ou `zip` (padrão: `tar`).
* **--dir:** salva o arquivo compactado neste diretório (padrão: `.`).
* **--file:** salva o arquivo compactado com o nome especificado.

## audit

Este comando é usado para auditar os pacotes instalados em relação a políticas
de dependência definidas, como avisos de segurança.
Ele verifica e lista avisos de vulnerabilidade de segurança usando a
[API do Packagist.org](https://packagist.org/apidoc#list-security-advisories)
por padrão, ou outros repositórios se especificados na seção `repositories` do
`composer.json`.
O comando também detecta pacotes abandonados, pacotes marcados como malware ou
pacotes que se enquadram em outras políticas de dependência.

O comando `audit` determina se existem pacotes vulneráveis, abandonados, malware
ou pacotes que se enquadram em outras políticas de dependência e retorna os
seguintes códigos de saída com base nos resultados:

* `0`: nenhum problema encontrado.
* `1`: pacotes encontrados que se enquadram nas políticas de dependência ou
  falha devido à ausência de pacotes obrigatórios.

```shell
php composer.phar audit
```

### Opções

* **--no-dev:** desabilita a auditoria de pacotes listados em `require-dev`.
* **--format (-f):** formato de saída da auditoria.
  Deve ser "table" (padrão), "plain", "json" ou "summary".
* **--locked:** audita pacotes a partir do arquivo de bloqueio,
  independentemente do que está atualmente no diretório `vendor`.
* **--abandoned:** comportamento em relação a pacotes abandonados.
  Deve ser "ignore", "report" ou "fail".
  Consulte também [config.audit.abandoned](06-config.md#abandoned).
  Passar esta flag substituirá o valor da configuração e a variável de ambiente.
* **--ignore-severity:** ignora avisos de um determinado nível de gravidade.
  Pode ser passada uma ou mais vezes para ignorar múltiplos níveis de gravidade.

## help

Para obter mais informações sobre um determinado comando, você pode usar `help`.

```shell
php composer.phar help install
```

## Preenchimento automático na linha de comando

O preenchimento automático na linha de comando pode ser habilitado executando o
comando `composer completion --help` e seguindo as instruções.

## Variáveis de ambiente

Você pode definir várias variáveis de ambiente que substituem determinadas
configurações.
Sempre que possível, recomenda-se especificar essas configurações na seção
`config` do `composer.json`.
Vale ressaltar que as variáveis de ambiente sempre terão precedência sobre os
valores especificados no `composer.json`.

### COMPOSER

Ao definir a variável de ambiente `COMPOSER`, é possível definir o nome do
arquivo `composer.json` como algum outro.

Por exemplo:

```shell
COMPOSER=outro-composer.json php composer.phar install
```

O arquivo de bloqueio gerado usará o mesmo nome: `outro-composer.lock` neste
exemplo.

### COMPOSER_ALLOW_SUPERUSER

Se definida como `1`, esta variável de ambiente desabilita o alerta sobre a
execução de comandos como root/superusuário.
Ela também desabilita a limpeza automática de sessões sudo; portanto, você
realmente deve defini-la apenas se usar o Composer como superusuário o tempo
todo, como em contêineres Docker.

### COMPOSER_ALLOW_UNSAFE_PHAR_METADATA

Esta variável de ambiente só tem efeito em versões do PHP anteriores à 8.0.
Nessas versões, o Composer recusa-se a ler ou extrair arquivos de distribuição
`tar`/`phar`, pois processar tal arquivo não é seguro ao lidar com entradas não
confiáveis no PHP < 8.0.
A solução recomendada é atualizar para o PHP 8.0 ou superior.
Se você não puder atualizar e aceitar o risco, defina isso como `1` para
permitir que o Composer processe esses arquivos mesmo assim.
O PHP 8.0+ não é afetado e ignora essa configuração.

### COMPOSER_ALLOW_XDEBUG

Se definida como `1`, esta variável de ambiente permite executar o Composer
quando a extensão Xdebug estiver habilitada, sem reiniciar o PHP sem a extensão.

### COMPOSER_AUTH

A variável `COMPOSER_AUTH` permite configurar a autenticação como uma variável
de ambiente.
O conteúdo da variável deve ser um objeto JSON contendo objetos
[`http-basic`, `github-oauth`, `bitbucket-oauth`, ..., conforme necessário](articles/authentication-for-private-packages.md)
e seguindo as [especificações da configuração](06-config.md).

### COMPOSER_BIN_COMPAT

Substitui a configuração [`bin-compat`](06-config.md#bin-compat).

### COMPOSER_BIN_DIR

Ao definir esta opção, você pode alterar o diretório `bin`
([Binários dos fornecedores](articles/vendor-binaries.md)) para algo diferente
de `vendor/bin`.

### COMPOSER_CACHE_DIR

A variável `COMPOSER_CACHE_DIR` permite alterar o diretório de cache do
Composer, que também é configurável através da opção
[`cache-dir`](06-config.md#cache-dir).

Por padrão, ela aponta para `C:\Users\<usuário>\AppData\Local\Composer` (ou
`%LOCALAPPDATA%\Composer`) no Windows.
Em sistemas \*nix que seguem as
[Especificações de Diretório Base XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html),
ela aponta para `$XDG_CACHE_HOME/composer`.
Em outros sistemas \*nix e no macOS, ela aponta para `$COMPOSER_HOME/cache`.

### COMPOSER_CAFILE

Ao definir esta variável de ambiente, é possível definir um caminho para um
arquivo de pacote de certificados que será usado durante a verificação por par
SSL/TLS.

### COMPOSER_DISABLE_XDEBUG_WARN

Se definida como `1`, esta variável suprime o alerta de quando o Composer está
sendo executado com a extensão Xdebug habilitada.

### COMPOSER_DISCARD_CHANGES

Esta variável controla a opção de configuração
[`discard-changes`](06-config.md#discard-changes).

### COMPOSER_FUND

Se definida como `0`, esta variável de ambiente suprime os avisos de
financiamento durante a instalação.

### COMPOSER_HOME

A variável de ambiente `COMPOSER_HOME` permite alterar o diretório inicial do
Composer.
Este é um diretório global oculto (por usuário na máquina) compartilhado entre
todos os projetos.

Use `composer config --global home` para consultar a localização do diretório
home.

Por padrão, ela aponta para `C:\Users\<usuário>\AppData\Roaming\Composer` no
Windows, e `/Users/<usuário>/.composer` no macOS.
Em sistemas \*nix que seguem as
[Especificações de Diretório Base do XDG](https://specifications.freedesktop.org/basedir-spec/basedir-spec-latest.html),
ela aponta para `$XDG_CONFIG_HOME/composer`.
Em outros sistemas \*nix, ela aponta para `/home/<usuário>/.composer`.

Este diretório contém as chaves públicas usadas para verificar os downloads do
Composer durante o `self-update`; portanto, ele deve permitir escrita apenas
para o usuário proprietário da instalação do Composer e deve ser tratado como um
local confiável.

#### COMPOSER_HOME/config.json

Você pode colocar um arquivo `config.json` no diretório para o qual
`COMPOSER_HOME` aponta.
O Composer mesclará parcialmente (apenas as chaves `config` e `repositories`)
esta configuração com o arquivo `composer.json` do seu projeto ao executar os
comandos `install` e `update`.

Este arquivo permite definir [repositórios](05-repositories.md) e
[configurações](06-config.md) para os seus projetos.

Caso a configuração global corresponda à configuração _local_, a configuração
_local_ no `composer.json` do projeto sempre vence.

### COMPOSER_HTACCESS_PROTECT

O padrão é `1`.
Se definida como `0`, o Composer não criará arquivos `.htaccess` nos diretórios
home, cache e data do Composer.

### COMPOSER_MEMORY_LIMIT

Se definida, o valor é usado como `memory_limit` do PHP.

### COMPOSER_MIRROR_PATH_REPOS

Se definida como `1`, esta variável de ambiente altera a estratégia padrão do
repositório de caminhos para `mirror`, em vez de `symlink`.
Por ser a estratégia padrão definida, ela ainda pode ser substituída pelas
opções do repositório.

### COMPOSER_NO_INTERACTION

Se definida como `1`, esta variável de ambiente fará o Composer se comportar
como se você passasse a flag `--no-interaction` para todos os comandos.
Ela pode ser definida em servidores de construção/CI.

### COMPOSER_PROCESS_TIMEOUT

Esta variável de ambiente controla o tempo que o Composer espera por comandos
(como comandos do git) antes de finalizar a execução.
O valor padrão é 300 segundos (5 minutos).

### COMPOSER_ROOT_VERSION

Ao definir esta variável de ambiente, você pode especificar a versão do pacote
raiz, se ela não puder ser deduzida a partir das informações do VCS e não
estiver presente no `composer.json`.

### COMPOSER_VENDOR_DIR

Ao definir esta variável de ambiente, você pode fazer o Composer instalar as
dependências em um diretório diferente de `vendor`.

### COMPOSER_RUNTIME_ENV

Isso permite indicar em qual ambiente o Composer está sendo executado, o que
pode ajudar o Composer a contornar alguns problemas específicos do ambiente.
O único valor atualmente suportado é `virtualbox`, que ativa algumas chamadas
rápidas de `sleep()` para aguardar que o sistema de arquivos conclua a gravação
dos arquivos antes de tentar lê-los.
Você pode definir essa variável de ambiente se usar o Vagrant ou VirtualBox e
enfrentar problemas com arquivos que não são encontrados durante a instalação,
apesar de estarem presentes.

### http_proxy ou HTTP_PROXY
### HTTP_PROXY_REQUEST_FULLURI
### HTTPS_PROXY_REQUEST_FULLURI
### no_proxy ou NO_PROXY

Consulte a
[documentação sobre proxy](faqs/how-to-use-composer-behind-a-proxy.md) para
obter mais detalhes sobre como usar variáveis de ambiente de proxy.

### COMPOSER_MAX_PARALLEL_HTTP

Defina como um inteiro para configurar quantos arquivos podem ser baixados em
paralelo.
O padrão é `12` e deve estar entre `1` e `50`.
Se o seu proxy tiver problemas com concorrência, talvez você queira diminuir
este valor.
Aumentá-lo geralmente não resulta em ganhos de desempenho.

### COMPOSER_MAX_PARALLEL_PROCESSES

Defina como um número inteiro para configurar quantos processos podem ser
executados em paralelo.
O valor padrão é `10`, e deve estar entre `1` e `50`.

### COMPOSER_IPRESOLVE

Defina como `4` ou `6` para forçar a resolução de DNS via IPv4 ou IPv6.
Isso funciona apenas quando a extensão curl é usada para downloads.

### COMPOSER_SELF_UPDATE_TARGET

Se definida, faz com que o comando `self-update` salve o novo arquivo PHAR do
Composer neste caminho em vez de sobrescrever-se.
Útil para atualizar o Composer em sistemas de arquivos somente leitura.

### COMPOSER_DISABLE_NETWORK

Se definida como `1`, desabilita o acesso à rede (melhor esforço).
Esta variável pode ser usada para depurar ou executar o Composer em um avião ou
nave espacial com conectividade ruim.

Se definida como `prime`, os repositórios VCS do GitHub irão preparar o cache
para que ele possa ser usado totalmente offline com `1`.

### COMPOSER_DEBUG_EVENTS

Se definida como `1`, exibe informações sobre os eventos que estão sendo
disparados, o que pode ser útil para as pessoas autoras de plugin identificarem
o que está disparando e quando exatamente.

### COMPOSER_SKIP_SCRIPTS

Aceita uma lista de nomes de eventos separados por vírgula (por exemplo,
`post-install-cmd`) para os quais a execução de scripts deve ser ignorada.

### COMPOSER_NO_AUDIT

Se definida como `1`, equivale a passar a opção `--no-audit` para um comando
`require`, `update`, `remove` ou `create-project`.

### COMPOSER_AUDIT_ABANDONED

Defina como `ignore`, `report` ou `fail` para substituir a opção de configuração
[policy.abandoned.audit](06-config.md#audit).
Não tem efeito quando `policy.abandoned` está definido como `false` no
`composer.json`.

### COMPOSER_POLICY

Interruptor principal da política de dependências.
Defina como `0` para desabilitar a aplicação de todas as políticas de
dependência durante atualizações, instalações e auditorias, ou como `1` para
ativá-la.
Definir essa variável como `1` fará com que a configuração de política do
`composer.json` seja usada.
Se quiser alterar o valor da configuração, use `composer config policy 1` em vez
disso.

Quando definida como `0`, todas as substituições específicas de política
listadas abaixo são ignoradas — toda a configuração de política de dependência é
desabilitada.

### COMPOSER_NO_BLOCKING

Se definida como `1`, equivale a passar a opção `--no-blocking` para um comando
`require`, `update`, `remove`, `install` ou `create-project`.
Isso desabilita todos os bloqueios de dependências baseados em políticas.
Essa opção substitui a configuração `block` de cada política de dependência
configurada, como, por exemplo, [policy.advisories.block](06-config.md#block).

### COMPOSER_NO_SECURITY_BLOCKING

OBSOLETA; use [COMPOSER_NO_BLOCKING](#composer-no-blocking) em vez disso.

Se definida como `1`, equivale a passar a opção `--no-security-blocking` para um
comando `require`, `update`, `remove`, `install` ou `create-project`.
Isso permite a instalação de pacotes que possuem avisos de segurança ou que
foram abandonados.
Ela substitui a opção de configuração
[policy.advisories.block](06-config.md#block).

### COMPOSER_POLICY_ADVISORIES_BLOCK

Se definida como `1`, habilita o bloqueio de pacotes com avisos de segurança
durante a resolução de dependências (equivalente a definir
`policy.advisories.block` como `true`).
Se definida como `0`, desabilita o bloqueio.

### COMPOSER_POLICY_MALWARE_BLOCK

Se definida como `1`, habilita o bloqueio de pacotes sinalizados como malware
durante a resolução de dependências (equivalente a definir
`policy.malware.block` como `true`).
Se definida como `0`, desabilita o bloqueio.

### COMPOSER_POLICY_ABANDONED_BLOCK

Se definida como `1`, habilita o bloqueio de pacotes abandonados durante a
resolução de dependências (equivalente a definir `policy.abandoned.block` como
`true`).
Se definida como `0`, desabilita o bloqueio.

Este valor tem precedência sobre o valor da variável legada
[COMPOSER_SECURITY_BLOCKING_ABANDONED](#composer-security-blocking-abandoned)
quando esta estiver definida com um valor diferente.

### COMPOSER_SECURITY_BLOCKING_ABANDONED

OBSOLETA; use
[COMPOSER_POLICY_ABANDONED_BLOCK](#composer-policy-abandoned-block) em vez
disso.

Se definida como `1`, habilita o bloqueio de pacotes abandonados durante a
resolução de dependências (equivalente a definir a configuração
`audit.block-abandoned` como `true`).
Se definida como `0`, desabilita o bloqueio de pacotes abandonados.
Ela substitui a opção de configuração
[audit.block-abandoned](06-config.md#block-abandoned).

### COMPOSER_NO_DEV

Se definida como `1`, equivale a passar a opção `--update-no-dev` para o comando
`require` ou a opção `--no-dev` para os comandos `install` ou `update`.
Você pode substituir isso para um comando específico definindo
`COMPOSER_NO_DEV=0`.

### COMPOSER_PREFER_STABLE

Se definida como `1`, equivale a passar a opção `--prefer-stable` para os
comandos `update` ou `require`.

### COMPOSER_PREFER_LOWEST

Se definida como `1`, equivale a passar a opção `--prefer-lowest` para `update`
ou `require`.

### COMPOSER_PREFER_DEV_OVER_PRERELEASE

Se definida como `1`, ao resolver dependências com `--prefer-stable` e
`--prefer-lowest` habilitados, versões de desenvolvimento são tratadas como mais
estáveis do que versões alpha/beta/RC nos casos em que não existe uma versão
estável.
Isso é útil para testar as versões mais antigas (mínimas), mantendo a
preferência por branches que podem conter correções críticas em vez de versões
de pré-lançamento.

### COMPOSER_MINIMAL_CHANGES

Se definida como `1`, é o equivalente a passar a opção `--minimal-changes` para
`update`, `require` ou `remove`.

### COMPOSER_IGNORE_PLATFORM_REQ ou COMPOSER_IGNORE_PLATFORM_REQS

Se `COMPOSER_IGNORE_PLATFORM_REQS` estiver definido como `1`, é o equivalente a
passar o argumento `--ignore-platform-reqs`.

Caso contrário, especificar uma lista separada por vírgulas em
`COMPOSER_IGNORE_PLATFORM_REQ` ignorará esses requisitos específicos.

Por exemplo, se uma estação de trabalho de desenvolvimento nunca executar
consultas de banco de dados, isso pode ser usado para ignorar o requisito de que
as extensões de banco de dados estejam disponíveis.
Se você definir `COMPOSER_IGNORE_PLATFORM_REQ=ext-oci8`, o Composer permitirá
que pacotes sejam instalados mesmo que a extensão PHP `oci8` não esteja
habilitada.

### COMPOSER_WITH_DEPENDENCIES

Se definida como `1`, é equivalente a passar a opção `--with-dependencies` para
`update`, `require` ou `remove`.

### COMPOSER_WITH_ALL_DEPENDENCIES

Se definida como `1`, é equivalente a passar a opção `--with-all-dependencies`
para `update`, `require` ou `remove`.

### SHELL_VERBOSITY

Como o Composer usa o [symfony/console](https://github.com/symfony/console),
você pode definir o
[nível de verbosidade](https://symfony.com/doc/current/console/verbosity.html).

`SHELL_VERBOSITY=-1` para ocultar a saída do Composer (isso é equivalente a usar
a opção `--quiet` da linha de comando).

Observe que isso se aplicará a todas as ferramentas que dependem de
`symfony/console`.
Você pode definir `SHELL_VERBOSITY=0` após as chamadas ao Composer para
restaurar o nível de detalhamento padrão.

&larr; [Bibliotecas](02-libraries.md) | [Esquema](04-schema.md) &rarr;
