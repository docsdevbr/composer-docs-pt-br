---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/handling-private-packages.md
source_revision: d1167b6348aa64aa8df94a43df8bb78afcf448ac
translation_status: ready

tagline: Hospedando e instalando pacotes privados do Composer
---

# Gerenciando pacotes privados

## Private Packagist

O [Private Packagist](https://packagist.com) é um produto comercial de
hospedagem de pacotes que oferece suporte profissional, gerenciamento via web de
pacotes públicos e privados, além de permissões de acesso granulares.
O Private Packagist realiza o espelhamento de arquivos ZIP dos pacotes, tornando
as instalações mais rápidas e independentes de sistemas de terceiros — por
exemplo, você pode realizar uma implantação mesmo que o GitHub esteja fora do
ar, pois seus arquivos ZIP estão espelhados.

O Private Packagist está disponível como uma solução SaaS hospedada ou como um
pacote auto-hospedado on-premise, oferecendo uma experiência de configuração
interativa.

Parte da receita do Private Packagist é utilizada para financiar o
desenvolvimento e a hospedagem do Composer e do Packagist.org; portanto,
utilizá-lo é uma ótima maneira de apoiar financeiramente a manutenção desses
projetos de código aberto.
Você pode encontrar mais informações sobre como configurar seu próprio
repositório de pacotes em [Packagist.com](https://packagist.com).

## Satis

Por outro lado, o Satis é de código aberto, mas funciona apenas como um gerador
de repositórios `composer` estáticos.
Ele é, de certa forma, uma versão ultraleve e baseada em arquivos estáticos do
Packagist, podendo ser utilizado para hospedar os metadados de pacotes privados
da sua empresa ou de projetos pessoais.
Você pode instalá-lo usando o
[Composer](https://github.com/composer/satis?tab=readme-ov-file#run-from-source)
ou o
[Docker](https://github.com/composer/satis?tab=readme-ov-file#run-as-docker-container).

### Configuração

Por exemplo, suponha que você tenha alguns pacotes que deseja reutilizar em toda
a sua empresa, mas não quer torná-los código aberto.
Primeiro, você definiria uma configuração do Satis: um arquivo JSON que lista os
seus [repositórios](../05-repositories.md) selecionados.

O nome de arquivo padrão é `satis.json`, mas pode ser qualquer nome de sua
preferência.

Aqui está um exemplo de configuração: observe que ela contém alguns repositórios
VCS, mas poderiam ser quaisquer tipos de [repositórios](../05-repositories.md).
Ela utiliza `"require-all": true`, o que seleciona todas as versões de todos os
pacotes nos repositórios que você definiu.

O arquivo padrão que o Satis procura é o `satis.json`, localizado na raiz do
repositório.

```json
{
    "name": "my/repository",
    "homepage": "http://packages.example.org",
    "repositories": [
        { "type": "vcs", "url": "https://github.com/mycompany/privaterepo" },
        { "type": "vcs", "url": "http://svn.example.org/private/repo" },
        { "type": "vcs", "url": "https://github.com/mycompany/privaterepo2" }
    ],
    "require-all": true
}
```

Se você quiser selecionar manualmente os pacotes desejados, pode listar todos
eles no seu repositório Satis dentro da chave `require` padrão do Composer,
utilizando a restrição `"*"` para garantir que todas as versões sejam
selecionadas — ou outra restrição, caso precise de versões específicas.

```json
{
    "repositories": [
        { "type": "vcs", "url": "https://github.com/mycompany/privaterepo" },
        { "type": "vcs", "url": "http://svn.example.org/private/repo" },
        { "type": "vcs", "url": "https://github.com/mycompany/privaterepo2" }
    ],
    "require": {
        "company/package": "*",
        "company/package2": "*",
        "company/package3": "2.0.0"
    }
}
```

Depois de fazer isso, você executa:

```shell
php bin/satis build <configuration file> <build dir>
```

Após ajustar esse processo, o procedimento padrão seria executar esse comando
como uma tarefa do cron em um servidor.
Isso atualizaria todas as informações dos seus pacotes, de forma semelhante ao
que o Packagist faz.

Vale ressaltar que, se seus pacotes privados estiverem hospedados no GitHub, o
servidor precisará de uma chave SSH que conceda acesso a eles; além disso, você
deve adicionar a flag `--no-interaction` (ou `-n`) ao comando para garantir que
ele utilize a autenticação por chave SSH em vez de solicitar uma senha.
Essa também é uma boa estratégia para servidores de integração contínua.

Configure um host virtual que aponte para o diretório `web/`, por exemplo,
`packages.example.org`.
Alternativamente, se estiver usando PHP >= 5.4.0, você pode utilizar o servidor
CLI embutido (`php -S localhost:port -t satis-output-dir/`) como uma solução
temporária.

### Atualizações parciais

Você pode instruir o Satis a atualizar seletivamente apenas pacotes específicos
ou a processar apenas um repositório com uma determinada URL.
Isso reduz o tempo necessário para reconstruir o arquivo `package.json` e é útil
caso você utilize webhooks (personalizados) para acionar reconstruções sempre
que código for enviado para um de seus repositórios.

Para reconstruir apenas pacotes específicos, informe os nomes dos pacotes na
linha de comando, desta forma:

```shell
php bin/satis build satis.json web/ this/package that/other-package
```

Observe que isso ainda exigirá buscar e escanear todos os seus repositórios VCS,
pois qualquer repositório VCS pode conter (em qualquer branch) um dos pacotes
selecionados.

Se você quiser escanear apenas o pacote selecionado, e não todos os repositórios
VCS, precisará declarar um *nome* para todos os seus pacotes (isso funciona
apenas para repositórios do tipo VCS):

```json
{
    "repositories": [
        { "name": "company/privaterepo", "type": "vcs", "url": "https://github.com/mycompany/privaterepo" },
        { "name": "private/repo", "type": "vcs", "url": "http://svn.example.org/private/repo" },
        { "name": "mycompany/privaterepo2", "type": "vcs", "url": "https://github.com/mycompany/privaterepo2" }
    ]
}
```

Se você quiser verificar apenas um único repositório e atualizar todos os
pacotes encontrados nele, passe a URL do repositório VCS como um argumento
opcional:

```shell
php bin/satis build --repository-url https://only.my/repo.git satis.json web/
```

### Uso

Nos seus projetos, tudo o que você precisa fazer agora é adicionar o seu próprio
repositório Composer usando `packages.example.org` como URL; assim, você poderá
exigir seus pacotes privados e tudo deverá funcionar perfeitamente.
Você não precisa mais copiar todos os seus repositórios em cada projeto, apenas
aquele repositório único que se atualizará automaticamente.

```json
{
    "repositories": [ { "type": "composer", "url": "http://packages.example.org/" } ],
    "require": {
        "company/package": "1.2.0",
        "company/package2": "1.5.2",
        "company/package3": "dev-master"
    }
}
```

### Segurança

Para proteger seu repositório privado, você pode hospedá-lo via SSH ou SSL
utilizando um certificado de cliente.
Em seu projeto, você pode usar o parâmetro `options` para especificar as opções
de conexão com o servidor.

Exemplo de uso de um repositório personalizado via SSH (requer a extensão SSH2
do PECL):

```json
{
    "repositories": [{
        "type": "composer",
        "url": "ssh2.sftp://example.org",
        "options": {
            "ssh2": {
                "username": "composer",
                "pubkey_file": "/home/composer/.ssh/id_rsa.pub",
                "privkey_file": "/home/composer/.ssh/id_rsa"
            }
        }
    }]
}
```

> **Dica:** Consulte as
> [opções de contexto ssh2](https://secure.php.net/manual/en/wrappers.ssh2.php#refsect1-wrappers.ssh2-options)
> para mais informações.

Exemplo utilizando SSL/TLS (HTTPS) com um certificado de cliente:

```json
{
    "repositories": [{
         "type": "composer",
         "url": "https://example.org",
         "options": {
             "ssl": {
                 "local_cert": "/home/composer/.ssl/composer.pem"
             }
         }
    }]
}
```

> **Dica:** Consulte as
> [opções de contexto SSL](https://secure.php.net/manual/en/context.ssl.php)
> para mais informações.

Exemplo utilizando um campo de cabeçalho HTTP personalizado para autenticação
por token:

```json
{
    "repositories": [{
        "type": "composer",
        "url": "https://example.org",
        "options":  {
            "http": {
                "header": [
                    "API-TOKEN: YOUR-API-TOKEN"
                ]
            }
        }
    }]
}
```

### Autenticação

A autenticação pode ser realizada de
[várias maneiras diferentes](authentication-for-private-packages.md).

### Downloads

Quando repositórios do GitHub, GitLab ou Bitbucket são espelhados na sua
instância local do Satis, o processo de construção incluirá a localização dos
downloads disponibilizados por essas plataformas.
Isso significa que o repositório e a sua configuração dependem da
disponibilidade desses serviços.

Ao mesmo tempo, isso implica que todo código hospedado em outro lugar (em outro
serviço ou, por exemplo, no Subversion) não terá downloads disponíveis e,
portanto, as instalações geralmente levam muito mais tempo.

Para permitir que sua instalação do Satis crie downloads para todos os seus
pacotes (Git, Mercurial e Subversion), adicione o seguinte ao seu `satis.json`:

```json
{
    "archive": {
        "directory": "dist",
        "format": "tar",
        "prefix-url": "https://amazing.cdn.example.org",
        "skip-dev": true
    }
}
```

#### Explicação das opções

* `directory`: obrigatório, o local dos arquivos de distribuição (dentro do
  `output-dir`).
* `format`: opcional, `zip` (padrão) ou `tar`.
* `prefix-url`: opcional, local dos downloads; por padrão, é a página inicial
  (definida em `satis.json`) seguida pelo `directory`.
* `skip-dev`: opcional, `false` por padrão; quando ativado (`true`), o Satis não
  criará downloads para branches.
* `absolute-directory`: opcional, um diretório _local_ onde os arquivos de
  distribuição são salvos, em vez de `output-dir`/`directory`.
* `whitelist`: opcional; se definida como uma lista de nomes de pacotes, o Satis
  salvará apenas os arquivos de distribuição desses pacotes.
* `blacklist`: opcional; se definida como uma lista de nomes de pacotes, o Satis
  não salvará os arquivos de distribuição desses pacotes.
* `checksum`: opcional, `true` por padrão; quando desativado (`false`), o Satis
  não fornecerá o checksum sha1 para os arquivos de distribuição.

Uma vez habilitado, todos os downloads (incluindo aqueles do GitHub e Bitbucket)
serão substituídos por uma versão _local_.

#### `prefix-url`

Adicionar um prefixo de host à URL é especialmente útil se os downloads
estiverem hospedados em um bucket privado do Amazon S3 ou em uma CDN.
O uso de uma CDN melhoraria drasticamente os tempos de download e,
consequentemente, a instalação do pacote.

Exemplo: um `prefix-url` definido como `https://my-bucket.s3.amazonaws.com` (com
o `directory` definido como `dist`) gera URLs de download no seguinte formato:
`https://my-bucket.s3.amazonaws.com/dist/vendor-package-version-ref.zip`.

### Saídas web

* `output-html`: opcional, `true` por padrão; quando desativado (`false`), o
  Satis não gerará a página `output-dir/index.html`.
* `twig-template`: opcional, um caminho para um template
  [Twig](https://twig.sensiolabs.org/) personalizado para a página
  `output-dir/index.html`.

### Pacotes abandonados

Para permitir que sua instalação do Satis indique que alguns pacotes estão
abandonados, adicione o seguinte ao seu `satis.json`:

```json
{
    "abandoned": {
        "company/package": true,
        "company/package2": "company/newpackage"
    }
}
```

O valor `true` indica que o pacote está realmente abandonado, enquanto o valor
`"company/newpackage"` especifica que o pacote foi substituído pelo pacote
`company/newpackage`.

Observe que todos os pacotes definidos como abandonados em seus próprios
arquivos `composer.json` também serão marcados como abandonados.

### Resolvendo dependências

É possível fazer com que o Satis resolva e adicione automaticamente todas as
dependências dos seus projetos.
Esse recurso pode ser utilizado em conjunto com a funcionalidade de downloads
para criar um espelho local completo dos pacotes.
Adicione o seguinte ao seu `satis.json`:

```json
{
    "require-dependencies": true,
    "require-dev-dependencies": true
}
```

Ao buscar pacotes, o Satis tentará resolver todos os pacotes necessários a
partir dos repositórios listados.
Portanto, se você precisar de um pacote do Packagist, precisará defini-lo no seu
`satis.json`.

As dependências de desenvolvimento são incluídas apenas se o parâmetro
`require-dev-dependencies` estiver definido como `true`.

### Outras opções

* `providers`: opcional, `false` por padrão; quando habilitado (`true`), cada
  pacote será gravado em um arquivo de inclusão separado, que só será carregado
  pelo Composer quando o pacote for realmente necessário.
  Acelera o processamento do Composer para repositórios com muitos pacotes,
  como, por exemplo, o Packagist.
* `output-dir`: opcional, define onde os arquivos do repositório serão gerados,
  caso não seja fornecido como argumento ao executar o comando `build`.
* `config`: opcional, permite definir todas as opções de configuração do
  Composer, exceto `archive-format` e `archive-dir`, uma vez que essa
  configuração é feita por meio de [archive](#downloads).
  Consulte a documentação sobre o [esquema de configuração](../06-config.md)
  para mais detalhes.
* `notify-batch`: opcional, especifica uma URL que será chamada sempre que um
  usuário instalar um pacote.
  Consulte [notify-batch](../05-repositories.md#notify-batch).
