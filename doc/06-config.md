---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/06-config.md
source_revision: 2ce09be45d1f93264540913317dd48eb2613d26d
translation_status: ready
---

# Config

Este capítulo descreve a seção `config` do [esquema](04-schema.md) do
`composer.json`.

## process-timeout

O tempo limite, em segundos, para a execução de processos; o padrão é 300 (5
minutos).
Define por quanto tempo processos como `git clone` podem ser executados antes
que o Composer considere que eles morreram.
Pode ser necessário aumentar esse valor se você tiver uma conexão lenta ou
fornecedores muito grandes.

Exemplo:

```json
{
    "config": {
        "process-timeout": 900
    }
}
```

### Desativando o tempo limite para um comando de script individual

Para desabilitar o tempo limite do processo em um comando personalizado na seção
`scripts`, está disponível um método auxiliar estático:

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

## allow-plugins

O padrão é `{}`, o que não permite o carregamento de nenhum plugin.

A partir do Composer 2.2.0, a opção `allow-plugins` adiciona uma camada de
segurança, permitindo restringir quais plugins do Composer podem executar código
durante a execução do Composer.

Quando um novo plugin é ativado pela primeira vez, um plugin que ainda não
consta na lista da opção de configuração, o Composer exibirá um alerta.
Se você executar o Composer de forma interativa, ele solicitará que você decida
se deseja ou não executar o plugin.

Use esta configuração para permitir a execução de código apenas por pacotes em
que você confia.
Defina-a como um objeto cujas chaves sejam padrões de nomes de pacotes.
Os valores devem ser **true** para permitir ou **false** para não permitir,
suprimindo alertas e solicitações adicionais.

```json
{
    "config": {
        "allow-plugins": {
            "terceiros/plugin-necessário": true,
            "minha-organização/*": true,
            "plugin/desnecessário": false
        }
    }
}
```

Você também pode definir a própria opção de configuração como `false` para
desabilitar todos os plugins, ou como `true` para permitir a execução de todos
os plugins (o que NÃO é recomendado).
Por exemplo:

```json
{
    "config": {
        "allow-plugins": false
    }
}
```

## use-include-path

O padrão é `false`.
Se definida como `true`, o autoloader do Composer também buscará classes no
`include_path` do PHP.

## preferred-install

O padrão é `dist` e pode ser definido como `source`, `dist` ou `auto`.
Esta opção permite definir o método de instalação que o Composer preferirá usar.
Pode, opcionalmente, ser um objeto cujas chaves sejam padrões de nomes de
pacotes, permitindo preferências de instalação mais granulares.

```json
{
    "config": {
        "preferred-install": {
            "minha-organização/pacote-estável": "dist",
            "minha-organização/*": "source",
            "organização-parceira/*": "auto",
            "*": "dist"
        }
    }
}
```

- `source` significa que o Composer instalará pacotes a partir de seu `source`,
  caso ele exista.
  Geralmente, isso corresponde a um `git clone` ou operação equivalente de
  checkout no sistema de controle de versão usado pelo pacote.
  É útil se você deseja corrigir uma falha em um projeto e obter um clone local
  do Git da dependência diretamente.
- `auto` é o comportamento legado, no qual o Composer usa `source`
  automaticamente para versões de desenvolvimento e `dist` nos demais casos.
- `dist` (o padrão a partir do Composer 2.1) significa que o Composer instala a
  partir de `dist`, sempre que possível.
  Geralmente, isso envolve o download de um arquivo zip, o que é mais rápido do
  que clonar o repositório inteiro.

> **Nota:** A ordem importa.
> Padrões mais específicos devem aparecer antes de padrões mais genéricos.
> Ao combinar a notação de string com a configuração de hash nas configurações
> globais e de pacotes, a notação de string é convertida para um padrão de
> pacote `*`.

> **Dica:** Se você deseja obter o código-fonte de alguns pacotes para realizar
> edições locais, mas apenas em sua própria máquina (e não no CI), prefira
> configurar o `preferred-install` globalmente em vez de incluí-lo no controle
> de versão do projeto.
> Por exemplo:
>
> ```
> composer config --global preferred-install.meu-fornecedor/* source
> ```
>
> Assim, o CI continua instalando a partir de `dist` por padrão, enquanto sua
> máquina de desenvolvimento sempre obtém esses pacotes a partir da fonte.
> Isso evita depender de uma falha na instalação via fonte para recorrer ao
> `dist`, além de poupar o CI de tentar um clone inicialmente, mantendo o log de
> saída mais enxuto e o processo de CI mais rápido.

## source-fallback

> **Obsoleto:** esta opção está obsoleta e será removida no Composer 2.11.
> Não a defina, a menos que seja absolutamente necessário.
> Ela existe como uma opção temporária enquanto desativamos o fallback de `dist`
> para `source` por motivos de segurança (a alternância silenciosa de um pacote
> `dist` para um checkout de `source` menos confiável ou manipulado traz
> implicações de segurança).
> Se você tiver um caso de uso legítimo para reabilitar o fallback de `dist`
> para `source` e não quiser que o removamos na versão 2.11, por favor, abra uma
> issue em https://github.com/composer/composer/issues para nos informar.

O padrão é `false`.
Quando definido como `true`, uma instalação via **dist** que falhar recorrerá a
um checkout via **source**.
Esta opção controla apenas a direção `dist` → `source`; o fallback de um
checkout via `source` com falha para o artefato `dist` é sempre permitido,
independentemente desta configuração.

```json
{
    "config": {
        "source-fallback": true
    }
}
```

> **Nota:** Com esta opção em seu valor padrão (`false`) e uma falha no download
> do `dist`, o Composer gera um erro imediatamente em vez de tentar um checkout
> via `source`.
> Certifique-se de que sua fonte de instalação preferida (`preferred-install`)
> esteja configurada corretamente.

## policy

Configuração unificada de política de dependências.
Controla o comportamento do Composer para dependências com avisos de segurança,
marcadas como malware, pacotes abandonados e políticas de dependência
personalizadas.
Relatórios de auditoria podem ser gerados com `composer audit`; o bloqueio
impede que versões de pacotes inseguras ou marcadas de outra forma sejam
instaladas durante `composer update`, `require` ou `remove` e, no caso de
malware, também durante um `composer install`.

