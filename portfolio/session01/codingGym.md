---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/teaching.yml
---

@import "../../lectures/css/datavis.less"

<!-- Everything above this line should probably be left untouched. -->

# Coding Gym 1 : Functions

## Induction

These exercises are designed to help you build confidence programming with Elm. They assume no prior experience of coding in any language and comprise simple tasks that build your familiarity with the process of coding in litvis. In many cases they involve repetitions of limited coding tasks, but just like repeats in a gym, the more you do, the easier it gets.

We will not focus on visualization (for that, see the main lecture notes), but instead concentrate on basic principles of coding in a functional language.

This document and all the other Coding Gyms provide both the exercise instructions, and space for you to complete them by adding your own code.

Make sure you are viewing this page in _VSCode_ and have the litvis preview window open (`Ctrl/Cmd-K` followed by `V`). This should show a nicely formatted version of the page on the right hand side.

{(annotation|}

If you want to add your own notes to this page you can do so conveniently by placing your extra content inside these **annotation labels**, noting carefully the use of the different bracket markers `{`, `(`, `|`, `)` and `}`.

{|annotation)}

## 1.1. Simple functions

Our first coding exercises will create some functions to return values of some kind. You can give a function any name you like as long as it starts with a lowercase letter and has no spaces. The first line of a function definition is its _type annotation_ and reports the type of value it will generate. In the examples below, `myName` generates a `String` (some text), `myYearOfBirth` an integer (whole number) and `myFavouriteNumber` a floating point number (a number with a decimal point).

Below the type annotation you name the function again followed by `=` and then on a new line indented should be the code that does the work of generating a value to return.

```elm {l r}
myName : String
myName =
    "Ada Lovelace"


myYearOfBirth : Int
myYearOfBirth =
    1815


myFavouriteNumber : Float
myFavouriteNumber =
    2.71828
```

You can create your own function in a _code block_ in your document by using a pair of triple backticks as follows:

````markdown
```elm {l raw}

```
````

The `l` in the code block header means display the literate code in the preview window. `raw` (or just `r` if you prefer) means that the value of any functions you create in the block will be displayed in the preview. For most of these coding gym exercises you will need to include at least `raw` output so you can see if your function has returned the value you expect.

{(task|}

Create separate functions, that display the following, each in their own `raw` code block. If created successfully, you should see the result of each function appear in the preview window. To start you off, I've created a few code blocks for the first few questions below the task window, but you will need to start adding your own after completing the first three functions.

- Your own first name.
- Your own last name.
- The (approximate) number of minutes it takes for you to commute to university.
- The [golden ratio](https://en.wikipedia.org/wiki/Golden_ratio) $\phi$ to 5 decimal places.
- A quote of your choice about data visualization.
- The year of the London Cholera epidemic that led to John Snow mapping clusters of cases around the Broad St pump.

{|task)}

```elm {l raw}
-- Place your first name function here inside the code block.
```

```elm {l raw}
-- Place your last name function here inside the code block.
```

```elm {l raw}
-- Place your commuting minutes function here inside the code block.
```

## 1.2. Simple functions with operators

As well as assigning numbers or strings directly to a function, you can also get the code to perform calculations with _operators_. For example, to add two numbers together you might have a function:

```elm {l r}
daysWorking : Int
daysWorking =
    5 + 6
```

Other operators you might use include `-` (subtraction), `*` (multiplication), `//` (integer division), `/` (floating point division), `^` (exponentiation, such as `4^2` for four-squared) and `++` for concatenating (joining) strings. These operators can be applied directly to numbers (or strings) or to functions that generate numbers (or strings). If you need to control the order in which operators apply, you can use brackets. For example `2 * (3 + 4)` is 14, not 10.

As with other functions built into the Elm language, you can refer to the [Elm documentation](https://package.elm-lang.org/packages/elm/core/latest/Basics) for examples and details of how to use them.

{(task|}

Create functions to calculate the following and show their output in litvis with `raw` output.

- The product of the integers 1 to 6 (i.e. 1 x 2 x 3 x 4 x 5 x 6).
- The number of seconds in 2022.
- Half of one plus the square root of five (${1+\sqrt 5}\over{2}$). To calculate a square root, see the [elm documentation](https://package.elm-lang.org/packages/elm/core/latest/Basics#sqrt).

- Your _James Bond name_ (i.e. in the form "Bond, James Bond").
- The mean (average) of the even numbers from 2 to 10.

{|task)}
