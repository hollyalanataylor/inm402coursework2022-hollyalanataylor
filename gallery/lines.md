---
follows: gallery
id: litvis
---

@import "../lectures/css/datavis.less"

_Elm-Vegalite Gallery._

1.  [Scatter and strip plots](scatter.md)
1.  [Bar charts](bars.md)
1.  **Line charts**
1.  [Radial charts](radial.md)
1.  [Area charts and streamgraphs](area.md)
1.  [Table-based charts](table.md)
1.  [Distribution charts](distrib.md)
1.  [Labelling, Annotation and Layering](layers.md)
1.  [Interactive charts](interactive.md)
1.  [Interactive linked views](interactiveLinked.md)
1.  [Faceted charts](facet.md)
1.  [Repeats and concatenation](concats.md)
1.  [Geographic maps](geo.md)

---

# Line charts

Examples that use data from external sources tend to use files from the Vega-Lite and giCentre data servers. For consistency the paths to these data locations are defined here:

```elm {l}
vegaPath : String
vegaPath =
    "https://cdn.jsdelivr.net/npm/vega-datasets@2.2/data/"


giCentrePath : String
giCentrePath =
    "https://gicentre.github.io/data/"
```

## Simple time series

Line charts work well for showing how quantitative variables change over time. Here [filter](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#filter) is used to extract just the Google stock price from a more complex dataset.

```elm {v}
singleLineChart : Spec
singleLineChart =
    let
        data =
            dataFromUrl (vegaPath ++ "stocks.csv") []

        trans =
            transform << filter (fiExpr "datum.symbol === 'GOOG'")

        enc =
            encoding
                << position X [ pName "date", pTemporal, pTitle "" ]
                << position Y [ pName "price", pQuant ]
    in
    toVegaLite [ width 400, data, trans [], enc [], line [] ]
```

## Line chart with point markers

Similar to the previous example, but with additional point markers at each data point. Point markers can be specified with [maPoint](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#maPoint) and customised with [pmMarker](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#pmMarker). This can be useful when distinguishing actual data values from interpolated (line) symbols.

```elm {v}
lineChartWithMarkers : Spec
lineChartWithMarkers =
    let
        data =
            dataFromUrl (vegaPath ++ "stocks.csv") []

        trans =
            transform << filter (fiExpr "datum.symbol === 'GOOG'")

        enc =
            encoding
                << position X [ pName "date", pTemporal, pTitle "" ]
                << position Y [ pName "price", pQuant ]
    in
    toVegaLite [ width 400, data, trans [], enc [], line [ maPoint (pmMarker [ maColor "black" ]) ] ]
```

## Smoothed line chart

To de-emphasise data points and emphasise trend, a smoothed line interpolation may be applied between points using [mInterpolate](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#maInterpolate) and setting the interpolation type to [miMonotone](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#miMonotone).

```elm {v}
smoothLine : Spec
smoothLine =
    let
        data =
            dataFromUrl (vegaPath ++ "stocks.csv") []

        trans =
            transform << filter (fiExpr "datum.symbol === 'GOOG'")

        enc =
            encoding
                << position X [ pName "date", pTemporal ]
                << position Y [ pName "price", pQuant ]
    in
    toVegaLite [ width 400, data, trans [], enc [], line [ maInterpolate miMonotone ] ]
```

## Trend line

To provide an even smoother trend, we can use the [loess](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#loess) transformation to calculate the locally smoothed relationship between two variables. Loess expects two quantitative variables, so we convert the date into a numeric value based on its year and 1/12th of its numeric month. To keep the calculated year values as whole numbers on the axis labels we explicitly format them with [axFormat](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#axFormat) and provide the [number formatting code](https://github.com/d3/d3-format#locale_format) `d` to round to a whole number.

```elm {v}
trendLine : Spec
trendLine =
    let
        data =
            dataFromUrl (vegaPath ++ "stocks.csv") []

        trans =
            transform
                << filter (fiExpr "datum.symbol === 'GOOG'")
                << calculateAs "year(datum.date) + month(datum.date)/12" "yrMonth"
                << loess "price" "yrMonth" []

        enc =
            encoding
                << position X [ pName "yrMonth", pQuant, pAxis [ axFormat "d", axTitle "" ] ]
                << position Y [ pName "price", pQuant ]
    in
    toVegaLite [ width 400, data, trans [], enc [], line [] ]
```

## Stepped line chart

If the value being shown with a line chart moves between discrete steps, line interpolation can be made to step between values with [miStepwise](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#miStepwise), [miStepAfter](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#miStepAfter) or [miStepBefore](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#miStepBefore).

```elm {v}
steppedLine : Spec
steppedLine =
    let
        data =
            dataFromUrl (vegaPath ++ "stocks.csv") []

        trans =
            transform
                << filter (fiExpr "datum.symbol === 'GOOG'")

        enc =
            encoding
                << position X [ pName "date", pTemporal, pTitle "" ]
                << position Y [ pName "price", pQuant ]
    in
    toVegaLite [ width 400, data, trans [], enc [], line [ maInterpolate miStepAfter ] ]
```

## Line chart with conditional axes

We can make the grid lines and axis labels on any chart conditional on some properties of the data. Here we make the grid and labels depend on the month of the year (making January bolder with year label).

```elm {v}
conditionalAxes : Spec
conditionalAxes =
    let
        cfg =
            configure
                << configuration (coAxis [ axcoDomainColor "#ddd", axcoTickColor "#ddd" ])

        data =
            dataFromUrl (vegaPath ++ "stocks.csv") []

        trans =
            transform
                << filter (fiExpr "datum.symbol === 'GOOG'")

        enc =
            encoding
                << position X
                    [ pName "date"
                    , pScale [ scDomain (doMinDt [ dtYear 2006 ]) ]
                    , pTemporal
                    , pAxis
                        [ axTitle ""
                        , axLabelAlign haLeft
                        , axLabelExpr "[timeFormat(datum.value, '%b'), timeFormat(datum.value, '%m') == '01' ? timeFormat(datum.value, '%Y') : '']"
                        , axLabelOffset 4
                        , axLabelPadding -24
                        , axTickSize 30
                        , axDataCondition
                            (fiEqual "value" (dt [ dtMonth Jan ]) |> fiOpTrans (mTimeUnit monthDate))
                            (cAxGridDash [] [ 2, 2 ])
                        , axDataCondition
                            (fiEqual "value" (dt [ dtMonth Jan ]) |> fiOpTrans (mTimeUnit monthDate))
                            (cAxTickDash [] [ 2, 2 ])
                        , axDataCondition
                            (fiEqual "value" (dt [ dtMonth Jan ]) |> fiOpTrans (mTimeUnit monthDate))
                            (cAxGridColor "#999" "#ccc")
                        ]
                    ]
                << position Y [ pName "price", pQuant ]
    in
    toVegaLite [ cfg [], width 600, data, trans [], enc [], line [ maClip True ] ]
```

## Coloured multi-line chart

We can show multiple lines on a chart by encoding categorical variable with colour. Here each company (unhelpfully called `symbol` in the stocks dataset) is encoded with colour, that has the effect generating a separate line for each set of stock price values associated with each company.

```elm {v}
multiLines : Spec
multiLines =
    let
        data =
            dataFromUrl (vegaPath ++ "stocks.csv") []

        enc =
            encoding
                << position X [ pName "date", pTemporal, pTitle "" ]
                << position Y [ pName "price", pQuant ]
                << color [ mName "symbol", mTitle "Company" ]
    in
    toVegaLite [ width 400, data, enc [], line [] ]
```

## Textured multi-line chart

Instead of colour, we can use the [strokeDash channel](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#strokeDash) to generate different stroke styles for categorical data. These could be double encoded with colour by adding a colour channel as in the example above.

```elm {v}
texturedLines : Spec
texturedLines =
    let
        data =
            dataFromUrl (vegaPath ++ "stocks.csv") []

        enc =
            encoding
                << position X [ pName "date", pTemporal, pTitle "" ]
                << position Y [ pName "price", pQuant ]
                << strokeDash [ mName "symbol", mTitle "Company" ]
    in
    toVegaLite [ width 400, data, enc [], line [] ]
```

## Single colour multi-line chart

Sometimes you may wish to split a line chart by some variable (as the colour example above), but not allocate a different colour to each line. This can be achieved by encoding with a [detail](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#detail) channel rather than [color](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#color) channel.

In this example, we take a year's timestamps of Oxides of Nitrogen data (NOX) (air pollution) in Putney, London, and derive new data fields, with [calculateAs](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#calculateAs), that extract hour of day and day of year. Splitting by day of year with the `detail` channel gives 365 lines, each covering 24 hours of data. Because there so many overlapping lines, the line opacity is set to 5% (with [maOpacity](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#maOpacity)) to provide a visual indication of line density.

```elm {v}
airQualityLines : Spec
airQualityLines =
    let
        data =
            dataFromUrl (giCentrePath ++ "putneyAirQuality2018.csv") []

        trans =
            transform
                << calculateAs "hours(datum.DateTime) + (minutes(datum.DateTime)/60)"
                    "hourOfDay"
                << calculateAs "month(datum.DateTime) + (date(datum.DateTime) / 100)"
                    "dayOfYear"

        enc =
            encoding
                << position X [ pName "hourOfDay", pQuant ]
                << position Y [ pName "NOX", pQuant ]
                << detail [ dName "dayOfYear" ]
    in
    toVegaLite [ width 600, height 300, data, trans [], enc [], line [ maOpacity 0.05 ] ]
```

## Multi-trail chart

A 'trail' is like a line but its thickness can be used to encode a data field. The air quality example below is similar to that above except we use the [trail](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#trail) mark instead of [line](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#line) and control the width of the trail lines via the [size](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#size) channel. The effect is to give greater weighting to higher pollution levels.

```elm {v}
airQualityTrails : Spec
airQualityTrails =
    let
        data =
            dataFromUrl (giCentrePath ++ "putneyAirQuality2018.csv") []

        trans =
            transform
                << calculateAs "hours(datum.DateTime) + (minutes(datum.DateTime)/60)" "hourOfDay"
                << calculateAs "month(datum.DateTime) + (date(datum.DateTime) / 100)" "dayOfYear"

        enc =
            encoding
                << position X [ pName "hourOfDay", pQuant ]
                << position Y [ pName "NOX", pQuant ]
                << detail [ dName "dayOfYear" ]
                << size [ mName "NOX", mQuant, mLegend [] ]
    in
    toVegaLite
        [ width 600, height 300, data, trans [], enc [], trail [ maOpacity 0.1 ] ]
```

## Connected Scatterplot

A [connected scatterplot](https://research.tableau.com/sites/default/files/Haroz-TVCG-2016.pdf) can be created by customizing line order and adding a point marker in the line mark definition. Connected scatterplots are useful when you wish to show how a relationship between two variables changes over time or some other continuous variable.

In this example, the mean UK household monthly income of the poorest and richest 5% after housing costs is compared joining points in sequence from 1961 (bottom left) to 2020 (top right). It shows how income inequality has increased over time, in the 1980s and since the 2008 financial crisis.

_Data from the [Institute for Fiscal Studies](https://www.ifs.org.uk/tools_and_resources/incomes_in_uk)_

```elm {v}
connectedScatter : Spec
connectedScatter =
    let
        data =
            dataFromUrl (giCentrePath ++ "incomeInequality2020.csv") []

        enc =
            encoding
                << position X [ pName "5pcIncome", pQuant, pScale [ scZero True ] ]
                << position Y [ pName "95pcIncome", pQuant, pScale [ scZero True ] ]
                << order [ oName "Year" ]
    in
    toVegaLite
        [ width 400
        , height 400
        , data
        , enc []
        , line
            [ maPoint (pmMarker [ maFilled False, maStrokeWidth 1 ])
            , maInterpolate miMonotone
            ]
        ]
```

## Slope Charts

Slope charts, proposed by [Edward Tufte](https://www.edwardtufte.com/bboard/q-and-a-fetch-msg?msg_id=0003nk), allow comparison of pairs of values in a list, typically over time. In this example, changes in median crop yields between 1931 and 1932 are compared for different farming sites, highlighting the erroneous value of "Morris" in 1931.

```elm {v}
slopeChart : Spec
slopeChart =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        data =
            dataFromUrl (vegaPath ++ "barley.json") []

        enc =
            encoding
                << position X
                    [ pName "year"
                    , pScale [ scPadding 0.1 ]
                    , pAxis [ axTitle "", axDomain False, axLabelAngle 0 ]
                    ]
                << position Y
                    [ pName "yield"
                    , pAggregate opMedian
                    , pAxis [ axGrid False ]
                    ]
                << color [ mName "site" ]
    in
    toVegaLite [ width 200, cfg [], data, enc [], line [] ]
```

## Comet Chart

We can use trail marks whose thickness can vary along their length to create 'comet charts' for comparing variables in two categories (here, two points in time). There are more space-efficient than slope charts, although less precise in their ability to show the degree of change between states.

```elm {v}
cometChart : Spec
cometChart =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coLegend [ lecoOrient loBottom, lecoDirection moHorizontal ])

        data =
            dataFromUrl (vegaPath ++ "barley.json")

        trans =
            transform
                << pivot "year" "yield" [ piGroupBy [ "variety", "site" ] ]
                << foldAs [ "1931", "1932" ] "year" "yield"
                << calculateAs "toNumber(datum.year)" "year"
                << calculateAs "datum['1932'] - datum['1931']" "delta"

        enc =
            encoding
                << position X [ pName "year", pTitle "" ]
                << position Y [ pName "variety", pTitle "Variety" ]
                << size
                    [ mName "yield"
                    , mQuant
                    , mScale [ scRange (raNums [ 0, 12 ]) ]
                    , mLegend
                        [ leTitle "Barley Yield (bushels/acre)"
                        , leValues (nums [ 20, 60 ])
                        ]
                    ]
                << color [ mName "delta", mQuant, mScale [ scDomain (doMid 0) ], mTitle "Yield Delta (%)" ]
                << column [ fName "site", fHeader [ hdTitle "Site" ] ]
                << tooltips [ [ tName "year", tQuant ], [ tName "yield" ] ]
    in
    toVegaLite [ cfg [], data [], trans [], enc [], trail [] ]
```

## Invalid Values

Line chart with markers and invalid (null) values. Here, for demonstration purposes, invalid points are generated explicitly with a [calculateAs](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#calculateAs) transform.

```elm {v}
invalidExample : Spec
invalidExample =
    let
        data =
            dataFromColumns []
                << dataColumn "x" (nums [ 1, 2, 3, 4, 5, 6, 7 ])
                << dataColumn "y" (nums [ 10, 30, -99, 15, -99, 40, 20 ])

        trans =
            transform
                << calculateAs "if(datum.y == -99, null, datum.y)" "yPrime"

        enc =
            encoding
                << position X [ pName "x", pQuant ]
                << position Y [ pName "yPrime", pQuant ]
    in
    toVegaLite [ data [], trans [], enc [], line [ maPoint (pmMarker []) ] ]
```

## Change in rank over time

Ranks are calculated with the [window](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#window) function rather than stored in the data table. World Cup 2018, Group F teams.

```elm {v}
ranks : Spec
ranks =
    let
        data =
            dataFromColumns []
                << dataColumn "team" (strs [ "Germany", "Mexico", "South Korea", "Sweden", "Germany", "Mexico", "South Korea", "Sweden", "Germany", "Mexico", "South Korea", "Sweden" ])
                << dataColumn "matchday" (nums [ 1, 1, 1, 1, 2, 2, 2, 2, 3, 3, 3, 3 ])
                << dataColumn "point" (nums [ 0, 3, 0, 3, 3, 6, 0, 3, 3, 6, 3, 6 ])
                << dataColumn "diff" (nums [ -1, 1, -1, 1, 0, 2, -2, 0, -2, -1, 0, 3 ])

        trans =
            transform
                << window [ ( [ wiOp woRank ], "rank" ) ]
                    [ wiSort [ wiDescending "point", wiDescending "diff" ], wiGroupBy [ "matchday" ] ]

        enc =
            encoding
                << position X [ pName "matchday", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "rank" ]
                << color [ mName "team", mScale teamColours ]

        teamColours =
            categoricalDomainMap
                [ ( "Germany", "black" )
                , ( "Mexico", "#127153" )
                , ( "South Korea", "#c91a3c" )
                , ( "Sweden", "#0c71ab" )
                ]
    in
    toVegaLite [ width 100, height 120, data [], trans [], enc [], line [ maOrient moVertical ] ]
```

## Sequence Generators

While Elm is usually the best choice for programmatically generated data, Vega-Lite also has some generators for creating numeric sequences. Here we use [dataSequenceAs](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#dataSequenceAs) and a [calculateAs transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#calculateAs) to create some sine and cosine waves.

```elm {v}
trigSequence : Spec
trigSequence =
    let
        data =
            dataSequenceAs 0 12.7 0.1 "u"

        trans =
            transform
                << calculateAs "sin(datum.u)" "v"
                << calculateAs "cos(datum.u)" "w"

        encSin =
            encoding
                << position X [ pName "u", pQuant, pTitle "x" ]
                << position Y [ pName "v", pQuant, pTitle "sin(x)" ]

        specSin =
            asSpec [ encSin [], line [] ]

        encCos =
            encoding
                << position X [ pName "u", pQuant, pTitle "x" ]
                << position Y [ pName "w", pQuant, pTitle "cos(x)" ]

        specCos =
            asSpec [ encCos [], line [ maStroke "firebrick" ] ]
    in
    toVegaLite [ width 300, height 150, data, trans [], layer [ specSin, specCos ] ]
```

## Combining solid and dashed lines

By default, the first two categorical dash styles are solid and dashed, which allows us to combine the two styles in the same line. Here we use the approach to symbolise observed and predicted data values.

```elm {v}
observedAndPredicted : Spec
observedAndPredicted =
    let
        data =
            dataFromColumns []
                -- Note the duplicate of the 4th data value to join dash to solid line.
                << dataColumn "a" (strs [ "A", "B", "C", "D", "D", "E", "F" ])
                << dataColumn "b" (nums [ 28, 55, 91, 81, 81, 19, 87 ])
                << dataColumn "predicted" (boos [ False, False, False, False, True, True, True ])

        enc =
            encoding
                << position X [ pName "a", pOrdinal, pAxis [ axTitle "", axLabelAngle 0 ] ]
                << position Y [ pName "b", pQuant, pTitle "" ]
                << strokeDash [ mName "predicted" ]
                << color [ mName "predicted" ]
    in
    toVegaLite [ width 200, data [], enc [], line [] ]
```

## Benchmark Line Chart

Uses the [window](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#window) transform to calculate frames per second values.

```elm {v}
framesPerSecond : Spec
framesPerSecond =
    let
        falconData =
            Json.Encode.list Json.Encode.float [ 16.81999969482422, 19.759998321533203, 16.079999923706055, 19.579999923706055, 16.420000076293945, 16.200000762939453, 16.020000457763672, 15.9399995803833, 16.280000686645508, 16.119998931884766, 16.15999984741211, 16.119998931884766, 16.139999389648438, 16.100000381469727, 16.200000762939453, 16.260000228881836, 19.35999870300293, 19.700000762939453, 15.9399995803833, 19.139999389648438, 16.200000762939453, 16.119998931884766, 19.520000457763672, 19.700000762939453, 16.200000762939453, 20.979999542236328, 16.299999237060547, 16.420000076293945, 16.81999969482422, 16.5, 16.560001373291016, 16.18000030517578, 16.079999923706055, 16.239999771118164, 16.040000915527344, 16.299999237060547, 19.399999618530273, 15.699999809265137, 16.239999771118164, 15.920000076293945, 16.259998321533203, 16.219999313354492, 16.520000457763672, 16.459999084472656, 16.360000610351563, 15.719999313354492, 16.060001373291016, 15.960000991821289, 16.479999542236328, 16.600000381469727, 16.240001678466797, 16.940000534057617, 16.220001220703125, 15.959999084472656, 15.899999618530273, 16.479999542236328, 16.31999969482422, 15.75999927520752, 15.999998092651367, 16.18000030517578, 16.219999313354492, 15.800000190734863, 16.139999389648438, 16.299999237060547, 16.360000610351563, 16.260000228881836, 15.959999084472656, 15.9399995803833, 16.53999900817871, 16.139999389648438, 16.259998321533203, 16.200000762939453, 15.899999618530273, 16.079999923706055, 16.079999923706055, 15.699999809265137, 15.660000801086426, 16.139999389648438, 23.100000381469727, 16.600000381469727, 16.420000076293945, 16.020000457763672, 15.619999885559082, 16.35999870300293, 15.719999313354492, 15.920001029968262, 15.5600004196167, 16.34000015258789, 22.82000160217285, 15.660000801086426, 15.5600004196167, 16, 16, 15.819999694824219, 16.399999618530273, 16.46000099182129, 16.059999465942383, 16.239999771118164, 15.800000190734863, 16.15999984741211, 16.360000610351563, 19.700000762939453, 16.10000228881836, 16.139999389648438, 15.819999694824219, 16.439998626708984, 16.139999389648438, 16.020000457763672, 15.860000610351563, 16.059999465942383, 16.020000457763672, 15.920000076293945, 15.819999694824219, 16.579999923706055, 15.880000114440918, 16.579999923706055, 15.699999809265137, 19.380001068115234, 19.239999771118164, 16, 15.980000495910645, 15.959999084472656, 16.200000762939453, 15.980000495910645, 16.34000015258789, 16.31999969482422, 16.260000228881836, 15.920000076293945, 15.540000915527344, 16.139999389648438, 16.459999084472656, 16.34000015258789, 15.819999694824219, 19.719999313354492, 15.75999927520752, 16.499998092651367, 15.719999313354492, 16.079999923706055, 16.439998626708984, 16.200000762939453, 15.959999084472656, 16, 16.100000381469727, 19.31999969482422, 16.100000381469727, 16.18000030517578, 15.959999084472656, 22.639999389648438, 15.899999618530273, 16.279998779296875, 16.100000381469727, 15.920000076293945, 16.079999923706055, 16.260000228881836, 15.899999618530273, 15.820001602172852, 15.699999809265137, 15.979998588562012, 16.380001068115234, 16.040000915527344, 19.420000076293945, 15.9399995803833, 16.15999984741211, 15.960000991821289, 16.259998321533203, 15.780000686645508, 15.880000114440918, 15.980000495910645, 16.060001373291016, 16.119998931884766, 23.020000457763672, 15.619999885559082, 15.920000076293945, 16.060001373291016, 14.780000686645508, 16.260000228881836, 19.520000457763672, 16.31999969482422, 16.600000381469727, 16.219999313354492, 19.740001678466797, 19.46000099182129, 15.940000534057617, 15.839999198913574, 16.100000381469727, 16.46000099182129, 16.17999839782715, 16.100000381469727, 15.9399995803833, 16.060001373291016, 15.860000610351563, 15.819999694824219, 16.03999900817871, 16.17999839782715, 15.819999694824219, 17.299999237060547, 15.9399995803833, 15.739999771118164, 15.719999313354492, 15.679998397827148, 15.619999885559082, 15.600000381469727, 16.03999900817871, 15.5, 15.600001335144043, 19.439998626708984, 15.960000991821289, 16.239999771118164, 16.040000915527344, 16.239999771118164 ]

        squareData =
            Json.Encode.list Json.Encode.float [ 24.200000762939453, 17.899999618530273, 15.800000190734863, 58.400001525878906, 151, 2523.10009765625, 245.3000030517578, 136, 72.30000305175781, 55.70000076293945, 42.400001525878906, 37.70000076293945, 30.100000381469727, 30.100000381469727, 21.799999237060547, 20.600000381469727, 21.799999237060547, 17.600000381469727, 18.200000762939453, 21, 941.7000122070313, 177.39999389648438, 2821.800048828125, 359.20001220703125, 318, 217.10000610351563, 126, 69, 57.79999923706055, 45.29999923706055, 35.599998474121094, 29.100000381469727, 23.799999237060547, 44.20000076293945, 17.700000762939453, 17.700000762939453, 15.699999809265137, 27.799999237060547, 22.799999237060547, 3853.60009765625, 91.5999984741211, 181.39999389648438, 476.29998779296875, 265.8999938964844, 254.60000610351563, 2583.199951171875, 124.80000305175781, 73.19999694824219, 56.400001525878906, 48.70000076293945, 41.599998474121094, 21.100000381469727, 20.299999237060547, 21.299999237060547, 18.299999237060547, 17.100000381469727, 19.5, 828.2000122070313, 162.1999969482422, 217.89999389648438, 205.5, 197.60000610351563, 2249.800048828125, 103.0999984741211, 71.69999694824219, 57.599998474121094, 41.400001525878906, 34.5, 22, 20.5, 21.700000762939453, 18.299999237060547, 17.299999237060547, 19.399999618530273, 666.7999877929688, 214.89999389648438, 212.3000030517578, 125.80000305175781, 67.69999694824219, 56.099998474121094, 45.79999923706055, 38.29999923706055, 33, 35.400001525878906, 22.700000762939453, 19.399999618530273, 19.899999618530273, 24.100000381469727, 19.299999237060547, 21.299999237060547, 3508.699951171875, 204.10000610351563, 125.4000015258789, 65.30000305175781, 60.79999923706055, 44.099998474121094, 36.29999923706055, 30.5, 28.600000381469727, 16.5, 18.600000381469727, 23.700000762939453, 22.299999237060547, 17.600000381469727, 19.200000762939453, 448.79998779296875, 124.4000015258789, 66.5999984741211, 53.5, 51, 45.20000076293945, 28.399999618530273, 29.200000762939453, 26.700000762939453, 25.899999618530273, 18.100000381469727, 17.600000381469727, 20.100000381469727, 25.200000762939453, 3332, 67.5, 53.599998474121094, 56.599998474121094, 39.900001525878906, 27.600000381469727, 29.600000381469727, 33.5, 17.200000762939453, 18.799999237060547, 25.200000762939453, 16.700000762939453, 16.899999618530273, 240.1999969482422, 52.400001525878906, 42.099998474121094, 33.900001525878906, 28, 28.600000381469727, 17.299999237060547, 20, 21, 22.799999237060547, 16.700000762939453, 19.200000762939453, 175.39999389648438, 43.5, 34.70000076293945, 29.700000762939453, 34.900001525878906, 25.799999237060547, 17.299999237060547, 22.600000381469727, 17.600000381469727, 17.200000762939453, 19.200000762939453, 111.80000305175781, 35.400001525878906, 27.600000381469727, 25.399999618530273, 21.899999618530273, 18.600000381469727, 18.100000381469727, 21.200000762939453, 17.899999618530273, 17, 80.5999984741211, 29.799999237060547, 30.100000381469727, 16, 26.799999237060547, 17.5, 22.299999237060547, 16.799999237060547, 22.399999618530273, 77.4000015258789, 31, 29.700000762939453, 28.700000762939453, 26, 16.899999618530273, 15.800000190734863, 19, 52.599998474121094, 25.200000762939453, 16.700000762939453, 17.899999618530273, 21, 19.799999237060547, 18.799999237060547, 46.5, 17.5, 16.799999237060547, 18.299999237060547, 18.299999237060547, 14.899999618530273, 41, 18.299999237060547, 17.299999237060547, 17, 17.5, 32.29999923706055, 22.600000381469727, 16.600000381469727, 17.899999618530273, 25.600000381469727, 17.5, 20.299999237060547, 25.200000762939453, 18.600000381469727, 17.700000762939453 ]

        transSquare =
            transform
                << window [ ( [ wiOp woRowNumber ], "row" ) ] []
                << calculateAs "1000/datum.data" "fps"
                << calculateAs "'Square Crossfilter (3M)'" "system"

        transFalcon =
            transform
                << window [ ( [ wiOp woRowNumber ], "row" ) ] []
                << calculateAs "1000/datum.data" "fps"
                << calculateAs "'Falcon'" "system"

        enc =
            encoding
                << position X [ pName "row", pQuant, pAxis [ axGrid False, axTitle "Trial" ], pScale [ scNice niFalse ] ]
                << position Y [ pName "fps", pAxis [ axGrid False, axTitle "Frames per Second (fps)" ], pScale [ scType scLog ] ]
                << color [ mName "system", mLegend [ leOrient loBottomRight, leTitle "System" ] ]
                << size [ mNum 1 ]

        specFalcon =
            asSpec [ dataFromSource "falcon" [], transFalcon [], line [] ]

        specSquare =
            asSpec [ dataFromSource "square" [], transSquare [], line [] ]
    in
    toVegaLite
        [ width 400
        , height 200
        , datasets [ ( "falcon", dataFromJson falconData [] ), ( "square", dataFromJson squareData [] ) ]
        , enc []
        , layer [ specFalcon, specSquare ]
        ]
```

## Parallel Coordinates Plot

Data are scaled to a common [0-1] range using [joinAggregate](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#joinAggregate) and [calculateAs](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#calculateAs) transform functions. Axes are drawn manually as layers containing rules and text labels.

```elm {v}
parallelCoords : Spec
parallelCoords =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoDomain False, axcoLabelAngle 0, axcoTickColor "#ccc" ] |> coAxisXFilter)
                << configuration
                    (coMarkStyles
                        [ ( "label", [ maBaseline vaMiddle, maAlign haRight, maDx -5, maTooltip ttNone ] )
                        , ( "tick", [ maOrient moHorizontal, maTooltip ttNone ] )
                        ]
                    )

        data =
            dataFromUrl (vegaPath ++ "penguins.json") []

        trans =
            transform
                -- Remove missing data
                << filter (fiExpr "datum['Beak Length (mm)']")
                -- Give each table row a unique number. This is needed later so we can
                -- draw a line for each 'row' (i.e. sample) in the original data table.
                << window [ ( [ wiAggregateOp opCount ], "index" ) ] []
                -- Gather data so that all measurements are in one column (value) and
                -- a reference to the variable each represents is in another (key).
                << fold [ "Beak Length (mm)", "Beak Depth (mm)", "Flipper Length (mm)", "Body Mass (g)" ]
                -- Find min and max values for each variable (Beak Length, Body Mass etc.
                -- Grouping by key to have a min/max for each variable.
                << joinAggregate [ opAs opMin "value" "min", opAs opMax "value" "max" ] [ wiGroupBy [ "key" ] ]
                --  Calculate a normalised version of each measurement
                << calculateAs "(datum.value - datum.min) / (datum.max-datum.min)" "normVal"
                << calculateAs "(datum.min + datum.max) / 2" "mid"

        encLine =
            encoding
                << position X [ pName "key" ]
                << position Y [ pName "normVal", pQuant, pAxis [] ]
                << color [ mName "Species", mTitle "" ]
                << detail [ dName "index" ]
                << tooltips
                    [ [ tName "Beak Length (mm)", tQuant ]
                    , [ tName "Beak Depth (mm)", tQuant ]
                    , [ tName "Flipper Length (mm)", tQuant ]
                    , [ tName "Body Mass (g)", tQuant ]
                    ]

        specLine =
            asSpec [ encLine [], line [ maOpacity 0.3 ] ]

        encAxis =
            encoding
                << position X [ pName "key", pTitle "" ]
                << detail [ dAggregate opCount ]

        specAxis =
            asSpec [ encAxis [], rule [ maColor "#ccc" ] ]

        encAxisLabelsTop =
            encoding
                << position X [ pName "key" ]
                << position Y [ pNum 0 ]
                << text [ tName "max", tAggregate opMax ]

        specAxisLabelsTop =
            asSpec [ encAxisLabelsTop [], textMark [ maStyle [ "label" ] ] ]

        encAxisLabelsMid =
            encoding
                << position X [ pName "key" ]
                << position Y [ pNum 150 ]
                << text [ tName "mid", tAggregate opMin ]

        specAxisLabelsMid =
            asSpec [ encAxisLabelsMid [], textMark [ maStyle [ "label" ] ] ]

        encAxisLabelsBot =
            encoding
                << position X [ pName "key" ]
                << position Y [ pHeight ]
                << text [ tName "min", tAggregate opMin ]

        specAxisLabelsBot =
            asSpec [ encAxisLabelsBot [], textMark [ maStyle [ "label" ] ] ]
    in
    toVegaLite
        [ cfg []
        , width 600
        , height 300
        , title "Penguin Morphology" [ tiOffset 20 ]
        , data
        , trans []
        , layer [ specAxis, specLine, specAxisLabelsTop, specAxisLabelsMid, specAxisLabelsBot ]
        ]
```
