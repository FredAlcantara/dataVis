---
id: litvis

narrative-schemas:
  - ../../lectures/narrative-schemas/project.yml

elm:
  dependencies:
    gicentre/elm-vegalite: latest
    gicentre/tidy: latest
---

@import "../../lectures/css/datavis.less"

```elm {l=hidden}
import Tidy exposing (..)
import VegaLite exposing (..)
```

# Data Visualization Project Summary

{(questions|}

**Does the secondary education curriculum provide equal opportunities for all its students?**

- What is the difference between male and female performance in GCSE achievement?
- Is there a significant correlation between a student's area and their overall attainment? Do they perform equally as those in neighboring boroughs?
- Does the choice of optional subjects have an influence on GCSE achievement?

{|questions)}

{(visualization|}

```elm {l=hidden}
colourSelection =
    categoricalDomainMap
        [ ( "Female", "rgb(2231, 84, 128)" )
        , ( "Male", "rgb(0, 0, 255)" )
        ]


dataElectiveModules =
    dataFromUrl "https://fredalcantara.github.io/data/Elective_modules.csv" []


dataCoreModules =
    dataFromUrl "https://fredalcantara.github.io/data/Core_modules_result.csv" []


dataCandidates =
    dataFromUrl "https://fredalcantara.github.io/data/candidates.csv" []
```

```elm {l=hidden}
overallLegend : String -> String -> Spec
overallLegend field selName =
    let
        enc =
            encoding
                << position Y
                    [ pName field
                    , pAxis [ axTitle "Gender", axDomain False, axTicks True ]
                    ]
                << color [ mName field, mScale colourSelection, mLegend [] ]

        ps =
            params
                << param selName
                    [ paSelect sePoint [ seEncodings [ chColor ] ] ]
    in
    asSpec [ ps [], enc [], square [ maSize 300, maOpacity 20 ] ]
```

```elm {l=hidden v interactive}
visualisationCoreModules : Spec
visualisationCoreModules =
    let
        trans =
            transform
                << filter (fiSelection "legendSel")

        enc =
            encoding
                << position X
                    [ pName "Year"
                    , pTemporal
                    , pAxis
                        [ axTitle "Year"
                        , axTicks False
                        , axDomain False
                        , axLabelAngle -60
                        ]
                    ]
                << position Y
                    [ pName "CorHigher"
                    , pQuant
                    , pAxis
                        [ axTitle "% of grade 4 (C) and above"
                        , axValues (nums [])
                        ]
                    ]
                << color [ mName "Gender", mScale colourSelection, mTitle "Gender" ]
                << column [ fName "Subject", fHeader [ hdTitle "Core Subjects" ] ]
                << tooltips
                    [ [ tName "Year", tTemporal, tFormat "%Y" ]
                    , [ tName "CorHigher"
                      , tTitle "Cumulative percentage outcome, achieving grade 4 or higher"
                      ]
                    ]

        ps =
            params
                << param "zoom"
                    [ paSelect seInterval []
                    , paBindScales
                    ]

        chartSpec =
            asSpec
                [ height 200
                , width 300
                , trans []
                , enc []
                , line [ maPoint (pmMarker []) ]
                , ps []
                ]

        cfg =
            configure
                << configuration (coTitle [ ticoFont "Roboto Slab", ticoFontSize 22, ticoAnchor anMiddle ])
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False ])
    in
    toVegaLite
        [ title "GCSE Achievement, Male and Female" []
        , dataCoreModules
        , hConcat [ chartSpec, overallLegend "Gender" "legendSel" ]
        , cfg []
        ]
```

```elm {l=hidden v interactive}
visualisationElectiveModules : Spec
visualisationElectiveModules =
    let
        trans =
            transform
                << filter (fiSelection "legendSel")

        enc =
            encoding
                << position X
                    [ pName "Year"
                    , pTemporal
                    , pAxis
                        [ axTitle "Year"
                        , axTicks False
                        , axDomain False
                        , axLabelAngle -60
                        ]
                    ]
                << position Y
                    [ pName "CorHigher"
                    , pQuant
                    , pAxis
                        [ axTitle "% of grades 4 (C) and above"
                        , axValues (nums [])
                        ]
                    ]
                << color [ mName "Gender", mScale colourSelection, mTitle "Gender" ]
                << column [ fName "Subject", fHeader [ hdTitle "Elective Subjects" ] ]
                << tooltips
                    [ [ tName "Year", tTemporal, tFormat "%Y" ]
                    , [ tName "CorHigher"
                      , tTitle "Cumulative percentage outcome, achieving grade 4 or higher"
                      ]
                    ]

        ps =
            params
                << param "zoom"
                    [ paSelect seInterval []
                    , paBindScales
                    ]

        chartSpec =
            asSpec
                [ height 200
                , width 300
                , trans []
                , enc []
                , line [ maPoint (pmMarker []) ]
                , ps []
                ]

        cfg =
            configure
                << configuration (coTitle [ ticoFont "Roboto Slab", ticoFontSize 22, ticoAnchor anStart ])
                << configuration (coView [ vicoStroke Nothing ])
                << configuration (coAxis [ axcoTicks False, axcoDomain False ])
    in
    toVegaLite
        [ dataElectiveModules
        , hConcat [ chartSpec, overallLegend "Gender" "legendSel" ]
        , cfg []
        ]
```

_**Figure 1** - Comparison of GCSE achievement in core and elective subjects between males and females._

```elm {v interactive}
numberOfCertificates : Spec
numberOfCertificates =
    let
        backgroundEnc =
            encoding
                << position X [ pName "Year" ]
                << position Y [ pName "Candidates", pAggregate opSum, pTitle "Certificates" ]

        backgroundSpec =
            asSpec [ backgroundEnc [], bar [ maTooltip ttEncoding, maOpacity 0.2, maColor "black" ] ]

        groupedEnc =
            encoding
                << position X
                    [ pName "Year"
                    , pAxis
                        [ axTitle "Year"
                        , axTicks False
                        , axDomain False
                        , axLabelAngle -60
                        ]
                    ]
                << position XOffset [ pName "Gender" ]
                << position Y
                    [ pName "Candidates"
                    , pAggregate opSum
                    , pAxis
                        [ axTitle ""
                        , axTicks False
                        , axDomain False
                        , axLabelAngle -60
                        ]
                    ]
                << color [ mName "Gender", mScale colourSelection, mTitle "Gender" ]
                << tooltips
                    [ [ tName "Gender"
                      , tNominal
                      ]
                    , [ tName "Candidates"
                      , tTitle "Certificates"
                      ]
                    ]

        groupedSpec =
            asSpec [ widthStep 25, height 500, groupedEnc [], bar [] ]

        cfg =
            configure
                << configuration (coTitle [ ticoFont "Roboto Slab", ticoFontSize 22, ticoAnchor anMiddle ])
    in
    toVegaLite [ title "Number of Certificates, All subjects" [], dataCandidates, layer [ backgroundSpec, groupedSpec ], cfg [] ]
```

_**Figure 2** - Number of GCSE certificates obtained throughout the years amongst male and female candidates._

```elm {l=hidden}
dataBoroughTable : Table
dataBoroughTable =
    """GSSCode,Area,Year,Attainment8,Pupils
E09000002,Barking and Dagenham,2016,49.7,2201
E09000003,Barnet,2016,56.1,3647
E09000004,Bexley,2016,52.2,3192
E09000005,Brent,2016,51.5,3035
E09000006,Bromley,2016,53.7,3305
E09000007,Camden,2016,50.6,1422
E09000008,Croydon,2016,48.5,3844
E09000009,Ealing,2016,50.9,2781
E12000006,East,2016,50.4,61059
E12000004,East Midlands,2016,48.9,47204
E09000010,Enfield,2016,50.4,3579
E92000001,England,2016,50.1,537808
E09000011,Greenwich,2016,49.6,2204
E09000012,Hackney,2016,52.5,2038
E09000013,Hammersmith and Fulham,2016,54.1,1350
E09000014,Haringey,2016,50.1,2231
E09000015,Harrow,2016,53.1,2127
E09000016,Havering,2016,50,2917
E09000017,Hillingdon,2016,51.2,3136
E09000018,Hounslow,2016,51.1,2667
E13000001,Inner London,2016,51.3,25070
E09000019,Islington,2016,50.6,1397
E09000020,Kensington and Chelsea,2016,56.6,740
E09000021,Kingston upon Thames,2016,58.2,1515
E09000022,Lambeth,2016,49.7,1887
E09000023,Lewisham,2016,47.5,2334
E12000007,London,2016,51.9,76596
E09000024,Merton,2016,52.4,1440
E09000025,Newham,2016,50.9,3489
E12000001,North East,2016,48.7,26076
E12000002,North West,2016,49.4,74057
E13000002,Outer London,2016,52.3,51526
E09000026,Redbridge,2016,53.9,3363
E09000027,Richmond upon Thames,2016,54.6,1367
E12000008,South East,2016,51,85618
E12000009,South West,2016,50.3,52421
E09000028,Southwark,2016,52.9,2386
E09000029,Sutton,2016,58.7,2667
E09000030,Tower Hamlets,2016,50.2,2570
E09000031,Waltham Forest,2016,50.4,2539
E09000032,Wandsworth,2016,52.1,1660
E12000005,West Midlands,2016,49.2,60215
E09000033,Westminster,2016,54.5,1566
E12000003,Yorkshire and the Humber,2016,48.9,54562
E09000002,Barking and Dagenham,2017,46.7,2185
E09000003,Barnet,2017,54.7,3528
E09000004,Bexley,2017,49,3141
E09000005,Brent,2017,49,2908
E09000006,Bromley,2017,49.8,3258
E09000007,Camden,2017,48.3,1513
E09000008,Croydon,2017,45,3579
E09000009,Ealing,2017,48.7,2722
E12000006,East,2017,46.7,60153
E12000004,East Midlands,2017,45.4,45492
E09000010,Enfield,2017,46.2,3487
E92000001,England,2017,46.4,524932
E09000011,Greenwich,2017,45.9,2166
E09000012,Hackney,2017,49.4,2002
E09000013,Hammersmith and Fulham,2017,50.9,1313
E09000014,Haringey,2017,46.5,2127
E09000015,Harrow,2017,49.7,2049
E09000016,Havering,2017,47.5,2794
E09000017,Hillingdon,2017,47.1,3075
E09000018,Hounslow,2017,48,2641
E13000001,Inner London,2017,48.2,24971
E09000019,Islington,2017,45.6,1377
E09000020,Kensington and Chelsea,2017,55,739
E09000021,Kingston upon Thames,2017,55.5,1517
E09000022,Lambeth,2017,44.3,1949
E09000023,Lewisham,2017,44.2,2247
E12000007,London,2017,48.9,75518
E09000024,Merton,2017,50.2,1420
E09000025,Newham,2017,48.4,3523
E12000001,North East,2017,44.6,25177
E12000002,North West,2017,45.6,72289
E13000002,Outer London,2017,49.2,50547
E09000026,Redbridge,2017,51.2,3458
E09000027,Richmond upon Thames,2017,52.7,1374
E12000008,South East,2017,47.4,83903
E12000009,South West,2017,46.2,50308
E09000028,Southwark,2017,50.5,2328
E09000029,Sutton,2017,56.2,2759
E09000030,Tower Hamlets,2017,47.2,2623
E09000031,Waltham Forest,2017,45.5,2486
E09000032,Wandsworth,2017,49.5,1695
E12000005,West Midlands,2017,45.4,58919
E09000033,Westminster,2017,52.6,1535
E12000003,Yorkshire and the Humber,2017,45.4,53173
E09000002,Barking and Dagenham,2018,46.1,2199
E09000003,Barnet,2018,56,3810
E09000004,Bexley,2018,49.6,3032
E09000005,Brent,2018,49.9,2858
E09000006,Bromley,2018,50.3,3214
E09000007,Camden,2018,48,1533
E09000008,Croydon,2018,45.8,3452
E09000009,Ealing,2018,50,2853
E12000006,East,2018,47,59100
E12000004,East Midlands,2018,45.5,45234
E09000010,Enfield,2018,46.3,3414
E92000001,England,2018,44.5,583617
E09000011,Greenwich,2018,44.5,2269
E09000012,Hackney,2018,49,2058
E09000013,Hammersmith and Fulham,2018,52.9,1291
E09000014,Haringey,2018,46.3,2190
E09000015,Harrow,2018,50.7,2044
E09000016,Havering,2018,46.9,2774
E09000017,Hillingdon,2018,47.8,3064
E09000018,Hounslow,2018,49.4,2666
E13000001,Inner London,2018,48.3,25466
E09000019,Islington,2018,46.3,1404
E09000020,Kensington and Chelsea,2018,51.6,718
E09000021,Kingston upon Thames,2018,57.8,1515
E09000022,Lambeth,2018,44.6,2114
E09000023,Lewisham,2018,44.9,2104
E12000007,London,2018,49.4,76224
E09000024,Merton,2018,49.7,1404
E09000025,Newham,2018,48.7,3745
E12000001,North East,2018,44.9,24838
E12000002,North West,2018,45.7,71555
E13000002,Outer London,2018,49.9,50758
E09000026,Redbridge,2018,53.1,3508
E09000027,Richmond upon Thames,2018,51.7,1479
E12000008,South East,2018,47.8,82803
E12000009,South West,2018,46.7,49425
E09000028,Southwark,2018,50.2,2386
E09000029,Sutton,2018,58.1,2719
E09000030,Tower Hamlets,2018,46.8,2721
E09000031,Waltham Forest,2018,46.1,2484
E09000032,Wandsworth,2018,50.8,1707
E12000005,West Midlands,2018,45.2,58845
E09000033,Westminster,2018,52.9,1495
E12000003,Yorkshire and the Humber,2018,45.1,53178
E09000002,Barking and Dagenham,2019,46.4,2353
E09000003,Barnet,2019,57.1,3804
E09000004,Bexley,2019,49.6,3115
E09000005,Brent,2019,50.2,3038
E09000006,Bromley,2019,50.8,3312
E09000007,Camden,2019,48.6,1571
E09000008,Croydon,2019,45.5,3640
E09000009,Ealing,2019,50.9,2956
E12000006,East,2019,47,60993
E12000004,East Midlands,2019,45.8,47019
E09000010,Enfield,2019,46.5,3606
E92000001,England,2019,46.8,540006
E09000011,Greenwich,2019,45.3,2364
E09000012,Hackney,2019,49.2,2230
E09000013,Hammersmith and Fulham,2019,53.9,1326
E09000014,Haringey,2019,46.9,2336
E09000015,Harrow,2019,50.9,2210
E09000016,Havering,2019,48.5,2829
E09000017,Hillingdon,2019,47.7,3203
E09000018,Hounslow,2019,49.3,2731
E13000001,Inner London,2019,48.4,26861
E09000019,Islington,2019,45.8,1465
E09000020,Kensington and Chelsea,2019,53.6,867
E09000021,Kingston upon Thames,2019,56.9,1550
E09000022,Lambeth,2019,44.1,2189
E09000023,Lewisham,2019,43.7,2187
E12000007,London,2019,49.7,79701
E09000024,Merton,2019,51.1,1455
E09000025,Newham,2019,48.8,3933
E12000001,North East,2019,44.7,25455
E12000002,North West,2019,45.5,74006
E13000002,Outer London,2019,50.4,52840
E09000026,Redbridge,2019,54,3687
E09000027,Richmond upon Thames,2019,54.1,1586
E12000008,South East,2019,48,85952
E12000009,South West,2019,46.7,51455
E09000028,Southwark,2019,49.5,2509
E09000029,Sutton,2019,58.6,2796
E09000030,Tower Hamlets,2019,48.4,2808
E09000031,Waltham Forest,2019,46.2,2605
E09000032,Wandsworth,2019,49.4,1785
E12000005,West Midlands,2019,45.6,60984
E09000033,Westminster,2019,53.4,1655
E12000003,Yorkshire and the Humber,2019,45.4,54441
E09000002,Barking and Dagenham,2020,49.7,2525
E09000003,Barnet,2020,60.1,4010
E09000004,Bexley,2020,53.5,3150
E09000005,Brent,2020,53.1,3078
E09000006,Bromley,2020,54.5,3411
E09000007,Camden,2020,52.2,1573
E09000008,Croydon,2020,48.9,3755
E09000009,Ealing,2020,53.5,3043
E12000006,East,2020,50.3,63682
E12000004,East Midlands,2020,49.1,48542
E09000010,Enfield,2020,49.8,3767
E92000001,England,2020,50.2,561994
E09000011,Greenwich,2020,50.2,2448
E09000012,Hackney,2020,53.2,2286
E09000013,Hammersmith and Fulham,2020,56.1,1458
E09000014,Haringey,2020,51.4,2484
E09000015,Harrow,2020,53.4,2334
E09000016,Havering,2020,51.9,2842
E09000017,Hillingdon,2020,52.1,3301
E09000018,Hounslow,2020,52.6,2916
E13000001,Inner London,2020,52.3,28132
E09000019,Islington,2020,49.7,1471
E09000020,Kensington and Chelsea,2020,58,899
E09000021,Kingston upon Thames,2020,58.9,1767
E09000022,Lambeth,2020,49.3,2250
E09000023,Lewisham,2020,48.4,2305
E12000007,London,2020,53.2,83129
E09000024,Merton,2020,53.1,1555
E09000025,Newham,2020,53.1,4079
E12000001,North East,2020,48.4,26395
E12000002,North West,2020,49,77572
E13000002,Outer London,2020,53.6,54997
E09000026,Redbridge,2020,56,3815
E09000027,Richmond upon Thames,2020,57.5,1667
E12000008,South East,2020,51.4,88737
E12000009,South West,2020,50.4,52331
E09000028,Southwark,2020,53.7,2659
E09000029,Sutton,2020,61.3,2920
E09000030,Tower Hamlets,2020,50.1,3082
E09000031,Waltham Forest,2020,50,2693
E09000032,Wandsworth,2020,52.7,1865
E12000005,West Midlands,2020,49,64062
E09000033,Westminster,2020,57.1,1721
E12000003,Yorkshire and the Humber,2020,48.3,57544
E09000002,Barking and Dagenham,2021,50.5,2805
E09000003,Barnet,2021,60.8,4100
E09000004,Bexley,2021,54,3262
E09000005,Brent,2021,53.7,3160
E09000006,Bromley,2021,55.2,3356
E09000007,Camden,2021,53.1,1644
E09000008,Croydon,2021,50,3831
E09000009,Ealing,2021,53.6,3276
E12000006,East,2021,51,64647
E12000004,East Midlands,2021,49.6,49843
E09000010,Enfield,2021,51.1,3783
E92000001,England,2021,50.9,575863
E09000011,Greenwich,2021,51.2,2695
E09000012,Hackney,2021,54,2331
E09000013,Hammersmith and Fulham,2021,58.1,1458
E09000014,Haringey,2021,51.4,2514
E09000015,Harrow,2021,54.8,2521
E09000016,Havering,2021,52.2,2882
E09000017,Hillingdon,2021,52.8,3420
E09000018,Hounslow,2021,53.9,2916
E13000001,Inner London,2021,53.4,28891
E09000019,Islington,2021,52.2,1493
E09000020,Kensington and Chelsea,2021,57.9,896
E09000021,Kingston upon Thames,2021,61.4,1828
E09000022,Lambeth,2021,51.3,2264
E09000023,Lewisham,2021,49.1,2340
E12000007,London,2021,54.1,85646
E09000024,Merton,2021,53.2,1555
E09000025,Newham,2021,54.5,4311
E12000001,North East,2021,49.2,27055
E12000002,North West,2021,49.6,79949
E13000002,Outer London,2021,54.5,56755
E09000026,Redbridge,2021,56.8,3874
E09000027,Richmond upon Thames,2021,58.1,1736
E12000008,South East,2021,52.1,90615
E12000009,South West,2021,51.4,53700
E09000028,Southwark,2021,55,3002
E09000029,Sutton,2021,62,3048
E09000030,Tower Hamlets,2021,51.7,2967
E09000031,Waltham Forest,2021,51.5,2707
E09000032,Wandsworth,2021,52.2,1919
E12000005,West Midlands,2021,49.5,65625
E09000033,Westminster,2021,57.6,1752
E12000003,Yorkshire and the Humber,2021,49.1,58783
"""
        |> fromCSV
```

```elm {v interactive}
attainmentMap : Spec
attainmentMap =
    let
        boundaryData =
            dataFromUrl "https://fredalcantara.github.io/data/londonBoroughs.json"
                [ topojsonFeature "boroughs" ]

        attainmentScoreData =
            dataFromColumns []
                << dataColumn "GSSCode" (strColumn "GSSCode" dataBoroughTable |> strs)
                << dataColumn "Attainment8" (numColumn "Attainment8" dataBoroughTable |> nums)
                << dataColumn "Year" (numColumn "Year" dataBoroughTable |> nums)
                << dataColumn "Area" (strColumn "Area" dataBoroughTable |> strs)
                << dataColumn "Pupils" (numColumn "Pupils" dataBoroughTable |> nums)

        ps =
            params
                << param "timeSlider"
                    [ paValue (dataObject [ ( "Year", num 2016 ) ])
                    , paSelect sePoint []
                    , paBind (ipRange [ inName "Year", inMin 2016, inMax 2021, inStep 1 ])
                    ]

        trans =
            transform
                << lookup "GSSCode" boundaryData "properties.GSSCode" (luAs "geo")
                << filter (fiExpr "isValid(datum.geo)")
                << filter (fiSelection "timeSlider")

        enc =
            encoding
                << shape [ mName "geo", mGeo ]
                << color
                    [ mName "Attainment8"
                    , mQuant
                    , mTitle "Attainment 8 Score"
                    , mScale [ scDomain (doNums [ 10, 70 ]) ]
                    , mScale [ scScheme "bluegreen" [] ]
                    ]
                << tooltips [ [ tName "Area", tTitle "Borough" ], [ tName "Attainment8", tTitle "Score" ], [ tName "Pupils", tTitle "Candidates" ] ]

        cfg =
            configure
                << configuration (coTitle [ ticoFont "Roboto Slab", ticoFontSize 22, ticoAnchor anMiddle ])
                << configuration (coPadding (paSize 50))
    in
    toVegaLite
        [ title "Overall Attainment, London boroughs" []
        , attainmentScoreData []
        , ps []
        , trans []
        , width 800
        , height 500
        , enc []
        , geoshape [ maStroke "black" ]
        , cfg []
        ]
```

_**Figure 3** - Overall attainment scores across London boroughs between 2016 and 2021._

{|visualization)}

