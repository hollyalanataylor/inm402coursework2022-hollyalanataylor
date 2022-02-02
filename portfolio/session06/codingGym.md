---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/teaching.yml
---

@import "../../lectures/css/datavis.less"

<!-- Everything above this line should probably be left untouched. -->

# Coding Gym 6: Maybe, Point-Free Style and Functional Composition

_These exercises are designed to help you build confidence programming with Elm. They assume no prior experience of coding in any language other than that arising from previous weeks of this module. This document is designed to be viewed in VSCode with litvis so you can add your own code blocks to answer each question._

## 6.1 Maybe

While most functions are able to return a meaningful value for any given input parameters (e.g. [List.length](https://package.elm-lang.org/packages/elm/core/latest/List#length), [String.fromInt](https://package.elm-lang.org/packages/elm/core/latest/String#fromInt)), some may only be able to do so for certain values of input. For example, [List.maximum](https://package.elm-lang.org/packages/elm/core/latest/List#maximum) finds the maximum value in a list of comparable values, but what would be the maximum of an empty list? Functions like this that may or may not be able to evaluate an input, can return a special type of value called [Maybe](https://package.elm-lang.org/packages/elm/core/latest/Maybe).

Something of type `Maybe` can either be a special value `Nothing` in cases where it has no meaningful value, or `Just` a value when it does represent a meaningful value. For example:

```elm {l r}
test1 : Maybe Int
test1 =
    List.maximum [ 1, 5, 2, 6, 3 ]
```

```elm {l r}
test2 : Maybe Int
test2 =
    List.maximum []
```

We can extract a value from a `Maybe` with the function [Maybe.withDefault](https://package.elm-lang.org/packages/elm/core/latest/Maybe#withDefault) that allows us to provide a default value to return in the case that a function evaluates to `Nothing` and a simple value in cases where it evaluates to `Just` something. For example:

```elm {l r}
test3 : Int
test3 =
    List.maximum [ 1, 5, 2, 6, 3 ] |> Maybe.withDefault -999
```

```elm {l r}
test4 : Int
test4 =
    List.maximum [] |> Maybe.withDefault -999
```

{(task|}

- Create and test a function that finds the alphabetically first name in a list of strings or reports "No names provided" if the list is empty.

- Create and test a function that reports the range (difference between the maximum and minimum) of a list of numbers or 0 if the list is empty.

- Create a function that converts a String to a number, or returns a 0 if the conversion cannot be made (_Hint: Have a look at 'Float Conversions' in the [String documentation](https://package.elm-lang.org/packages/elm/core/latest/String)_).

- Create a function that converts a list of Strings into a list of numbers, removing any values that cannot be converted. For example, it should convert `["1","2","Tree","4"]` into `[1,2,4]`. (_Hint: Have a look at [List.filterMap](https://package.elm-lang.org/packages/elm/core/latest/List#filterMap) which can apply a mapping to a list and filter out `Nothings`_).

- Create a function that removes the first item from a List. If the list has one or zero items in it, the function should return an empty list. While it would be possible to use [List.drop](https://package.elm-lang.org/packages/elm/core/latest/List#drop) to do this directly, see if you can do so using [List.tail](https://package.elm-lang.org/packages/elm/core/latest/List#tail) instead.

{|task)}

## 6.2 Point-Free Style and Functional Composition

Suppose we wished to create a function that given a number will return the cube of that number plus six. And for the purposes of this illustration, suppose we only had access to these two functions:

```elm {l}
cube : Float -> Float
cube n =
    n * n * n


plusSix : Float -> Float
plusSix n =
    n + 6
```

We could use the pipe operator to call both functions in order to evaluate an answer:

```elm {l}
combined : Float -> Float
combined n =
    n |> cube |> plusSix
```

```elm {l raw siding}
answer : Float
answer =
    combined 10
```

Alternatively we can use _functional composition_ to create a single new function that combines the two others:

```elm {l}
combined2 : Float -> Float
combined2 =
    cube >> plusSix
```

```elm {l raw}
answer2 : Float
answer2 =
    combined2 10
```

This alternative form of function definition is an example of _point-free style_ where we don't specify the parameter (storing the number to be cubed and have 6 added) but instead define another function that will have that parameter.

The general rule for the functional composition operator `>>` is

`g(f(x))` is equivalent to `(f >> g) (x)`

We can also reverse the order of composition with the `<<` operator:

`g(f(x))` is equivalent to `(g << f) (x)`

The left-pointing composition operator often makes more intuitive sense to understand.
For example, we could create a test for evenness by combing an `odd` function with Elm's [not](http://package.elm-lang.org/packages/elm-lang/core/5.1.1/Basics#not) function to negate it:

```elm {l}
odd : Int -> Bool
odd n =
    modBy 2 n == 1


even : Int -> Bool
even =
    not << odd
```

```elm {l raw}
answer : Bool
answer =
    even 240
```

To summarise, `not << odd` is simply a more compact way of saying `not <| odd n`, which is also the equivalent of saying `n |> odd |> not`.

We can also treat 'infix' operators like `+`, `*`, and `==` as functions using point-free style by placing them inside brackets. So, for example, the following three functions are all equivalent:

```elm {l siding}
double : Float -> Float
double n =
    2 * n
```

```elm {l siding}
double : Float -> Float
double n =
    (*) 2 n
```

```elm {l siding}
double : Float -> Float
double =
    (*) 2
```

{(task|}

- Re-express the following functions using point-free style.

```elm {l}
q1 : Int -> Int
q1 n =
    100 + n


q2 : String -> String
q2 name =
    String.toLower name


q3 : Int -> Int
q3 a =
    1 - (3 ^ a)


q4 : List String -> List String
q4 names =
    names
        |> List.map (\name -> String.toUpper name)
        |> List.map (\name -> String.replace "CAT" "DOG" name)


q5 : List Int -> List Int -> List Int
q5 list1 list2 =
    List.map2 (\n1 n2 -> n1 * n2) list1 list2
```

{|task)}

The following can be used to test that your new functions generate the same results as the originals:

^^^elm {r=(q1 3)}^^^

^^^elm {r=(q2 "The cat sat on Matt")}^^^

^^^elm {r=(q3 8)}^^^

^^^elm {r=(q4 ["A catalogue of catastrophes","Catch 22", "A Catcher in the Rye"])}^^^

^^^elm {r=(q5 [3,2,1][10,20,30])}^^^