Defina como `false` para desabilitar a aplicação de todas as políticas de
dependência:

```json
{
    "config": {
        "policy": false
    }
}
```

> **Migrando de `config.audit`?**
> Consulte
> [Como `config.audit` interage com `config.policy`](#como-config-audit-interage-com-config-policy)
> para entender como as chaves legadas ainda são respeitadas como fallback
> enquanto você realiza a migração.

### advisories

Configuração para pacotes afetados por avisos de segurança.

#### block

O padrão é `true`.
Quando definido como `true`, versões de pacotes com avisos de segurança ativos
são bloqueadas e não podem ser instaladas durante `update`, `require` ou
`remove`, a menos que o aviso ou o pacote seja ignorado.

```json
{
    "config": {
        "policy": {
            "advisories": {
                "block": false
            }
        }
    }
}
```

#### audit

O padrão é `fail`.
Define como o `composer audit` trata pacotes com avisos de segurança.

- `ignore` — avisos não são relatados.
- `report` — avisos são relatados, mas não geram um código de saída diferente
  de zero.
- `fail` — avisos fazem com que o `composer audit` termine com um código
  diferente de zero.

```json
{
    "config": {
        "policy": {
            "advisories": {
                "audit": "report"
            }
        }
    }
}
```

#### ignore-id

Uma lista de IDs de avisos (CVE, GHSA, PKSA, …) a serem ignorados.
Cada entrada pode incluir, opcionalmente, um motivo e um escopo
(`on-block`/`on-audit`) para limitar onde a regra de ignorar se aplica.

##### Lista simples:

```json
{
    "config": {
        "policy": {
            "advisories": {
                "ignore-id": ["CVE-1234", "GHSA-xx"]
            }
        }
    }
}
```

##### Com motivos:

```json
{
    "config": {
        "policy": {
            "advisories": {
                "ignore-id": {
                    "CVE-1234": "Não afetado.",
                    "GHSA-xx": "Patch aplicado."
                }
            }
        }
    }
}
```

##### Com escopo:

`on-block: false` significa que o aviso não bloqueia mais atualizações, mas
ainda é relatado na auditoria.
`on-audit: false` significa que o aviso ainda bloqueia atualizações, mas não é
mais relatado na auditoria.

```json
{
    "config": {
        "policy": {
            "advisories": {
                "ignore-id": {
                    "CVE-1234": {"on-block": false, "reason": "Patch aplicado, mas ainda queremos o bloqueio."},
                    "GHSA-xx":  {"on-audit": false, "reason": "Falso positivo, mas ainda queremos o relatório."}
                }
            }
        }
    }
}
```

#### ignore

Uma lista de nomes de pacotes a serem ignorados no processamento de avisos de
segurança.
Suporta caracteres curinga e restrições de versão opcionais.
Consulte o [formato de ignore](#formato-de-ignore) para ver todas as variantes
de sintaxe suportadas.

#### ignore-severity

Uma lista de níveis de gravidade de avisos a serem ignorados: `low` (baixo),
`medium` (médio), `high` (alto), `critical` (crítico).

##### Lista simples:

```json
{
    "config": {
        "policy": {
            "advisories": {
                "ignore-severity": ["low", "medium"]
            }
        }
    }
}
```

##### Com escopo definido:

```json
{
    "config": {
        "policy": {
            "advisories": {
                "ignore-severity": {
                    "low":    {"on-block": false},
                    "medium": {"on-audit": false, "reason": "Tratado via WAF"}
                }
            }
        }
    }
}
```

### abandoned

Configuração para pacotes abandonados.

#### block

O padrão é `false`.
Quando definido como `true`, pacotes abandonados não podem ser instalados
durante as operações `update`, `require` ou `remove`.

```json
{
    "config": {
        "policy": {
            "abandoned": {
                "block": true
            }
        }
    }
}
```

#### audit

O padrão é `fail`.
Define como o `composer audit` trata pacotes abandonados.

- `ignore` — pacotes abandonados não são relatados.
- `report` — pacotes abandonados são relatados, mas não geram um código de saída
  diferente de zero.
- `fail` — pacotes abandonados fazem com que o `composer audit` termine com um
  código diferente de zero.

```json
{
    "config": {
        "policy": {
            "abandoned": {
                "audit": "report"
            }
        }
    }
}
```

Essa configuração pode ser sobrescrita pela variável de ambiente
[`COMPOSER_AUDIT_ABANDONED`](03-cli.md#composer-audit-abandoned) ou pela opção
de CLI [`--abandoned`](03-cli.md#audit).

#### ignore

Uma lista de nomes de pacotes (ou padrões) a serem ignorados na verificação de
pacotes abandonados, independentemente de seu status de abandono.
Consulte o [formato de ignore](#formato-de-ignore) para ver todas as variantes
de sintaxe suportadas.

```json
{
    "config": {
        "policy": {
            "abandoned": {
                "ignore": {
                    "acme/*": "Agendado para substituição no próximo trimestre.",
                    "vendor/legacy": {"on-block": false, "reason": "Permitir em atualizações, mas ainda assim reportar."}
                }
            }
        }
    }
}
```

### malware

Configuração para versões de pacotes sinalizadas como contendo malware.

#### block

O padrão é `true`.
Quando definido como `true`, versões de pacotes sinalizadas como malware são
bloqueadas.

#### block-scope

O padrão é `all`.
Controla quais comandos acionam o bloqueio:

- `all` — bloqueia durante `update`, `require`, `remove` e `install`.
- `update` — bloqueia apenas durante `update`, `require` e `remove`.
- `install` — bloqueia apenas durante `install`.

```json
{
    "config": {
        "policy": {
            "malware": {
                "block-scope": "update"
            }
        }
    }
}
```

#### audit

O padrão é `fail`.
Aceita os mesmos valores que [advisories.audit](#audit).

#### ignore

Nomes de pacotes a serem excluídos das verificações de malware.
Consulte o [formato de ignore](#formato-de-ignore) para ver todas as variantes
de sintaxe suportadas.

#### ignore-source

Uma lista de nomes de fontes a serem excluídos das verificações de malware.

```json
{
    "config": {
        "policy": {
            "malware": {
                "ignore-source": ["aikido"]
            }
        }
    }
}
```

### ignore-unreachable

O padrão é `["update", "install"]`.
Quando a operação está listada aqui, repositórios e políticas com fontes de URL
inacessíveis ou que retornam uma resposta diferente de 200 são ignorados
silenciosamente, em vez de causarem um erro.
Útil em ambientes onde nem todos os repositórios de pacotes são acessíveis.

Defina como `true` para ignorar repositórios e fontes de políticas de
dependência personalizadas inacessíveis em todas as operações, e como `false`
para não ignorá-los em nenhuma operação.
Os valores possíveis são: `audit`, `install` e `update`.

```json
{
    "config": {
        "policy": {
            "ignore-unreachable": ["install", "update", "audit"]
        }
    }
}
```

### Políticas de dependência personalizadas

Além das políticas de dependência integradas `advisories`, `malware` e
`abandoned`, você pode definir políticas de dependência personalizadas com nomes
próprios.
Uma política de dependência personalizada requer seu próprio conjunto de versões
de pacotes, fornecido por uma ou mais fontes (informadas por repositórios de
pacotes ou definidas explicitamente aqui).

```json
{
    "config": {
        "policy": {
            "minha-política": {
                "block": true,
                "audit": "fail",
                "sources": [
                    {"type": "url", "url": "https://example.org/minha-lista-de-pacotes-ruins.json"}
                ],
                "ignore": {
                    "fornecedor/pacote": "Avaliado e aceito."
                }
            }
        }
    }
}
```

As URLs de origem devem usar `https://`. `http://` e outros esquemas são
rejeitados tanto no momento da validação do esquema (`composer validate`) quanto
no momento do carregamento da configuração.

Uma fonte do tipo `url` é consultada da mesma forma que a
[`api-url`](05-repositories.md#filter) de um repositório: o Composer envia uma
requisição POST contendo as PURLs dos pacotes relevantes e o nome da política de
dependência personalizada, aguardando o retorno das entradas de filtro
correspondentes.
A requisição não é armazenada em cache no lado do cliente, pois o corpo de cada
requisição é diferente.
As pessoas implementadoras devem estar cientes de que excesso de nomes de
pacotes (algumas centenas seriam um número normal) podem ser enviadas.
As PURLs enviadas representam o conjunto completo de nomes de pacotes candidatos
coletados *antes* da resolução de dependências; portanto, eles identificam
pacotes apenas pelo nome (sem restrições de versão nesta etapa), e nem todo
pacote enviado será necessariamente selecionado pelo resolvedor posteriormente.

O endpoint recebe um corpo JSON no seguinte formato:

```json
{
    "packages": ["pkg://composer/fornecedor/pacote", "pkg://composer/outro/pacote"],
    "lists": ["minha-política"]
}
```

O corpo da requisição reutiliza o formato de transmissão da
[`api-url`](05-repositories.md#filter) de um repositório do Composer.
Nesse caso, um único endpoint pode atender a várias listas de filtros nomeadas
(por exemplo, `malware` e `typosquatting`); assim, `lists` é um array que
especifica quais delas o Composer deseja, e a resposta é um objeto `filter`
indexado pelo nome da lista.
Uma política de dependência personalizada não possui esse tipo de multiplexação:
sua fonte `url` existe apenas para atender a essa política específica.
Portanto, o Composer sempre envia o nome da política como o único elemento de
`lists` (de modo que o array contém exatamente um valor neste caso), e o
endpoint deve tratar qualquer nome de lista recebido como uma referência a essa
política.
O endpoint deve retornar um JSON no seguinte formato:

```json
{
    "filter": [
        {
            "package": "fornecedor/pacote",
            "constraint": ">=1.0.0,<1.2.0",
            "url": "https://example.org/filters/123",
            "reason": "Avaliado e rejeitado.",
            "id": "PKFE-xxxx-xxxx-xxxx"
        }
    ]
}
```

Como a fonte da `url` fornece apenas esta política específica, a resposta
dispensa a estruturação por lista usada pela `api-url` de um repositório (na
qual `filter` é um objeto que mapeia o nome de cada lista solicitada às suas
entradas).
Aqui, em vez disso, `filter` é um array simples de entradas que pertencem a essa
política.
Os campos `package` e `constraint` são obrigatórios em cada entrada; `url`,
`reason` e `id` são opcionais.
Entradas cujo pacote não corresponda a um pacote da requisição são ignoradas.

Os nomes das políticas de dependência personalizadas não devem entrar em
conflito com os nomes reservados `advisories`, `malware` ou `abandoned`, e não
devem começar com `ignore` (a única chave com o prefixo `ignore` permitida neste
nível é a configuração documentada `ignore-unreachable`).

Os seguintes nomes estão reservados para futuras políticas de dependência
integradas e não podem ser usados como nomes de políticas de dependência
personalizadas: `package`, `packages`, `license`, `licence`, `licenses`,
`licences`, `support`, `maintenance`, `security`, `minimum-release-age`.
O Composer rejeita qualquer chave conflitante, tanto no momento da validação do
esquema (`composer validate`) quanto no momento do carregamento da configuração.

### Formato de `ignore`

A chave `ignore` em qualquer política de dependência aceita padrões de nomes de
pacotes com restrições de versão opcionais e escopo definido por regra.
Todos os formatos podem ser misturados no mesmo mapa.

##### Lista simples (ignora todas as versões):


```json
{
    "config": {
        "policy": {
            "<política>": {
                "ignore": ["fornecedor/pacote", "acme/*"]
            }
        }
    }
}
```

##### Com motivo:

```json
{
    "config": {
        "policy": {
            "<política>": {
                "ignore": {
                    "fornecedor/pacote": "Avaliado, sem risco."
                }
            }
        }
    }
}
```

##### Com restrição de versão:

```json
{
    "config": {
        "policy": {
            "<política>": {
                "ignore": {
                    "fornecedor/pacote": {"constraint": "^2.0", "reason": "Apenas a v2 é afetada."}
                }
            }
        }
    }
}
```

##### Com escopo:

`on-block: false` ignora apenas para auditoria (o pacote ainda é bloqueado
durante atualizações, pois `on-block` está desabilitada).
`on-audit: false` ignora apenas para bloqueio (o pacote ainda é relatado na
auditoria).

```json
{
    "config": {
        "policy": {
            "<política>": {
                "ignore": {
                    "fornecedor/pacote": {"on-audit": false, "reason": "Gambiarra aplicada; manter relatório."}
                }
            }
        }
    }
}
```

##### Múltiplas regras para o mesmo pacote:

```json
{
    "config": {
        "policy": {
            "<política>": {
                "ignore": {
                    "fornecedor/pacote": [
                        {"constraint": "^1.0", "on-audit": false},
                        {"constraint": "^2.0", "reason": "v2 corrigida."}
                    ]
                }
            }
        }
    }
}
```

## audit

> **Obsoleto.**
> Use [`config.policy`](#policy) em vez disso.
> Todas as chaves de `config.audit` ainda são suportadas para fins de
> compatibilidade com versões anteriores, mas serão removidas em uma futura
> versão principal.

Opções de configuração para auditoria de segurança e bloqueio de versões.
Relatórios de auditoria podem ser gerados com `composer audit`, e versões
resumidas são exibidas automaticamente ao final dos comandos `update` ou
`require`.
O bloqueio de versões descarta versões de pacotes identificadas como inseguras
ou abandonadas, dependendo da configuração, antes da resolução das dependências,
garantindo que não possam ser instaladas.

### Como `config.audit` interage com `config.policy`

As chaves legadas de `config.audit` são lidas apenas como fallback quando a
seção correspondente de [`config.policy`](#policy) está **ausente**.
Esse mecanismo de fallback funciona no modelo "tudo ou nada" para cada política
de dependência integrada:

- Se [`config.policy.advisories`](#advisories) estiver definida (com qualquer
  valor, inclusive `false`), todas as chaves `audit.*` relacionadas a avisos de
  segurança ([`audit.block-insecure`](#block-insecure),
  [`audit.ignore`](#ignore-3), [`audit.ignore-severity`](#ignore-severity-1))
  são **totalmente ignoradas** — apenas [`policy.advisories.block`](#block),
  [`policy.advisories.ignore`](#ignore) e
  [`policy.advisories.ignore-severity`](#ignore-severity) são lidas.
  Não há suporte para misturar configurações (por exemplo, definir
  `policy.advisories.block` esperando que `audit.ignore-severity` ainda seja
  aplicada).
  Migre todas as configurações relacionadas a avisos de segurança em conjunto.
- Se [`config.policy.abandoned`](#abandoned) estiver definida (com qualquer
  valor, incluindo `false`), todas as chaves `audit.*` relacionadas a pacotes
  abandonados ([`audit.block-abandoned`](#block-abandoned),
  [`audit.abandoned`](#abandoned-1),
  [`audit.ignore-abandoned`](#ignore-ignore-abandoned)) são **totalmente
  ignoradas** — apenas [`policy.abandoned.block`](#block-1),
  [`policy.abandoned.audit`](#audit-1) e [`policy.abandoned.ignore`](#ignore-1)
  são lidas.
- As duas políticas de dependência nativas são independentes: é permitido
  configurar `policy.advisories` mantendo as configurações de pacotes
  abandonados em `audit.*`, e vice-versa.
- Definir [`policy.ignore-unreachable`](#ignore-unreachable) substitui a chave
  legada [`audit.ignore-unreachable`](#ignore-unreachable-1).

### ignore

> **Obsoleto.**
> Use [`config.policy.advisories.ignore-id`](#ignore-id) para IDs de avisos de
> segurança (CVE, GHSA, PKSA) e [`config.policy.advisories.ignore`](#ignore)
> para nomes de pacotes.
> Nota: o novo formato usa valores booleanos `on-block`/`on-audit` em vez de
> `"apply": "audit|block|all"`.

Uma lista de IDs de avisos de segurança, IDs remotos, IDs CVE ou nomes de
pacotes (não recomendado) ignorados em relatórios de auditoria e/ou no bloqueio
de versões.

#### Formato simples com motivos:

```json
{
    "config": {
        "audit": {
            "ignore": {
                "CVE-1234": "O componente afetado não está em uso.",
                "GHSA-xx": "A correção de segurança foi aplicada como um patch.",
                "PKSA-yy": "Devido a medidas de mitigação existentes, a atualização pode ser adiada."
            }
        }
    }
}
```

#### Formato simples sem motivos:

```json
{
    "config": {
        "audit": {
            "ignore": ["CVE-1234", "GHSA-xx", "PKSA-yy"]
        }
    }
}
```

#### Formato detalhado com escopo de aplicação:

O formato detalhado permite controlar se uma regra de ignorar se aplica apenas a
relatórios de auditoria, apenas ao bloqueio de versão ou a ambos.
O campo `apply` aceita:

- `audit` - ignorar apenas para relatórios de auditoria (o alerta não aparece
  nos relatórios de auditoria, mas o pacote ainda é bloqueado durante
  atualizações).
- `block` - ignorar apenas para bloqueio de versão (o pacote pode ser usado
  durante atualizações, mas o alerta ainda aparece nos relatórios de auditoria).
- `all` - ignorar tanto em relatórios de auditoria quanto no bloqueio de versão
  (comportamento padrão).

```json
{
    "config": {
        "audit": {
            "ignore": {
                "CVE-1234": {
                    "apply": "audit",
                    "reason": "Não se aplica a nós, portanto não reportar, mas ainda queremos garantir que não usemos esta versão em atualizações."
                },
                "GHSA-xx": {
                    "apply": "block",
                    "reason": "Gambiarra aplicada; a correção só será feita na próxima semana. Permitir durante atualizações, mas ainda reportar em auditorias."
                },
                "PKSA-yy": {
                    "apply": "all",
                    "reason": "Relatório falso; ignorar completamente em todos os contextos."
                }
            }
        }
    }
}
```

Todos os formatos podem ser combinados na mesma configuração.

### abandoned

> **Obsoleto.**
> Use [`config.policy.abandoned.audit`](#audit-1) em vez disso.

O padrão é `fail` desde o Composer 2.7 (o padrão era `report` no Composer 2.6,
quando a opção foi adicionada).
Define se e como os relatórios de auditoria devem reportar pacotes abandonados.
Existem três valores possíveis:

- `ignore` significa que os relatórios de auditoria não consideram pacotes
  abandonados de forma alguma.
- `report` significa que pacotes abandonados são reportados como um erro, mas
  não fazem com que o comando `composer audit` retorne um código de saída
  diferente de zero.
- `fail` significa que pacotes abandonados farão com que o comando de auditoria
  falhe com um código de saída diferente de zero.

Observe que isso se aplica apenas aos relatórios de auditoria; essa configuração
não afeta o bloqueio de versões de pacotes inseguras.
Para configurar o bloqueio de pacotes abandonados, consulte a opção
[`block-abandoned`](#block-abandoned).

```json
{
    "config": {
        "audit": {
            "abandoned": "report"
        }
    }
}
```

Desde o Composer 2.7, a opção pode ser sobrescrita pela variável de ambiente
[`COMPOSER_AUDIT_ABANDONED`](03-cli.md#composer-audit-abandoned).

Desde o Composer 2.8, a opção pode ser sobrescrita pela opção de linha de
comando [`--abandoned`](03-cli.md#audit), que tem precedência tanto sobre o
valor de configuração quanto sobre a variável de ambiente.

### ignore-abandoned

> **Obsoleto.**
> Use [`config.policy.abandoned.ignore`](#ignore-1) em vez disso.
> Nota: o novo formato usa booleanos `on-block`/`on-audit` em vez de
> `"apply": "audit|block|all"`.

Uma lista de nomes de pacotes abandonados ignorados nos relatórios de auditoria
e/ou no bloqueio de versões.
Permite selecionar pacotes que você deseja continuar usando, apesar de seu
estado de abandono.

#### Formato simples com motivos:

```json
{
    "config": {
        "audit": {
            "ignore-abandoned": {
                "acme/*": "Trabalho agendado para remoção no próximo mês.",
                "acme/pacote": "Dependência transitiva, porém inacessível e sem uso ativo no contexto do nosso projeto."
            }
        }
    }
}
```

#### Formato simples sem motivos:

```json
{
    "config": {
        "audit": {
            "ignore-abandoned": ["acme/*", "acme/pacote"]
        }
    }
}
```

#### Formato detalhado com escopo de aplicação:

O formato detalhado permite controlar se a regra de ignorar se aplica apenas a
relatórios de auditoria, apenas ao bloqueio de versão ou a ambos.
O campo `apply` aceita:

- `audit` - ignorar apenas para relatórios de auditoria (o pacote não aparece
  nos relatórios de auditoria, mas ainda é bloqueado durante atualizações se
  [`block-abandoned`](#block-abandoned) estiver habilitada.
- `block` - ignorar apenas para bloqueio de versão (o pacote pode ser usado
  durante atualizações mesmo se [`block-abandoned`](#block-abandoned) estiver
  habilitada, mas ainda aparece nos relatórios de auditoria).
- `all` - ignorar para relatórios de auditoria e bloqueio de versão
  (comportamento padrão).

```json
{
    "config": {
        "audit": {
            "ignore-abandoned": {
                "acme/pacote": {
                    "apply": "block",
                    "reason": "Permitir durante atualizações, mas ainda reportar como abandonado"
                },
                "vendor/*": {
                    "apply": "all",
                    "reason": "Mantemos esses pacotes internamente"
                }
            }
        }
    }
}
```

Todos os formatos podem ser combinados na mesma configuração.

### ignore-severity

> **Obsoleto.**
> Use [`config.policy.advisories.ignore-severity`](#ignore-severity) em vez
> disso.
> Nota: o novo formato usa valores booleanos `on-block`/`on-audit` em vez de
> `"apply": "audit|block|all"`.

O padrão é `[]`.
Uma lista de níveis de severidade ignorados para relatórios de auditoria e/ou
bloqueio de versão.

#### Formato simples:

```json
{
    "config": {
        "audit": {
            "ignore-severity": ["low", "medium"]
        }
    }
}
```

#### Formato detalhado com escopo de aplicação:

O formato detalhado permite controlar se uma regra de ignorar se aplica apenas a
relatórios de auditoria, apenas ao bloqueio de versões ou a ambos.
O campo `apply` aceita:

- `audit` - ignorar apenas para relatórios de auditoria (avisos com essa
  severidade não aparecem nos relatórios de auditoria, mas os pacotes ainda são
  bloqueados durante atualizações).
- `block` - ignorar apenas para bloqueio de versões (pacotes podem ser usados
  durante atualizações, mas avisos com essa severidade ainda aparecem nos
  relatórios de auditoria).
- `all` - ignorar tanto na auditoria quanto no bloqueio (comportamento padrão).

```json
{
    "config": {
        "audit": {
            "ignore-severity": {
                "low": {
                    "apply": "all"
                },
                "medium": {
                    "apply": "block"
                }
            }
        }
    }
}
```

Todos os formatos podem ser combinados na mesma configuração.

### ignore-unreachable

> **Obsoleto.**
> Use [`config.policy.ignore-unreachable`](#ignore-unreachable) em vez disso.

O padrão é `false`.
Define se repositórios inacessíveis devem ser ignorados durante um
`composer audit`.
Isso pode ser útil se você estiver executando o comando em um ambiente a partir
do qual nem todos os repositórios podem ser acessados.
Essa configuração não se aplica ao bloqueio de versões ou a relatórios de
auditoria gerados fora do comando `composer audit`.

```json
{
    "config": {
        "audit": {
            "ignore-unreachable": true
        }
    }
}
```

### block-insecure

> **Obsoleto.**
> Use [`config.policy.advisories.block`](#block) em vez disso.

O padrão é `true`.
Se definida como `true`, quaisquer versões de pacotes afetadas por avisos de
segurança serão bloqueadas e não poderão ser usadas durante os comandos
`composer update`, `require` ou `delete`, a menos que os avisos de segurança
sejam ignorados.
Se [`block-abandoned`](#block-abandoned) estiver habilitada, o bloqueio de
versões também impedirá o uso de pacotes abandonados.

```json
{
    "config": {
        "audit": {
            "block-insecure": false
        }
    }
}
```

### block-abandoned

> **Obsoleto.**
> Use [`config.policy.abandoned.block`](#block-1) em vez disso.

O padrão é `false`.
Se definida como `true`, pacotes abandonados não poderão ser usados durante os
comandos `composer update`, `require` ou `delete`.
Aplica-se apenas se o bloqueio de versões não tiver sido desabilitado definindo
[`block-insecure`](#block-insecure) como `false`.

```json
{
    "config": {
        "audit": {
            "block-abandoned": true
        }
    }
}
```

## use-parent-dir

Ao executar o Composer em um diretório onde não existe um arquivo
`composer.json`, se houver um presente em um diretório acima, o Composer
perguntará por padrão se você deseja usar o `composer.json` daquele diretório.

Se você quiser sempre responder "sim" a essa pergunta, pode definir esse valor
de configuração como `true`.
Para nunca ser perguntado, defina-o como `false`.
O padrão é `prompt`.

> **Nota:** Esta configuração deve ser definida na sua configuração global de
> usuário para funcionar.
> Use, por exemplo, `php composer.phar config --global use-parent-dir true` para
> defini-la.

## store-auths

Define o que fazer após solicitar autenticação; uma das opções: `true` (sempre
armazenar), `false` (não armazenar) e `prompt` (perguntar sempre).
O padrão é `prompt`.

## github-protocols

O padrão é `["https", "ssh", "git"]`.
Uma lista de protocolos a serem usados ao clonar do github.com, em ordem de
prioridade.
Por padrão, o `git` está incluído, mas apenas se [secure-http](#secure-http)
estiver desabilitada, pois o protocolo git não é criptografado.
Se você quiser que as URLs de push do seu repositório remoto `origin` usem https
em vez de ssh (`git@github.com:...`), defina a lista de protocolos apenas como
`["https"]`; assim, o Composer deixará de sobrescrever a URL de push para uma
URL ssh.

## github-oauth

Uma lista de nomes de domínio e chaves OAuth.
Por exemplo, usar `{"github.com": "token-oauth"}` como valor desta opção fará
com que o `token-oauth` seja usado para acessar repositórios privados no GitHub
e para contornar o limite de requisições (rate limiting) baixo baseado em IP da
API deles.
O Composer pode solicitar credenciais quando necessário, mas elas também podem
ser definidas manualmente.
Leia mais sobre como obter um token OAuth para o GitHub e a sintaxe da CLI
[aqui](articles/authentication-for-private-packages.md#github-oauth).

## gitlab-domains

O padrão é `["gitlab.com"]`.
Uma lista de domínios de servidores GitLab.
Isso é usado se você usar o tipo de repositório `gitlab`.

## gitlab-oauth

Uma lista de nomes de domínio e chaves OAuth.
Por exemplo, usar `{"gitlab.com": "token-oauth"}` como valor desta opção fará
com que o `token-oauth` seja usado para acessar repositórios privados no GitLab.
Observação: se o pacote não estiver hospedado no gitlab.com, os nomes de domínio
também devem ser especificados na opção
[`gitlab-domains`](06-config.md#gitlab-domains).
Mais informações podem ser encontradas
[aqui](articles/authentication-for-private-packages.md#gitlab-oauth).

## gitlab-token

Uma lista de nomes de domínio e tokens privados.
O token privado pode ser uma simples string ou um array contendo nome de usuário
e token.
Por exemplo, usar `{"gitlab.com": "token-privado"}` como valor desta opção fará
com que `token-privado` seja usado para acessar repositórios privados no GitLab.
Usar
`{"gitlab.com": {"username": "usuário-gitlab", "token": "token-privado"}}`
usará tanto o nome de usuário quanto o token para a funcionalidade deploy tokens
do GitLab (https://docs.gitlab.com/ee/user/project/deploy_tokens/).
Observação: se o pacote não estiver hospedado em `gitlab.com`, os nomes de
domínio também devem ser especificados na opção
[`gitlab-domains`](06-config.md#gitlab-domains).
O token deve possuir o escopo `api` ou `read_api`.
Mais informações podem ser encontradas
[aqui](articles/authentication-for-private-packages.md#gitlab-token).

## gitlab-protocol

Um protocolo a ser forçado ao criar a URL do repositório para o valor `source`
nos metadados do pacote.
Pode ser `git` ou `http` (`https` é tratado como sinônimo de `http`).
Útil ao trabalhar com projetos que referenciam repositórios privados que serão
posteriormente clonados em jobs do GitLab CI usando um
[GitLab CI_JOB_TOKEN](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html#predefined-variables-reference)
com autenticação básica HTTP.
Por padrão, o Composer gera uma URL git sobre SSH para repositórios privados e
HTTP(S) apenas para repositórios públicos.

## forgejo-domains

O padrão é `["codeberg.org"]`.
Uma lista de domínios de servidores Forgejo.
Isso é usado se você empregar o tipo de repositório `forgejo`.

## forgejo-token

Uma lista de nomes de domínio e pares de nome de usuário/token de acesso para
autenticação nesses domínios.
Por exemplo, usar
`{"codeberg.org": {"username": "usuário-forgejo", "token": "token-de-accesso"}}`
como valor desta opção permitirá que o Composer se autentique no codeberg.org.
Observação: se o pacote não estiver hospedado no codeberg.org, os nomes de
domínio também deverão ser especificados com a opção
[`forgejo-domains`](06-config.md#forgejo-domains).
Mais informações também podem ser encontradas
[aqui](articles/authentication-for-private-packages.md#forgejo-token).

## disable-tls

O padrão é `false`.
Se definida como `true`, todas as URLs HTTPS serão tentadas via HTTP, e nenhuma
criptografia em nível de rede será realizada.
Habilitar essa opção representa um risco de segurança e NÃO é recomendado.
A melhor abordagem é habilitar a extensão `php_openssl` no `php.ini`.
Habilitar essa opção desabilitará implicitamente a opção `secure-http`.

## secure-http

O padrão é `true`.
Se definida como `true`, apenas URLs HTTPS têm permissão para serem baixadas via
Composer.
Se você precisar realmente de acesso HTTP a algum recurso, pode desabilitar essa
opção; no entanto, usar o [Let's Encrypt](https://letsencrypt.org/) para obter
um certificado SSL gratuito é, geralmente, uma alternativa melhor.

## bitbucket-oauth

Uma lista de nomes de domínio e consumidores.
Por exemplo:
`{"bitbucket.org": {"consumer-key": "minha-chave", "consumer-secret": "meu-segredo"}}`.
Leia mais
[aqui](articles/authentication-for-private-packages.md#bitbucket-oauth).

## cafile

Localização do arquivo da Autoridade Certificadora no sistema de arquivos local.
No PHP 5.6+, recomenda-se definir isso por meio de `openssl.cafile` no
`php.ini`, embora o PHP 5.6+ deva ser capaz de detectar automaticamente o
arquivo de CA do sistema.

## capath

Se `cafile` não for especificado ou se o certificado não for encontrado nele, o
diretório indicado por `capath` será pesquisado em busca de um certificado
adequado.
O `capath` deve ser um diretório de certificados com hash correto.

## http-basic

Uma lista de nomes de domínio e nomes de usuário/senhas para autenticação nesses
domínios.
Por  exemplo, usar `{"example.org": {"username": "alice", "password": "foo"}}`
como valor desta opção permitirá que o Composer se autentique no example.org.
Mais informações podem ser encontradas
[aqui](articles/authentication-for-private-packages.md#http-basic).

## bearer

Uma lista de nomes de domínio e tokens para autenticação nesses domínios.
Por exemplo, usar `{"example.org": "foo"}` como valor desta opção permitirá que
o Composer se autentique no example.org usando um cabeçalho
`Authorization: Bearer foo`.

## platform

Permite simular pacotes de plataforma (PHP e extensões) para que você possa
emular um ambiente de produção ou definir sua plataforma de destino na
configuração.
Exemplo: `{"php": "7.0.3", "ext-alguma-coisa": "4.0.3"}`.

Isso garante que nenhum pacote que exija uma versão superior ao PHP 7.0.3 possa
ser instalado, independentemente da versão real do PHP que você executa
localmente.
No entanto, isso também significa que as dependências não são mais verificadas
corretamente; se você executar o PHP 5.6, a instalação ocorrerá normalmente,
pois o sistema assume a versão 7.0.3, mas a execução falhará posteriormente.
Isso também significa que, se `{"php":"7.4"}` for especificado, não serão
usados pacotes que definam `7.4.1` como versão mínima.

Portanto, ao usar esse recurso, recomenda-se — e é mais seguro — executar também
o comando [`check-platform-reqs`](03-cli.md#check-platform-reqs) como parte de
sua estratégia de implantação.

Se uma dependência exigir uma extensão que você não tem instalada localmente,
você pode ignorá-la passando `--ignore-platform-req=ext-foo` para os comandos
`update`, `install` ou `require`.
A longo prazo, porém, você deve instalar as extensões necessárias; se ignorar
uma agora e um novo pacote adicionado um mês depois também a exigir, você poderá
introduzir problemas em produção sem perceber.

Se você tiver uma extensão instalada localmente, mas *não* em produção, pode
querer ocultá-la artificialmente do Composer usando `{"ext-foo": false}`.

## vendor-dir

O padrão é `vendor`.
Você pode instalar dependências em um diretório diferente, se desejar.
`$HOME` e `~` serão substituídos pelo caminho do seu diretório pessoal em
`vendor-dir` e em todas as opções `*-dir` listadas abaixo.

## bin-dir

O padrão é `vendor/bin`.
Se um projeto incluir binários, serão criados links simbólicos para eles neste
diretório.

## data-dir

O padrão é `C:\Users\<usuário>\AppData\Roaming\Composer` no Windows,
`$XDG_DATA_HOME/composer` em sistemas Unix que seguem as Especificações de
Diretório Base XDG, e `$COMPOSER_HOME` em outros sistemas Unix.
Atualmente, é usado apenas para armazenar arquivos `composer.phar` anteriores,
permitindo reverter para versões mais antigas.
Consulte também [COMPOSER_HOME](03-cli.md#composer-home).

Como o comando `self-update --rollback` restaura um `composer.phar` armazenado
anteriormente neste diretório, ele deve ter permissão de escrita apenas para o
usuário proprietário da instalação do Composer e deve ser tratado como um local
confiável.
Um diretório com permissão de escrita para outros usuários permitiria que eles
inserissem um arquivo `.phar` malicioso que uma reversão com privilégios poderia
instalar.

## cache-dir

O padrão é `C:\Users\<usuário>\AppData\Local\Composer` no Windows,
`/Users/<usuário>/Library/Caches/composer` no macOS, `$XDG_CACHE_HOME/composer`
em sistemas Unix que seguem as Especificações de Diretório Base XDG, e
`$COMPOSER_HOME/cache` em outros sistemas Unix.
Armazena todos os caches usados pelo Composer.
Consulte também [COMPOSER_HOME](03-cli.md#composer-home).

## cache-files-dir

O padrão é `$cache-dir/files`.
Armazena os arquivos zip dos pacotes.

## cache-repo-dir

O padrão é `$cache-dir/repo`.
Armazena metadados de repositório para o tipo `composer` e repositórios VCS dos
tipos `svn`, `fossil`, `github` e `bitbucket`.

## cache-vcs-dir

O padrão é `$cache-dir/vcs`.
Armazena clones VCS para carregar metadados de repositório VCS para os tipos
`git`/`hg` e para acelerar as instalações.

## cache-files-ttl

O padrão é `15552000` (6 meses).
O Composer armazena em cache todos os pacotes `dist` (zip, tar, ...) que baixa.
Por padrão, esses arquivos são removidos após seis meses sem uso.
Esta opção permite ajustar essa duração (em segundos) ou desabilitá-la
completamente, definindo-a como `0`.

## cache-files-maxsize

O padrão é `300MiB`.
O Composer armazena em cache todos os pacotes `dist` (zip, tar, ...) que baixa.
Quando a coleta de lixo é executada periodicamente, esse é o tamanho máximo que
o cache poderá usar.
Arquivos mais antigos (menos usados) serão removidos primeiro até que o cache se
enquadre no limite.

## cache-read-only

O padrão é `false`.
Define se o cache do Composer deve ser usado em modo somente leitura.

## bin-compat

O padrão é `auto`.
Determina a compatibilidade dos binários a serem instalados.
Se for `auto`, o Composer instala arquivos proxy `.bat` apenas no Windows ou
WSL.
Se definida como `full`, tanto os arquivos `.bat` para Windows quanto os scripts
para sistemas operacionais baseados em Unix serão instalados para cada binário.
Isso é útil principalmente se você executa o Composer em uma VM Linux, mas ainda
deseja que os proxies `.bat` estejam disponíveis para uso no sistema operacional
host Windows.
Se definida como `proxy`, o Composer criará apenas arquivos proxy no estilo
bash/Unix e nenhum arquivo `.bat`, mesmo no Windows/WSL.

## prepend-autoloader

O padrão é `true`.
Se for `false`, o autoloader do Composer não será inserido no início da cadeia
de autoloaders existentes.
Isso às vezes é necessário para corrigir problemas de interoperabilidade com
outros autoloaders.

## autoloader-suffix

O padrão é `null`.
Quando definido como uma string não vazia, esse valor será usado como um sufixo
para o autoloader do Composer gerado.
Se definido como `null`, o valor `content-hash` do arquivo `composer.lock` será
usado, se disponível; caso contrário, um sufixo aleatório será gerado.

## optimize-autoloader

O padrão é `false`.
Se for `true`, sempre otimiza ao gerar o autoloader.

## sort-packages

O padrão é `false`.
Se for `true`, o comando `require` mantém os pacotes ordenados por nome no
`composer.json` ao adicionar um pacote.

## classmap-authoritative

O padrão é `false`.
Se for `true`, o autoloader do Composer carregará classes apenas a partir do
mapa de classes.
Implica `optimize-autoloader`.

## apcu-autoloader

O padrão é `false`.
Se for `true`, o autoloader do Composer verificará a presença do APCu e o usará
para armazenar em cache classes encontradas/não encontradas quando a extensão
estiver habilitada.

## github-domains

O padrão é `["github.com"]`.
Uma lista de domínios para usar no modo GitHub.
Isso é usado para configurações do GitHub Enterprise.

## github-expose-hostname

O padrão é `true`.
Se definido como `false`, os tokens OAuth criados para acessar a API do GitHub
conterão uma data em vez do nome de host da máquina.

## use-github-api

O padrão é `true`.
Semelhante à chave `no-api` em um repositório específico, definir
`use-github-api` como `false` estabelece o comportamento global para todos os
repositórios do GitHub: clonar o repositório como qualquer outro repositório
git, em vez de usar a API do GitHub.
No entanto, diferentemente do uso direto do driver `git`, o Composer ainda
tentará usar os arquivos zip do GitHub.

## notify-on-install

O padrão é `true`.
O Composer permite que repositórios definam uma URL de notificação, para serem
notificados sempre que um pacote desse repositório for instalado.
Esta opção permite desabilitar esse comportamento.

## discard-changes

O padrão é `false` e pode assumir os valores `true`, `false` ou `"stash"`.
Esta opção permite definir o comportamento padrão para lidar com atualizações
quando há alterações locais pendentes em modo não interativo.
`true` sempre descartará as alterações nos pacotes dos fornecedores, enquanto
`stash` tentará armazenar temporariamente e reaplicar as alterações.
Use isso em servidores de CI ou scripts de deploy se você costuma ter
dependências modificadas.

## archive-format

O padrão é `tar`.
Substitui o formato padrão usado pelo comando `archive`.

## archive-dir

O padrão é `.`.
Destino padrão para arquivos criados pelo comando `archive`.

Exemplo:

```json
{
    "config": {
        "archive-dir": "/home/<usuário>/.composer/repo"
    }
}
```

## htaccess-protect

O padrão é `true`.
Se definido como `false`, o Composer não criará arquivos `.htaccess` nos
diretórios home, de cache e de dados do Composer.

## lock

O padrão é `true`.
Se definido como `false`, o Composer não criará um arquivo `composer.lock` e o
ignorará caso um já exista.

## platform-check

O padrão é `php-only`, que verifica apenas a versão do PHP.
Defina como `true` para verificar também a presença de extensões.
Se definida como `false`, o Composer não criará nem exigirá um arquivo
`platform_check.php` como parte da inicialização do autoloader.

## secure-svn-domains

O padrão é `[]`.
Lista domínios que devem ser considerados confiáveis/marcados como usando um
transporte seguro de Subversion/SVN.
Por padrão, o protocolo `svn://` é considerado inseguro e gera um erro, mas você
pode definir esta opção de configuração como `["example.org"]` para permitir o
uso de URLs svn nesse nome de host.
Esta é uma alternativa melhor/mais segura do que desabilitar completamente o
`secure-http`.

## bump-after-update

O padrão é `false` e pode ser definido como `true`, `false`, `"dev"` ou
`"no-dev"`.
Se definido como `true`, o Composer executará o comando `bump` após executar
o comando `update`.
Se definido como `dev` ou `no-dev`, apenas as dependências correspondentes
terão suas versões atualizadas.

## allow-missing-requirements

O padrão é `false`.
Ignora erros durante o `install` caso existam requisitos ausentes — ou seja,
quando o arquivo de bloqueio não está atualizado em relação às alterações mais
recentes no `composer.json`.

## update-with-minimal-changes

O padrão é `false`.
Se definido como `true`, o Composer realizará apenas as alterações absolutamente
necessárias nas dependências transitivas durante a atualização.
Também pode ser definida através da variável de ambiente
`COMPOSER_MINIMAL_CHANGES=1`.

&larr; [Repositórios](05-repositories.md) | [Tempo de execução](07-runtime.md) &rarr;
