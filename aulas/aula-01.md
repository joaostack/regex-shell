# Aula 1 - O que são expressões regulares

Expressões regulares (ou *regex*, de __reg__ular __ex__pression, como é popularmente conhecida) são expressões que descrevem padrões capazes de representar conjuntos de textos. Sua principal finalidade é localizar sequências de caracteres que correspondam aos padrões descritos -- quando a correspondência é encontrada, nós dizemos que houve um "casamento" (*match*, em inglês).

## 1.1 Como assim, "padrões"?

Um padrão diz respeito a qualquer conjunto de elementos que caracterizem o texto buscado. Por exemplo, se observarmos as linhas do arquivo `/etc/passwd`, nós veremos que todas elas contém campos de dados separados pelo caractere `:`.

```
:~$ head /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
```

Portanto, cada uma dessas linhas segue um padrão que nós poderíamos descrever como: *sete conjuntos compostos de zero ou mais caracteres quaisquer intercalados pelo caractere `:`*.

Com uma expressão regular, a descrição dessas linhas poderia ser feita assim:

```
^.*:.*:.*:.*:.*:.*:.*$
```

Onde:

```
^  → representa o início da linha;
.* → representa zero ou mais caracteres quaisquer;
$  → representa o fim da linha.
```

Mas a nossa expressão regular não precisa ser tão grande, observe:

```
^(.*:){6}.*$
```

A descrição do padrão é basicamente a mesma, nós só utilizamos os mecanismos das expressões regulares para montar um representação mais sucinta:

```
(.*:) → grupo de zero ou mais caracteres quaisquer seguidos de `:`;
{6}   → a entidade anterior (o grupo) ocorre seis vezes seguidas na linha;
.*    → zero ou mais caracteres quaisquer.
```

Dependendo do que estivermos procurando, pode ser o caso de descrevermos apenas algo que está nas linhas do texto em vez de descrever o padrão de uma linha inteira. Por exemplo, digamos que o objetivo seja encontrar a hora em um texto produzido pela saída do utilitário `date`:

```
:~$ date
qui 05 out 2021 08:42:28 -03
```

Sem argumentos, a hora informada pelo `date` pode ser descrita como: *três grupos de dois dígitos intercalados pelo caractere `:`*. Pensando de forma simplificada, este padrão pode ser descrito com a expressão regular abaixo:

```
[0-9][0-9]:[0-9][0-9]:[0-9][0-9]
```

Onde `[0-9]` é um dígito qualquer entre `0` e `9`.

Certamente, essa expressão casaria com todas as horas, minutos e segundos possíveis, mas ela não é específica o suficiente para que, analisando apenas o que nós escrevemos, outras pessoas sejam capazes de dizer que estamos buscando uma hora. Além disso, se o objetivo fosse validar uma *string* (uma sequência de caracteres), sequências como `92:75:83`, por exemplo, passariam como horas válidas. Neste caso, nós temos que ser mais específicos:

```
([01][0-9]|2[0-3])(:[0-5][0-9]){2}
```

Destrinchando:

```
([01][0-9]|2[0-3]) → Dígito 0 ou 1 seguidos de qualquer dígito ou
                     o dígito 2 seguido de um dígito de 0 a 3.

(:[0-5][0-9]){2}   → Dois grupos contendo um dígito de 0 a 5
                     seguido de qualquer dígito.
```

Muitas vezes, o que nós precisamos é encontrar uma palavra que ocorra em uma posição específica de uma linha de texto. Acompanhe o exemplo:

```
:~$ man -k nano
nano (1)             - Nano's ANOther editor, inspirado no Pico
clock_nanosleep (2)  - high-resolution sleep with specifiable clock
editor (1)           - Nano's ANOther editor, inspired by Pico
futimens (3)         - change file timestamps with nanosecond precision
nanorc (5)           - GNU nano's configuration file
nanosleep (2)        - high-resolution sleep
pico (1)             - Nano's ANOther editor, inspired by Pico
rnano (1)            - a restricted nano
traceroute-nanog (1) - print the route packets trace to network host
utimensat (2)        - change file timestamps with nanosecond precision

```