{(insights|}

### i. Exploring Gender Performance

There is a clear distinction between gender performance in the UK secondary education system. Using **_Figure 1_**, we can deduce that females perform better than males in the given examples, core subjects and some elective subjects. When looking at the core subjects (specifically English Language, Literature and Science) there is a clear representation that females are passing or achieving higher than male pupils.

In relation to Mathematics, there was almost no disparity shown due to how close the achievement metrics were. Using a combination of the legend filter, zoom-in and tool-tip features made the findings interesting. At the origin of the visualization (2008), females achieved a greater percentage of passing rate than males. A year later, male pupils obtained better results that lasted until 2019. The result is insightful because Mathematics is a subject which does not contain any form of coursework or controlled assessment, thus, perhaps this is the reason results between the two genders were so narrow compared to other subjects. Furthermore, the elective subjects showcases a similar pattern. These included coursework tasks and presents females obtaining a better percentage of achieving a pass or higher than males. The visualization confirmations that females tend to do better than males in GCSE performance and provide a factor as to why this may be the case.

### ii. Exploring Location Factors

The use of a filtered choropleth map (**_Figure 3_**) illustrates Attainment 8 scores in London boroughs throughout the years, providing me with an insight that the location of a pupil's institution is a factor in their overall attainment. Using the time-scroller showed how attainment scores in across different boroughs progressed both positively and negatively over a 5-year period. With this, patterns can be identified. The following boroughs: Sutton, Kingston Upon Thames, Richmond Upon Thames, Hammersmith and Fulham, Kensington and Chelsea, Westminster and Barnet are areas which consistently recorded better attainment scores than the others.

So, why is this the case? Although the visualization cannot present us with a definite answer, we can assume and use the knowledge we know. Perhaps it is due to the total spending and budget allocation by local councils on the education system per London borough. As a result, some institutes have access to better equipment and facilities than others, thus, allowing students to benefit from far better learning experiences than those on the lower-end. Another reason is the ideology or stigma surrounding specific areas in London. Students who study in a negatively-labelled area are influenced into believing that their path is destined to be unsuccessful, and therefore do not work as hard and achieve less academically.

The visualization showcased a clear pattern of boroughs retaining their academic scores to be good throughout the years, whilst others remained the same and having no considerable improvement, confirming that location of where students study is a notable factor in their achievement.

### iii. Exploring Subject Choice

When referring to both sets of diagrams in **_Figure 1_**, it is evident that female candidates achieve higher than males, particularly in male-dominated optional subjects (namely Computer Science and Physical Education) where the differential is more apparent. This is further highlighted in **_Figure 2_** over a decade-long stretch between 2008 and 2018. Following this time period in the same diagram, male achievement saw a slight edge being gained.

Possible reasons for this shift in the general trend observed could be due to the effects caused in light of the COVID-19 global pandemic, which began towards the conclusion of 2019. All levels of the education system were forced to adapt by switching to an online approach to learning, resulting in different responses in overall engagement by candidates of both genders. Although **_Figure 1_** conveys that female achievement was still higher than that of their male counterparts during the pandemic, the results in **_Figure 2_** are more indicative of a greater ratio of male to female pupils being more successful across the board.

{|insights)}

