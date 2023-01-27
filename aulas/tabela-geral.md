# Tabela geral de expressões regulares

| Metacaractere | Tipo          | Descrição                                                                                       |
|---------------|---------------|-------------------------------------------------------------------------------------------------|
| `.`           | Representante | Qualquer caractere único.                                                                       |
| `[...]`       | Representante | Um caractere na lista.                                                                          |
| `[^...]`      | Representante | Qualquer caractere fora da lista.                                                               |
| `\w`          | Classe        | Um caractere válido em palavras: `[A-Za-z0-9_]`                                                 |
| `\W`          | Classe        | Um caractere inválido em palavras: `[^A-Za-z0-9_]`                                              |
| `\s`          | Classe        | Um caractere em branco.                                                                         |
| `\S`          | Classe        | Tudo menos caracteres em branco.                                                                |
| `?`           | Quantificador | A entidade anterior (caractere literal, padrão ou representante) pode ocorrer zero ou uma vez.  |
| `*`           | Quantificador | A entidade anterior pode ocorrer zero ou mais vezes.                                            |
| `+`           | Quantificador | A entidade anterior pode ocorrer uma ou mais vezes.                                             |
| `{min,max}`   | Quantificador | A entidade anterior pode ocorrer em qualquer quantidade entre `mín` e  `max` (inclusive).       |
| `^`           | Âncora        | Marca o começo de uma linha.                                                                    |
| `$`           | Âncora        | Marca o fim de uma linha.                                                                       |
| `\b`          | Âncora        | Especifica que o padrão ocorre no início ou no fim de uma palavra.                              |
| `\B`          | Âncora        | Especifica que o padrão **não ocorre** no início ou no fim de uma palavra.                      |
| `\`           | Citação       | Remove o significado especial de metacaracteres (ou atribui significado especial a caracteres). |
| `\|`           | Operador      | Expressa padrões alternativos (um ou outro).                                                    |
| `(...)`       | Grupo         | Agrupa, aninha e define a precedência e a associação de padrões.                                |
| `\1...\9`     | Registradores | Reutiliza padrões casados em grupos anteriores conforme a ordem das ocorrências (máximo de 9).  |


