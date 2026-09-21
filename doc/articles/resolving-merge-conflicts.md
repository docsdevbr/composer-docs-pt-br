---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/resolving-merge-conflicts.md
source_revision: 2a994c9913a282d072a25c623d512fb33fd5354b
translation_status: ready

tagline: Sobre resolver conflitos com elegância durante o merge
---

# Resolvendo conflitos de merge

Ao trabalhar em equipe no mesmo projeto Composer, você eventualmente se deparará
com uma situação em que várias pessoas adicionaram, atualizaram ou removeram
algo nos arquivos `composer.json` e `composer.lock` em diferentes branches.
Quando esses branches forem finalmente mesclados, surgirão conflitos de merge.
Resolver esses conflitos não é tão simples quanto em outros arquivos,
especialmente no caso do arquivo `composer.lock`.

> **Nota:** Pode não ficar imediatamente óbvio por que o merge baseada em texto
> não é possível para arquivos de bloqueio; então, vamos imaginar o seguinte
> exemplo, no qual queremos fazer o merge de dois branches:
>
> - O branch 1 adicionou o pacote A, que requer o pacote B.
>   O pacote B está travado na versão `1.0.0`.
> - O branch 2 adicionou o pacote C, que entra em conflito com todas as versões
>   do pacote B anteriores à `1.2.0`.
>
> Um merge baseado em texto resultaria no pacote A versão `1.0.0`, pacote B
> versão `1.0.0` e pacote C versão `1.0.0`.
> Esse é um resultado inválido, pois o conflito do pacote C não foi considerado
> e exigiria uma atualização do pacote B.

## 1. Reaplicando alterações

O método mais seguro para fazer o merge de arquivos do Composer é aceitar a
versão de um dos branches e aplicar as alterações do outro branch.

Um exemplo envolvendo dois branches:

1. O pacote 'A' foi adicionado.
2. O pacote 'B' foi removido e o pacote 'C' foi adicionado.

Para resolver o conflito ao fazer o merge desses dois branches:

- Escolhemos o branch que contém mais alterações e aceitamos os arquivos
  `composer.json` e `composer.lock` desse branch.
  Neste caso, escolhemos os arquivos do Composer do branch 2.
- Reaplicamos as alterações do outro branch (branch 1).
  Neste caso, precisamos executar `composer require package/A` novamente.

## 2. Validando seus arquivos mesclados

Antes de fazer o commit, certifique-se de que os arquivos `composer.json` e
`composer.lock` resultantes sejam válidos.
Para isso, execute os seguintes comandos:

```shell
php composer.phar validate
php composer.phar install [--dry-run]
```

## Automatizando a resolução de conflitos de merge com o Git

A resolução de conflitos do Git _poderia_ ser aprimorada com o uso de um driver
de merge personalizado.

