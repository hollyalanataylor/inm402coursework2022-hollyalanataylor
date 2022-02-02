---
follows: gallery
id: litvis
---

@import "../lectures/css/datavis.less"

_Elm-Vegalite Gallery._

1.  [Scatter and strip plots](scatter.md)
1.  [Bar charts](bars.md)
1.  [Line charts](lines.md)
1.  [Radial charts](radial.md)
1.  [Area charts and streamgraphs](area.md)
1.  [Table-based charts](table.md)
1.  **Distribution charts**
1.  [Labelling, Annotation and Layering](layers.md)
1.  [Interactive charts](interactive.md)
1.  [Interactive linked views](interactiveLinked.md)
1.  [Faceted charts](facet.md)
1.  [Repeats and concatenation](concats.md)
1.  [Geographic maps](geo.md)

---

# Distribution charts

Approaches to visualizing the statistical distributions of data sets.

Examples that use data from external sources tend to use files from the Vega-Lite and giCentre data servers. For consistency the paths to these data locations are defined here:

```elm {l}
vegaPath : String
vegaPath =
    "https://cdn.jsdelivr.net/npm/vega-datasets@2.2/data/"


giCentrePath : String
giCentrePath =
    "https://gicentre.github.io/data/"
```

## Frequency histogram

Frequency histograms (counts of values in ordered categories or 'bins') can be created by binning a variable on one axis and aggregating by counting the number of items in each bin on the other. Here we create a frequency histogram of oxides of nitrogen concentrations (air pollution) for a year of readings for Putney High Street, London.

```elm {v}
freqHisto : Spec
freqHisto =
    let
        data =
            dataFromUrl (giCentrePath ++ "putneyAirQuality2018.csv")
                [ parse [ ( "NOX", foNum ) ] ]

        enc =
            encoding
                << position X
                    [ pName "NOX"
                    , pBin [ biStep 20 ] -- Size of each bin
                    , pTitle "NOX concentration (μg/m3)"
                    ]
                << position Y [ pAggregate opCount ]
    in
    toVegaLite [ width 600, data, enc [], bar [] ]
```

## Relative Frequency

Alternatively we can represent frequency as a proportion of the total even though this has no effect on the shape of the distribution it may provide a more useful scaling of the y-axis. The data are binned in the first transform. The number of values per bin and the total number are calculated in the second and third transforms and the relative frequency in the last transformation step.

```elm {v interactive}
relFreq : Spec
relFreq =
    let
        data =
            dataFromUrl (giCentrePath ++ "putneyAirQuality2018.csv")
                [ parse [ ( "NOX", foNum ) ] ]

        trans =
            transform
                << binAs [ biStep 20 ] "NOX" "bin_NOX"
                << aggregate [ opAs opCount "" "Count" ] [ "bin_NOX", "bin_NOX_end" ]
                << joinAggregate [ opAs opSum "Count" "totalCount" ] []
                << calculateAs "datum.Count/datum.totalCount" "percentOfTotal"

        enc =
            encoding
                << position X [ pName "bin_NOX", pBinned, pTitle "NOX concentration (μg/m3)" ]
                << position X2 [ pName "bin_NOX_end" ]
                << position Y
                    [ pName "percentOfTotal"
                    , pQuant
                    , pTitle "Relative frequency"
                    , pAxis [ axFormat ".1~%" ]
                    ]
    in
    toVegaLite [ width 600, data, trans [], enc [], bar [ maTooltip ttEncoding ] ]
```

## Cumulative Frequency Distribution

