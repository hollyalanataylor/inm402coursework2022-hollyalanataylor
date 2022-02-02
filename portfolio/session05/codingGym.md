---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/teaching.yml
---

@import "../../lectures/css/datavis.less"

<!-- Everything above this line should probably be left untouched. -->

# Coding Gym 5 : Boolean Operators and Filtering

_These exercises are designed to help you build confidence programming with Elm. They assume no prior experience of coding in any language other than that arising from previous weeks of this module. This document is designed to be viewed in VSCode with litvis so you can add your own code blocks to answer each question._

## 5.1. Boolean Operators

Most of the operators and expressions you have created so far in Elm have been to derive either new numbers or new strings from existing numbers and strings (e.g. `3 + 4 * 5` or `"hot" ++ " potatoes"`). We can also create expressions that evaluate to either a Boolean `True` or `False`. For example:

```elm {l}
isDatavis : String -> Bool
isDatavis moduleCode =
    moduleCode == "INM402" || moduleCode == "IN3030"
```

which we can test with shorthand 'triple hat' notation:

^^^elm r=(isDatavis "IN1007")^^^

^^^elm r=(isDatavis "IN3030")^^^

Valid Boolean operators include:

| Operator | Description                           | Example                                           |
| -------- | ------------------------------------- | ------------------------------------------------- |
| `==`     | 'Equal to' test                       | `moduleCode == "IN3030"`                          |
| `/=`     | 'Not equal to' test                   | `moduleCode /= "IN3030"`                          |
| `<`      | 'Less than' test                      | `age < 18`                                        |
| `>`      | 'Greater than' test                   | `record > 0`                                      |
| `<=`     | 'Less than or equal to' test          | `population <= 50000`                             |
| `>=`     | 'Greater than or equal to' test       | `temperature >= 30`                               |
| `&&`     | 'AND' operator                        | `temperature > -5 && temperature <= 30`           |
| `‖`      | 'OR' operator                         | `moduleCode == "INM402" ‖ moduleCode == "IN3030"` |
| `not`    | Negate (reverse) a Boolean expression | `not (favourite == "cat")`                        |

{(task|}

Using Boolean operators, write functions to do the following. You should also add test functions to check they behave as you expect them to (you can use triple hat notation as above or write your own functions in code blocks).

- Determine if a number provided as a parameter is positive or not
- Determine if a string provided as a parameter matches the word `rosebud`
- Determine if a number provided as a parameter is odd (hint, have a look at [modBy](https://package.elm-lang.org/packages/elm/core/latest/Basics#modBy) to help.)
- Determine if a list of numbers provided as a parameter has at least 3 entries
- Determine if a number is not divisible by 5 and even.

{|task)}

## 5.2. Filtering Lists

Functions that take a value as input and return either `True` or `False` like the ones you have created in the previous task are called _predicates_. They can be usefully applied to a list of items, _filtering_ the values where the predicate is `True`. For example:

```elm {l r}
smallNumbers : List Int
smallNumbers =
    List.filter (\n -> n > 0 && n < 10) (List.range -100 100)
```

{(task|}

Filter the following lists using the predicates you created in the previous task:

- Filter only the positive numbers in `List.range -100 20`
- Filter only words that match `rosebud` in the following list `["tiddles","abc123","systemDefault","rosebud","7August1993"]`
- Filter only even numbers in `List.range 1 40`
- Filter only lists of at least 3 entries from the following list of lists:

  ```elm
  [[1,2], [10,11,21,24], [], [0,0,0,0,0,0], [-1,-2,-3]]
  ```

- Provide a list of all the even numbers between 1 and 40 that are not divisible by 5

{|task)}
