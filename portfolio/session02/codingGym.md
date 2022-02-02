---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/teaching.yml
---

@import "../../lectures/css/datavis.less"

<!-- Everything above this line should probably be left untouched. -->

# Coding Gym 2 : Lists

_These exercises are designed to help you build confidence programming with Elm. They assume no prior experience of coding in any language other than that arising from previous weeks of this module. This document is designed to be viewed in VSCode with litvis so you can add your own code blocks to answer each question._

## 2.1. Lists of items

In the previous gym you created functions that generated single values such as an integer, a floating point number or a String of text. We move on now to consider [lists of items](https://package.elm-lang.org/packages/elm/core/latest/List). Each item can again be a number, some text etc. but now we can create an ordered collection of them.

Lists are indicated with square brackets and items separated with commas. Lists can also be empty. For example:

```elm {l}
people : List String
people =
    [ "Ada Lovelace", "Alan Turing", "Jacques Bertin" ]


birthYears : List Int
birthYears =
    [ 1815, 1912, 1918 ]


todoList : List String
todoList =
    []
```

{(task|}

Create separate functions, each in their own code block, that display the following. Remember to include `elm {l raw}` in each code block header so you can see the results. If created successfully, you should see the evaluation of each function appear in the preview window.

- the names of your four cities you have visited.
- the integers 70 to 79 inclusive.
- the [triangular numbers](https://en.wikipedia.org/wiki/Triangular_number) with one or two digits.
- The first 8 numbers in the sequence 1/1, 1/2, 1/3, 1/4 etc.
- The colours of the rainbow.
- A list of mammals that have twelve legs.

{|task)}

## 2.2. In-built functions to generate lists

Elm has a number of functions that make it easier to create lists. In particular [List.range](https://package.elm-lang.org/packages/elm/core/latest/List#range) for creating a range of integers; [List.repeat](https://package.elm-lang.org/packages/elm/core/latest/List#repeat) for creating a list of repeated items, the [cons operator `::`](<https://package.elm-lang.org/packages/elm/core/latest/List#(::)>) for adding an item to a list and the [++](https://package.elm-lang.org/packages/elm/core/latest/Basics#++) operator for combining two lists. For example:

```elm {l raw}
firstTen : List Int
firstTen =
    List.range 1 10


fiveGoldRings : List String
fiveGoldRings =
    List.repeat 5 "gold ring"


newKidOnTheBlock : List String
newKidOnTheBlock =
    "New kid" :: [ "old kid", "another old kid" ]


oddsAndEvens : List Int
oddsAndEvens =
    [ 1, 3, 5, 7, 9 ] ++ [ 2, 4, 6, 8 ]
```

{(task|}

Choosing from the list-generating functions above, create separate functions to do the following:

- List the whole numbers (integers) 2001 to 2022 inclusive
- List the integers between -149 and -101 inclusive
- Add your own name added to this group of travellers: `[ "Ripley", "Vasquez", "Bishop", "Burke" ]`
- The word "dozen" repeated a dozen times.
- Create a single list made up by combining a list of three animals, three vegetables and three minerals.

{|task)}

## 2.3. Summarising Lists

Elm has some in-built functions that can summarise the contents of a list including [List.length](https://package.elm-lang.org/packages/elm/core/latest/List#length), [List.sum](https://package.elm-lang.org/packages/elm/core/latest/List#sum) and [List.product](https://package.elm-lang.org/packages/elm/core/latest/List#product). For example:

```elm {l raw}
firstTenTotal : Int
firstTenTotal =
    List.sum (List.range 1 10)


numberOfAnimals : Int
numberOfAnimals =
    List.length [ "cat", "dog", "giraffe" ]


power : Int
power =
    List.product [ 2, 3, 4 ]
```

{(task|}

Using the list-summarising functions above, create separate functions to calculate the following:

- The total of the integers from -149 to -101
- The number of items in the combined list `[] ++ [ "Ripley", "Vasquez", "Bishop", "Burke" ]`
- The mean (average) of all integers from -149 to -101
- 8 factorial (equivalent to `8 * 7 * 6 * 5 * 4 * 3 * 2 * 1`)

{|task)}

## 2.4 Tuples

Sometimes we might wish to operate on a very small collection of items rather than a list of unspecified length. For this we can use a [tuple](https://package.elm-lang.org/packages/elm/core/latest/Tuple) which can contain two or three items only. An unlike a list, the items do not need to be of the same type.

Tuples are indicated with round rather than square brackets. For example:

```elm {l r}
ada : ( String, Int )
ada =
    ( "Ada Lovelace", 1815 )
```

{(task|}

Create a functions that return tuples that represent:

- Your name and commuting time.
- A point in 3d space (requiring 3 coordinates)
- A list of 2d coordinates.

{|task)}
