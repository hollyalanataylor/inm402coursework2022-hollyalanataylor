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
1.  [Distribution charts](distrib.md)
1.  **Labelling, Annotation and Layering**
1.  [Interactive charts](interactive.md)
1.  [Interactive linked views](interactiveLinked.md)
1.  [Faceted charts](facet.md)
1.  [Repeats and concatenation](concats.md)
1.  [Geographic maps](geo.md)

---

# Labelling, Annotation and Layering

Examples that use data from external sources tend to use files from the Vega-Lite data server. For consistency the path to the data location is defined here:

```elm {l}
path : String
path =
    "https://cdn.jsdelivr.net/npm/vega-datasets@2.2/data/"
```

## Labelled bars

Text annotations can be added by creating a separate layer containing text marks. Because bar position and text position match (subject to text alignment), we only need to specify the position encoding (`enc`) once.

```elm {v}
barLabels : Spec
barLabels =
    let
        data =
            dataFromColumns []
                << dataColumn "pet" (strs [ "Cats", "Dogs", "Guinea pigs" ])
                << dataColumn "value" (nums [ 28, 55, 43 ])

        enc =
            encoding
                << position X [ pName "value", pQuant, pTitle "" ]
                << position Y [ pName "pet", pTitle "" ]

        encText =
            encoding
                << text [ tName "value" ]

        specText =
            asSpec
                [ textMark
                    [ maAlign haRight -- Align text right
                    , maBaseline vaMiddle -- Centre vertical alignment in each bar
                    , maDx -3 -- Shift left 3 pixels
                    , maColor "white" -- Contrast with bar colour
                    ]
                , encText []
                ]
    in
    toVegaLite [ data [], enc [], layer [ asSpec [ bar [] ], specText ] ]
```

## Compact Labelled bars

Like the previous example, but we can move the y-axis labels into the bar area and drop the horizontal axis to create a more compact representation.

```elm {v}
compactBarLabels : Spec
compactBarLabels =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        data =
            dataFromColumns []
                << dataColumn "pet" (strs [ "Cats", "Dogs", "Guinea pigs" ])
                << dataColumn "value" (nums [ 28, 55, 43 ])

        enc =
            encoding
                << position X [ pName "value", pQuant, pAxis [] ]
                << position Y
                    [ pName "pet"
                    , pTitle ""
                    , pAxis
                        [ axLabelAlign haLeft -- Align labels on left side
                        , axZIndex 1 -- Ensure labels on top of bars
                        , axLabelColor "white" -- White text against dark bars
                        , axLabelPadding -3 -- Shift labels right a little
                        , axTicks False -- No need for tick marks
                        ]
                    ]

        encText =
            encoding
                << text [ tName "value", tQuant ]

        specText =
            asSpec
                [ textMark
                    [ maAlign haRight -- Align text right
                    , maBaseline vaMiddle -- Centre vertical alignment in each bar
                    , maDx -3 -- Shift left 3 pixels
                    , maColor "white" -- Contrast with bar colour
                    ]
                , encText []
                ]
    in
    toVegaLite [ cfg [], data [], enc [], layer [ asSpec [ bar [] ], specText ] ]
```

## Conditional Labels

Label colour can be made dependent on a data value so that we can ensure white text on a dark background and black text on a light background. Notice also the use of emoji for category labels.

```elm {v}
conditionalLabel : Spec
conditionalLabel =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        data =
            dataFromColumns []
                << dataColumn "fruit" (strs [ "🍊", "🍇", "🍏", "🍌", "🍐", "🍋", "🍎", "🍉" ])
                << dataColumn "count" (nums [ 21, 13, 8, 5, 3, 2, 1, 1 ])

        enc =
            encoding
                << position X
                    [ pName "count"
                    , pQuant
                    , pAxis [ axGrid False, axTitle "" ]
                    ]
                << position Y
                    [ pName "fruit"
                    , pOrdinal
                    , pSort [ soByChannel chX, soDescending ]
                    , pAxis [ axTitle "", axTicks False ]
                    ]

        barEnc =
            encoding
                << color [ mName "count", mQuant, mLegend [] ]

        specBar =
            asSpec [ barEnc [], bar [] ]

        labelEnc =
            encoding
                << text [ tName "count", tQuant ]
                << color
                    [ mCondition (prTest (expr "datum.count > 10"))
                        [ mStr "white" ]
                        [ mStr "black" ]
                    ]

        specText =
            asSpec [ labelEnc [], textMark [ maAlign haRight, maXOffset -4 ] ]
    in
    toVegaLite [ cfg [], width 400, data [], enc [], layer [ specBar, specText ] ]
```

## Overlaid mean

