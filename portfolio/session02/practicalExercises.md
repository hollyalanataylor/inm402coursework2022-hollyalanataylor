---
id: litvis

narrative-schemas:
  - ../../narrative-schemas/teaching.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../../css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

<!-- Everything above this line should probably be left untouched. -->

# Session 2: Practical Exercises

{(task|}

Use this document as a place to add your answers to the week's practical exercises.

{|task)}

## 7. Practical Exercises

{(task|} Please complete the following exercises before next week's session. They should be added to your session02 folder of your portfolio. Remember to stage, commit and push your answers to github so I can see them. {|task)}

### 1. Evaluating Design Approaches

![Good discussion topic](https://img.shields.io/badge/Good%20discussion%20topic-blue.svg)

Make sure you have completed all the annotations in response to the tasks above in these lecture notes (e.g. questions on Corum's talk, Weber's scarf etc.). For your continuous assessment, copy your annotations (or a summary of them) into `practicalExercises.md` in session02 of your portfolio folder.

### 2. Creating a Minimalist Tufte Visualization

Create a litvis document called `imdbRoot.md` that contains the following:

````markdown
---
id: litvis

elm:
  dependencies:
    gicentre/elm-vegalite: latest
---

```elm {l=hidden}
import VegaLite exposing (..)
```

# Internet Movie Database

Here is the data source for creating visualizations based on the Internet Movie Database (IMDB):

```elm {l}
data : Data
data =
    dataFromUrl "https://vega.github.io/vega-lite/data/movies.json" []
```
````

Create a second document called `defaultHistogram.md` that contains the following simple histogram specification:

````markdown
---
follows: imdbRoot
---

A default histogram:

```elm {l v}
histogram : Spec
histogram =
    let
        enc =
            encoding
                << position X [ pName "IMDB Rating", pBin [] ]
                << position Y [ pAggregate opCount ]
                << color [ mName "IMDB Rating", mOrdinal ]
    in
    toVegaLite [ data, enc [], bar [] ]
```
````

Finally, create a third document that also `follows: imdbRoot`, but this time containing a modified version of the histogram that follows some of Tufte's minimalist design principles.

_Note:_ This example creates a frequency histogram from a single data column (`IMDB Rating`) using the elm-vegalite functions [pBin](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pBin) and [pAggregate opCount](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pAggregate). You won't need to change those aspects of the specification, but you should be able to change others such as axes, legends, colour encoding etc.

Add a brief evaluation to this third litvis document that assesses which elements of the minimalist design work and which do not.

### 3. _Creativity Challenge:_ Hand Drawn Personal Visualization Sketch

![Good discussion topic](https://img.shields.io/badge/Good%20discussion%20topic-blue.svg)

Following the principles of data humanism, try logging some personal data (can be anything ranging from time spent outside, to a food diary, to communications with others) and sketching a hand drawn design. Take a photo of it and embed it in your `practicalExercises.md` litvis document (in session02 of your portfolio) with a brief explanation of what you have visualized and a brief reflection on the process of constructing and viewing the visualization.

Share your sketch by posting a photo of it and brief explanation on Slack.
