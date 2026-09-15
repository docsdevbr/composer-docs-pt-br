---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

tagline: Como reduzir o impacto do autoloader no desempenho

source_url: https://github.com/composer/composer/blob/2.10.3/doc/articles/autoloader-optimization.md
source_revision: e58aad45b51c6b29b9f47800a182d7e86e654201
translation_status: ready
---

# Otimização do autoloader

Por padrão, o autoloader do Composer é relativamente rápido.
No entanto, devido à forma como as regras de autoloading PSR-4 e PSR-0 são
configuradas, ele precisa verificar o sistema de arquivos antes de resolver
definitivamente o nome de uma classe.
Isso reduz um pouco a velocidade, mas é conveniente em ambientes de
desenvolvimento, pois, ao adicionar uma nova classe, ela pode ser imediatamente
detectada e utilizada sem a necessidade de reconstruir a configuração do
autoloader.

O problema, entretanto, é que em produção geralmente queremos que tudo ocorra o
mais rápido possível, já que é possível reconstruir a configuração a cada
implantação e novas classes não surgem aleatoriamente entre as implantações.

Por isso, o Composer oferece algumas estratégias para otimizar o autoloader.

> **Nota:** Você **não deve** habilitar nenhuma dessas otimizações em ambientes
> de **desenvolvimento**, pois todas elas causarão diversos problemas ao
> adicionar ou remover classes.
> Os ganhos de desempenho não compensam os transtornos em um cenário de
> desenvolvimento.

## Otimização nível 1: geração de mapa de classes

### Como habilitá-la?

Existem algumas opções para habilitar isso:

- Defina `"optimize-autoloader": true` dentro da chave de configuração do
  `composer.json`.
- Execute `install` ou `update` com a flag `-o` ou `--optimize-autoloader`.
- Execute `dump-autoload` com a flag `-o` ou `--optimize`.

### O que isso faz?

A geração do mapa de classes essencialmente converte regras PSR-4/PSR-0 em
regras de mapa de classes.
Isso torna tudo bem mais rápido, pois, para classes conhecidas, o mapa de
classes retorna o caminho instantaneamente; como o Composer garante que a classe
está lá, não é necessária nenhuma verificação no sistema de arquivos.

No PHP 5.6 ou superior, o mapa de classes também é armazenado em cache no
opcache, o que melhora significativamente o tempo de inicialização.
Se você garantir que o opcache esteja habilitado, o mapa de classes deverá ser
carregado quase instantaneamente, tornando o carregamento de classes rápido.

### Considerações

Não há desvantagens reais com esse método.
Ele deve estar sempre habilitado em produção.

O único problema é que ele não registra falhas de carregamento (ou seja, quando
não consegue encontrar uma determinada classe); nesses casos, o sistema recorre
às regras PSR-4, o que ainda pode resultar em verificações lentas no sistema de
arquivos.
Para resolver esse problema, existem duas opções de otimização de nível 2, e
você pode optar por habilitar uma delas caso realize muitas verificações
`class_exists` para classes que não existem no seu projeto.

## Otimização de nível 2/A: mapas de classes autoritativos

### Como habilitá-la?

Existem algumas opções para habilitar isso:

- Defina `"classmap-authoritative": true` dentro da chave `config` do
  `composer.json`.
- Execute `install` ou `update` com a flag `-a` / `--classmap-authoritative`.
- Execute `dump-autoload` com a flag `-a` / `--classmap-authoritative`.

### O que isso faz?

Habilitar essa opção habilita automaticamente as otimizações de *mapa de
classes* de nível 1.

Essa opção determina que, se algo não for encontrado no *mapa de classes*, então
o item não existe e o autoloader não deve tentar procurá-lo no sistema de
arquivos seguindo as regras da PSR-4.

### Considerações

Essa opção faz com que o autoloader retorne sempre muito rapidamente.
Por outro lado, isso também significa que, caso uma classe seja gerada em tempo
de execução por algum motivo, ela não poderá ser carregada automaticamente.
Se o seu projeto ou alguma de suas dependências fizer isso, você poderá
enfrentar problemas de "classe não encontrada" em produção.
Habilite isso com cautela.

> Nota: Isso não pode ser combinado com as otimizações de nível 2/B.
> Você deve escolher uma delas, pois ambas abordam o mesmo problema de maneiras
> diferentes.

## Otimização de nível 2/B: cache APCu

### Como habilitá-la?

Existem algumas opções para habilitar isso:

- Defina `"apcu-autoloader": true` dentro da chave `config` do `composer.json`.
- Execute `install` ou `update` com `--apcu-autoloader`.
- Execute `dump-autoload` com `--apcu`.

### O que isso faz?

Essa opção adiciona um cache APCu como fallback para o mapa de classes.
No entanto, ela não gera automaticamente o mapa de classes; portanto, você ainda
deve habilitar as otimizações de nível 1 manualmente, se desejar.

Independentemente de uma classe ser encontrada ou não, essa informação é sempre
armazenada em cache na APCu, permitindo um retorno rápido na próxima requisição.

### Considerações

Essa opção requer a APCu, que pode ou não estar disponível para você.
Ela também utiliza memória APCu para fins de autoloading, mas seu uso é seguro e
não resulta em classes não encontradas, ao contrário da otimização de mapa de
classes autoritativo mencionada anteriormente.

> Nota: Isso não pode ser combinado com as otimizações de nível 2/A.
> Você deve escolher uma delas, pois ambas abordam o mesmo problema de maneiras
> diferentes.