Monthly precipitation in Seattle with the annual mean precipitation overlaid using a [rule](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#rule) mark. This is created by specifying two [layers](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#layer) – one for the bars and one for the summary line.

```elm {v}
meanOverlay : Spec
meanOverlay =
    let
        data =
            dataFromUrl (path ++ "seattle-weather.csv") []

        encBar =
            encoding
                << position X
                    [ pName "date"
                    , pTimeUnit month
                    , pAxis [ axTitle "", axLabelAngle 0 ]
                    , pOrdinal
                    ]
                << position Y [ pName "precipitation", pAggregate opMean ]

        specBar =
            asSpec [ encBar [], bar [] ]

        encLine =
            encoding
                << position Y [ pName "precipitation", pAggregate opMean ]

        specLine =
            asSpec [ encLine [], rule [ maSize 2, maColor "hsl(0,30%,50%)" ] ]
    in
    toVegaLite [ width 300, data, layer [ specBar, specLine ] ]
```

## Threshold overlay

Bar chart that highlights values beyond a threshold. The [PM2.5 value](https://laqm.defra.gov.uk/public-health/pm25.html) (a measure of air pollution) for Beijing observed over 15 days, highlighting the days when PM2.5 level is hazardous to human health.

This makes use of multiple layers. A layer containing the blue bars is combined with one containing just the red bar segments that exceed the threshold (together they are `layer0` below). Further layers with the threshold line and the text annotation are combined as `layer1`. Both `layer0` and `layer1` are combined in the final specification.

```elm {v}
thresholdBars : Spec
thresholdBars =
    let
        data =
            dataFromColumns []
                << dataColumn "day" (nums (List.map toFloat (List.range 1 15)))
                << dataColumn "pm25" (nums [ 54.8, 112.1, 63.6, 37.6, 79.7, 137.9, 120.1, 103.3, 394.8, 199.5, 72.3, 51.1, 112.0, 174.5, 130.5 ])

        thresholdData =
            dataFromRows []
                << dataRow [ ( "ThresholdValue", num 300 ), ( "Threshold", str "hazardous" ) ]

        encBar =
            encoding
                << position X [ pName "day", pAxis [ axLabelAngle 0 ], pOrdinal ]
                << position Y [ pName "pm25", pQuant ]

        specBar =
            asSpec [ encBar [], bar [] ]

        trans =
            transform
                << filter (fiExpr "datum.pm25 >= 300")
                << calculateAs "300" "baseline"

        encUpperBar =
            encoding
                -- Bars that exceed threshold have a bottom (baseline) and top (pm25) position
                << position X [ pName "day", pAxis [ axLabelAngle 0 ] ]
                << position Y [ pName "baseline", pQuant ]
                << position Y2 [ pName "pm25" ]

        specUpperBar =
            asSpec [ trans [], encUpperBar [], bar [ maColor "hsl(0,40%,50%)" ] ]

        layer0 =
            asSpec [ data [], layer [ specBar, specUpperBar ] ]

        specRule =
            asSpec [ rule [], encRule [] ]

        encRule =
            encoding
                << position Y [ pName "ThresholdValue", pQuant ]

        encText =
            encoding
                << position X [ pWidth ]
                << position Y [ pName "ThresholdValue", pTitle "PM2.5 concentration", pQuant ]
                << text [ tName "Threshold" ]

        specText =
            asSpec [ encText [], textMark [ maAlign haRight, maDx -2, maDy -6 ] ]

        layer1 =
            asSpec [ thresholdData [], layer [ specRule, specText ] ]
    in
    toVegaLite [ layer [ layer0, layer1 ] ]
```

## Labelled lines

Atmospheric carbon dioxide by decade. As we are positioning each decade line according to its carbon dioxide values, adding a text label for each allows us to identify which line corresponds to which decade.

```elm {v}
label3 : Spec
label3 =
    let
        data =
            dataFromUrl (path ++ "co2-concentration.csv") []

        trans =
            transform
                << calculateAs "year(datum.Date)" "year"
                << calculateAs "month(datum.Date)" "month"
                << calculateAs "floor(datum.year / 10)" "decade"
                << calculateAs "(datum.year % 10) + (datum.month / 12)" "scaledDate"

        encPosition =
            encoding
                << position X
                    [ pName "scaledDate"
                    , pAxis [ axTitle "Year into decade", axTickCount (niTickCount 10) ]
                    , pQuant
                    ]
                << position Y
                    [ pName "CO2"
                    , pScale [ scZero False ]
                    , pTitle "CO2 concentration in ppm"
                    , pQuant
                    ]

        encLine =
            encoding
                << detail [ dName "decade" ]

        specLine =
            asSpec [ encLine [], line [ maOrient moVertical ] ]

        transText =
            transform
                << aggregate [ opAs (opArgMin Nothing) "scaledDate" "aggregated" ] [ "decade" ]
                << calculateAs "datum.aggregated.scaledDate" "scaledDate"
                << calculateAs "datum.aggregated.CO2" "CO2"

        encText =
            encoding
                << text [ tName "aggregated.year" ]

        specText =
            asSpec
                [ transText []
                , encText []
                , textMark [ maSize 8, maAlign haLeft, maBaseline vaTop, maDx 2, maDy 3 ]
                ]
    in
    toVegaLite
        [ width 600
        , height 300
        , data
        , trans []
        , encPosition []
        , layer [ specLine, specText ]
        ]
```

## Annotated time periods

The population of the German city of Falkensee over time with annotated time periods highlighted.

```elm {v}
falkensee : Spec
falkensee =
    let
        data =
            dataFromColumns []
                << dataColumn "year" (strs [ "1875", "1890", "1910", "1925", "1933", "1939", "1946", "1950", "1964", "1971", "1981", "1985", "1989", "1990", "1991", "1992", "1993", "1994", "1995", "1996", "1997", "1998", "1999", "2000", "2001", "2002", "2003", "2004", "2005", "2006", "2007", "2008", "2009", "2010", "2011", "2012", "2013", "2014" ])
                << dataColumn "population" (nums [ 1309, 1558, 4512, 8180, 15915, 24824, 28275, 29189, 29881, 26007, 24029, 23340, 22307, 22087, 22139, 22105, 22242, 22801, 24273, 25640, 27393, 29505, 32124, 33791, 35297, 36179, 36829, 37493, 38376, 39008, 39366, 39821, 40179, 40511, 40465, 40905, 41258, 41777 ])

        highlightData =
            dataFromColumns []
                << dataColumn "start" (strs [ "1933", "1948" ])
                << dataColumn "end" (strs [ "1945", "1989" ])
                << dataColumn "event" (strs [ "Nazi Rule", "GDR (East Germany)" ])

        encRects =
            encoding
                << position X [ pName "start", pTimeUnit year ]
                << position X2 [ pName "end", pTimeUnit year ]
                << color [ mName "event" ]

        specRects =
            asSpec [ highlightData [], encRects [], rect [ maOpacity 0.5 ] ]

        encPopulation =
            encoding
                << position X [ pName "year", pTimeUnit year, pAxis [ axTitle "", axGrid False ] ]
                << position Y [ pName "population", pQuant ]

        specLine =
            asSpec
                [ encPopulation []
                , line
                    [ maColor "#666"
                    , maInterpolate miMonotone
                    , maPoint (pmMarker [ maColor "#666" ])
                    ]
                ]
    in
    toVegaLite [ width 500, data [], layer [ specRects, specLine ] ]
```

## Likert Questionnaire Chart

Distributions and medians of Likert Scale Ratings from Figure 9 of [Interactive Repair of Tables Extracted from PDF Documents on Mobile Devices](http://idl.cs.washington.edu/files/2019-InteractiveTableRepair-CHI.pdf).

This example uses inline data, but any Likert-scale data source can be shown in this way.

```elm {v}
likert : Spec
likert =
    let
        medians =
            dataFromColumns []
                << dataColumn "name" (strs [ "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:" ])
                << dataColumn "median" (nums [ 1.999976, 2, 1.999969, 2.500045, 1.500022, 2.99998, 4.500007 ])
                << dataColumn "lo" (strs [ "Easy", "Easy", "Toolbar", "Toolbar", "Toolbar", "Toolbar", "Phone" ])
                << dataColumn "hi" (strs [ "Hard", "Hard", "Gesture", "Gesture", "Gesture", "Gesture", "Tablet" ])

        values =
            dataFromColumns []
                << dataColumn "value" (strs [ "P1", "2", "2", "3", "4", "2", "5", "5", "1", "1", "P2", "2", "3", "4", "5", "5", "5", "5", "1", "1", "P3", "2", "2", "2", "1", "2", "1", "5", "1", "0", "P4", "3", "3", "2", "2", "4", "1", "5", "1", "0", "P5", "2", "2", "4", "4", "4", "5", "5", "0", "1", "P6", "1", "3", "3", "4", "4", "4", "4", "0", "1", "P7", "2", "3", "4", "5", "3", "2", "4", "0", "0", "P8", "3", "1", "2", "4", "2", "5", "5", "0", "0", "P9", "2", "3", "2", "4", "1", "4", "4", "1", "1", "P10", "2", "2", "1", "1", "1", "1", "5", "1", "1", "P11", "2", "2", "1", "1", "1", "1", "4", "1", "0", "P12", "1", "3", "2", "3", "1", "3", "3", "0", "1", "P13", "2", "2", "1", "1", "1", "1", "5", "0", "0", "P14", "3", "3", "2", "2", "1", "1", "1", "1", "1", "P15", "4", "5", "1", "1", "1", "1", "5", "1", "0", "P16", "1", "3", "2", "2", "1", "4", "5", "0", "1", "P17", "3", "2", "2", "2", "1", "3", "2", "0", "0" ])
                << dataColumn "name" (strs [ "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First", "Participant ID", "Identify Errors:", "Fix Errors:", "Easier to Fix:", "Faster to Fix:", "Easier on Phone:", "Easier on Tablet:", "Device Preference:", "Tablet_First", "Toolbar_First" ])
                << dataColumn "id" (strs [ "P1", "P1", "P1", "P1", "P1", "P1", "P1", "P1", "P1", "P1", "P2", "P2", "P2", "P2", "P2", "P2", "P2", "P2", "P2", "P2", "P3", "P3", "P3", "P3", "P3", "P3", "P3", "P3", "P3", "P3", "P4", "P4", "P4", "P4", "P4", "P4", "P4", "P4", "P4", "P4", "P5", "P5", "P5", "P5", "P5", "P5", "P5", "P5", "P5", "P5", "P6", "P6", "P6", "P6", "P6", "P6", "P6", "P6", "P6", "P6", "P7", "P7", "P7", "P7", "P7", "P7", "P7", "P7", "P7", "P7", "P8", "P8", "P8", "P8", "P8", "P8", "P8", "P8", "P8", "P8", "P9", "P9", "P9", "P9", "P9", "P9", "P9", "P9", "P9", "P9", "P10", "P10", "P10", "P10", "P10", "P10", "P10", "P10", "P10", "P10", "P11", "P11", "P11", "P11", "P11", "P11", "P11", "P11", "P11", "P11", "P12", "P12", "P12", "P12", "P12", "P12", "P12", "P12", "P12", "P12", "P13", "P13", "P13", "P13", "P13", "P13", "P13", "P13", "P13", "P13", "P14", "P14", "P14", "P14", "P14", "P14", "P14", "P14", "P14", "P14", "P15", "P15", "P15", "P15", "P15", "P15", "P15", "P15", "P15", "P15", "P16", "P16", "P16", "P16", "P16", "P16", "P16", "P16", "P16", "P16", "P17", "P17", "P17", "P17", "P17", "P17", "P17", "P17", "P17", "P17" ])

        enc =
            encoding
                << position Y
                    [ pName "name"
                    , pSort []
                    , pAxis
                        [ axDomain False
                        , axOffset 50
                        , axLabelFontWeight fwBold
                        , axTicks False
                        , axGrid True
                        , axTitle ""
                        ]
                    ]

        trans =
            transform
                << filter (fiExpr "datum.name != 'Toolbar_First'")
                << filter (fiExpr "datum.name != 'Tablet_First'")
                << filter (fiExpr "datum.name != 'Participant ID'")

        encCircle =
            encoding
                << position X
                    [ pName "value"
                    , pQuant
                    , pScale [ scDomain (doNums [ 0, 6 ]) ]
                    , pAxis [ axGrid False, axValues (nums [ 1, 2, 3, 4, 5 ]) ]
                    ]
                << size [ mAggregate opCount, mLegend [ leTitle "Number of Ratings", leOffset 75 ] ]

        specCircle =
            asSpec
                [ dataFromSource "values" []
                , trans []
                , encCircle []
                , circle [ maColor "#6eb4fd" ]
                ]

        encTick1 =
            encoding
                << position X
                    [ pName "median"
                    , pQuant
                    , pScale [ scDomain (doNums [ 0, 6 ]) ]
                    , pTitle ""
                    ]

        specTick1 =
            asSpec [ encTick1 [], tick [ maColor "black" ] ]

        encTextLo =
            encoding
                << text [ tName "lo" ]

        specTextLo =
            asSpec [ encTextLo [], textMark [ maX -5, maAlign haRight ] ]

        encTextHi =
            encoding
                << text [ tName "hi" ]

        specTextHi =
            asSpec [ encTextHi [], textMark [ maX 255, maAlign haLeft ] ]

        cfg =
            configure << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite
        [ cfg []
        , width 250
        , height 175
        , datasets [ ( "medians", medians [] ), ( "values", values [] ) ]
        , dataFromSource "medians" []
        , title "Questionnaire Ratings" []
        , enc []
        , layer [ specCircle, specTick1, specTextLo, specTextHi ]
        ]
```

## Likert Summary Chart

Comparison of Likert scale ratings between two conditions. From Figure 11 in [Wongsuphasawat et al (2017)](http://idl.cs.washington.edu/files/2017-Voyager2-CHI.pdf).

```elm {v}
likertSummary : Spec
likertSummary =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coMarkStyles [ ( "arrow-label", [ maDy 12, maFontSize 9.5 ] ) ])
                << configuration (coTitle [ ticoFontSize 12 ])

        likertData =
            dataFromColumns []
                << dataColumn "measure" (strs [ "Open Exploration", "Focused Question Answering", "Open Exploration", "Focused Question Answering" ])
                << dataColumn "mean" (nums [ 1.81, -1.69, 2.19, -0.06 ])
                << dataColumn "lo" (nums [ 1.26, -2.33, 1.67, -0.47 ])
                << dataColumn "hi" (nums [ 2.37, -1.05, 2.71, 0.35 ])
                << dataColumn "study" (strs [ "PoleStar vs Voyager", "PoleStar vs Voyager", "PoleStar vs Voyager 2", "PoleStar vs Voyager 2" ])

        labelData =
            dataFromColumns []
                << dataColumn "from" (nums [ -0.25, 0.25 ])
                << dataColumn "to" (nums [ -2.9, 2.9 ])
                << dataColumn "label" (strs [ "PoleStar", "Voyager / Voyager 2" ])

        encLikert =
            encoding
                << position Y
                    [ pName "study"
                    , pAxis [ axTitle "", axLabelPadding 5, axDomain False, axTicks False, axGrid False ]
                    ]

        encLikertWhiskers =
            encoding
                << position X
                    [ pName "lo"
                    , pQuant
                    , pScale [ scDomain (doNums [ -3, 3 ]) ]
                    , pAxis
                        [ axTitle ""
                        , axGridDash [ 3, 3 ]
                        , axDataCondition (expr "datum.value === 0") (cAxGridColor "#666" "#ccc")
                        ]
                    ]
                << position X2 [ pName "hi" ]

        specLikertWhiskers =
            asSpec [ encLikertWhiskers [], rule [] ]

        encLikertMeans =
            encoding
                << position X [ pName "mean", pQuant ]
                << color [ mName "measure", mScale [ scRange (raStrs [ "black", "white" ]) ], mLegend [] ]

        specLikertMeans =
            asSpec [ encLikertMeans [], circle [ maStroke "black", maOpacity 1 ] ]

        specLikert =
            asSpec
                [ title "Mean of Subject Ratings (95% CIs)" [ tiFrame tfBounds ]
                , encLikert []
                , layer [ specLikertWhiskers, specLikertMeans ]
                ]

        encLabel1 =
            encoding
                << position X [ pName "from", pQuant, pScale [ scZero False ], pAxis [] ]
                << position X2 [ pName "to" ]

        specLabel1 =
            asSpec [ encLabel1 [], rule [] ]

        encLabel2 =
            encoding
                << position X [ pName "to", pQuant, pAxis [] ]
                << shape
                    [ mCondition (prTest (expr "datum.to > 0"))
                        [ mSymbol symTriangleRight ]
                        [ mSymbol symTriangleLeft ]
                    ]

        specLabel2 =
            asSpec [ encLabel2 [], point [ maFilled True, maSize 60, maFill "black" ] ]

        transLabel3 =
            transform
                << filter (fiExpr "datum.label === 'PoleStar'")

        encLabel3 =
            encoding
                << position X [ pName "from", pQuant, pAxis [] ]

        specLabel3 =
            asSpec
                [ transLabel3 []
                , encLabel3 []
                , textMark
                    [ maText "PoleStar\nmore valuable"
                    , maAlign haRight
                    , maStyle [ "arrow-label" ]
                    ]
                ]

        transLabel4 =
            transform
                << filter (fiExpr "datum.label !== 'PoleStar'")

        encLabel4 =
            encoding
                << position X [ pName "from", pQuant, pAxis [] ]

        specLabel4 =
            asSpec
                [ transLabel4 []
                , encLabel4 []
                , textMark
                    [ maText "Voyager / Voyager 2\nmore valuable"
                    , maAlign haLeft
                    , maStyle [ "arrow-label" ]
                    ]
                ]

        specLabels =
            asSpec [ labelData [], layer [ specLabel1, specLabel2, specLabel3, specLabel4 ] ]
    in
    toVegaLite [ cfg [], likertData [], spacing 10, vConcat [ specLikert, specLabels ] ]
```

## Candlestick chart

Like box and whisker plots, so-called "candlestick charts" can show four related values as a single glyph. The box and whisker glyphs are created by overlaying two layers – one for the whiskers using a [rule](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#rule) mark and the other for the boxes using the [bar](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#bar) mark.

This example inspired by the one [originally specified with Protovis](http://mbostock.github.io/protovis/ex/candlestick.html) shows daily June summaries of a stock price with colour indicating whether the stock gained (blue) or lost (orange) price during each day.

```elm {v}
candlestick : Spec
candlestick =
    let
        table =
            """date,open,high,low,close
                01-Jun-2009,28.70,30.05,28.45,30.04
                02-Jun-2009,30.04,30.13,28.30,29.63
                03-Jun-2009,29.62,31.79,29.62,31.02
                04-Jun-2009,31.02,31.02,29.92,30.18
                05-Jun-2009,29.39,30.81,28.85,29.62
                08-Jun-2009,30.84,31.82,26.41,29.77
                09-Jun-2009,29.77,29.77,27.79,28.27
                10-Jun-2009,26.90,29.74,26.90,28.46
                11-Jun-2009,27.36,28.11,26.81,28.11
                12-Jun-2009,28.08,28.50,27.73,28.15
                15-Jun-2009,29.70,31.09,29.64,30.81
                16-Jun-2009,30.81,32.75,30.07,32.68
                17-Jun-2009,31.19,32.77,30.64,31.54
                18-Jun-2009,31.54,31.54,29.60,30.03
                19-Jun-2009,29.16,29.32,27.56,27.99
                22-Jun-2009,30.40,32.05,30.30,31.17
                23-Jun-2009,31.30,31.54,27.83,30.58
                24-Jun-2009,30.58,30.58,28.79,29.05
                25-Jun-2009,29.45,29.56,26.30,26.36
                26-Jun-2009,27.09,27.22,25.76,25.93
                29-Jun-2009,25.93,27.18,25.29,25.35
                30-Jun-2009,25.36,27.38,25.02,26.35"""
                |> fromCSV

        data =
            dataFromColumns []
                << dataColumn "date" (strColumn "date" table |> strs)
                << dataColumn "open" (numColumn "open" table |> nums)
                << dataColumn "high" (numColumn "high" table |> nums)
                << dataColumn "low" (numColumn "low" table |> nums)
                << dataColumn "close" (numColumn "close" table |> nums)

        enc =
            encoding
                << position X [ pName "date", pTemporal, pTitle "" ]
                << color
                    [ mCondition (prTest (expr "datum.open < datum.close"))
                        [ mStr "orange" ]
                        [ mStr "steelBlue" ]
                    ]

        encRule =
            encoding
                << position Y [ pName "low", pQuant, pScale [ scZero False ] ]
                << position Y2 [ pName "high" ]

        specRule =
            asSpec [ encRule [], rule [] ]

        encBar =
            encoding
                << position Y [ pName "open", pQuant ]
                << position Y2 [ pName "close" ]

        specBar =
            asSpec [ encBar [], bar [ maSize 8 ] ]
    in
    toVegaLite [ width 400, data [], enc [], layer [ specRule, specBar ] ]
```

## Waterfall Chart

Shows monthly profit and loss over time. Calculates the difference between adjacent values using the window operation [woLead](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#woLead).

```elm {v}
waterfall : Spec
waterfall =
    let
        data =
            dataFromColumns []
                << dataColumn "label" (strs [ "Begin", "Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec", "End" ])
                << dataColumn "amount" (nums [ 4000, 1707, -1425, -1030, 1812, -1067, -1481, 1228, 1176, 1146, 1205, -1388, 1492, 0 ])

        trans =
            transform
                << window [ ( [ wiAggregateOp opSum, wiField "amount" ], "sum" ) ] []
                << window [ ( [ wiOp woLead, wiField "label" ], "lead" ) ] []
                << calculateAs "datum.lead === null ? datum.label : datum.lead" "lead"
                << calculateAs "datum.label === 'End' ? 0 : datum.sum - datum.amount" "previous_sum"
                << calculateAs "datum.label === 'End' ? datum.sum : datum.amount" "amount"
                << calculateAs "(datum.label !== 'Begin' && datum.label !== 'End' && datum.amount > 0 ? '+' : '') + datum.amount" "text_amount"
                << calculateAs "(datum.sum + datum.previous_sum) / 2" "center"
                << calculateAs "datum.sum < datum.previous_sum ? datum.sum : ''" "sum_dec"
                << calculateAs "datum.sum > datum.previous_sum ? datum.sum : ''" "sum_inc"

        enc =
            encoding
                << position X [ pName "label", pOrdinal, pSort [], pTitle "Months" ]

        enc1 =
            encoding
                << position Y [ pName "previous_sum", pQuant, pTitle "Amount" ]
                << position Y2 [ pName "sum" ]
                << color
                    [ mConditions
                        [ ( prTest (expr "datum.label === 'Begin' || datum.label === 'End'"), [ mStr "#f7e0b6" ] )
                        , ( prTest (expr "datum.sum < datum.previous_sum"), [ mStr "#f78a64" ] )
                        ]
                        [ mStr "#93c4aa" ]
                    ]

        spec1 =
            asSpec [ enc1 [], bar [ maSize 45 ] ]

        enc2 =
            encoding
                << position X2 [ pName "lead" ]
                << position Y [ pName "sum", pQuant ]

        spec2 =
            asSpec
                [ enc2 []
                , rule
                    [ maColor "#404040"
                    , maOpacity 1
                    , maStrokeWidth 2
                    , maXOffset -22.5
                    , maX2Offset 22.5
                    ]
                ]

        enc3 =
            encoding
                << position Y [ pName "sum_inc", pQuant ]
                << text [ tName "sum_inc" ]

        spec3 =
            asSpec
                [ enc3 []
                , textMark
                    [ maDy -8
                    , maFontWeight fwBold
                    , maColor "#404040"
                    ]
                ]

        enc4 =
            encoding
                << position Y [ pName "sum_dec", pQuant ]
                << text [ tName "sum_dec" ]

        spec4 =
            asSpec
                [ enc4 []
                , textMark
                    [ maDy 8
                    , maBaseline vaTop
                    , maFontWeight fwBold
                    , maColor "#404040"
                    ]
                ]

        enc5 =
            encoding
                << position Y [ pName "center", pQuant ]
                << text [ tName "text_amount" ]
                << color
                    [ mCondition (prTest (expr "datum.label === 'Begin' || datum.label === 'End'"))
                        [ mStr "#725a30" ]
                        [ mStr "white" ]
                    ]

        spec5 =
            asSpec [ enc5 [], textMark [ maBaseline vaMiddle, maFontWeight fwBold ] ]
    in
    toVegaLite
        [ width 800
        , height 450
        , data []
        , trans []
        , enc []
        , layer [ spec1, spec2, spec3, spec4, spec5 ]
        ]
```

## Ranged dot plot

A ranged dot plot that uses 'layer' to convey changing life expectancy for the five most populous countries (between 1955 and 2000).

```elm {v}
rangedDotPlot : Spec
rangedDotPlot =
    let
        data =
            dataFromUrl (path ++ "countries.json") []

        trans =
            transform
                << filter (fiOneOf "country" (strs [ "China", "India", "United States", "Indonesia", "Brazil" ]))
                << filter (fiOneOf "year" (nums [ 1955, 2000 ]))

        encCountry =
            encoding
                << position Y
                    [ pName "country"
                    , pAxis [ axTitle "Country", axOffset 5, axTicks False, axMinExtent 70, axDomain False ]
                    ]

        encLine =
            encoding
                << position X [ pName "life_expect", pQuant ]
                << detail [ dName "country" ]

        specLine =
            asSpec [ line [ maColor "#db646f" ], encLine [] ]

        encPoints =
            encoding
                << position X
                    [ pName "life_expect"
                    , pQuant
                    , pTitle "Life Expectancy (years)"
                    ]
                << color
                    [ mName "year"
                    , mScale (domainRangeMap ( 1955, "#e6959c" ) ( 2000, "#911a24" ))
                    , mTitle "Year"
                    ]

        specPoints =
            asSpec [ encPoints [], point [ maFilled True, maOpacity 1, maSize 100 ] ]
    in
    toVegaLite [ data, trans [], encCountry [], layer [ specLine, specPoints ] ]
```

## Bullet Chart

Used typically to display performance data, bullet charts function like bar charts, but are accompanied by extra visual elements for context. Originally developed by Stephen Few as a more space efficient alternative to dashboard gauges and meters. This example based on the [dataviz catalogue example](https://datavizcatalogue.com/methods/bullet_graph.html).

```elm {v}
layer3 : Spec
layer3 =
    let
        cfg =
            configure << configuration (coTick [ maThickness 2 ])

        row title ranges measures marker =
            Json.Encode.object
                [ ( "title", Json.Encode.string title )
                , ( "ranges", Json.Encode.list Json.Encode.float ranges )
                , ( "measures", Json.Encode.list Json.Encode.float measures )
                , ( "markers", Json.Encode.list Json.Encode.float [ marker ] )
                ]

        data =
            dataFromJson
                (Json.Encode.list identity
                    [ row "Revenue" [ 150, 225, 300 ] [ 220, 270 ] 250
                    , row "Profit" [ 20, 25, 30 ] [ 21, 23 ] 26
                    , row "Order size" [ 350, 500, 600 ] [ 100, 320 ] 550
                    , row "New customers" [ 1400, 2000, 2500 ] [ 1000, 1650 ] 2100
                    , row "Satisfaction" [ 3.5, 4.25, 5 ] [ 3.2, 4.7 ] 4.4
                    ]
                )

        res =
            resolve << resolution (reScale [ ( chX, reIndependent ) ])

        enc1 =
            encoding
                << position X
                    [ pName "ranges[2]"
                    , pQuant
                    , pScale [ scNice niFalse ]
                    , pTitle ""
                    ]

        spec1 =
            asSpec [ enc1 [], bar [ maColor "#eee" ] ]

        enc2 =
            encoding
                << position X [ pName "ranges[1]", pQuant ]

        spec2 =
            asSpec [ enc2 [], bar [ maColor "#ddd" ] ]

        enc3 =
            encoding
                << position X [ pName "ranges[0]", pQuant ]

        spec3 =
            asSpec [ enc3 [], bar [ maColor "#ccc" ] ]

        enc4 =
            encoding
                << position X [ pName "measures[1]", pQuant ]

        spec4 =
            asSpec [ enc4 [], bar [ maColor "lightsteelblue", maSize 10 ] ]

        enc5 =
            encoding
                << position X [ pName "measures[0]", pQuant ]

        spec5 =
            asSpec [ enc5 [], enc5 [], bar [ maColor "steelblue", maSize 10 ] ]

        enc6 =
            encoding
                << position X [ pName "markers[0]", pQuant ]

        spec6 =
            asSpec [ enc6 [], tick [ maColor "black" ] ]
    in
    toVegaLite
        [ cfg []
        , data []
        , res []
        , facet
            [ rowBy
                [ fName "title"
                , fOrdinal
                , fHeader
                    [ hdLabelAngle 0
                    , hdLabelAlign haLeft
                    , hdTitle ""
                    ]
                ]
            ]
        , specification (asSpec [ layer [ spec1, spec2, spec3, spec4, spec5, spec6 ] ])
        ]
```

## Dual Axis Chart

A bar/area chart with dual axes created by setting the y scales in a layered chart to be independent.

```elm {v}
dualAxes : Spec
dualAxes =
    let
        data =
            dataFromUrl (path ++ "weather.csv") []

        trans =
            transform
                << filter (fiExpr "datum.location == \"Seattle\"")
                << calculateAs "datum.precipitation * 25.4" "precipitationmm"

        encTime =
            encoding
                << position X
                    [ pName "date"
                    , pTimeUnit month
                    , pAxis [ axFormat "%b", axTitle "" ]
                    ]

        encArea =
            encoding
                << position Y
                    [ pName "temp_max"
                    , pAggregate opMean
                    , pScale [ scDomain (doNums [ 0, 30 ]) ]
                    , pAxis [ axTitle "Average Temperature (°C)", axTitleColor "#85C5A6" ]
                    ]
                << position Y2 [ pName "temp_min", pAggregate opMean ]

        specArea =
            asSpec [ encArea [], area [ maOpacity 0.3, maColor "#85C5A6" ] ]

        encLine =
            encoding
                << position Y
                    [ pName "precipitationmm"
                    , pAggregate opMean
                    , pAxis [ axTitle "Precipitation (mm)", axTitleColor "#85A9C5" ]
                    ]

        specLine =
            asSpec [ encLine [], line [ maStroke "#85A9C5", maInterpolate miMonotone ] ]

        res =
            resolve
                << resolution (reScale [ ( chY, reIndependent ) ])
    in
    toVegaLite
        [ width 400
        , height 300
        , data
        , trans []
        , encTime []
        , res []
        , layer [ specArea, specLine ]
        ]
```

## "Bar-bell" Chart

Inspired by this [Vega example](https://vega.github.io/vega-editor/?mode=vega&spec=weather). Weekly weather data plot representing high/low ranges of record temperatures (light grey), average temperatures (dark grey), and both predicted and observed temperatures (black) for the given week. The first five days have high/low ranges of observed temperatures, and the last five days have ranges of predicted temperatures, where the upper barbell represents the range of high temperature predictions and the lower barbell represents the range of low temperature predictions. Created by @melissatdiamond.

```elm {v}
barbell : Spec
barbell =
    let
        data =
            dataFromUrl (path ++ "weather.json") []

        enc1 =
            encoding
                << position Y
                    [ pName "record.low"
                    , pQuant
                    , pScale [ scDomain (doNums [ 10, 70 ]) ]
                    , pAxis [ axTitle "Temperature (F)" ]
                    ]
                << position Y2 [ pName "record.high" ]
                << position X [ pName "id", pTitle "Day" ]

        spec1 =
            asSpec [ enc1 [], bar [ maSize 20, maColor "lightgrey" ] ]

        enc2 =
            encoding
                << position Y [ pName "normal.low", pQuant ]
                << position Y2 [ pName "normal.high" ]
                << position X [ pName "id" ]

        spec2 =
            asSpec [ enc2 [], bar [ maSize 20, maColor "grey" ] ]

        enc3 =
            encoding
                << position Y [ pName "actual.low", pQuant ]
                << position Y2 [ pName "actual.high" ]
                << position X [ pName "id" ]

        spec3 =
            asSpec [ enc3 [], bar [ maSize 12, maColor "black" ] ]

        enc4 =
            encoding
                << position Y [ pName "forecast.low.low", pQuant ]
                << position Y2 [ pName "forecast.low.high" ]
                << position X [ pName "id" ]

        spec4 =
            asSpec [ enc4 [], bar [ maSize 12, maColor "black" ] ]

        enc5 =
            encoding
                << position Y [ pName "forecast.low.high", pQuant ]
                << position Y2 [ pName "forecast.high.low" ]
                << position X [ pName "id" ]

        spec5 =
            asSpec [ enc5 [], bar [ maSize 3, maColor "black" ] ]

        enc6 =
            encoding
                << position Y [ pName "forecast.high.low", pQuant ]
                << position Y2 [ pName "forecast.high.high" ]
                << position X [ pName "id" ]

        spec6 =
            asSpec [ enc6 [], bar [ maSize 12, maColor "black" ] ]

        enc7 =
            encoding
                << position X
                    [ pName "id"
                    , pOrdinal
                    , pAxis
                        [ axDomain False
                        , axTicks False
                        , axLabels False
                        , axTitle "Day"
                        , axTitlePadding 25
                        , axOrient siTop
                        ]
                    ]
                << text [ tName "day" ]

        spec7 =
            asSpec [ enc7 [], textMark [ maAlign haCenter, maDy -105 ] ]
    in
    toVegaLite
        [ title "Weekly Weather Observations and Predictions" []
        , width 250
        , height 200
        , data
        , layer [ spec1, spec2, spec3, spec4, spec5, spec6, spec7 ]
        ]
```

## Multi-Layer Chart

We can build more sophisticated designs by layering many specs on top of each other. Here is a reproduction [this design](https://www.reddit.com/r/dataisbeautiful/comments/azjti7/leonardo_dicaprio_refuses_to_date_a_woman_over_25) and combined with [XKCD's Creepiness Rule](https://xkcd.com/314/) showing Leonardo DiCaprio's dating history.

While the spec looks complicated, such designs can be built layer by layer in a systematic way.

```elm {v}
diCaprio : Spec
diCaprio =
    let
        minAge age =
            toFloat age / 2 + 7

        maxAge age =
            toFloat ((age - 7) * 2)

        textColour =
            "rgb(143,154,174)"

        dcColour =
            "rgb(223,117,45)"

        partnerColour =
            "rgb(91,198,214)"

        annotationFont =
            maFont "Roboto Slab"

        dcData =
            dataFromColumns []
                << dataColumn "year" (List.range 1999 2021 |> List.map toFloat |> nums)
                << dataColumn "dcAge" (List.range 24 46 |> List.map toFloat |> nums)
                << dataColumn "minAge" (List.range 24 46 |> List.map minAge |> nums)
                << dataColumn "maxAge" (List.range 24 46 |> List.map maxAge |> nums)
                << dataColumn "partnerAge" ([ 18, 19, 20, 21, 22, 23, 20, 21, 22, 23, 24, 25, 23, 22, 20, 21, 25, 24, 25, 20, 21, 22, 23 ] |> nums)

        annotationData =
            dataFromRows []
                << dataRow
                    [ ( "name", str "Gisele Bundchen" )
                    , ( "start", num 1999 )
                    , ( "end", num 2004 )
                    ]
                << dataRow
                    [ ( "name", str "Bar Refaeli" )
                    , ( "start", num 2005 )
                    , ( "end", num 2010 )
                    ]
                << dataRow
                    [ ( "name", str "Blake Lively" )
                    , ( "start", num 2011 )
                    , ( "end", num 2011 )
                    ]
                << dataRow
                    [ ( "name", str "Erin Heatherton" )
                    , ( "start", num 2012 )
                    , ( "end", num 2012 )
                    ]
                << dataRow
                    [ ( "name", str "Toni Garrn" )
                    , ( "start", num 2013 )
                    , ( "end", num 2014 )
                    ]
                << dataRow
                    [ ( "name", str "Kelly Rohrbach" )
                    , ( "start", num 2015 )
                    , ( "end", num 2015 )
                    ]
                << dataRow
                    [ ( "name", str "Nina Agdal" )
                    , ( "start", num 2016 )
                    , ( "end", num 2017 )
                    ]
                << dataRow
                    [ ( "name", str "Camilla Morrone" )
                    , ( "start", num 2018 )
                    , ( "end", num 2021 )
                    ]

        dcAnnotationData =
            dataFromRows []
                << dataRow
                    [ ( "dcX", num 2021 )
                    , ( "dcY", num 46 )
                    , ( "dcAnnotation", str "Leo's age" )
                    ]
                << dataRow
                    [ ( "dcX", num 2014 )
                    , ( "dcY", num 33 )
                    , ( "dcAnnotation", str "xkcd non-creepiness range" )
                    ]

        partnerAnnotationData =
            dataFromRows []
                << dataRow
                    [ ( "partnerX", num 2020 )
                    , ( "partnerY", num 26 )
                    , ( "partnerAnnotation", str "partner's age" )
                    ]

        -- XKCD Creepiness range
        encBand =
            encoding
                << position X [ pName "year", pOrdinal, pTitle "" ]
                << position Y
                    [ pName "minAge"
                    , pQuant
                    , pScale [ scZero False, scDomain (doNums [ 16, 50 ]) ]
                    , pTitle ""
                    ]
                << position Y2 [ pName "maxAge" ]

        specBand =
            asSpec [ encBand [], area [ maClip True, maFill dcColour, maOpacity 0.2 ] ]

        -- Leo's age
        encDiCaprio =
            encoding
                << position X [ pName "year", pOrdinal ]
                << position Y [ pName "dcAge", pQuant ]

        specDiCaprio =
            asSpec
                [ encDiCaprio []
                , line
                    [ maColor dcColour
                    , maStrokeWidth 1
                    , maPoint (pmMarker [ maStroke dcColour, maStrokeWidth 1.5, maFill "rgb(42,24,12)" ])
                    ]
                ]

        encDiCaprioText =
            encoding
                << position X [ pName "year", pOrdinal ]
                << position Y [ pName "dcAge", pQuant ]
                << text [ tName "dcAge", tQuant ]

        specDiCaprioText =
            asSpec [ encDiCaprioText [], textMark [ maColor dcColour, maDy -11 ] ]

        encDiCaprioLabel =
            encoding
                << position X [ pName "dcX", pOrdinal ]
                << position Y [ pName "dcY", pQuant ]
                << text [ tName "dcAnnotation" ]

        specDiCaprioLabel =
            asSpec
                [ dcAnnotationData []
                , encDiCaprioLabel []
                , textMark [ maColor dcColour, maAlign haLeft, annotationFont, maDx 10, maDy 5, maSize 14 ]
                ]

        -- Partners' ages
        encPartners =
            encoding
                << position X [ pName "year", pOrdinal ]
                << position Y
                    [ pName "partnerAge"
                    , pQuant
                    , pScale [ scZero False ]
                    ]
                << position Y2 [ pDatum (num 15) ]

        specPartners =
            asSpec
                [ encPartners []
                , bar
                    [ maColorGradient grLinear
                        [ grX1 1
                        , grX2 1
                        , grY1 1
                        , grY2 0
                        , grStops [ ( 0, "black" ), ( 1, partnerColour ) ]
                        ]
                    ]
                ]

        -- Partners' names
        encPartnerText =
            encoding
                << position X [ pName "year", pOrdinal ]
                << position Y [ pName "partnerAge", pQuant ]
                << text [ tName "partnerAge", tQuant ]

        specPartnerText =
            asSpec [ encPartnerText [], textMark [ maColor partnerColour, maDy -7 ] ]

        encPartnerRange =
            encoding
                << position X [ pName "start", pOrdinal ]
                << position X2 [ pName "end" ]
                << position Y [ pNum 420 ]

        specPartnerRange =
            asSpec [ annotationData [], encPartnerRange [], rule [ maColor partnerColour ] ]

        encPartnerNames =
            encoding
                << position X [ pName "start", pOrdinal ]
                << position Y [ pNum 435 ]
                << text [ tName "name" ]

        specPartnerNames =
            asSpec
                [ annotationData []
                , encPartnerNames []
                , textMark [ maColor partnerColour, maAlign haLeft, maAngle 30, annotationFont ]
                ]

        encPartnerLabel =
            encoding
                << position X [ pName "partnerX", pOrdinal ]
                << position Y [ pName "partnerY", pQuant ]
                << text [ tName "partnerAnnotation" ]

        specPartnerLabel =
            asSpec
                [ partnerAnnotationData []
                , encPartnerLabel []
                , textMark [ maColor partnerColour, maAlign haLeft, annotationFont, maDx 17, maDy 5, maSize 14 ]
                ]

        cfg =
            configure
                << configuration (coScale [ sacoBandPaddingInner 0.5 ])
                << configuration
                    (coAxis
                        [ axcoGridOpacity 0.1
                        , axcoTicks False
                        , axcoDomain False
                        , axcoLabelColor textColour
                        , axcoLabelAngle 0
                        ]
                    )
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coPadding (paSize 60))
                << configuration (coBackground "black")
                << configuration (coText [ maColor textColour ])
                << configuration (coTitle [ ticoColor textColour, ticoFont "Roboto Slab", ticoFontSize 22, ticoAnchor anStart ])
    in
    toVegaLite
        [ title "Leonardo DiCaprio has never dated a woman over 25" []
        , width 650
        , height 400
        , cfg []
        , dcData []
        , layer
            [ specBand
            , specDiCaprio
            , specDiCaprioText
            , specDiCaprioLabel
            , specPartners
            , specPartnerText
            , specPartnerRange
            , specPartnerNames
            , specPartnerLabel
            ]
        ]
```

## Playfair's Wheat and Wages Chart

Another example of a complex layered chart built up from simpler components.

Wheat and Wages. A recreation of [William Playfair's chart](https://apandre.files.wordpress.com/2011/03/oldcombocharwagesofmechanicvspriceofwheat1821.jpg) visualizing the price of wheat, the wages of a mechanic, and the reigning British monarch. Adapted from a chart by @manzt.

```elm {v}
wheatAndWages : Spec
wheatAndWages =
    let
        data =
            dataFromUrl (path ++ "wheat.json") []

        dataAnnotation =
            dataFromRows []
                << dataRow [ ( "x", str "1626" ), ( "y", num 8 ), ( "name", str "Weekly Wages of a Good Mechanic" ) ]

        dataMonarch =
            dataFromUrl (path ++ "monarchs.json")
                [ parse [ ( "start", foDate "%Y" ), ( "end", foDate "%Y" ) ] ]

        dataCurves =
            dataFromRows []
                << dataRow [ ( "x", str "1675" ), ( "y", num 80.3 ), ( "curve", str "inset" ) ]

        curves =
            categoricalDomainMap [ ( "inset", "m-43 a43,25 0 1,0 86,0a43,25 0 1,0 -86,0 a43,25.5 0 1,0 86,0a43,25.5 0 1,0 -86,0" ) ]

        dataText1 =
            dataFromRows [ parse [ ( "x", foDate "%Y %m" ) ] ]
                << dataRow [ ( "x", str "1675 1" ), ( "y", num 76 ), ( "name", str "CHART" ) ]
                << dataRow [ ( "x", str "1675 6" ), ( "y", num 76 ), ( "name", str "CHART" ) ]

        dataText2 =
            dataFromRows []
                << dataRow [ ( "x", str "1675" ), ( "y", num 72.5 ), ( "name", str "Showing at One View" ) ]
                << dataRow [ ( "x", str "1675" ), ( "y", num 68 ), ( "name", str "The Price of The Quarter of Wheat" ) ]
                << dataRow [ ( "x", str "1675" ), ( "y", num 58 ), ( "name", str "The Year 1565 to 1821" ) ]

        dataText3 =
            dataFromRows []
                << dataRow [ ( "x", str "1675" ), ( "y", num 62 ), ( "name", str "⤙ from ⤚" ) ]
                << dataRow [ ( "x", str "1675" ), ( "y", num 55 ), ( "name", str "⤙ by ⤚" ) ]

        dataText4 =
            dataFromRows []
                << dataRow [ ( "x", str "1574" ), ( "y", num 102 ), ( "name", str "16th Century" ) ]
                << dataRow [ ( "x", str "1650" ), ( "y", num 102 ), ( "name", str "17th Century" ) ]
                << dataRow [ ( "x", str "1750" ), ( "y", num 102 ), ( "name", str "18th Century" ) ]
                << dataRow [ ( "x", str "1822" ), ( "y", num 102 ), ( "name", str "19th Century" ) ]
                << dataRow [ ( "x", str "1675" ), ( "y", num 64.3 ), ( "name", str "& Wages of Labour by the Week" ) ]
                << dataRow [ ( "x", str "1675" ), ( "y", num 52.7 ), ( "name", str "WILLIAM PLAYFAIR" ) ]

        dataCentury =
            dataFromColumns [ parse [ ( "year", foDate "%Y" ) ] ]
                << dataColumn "year" (nums [ 1565, 1590, 1600, 1605, 1650, 1695, 1700, 1705, 1750, 1795, 1800, 1805, 1810, 1830 ])
                << dataColumn "y" (nums [ 106, 102, 100, 101, 106, 101, 100, 101, 106, 101, 100, 102, 103.5, 106 ])
                << dataColumn "y2" (nums [ 105, 102, 100, 101, 105, 101, 100, 101, 105, 101, 100, 102, 103.5, 105 ])

        transWages =
            transform
                << filter (fiExpr "year(datum.year) <= 1810")

        transMonarchBar =
            transform
                << calculateAs "((!datum.commonwealth && datum.index % 2) ? -1: 1) * 1.5 + 97" "y"
                << calculateAs "97" "x"

        transMonarchText =
            transform
                << calculateAs "((!datum.commonwealth && datum.index % 2) ? -1: 1) + 94" "y"
                << calculateAs "+datum.start + (+datum.end - +datum.start)/2" "x"

        cfg =
            configure
                << configuration
                    (coAxis
                        [ axcoTitleFont "Pinyon Script"
                        , axcoTitleFontWeight fwBold
                        , axcoLabelFont "Pinyon Script"
                        , axcoLabelFontSize 8
                        , axcoLabelFontWeight fwBold
                        ]
                    )
                << configuration
                    (coText
                        [ maFont "Pinyon Script"
                        , maFontWeight fwBold
                        , maAlign haCenter
                        ]
                    )

        encWheat =
            encoding
                << position X
                    [ pName "year"
                    , pTemporal
                    , pAxis
                        [ axDomainWidth 2
                        , axDomainColor "rgb(46,41,43)"
                        , axTicks False
                        , axTickCount (niInterval year 5)
                        , axGridColor "black"
                        , axGridOpacity 0.6
                        , axDataCondition (expr "year(datum.value) % 50 == 0") (cAxGridWidth 2 0.5)
                        , axLabelExpr "if (year(datum.value) % 10 == 5, ' ', if(year(datum.value) % 50 == 0, utcFormat(datum.value,'%Y'), utcFormat(datum.value,'%y')))"
                        , axTitle "5 Years each division"
                        , axZIndex 1
                        ]
                    ]
                << position Y
                    [ pName "wheat"
                    , pQuant
                    , pAxis
                        [ axTickCount (niTickCount 20)
                        , axTicks False
                        , axLabelPadding 4
                        , axGridColor "black"
                        , axDataCondition (expr "datum.value % 10 == 0") (cAxGridWidth 2 0.5)
                        , axLabelExpr "if (datum.value % 10 == 5, '5', datum.value)"
                        , axDomainWidth 2
                        , axDomainColor "rgb(46,41,43)"
                        , axTitle ""
                        , axZIndex 1
                        ]
                    , pScale [ scDomain (doNums [ 0, 100 ]) ]
                    ]

        specWheat =
            asSpec
                [ encWheat []
                , area
                    [ maInterpolate miStepAfter
                    , maColorGradient grLinear
                        [ grX1 1
                        , grX2 1
                        , grY1 1
                        , grY2 0
                        , grStops
                            [ ( 0.2, "white" )
                            , ( 0.4, "black" )
                            ]
                        ]
                    , maOpacity 0.8
                    ]
                ]

        encWages =
            encoding
                << position X [ pName "year", pTemporal ]
                << position Y
                    [ pName "wages"
                    , pQuant
                    , pAxis [ axDomainWidth 2 ]
                    ]

        specMechanic =
            asSpec
                [ transWages []
                , encWages []
                , layer
                    [ asSpec [ area [ maColor "rgb(170,210,220)", maLine (lmMarker [ maColor "black", maStrokeWidth 1 ]) ] ]
                    , asSpec [ line [ maColor "rgb(215,102,110)", maStrokeWidth 3, maYOffset -2 ] ]
                    ]
                ]

        specAnnotation =
            asSpec [ dataAnnotation [], encText [], textMark [ maAngle -2 ] ]

        encMonarchBar =
            encoding
                << position X [ pName "start", pTemporal ]
                << position X2 [ pName "end" ]
                << position Y [ pName "y", pQuant ]
                << position Y2 [ pName "x" ]
                << fill
                    [ mName "commonwealth"
                    , mScale [ scRange (raStrs [ "black", "white" ]) ]
                    , mLegend []
                    ]

        specMonarchBar =
            asSpec [ dataMonarch, transMonarchBar [], encMonarchBar [], bar [ maStroke "black" ] ]

        encText =
            encoding
                << position X [ pName "x", pTemporal ]
                << position Y [ pName "y", pQuant ]
                << text [ tName "name" ]

        encCurves =
            encoding
                << position X [ pName "x", pTemporal ]
                << position Y [ pName "y", pQuant ]
                << shape [ mName "curve", mScale curves, mLegend [] ]

        specCurves =
            asSpec [ dataCurves [], encCurves [], point [ maStroke "black", maOpacity 1 ] ]

        specText =
            asSpec
                [ encText []
                , layer
                    [ asSpec [ dataMonarch, transMonarchText [], textMark [] ]
                    , asSpec [ dataText1 [], textMark [ maFontSize 20, maFont "Old Standard TT" ] ]
                    , asSpec [ dataText2 [], textMark [ maFontSize 15 ] ]
                    , asSpec [ dataText3 [], textMark [] ]
                    , asSpec [ dataText4 [], textMark [ maFont "Old Standard TT" ] ]
                    ]
                ]

        encCentury =
            encoding
                << position X [ pName "year", pTemporal ]
                << position Y [ pName "y", pQuant ]
                << position Y2 [ pName "y2" ]

        specCentury =
            asSpec
                [ dataCentury []
                , encCentury []
                , area [ maStroke "black", maFill "black", maStrokeWidth 3, maInterpolate miMonotone ]
                ]
    in
    toVegaLite
        [ cfg []
        , width 900
        , height 450
        , data
        , layer
            [ specWheat
            , specMechanic
            , specAnnotation
            , specMonarchBar
            , specCurves
            , specText
            , specCentury
            ]
        ]
```