The cumulative frequency distribution can be calculated using the [window transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#3-16-window-transformations).

```elm {v}
cumulativeF : Spec
cumulativeF =
    let
        data =
            dataFromUrl (giCentrePath ++ "putneyAirQuality2018.csv")
                [ parse [ ( "NOX", foNum ) ] ]

        trans =
            transform
                << window [ ( [ wiAggregateOp opCount ], "cumulativeCount" ) ]
                    [ wiSort [ wiAscending "NOX" ] ]

        enc =
            encoding
                << position X
                    [ pName "NOX"
                    , pQuant
                    , pTitle "NOX concentration (μg/m3)"
                    , pScale [ scDomain (doNums [ 0, 700 ]) ]
                    ]
                << position Y
                    [ pName "cumulativeCount"
                    , pQuant
                    ]
    in
    toVegaLite [ width 500, data, trans [], enc [], area [ maClip True ] ]
```

## Density Distribution

We can estimate a continuous density distribution from a set of quantitative measurements using the [density (KDE) transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#3-12-density-estimation).

```elm {v}
densityDist : Spec
densityDist =
    let
        data =
            dataFromUrl (giCentrePath ++ "putneyAirQuality2018.csv")
                [ parse [ ( "NOX", foNum ) ] ]

        trans =
            transform
                << density "NOX" [ dnMinSteps 200 ]

        enc =
            encoding
                << position X
                    [ pName "value"
                    , pQuant
                    , pTitle "NOX concentration (μg/m3)"
                    ]
                << position Y [ pName "density", pQuant ]
    in
    toVegaLite [ width 500, data, trans [], enc [], area [] ]
```

## Multiple Density Distributions

We can stack density estimates when comparing distributions in different categories. When stacking densities it is important to specify the extent and number of steps of the density estimation (here between 2,500 and 6,500 grammes body mass and 80 steps) so that all grouped estimates (here penguin species) share the same estimation parameters before being stacked.

```elm {v}
multiDensity : Spec
multiDensity =
    let
        data =
            dataFromUrl (vegaPath ++ "penguins.json") []

        trans =
            transform
                << density "Body Mass (g)"
                    [ dnSteps 80
                    , dnGroupBy [ "Species" ]
                    , dnExtent 2500 6500
                    ]

        enc =
            encoding
                << position X [ pName "value", pQuant, pTitle "Body mass (g)" ]
                << position Y [ pName "density", pQuant, pStack stZero ]
                << color [ mName "Species" ]
    in
    toVegaLite
        [ title "Body Mass of Penguins" []
        , width 400
        , height 120
        , data
        , trans []
        , enc []
        , area [ maStroke "white", maStrokeWidth 0.5 ]
        ]
```

## Tukey boxplot

Boxplots summarise a distribution by showing various summary statistics of it. A [Tukey boxplot](https://en.wikipedia.org/wiki/Box_plot) has whiskers that span from `Q1 - k*IQR` to `Q3 + k*IQR` where `Q1` and `Q3` are the first and third quartiles, `IQR` is the interquartile range (`Q3-Q1`) and `k` is, by default, 1.5. The inner 'box' is centred on the median (`Q2`) with a width of `IQR`. Any outliers beyond the whiskers are shown with [point](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#point) marks.

Here is a Tukey boxplot summarising the distribution of IMDB movie ratings:

```elm {v}
tukeyBoxplot : Spec
tukeyBoxplot =
    let
        data =
            dataFromUrl (vegaPath ++ "movies.json") []

        enc =
            encoding
                << position X [ pName "IMDB Rating", pQuant ]
    in
    toVegaLite [ width 400, data, enc [], boxplot [] ]
```

We can combine boxplots easily in order to compare distributions in different categories. Here, for example, comparison of penguin weights by species.

```elm {v}
tukeyBoxplots : Spec
tukeyBoxplots =
    let
        data =
            dataFromUrl (vegaPath ++ "penguins.json") []

        enc =
            encoding
                << position X [ pName "Body Mass (g)", pQuant, pScale [ scZero False ] ]
                << position Y [ pName "Species", pAxis [ axLabelAngle 0, axTitle "" ] ]
                << color [ mName "Species", mLegend [] ]
    in
    toVegaLite [ width 400, data, enc [], boxplot [] ]
```

## Range boxplot

An alternative to the Tukey boxplot is to extend the whiskers to the minimum and maximum values of the distribution (the inner box remains centred on the median and with a width of the interquartile range):

```elm {v}
rangeBoxplot : Spec
rangeBoxplot =
    let
        data =
            dataFromUrl (vegaPath ++ "movies.json") []

        enc =
            encoding
                << position X [ pName "IMDB Rating", pQuant ]
    in
    toVegaLite [ width 400, data, enc [], boxplot [ maExtent exRange ] ]
```

## Multiple boxplots

Boxplots marks can be used to summarise binned categories. Here we show boxplots of global temperature anomalies for the 12 monthly readings each year since 1960.

```elm {v}
boxplots : Spec
boxplots =
    let
        data =
            dataFromUrl (giCentrePath ++ "temperatureAnomalies.json") []

        trans =
            transform
                << filter (fiExpr "year(datum.Date) >= 1960")

        enc =
            encoding
                << position X
                    [ pName "Date"
                    , pTimeUnit year -- Aggregate dates into yearly groups
                    , pTitle ""
                    ]
                << position Y
                    [ pName "Anomaly"
                    , pAxis []
                    , pQuant
                    ]
    in
    toVegaLite
        [ width 600
        , data
        , trans []
        , enc []
        , boxplot [ maSize 5 ] -- Width of boxes controlled with maSize
        ]
```

## Range Bands

The spread of a distribution can be represented as a range between the minimum and maximum values in an aggregated group (encoded with `Y` and `Y2` position channels). This can be used to generate bands made up of consecutive range pairs.

```elm {v}
rangeBands : Spec
rangeBands =
    let
        data =
            dataFromUrl (giCentrePath ++ "temperatureAnomalies.json") []

        enc =
            encoding
                << position X
                    [ pName "Date"
                    , pTimeUnit year -- Aggregate dates into yearly groups
                    , pTitle ""
                    ]
                << position Y [ pName "Anomaly", pAggregate opMax ]
                << position Y2 [ pName "Anomaly", pAggregate opMin ]
    in
    toVegaLite
        [ width 600
        , data
        , enc []
        , area
            [ maOpacity 0.4 -- Subdue the visual impact of the range
            , maInterpolate miMonotone -- Join with curved boundaries
            ]
        ]
```

## Confidence Bands

Uncertainty around an aggregate value or distribution can be represented as a range by calculating lower and upper confidence intervals. This can be used to generate confidence bands made up of consecutive ranges. Here two layers are used to show both the confidence bands and the individual values that make up the distribution.

```elm {v}
confidenceBands : Spec
confidenceBands =
    let
        data =
            dataFromUrl (giCentrePath ++ "temperatureAnomalies.json") []

        bandEnc =
            encoding
                << position X [ pName "Date", pTimeUnit year, pTitle "" ]
                << position Y [ pName "Anomaly", pAggregate opCI0, pAxis [] ]
                << position Y2 [ pName "Anomaly", pAggregate opCI1 ]

        bandSpec =
            asSpec [ bandEnc [], area [ maOpacity 0.4, maInterpolate miMonotone ] ]

        lineEnc =
            encoding
                << position X [ pName "Date", pTemporal ]
                << position Y [ pName "Anomaly", pQuant ]

        lineSpec =
            asSpec [ lineEnc [], line [ maStrokeWidth 0.2 ] ]
    in
    toVegaLite [ width 600, data, layer [ bandSpec, lineSpec ] ]
```

## Confidence Interval Error Bars

A distribution can be summarised by its mean value and some measure of spread. Here we create error bars by layering a [rule](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#rule) between lower and upper confidence intervals on top of a circle at the mean position.

```elm {v}
ciErrorBars : Spec
ciErrorBars =
    let
        data =
            dataFromUrl (vegaPath ++ "barley.json") []

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        encPoints =
            encoding
                << position X
                    [ pName "yield"
                    , pAggregate opMean -- Find mean yield in each group
                    , pScale [ scZero False ] -- Don't force a zero origin.
                    , pTitle "barley yield"
                    ]
                << position Y [ pName "variety", pOrdinal ]

        specPoints =
            asSpec [ circle [], encPoints [] ]

        encCIs =
            encoding
                << position X [ pName "yield", pAggregate opCI0 ]
                << position X2 [ pName "yield", pAggregate opCI1 ]
                << position Y [ pName "variety" ]

        specCIs =
            asSpec [ rule [], encCIs [] ]
    in
    toVegaLite [ cfg [], data, layer [ specPoints, specCIs ] ]
```

## Standard Deviation Error Bars

Other summary measures like standard deviation have only a single value not an upper and lower bound. To create error bars between standard deviations we need to calculate the lower and upper bounds explicitly with a [transform](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#transform).

```elm {v}
stdevBars : Spec
stdevBars =
    let
        data =
            dataFromUrl (vegaPath ++ "barley.json") []

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        trans =
            transform
                << aggregate
                    [ opAs opMean "yield" "mean"
                    , opAs opStdev "yield" "stdev"
                    ]
                    [ "variety" ]
                << calculateAs "datum.mean-datum.stdev" "lower"
                << calculateAs "datum.mean+datum.stdev" "upper"

        encMeans =
            encoding
                << position X
                    [ pName "mean"
                    , pScale [ scZero False ]
                    , pTitle "Barley Yield"
                    , pQuant
                    ]
                << position Y [ pName "variety" ]

        specMeans =
            asSpec [ encMeans [], circle [] ]

        encStdevs =
            encoding
                << position X [ pName "upper", pQuant ]
                << position X2 [ pName "lower" ]
                << position Y [ pName "variety" ]

        specStdevs =
            asSpec [ rule [], encStdevs [] ]
    in
    toVegaLite [ cfg [], data, trans [], layer [ specMeans, specStdevs ] ]
```

## Q-Q Plots

Suppose we wished to compare the distribution of Internet Movie Database (IMDB) ratings with those of a theoretical normal distribution. We can standardise ratings which are normally on a scale of 1-10 by their [z-score](https://en.wikipedia.org/wiki/Standard_score). This tells us how many standard deviations from the mean score any one movie is.

We can display the distribution of standardised IMDB scores (blue) along with the theoretical normal distribution (black line) to give us a visual comparison.

```elm {v}
denPlots : Spec
denPlots =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        movieData =
            dataFromUrl (vegaPath ++ "movies.json") []

        trans1 =
            transform
                << filter (fiExpr "datum['IMDB Rating'] !== null")
                << joinAggregate [ opAs opMean "IMDB Rating" "mean", opAs opStdev "IMDB Rating" "stdev" ] []
                << calculateAs "(datum['IMDB Rating']-datum.mean)/datum.stdev" "zScore"
                << density "zScore" []

        enc1 =
            encoding
                << position X
                    [ pName "value"
                    , pAxis [ axTitle "Standardised IMDB Rating", axGrid False ]
                    , pQuant
                    ]
                << position Y [ pName "density", pQuant, pAxis [] ]

        spec1 =
            asSpec
                [ movieData
                , trans1 []
                , enc1 []
                , area [ maOpacity 0.5 ]
                ]

        seqData =
            dataSequenceAs -4 4 0.05 "p"

        transNormal =
            transform
                << calculateAs "densityNormal(datum.p)" "norm"

        encNormal =
            encoding
                << position X [ pName "p", pQuant ]
                << position Y [ pName "norm", pQuant, pTitle "" ]

        specNormal =
            asSpec
                [ seqData
                , transNormal []
                , encNormal []
                , area [ maFilled False, maStroke "black" ]
                ]
    in
    toVegaLite [ cfg [], width 300, layer [ spec1, specNormal ] ]
```

A [Quantile-Quantile plot](https://en.wikipedia.org/wiki/Q–Q_plot) or Q-Q plot allows us to compare two probability distributions more precisely by counting the number of values in increasingly larger quantile bands. The rate of change of these counts is sensitive to the nature of the distribution so plotting one against another provides a visual summary of how two distributions differ from one another.

The grey line shows what we would expect if the IMDB rating was a perfect normal distribution. Deviation in this example shows as that the steeper line of dots to the left of the distribution (low IMDB rating) are more dispersed than we expect, and the shallower line at the right tells us that higher IMDB ratings are more compact than we would expect.

```elm {v}
qqPlot : Spec
qqPlot =
    let
        data =
            dataFromUrl (vegaPath ++ "movies.json") []

        trans =
            transform
                << filter (fiExpr "datum['IMDB Rating'] !== null")
                << joinAggregate [ opAs opMean "IMDB Rating" "mean", opAs opStdev "IMDB Rating" "stdev" ] []
                << calculateAs "(datum['IMDB Rating']-datum.mean)/datum.stdev" "zScore"
                << quantile "zScore" [ qtStep 0.01 ]
                << calculateAs "quantileNormal(datum.prob)" "norm"

        encQQ =
            encoding
                << position X [ pName "norm", pQuant, pTitle "Expected Normal" ]
                << position Y [ pName "value", pQuant, pTitle "Observed IMDB" ]

        circleSpec =
            asSpec [ data, trans [], encQQ [], circle [] ]

        lineData =
            dataFromColumns []
                << dataColumn "p" (nums [ -3, 3 ])

        encLine =
            encoding
                << position X [ pName "p", pQuant ]
                << position Y [ pName "p", pQuant ]

        lineSpec =
            asSpec [ lineData [], encLine [], line [ maColor "black", maOpacity 0.3 ] ]
    in
    toVegaLite [ width 300, height 300, layer [ circleSpec, lineSpec ] ]
```
