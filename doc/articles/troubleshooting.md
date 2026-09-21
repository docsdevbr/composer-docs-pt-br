---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/troubleshooting.md
source_revision: 1e7857d682d3d88f9e108623b89f529eb4bf4ac7
translation_status: ready

tagline: Resolvendo problemas
---

# Solução de problemas

Esta é uma lista de problemas comuns ao usar o Composer e como evitá-los.

## Geral

1. Ao enfrentar qualquer tipo de problema ao usar o Composer, certifique-se de
   **utilizar a versão mais recente**.
   Veja [self-update](../03-cli.md#self-update) para mais detalhes.

2. Antes de pedir ajuda, execute [`composer diagnose`](../03-cli.md#diagnose)
   para verificar problemas comuns.
   Se tudo estiver correto, prossiga para as próximas etapas.

3. Certifique-se de que não há problemas com sua configuração executando as
   verificações do instalador via
   `curl -sS https://getcomposer.org/installer | php -- --check`.

4. Tente limpar o cache do Composer executando `composer clear-cache`.

5. Ao solucionar problemas, certifique-se de **instalar as dependências
   diretamente a partir do seu `composer.json`** usando
   `rm -rf vendor && composer update -v`; isso elimina possíveis interferências
   de instalações existentes na pasta `vendor` ou de entradas no arquivo
   `composer.lock`.

## Pacote não encontrado

1. Verifique se **não há erros de digitação** no seu `composer.json` ou nos
   nomes de branches e tags do repositório.

2. Certifique-se de definir a
   **[minimum-stability](../04-schema.md#minimum-stability)** correta.
   Para começar, ou para garantir que isso não seja um problema, defina
   `minimum-stability` como "dev".

3. Pacotes que **não vêm do [Packagist](https://packagist.org/)** devem sempre
   ser **definidos no pacote raiz** (o pacote que depende de todas as
   dependências).

4. Use o **mesmo nome de fornecedor e de pacote** em todos os branches e tags do
   seu repositório, especialmente ao manter um fork de terceiros e utilizar o
   `replace`.

5. Se você estiver atualizando para uma versão recém-publicada de um pacote,
   lembre-se de que o Packagist pode levar até 1 minuto para tornar novos
   pacotes visíveis ao Composer.

6. Se você estiver atualizando um único pacote, ele próprio pode depender de
   versões mais recentes.
   Nesse caso, adicione o argumento `--with-dependencies` **ou** inclua no
   comando todas as dependências que precisam ser atualizadas.

## O pacote não está sendo atualizado para a versão esperada

Tente executar `php composer.phar why-not [package-name] [expected-version]`.

## Dependências no pacote raiz

Quando o seu pacote raiz depende de um pacote que, por sua vez, depende (direta
ou indiretamente) do próprio pacote raiz, podem ocorrer problemas em dois casos:

1. Durante o desenvolvimento, se você estiver em um branch como `dev-main` e ele
   não tiver um [branch-alias](aliases.md#alias-de-branch) definido, e a
   dependência no pacote raiz exigir a versão `^2.0`, por exemplo, a versão
   `dev-main` não atenderá a esse requisito.
   A melhor solução aqui é garantir que você defina um alias de branch.

2. Em execuções de CI (Integração Contínua), o problema pode ser que o Composer
   não consiga detectar corretamente a versão do pacote raiz.
   Se for um clone do git, geralmente funciona bem e o Composer detecta a versão
   do branch atual; no entanto, alguns sistemas de CI realizam clones
   superficiais, o que pode fazer com que o processo falhe ao testar pull
   requests e feature branches.
   Nesses casos, o alias do branch pode não ser reconhecido.
   A melhor solução é definir a versão em que você está por meio de uma variável
   de ambiente chamada `COMPOSER_ROOT_VERSION`.
   Você pode defini-la como `dev-main`, por exemplo, para definir a versão do
   pacote raiz como `dev-main`.
   Use, por exemplo: `COMPOSER_ROOT_VERSION=dev-main composer install` para
   exportar a variável apenas para a chamada do composer, ou defina-a
   globalmente nas variáveis de ambiente do CI.

## Detecção da versão do pacote raiz

O Composer precisa saber a versão do pacote raiz para resolver dependências de
forma eficaz.
A versão do pacote raiz é determinada usando uma abordagem hierárquica:

1. **Campo `version` no `composer.json`**: primeiramente, o Composer procura por
   um campo `version` no arquivo `composer.json` da raiz do projeto.
   Se estiver presente, esse campo especifica diretamente a versão do pacote
   raiz.
   Isso geralmente não é recomendado, pois exige atualização constante, mas é
   uma opção.

2. **Variável de ambiente**: em seguida, o Composer verifica a variável de
   ambiente `COMPOSER_ROOT_VERSION`.
   Essa variável pode ser definida explicitamente pela pessoa usuária para
   determinar a versão do pacote raiz, oferecendo uma maneira direta de informar
   ao Composer a versão exata, especialmente em ambientes de CI/CD ou quando o
   método VCS não é aplicável.

3. **Inspeção do Sistema de Controle de Versão (VCS)**: o Composer tenta então
   deduzir a versão interagindo com o sistema de controle de versão do projeto.
   Por exemplo, em projetos versionados com Git, o Composer executa comandos
   específicos do Git para deduzir a versão atual do projeto com base em tags,
   branches e histórico de commits.
   Se o diretório `.git` estiver ausente ou o histórico estiver incompleto, por
   exemplo, se o CI estiver usando um clone superficial, essa detecção pode
   falhar em encontrar a versão correta.

4. **Alternativa de contingência**: se tudo mais falhar, o Composer usa `1.0.0`
   como versão padrão.

Observe que depender da versão/alternativa de contingência pode levar a
problemas de resolução de dependências, especialmente quando o pacote raiz
depende de um pacote que, por sua vez, depende (direta ou indiretamente)
[do próprio pacote raiz](#dependências-no-pacote-raiz).

## Problemas de tempo limite de rede, erro do cURL

Se você encontrar uma mensagem semelhante a esta:

```
Failed to download * curl error 28 while downloading * Operation timed out after 300000 milliseconds
```

Significa que sua rede provavelmente está tão lenta que uma requisição levou
mais de 300 segundos para ser concluída.
Esse é o tempo limite mínimo que o Composer utiliza, mas você pode aumentá-lo
alterando o valor de `default_socket_timeout` no seu `php.ini` para um valor
mais alto.

## Pacote não encontrado em uma construção do Jenkins

1. Consulte o item ["Pacote não encontrado"](#pacote-não-encontrado) acima.

2. A operação de `git-clone` ou `checkout` no Jenkins deixa o branch em um
   estado de "detached HEAD" (HEAD desanexado).
   Como resultado, o Composer pode não conseguir identificar a versão do branch
   que está sendo utilizada e pode não conseguir resolver uma
   [dependência do pacote raiz](#dependências-no-pacote-raiz).
   Para resolver esse problema, você pode utilizar a opção "Additional
   Behaviours" (Comportamentos adicionais) -> "Check out to specific local
   branch" (Fazer checkout em um branch local específico) nas configurações do
   Git da sua tarefa no Jenkins, definindo o branch local como o memso branch
   que você está selecionando no checkout.
   Dessa forma, o checkout não ficará mais em estado desanexado e a dependência
   do pacote raiz deverá ser satisfeita.

## Tenho uma dependência que contém uma definição de "repositories" no seu `composer.json`, mas ela parece estar sendo ignorada.

A propriedade de configuração [`repositories`](../04-schema.md#repositories) é
definida como [root-only](../04-schema.md#pacote-raiz).
Ela não é herdada.
Você pode ler mais sobre os motivos disso no artigo
"[Por que o Composer não consegue carregar repositórios recursivamente?](../faqs/why-cant-composer-load-repositories-recursively.md)".
A solução alternativa mais simples para essa limitação é mover ou duplicar a
definição de `repositories` para o seu arquivo `composer.json` raiz.

## Fixei uma dependência em um commit específico, mas estou obtendo resultados inesperados.

Embora o Composer ofereça suporte à fixação de dependências em um commit
específico usando a sintaxe `#commit-ref`, existem algumas ressalvas que devem
ser levadas em consideração.
A mais importante está [documentada](../04-schema.md#links-de-pacotes), mas é
frequentemente ignorada:

> **Nota:** Embora isso seja conveniente às vezes, não é como você deve utilizar
> pacotes a longo prazo, pois envolve uma limitação técnica.
> Os metadados do `composer.json` ainda serão lidos a partir do nome do branch
> que você especificar antes do hash.
> Por isso, em alguns casos, essa não será uma solução prática, e você deve
> sempre tentar migrar para versões com tag assim que possível.

Não existe uma solução alternativa simples para essa limitação.
Portanto, recomenda-se fortemente que você não a utilize.

## Precisa substituir a versão de um pacote?

Digamos que seu projeto dependa do pacote A, que por sua vez depende de uma
versão específica do pacote B (por exemplo, 0.1).
Mas você precisa de uma versão diferente desse pacote B (por exemplo, 0.11).

Você pode resolver isso criando um alias da versão 0.11 para a 0.1:

`composer.json`:

```json
{
    "require": {
        "A": "0.2",
        "B": "0.11 as 0.1"
    }
}
```

Consulte [aliases](aliases.md) para mais informações.

## Identificando a origem de um valor de configuração

Use `php composer.phar config --list --source` para ver a origem de cada valor
de configuração.

## Erros de limite de memória

A primeira providência é garantir que você esteja utilizando o Composer 2 e, se
possível, a versão 2.2.0 ou superior.

O Composer 1 consumia muito mais memória; atualizar para a versão mais recente
proporcionará resultados muito melhores e mais rápidos.

Às vezes, o Composer pode falhar na execução de alguns comandos, exibindo a
seguinte mensagem:

`PHP Fatal error:  Allowed memory size of XXXXXX bytes exhausted <...>`

Nesse caso, é necessário aumentar o `memory_limit` do PHP.

> **Nota:** O Composer aumenta internamente o `memory_limit` para `1.5G`.

Para obter o valor atual do `memory_limit`, execute:

```shell
php -r "echo ini_get('memory_limit').PHP_EOL;"
```

Tente aumentar o limite no seu arquivo `php.ini` (por exemplo,
`/etc/php5/cli/php.ini` para sistemas baseados em Debian):

```ini
; Use -1 for unlimited or define an explicit value like 2G
memory_limit = -1
```

O Composer também respeita um limite de memória definido pela variável de
ambiente `COMPOSER_MEMORY_LIMIT`:

```shell
COMPOSER_MEMORY_LIMIT=-1 composer.phar <...>
```

Ou você pode aumentar o limite com um argumento de linha de comando:

```shell
php -d memory_limit=-1 composer.phar <...>
```

No entanto, observe que definir o limite de memória usando esses métodos resolve
principalmente problemas de memória no próprio Composer e em seus processos
imediatos.
Processos filhos ou comandos externos invocados pelo Composer ainda podem exigir
ajustes separados, caso possuam seus próprios requisitos de memória.

Esse problema também pode ocorrer em instâncias do cPanel quando a proteção
contra fork bomb no shell está habilitada.
Para mais informações, consulte a
[documentação](https://documentation.cpanel.net/display/68Docs/Shell+Fork+Bomb+Protection)
sobre o recurso de proteção contra fork bomb no site do cPanel.

## Impacto do Xdebug no Composer

Para melhorar o desempenho quando a extensão Xdebug está habilitada, o Composer
reinicia automaticamente o PHP sem ela.
Você pode substituir esse comportamento usando uma variável de ambiente:
`COMPOSER_ALLOW_XDEBUG=1`.

O Composer sempre exibirá um aviso se o Xdebug estiver sendo usado, mas você
pode desativar esse aviso com uma variável de ambiente:
`COMPOSER_DISABLE_XDEBUG_WARN=1`.
Se você vir esse aviso inesperadamente, significa que o processo de
reinicialização falhou: por favor, relate esse
[problema](https://github.com/composer/composer/issues).

## "O sistema não pode encontrar o caminho especificado" (Windows)

1. Abra o regedit.
2. Procure por uma chave `AutoRun` em
   `HKEY_LOCAL_MACHINE\Software\Microsoft\Command Processor`,
   `HKEY_CURRENT_USER\Software\Microsoft\Command Processor` ou
   `HKEY_LOCAL_MACHINE\Software\Wow6432Node\Microsoft\Command Processor`.
3. Verifique se ela contém algum caminho para um arquivo inexistente; se for o
   caso, remova-o.

## Problema com certificado SSL: não foi possível obter o certificado do emissor local

1. Verifique se o seu repositório de certificados raiz (ou pacote de CA) está
   atualizado.
   Execute `composer diagnose -vvv` e procure pelas linhas `Checked CA file ...`
   ou `Checked directory ...` no início da saída do comando.
   Isso mostrará onde o Composer está procurando pelo pacote de CA.
   Você pode obter um
   [novo arquivo cacert.pem no site do cURL](https://curl.se/docs/caextract.html)
   e salvá-lo nesse local.
2. Se isso não resolver o problema, mesmo que o Composer encontre um pacote de
   CA válido, tente desativar seu antivírus e o firewall para verificar se isso
   ajuda.
   Já observamos casos em que o Avast no Windows, por exemplo, impediu o
   funcionamento correto do Composer.
   Para desativar a verificação de HTTPS no Avast, vá em "Proteção > Módulos
   principais > Módulo Web > **desmarque** a opção Ativar verificação de HTTPS".
   Se isso resolver, você deve relatar o problema ao fabricante do software para
   que eles possam, quem sabe, fazer melhorias.

## Limite de taxa da API e tokens OAuth

Devido aos limites de taxa da API do GitHub, pode acontecer de o Composer
solicitar autenticação, pedindo seu nome de usuário e senha, para prosseguir com
a execução.

Caso prefira não fornecer suas credenciais do GitHub ao Composer, você pode
criar um token manualmente seguindo o
[procedimento documentado aqui](authentication-for-private-packages.md#github-oauth).

Agora, o Composer deverá realizar a instalação ou atualização sem solicitar
autenticação.

## Erros `proc_open(): fork failed`

Se o Composer apresentar o erro `proc_open(): fork failed` ao executar alguns
comandos:

`PHP Fatal error: Uncaught exception 'ErrorException' with message 'proc_open(): fork failed - Cannot allocate memory' in phar`

Isso pode estar ocorrendo porque a VPS ficou sem memória e não possui espaço de
swap habilitado.

```shell
free -m
```
```text
total used free shared buffers cached
Mem: 2048 357 1690 0 0 237
-/+ buffers/cache: 119 1928
Swap: 0 0 0
```

Para habilitar o swap, você pode usar, por exemplo:

```shell
/bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
/sbin/mkswap /var/swap.1
/bin/chmod 0600 /var/swap.1
/sbin/swapon /var/swap.1
```

Você pode criar um arquivo de swap permanente seguindo este
[tutorial](https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-ubuntu-14-04).

## Erros `proc_open(): failed to open stream` (Windows)

Se o Composer apresentar erros `proc_open(NUL)` no Windows:

`proc_open(NUL): failed to open stream: No such file or directory`

Isso pode estar acontecendo porque você está trabalhando em um diretório do
_OneDrive_ e utilizando uma versão do PHP que não oferece suporte à semântica do
sistema de arquivos desse serviço.
O problema foi corrigido nas versões 7.2.23 e 7.3.10 do PHP.

Alternativamente, o problema pode ocorrer porque o serviço Null do Windows não
está habilitado.
Para mais informações, consulte esta
[issue](https://github.com/composer/composer/issues/7186#issuecomment-373134916).

## Modo de operação reduzida

Devido a alguns problemas intermitentes no Travis e em outros sistemas,
introduzimos um modo de rede de operação reduzida que ajuda o Composer a
concluir a execução com sucesso, embora desabilite algumas otimizações.
Esse modo é ativado automaticamente assim que um problema é detectado.
Se você encontrar esse problema esporadicamente, provavelmente não precisa se
preocupar (uma rede lenta ou sobrecarregada também pode causar esses timeouts;
no entanto, se ele ocorrer repetidamente, convém verificar as opções abaixo para
identificar e resolver a situação.

Se você foi direcionado a esta página, verifique os seguintes pontos:

- Se estiver usando o antivírus ESET, vá em "Configurações Avançadas" e
  desabilite o "Scanner HTTP" na seção "Proteção de acesso à web".
- Se estiver usando IPv6, tente desativá-lo.
  Se isso resolver seus problemas, entre em contato com seu provedor de internet
  ou com a empresa de hospedagem do servidor; o problema não está no Packagist,
  mas nas regras de roteamento entre você e o Packagist (ou seja, na
  infraestrutura geral da internet).
  A melhor maneira de solucionar isso é alertar as pessoas engenheiras de rede
  que têm autonomia para corrigir o problema.
  Consulte a próxima seção para ver soluções alternativas para IPv6.
- Se nenhuma das opções acima ajudar, por favor, relate o erro.

## Tempo limite da operação excedido (problemas com IPv6)

Você pode encontrar erros se o IPv6 não estiver configurado corretamente.
Um erro comum é:

```text
The "https://getcomposer.org/version" file could not be downloaded: failed to
open stream: Operation timed out
```

Recomendamos corrigir sua configuração de IPv6.
Se isso não for possível, você pode tentar as seguintes soluções alternativas:

**Solução alternativa genérica:**

Defina a variável de ambiente
[`COMPOSER_IPRESOLVE=4`](../03-cli.md#composer-ipresolve), o que forçará o curl
a resolver domínios usando IPv4.
Isso funciona apenas quando a extensão curl é utilizada para os downloads.

**Solução alternativa para Linux:**

No Linux, parece que executar este comando ajuda a dar prioridade ao tráfego
IPv4 sobre o IPv6, o que é uma alternativa melhor do que desativar o IPv6
completamente:

```shell
sudo sh -c "echo 'precedence ::ffff:0:0/96 100' >> /etc/gai.conf"
```

**Solução alternativa para Windows:**

No Windows, receio que a única maneira seja desativar o IPv6 completamente (seja
no Windows ou no seu roteador doméstico).

**Solução alternativa para Mac OS X:**

Obtenha o nome do seu dispositivo de rede:

```shell
networksetup -listallnetworkservices
```

Desabilite o IPv6 nesse dispositivo (neste caso, "Wi-Fi"):

```shell
networksetup -setv6off Wi-Fi
```

Execute o Composer...

Você pode reabilitar o IPv6 com:

```shell
networksetup -setv6automatic Wi-Fi
```

Dito isso, se isso resolver o seu problema, por favor, entre em contato com seu
provedor de internet para tentar solucionar os erros de roteamento.
Essa é a melhor maneira de resolver a situação para todas as pessoas.

## O Composer trava com o SSH ControlMaster

Ao tentar instalar pacotes de um repositório Git utilizando a configuração
`ControlMaster`para sua conexão SSH, o Composer pode travar indefinidamente, e
você verá um processo `sh` no estado `defunct` na sua lista de processos.

A causa disso é um bug no SSH:
https://bugzilla.mindrot.org/show_bug.cgi?id=1988.

Como solução alternativa, abra uma conexão SSH com o host do Git antes de
executar o Composer:

```shell
ssh -t git@mygitserver.tld
php composer.phar update
```

Consulte também https://github.com/composer/composer/issues/4180 para mais
informações.

## Arquivos ZIP não são descompactados corretamente.

O Composer pode descompactar arquivos ZIP usando o utilitário `unzip` ou `7z`
(7-Zip) do sistema, ou a classe nativa `ZipArchive` do PHP.
Em sistemas operacionais onde arquivos ZIP podem conter permissões e links
simbólicos, recomendamos instalar o `unzip` ou o `7z`, pois esses recursos não
são suportados pela classe `ZipArchive`.

## Desativando o otimizador de pool

No Composer, a classe `Pool` contém todos os pacotes relevantes para o processo
de resolução de dependências.
Ela é utilizada para gerar todas as regras que são, então, passadas para o
resolvedor de dependências.
Para melhorar o desempenho, o Composer tenta otimizar esse `Pool` removendo
precocemente informações de pacotes que não são necessárias.

Se tudo correr bem, você não deverá notar problemas com isso; no entanto, caso
se depare com um resultado inesperado, como um conjunto de dependências
impossível de resolver ou conflitos nos quais você acredita que o Composer
esteja errado, você pode desativar o otimizador usando a variável de ambiente
`COMPOSER_POOL_OPTIMIZER` e executar a atualização novamente, desta forma:

```shell
COMPOSER_POOL_OPTIMIZER=0 php composer.phar update
```

Agora, verifique se o resultado permanece o mesmo.
O processo de resolução de dependências levará significativamente mais tempo e
consumirá muito mais memória.

Se o resultado for diferente, é provável que você tenha encontrado um problema
no otimizador de pool.
Por favor, [relate esse problema](https://github.com/composer/composer/issues)
para que ele possa ser corrigido.
