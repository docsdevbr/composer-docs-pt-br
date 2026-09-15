---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/authentication-for-private-packages.md
source_revision: 00b43d4ebd9c031e6f3be18113f948eee70efe8b
translation_status: ready

tagline: Acesse pacotes e repositórios privados
---

# Autenticação para pacotes e repositórios privados

Seu [servidor de pacotes privado](handling-private-packages.md) ou sistema de
controle de versão provavelmente está protegido por uma ou mais opções de
autenticação.
Para permitir que seu projeto acesse esses pacotes e repositórios, você
precisará informar ao Composer como se autenticar no servidor que os hospeda.

## Princípios de autenticação

Sempre que o Composer encontra um repositório protegido, primeiro ele tenta
autenticar-se usando as credenciais já definidas.
Quando nenhuma dessas credenciais se aplica, ele solicita as credenciais e as
salva (ou salva um token, caso consiga obtê-lo).

|                                            Tipo                                            | Gerado por solicitação? |
|:------------------------------------------------------------------------------------------:|:-----------------------:|
|                         [http-basic](#autenticação-com-http-basic)                         |           sim           |
|                [http-basic em linha](#autenticação-com-http-basic-em-linha)                |           não           |
|                        [HTTP Bearer](#autenticação-com-http-bearer)                        |           não           |
|       [Cabeçalhos personalizados](#autenticação-com-cabeçalhos-personalizados)             |           não           |
| [Cabeçalhos personalziados em linha](#autenticação-com-cabeçalhos-personalizados-em-linha) |           não           |
|                       [gitlab-oauth](#autenticação-com-gitlab-oauth)                       |           sim           |
|                       [gitlab-token](#autenticação-com-gitlab-token)                       |           sim           |
|                       [github-oauth](#autenticação-com-github-oauth)                       |           sim           |
|                    [bitbucket-oauth](#autenticação-com-bitbucket-oauth)                    |           sim           |
|        [Certificados TLS de cliente](#autenticação-com-certificados-tls-de-cliente)        |           não           |
|                      [forgejo-token](#autenticação-com-forgejo-token)                      |           sim           |

Às vezes, a autenticação automática não é possível, ou você pode querer
predefinir credenciais de autenticação.

As credenciais podem ser armazenadas em 4 locais diferentes: em um arquivo
`auth.json` do projeto, em um `auth.json` global, no próprio `composer.json` ou
na variável de ambiente `COMPOSER_AUTH`.

### Autenticação no `auth.json` por projeto

Neste método de armazenamento de autenticação, um arquivo `auth.json` estará
presente na mesma pasta que o arquivo `composer.json` do projeto.
Você pode criar e editar esse arquivo usando a linha de comando ou
criá-lo/editá-lo manualmente.

> **Nota: Certifique-se de que o arquivo `auth.json` esteja no `.gitignore`**
> para evitar expor credenciais no seu histórico do git.

### Credenciais de autenticação globais

Se você não quiser fornecer credenciais para cada projeto em que trabalha,
armazenar suas credenciais globalmente pode ser uma ideia melhor.
Essas credenciais são armazenadas em um `auth.json` global no diretório inicial
do Composer.

#### Editando as credenciais globais na linha de comando

É possível editar todos os métodos de autenticação usando a linha de comando:

- [http-basic](#autenticação-com-http-basic-na-linha-de-comando)
- [http-basic em linha](#autenticação-com-http-basic-em-linha-na-linha-de-comando)
- [HTTP Bearer](#autenticação-com-http-bearer-na-linha-de-comando)
- [Cabeçalhos personalizados](#autenticação-com-cabeçalhos-personalizados-na-linha-de-comando)
- [gitlab-oauth](#autenticação-com-gitlab-oauth-na-linha-de-comando)
- [gitlab-token](#autenticação-com-gitlab-token-na-linha-de-comando)
- [github-oauth](#autenticação-com-github-oauth-na-linha-de-comando)
- [bitbucket-oauth](#autenticação-com-bitbucket-oauth-na-linha-de-comando)
- [forgejo-token](#autenticação-com-forgejo-token-na-linha-de-comando)

#### Editando as credenciais de autenticação global manualmente

> **Nota:** Não é recomendado editar manualmente suas opções de autenticação,
> pois isso pode resultar em JSON inválido.
> Em vez disso, use preferencialmente
> [a linha de comando](#editando-as-credenciais-globais-na-linha-de-comando).

Para editá-las manualmente, execute o seguinte:

```shell
php composer.phar config --global --editor [--auth]
```

Para implementações de autenticação específicas, consulte suas seções:

- [http-basic](#autenticação-manual-com-http-basic)
- [http-basic em linha](#autenticação-manual-com-http-basic-em-linha)
- [HTTP Bearer](#autenticação-manual-com-http-bearer)
- [Cabeçalhos personalizados](#autenticação-manual-com-cabeçalhos-personalizados)
- [Cabeçalhos personalizados em linha](#autenticação-manual-com-cabeçalhos-personalizados-em-linha)
- [gitlab-oauth](#autenticação-manual-com-gitlab-oauth)
- [gitlab-token](#autenticação-manual-com-gitlab-token)
- [github-oauth](#autenticação-manual-com-github-oauth)
- [bitbucket-oauth](#autenticação-manual-com-bitbucket-oauth)
- [Client TLS certificates](#autenticação-manual-com-certificados-tls-de-cliente)
- [forgejo-token](#autenticação-manual-com-forgejo-token)

Editar manualmente este arquivo em vez de usar a linha de comando pode resultar
em erros de JSON inválido.
Para corrigir isso, é preciso abrir o arquivo em um editor e corrigir o erro.
Para encontrar a localização do arquivo `auth.json` global, execute:

```shell
php composer.phar config --global home
```

A pasta conterá o arquivo `auth.json` global, se ele existir.

É possível abrir esse arquivo em um editor e corrigir o erro.

### Autenticação no próprio arquivo `composer.json`

> **Nota:** **Isso não é recomendado**, pois essas credenciais serão visíveis
> para qualquer pessoa com acesso ao `composer.json` quando ele for
> compartilhado por meio de um sistema de controle de versão como o git ou
> se uma pessoa atacante obtiver acesso (leitura) aos arquivos do servidor de
> produção.

Também é possível adicionar credenciais a um `composer.json` por projeto na
seção `config` ou diretamente na definição do repositório.

### Autenticação usando a variável de ambiente `COMPOSER_AUTH`

> **Nota:** Usar o método de variável de ambiente na linha de comando também tem
> implicações de segurança.
> Essas credenciais serão provavelmente armazenadas em memória e podem ser
> persistidas em um arquivo como `~/.bash_history` (Linux) ou
> `ConsoleHost_history.txt` (PowerShell no Windows) ao fechar a sessão.

A última opção para fornecer credenciais ao Composer é usar a variável de
ambiente `COMPOSER_AUTH`.
Essa variável pode ser passada como variável de linha de comando ou definida
como uma variável de ambiente real.
Leia mais sobre o uso desta variável de ambiente
[aqui](../03-cli.md#composer-auth).

## Métodos de autenticação

### Autenticação com `http-basic`

#### Autenticação com `http-basic` na linha de comando

```shell
php composer.phar config [--global] http-basic.repo.example.org username password
```

No comando acima, a chave de configuração `http-basic.repo.example.org` possui
duas partes:

- `http-basic` é o método de autenticação.
- `repo.example.org` é o nome de host do repositório e deve ser substituído pelo
  nome de host do seu repositório.

#### Autenticação manual com `http-basic`

```shell
php composer.phar config [--global] --editor --auth
```

```json
{
    "http-basic": {
        "example.org": {
            "username": "username",
            "password": "password"
        }
    }
}
```

### Autenticação com `http-basic` em linha

Para o método de autenticação `http-basic` em linha, as credenciais não são
armazenadas em um arquivo `auth.json` separado no projeto ou globalmente, mas
no `composer.json` ou na configuração global no mesmo local onde a definição do
repositório foi adicionada.

Certifique-se de que o nome de usuário e a senha estejam codificados conforme a
[RFC 3986](https://www.rfc-editor.org/rfc/rfc3986#section-2.1) (2.1. Codificação
percentual).
Se o nome de usuário, por exemplo, for um endereço de e-mail, ele precisará ser
passado como `name%40example.com`.

#### Autenticação com `http-basic` em linha na linha de comando

```shell
php composer.phar config [--global] repositories.unique-name composer https://username:password@repo.example.org
```

#### Autenticação manual com `http-basic` em linha

```shell
php composer.phar config [--global] --editor
```

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://username:password@example.org"
        }
    ]
}
```

### Autenticação com HTTP Bearer

#### Autenticação com HTTP Bearer na linha de comando

```shell
php composer.phar config [--global] bearer.repo.example.org token
```

No comando acima, a chave de configuração `bearer.repo.example.org` possui duas
partes:

- `bearer` é o método de autenticação.
- `repo.example.org` é o nome de host do repositório e deve ser substituído pelo
  nome de host do seu repositório.

#### Autenticação manual com HTTP Bearer

```shell
php composer.phar config [--global] --editor --auth
```

```json
{
    "bearer": {
        "example.org": "TOKEN"
    }
}
```

### Autenticação com cabeçalhos personalizados

Use cabeçalhos HTTP personalizados para autenticação com repositórios privados
que exigem autenticação baseada em cabeçalho.

#### Autenticação com cabeçalhos personalizados na linha de comando

```shell
php composer.phar config [--global] custom-headers.repo.example.org "API-TOKEN: YOUR-API-TOKEN" "X-CUSTOM-HEADER: Value"
```

No comando acima, a chave de configuração `custom-headers.repo.example.org` é
composta por duas partes:

- `custom-headers` é o método de autenticação.
- `repo.example.org` é o nome de host do repositório e deve ser substituído pelo
  nome de host do seu repositório.

Você pode fornecer múltiplos cabeçalhos personalizados como argumentos
separados.
Cada cabeçalho deve estar no formato padrão de cabeçalho HTTP
`"Header-Name: Header-Value"`.

#### Autenticação manual com cabeçalhos personalizados

```shell
php composer.phar config [--global] --editor --auth
```

```json
{
    "custom-headers": {
        "repo.example.org": [
            "API-TOKEN: YOUR-API-TOKEN",
            "X-CUSTOM-HEADER: Value"
        ]
    }
}
```

### Autenticação com cabeçalhos personalizados em linha

#### Autenticação manual com cabeçalhos personalizados em linha

Para o método de autenticação via cabeçalhos personalizados em linha, os
cabeçalhos personalizados são definidos diretamente no seu arquivo
`composer.json`, como parte da configuração do repositório.

```shell
php composer.phar config [--global] --editor
```

```json
{
    "repositories": [
        {
            "type": "composer",
            "url": "https://repo.example.org",
            "options": {
                "http": {
                    "header": [
                        "API-TOKEN: YOUR-API-TOKEN",
                        "X-CUSTOM-HEADER: Value"
                    ]
                }
            }
        }
    ]
}
```

### Autenticação com `gitlab-oauth`

> **Nota:** Para que a autenticação do GitLab funcione em instâncias privadas do
> GitLab, a seção [`gitlab-domains`](../06-config.md#gitlab-domains) também deve
> conter a URL.

#### Autenticação com `gitlab-oauth` na linha de comando

```shell
php composer.phar config [--global] gitlab-oauth.gitlab.example.org token
```

No comando acima, a chave de configuração `gitlab-oauth.gitlab.example.org`
possui duas partes:

- `gitlab-oauth` é o método de autenticação.
- `gitlab.example.org` é o nome de host da instância do GitLab e deve ser
  substituído pelo nome de host da sua instância GitLab ou usar `gitlab.com` se
  não tiver uma instância própria do GitLab.

#### Autenticação manual com `gitlab-oauth`

```shell
php composer.phar config [--global] --editor --auth
```

```json
{
    "gitlab-oauth": {
        "example.org": "token"
    }
}
```

### Autenticação com `gitlab-token`

> **Nota:** Para que a autenticação do GitLab funcione em instâncias privadas do
> GitLab, a seção [`gitlab-domains`](../06-config.md#gitlab-domains) também deve
> conter a URL.

Para criar um token de acesso, acesse a
[seção de tokens de acesso no GitLab](https://gitlab.com/-/user_settings/personal_access_tokens)
(ou a URL equivalente em sua instância privada) e crie um token.
Consulte também a
[documentação de tokens de acesso do GitLab](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html#creating-a-personal-access-token)
para obter mais informações.

Ao criar um token do GitLab manualmente, certifique-se de que ele tenha o escopo
`read_api` ou `api`.

#### Autenticação com `gitlab-token` na linha de comando

```shell
php composer.phar config [--global] gitlab-token.gitlab.example.org token
```

No comando acima, a chave de configuração `gitlab-token.gitlab.example.org`
possui duas partes:

- `gitlab-token` é o método de autenticação.
- `gitlab.example.org` é o nome de host da instância do GitLab e deve ser
  substituído pelo nome de host da sua instância GitLab ou usar `gitlab.com` se
  não tiver uma instância própria do GitLab.

#### Autenticação manual com `gitlab-token`

```shell
php composer.phar config [--global] --editor --auth
```

```json
{
    "gitlab-token": {
        "example.org": "token"
    }
}
```

### Autenticação com `github-oauth`

Atualmente, o GitHub oferece dois tipos de tokens de acesso:

- [Tokens de escopo refinado (fine-grained tokens)](https://github.com/settings/personal-access-tokens)
- [Tokens (clássicos)](https://github.com/settings/personal-access-tokens)

Eles podem ser encontrados em
[Configurações](https://github.com/settings/profile), na parte inferior do menu
lateral esquerdo ([Opções de desenvolvedor](https://github.com/settings/apps)).
Para criar um token de acesso, vá para a
[seção de configurações de token no GitHub](https://github.com/settings/personal-access-tokens)
e
[gere um token](https://github.com/settings/personal-access-tokens/new).

Leia mais sobre
[Tokens de Acesso Pessoal](https://docs.github.com/pt/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).

É
[recomendado](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#types-of-personal-access-tokens)
usar tokens de escopo refinado, pois você pode ter um controle muito mais
rigoroso sobre o que pode ser acessado.
O Composer requer acesso somente leitura aos metadados e ao conteúdo dos
repositórios.

Geralmente, um token de escopo refinado com acesso somente leitura a
repositórios públicos pode ser suficiente.
Mesmo sem permissões adicionais, esses tokens
[aumentam seus limites de taxa da API](https://docs.github.com/en/rest/using-the-rest-api/rate-limits-for-the-rest-api).

Permissões adicionais serão necessárias nos seguintes casos:

- Você está usando entradas do tipo `vcs` em `repositories` no seu arquivo
  `composer.json` que apontam para repositórios privados.
- Você está clonando o `source` ou baixando arquivos `dist` de repositórios
  privados via HTTPS (e não via SSH, por exemplo).

Nesses casos, crie um token de escopo refinado com acesso somente leitura a
"contents" (conteúdo).
O token pode ser vinculado a todos os seus repositórios ou aos da organização,
ou até mesmo ter seu escopo restrito a apenas repositórios selecionados.

Até novembro de 2025, os tokens de escopo granular (fine-grained tokens)
apresentam a limitação de permitir o acesso aos repositórios privados de apenas
um usuário _ou_ de uma única organização por vez.
Recomendamos consultar a
[documentação do GitHub](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#fine-grained-personal-access-tokens-limitations)
para obter detalhes sobre isso.
Se você precisar trabalhar com repositórios de organizações diferentes,
verifique se o uso de uma configuração de autenticação específica para o
diretório atende às suas necessidades.

Como última opção, um token "clássico" com o escopo `repo` concederá amplo
acesso a todos os seus repositórios privados, incluindo permissões de escrita e
muito mais.
A
[documentação sobre escopos](https://docs.github.com/en/developers/apps/building-oauth-apps/scopes-for-oauth-apps)
contém a lista completa.
Tenha cuidado ao usar esse tipo de token.

Independentemente do tipo de token usado, recomendamos o uso de tokens com tempo
de vida limitado.
Isso reduz a exposição caso o token seja comprometido.

#### Autenticação com `github-oauth` na linha de comando

```shell
php composer.phar config [--global] github-oauth.github.com token
```

No comando acima, a chave de configuração `github-oauth.github.com`
possui duas partes:

- `github-oauth` é o método de autenticação.
- `github.com` é o nome do host ao qual este token se aplica.
  Para o GitHub, provavelmente você não precisará alterar isso.

#### Autenticação manual com `github-oauth`

```shell
php composer.phar config [--global] --editor --auth
```

```json
{
    "github-oauth": {
        "github.com": "token"
    }
}
```

### Autenticação com `bitbucket-oauth`

O driver BitBucket usa OAuth para acessar os repositórios privados por meio das
APIs REST do BitBucket, e será necessário criar um consumidor OAuth para usar o
driver; consulte a
[documentação da Atlassian](https://support.atlassian.com/bitbucket-cloud/docs/use-oauth-on-bitbucket-cloud/).
Será necessário preencher a URL da chamada de retorno com algo que satisfaça o
BitBucket, mas o endereço não precisa ir a lugar algum e não é usado pelo
Composer.

O consumidor criado precisa ter, no mínimo, as permissões "Repository > Read".

#### Autenticação com `bitbucket-oauth` na linha de comando

```shell
php composer.phar config [--global] bitbucket-oauth.bitbucket.org consumer-key consumer-secret
```

No comando acima, a chave de configuração `bitbucket-oauth.bitbucket.org` possui
duas partes:

- `bitbucket-oauth` é o método de autenticação.
- `bitbucket.org` é o nome do host ao qual este token se aplica.
  Não é necessário alterar isso, a menos que tenha uma instância privada.

#### Autenticação manual com `bitbucket-oauth`

```shell
php composer.phar config [--global] --editor --auth
```

```json
{
    "bitbucket-oauth": {
        "bitbucket.org": {
            "consumer-key": "key",
            "consumer-secret": "secret"
        }
    }
}
```

### Autenticação com tokens de API do Bitbucket

Como alternativa aos consumidores OAuth, você pode usar tokens de API da
Atlassian.
Crie um token com o escopo `read:repository:bitbucket` na
[sua conta Atlassian](https://id.atlassian.com/manage-profile/security/api-tokens).

Após obter o token, configure-o utilizando autenticação HTTP Basic, com o e-mail
da sua conta Atlassian como nome de usuário e o token como senha:

```shell
php composer.phar config [--global] http-basic.bitbucket.org your@email.com api-token
```

Ou manualmente no arquivo `auth.json`:

```shell
php composer.phar config [--global] --editor --auth
```

```json
{
    "http-basic": {
        "bitbucket.org": {
            "username": "your@email.com",
            "password": "api-token"
        }
    }
}
```

### Autenticação com certificados TLS de cliente

Acesso a repositórios privados que exigem certificados TLS de cliente.

Para configuração global ou em nível de projeto, consulte
[Gerenciamento de pacotes privados: seção de segurança](handling-private-packages.md#security).

#### Autenticação manual com certificados TLS de cliente

```shell
php composer.phar config [--global] --editor --auth
```

```json
{
    "client-certificate": {
        "repo.example.org": {
            "local_cert": "/path/to/certificate",
            "local_pk": "/path/to/key",
            "passphrase": "MySecretPassword"
        }
    }
}
```

As opções suportadas são `local_cert` (obrigatória), `local_pk` e `passphrase`.
Mais informações sobre as opções podem ser encontradas em
[Opções de contexto SSL](https://www.php.net/manual/en/context.ssl.php).

Estas opções podem ser omitidas:

- `local_pk`: caso o certificado e a chave privada estejam em um único arquivo.
- `passphrase`: caso a chave privada não possua senha.

### Autenticação com `forgejo-token`

> **Nota:** Para que a autenticação no Forgejo funcione em instâncias privadas
> do Forgejo, a seção [`forgejo-domains`](../06-config.md#forgejo-domains)
> também deve conter o domínio.

Para criar um token de acesso, acesse a seção de aplicações no Forgejo (ou a URL
equivalente na sua instância privada) e crie um token de acesso.
Consulte também
[a documentação de tokens de acesso do Forgejo](https://docs.codeberg.org/advanced/access-token/)
para mais informações.

Ao criar um token de acesso do Forgejo, certifique-se de que ele tenha o escopo
`read:repository`.

#### Autenticação com `forgejo-token` na linha de comando

```shell
php composer.phar config [--global] forgejo-token.forgejo.example.org username access-token
```

No comando acima, a chave de configuração `forgejo-token.forgejo.example.org`
consiste em duas partes:

- `forgejo-token` é o método de autenticação.
- `forgejo.example.org` é o nome de host da instância do Forgejo e deve ser
  substituído pelo nome de host da sua instância do Forgejo ou usar
  `codeberg.org` caso não possua uma instância própria do Forgejo.

#### Autenticação manual com `forgejo-token`

```shell
php composer.phar config [--global] --editor --auth
```

```json
{
    "forgejo-token": {
        "forgejo.example.org": {
            "username": "forgejo-user",
            "token": "access-token"
        }
    }
}
```
