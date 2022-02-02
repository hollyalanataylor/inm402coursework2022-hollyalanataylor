---
id: litvis

narrative-schemas:
  - ../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

@import "../css/datavis.less"

```elm {l=hidden}
import VegaLite exposing (..)
```

<!-- Everything above this line should probably be left untouched. -->

# Session 1: Literate Visualization

## Table of Contents

1.  [What is Data Visualization?](#1-what-is-data-visualization?)
2.  [Literate Visualization and Litvis](#2-literate-visualization-and-litvis)
3.  [Data Visualization Models](#3-data-visualization-models)
4.  [Conclusions](#4-conclusions)
5.  [Recommended Reading](#5-recommended-reading)
6.  [Practical Exercises](#6-practical-exercises)

{(aim|}

This session is designed to introduce the process of designing and building data visualizations following principles of _literate visualization_. It introduces the notebook environment _litvis_, which allows you to specify visualization designs and to explain your design choices.

By the end of this session you should be able to:

- Critique a data visualization based on 5 simple criteria.
- Install the _VSCode_ editor with the _litvis_ plug-in on your computer.
- Write simple markdown text in a litvis document.
- Specify a simple visualization using elm-vegalite.
- _Push_ changes to a litvis document to a remote repository on GitHub.
- Begin to relate a visualization design and specification to Munzner's 'what-why-how' framework.
- Begin to relate a visualization specification to Wilkinson's 7 _Grammar of Graphics_ steps.

{|aim)}

---

## 1. What is Data Visualization?

Data visualization is a comparatively recent discipline (although [not that recent](https://youtu.be/jTuYxWB3lOc)) to emerge from the need to analyse and communicate patterns in complex datasets. While there is a long history of using pictures and graphics to represent data, we can consider modern data visualization as often having a number of identifiable characteristics:

1.  Represents a complex dataset graphically that could not as effectively be represented via other means.
2.  Often emphasises _connections_ and _comparisons_ between items of data.
3.  Frequently involves _interaction_ to allow someone to influence what they see.
4.  A visual _narrative_ that supports the construction of knowledge.
5.  Generates an _aesthetic response_ in order to encourage people to engage with the data or question.

What unites these characteristics is that they acknowledge the "**human in the loop**" of the visualization process. That is, while we use computers to help generate visualizations, it is, at its heart, a human process relying on human cognition and response to support human activity. This is important to recognise so that we don't put effort into designing and implementing visualization for tasks that are more readily achieved by computation alone.

One of the skills you will develop over the course of this module is the _design critique_. That doesn't mean being negative about a visualization design, but rather providing a justified set of judgements about what does and does not work effectively in a visualization. Using the five criteria above is just one of many ways in which you might structure such a critique.

{(task|}

Consider the data visualizations [shortlisted for the Information is Beautiful awards](https://www.informationisbeautifulawards.com/showcase?action=index&award=2019&controller=showcase&page=1&pcategory=short-list&type=awards). Using the 5 criteria listed above, select what you think represents a good example of data visualization from the shortlist. Jot down a brief justification of your choice for each of the 5 criteria. Are there any examples in that list that clearly fail as data visualizations?

If you are viewing these notes in [litvis](#2-literate-visualization-and-litvis) to can write your answers below between the labels `{(annotation|}` and `{|annotation)}`.

{|task)}

{(annotation|}

Add your 5-point critique here:

{|annotation)}

If you want to explore a wider range of recent data visualizations, this [list of lists](https://www.maartenlambrechts.com/2021/12/29/the-list-of-2021-visualization-lists.html) from Maarten Lambrechts and [best datavis projects of 2021](https://flowingdata.com/2021/12/31/best-of-2021) by Nathan Yau are good places to start.

In her book [Visualization Analysis and Design](#5-recommended-reading), Tamara Munzner asks an important question — **why do data visualization?** (Munzner, 2015, pp.1-9). This is important because in answering it, you should find it easier to design effective data visualizations. She considers a 'decision-making loop' where the visualization helps you to decide something, perhaps where to invest resources, what further information to gather, what action to take etc. She considers both the 'computer in the loop' and the 'human in the loop'. In other words, what is it that computation can offer in supporting decision making and what is it that humans can offer.

As an example, imagine you have a dataset that represents reported crime in various districts in a region. If you simply wanted the answer to the question "which district has the highest reported crime?", data visualization would not be very helpful here because you could answer the question with a simple database query (i.e. it is only 'the computer in the loop'). If on the other hand a crime analyst wanted to know whether reported crime reflected similar patterns and trends to perceived crime patterns or they wanted to explore the data to see if there were any unexpected patterns or outliers that might suggest a change in policy, visualization might be more useful (i.e. decision-making that requires 'the human in the loop').

{(infobox|}

If you design a data visualization to answer a question such as what is characteristic x in my dataset? (e.g. which country has the highest GDP?), data visualization will have little to offer that you couldn’t find out more easily with a simple database query. Good data visualization answers more complex questions that don’t have an easily computable answer.

{|infobox)}

Over the coming weeks, you will refine your critical ability to evaluate good design in data visualization. You will consider models, rules and heuristics that assist effective design, from the use of colour and symbolisation to layout and interaction. This should help you both to evaluate others' data visualizations, and to design your own more effectively.

### 1.1 What do we see when we visualize data?

Imagine we have some quantity represented by the 80 lines shown below — the length of each line represents the magnitude of some variable. If it helps, you could, for example, imagine the length of each line represents the average module mark of a student in a class (longer lines → higher marks).

{(task|}

Spend 30 seconds looking at the image below and see if you can use it to infer the characteristics of the data it represent.

{|task)}

![80 lines](https://staff.city.ac.uk/~jwo/datavis2022/session01/images/lines80.png)

{(annotation|}

Add your initial observations about the data represented by these 80 lines here.

{|annotation)}

In the live session in week 1, we will spend some time asking questions about the data represented by these lines. In doing so, we will consider how visualization design and understanding of human _perception_ (what we see) and _cognition_ (what we understand) can help us help us design more effectively.

For more on the science behind our perception of visual representations of data see [Franconeri et al (2021) – The science of visual data communication: What works](https://journals.sagepub.com/doi/full/10.1177/15291006211051956). This is a comprehensive review paper that covers many of the key themes we will visit during this module.

## 2. Literate Visualization and Litvis

In this module we will approach visualization design using _literate visualization_. This involves not only writing the code to generate data visualizations (_"what"_ specifications), but also combining the visualizations with a textual narrative that explains your design choices (_"why"_ justifications). We will consider frameworks to help you structure those justifications over the coming weeks so that you become more adept and efficient at data visualization design.

{(infobox|}

For more detail on literate visualization as a process and litvis as a technology, see [Wood et al, 2019](#5-recommended-reading) and this video presentation [Why not how! Telling Visualization Design Stories](https://youtu.be/pMHmQX3TZ8A). The general portal for literate visualization can be found at [litvis.org](http://litvis.org)

{|infobox)}

The elements of literate visualization are summarised below:

![litvis approach](https://staff.city.ac.uk/~jwo/datavis2022/session01/images/litvisApproach.png)

To make the process easier we will be using software called _litvis_ to allow you to specify visualization designs and integrate them with your design justifications. Litvis is a form of 'notebook' environment using Knuth's principles of [literate programming](https://en.wikipedia.org/wiki/Literate_programming) similar to [Jupyter](https://jupyter.org), [Observable](https://beta.observablehq.com) or [R-markdown](https://rmarkdown.rstudio.com) that integrate text, code and rendered output.

### 2.1 Setting Up Litvis

Before you can get going writing litvis documents you need to

1.  [Install the _VSCode_ editor with the litvis plug-in](https://staff.city.ac.uk/~jwo/datavis2022/programming/installingLitvis.pdf).

2.  [Set up git and GitHub](https://staff.city.ac.uk/~jwo/datavis2022/programming/settingUpGitHub.pdf).

The process is a little fiddly, but once you have set things up on your machine, you shouldn't need to do so again.
After you have completed both steps, you should have a folder structure (`session01`, `session02`, `gallery`, `portfolio` etc.) into which you can add these notes and any other content you create for this module.

Whenever you need to create a new litvis document, just open VSCode and create a new document with the `.md` (short for _markdown_) extension.

A litvis document looks like the following, containing an editing window on the left and the formatted document with graphics on the right:

![litvis anatomy](https://staff.city.ac.uk/~jwo/datavis2022/session01/images/litvisAnatomy.jpg)

_Frontmatter_ is enclosed by a pair of `---` and can contain references to any coding libraries and _narrative schemas_ (see below) that you wish to take advantage of in your document.

Below the frontmatter is the main body of the document that will contain a mixture of text to display and code to generate visualizations.

### 2.2 Textual Narrative

To write text in litvis you use [Github-Flavored markdown](https://guides.github.com/features/mastering-markdown/), which is an easy-to-learn way of writing formatted text. It is designed to be readable both in its 'raw' form and when formatted.

For example, here is how a heading, paragraph of text containing a hyperlink, and a bulleted list may be written in markdown and litvis:

```markdown
## Visualization Analysis & Design

A [book by Tamara Munzner](https://tinyurl.com/ycqp5cf2) that covers the _motivations_,
_approaches_ and _techniques_ for designing and building data visualizations.
Its approach is practitioner-focussed but built on a sound theoretical framework.
Some of the key elements of the book are:

- What: Data abstraction
- Why : Task abstraction
- Validating visualization
- Marks and channels
- Analysis case studies
```

When formatted, it might look like this:

> ## Visualization Analysis & Design
>
> A [book by Tamara Munzner](https://tinyurl.com/ycqp5cf2) that covers the _motivations_, _approaches_ and _techniques_ for designing and building data visualizations. Its approach is practitioner-focussed but built on a sound theoretical framework.
> Some of the key elements of the book are:
>
> - What: Data abstraction
> - Why : Task abstraction
> - Validating visualization
> - Marks and channels
> - Analysis case studies

To help guide you writing a document, you can follow _narrative schemas_, which are sets of _labels_, _rules_ and _styles_ that may be used in your document. For example, we might have a label called `aim` and a rule that says every document that follows the schema must contain at least one aim. Additionally, the schema might include a _style_ that highlights any text inside a pair of labels. A pair of 'aim' labels in a litvis document might look like this:

```markdown
{(aim|} Some text here {|aim)}
```

We will look at narrative schemas in later weeks when we consider different models for validating visualization designs.

### 2.3 Visualization Specification

To create visual depictions of data we need to specify the rules by which elements of data are transformed into something we can see and possibly interact with. We will be doing that by using the [Vega-Lite](https://vega.github.io/vega-lite/) grammar of interactive graphics. Vega-Lite reflects a _declarative_ approach to visualization creation. That is, you don't specify _how_ to create a graphic (as you might in a programming language like Java or Python), but instead _what_ you want the graphic design to be. For those familiar with relational databases, you can think of Vega-Lite as a visualization equivalent of [SQL](https://en.wikipedia.org/wiki/SQL).

In its native form, a Vega-Lite visualization specification is expressed as [JSON](https://en.wikipedia.org/wiki/JSON) (JavaScript Object Notation). While this is a flexible and web-friendly way of representing data, it is not very easy to work with directly, especially when visualization specifications become more complex. Instead, in Litvis, we will use the language [Elm](https://elm-lang.org) to generate Vega-Lite JSON specifications, with the library [elm-vegalite](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/).

{(infobox|}

Elm is likely to be an entirely new language to almost all of you. For those who are taking or have taken the module _IN3043 Functional Programming_ and have used [Haskell](https://www.haskell.org), here is a [comparison of Haskell and Elm](https://staff.city.ac.uk/~jwo/datavis2022/programming/haskell.pdf). Don't worry if this is all new to you – Elm is designed to be beginner friendly and we will build our use of Elm slowly and steadily though this module.

In parallel with the visualization lab exercises for this module, you will also find regular _coding gym_ exercises that will help you learn the basics of the Elm language. Doing so should make it easier and quicker for you to create visualization specifications.

{|infobox)}

#### London Bicycle Hires

Suppose we wished to create a data visualization that helps us understand the patterns of (Santander) bicycle hires in London over a ten year period. We have access to a [dataset from TfL](https://gicentre.github.io/data/bicycleHiresLondon.csv) that records the number of hires per month and the average length of time a bike was hired.

Here is a litvis document with a visualization specification that does this (you can copy this into a new litvis document called `bicyclehires.md` if you wish to try it for yourself):

````txt
---
elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

```elm {l=hidden}
import VegaLite exposing (..)
```

```elm {v}
myBarchart : Spec
myBarchart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv"

        enc =
            encoding
                << position X [ pName "Month", pTemporal ]
                << position Y [ pName "NumberOfHires", pQuant ]

    in
    toVegaLite [ width 640, data [], enc [], bar [] ]
```
````

The three lines at the top encased in a pair of `---` form the _frontmatter_ and in this case state that we wish to use the Elm language with the package `elm-vegalite`. By including this dependency, Litvis will download the elm-vegalite package and store it locally on your machine.

The rest of the document contains two _code blocks_, each identified by a pair of triple backticks.

The first code block has the _header_ `elm {l=hidden}` which means the block will contain Elm code but its contents should be hidden in the formatted view. The content itself – `import VegaLite exposing (..)` means "_make all the functions that come with the Vega-Lite package available to all code blocks in this document_". This is a necessary first step in order to make use of the library that will convert your Elm code into Vega-Lite JSON for rendering.

Most of your litvis documents will have a very similar first 9 lines as above, so you can usually just copy/paste them whenever you need to create a new document.

#### A bar chart specification

Now let's consider the second code block. It has the header `elm {v}` which indicates that the code block is written in Elm and should create a visualization. The block itself contains the main visualization specification which we will consider line by line:

```elm {l highlight=[1,2,3,4,5,6,7,8,9,10,11,12]}
myBarchart : Spec
myBarchart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv"

        enc =
            encoding
                << position X [ pName "Month", pTemporal ]
                << position Y [ pName "NumberOfHires", pQuant ]
    in
    toVegaLite [ width 640, data [], enc [], bar [] ]
```

Programs in Elm work by defining _functions_ that perform specific tasks. In this case we have created a function called `myBarChart` (named in lines 1 and 2) that will provide the visualization specification. The name of the function is up to you, but it makes sense to give it a name that describes what the function does in some way. Function names in Elm should be in [lowerCamelCase](https://en.wikipedia.org/wiki/Camel_case). There are two lines in a main function declaration: the first (line 1) states that the function `myBarChart` will provide a `Spec` when it is executed. A Spec is elm-vegalite's name for a visualization specification; all the visualization specifications you will write will return a `Spec` in this way. Below that (line 2), we name the function again with an `=` symbol below which is the code that actually creates the specification.

Within the `myBarChart` function, we can define further functions using `let..in` (lines 3-11). Most specifications you write will include at least two of these - one that states the source of the data you will be visualizing (lines 4-5 in this example) and one to specify the _encoding rules_ that declare which elements of the data will be transformed into which visual _channels_ (lines 7-10 in this example).

Visual encoding is achieved by calling elm-vega's [encoding](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#encoding) function (line 8) and then _chaining_ a series of channel encoding rules with the `<<` symbol (lines 9-10).

Line 9 can be translated as "_encode the temporal data in the `Month` column of the dataset as the x (horizontal) position in the visualization_". Likewise, line 10 says "_encode the quantitative data in the `NumberOfHires` column of the dataset as the y (vertical) position in the visualization_". When encoding a channel (a [position](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#position) channel in this example), it is possible to specify a range of encoding options, so these are separated by commas and encased in `[` ... `]` indicating an Elm [list](https://package.elm-lang.org/packages/elm/core/latest/List).

`pTemporal` and `pQuant` are examples of _positional measurement types_. The possible types include:

| Measurement type | Description                                    | Examples                              |
| ---------------- | ---------------------------------------------- | ------------------------------------- |
| `Nominal`        | Categories that have no order                  | People's names; country names         |
| `Ordinal`        | Ordered categories                             | Small / Medium / Large; 1st, 2nd, 3rd |
| `Quant`          | Quantitative numerical measurements and counts | Price; temperature; frequency         |
| `Temporal`       | Time-related                                   | Dates; times                          |
| `Geo`            | Geographic locations                           | Longitude/Latitude; country outline   |

Finally, line 12 assembles the data and encoding functions in another list and further specifies that the data items should be specified with the [bar mark](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#bar) and that the visualization should be 640 pixels wide. The entire specification is now represented by this list which is passed to the [toVegaLite](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#toVegaLite) function to convert it into JSON ready for rendering.

When viewed in Litvis, the specification generates the following visualization:

^^^elm v=myBarchart^^^

Notice that nowhere in our specification did we need to say where to position axis labels, how many bars we needed, what scale range to use on either axis etc. That's the beauty of declarative specification – we simply state **what** we want ("_give me a bar chart that positions bars according to month and sizes them according to number of hires_"), and Litvis/Vega-Lite works out how to draw it. This means we can spend more time thinking about visualization design, and less about coding the details required to render it.

Suppose we wanted to incorporate the average length of time a bike was hired for into our visualization. One way of doing this would be to colour each bar according to the average hire duration for that month. We could amend our specification to do this as follows:

```elm {l highlight=[11]}
mySecondBarchart : Spec
mySecondBarchart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv"

        enc =
            encoding
                << position X [ pName "Month", pTemporal ]
                << position Y [ pName "NumberOfHires", pQuant ]
                << color [ mName "AvHireTime", mQuant ]
    in
    toVegaLite [ width 640, data [], enc [], bar [] ]
```

Here, line 11 adds another channel encoding, this time the [color](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#color) channel rather than position, encoding the data in the `AvHireTime` column of the dataset. When we were encoding position, we referenced the data column with [pName](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pName) and its type with [pQuant](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pQuant) or [pTemporal](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pTemporal), whereas when encoding a colour channel, we use [mName](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#mName) and [mQuant](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#mQuant) where `m` refers to "mark" rather than the `p` of "position".

With just this one extra line, our visualization now includes a default legend and coloured bars!

^^^elm {v=mySecondBarchart}^^^

{(task|}

What explains the darker bars during 2020? Can you say anything else about the relationship between the number of bicycle hires and the average duration of each hire?

{|task)}

{(infobox|}

You may have noticed the links in these notes to documentation describing various _elm-vegalite_ functions. I recommend bookmarking this [VegaLite API documentation page](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite) as it is a valuable resource for customising your litvis visualization specifications. You may also find it useful to bookmark [Elm's core API documentation page](https://package.elm-lang.org/packages/elm/core/latest/).

{|infobox)}

## 3. Data Visualization Models

You’ve seen many examples of data visualization (e.g. the [Information is Beautiful showcase](https://www.informationisbeautifulawards.com/showcase?award=2019&pcategory=winner&type=awards)) that help us to explore _why_ we create visualizations, and you’ve seen that we can construct our own visualizations by specifying _what_ we wish to create. In essence, the aim of this module is to bring together those two approaches.

In attempting to explore and answer the _why_ questions of visualization construction, we draw upon a range of skills including creativity, storytelling, task analysis and design. To make that easier, we will explore some frameworks that enable us to structure the questions we might ask ourselves in the design process. Two of the most relevant are those created by Tamara Munzner — one of the world’s leading information visualization academics, and the Leland Wilkinson who formalised the [Grammar of Graphics](#5-recommended-reading).

### 3.1 Munzner's Visualization Analysis and Design Framework

Munzner’s framework, that forms the structure of her book [Visualization Analysis and Design](#5-recommended-reading), is centred on three questions that drive data visualization design and production.

1.  **What** are you wishing to show?
2.  **Why** is the task, for which visualization contributes, being performed?
3.  **How** is the visualization constructed?

We will focus on the first two of these questions during this module (leaving the _how_ question to be solved by Vega-Lite), but it is worth emphasising now that the purpose of the framework is to make the task of choosing from the huge space of data, tasks and idioms (techniques) easier. It recognises that good visualization is as much thinking about the task it is designed to address as it is the selection of appropriate data and the creating of appropriate techniques or idioms.

[Chapter 1](#5-recommended-reading) of her book elaborates on this approach and is recommended reading for all.

### 3.2 The Grammar of Graphics

Several visualization systems including Vega-Lite (and notably R's [ggplot2](http://vita.had.co.nz/papers/layered-grammar.pdf)) base their approach on Leland Wilkinson's [Grammar of Graphics](#5-recommended-reading) (or GoG for short). The GoG provides a formal approach to considering the transformation steps necessary to turn some data into something visual. These are summarised below:

![grammar of graphics transforms](https://staff.city.ac.uk/~jwo/datavis2022/session01/images/grammarOfGraphics.png)

It is worth noting that for any given visualization, one or more of these 7 steps may involve an _identity transform_ (i.e. a transform that does not have any effect). For example, in the bicycle hires example above, the "statistics" stage is an identity transform in that no new statistics are generated between the "algebra" and "geometry" steps.

The GoG is useful because it identifies the steps we have to go through to fully and unambiguously specify what we want a visualization to do. It also helps us to ask 'why' questions at each stage to assist us in justifying our visualization designs.

It is not a perfect model, for example, the original GoG does not mention how interaction may be incorporated into a specification, nor how multiple views may be combined in a single visualization. We will consider how Vega-Lite (and litvis) has extended the grammar to incorporate these in later sessions, but if you would like a look-ahead, see this [OpenVis video presentation](https://youtu.be/9uaHRWj04D4) from the Vega-Lite team.

A note on _aesthetics_: Wilkinson identifies the last transformation "aesthetics" as the mapping of spatially embedded transformations of data into some perceivable form. That is, the transformation of the non-graphic into the graphic (or auditory or olfactory).

Other have argued that "aesthetics" is more than this. For example, Ge Wang, in his book _Artful Design_ says

> Aesthetics is how we experience a thing —how it emotionally, intellectually, psychologically, and socially affects us. It is everything **beyond** the thing's function. Yet aesthetics does not usurp or live apart from functionality; instead, it gives context, meaning, and essence to a thing, making it what it is.
>
> In the kitchen of design, aesthetics is not the spice with which you garnish the casserole of functionality: it **is** the casserole, the ingredients you bake into it.

We will discuss this wider view of aesthetics in later sessions as we consider critiques of visualization design and its role in storytelling and emotional _affect_.

## 4. Conclusions

Data visualization is still a comparatively new and exciting field in modern data analysis and communication. There are vast numbers of digital datasets available to us, but that we are often poorly equipped to handle such data and extract useful information from it. Data visualization offers an engaging and powerful means to explore the complexities of datasets and to communicate findings and stories about those data to others.

By taking this module, and following the approach of _literate visualization_, you will become equipped not only to evaluate and design effective data visualization, but also to build your own data visualization applications. We have seen in this first session how to get the software litvis to create simple data visualizations enclosed in a formatted text document. We will continue to use Litvis to build increasingly sophisticated documents that answer both the _what_ and _why_ of data visualization, and in doing so, you will develop your own visualization design skills.

## 5. Recommended Reading

To become good at data visualization you will need to read beyond the lecture notes. 'Recommended reading' points you to accessible sources of further information related to the content of each session.

If you read a single journal paper for this module, make it Franconeri et al (2021) which provides a comprehensive overview of many of the issues we will cover in this module.

**Franconeri, S., Padilla, L., Shah, P., Zacks, J. and Hullman, J.** (2021) [The science of visual data communication: What works](https://journals.sagepub.com/doi/full/10.1177/15291006211051956), _Psychological Science in the Public Interest_, 22(3) pp.110-161.

**Kirk, A.** (2019) Chapter 1: _Defining data visualization_, pp.15-30 in [Data Visualisation: A Handbook for Data Driven Design](https://go.exlibris.link/rwjXvTCv), Sage. (paper copy in library)

**Munzner, T.** (2015) Chapter 1: _What’s Vis and Why Do It?_, pp.1-19 in [Visualization Analysis and Design](https://go.exlibris.link/9jMy6fQG), CRC Press. (online and paper copies available from library)

**Wood, J., Kachkaev, A. and Dykes, J.** (2019) [Design exposition with literate visualization](https://openaccess.city.ac.uk/20081). _IEEE Transactions on Visualization and Computer Graphics_, 25(1) pp.759-768.

### References

There is no requirement to read 'references' but they are provided should you wish to follow a citation from the lecture notes in more detail.

**Wilkinson, L.** (2010a) [The Grammar of Graphics: Wiley interdisciplinary reviews](https://go.exlibris.link/2pPxZX51). _Computational Statistics_, 2010 (2), pp.673–677.

**Wilkinson, L.** (2010b) [The Grammar of Graphics](https://go.exlibris.link/Y2gQqNZG). Springer, ISBN: 978 0387 24544 7

---

## 6. Practical Exercises

{(task|}

Every week I will provide some exercises for you to complete in the interactive tutorials following each lecture, and/or in your own time. You should make best effort to complete these week by week and [push your answers to your GitHub repo](https://staff.city.ac.uk/~jwo/datavis2022/programming/settingUpGitHub.pdf). If you make a serious attempt at these exercises within the week (even if you don't finish all of them), you will receive marks that contribute to your overall coursework (20% in total). Doing so will also help you build the skills necessary to complete the project coursework that forms the main contribution to your module mark.

I encourage you to use the time in the scheduled interactive tutorial to discuss the questions with other students in your group. This might mean you spend most of the time in these lab sessions focussing on the more discursive questions, which I will label with ![Good discussion topic](https://img.shields.io/badge/Good%20discussion%20topic-blue.svg), leaving the more programming focussed ones for your own time. If you have an attempt at the questions before the live session, it may be easier to bring relevant questions for discussion.

{|task)}

A week after each session I will provide you with **feedback** on how you have got on (based on observations and questions in the labs and on Slack), along with answers to the questions.

### 1. Register with the datavis slack

If you haven't already done so, create an account on the [dataviscity slack](https://dataviscity.slack.com). You will need to use your `.city.ac.uk` email address when requested. This will be the main forum for discussion and questions/answers relating to the content of this module.

### 2. Setting up Litvis

If you haven't done so already, [install litvis on your computer](https://staff.city.ac.uk/~jwo/datavis2022/programming/installingLitvis.pdf) and [set up your GitHub coursework repo](https://staff.city.ac.uk/~jwo/datavis2022/programming/settingUpGitHub.pdf).

Make sure you have edited the `README.md` files in main folder of your local repo and in the `portfolio` folder to include your name and City email address.

### 3. Datavis evaluation

![Good discussion topic](https://img.shields.io/badge/Good%20discussion%20topic-blue.svg)

As indicated in the lecture notes, choose one of the examples from the [Information is Beautiful awards](https://www.informationisbeautifulawards.com/showcase?action=index&award=2019&controller=showcase&page=1&pcategory=short-list&type=awards) and provide a brief critique of it using the 5 'data visualization criteria':

1.  Represents a complex dataset graphically that could not as effectively be represented via other means.
2.  Often emphasises _connections_ and _comparisons_ between items of data.
3.  Frequently involves _interaction_ to allow a user to influence what they see.
4.  A _narrative_ supporting the construction of knowledge.
5.  Generates an _aesthetic response_ in order to encourage users to engage with the data or question.

You can add the text by starting VSCode and opening `practicalExercises.md` which you can find in the `portfolio` folder of your coursework repo. In your breakout lab group, discuss and compare your evaluations.

### 4. Creating a litvis sketch

Still within VSCode, copy the following specification into your `practicalExercises.md` file.

```elm {l}
bikehireBarchart : Spec
bikehireBarchart =
    let
        data =
            dataFromUrl "https://gicentre.github.io/data/bicycleHiresLondon.csv"

        enc =
            encoding
                << position X [ pName "Month", pTemporal ]
                << position Y [ pName "NumberOfHires", pQuant ]
    in
    toVegaLite [ width 640, data [], enc [], bar [] ]
```

- Add a `v` to the code block header (i.e. instead of `elm {l}`, make it `elm {l v}`) so that the specification is visualized as well as listed. Check you can view the formatted visualization (in VSCode , keyboard shortcut `Ctrl-K` or `Cmd-K` followed by `V`)

- Add some brief (markdown) text to the document explaining what the visualization is intended to show and any patterns you have observed from the visualization.

### 5. Creating a scatterplot

Add to the document you have created in the previous task by providing a specification to display a scatterplot comparing number of hires per month with the average time of a bike hire in a month.

**_Hint:_** A scatterplot uses the [circle mark](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#circle) rather than `bar` mark to symbolise each pair of values. You will also have to change which data columns will be encoded by `position X` and `position Y`.

### 6. Data Challenge

The site [What the UK thinks EU](https://whatukthinks.org/eu/) tracks and analyses UK attitudes towards the European Union. It includes data on opinion polls that have asked the question "_Should the United Kingdom remain a member of the European Union, or leave the European Union?_" since 2016.

The first few lines of the data table look like this:

| Date             | Pollster     | Answer                 | Percent |
| ---------------- | ------------ | ---------------------- | ------- |
| 2019-12-12 00:00 | BMG Research | remain                 | 49      |
| 2019-12-12 00:00 | BMG Research | leave                  | 41      |
| 2019-12-12 00:00 | BMG Research | no answer / don't know | 10      |
| 2019-12-06 00:00 | BMG Research | remain                 | 51      |
| 2019-12-06 00:00 | BMG Research | leave                  | 42      |
| 2019-12-06 00:00 | BMG Research | no answer / don't know | 7       |
| 2019-11-29 00:00 | BMG Research | remain                 | 50      |
| 2019-11-29 00:00 | BMG Research | leave                  | 42      |
| 2019-11-29 00:00 | BMG Research | no answer / don't know | 8       |
| 2019-11-21 00:00 | BMG Research | remain                 | 51      |
| 2019-11-21 00:00 | BMG Research | leave                  | 43      |
| 2019-11-21 00:00 | BMG Research | no answer / don't know | 6       |
| 2019-11-15 00:00 | BMG Research | remain                 | 50      |
| 2019-11-15 00:00 | BMG Research | leave                  | 42      |
| 2019-11-15 00:00 | BMG Research | no answer / don't know | 8       |
| 2019-10-30 00:00 | Survation    | remain                 | 52      |
| 2019-10-30 00:00 | Survation    | leave                  | 45      |
| 2019-10-30 00:00 | Survation    | no answer / don't know | 3       |
| :                | :            | :                      | :       |

You can access a copy of the data, which are stored in JSON format rather than CSV at
[https://gicentre.github.io/data/euPolls.json](https://gicentre.github.io/data/euPolls.json).

Can you create a litvis document that displays some aspect of these data and provides a brief justification for your visualization design and any patterns it reveals?

**_Hint:_** The visualization specification is likely to be similar to the bicycle hire example above, but you will need to change the URL of the data source, the names of the columns referenced by `pName` and `mName`. You may also wish to consider other graphic marks in addition to `bar` and `circle`, such as [line](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#line) and [area](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#area).

### 7. Design Challenge

![Good discussion topic](https://img.shields.io/badge/Good%20discussion%20topic-blue.svg)

Were you able to implement your design successfully in the data challenge above? It is likely that your design was constrained by what you were able to code in elm-vegalite. As a final design challenge, try sketching an alternative design on paper. Think about 'the human in the loop', and what would be most effective in providing a meaningful depiction of the polling data.

Take a photo of your sketch and add it to your session 1 `practicalExercises.md` document in your portfolio. You can add an image to a litvis document by storing the image file to the same folder and then adding the line

```markdown
![image caption](myImagefile.jpg)
```

where `myImagefile.jpg` is the name of the image file you have placed in the litvis document folder.

### 8. Coding Gym

Complete the session 1 coding gym which will give you practice at working with the Elm language. This can be found in the `portfolio` folder of your coursework repo.

### 9. Push Your Answers

As you complete your answers to the questions above, make sure you _stage_, _commit_ and _push_ your updated documents to your coursework repo. For instructions on how to do this, see _Managing your datavis repo_ in [Setting up git and GitHub](https://staff.city.ac.uk/~jwo/datavis2022/programming/settingUpGitHub.pdf).

---

_Check your progress._

- [ ] I can critique a data visualization based on the five criteria given in the lecture notes.
- [ ] I have installed the _VSCode_ editor with the _litvis_ plug-in on my computer.
- [ ] I have a GitHub account, git installed and a local coursework repo installed on my computer.
- [ ] I can write simple markdown text in a litvis document.
- [ ] I can specify a simple visualization using elm-vega.
- [ ] I have related a visualization design and specification to Munzner's 'what-why-how' framework.
- [ ] I have related a visualization specification to some of Wilkinson's 7 _Grammar of Graphics_ steps.
