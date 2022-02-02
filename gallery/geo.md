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
1.  [Labelling and annotation](layers.md)
1.  [Interactive charts](interactive.md)
1.  [Interactive linked views](interactiveLinked.md)
1.  [Faceted charts](facet.md)
1.  [Repeats and concatenation](concats.md)
1.  **Geographic maps**

---

# Geographic Mapping

Visualizations that handle geographic data, usually in the form of topoJSON files and/or longitude/latitude coordinate pairs.

Examples that use data from external sources tend to use files from the Vega-Lite data server. For consistency the path to the data location is defined here:

```elm {l}
path : String
path =
    "https://cdn.jsdelivr.net/npm/vega-datasets@2.2/data/"
```

## Map Projection

Geographic data need to be projected into 2 dimensions when shown on a page. How this is done depends on the a number of projection rules we can specify. This example allows the projection type and some of its parameters to be selected interactively.

```elm {v interactive}
projectionExplorer : Spec
projectionExplorer =
    let
        ps =
            params
                << param "type"
                    [ paValue (str "equalEarth")
                    , paBind
                        (ipSelect
                            [ inOptions
                                [ "albers"
                                , "albersUsa"
                                , "azimuthalEqualArea"
                                , "azimuthalEquidistant"
                                , "conicConformal"
                                , "conicEqualArea"
                                , "conicEquidistant"
                                , "equalEarth"
                                , "equirectangular"
                                , "gnomonic"
                                , "mercator"
                                , "naturalEarth1"
                                , "orthographic"
                                , "stereographic"
                                , "transverseMercator"
                                ]
                            ]
                        )
                    ]
                << param "scale" [ paValue (num 100), paBind (ipRange [ inMin 10, inMax 500 ]) ]
                << param "rotate0" [ paValue (num 0), paBind (ipRange [ inMin -180, inMax 180 ]) ]
                << param "rotate1" [ paValue (num 0), paBind (ipRange [ inMin -90, inMax 90 ]) ]
                << param "rotate2" [ paValue (num 0), paBind (ipRange [ inMin -180, inMax 180 ]) ]
                << param "parallels1" [ paValue (num 0), paBind (ipRange [ inMin -90, inMax 90 ]) ]
                << param "parallels2" [ paValue (num 45), paBind (ipRange [ inMin -90, inMax 90 ]) ]

        data =
            dataFromUrl (path ++ "world-110m.json") [ topojsonFeature "countries" ]

        sphereSpec =
            asSpec [ sphere, geoshape [ maFill "aliceblue" ] ]

        gratSpec =
            asSpec
                [ graticule [ grStep ( 15, 15 ) ]
                , geoshape [ maFilled False, maStrokeWidth 0.1, maStroke "black" ]
                ]

        countrySpec =
            asSpec [ data, geoshape [ maFill "#ccc" ] ]

        mp =
            projection
                [ prRotateExpr "rotate0" "rotate1" "rotate2"
                , prScale |> prNumExpr "scale"
                , prParallelsExpr "parallels1" "parallels2"
                , prType (prExpr "type")
                ]
    in
    toVegaLite
        [ ps []
        , width 700
        , height 450
        , mp
        , layer [ sphereSpec, countrySpec, gratSpec ]
        ]
```

## Choropleth Maps

Choropleth maps colour geographic areas by some data value. While uncluttered and appear to be easy to interpret, choropleths are vulnerable to misinterpretation when the size of geographic regions is not proportional to the phenomenon being investigated.

This example shows US unemployment rate by county, but larger, low density counties are more salient even if they contain fewer people.

```elm {v}
choropleth : Spec
choropleth =
    let
        countyData =
            dataFromUrl (path ++ "us-10m.json") [ topojsonFeature "counties" ]

        unemploymentData =
            dataFromUrl (path ++ "unemployment.tsv") []

        trans =
            transform
                << lookup "id" unemploymentData "id" (luFields [ "rate" ])

        proj =
            projection [ prType albersUsa ]

        enc =
            encoding
                << color [ mName "rate", mQuant ]
    in
    toVegaLite [ width 500, height 300, countyData, proj, trans [], enc [], geoshape [] ]
```