O comando `man -k` (*apropos*) exibe uma lista das páginas de manual que contenham textos que casem com a expressão regular informada no argumento seguinte. No exemplo, nós descrevemos o padrão com a sequência de caracteres `n-a-n-o`, mas ela ocorre em vários pontos diferentes da saída do comando. Observando o padrão das linhas, se quisermos ver apenas a linha que fala do manual do editor Nano, nós temos que especificar que o padrão casa com o início da linha:

```
:~$ man --regex -k '^nano'
nano (1)             - Nano's ANOther editor, inspirado no Pico
editor (1)           - Nano's ANOther editor, inspired by Pico
nanorc (5)           - GNU nano's configuration file
nanosleep (2)        - high-resolution sleep
pico (1)             - Nano's ANOther editor, inspired by Pico
```

Mas nós temos um problema: a implementação de expressões regulares do `man -k` é muito básica, não distingue maiúsculas e minúsculas e considera "início de linha" as ocorrências do padrão tanto no início do nome da página de manual quanto no início da sua descrição. Para resolver esse problema, vamos recorrer a outro utilitário que filtra linhas de texto a partir de expressões regulares -- o `grep`:

```
:~$ man -k '.*' | grep '^nano'
nano (1)             - Nano's ANOther editor, inspirado no Pico
nanorc (5)           - GNU nano's configuration file
nanosleep (2)        - high-resolution sleep
```

Excelente! Nós utilizamos a expressão `.*` para listar todas as páginas de manual e a expressão `^nano`, no `grep`, para filtrar apenas as linhas que contém a sequência `nano` no início. Mas isso não foi suficiente, porque tivemos dois resultados adicionais casando com o nosso padrão. Mais uma vez, temos que ser específicos:

```
:~$ man -k '.*' | grep '^nano\b'
nano (1)             - Nano's ANOther editor, inspirado no Pico
```

Agora sim! Com o `\b` nós estamos informando onde termina a palavra buscada.

A esta altura, já deve estar claro que a descrição de um padrão em uma expressão regular é feita combinando caracteres e outros símbolos gráficos que, dependendo da forma que são escritos, podem representar algo totalmente diferente dos seus significados originais: estamos falando dos **metacaracteres**.

> A forma correta de escrever *"metacaracteres"* é **"meta caracteres"**. Porém, a forma errada foi tão difundida que chegou a contaminar os resultados de buscas sobre o assunto na internet. Por este motivo, nós achamos melhor perpetuar o erro.

## 1.2 - Metacaracteres

Metacaracteres são, como o nome sugere, caracteres que vão além de seu papel normal de símbolos gráficos em um texto. No contexto da construção de expressões regulares, isso significa que alguns caracteres assumirão algum papel especial, que pode ser de:

* Representar a ocorrência de caracteres (representantes e classes);
* Descrever a quantidade em que esses caracteres ocorrem (quantificadores);
* Especificar com que parte do texto o padrão deve coincidir (âncoras);
* Alterar a precedência, a associação e a relação entre os padrões presentes na expressão (operadores diversos);
* Transformar metacaracteres em caracteres normais e vice-versa (escape);

A tabela abaixo mostra os metacaracteres comuns à maioria das implementações de expressões regulares para uso no shell de sistemas *unix-like* e GNU:

| Meta | Tipo | Descrição |
|---|---|---|
| `.`         | Representante | Qualquer caractere único.                                                                       |
| `[...]`     | Representante | Um caractere na lista.                                                                          |
| `[^...]`    | Representante | Um caractere fora da lista.                                                                     |
| `\w`        | Classe        | Um caractere válido em palavras: `[A-Za-z0-9_]`                                                 |
| `\W`        | Classe        | Um caractere inválido em palavras: `[^A-Za-z0-9_]`                                              |
| `\s`        | Classe        | Um caractere em branco.                                                                         |
| `\S`        | Classe        | Tudo menos caracteres em branco.                                                                |
| `?`         | Quantificador | A entidade anterior (caractere literal ou representante) pode ocorrer zero ou uma vez.          |
| `*`         | Quantificador | A entidade anterior pode ocorrer zero ou mais vezes.                                            |
| `+`         | Quantificador | A entidade anterior pode ocorrer uma ou mais vezes.                                             |
| `{min,max}` | Quantificador | A entidade anterior pode ocorrer em qualquer quantidade entre o mínimo e o máximo (inclusive).  |
| `^`         | Âncora        | Marca o começo de uma linha.                                                                    |
| `$`         | Âncora        | Marca o fim de uma linha.                                                                       |
| `\b`        | Âncora        | Especifica que o padrão ocorre no início ou no fim de uma palavra.                              |
| `\B`        | Âncora        | Especifica que o padrão **não ocorre** no início ou no fim de uma palavra.                      |
| `\`         | Escape        | Remove o significado especial de metacaracteres (ou atribui significado especial a caracteres). |
| `\|`        | Operador      | Expressa padrões alternativos (um ou outro).                                                    |
| `(...)`     | Grupo         | Agrupa, aninha e define a precedência e a associação de padrões.                                |
| `\1...\9`   | Retrovisor    | Reutiliza padrões casados em grupos anteriores conforme a ordem das ocorrências (máximo de 9).  |


> Com esses metacaracteres, é possível representar praticamente qualquer padrão de texto.

### Importante!

Os metacaracteres `?`, `*` e a lista `[...]` não devem ser confundidos com aqueles utilizados na formação de padrões de nomes de arquivos. A rigor, seus equivalentes em expressões regulares seriam:

| Shell    | Regex    | Descrição                              |
|----------|----------|----------------------------------------|
| `*`      | `.*`     | Zero ou mais caracteres quaisquer.     |
| `?`      | `.`      | Exatamente um caractere qualquer.      |
| `[...]`  | `[...]`  | Exatamente um caractere da lista.      |
| `[!...]` | `[^...]` | Exatamente um caractere fora da lista. |

### O metacaractere de escape

Embora seja sempre lembrado por remover os "poderes especiais" de metacaracteres (*kriptonita*, como diz Aurélio Jargas), a barra invertida também faz com que caracteres normais ganhem "poderes especiais", como acontece com algumas classes de caracteres (`\s`, `\w`, etc...), as âncoras das bordas (`\b` e `\B`) e com os retrovisores (`\1...\9`).

Algumas implementações de expressões regulares também podem exigir o escape de metacaracteres para que eles se comportem como metacaracteres.

Observe:

```
:~$ var='2 5 10'
:~$ grep '[0-9]' <<< $var
[2] [5] [10]
```

> Os trechos entre colchetes são os casamentos encontrados pelo `grep`.

Se quisermos que o casamento ocorra apenas com dois dígitos numéricos seguidos, nós podemos utilizar o quantificador `{2}` (exatamente duas ocorrências da entidade anterior):

```
:~$ var='2 5 10'
:~$ grep '[0-9]{2}' <<< $var
:~$
```

Aparentemente, nenhum casamento foi encontrado, mas isso é um "falso negativo", como podemos confirmar escapando as chaves:

```
:~$ var='2 5 10'
:~$ grep '[0-9]\{2\}' <<< $var
2 5 [10]
```

Esse comportamento pode ser alterado por opções explícitas, como a opção `-E` do `grep`:

```

