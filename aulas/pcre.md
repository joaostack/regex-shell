# Expressões regulares compatíveis com o Perl (PCRE)

## Quantificadores dobrados

| Modificador  | Descrição                        |
|--------------+----------------------------------|
| `??`         | Zero ou um "não-guloso".         |
| `*?`         | Zero ou mais "não-guloso".       |
| `+?`         | Um ou mais "não-guloso".         |
| `{min,max}?` | De mínimo a máximo "não-guloso". |


## Operadores estendidos do Perl

| Operador      | Descrição                                      |
|---------------+------------------------------------------------|
| `(?#TEXTO)`   | Comentário                                     |
| `(?:PADRÃO)`  | Grupo não capturado                            |
| `(?=PADRÃO)`  | Asserção positiva à frente de comprimento zero |
| `(?!PADRÃO)`  | Asserção negativa à frente de comprimento zero |
| `(?<=PADRÃO)` | Asserção positiva anterior de comprimento zero |
| `(?<!PADRÃO)` | Asserção negativa anterior de comprimento zero |
| `\K`          | Exclui os padrões casados anteriormente        |


### Comentários '(?#TEXTO)'

O agrupamento é ignorado e não participa do casamento:

```
:~$ str='muito quente leite quente'
:~$ grep -Po '(?# Eu sou um comentário)(leite|muito) quente' <<< $str
muito quente
leite quente
```

### Grupo não capturado '(?:PADRÃO)'

O subpadrão participa do casamento, mas não recebe um registro (`\1`, `\2`, etc...).

Grupo registrado normalmente:

```
:~$ grep -P '(quero)-\1' <<< 'quero-quero'
quero-quero
```

Grupo não registrado:

```
:~$ grep -P '(?:quero)-\1' <<< 'quero-quero'
grep: reference to non-existent subpattern
```

### Asserção positiva à frente '(?=PADRÃO)'

Uma *asserção positiva* é um padrão agrupado que deve encontrar, obrigatoriamente, um casamento na string, mas não é tratada como parte do resultado: daí ser definida como *de comprimento zero*. No caso do operador de asserção positiva *à frente* (*lookahead*, "olhe adiante"), o mecanismo de avaliação da expressão regular verifica se o padrão agrupado adiante possui correspondência para considerar se haverá ou não um casamento com a string:

```
:~$ str=$'salada mista\nsalada de frutas'
:~$ grep -P 'salada(?= mista)' <<< $str
[salada] mista
:~$ grep -Po 'salada(?= mista)' <<< "$str"
salada
```

**Importante:** os operadores de comprimento zero não registram grupos!

```
:~$ grep -P 'quero-(?=quero)' <<< 'quero-quero'
[quero-]quero
:~$ grep -P 'quero-(?=quero) \1' <<< 'quero-quero'
grep: reference to non-existent subpattern
```

### Asserção negativa à frente '(?!PADRÃO)'

Ao contrário da asserção positiva, uma asserção negativa determina um padrão que *não pode ser casado*:

```
:~$ str=$'salada mista\nsalada de frutas'
:~$ grep -P 'salada(?! mista)' <<< "$str"
[salada] de frutas
```

### Asserção positiva anterior '(?<=PADRÃO)'

Funciona como a asserção positiva à frente, mas condicionando os padrões seguintes ao casamento do padrão agrupado anteriormente (*lookbehind*, "olhe para trás"):

```
:~$ str=$'muito quente\nleite quente'
:~$ grep -P '(?<=leite )quente' <<< "$str"
leite [quente]
```

### Asserção negativa anterior '(?<!PADRÃO)'

Condiciona os padrões seguintes ao *não casamento* do padrão agrupado anteriormente:

```
:~$ str=$'muito quente\nleite quente'
:~$ grep -P '(?<!leite )quente' <<< "$str"
muito [quente]
```

### O operador '\K'

Introduzido na PCRE7.2, o operador `\K` também estabelece uma asserção positiva anterior (*lookbehind*), mas não está limitada a um padrão agrupado: ou seja, todos os padrões que vierem antes dele participarão do casamento, mas serão descartados no resultado da avaliação da expressão regular.

Por exemplo, se quiséssemos apenas o último campo da linha de `/etc/passwd` iniciada por `blau`:

```
~ $ grep -Po '^blau.*:\K/bin/bash' /etc/passwd
/bin/bash
```