Um exemplo disso pode ser encontrado no
[driver de merge do Composer para Git, de balbuf](https://github.com/balbuf/composer-git-merge-driver).

### Lidando com casos triviais

Em um pequeno número de casos, apenas o `content-hash` apresentará conflito,
pois o sistema de controle de versão pode conseguir realizar o merge do restante
do texto do arquivo sem problemas.
Isso geralmente ocorre quando dois pacotes diferentes foram adicionados ou
atualizados em cada lado do merge, sem sobreposição ou conflito de dependências.
Quando isso acontece, executar `composer update --lock` pode ser suficiente para
remover a marcação de conflito e atualizar o hash do arquivo de bloqueio.
Você também pode executar qualquer outra variante do comando `composer update`
para remover a marcação de conflito e, potencialmente, atualizar os pacotes.

## Considerações importantes

Lembre-se de que, sempre que ocorrerem conflitos de merge no arquivo de
bloqueio, perde-se a informação sobre a versão exata na qual os novos pacotes
foram fixados em um dos branches.
Quando o pacote A no branch 1 está restrito a `^1.2.0` e fixado na versão
`1.2.0`, ele pode ser atualizado se o branch 2 for usado como base e um novo
comando `composer require package/A:^1.2.0` for executado; isso ocorre porque o
comando utilizará a versão mais recente permitida pela restrição, sempre que
possível.
É possível que a versão 1.3.0 desse pacote já esteja disponível, passando então
a ser utilizada em vez da anterior.

A escolha correta das [restrições de versão](../articles/versions.md) e a
garantia de que os pacotes sigam o
[versionamento semântico](https://semver.org/) ao utilizar
[operadores de próxima versão significativa](versions.md#operadores-de-próxima-versão-significativa)
devem assegurar que o merge de branches não cause falhas devido à atualização
acidental de uma dependência.

## Recuperação de conflitos de merge resolvidos incorretamente

Se as etapas acima não forem seguidas e merges baseados em texto forem
realizados mesmo assim, seu projeto Composer pode acabar em um estado que
apresenta comportamento inesperado, pois o arquivo `composer.lock` não está
(totalmente) sincronizado com o arquivo `composer.json`.

Duas situações podem ocorrer nesse caso:

1. Existem pacotes nas seções `require` ou `require-dev` do arquivo
   `composer.json` que não constam no arquivo de bloqueio e, consequentemente,
   nunca são instalados.

   > **Nota:** A partir da versão 2.5 do Composer, a presença de pacotes
   > requeridos, mas não estão no `composer.lock`, resulta em erro ao executar o
   > comando `install`.

2. Existem pacotes no arquivo `composer.lock` que não são dependências diretas
   ou indiretas de nenhum dos pacotes requeridos.
   Como resultado, um pacote acaba sendo instalado, mesmo que o comando
   `composer why vendor/package` indique que ele não é requerido.

Existem várias maneiras de corrigir esses problemas.

### A. Começar do zero

A opção mais simples, porém de maior impacto, é executar um `composer update`
para chegar ao estado correto partindo do zero.

Uma desvantagem dessa abordagem é que as versões de pacotes anteriormente
fixadas serão atualizadas, uma vez que as informações sobre as versões
anteriores foram perdidas.
Se todas as suas dependências seguirem o
[versionamento semântico](https://semver.org/) e suas
[restrições de versão](../articles/versions.md) utilizarem
[operadores de próxima versão significativa](versions.md#operadores-de-próxima-versão-significativa),
isso não deve ser um problema; caso contrário, você poderá quebrar sua aplicação
inadvertidamente.

### B. Reconstruir a partir do histórico do Git

Uma opção que provavelmente não é muito viável em muitas situações, mas que
merece uma menção honrosa:

Deve ser possível reconstruir o estado correto dos pacotes voltando no histórico
do Git, localizando o arquivo `composer.lock` válido mais recente e adicionando
novamente as novas dependências a partir desse ponto.

### C. Resolver problemas manualmente

Existe uma opção para corrigir uma divergência entre os arquivos `composer.json`
e `composer.lock` sem precisar vasculhar o histórico do Git ou começar do zero.
Para isso, precisamos resolver os problemas 1 e 2 separadamente.

#### 1. Detectar e corrigir pacotes necessários ausentes

Para detectar qualquer pacote que seja necessário, mas não esteja instalado,
basta executar:

```shell
php composer.phar validate
```

Se houver pacotes necessários, mas não instalados, você deverá obter uma saída
semelhante a esta:

```shell
./composer.json is valid but your composer.lock has some errors
# Lock file errors
- Required package "vendor/package-name" is not present in the lock file.
This usually happens when composer files are incorrectly merged or the composer.json file is manually edited.
Read more about correctly resolving merge conflicts https://getcomposer.org/doc/articles/resolving-merge-conflicts.md
and prefer using the "require" command over editing the composer.json file directly https://getcomposer.org/doc/03-cli.md#require
```

Para corrigir isso, basta executar `composer update vendor/package-name` para
cada pacote listado aqui.
Após fazer isso para cada pacote listado, a execução de `composer validate`
novamente não deve apresentar erros no arquivo de bloqueio:

```shell
./composer.json is valid
```

#### 2. Detectar e corrigir pacotes supérfluos

Para detectar e corrigir pacotes que estão travados, mas não são dependências
diretas ou indiretas, você pode executar o seguinte comando:

```shell
php composer.phar remove --unused
```

Se não houver pacotes travados que não sejam dependências, o comando apresentará
a seguinte saída:

```shell
No unused packages to remove
```

Se houver pacotes a serem removidos, a saída será a seguinte:

```shell
vendor/package-name is not required in your composer.json and has not been removed
./composer.json has been updated
Running composer update vendor/package-name
Loading composer repositories with package information
Updating dependencies
Lock file operations: 0 installs, 0 updates, 1 removal
  - Removing vendor/package-name (1.0)
Writing lock file
Installing dependencies from lock file (including require-dev)
Nothing to install, update or remove
```