:~$ var='2 5 10'
:~$ grep -E '[0-9]{2}' <<< $var
2 5 [10]
```

É justamente este comportamento que caracteriza os dois modos de expressões regulares do `grep`: os modos **básico** e **estendido**: no modo básico, os metacaracteres `?`, `+`, `{`, `|`, `(` e `)` perdem seu significado especial, a menos que sejam escapados ou o modo estendido seja habilitado com a opção `-E`.

> O utilitário `sed` também trabalha com expressões regulares básicas e estendidas.

## 1.3 - Representantes e classes

Os metacaracteres representantes são aqueles que, como o nome diz, representam a ocorrência de um (e apenas um) caractere. Nesta categoria encontram-se:

| Metacaractere | Descrição                   |
|---------------|-----------------------------|
| `.`           | Qualquer caractere único.   |
| `[...]`       | Um caractere na lista.      |
| `[^...]`      | Um caractere fora da lista. |

> Uma nota importante antes de entrarmos nos próximos exemplos, é que o `grep` sempre retorna **linhas inteiras** que contenham o padrão descrito na expressão regular. Para obter apenas as partes das linhas que casarem com o padrão, é preciso informar a opção `-o` (*only*). Deste modo, cada correspondência encontrada será exibida em uma linha na saída.

### Ponto

O ponto casa com exatamente um caractere qualquer:

```
:~$ var='banana bacana barata bandana'
:~$ grep -o 'ba.a.a' <<< $var
banana
bacana
barata
```

Ele também casa com espaços e o próprio ponto:

```
:~$ num='2.75 2,75 2 75'
:~$ grep -o '2.75' <<< $num
2.75
2,75
2 75
```

Quando precisarmos representar um caractere ponto textual, ele tem que ser escapado:

```
:~$ num='2.75 2,75 2 75'
:~$ grep -o '2\.75' <<< $num
2.75
```

Mas o ponto não casa com o caractere de fim de linha:

```
:~$ var=$'1234\n'
:~$ grep '....' <<< "$var"
1234
:~$ grep '.....' <<< "$var"
:~$
```

No exemplo, nós utilizamos uma expansão de caracteres ANSI-C (`$'...'`) para que a quebra de linha (`\n`) fosse acrescentada como um quinto caractere à *string* atribuída a `var`.

Quando descrevemos o padrão buscado com quatro pontos, o `grep` nos retornou uma linha contendo os 4 primeiros caracteres em `var`. Mas, quando tentamos descrever um quinto caractere qualquer no padrão, nada foi retornado: o padrão simplesmente não tem correspondência na linha recebida pelo `grep`.

#### Desafio!

Se o que acabamos de dizer está correto, como você explica a saída abaixo?

```
:~$ var=$'1234\n'
:~$ grep '.*' <<< "$var"
1234

:~$
```

### Lista

Ao contrário do ponto, a lista só permite o casamento com caracteres especificados entre os colchetes, o que permite a construção de expressões regulares bem mais específicas.

Observe:

```
:~$ grep -o 'b.ta' <<< $'bata beta bota b\tta'
bata
beta
bota
b       ta
```

Neste comando, com o ponto, o segundo caractere pode ser qualquer coisa, inclusive caracteres como a tabulação na quarta palavra. Pode ser que isso seja desejável em algumas situações, mas é sempre bom avaliar se não podemos ser mais específicos, como no exemplo abaixo:

```
:~$ grep -o 'b[ao]ta' <<< $'bata beta bota b\tta'
bata
bota
```

Aqui, nós deixamos claro que queremos apenas dois caracteres na segunda posição (`a` ou `o`), o que forma um padrão que casa com apenas duas palavras: `bata` ou `bota`.

#### Sem significados especiais

Dentro dos colchetes de uma lista, todos os caracteres são textuais, menos o circunflexo (`^`) quando ele for o primeiro caractere da lista: ou seja, se quisermos casar com um caractere circunflexo, ele não poderá ser o primeiro a figurar na lista:

```
:~$ grep -o '2[.^*$]75' <<< '2.75 2^75 2*75 2$75'
2.75
2^75
2*75
2$75
```

#### Casando colchetes

Os colchetes também podem entrar na lista, mas é preciso tomar cuidado com o `]`: se for o caso, ele tem que ser o primeiro caractere na lista:

```
:~$ grep -o '2[].,[]75' <<< '2.75 2,75 2[75 2]75'
2.75
2,75
2[75
2]75
```

#### Faixas de caracteres

Outra coisa que podemos fazer nas listas é expressar faixas de caracteres. Embora seja mais comum o uso de faixas de letras e números, é possível criar faixas com quase todos os tipos de caracteres iniciais e finais. A única limitação (e um bom motivo para não abusar das possibilidades) é que os caracteres válidos dependerão das coleções de caracteres da localidade do sistema:

```
:~$ grep -o '[9-a]' <<< '9 ; : < = > ? @ a'
9
a
:~$ LC_ALL=C grep -o '[9-a]' <<< '9 ; : < = > ? @ a'
9
;
:
<
=
>
?
@
a

```

Na tabela de caracteres da localidade `pt_BR` (padrão do meu sistema), os símbolos gráficos da *string* testada não estão entre `9` e `a`. Mas, alterando a localidade para `C`, a tabela ASCII é utilizada e todos os símbolos gráficos casam com o intervalo da lista.

Na verdade, até por uma questão de segurança, os intervalos mais utilizados são:

* `A-Z` - letras maiúsculas
* `a-z` - letras minúsculas
* `0-9` - dígitos

Também não precisamos utilizar sempre os extremos dos intervalos. Faixas como `b-f` ou `3-8`, por exemplo, são perfeitamente válidas.

#### Casando o traço

Como o traço (`-`) é utilizado para expressar faixas de caracteres, ele também exigirá um cuidado especial quando for apenas mais um caractere na lista: ele deve ser o primeiro ou o último!

```
 $ grep -o '[0-9-]' <<< '1 2 3 -'