We can use [repeatFlow](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#repeatFlow) to juxtapose a series of choropleths, here at US State level.

```elm {v}
choropleths : Spec
choropleths =
    let
        data =
            dataFromUrl (path ++ "population_engineers_hurricanes.csv") []

        boundaryData =
            dataFromUrl (path ++ "us-10m.json") [ topojsonFeature "states" ]

        proj =
            projection [ prType albersUsa ]

        trans =
            transform
                << lookup "id" boundaryData "id" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color [ mRepeat arFlow, mQuant ]

        spec =
            asSpec [ width 300, data, trans [], proj, enc [], geoshape [] ]

        res =
            resolve
                << resolution (reScale [ ( chColor, reIndependent ) ])
    in
    toVegaLite
        [ columns 1
        , repeatFlow [ "population", "engineers", "hurricanes" ]
        , res []
        , specification spec
        ]
```

## Faceted Choropleth Maps

Income in the U.S. by state, faceted over income brackets. Default sorting order for string categories ("10000 to 14999", "15000 to 24999" etc) will be alphabetical which places "<10000" after all others, so we apply a custom sort placing it first. There is no need to specify the other categories as their alphabetic order is also numeric order.

```elm {v}
facetedChoropleth : Spec
facetedChoropleth =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        data =
            dataFromUrl (path ++ "income.json") []

        boundaryData =
            dataFromUrl (path ++ "us-10m.json") [ topojsonFeature "states" ]

        proj =
            projection [ prType albersUsa ]

        trans =
            transform
                << lookup "id" boundaryData "id" (luAs "geo")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color [ mName "pct", mQuant, mScale [ scScheme "goldorange" [] ] ]

        spec =
            asSpec [ width 130, height 80, proj, enc [], geoshape [] ]
    in
    toVegaLite
        [ cfg []
        , data
        , trans []
        , columns 5
        , facetFlow [ fName "group", fSort [ soCustom (strs [ "<10000" ]) ] ]
        , specification spec
        ]
```

## Dot maps

US zip-codes: One dot per zip-code coloured by first digit.

```elm {v}
zipDots : Spec
zipDots =
    let
        zipcodeData =
            dataFromUrl (path ++ "zipcodes.csv") []

        proj =
            projection [ prType albersUsa ]

        trans =
            transform
                << calculateAs "substring(datum.zip_code, 0, 1)" "digit"

        enc =
            encoding
                << position Longitude [ pName "longitude" ]
                << position Latitude [ pName "latitude" ]
                << color [ mName "digit", mScale [ scScheme "category10" [] ] ]
    in
    toVegaLite
        [ width 500, height 300, zipcodeData, proj, trans [], enc [], circle [ maSize 1 ] ]
```

Similarly, one dot per airport in the US overlaid on a geoshape.

```elm {v}
airports : Spec
airports =
    let
        boundaryData =
            dataFromUrl (path ++ "us-10m.json") [ topojsonFeature "states" ]

        airportData =
            dataFromUrl (path ++ "airports.csv") []

        proj =
            projection [ prType albersUsa ]

        backdropSpec =
            asSpec [ boundaryData, geoshape [ maColor "#eee" ] ]

        enc =
            encoding
                << position Longitude [ pName "longitude" ]
                << position Latitude [ pName "latitude" ]

        overlaySpec =
            asSpec [ airportData, circle [ maSize 5, maColor "steelblue" ], enc [] ]
    in
    toVegaLite [ width 500, height 300, proj, layer [ backdropSpec, overlaySpec ] ]
```

## Flow Map

Direct flights between Chicago O'Hare (ORD) and every other US airport. Line thickness indicates relative numbers of flights between Chicago and destination airport.

The source data on flights includes counts for all airports, so we need to filter just those whose origin is Chicago. The location of each airport, required to plot on a map, is stored in a separate data source, so we use [lookup](https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#lookup) to join the two tables using the [iata code](https://en.wikipedia.org/wiki/IATA_airport_code) common to both.

```elm {v}
flowMap : Spec
flowMap =
    let
        flightData =
            dataFromUrl (path ++ "flights-airport.csv") []

        airportData =
            dataFromUrl (path ++ "airports.csv") []

        boundaryData =
            dataFromUrl (path ++ "us-10m.json") [ topojsonFeature "states" ]

        trans =
            transform
                << filter (fiExpr "datum.origin == 'ORD'")
                << lookup "origin" airportData "iata" (luFields [ "latitude", "longitude" ])
                << calculateAs "datum.latitude" "origin_latitude"
                << calculateAs "datum.longitude" "origin_longitude"
                << lookup "destination" airportData "iata" (luFields [ "latitude", "longitude" ])
                << calculateAs "datum.latitude" "dest_latitude"
                << calculateAs "datum.longitude" "dest_longitude"

        backdropSpec =
            asSpec [ boundaryData, geoshape [ maColor "#eee" ] ]

        enc =
            encoding
                << position Longitude [ pName "origin_longitude" ]
                << position Latitude [ pName "origin_latitude" ]
                << position Longitude2 [ pName "dest_longitude" ]
                << position Latitude2 [ pName "dest_latitude" ]
                << strokeWidth
                    [ mName "count"
                    , mQuant
                    , mScale [ scRange (raNums [ 0.1, 20 ]) ]
                    ]

        flightsSpec =
            asSpec [ flightData, trans [], rule [ maOpacity 0.1, maColor "brown" ], enc [] ]

        proj =
            projection [ prType albersUsa ]
    in
    toVegaLite [ width 800, height 500, proj, layer [ backdropSpec, flightsSpec ] ]
```

## Multi-stage flow map

Line drawn between airports in the U.S. simulating a multi-trip flight itinerary between Seattle and Orlando.

```elm {v}
multiStage : Spec
multiStage =
    let
        boundaryData =
            dataFromUrl (path ++ "us-10m.json") [ topojsonFeature "states" ]

        airportData =
            dataFromUrl (path ++ "airports.csv") []

        backdropSpec =
            asSpec [ boundaryData, geoshape [ maColor "#eee" ] ]

        airportsEnc =
            encoding
                << position Longitude [ pName "longitude" ]
                << position Latitude [ pName "latitude" ]

        airportsSpec =
            asSpec [ circle [ maSize 5, maColor "grey" ], airportsEnc [] ]

        itinerary =
            dataFromColumns []
                << dataColumn "airport"
                    (strs [ "SEA", "SFO", "LAX", "LAS", "DFW", "DEN", "ORD", "JFK", "ATL", "MCO" ])
                << dataColumn "order" (List.range 1 10 |> List.map toFloat |> nums)

        trans =
            transform
                << lookup "airport" airportData "iata" (luFields [ "latitude", "longitude" ])

        flightsEnc =
            encoding
                << position Longitude [ pName "longitude" ]
                << position Latitude [ pName "latitude" ]
                << order [ oName "order" ]

        flightsSpec =
            asSpec [ itinerary [], trans [], line [ maInterpolate miMonotone ], flightsEnc [] ]
    in
    toVegaLite
        [ width 800
        , height 500
        , airportData
        , projection [ prType albersUsa ]
        , layer [ backdropSpec, airportsSpec, flightsSpec ]
        ]
```

## Interactive map

Flights to/from airport nearest to mouse position.

Note the use of [lookup](#https://package.elm-lang.org/packages/gicentre/elm-vegalite/latest/VegaLite#lookup) to give separate names to the origin longitude/latitude (`"o"`) and destination longitude/latitude (`"d"`). The individual fields that have been renamed can be referenced with dot notation (`o.longitude`, `o.latitude`, `d.longitude` and `d.latitude`).

```elm {v interactive}
interactiveMap : Spec
interactiveMap =
    let
        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])

        airportData =
            dataFromUrl (path ++ "airports.csv") []

        flightData =
            dataFromUrl (path ++ "flights-airport.csv") []

        backdropSpec =
            asSpec
                [ dataFromUrl (path ++ "us-10m.json") [ topojsonFeature "states" ]
                , geoshape [ maFill "#ddd", maStroke "#fff" ]
                ]

        lineTrans =
            transform
                << filter (fiSelectionEmpty "mySelection")
                << lookup "origin" airportData "iata" (luAs "o")
                << lookup "destination" airportData "iata" (luAs "d")

        lineEnc =
            encoding
                << position Longitude [ pName "o.longitude" ]
                << position Latitude [ pName "o.latitude" ]
                << position Longitude2 [ pName "d.longitude" ]
                << position Latitude2 [ pName "d.latitude" ]

        lineSpec =
            asSpec [ flightData, lineTrans [], lineEnc [], rule [ maColor "black", maOpacity 0.35 ] ]

        airportTrans =
            transform
                << aggregate [ opAs opCount "" "routes" ] [ "origin" ]
                << lookup "origin" airportData "iata" (luFields [ "state", "latitude", "longitude" ])
                << filter (fiExpr "datum.state !== 'PR' && datum.state !== 'VI'")

        airportEnc =
            encoding
                << position Longitude [ pName "longitude" ]
                << position Latitude [ pName "latitude" ]
                << size [ mName "routes", mScale [ scRange (raNums [ 0, 1000 ]) ], mLegend [] ]
                << order [ oName "routes", oSort [ soDescending ] ]

        ps =
            params
                << param "mySelection" [ paSelect sePoint [ seOn "mouseover", seNearest True, seFields [ "origin" ] ] ]

        airportSpec =
            asSpec [ flightData, airportTrans [], ps [], airportEnc [], circle [] ]
    in
    toVegaLite
        [ cfg []
        , width 700
        , height 400
        , projection [ prType albersUsa ]
        , layer [ backdropSpec, lineSpec, airportSpec ]
        ]
```

## Map with labels

US state capitals overlaid on a map of the US.

```elm {v}
labelledMap : Spec
labelledMap =
    let
        boundaryData =
            dataFromUrl (path ++ "us-10m.json") [ topojsonFeature "states" ]

        data =
            dataFromUrl (path ++ "us-state-capitals.json") []

        backdropSpec =
            asSpec [ boundaryData, geoshape [ maFill "#dcc", maStroke "white" ] ]

        overlayEnc =
            encoding
                << position Longitude [ pName "lon" ]
                << position Latitude [ pName "lat" ]
                << text [ tName "city" ]

        overlaySpec =
            asSpec
                [ textMark [ maColor "#5a2d0c", maDy 2, maFontSize 8, maFont "serif" ]
                , overlayEnc []
                ]
    in
    toVegaLite
        [ width 800
        , height 500
        , data
        , projection [ prType albersUsa ]
        , layer [ backdropSpec, overlaySpec ]
        ]
```

## London Underground Lines

Geographic position of London Underground lines overlaid on a London borough map.

```elm {v}
tubeLines : Spec
tubeLines =
    let
        boroughBounds =
            dataFromUrl (path ++ "londonBoroughs.json") [ topojsonFeature "boroughs" ]

        boroughCentroids =
            dataFromUrl (path ++ "londonCentroids.json") []

        tubeLineData =
            dataFromUrl (path ++ "londonTubeLines.json") [ topojsonFeature "line" ]

        tubeLineColors =
            categoricalDomainMap
                [ ( "Bakerloo", "rgb(137,78,36)" )
                , ( "Central", "rgb(220,36,30)" )
                , ( "Circle", "rgb(255,206,0)" )
                , ( "District", "rgb(1,114,41)" )
                , ( "DLR", "rgb(0,175,173)" )
                , ( "Hammersmith & City", "rgb(215,153,175)" )
                , ( "Jubilee", "rgb(106,114,120)" )
                , ( "Metropolitan", "rgb(114,17,84)" )
                , ( "Northern", "rgb(0,0,0)" )
                , ( "Piccadilly", "rgb(0,24,168)" )
                , ( "Victoria", "rgb(0,160,226)" )
                , ( "Waterloo & City", "rgb(106,187,170)" )
                ]

        polySpec =
            asSpec
                [ boroughBounds
                , geoshape [ maFill "#eee", maStroke "white", maStrokeWidth 2 ]
                ]

        labelEnc =
            encoding
                << position Longitude [ pName "cx" ]
                << position Latitude [ pName "cy" ]
                << text [ tName "bLabel" ]
                << size [ mNum 8 ]
                << opacity [ mNum 0.6 ]

        trans =
            transform
                << calculateAs "indexof (datum.name,' ') > 0  ? substring(datum.name,0,indexof(datum.name, ' ')) : datum.name" "bLabel"

        labelSpec =
            asSpec [ boroughCentroids, trans [], textMark [], labelEnc [] ]

        tubeEnc =
            encoding
                << color
                    [ mName "id"
                    , mLegend [ leTitle "", leOrient loBottomLeft ]
                    , mScale tubeLineColors
                    ]

        routeSpec =
            asSpec
                [ tubeLineData
                , geoshape [ maFilled False, maStrokeWidth 2 ]
                , tubeEnc []
                ]

        cfg =
            configure
                << configuration (coView [ vicoStroke Nothing ])
    in
    toVegaLite
        [ width 700
        , height 500
        , cfg []
        , layer [ polySpec, labelSpec, routeSpec ]
        ]
```

---

## Other Maps

For a more diverse range of examples, see these litvis [30 day map challenge](https://github.com/jwoLondon/30dayMapChallenge) examples.