{(designJustification|}

### i. Colour

The first two visualizations used data predominantly focused on statistics regarding male and female performance in the UK secondary education system. The choice of colours blue and pink helped dictate the different genders' datasets, providing ease of readability and a straight-forward understanding of what the visualization is trying to show. The past propaganda and media created a social norm that the following colours are assigned to these genders. An article from the University of Missouri-Kansas City states, _"girls were reassigned with pink... romantic colour, and women were seen as more emotional"_ **_(Michael, 2018)_**. The following denotes that the past influenced this social construct of blue for males and pink for females. Furthermore, recent trends like gender reveal further influence this ideology, therefore applying these colour coordinations for this data visualization is suitable. The audience or reader can easily interpret and understand the visualization shown. In terms of analysing the visualization, blue and pink contrast very well with each other as the two colours are significantly different from one another. The use of a white background makes the two chosen colourations pop out and easy to view.

In **_Figure 3_**, only one colour scheme is used to represent attainment scores across London. Through the testing of many colours, utilsing a multi-colour choropleth map in this case study seems inaccurate. The map would appear messy and obscure, making it difficult to understand which area did well and which did not. As a result, one colour spectrum is used to represent the attainment score per borough via its density (the higher the attaintment, the more dense). The colour scheme used is called "bluegreen" and seems appropriate as the colouration of green is used in secondary education attainment scores.

### ii. Interaction

The visualizations use filtering as the focal source of interactivity. A book by Tamara Munzer explores the reasoning behind the use of interaction in data visualization. It states, _"an interactive vis tool can support investigation at multiple levels of detail"_ **_(Munzner, 2014, p.9)_**, implying that the use of interaction enables readers to have the ability to explore and make a better interpretation of what the visualization is presenting. With the research question and dataset gathered, using interactivity is imperative. At first, providing no form of interactivity in the visualization presented issues, such as overlapping data. When viewing the visualization, it was unclear to identify which data belonged to which demographic (in the case, male or female) and what the results were. The use of legend filtering (**_Figure 1_**) helped overcome this issue; viewers can now select the data they want to see and examine, exploring the visualization to obtain a better conclusion. Also, a tooltip was included for each of the figures to ensure viewers knew what results were found. The last filtering tool used was the time scroller in **_Figure 3_**. Having numerous choropleth maps seemed unnecessary and difficult to analyse, implementing a filtering interaction helped reduce the visual cluttering, further allowing users to witness the progression throughout the years.

### iii. Layout

The overview of the graphs. Figures **_1_** and **_2_** explore the following factors: gender performance in GCSE and subject choices, using quantitative and temporal datasets. The choropleth map in **_Figure 3_** examines the progression of attainment throughout the years in London boroughs to help answer if a student's location is a factor which can affect their achievement.

**_Figure 1_** implements a line graph and applies some concepts defined in Tufte's minimal design principle. The use of this type of graph in this case study is very suitable as can be _"used to track changes over short and long periods of time"_ **_(www.betterevaluation.org, n.d.)_**. With this, gender performance can be analysed over an elongated time period, allowing viewers to see patterns and make predictions about future results based on the data. As mentioned, Tufte's minimal design principles were applied to this visualization. The reasoning behind this approach is to provide readers with an immediate interpretation of what the graph is showing before they can investigate. I believe this method would give new ideas about why the result portrays this and even perhaps answer why females are generally performing better than males in education. The abstraction of unnecessary details and ink on the page (such as axis lines and the values of the y-axis) leaves viewers with only necessary information. From this, I have achieved Tufte's minimal design principle. Finally, I have used a juxtaposition layout. As the subjects can be divided into two sections (core and elective), showcasing the core modules in one column and the optios in another. This made it clearer to view and evaluate the data in each of the graphs.

**_Figure 2_** uses a bar chart to highlight the total number of GCSE certificates awarded throughout the years. A bar chart enabled me to group the nominal data (by gender) to showcase students' overall achievement over the same time period. Once grouped, the values were added together to form another specification that sums the achieved number of certificates. In terms of layout, the visualization is placed under the line graphs and the two link with one another, therefore, ensuring there is consistency.

**_Figure 3_** is a choropleth map, as previously mentioned. In terms of its layout, a legend of the colour scheme is included to provide viewers with information on the attainment score throughout the years the data covers - a denser shade correlates to a higher attainment score, making it easier to differentiate and contrast this statistic amongst adjacent boroughs to answer the related research question. Since there is no direct visual link to the proceeding two visualizations, it was placed at the very end.

{|designJustification)}

{(references|}

Michael, M. (2018). Sexism in Colors – Why is Pink for Girls and Blue for Boys? | UMKC Women’s Center. [online] info.umkc.edu. Available at: https://info.umkc.edu/womenc/2018/06/25/8369/

Munzner, T. (2014). Visualization Analysis and Design. [online] Google Books. CRC Press. Available at: https://books.google.co.uk/books?hl=en&lr=&id=NfkYCwAAQBAJ&oi=fnd&pg=PP1&dq=Munzner

www.betterevaluation.org. (n.d.). Line graph | BetterEvaluation. [online] Available at: https://www.betterevaluation.org/methods-approaches/methods/line-graph

{|references)}