1
2
3
-
```

> É mais comum vermos o traço sendo utilizado na última posição da lista, mas isso não é uma regra.

### Lista negada

A lista negada segue o mesmo conceito de uma lista normal, a diferença é que o casamento será feito com quaisquer outros caracteres que não estejam listados.

```
:~$ grep -o '[0-9]' <<< '1 2 3 a b c'
1
2
3
:~$ grep -o '[^0-9]' <<< '1 2 3 a b c'



a

b

c
```

> Observe que o padrão negado também casou com os espaços da string.

### Classes de caracteres

As listas são uma excelente forma de representar um conjunto restrito de caracteres, mas podem se tornar bastante longas e difíceis de ler em algumas situações. Para simplificar, nós temos algumas classes de caracteres, que são a predefinição de certas listas de uso frequente, por exemplo:

```
Caracteres válidos em palavras    \w     [A-Za-z0-9_]
Caracteres inválidos em palavras  \W     [^A-Za-z0-9_]
Espaço ou tabulação               \s     [ \t\n\r\f\v]
Tudo menos espaço ou tabulação    \S     [^ \t\n\r\f\v]
```

Dependendo da implementação das expressões regulares, nós ainda podemos encontrar outras classes, como: `\d`, para expressar dígitos decimais, `\x`, para representar dígitos hexadecimais, e `\O`, para os octais. Contudo, por não estarem universalmente disponíveis no shell GNU, seu uso deve ser evitado em favor de outro conjunto de classes conhecidas como **classes POSIX**.

> Na literatura estrangeira, as classes de caracteres simbolizadas por uma sequência de escape também são chamadas de "operadores".

### Classes POSIX

As normas POSIX especificam alguns recursos adicionais que podem ser utilizados em listas: os elementos de coleções, as equivalências e as classes nomeadas.

#### Elementos de coleção

Podem ser um caractere, uma sequência de caracteres que conte como um caractere único ou um nome predefinido para uma sequência de caracteres.

Quando os elementos de uma coleção aparecem entre `[.` e `.]`, tudo que estiver entre os pontos será tratado como uma sequência textual do padrão entre as opções da lista. Sendo assim, por exemplo, uma coleção `[.ch.]` (que é tratado como apenas um caractere no idioma tcheco, por exemplo) casaria com as ocorrências do dígrafo `ch` na string.

> Por ser altamente dependente de coleções localizadas, este recurso raramente é usado para descrever padrões no nosso idioma ou na localização padrão `C`, especialmente para avaliações feitas com utilitários GNU.

#### Classes de equivalência

Uma localização POSIX pode definir que alguns caracteres devem ser considerados idênticos para efeito de ordenação (pense em dicionários). Por exemplo, caracteres acentuados podem ser ignorados na ordenação de uma lista de palavras:

```
:~$ sort -d <<< $'árvore\nacento\nátomo\npeixe\nseta'
acento
peixe
árvore
seta
átomo
```

Perceba que, com a opção `-d` o utilitário `sort` ignorou o caractere `á` de `árvore` e `átomo` e procedeu a ordenação alfabética levando em conta o segundo caractere.

No contexto de uma expressão regular, portanto, o caractere `a` e suas versões acentuadas podem ser representados numa lista por uma classe de equivalência (ou "classe equivalente") deste modo: `[=a=]`

```
:~$  grep '[[=a=]]' <<< 'árvore acento átomo peixe seta'
[á]rvore [a]cento [á]tomo peixe set[a]
```

#### Classes nomeadas

A rigor, quando ouvimos alguém falar de "classes POSIX", é quase certo que estão se referindo às classes nomeadas das normas POSIX, que são predefinições de conjuntos de caracteres válidos para ocuparem a posição de um elemento nas listas:

```
[:upper:]  Letras maiúsculas            [A-Z]
[:lower:]  Letras minúsculas            [a-z]
[:alpha:]  Maiúsculas/minúsculas        [A-Za-z]
[:alnum:]  Letras e números             [A-Za-z0-9]
[:word:]   Letras, números e sublinhado [A-Za-z0-9_]
[:digit:]  Dígitos decimais             [0-9]
[:xdigit:] Dígitos em hexa              [0-9A-Fa-f]
[:blank:]  Espaço e tabulação           [ \t]
[:space:]  Caracteres em branco         [ \t\n\r\f\v]
[:graph:]  Caracteres imprimíveis       [^ \t\n\r\f\v]
[:print:]  Imprimíveis e o espaço       [^\t\n\r\f\v]
[:punct:]  Pontos e símbolos gráficos   (uma lista enorme!)
```
> Dependendo da localidade, pode haver outras classes nomeadas disponíveis.

## 1.4 - Quantificadores

Por padrão, cada entidade (um caractere textual, um representante ou um padrão agrupado) é tratada como uma ocorrência única e obrigatória no padrão. Para alterar isso, nós temos os metacaracteres quantificadores:

| Metacaractere | Descrição |
|---------------|-----------|
| `?`           | A entidade anterior pode ocorrer zero ou uma vez. |
| `*`           | A entidade anterior pode ocorrer zero ou mais vezes. |
| `+`           | A entidade anterior pode ocorrer uma ou mais vezes. |
| `{min,max}`   | A entidade anterior pode ocorrer em qualquer quantidade entre o mínimo e o máximo (inclusive). |
| `{0,max}`     | A entidade anterior pode ocorrer de zero até a quantidade máxima. |
| `{min,}`      | A entidade anterior pode ocorrer pelo menos na quantidade mínima. |
| `{qtde}`      | Especifica a quantidade exata de ocorrências da entidade anterior. |

## 1.5 - Âncoras

As âncoras não casam com caracteres, mas nos ajudam a especificar a posição da ocorrência de um padrão:

| Metacaractere | Descrição |
|---------------|-----------|
| `^`           | Marca o começo de uma linha. |
| `$`           | Marca o fim de uma linha. |
| `\b`          | Especifica que o padrão ocorre no início ou no fim de uma palavra. |
| `\B`          | Especifica que o padrão **não ocorre** no início ou no fim de uma palavra. |

> Quando falamos das classes, nós vimos que, no contexto das expressões regulares, "palavras" são sequências de caracteres formadas apenas por letras maiúsculas e minúsculas, dígitos e o caractere sublinhado: `[A-Za-z0-9_]`.

## 1.6 - Operador 'ou'

O operador "ou" (representado pelo caractere `|`) expressa padrões alternativos, ou seja, os padrões à esquerda e a direita do `|` são diferentes, mas ambos são válidos para efeito de um casamento:

```
:~$ grep -Eo 'bom dia|boa tarde' <<< $'bom dia\nboa tarde\nboa noite'
bom dia
boa tarde
```

> Observe o uso do modo estendido do `grep`!

O conceito é simples, mas algumas situações podem requerer um pouco mais de atenção. No exemplo acima, nós deixamos o `boa noite` de fora, mas poderíamos incluí-lo assim:

```
:~$ grep -Eo 'bom dia|boa tarde|boa noite' <<< $'bom dia\nboa tarde\nboa noite'
bom dia
boa tarde
boa noite
```

Como bons analistas de padrões, nós podemos tentar encurtar essa expressão:

```
:~$ grep -Eo 'bom dia|boa tarde|noite' <<< $'bom dia\nboa tarde\nboa noite'
bom dia
boa tarde
noite
```

Apesar do raciocínio estar correto, a aplicação do conceito não está: o operador `|` trabalha com padrões inteiros, não com partes de padrões. Logo, sem a opção `-o`, nós teríamos sido induzidos ao erro, porque o `grep` também teria retornado a linha `boa noite`, mas o padrão alternativo casado foi apenas `noite`.

É para resolver esse tipo de problema que nós temos os grupos:

```
:~$ grep -Eo 'bom dia|boa (tarde|noite)' <<< $'bom dia\nboa tarde\nboa noite'
bom dia
boa tarde
boa noite
```

## 1.7 - Grupos

Os parêntesis `(...)` são utilizados para agrupar subpadrões. Sendo assim, no exemplo anterior, o grupo com os padrões `tarde` e `noite` ainda são parte do padrão iniciado com `boa`. Consequentemente, o primeiro operador `|` da nossa expressão está lidando com os padrões alternativos `bom dia` e `boa (tarde|noite)`.

## 1.8 - Referências prévias (retrovisores)

Uma consequência do uso de agrupamentos, é que o mecanismo das expressões regulares faz um registro de todos os trechos casados com subpadrões na ordem em que são encontrados. Então, ainda no exemplo anterior, a *string* casada com o padrão `(tarde|noite)` ficará registrada e poderá ser referenciada na expressão regular pelo operador numerado `\1`:

```
$ grep -E 'bom dia|boa (tarde|noite) \1' <<< $'bom dia\nboa noite tarde\nboa noite noite'
bom dia
boa noite noite
```

## 1.9 - Casamentos 'gulosos' (greedy)

Por padrão, as implementações de expressões regulares POSIX e GNU nos modos básico (BRE) e estendido (ERE) sempre tentarão casar o padrão com a maior sequência de caracteres possível. Esse comportamento é chamado de casamento ganancioso (*greedy*) ou, como Aurélio Jargas e Julio Neves costumam dizer, "guloso". Normalmente, isso não afeta a maior parte dos usos das expressões regulares: localizar linhas de texto que casem com o padrão, por exemplo. Contudo, quando se trata de extrair e processar apenas a parte casada, isso se torna um problema.

Vamos observar novamente o exemplo do arquivo `/etc/passwd`. Desta vez, o nosso objetivo é extrair apenas os três primeiros campos da linha iniciada com a string `blau:`:

```
:~$ grep -Eo '^blau:(.*:){2}' /etc/passwd
blau:x:1000:1000:Blau Araujo,,,:/home/blau:
```

Como podemos ver, a parte casada da string incluiu todos os seis primeiros campos, o que aconteceu porque `.*:` sempre casará com a maior sequência de caracteres seguida de `:` na linha.

No `sed` seria a mesma coisa (com uma regex ligeiramente diferente):

```
:~$ sed -nE 's/(^blau:(.*:){2}).*/\1/p' /etc/passwd
blau:x:1000:1000:Blau Araujo,,,:/home/blau:
```

Isso significa que, nos modos ERE e BRE, se quisermos parte da string, ou tentamos ser mais verbosos e específicos nas nossas expressões regulares ou lançamos mão de outros recursos do shell para processar os casamentos encontrados.

Por exemplo:

```
:~$ sed -nE 's/^(blau:)((.*:){2})(.*:){3}.*/\1\2/p' /etc/passwd
blau:x:1000:
```

Ou ainda:

```
:~$ grep '^blau:' /etc/passwd | cut -d':' -f1-3
blau:x:1000
```

### Quantificadores dobrados

Em meados dos anos 1990, a versão 5 da linguagem Perl introduziu uma série de novos operadores de expressões regulares, entre eles, o modificador de quantificadores (ou "quantificadores dobrados"), que visa justamente implementar a possibilidade de realizar um casamento com a mínima sequência de caracteres que corresponda ao padrão. Então, nas especificações do Perl 5, todo quantificador seguido de `?` (que, em princípio, também é um quantificador, daí o "dobrado") indicará um casamento "não-guloso":

```
+?          Um ou mais não-guloso
*?          Zero ou mais não-guloso
{min,max}?  De mínimo a máximo não-guloso
??          Zero ou um não-guloso
```

Os quantificadores dobrados não estão implementados nos modos BRE e ERE, adotado por ferramentas GNU como o `awk`, o `grep` e o `sed`, mas o `grep` oferece um modo parcialmente implementado das especificações Perl (PCRE) com a opção `-P`.

Portanto, isso é possível:

```
:~$ grep -Po '^blau:(.*?:){2}' /etc/passwd
blau:x:1000:
```

## 1.10 - Operadores estendidos do Perl

Ainda entre as novidades do Perl 5, agora nós temos operadores capazes de dotar as expressões regulares de recursos que, originalmente, ficariam para os comandos e utilitários do shell encarregados pelo pós-processamento dos resultados obtidos.

### Estrutura

De modo geral, os operadores estendidos se parecem muito com agrupamentos (e, de fato, eles agrupam) iniciados com `?`:

```
(?<IDENTIFICADOR><CONTEÚDO>)
```

Onde o `IDENTIFICADOR` determina o tipo de operador e `CONTEÚDO` é o trecho da expressão regular que será afetado.

### Alguns operadores estendidos

Tendo em vista que estamos falando de expressões regulares no **shell GNU**, e que nem todos os recursos PCRE estão implementados, nós vamos nos limitar a alguns operadores estendidos mais comuns que podem ser úteis sem ferir muito a *filosofia Unix*.

| Operador      | Descrição |
|---------------|-----------|
| `(?#TEXTO)`   | Comentário |
| `(?:PADRÃO)`  | Grupo não-capturado |
| `(?=PADRÃO)`  | Asserção positiva à frente de comprimento zero |
| `(?!PADRÃO)`  | Asserção negativa à frente de comprimento zero |
| `(?<=PADRÃO)` | Asserção positiva anterior de comprimento zero |
| `(?<!PADRÃO)` | Asserção negativa anterior de comprimento zero |
| `\K`          | Exclui os padrões casados anteriormente |

### Comentários '(?#TEXTO)'

O agrupamento é ignorado na sua função de padrão e, portanto, não participa do casamento:

```
:~$ grep -P '(?# Isto é um comentário)(leite|muito) quente' <<< $'muito quente\nleite quente'
[muito quente]
[leite quente]
```

> Os colchetes indicam os padrões casados na saída do `grep`.

### Grupo não capturado '(?:PADRÃO)'

O subpadrão participa do casamento, mas não recebe um registro numerado (`\1`, `\2`, etc...):

```
:~$ grep -P '(quero)-\1' <<< 'quero-quero'
quero-quero

:~$ grep -P '(?:quero)-\1' <<< 'quero-quero'
grep: reference to non-existent subpattern
```

### Asserção positiva à frente '(?=PADRÃO)'

Uma **asserção positiva** é um padrão agrupado que deve encontrar, obrigatoriamente, um casamento na *string*, mas não é tratada como parte do resultado -- daí ser definida como **de comprimento zero**. No caso do operador de asserção positiva ***à frente*** (*lookahead*), o mecanismo de avaliação da expressão regular verifica se o padrão agrupado adiante possui correspondência para considerar se haverá ou não um casamento com a *string*:

```
:~$ grep -P 'salada(?= mista)' <<< $'salada mista\nsalada de frutas'
[salada] mista
```

> Os colchetes indicam os padrões casados na saída do `grep`.

Isso é útil em situações onde queremos obter apenas uma parte no começo da linha encontrada pelo `grep` sem a necessidade de encadear outras ferramentas numa *pipeline*. No `sed`, isso não faz muito sentido, já que podemos fazer buscas e substituições de textos.

**Importante:** os operadores de comprimento zero não registram subpadrões:

```
:~$ grep -P 'quero-(?=quero)' <<< 'quero-quero'
[quero-]quero

:~$ grep -P 'quero-(?=quero) \1' <<< 'quero-quero'
grep: reference to non-existent subpattern
```

### Asserção negativa à frente '(?!PADRÃO)'

Ao contrário da asserção positiva, uma asserção negativa determina um padrão que não pode ser casado:

```
:~$ grep -P 'salada(?! mista)' <<< $'salada mista\nsalada de frutas'
[salada] de frutas
```

### Asserção positiva anterior '(?<=PADRÃO)'

Funciona como a asserção positiva à frente, mas condicionando os padrões seguintes ao casamento do padrão agrupado anteriormente (*lookbehind*):

```
:~$ grep -P '(?<=leite )quente' <<< $'muito quente\nleite quente'
leite [quente]
```

### Asserção negativa anterior '(?<!PADRÃO)'

Condiciona os padrões seguintes ao **não casamento** do padrão agrupado anteriormente:

```
:~$ grep -P '(?<!leite )quente' <<< $'muito quente\nleite quente'
muito [quente]
```

### Operador '\K'

Introduzido na PCRE7.2, o operador `\K` também estabelece uma asserção positiva anterior (*lookbehind*), mas não está limitada a um padrão agrupado: ou seja, todos os padrões que vierem antes dele participarão do mecanismo de casamento, mas serão descartados no resultado da avaliação da expressão regular:

```
:~$ grep -P '^blau.*:/bin/bash' /etc/passwd
[blau:x:1000:1000:Blau Araujo,,,:/home/blau:/bin/bash]

~ $ grep -P '^blau.*:\K/bin/bash' /etc/passwd
blau:x:1000:1000:Blau Araujo,,,:/home/blau:[/bin/bash]
```

> Os colchetes indicam os padrões casados na saída do `grep`.
