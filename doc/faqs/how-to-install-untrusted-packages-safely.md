---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/faqs/how-to-install-untrusted-packages-safely.md
source_revision: 9da1948585f11e5af594d1fece682947f1b1fa99
translation_status: ready
---

# Como instalo pacotes não confiáveis com segurança? É seguro executar o Composer como superusuário ou root?

## Por que estou vendo um alerta/erro dizendo "Do not run Composer as root/super user"?

Sempre foi desaconselhado executar o Composer como root, pelos motivos
detalhados abaixo.

A partir da versão 2.4.2 do Composer, os plugins são desabilitados
automaticamente quando o programa é executado como root e não há indicação de
que a pessoa usuária esteja fazendo isso conscientemente.
Existem duas maneiras de conceder esse consentimento:

- Se você executar de forma interativa, o Composer perguntará se você tem
  certeza de que deseja continuar a execução como root.
  Se executar de forma não interativa, os plugins serão desabilitados, a menos
  que...
- Se você definir a variável de ambiente
  [`COMPOSER_ALLOW_SUPERUSER`](../03-cli.md#composer-allow-superuser) como `1`,
  isso também indica que você pretendia executar o Composer como root e aceita
  os riscos associados a essa ação.

## É seguro executar o Composer como superusuário ou root?

Certos comandos do Composer, incluindo `exec`, `install` e `update`, permitem a
execução de código de terceiros no seu sistema.
Isso ocorre devido aos recursos de "plugins" e "scripts".
Plugins e scripts têm acesso total à conta de usuário que executa o Composer.
Por isso, recomenda-se fortemente **evitar executar o Composer como
superusuário/root**.
Todos os comandos também disparam eventos que podem ser capturados por plugins;
portanto, a menos que sejam explicitamente desabilitados, os plugins instalados
serão carregados/executados por **qualquer** comando do Composer.

Você pode desativar plugins e scripts durante a instalação ou atualização de
pacotes usando a seguinte sintaxe, garantindo que apenas o código do Composer, e
nenhum código de terceiros, seja executado:

```shell
php composer.phar install --no-plugins --no-scripts ...
php composer.phar update --no-plugins --no-scripts ...
```

Dependendo do sistema operacional, já foram observados casos em que é possível
acionar a execução de arquivos no repositório usando um arquivo `composer.json`
especialmente elaborado.
Portanto, de modo geral, se você precisar instalar dependências não confiáveis,
deve isolá-las completamente em um contêiner ou ambiente equivalente.

Observe também que o comando `exec` sempre executará código de terceiros com as
permissões do usuário que está executando o `composer`.

Consulte a variável de ambiente
[`COMPOSER_ALLOW_SUPERUSER`](../03-cli.md#composer-allow-superuser) para mais
informações sobre como desativar os alertas.

## Executando o Composer dentro de contêineres Docker/Podman

O Composer tenta detectar se está sendo executado em um contêiner e, caso
positivo, permite a execução como root sem problemas adicionais.
No entanto, se essa detecção falhar, você verá avisos e os plugins serão
desativados, a menos que você defina a variável de ambiente
[`COMPOSER_ALLOW_SUPERUSER`](../03-cli.md#composer-allow-superuser).
