---
# SPDX-FileCopyrightText: Nils Adermann, Jordi Boggiano.
#
# SPDX-License-Identifier: MIT
# Documentation licensed under the MIT License.
# The original work was translated from English into Brazilian Portuguese.
# https://github.com/docsdevbr/composer-docs-pt-br/blob/-/LICENSES/MIT.txt

source_url: https://github.com/composer/composer/blob/2.10.3/doc/faqs/why-are-version-constraints-combining-comparisons-and-wildcards-a-bad-idea.md
source_revision: b12b50c679d6f18398fea165b4bbc5581104c2fa
translation_status: ready
---

# Por que restrições de versão combinando comparações e curingas são uma má ideia?

Um erro bastante comum que as pessoas cometem é definir restrições de versão em
seus pacotes usando formatos como `>=2.*` ou `>=1.1.*`.

Se você analisar o que isso realmente significa, perceberá rapidamente que não
faz muito sentido.
Se decompusermos `>=2.*`, temos duas partes:

- `>=2`, que indica que o pacote deve estar na versão `2.0.0` ou superior.
- `2.*`, que indica que o pacote deve estar entre a versão `2.0.0` (inclusive)
  e a `3.0.0` (exclusive).

Como se pode ver, ambas as regras concordam que o pacote deve ser `>=2.0.0`, mas
não é possível determinar se, ao escrever isso, você estava pensando em um
pacote na versão `3.0.0` ou não.
A versão `3.0.0` deveria ser aceita, já que você pediu `>=2`, ou rejeitada, já
que você pediu `2.*`?

Por isso, o Composer gera um erro indicando que essa definição é inválida.
A solução é refletir sobre o que você realmente deseja e usar apenas uma dessas
regras.
