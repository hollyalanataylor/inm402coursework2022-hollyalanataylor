---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/teaching.yml
---

@import "../../lectures/css/datavis.less"

<!-- Everything above this line should probably be left untouched. -->

# Coding Gym 3 : Parameterising and Transforming Lists

_These exercises are designed to help you build confidence programming with Elm. They assume no prior experience of coding in any language other than that arising from previous weeks of this module. This document is designed to be viewed in VSCode with litvis so you can add your own code blocks to answer each question._

## 3.1. Functions with Parameters

The functions you have created in the previous gyms return the same value every time they are called. You can customise what they return by providing input _parameters_ that are used to generate the output. Parameters are indicated in a type annotation, naming their type and separating each of them and the output with a `->` symbol. For example:

```elm {l}
add : Float -> Float -> Float
add firstNum secondNum =
    firstNum + secondNum


greeting : String -> String
greeting name =
    "Good evening " ++ name
```

Parameterised functions can be _called_ by providing values for each of the parameters after the function name. For example:

```elm {l raw}
summary : Float
summary =
    add 120 84
```

```elm {l raw}
welcomeMessage : String
welcomeMessage =
    greeting "Ada Lovelace"
```

{(task|}

Create separate parameterised functions, each in their own code block, that do the following. Each code block should have the header `elm {l}`. To see the result, you should create separate code blocks with the header `elm {l raw}` and than call the functions you have created providing values for the parameters. See the examples `summary` and `welcomeMessage` above if you are unsure. If created successfully, you should see the evaluation of these functions appear in the preview window.

- A function that given two words as `String` parameters returns a single string with the words separated by an 'and'.
- A function that given a single `String` parameter representing a colour, returns "My favourite colour is ", naming the colour provided.
- A function to add three numbers provided as parameters.
- A function that creates a list of integers from 1 to a number provided as a parameter.
- A function that given two `Float` parameters, provides the square root of the sum of their squares.

{|task)}

## 3.2. Transforming Lists

Just as we applied operators to individual items, we can apply them to all the items in a list. This is commonly achieved with the in-built function [List.map](https://package.elm-lang.org/packages/elm/core/latest/List#map). This function takes two parameters: the first is the function that will be applied to each item in the list and the second is the list itself. For example to increment all the values in a list by 1 we could do the following:

```elm {l}
incByOne : List Int -> List Int
incByOne myList =
    let
        inc num =
            num + 1
    in
    List.map inc myList
```

```elm {l raw}
newList : List Int
newList =
    incByOne [ 10, 20, 30, 40 ]
```

{(task|}

Create separate parameterised functions, each in their own code block, that do the following. Each code block should have the header `elm {l}`. Test them by creating separate functions that supply the list as a parameter (like `newList` above)

- A function that given a list of words, adds the word "potato" to each of them.
- A function that adds 13 to all the items in a list of numbers.
- A function that trebles each item in a list of numbers
- A function that finds the sum of the squares of all numbers in a list (_Hint: You may find the function [List.sum](https://package.elm-lang.org/packages/elm/core/latest/List#sum) helpful here_).
- A function that given an integer, creates a list of multiples of 3 (3, 6, 9 etc.) from 3 to that integer.

{|task)}
