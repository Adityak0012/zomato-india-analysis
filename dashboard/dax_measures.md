# Power BI: DAX measures

Create these in an empty table called `_Measures` (Home > Enter Data > name it `_Measures` > Load).
For each one: select `_Measures`, click **New measure**, paste one block, press Enter, then set the format shown.

Table names assumed (rename after loading):
`Restaurants` (zomato_india_clean), `RestaurantCuisines` (restaurant_cuisines),
`DimCity` (dim_city), `DimPriceRange` (dim_price_range), `DimCuisine` (dim_cuisine).

The "Check" column is what each card should show with no filters applied. If yours differs, something in the model is wrong.

## Core KPIs

| Measure | Format | Check |
|---|---|---|
| Total Restaurants | Whole number | 8,652 |
| Rated Restaurants | Whole number | 6,513 |
| Avg Rating | Decimal, 2 places | 3.35 |
| Median Cost for Two | Currency ₹, 0 places | 450 |
| Total Votes | Whole number | 1,187,163 |

```DAX
Total Restaurants = COUNTROWS ( Restaurants )
```
```DAX
Rated Restaurants = CALCULATE ( [Total Restaurants], Restaurants[is_rated] = 1 )
```
```DAX
Avg Rating = AVERAGE ( Restaurants[aggregate_rating] )
```
AVERAGE skips blanks, so unrated restaurants (blank rating) are left out automatically.
```DAX
Median Cost for Two = MEDIAN ( Restaurants[average_cost_for_two] )
```
Median, not average, because 5-star hotel restaurants (up to ₹8,000) pull the average up.
```DAX
Total Votes = SUM ( Restaurants[votes] )
```

## Service and visibility

| Measure | Format | Check |
|---|---|---|
| Online Delivery % | Percentage, 1 place | 28.0% |
| Table Booking % | Percentage, 1 place | 12.8% |
| Rated % | Percentage, 1 place | 75.3% |
| Rated 4+ % | Percentage, 1 place | 12.4% |

```DAX
Online Delivery % =
DIVIDE (
    CALCULATE ( [Total Restaurants], Restaurants[has_online_delivery] = 1 ),
    [Total Restaurants]
)
```
```DAX
Table Booking % =
DIVIDE (
    CALCULATE ( [Total Restaurants], Restaurants[has_table_booking] = 1 ),
    [Total Restaurants]
)
```
```DAX
Rated % = DIVIDE ( [Rated Restaurants], [Total Restaurants] )
```
```DAX
Rated 4+ % =
DIVIDE (
    CALCULATE ( [Rated Restaurants], Restaurants[aggregate_rating] >= 4 ),
    [Rated Restaurants]
)
```
```DAX
Median Votes = MEDIAN ( Restaurants[votes] )
```

## Comparison

```DAX
Avg Rating vs Overall =
[Avg Rating] - CALCULATE ( [Avg Rating], REMOVEFILTERS () )
```
REMOVEFILTERS() clears every filter, so the second part is always the overall average (3.35). Positive = better than average. Format: decimal, 2 places, with a + sign: `+0.00;-0.00;0.00`.
```DAX
City Rank by Rating =
IF (
    HASONEVALUE ( DimCity[city] ),
    RANKX ( ALL ( DimCity[city] ), [Avg Rating], , DESC, DENSE )
)
```

## Dynamic title text

```DAX
Selected City =
IF (
    ISFILTERED ( DimCity[city] ),
    CONCATENATEX ( VALUES ( DimCity[city] ), DimCity[city], ", " ),
    "All cities"
)
```

## Calculated columns (Restaurants table, not _Measures)

Select the `Restaurants` table, click **New column**, paste:
```DAX
Online Delivery = IF ( Restaurants[has_online_delivery] = 1, "Yes", "No" )
```
```DAX
Table Booking = IF ( Restaurants[has_table_booking] = 1, "Yes", "No" )
```
```DAX
Vote Band =
SWITCH (
    TRUE (),
    Restaurants[votes] <= 10, "01  Up to 10",
    Restaurants[votes] <= 50, "02  11-50",
    Restaurants[votes] <= 200, "03  51-200",
    Restaurants[votes] <= 1000, "04  201-1000",
    "05  1000+"
)
```
The number prefix keeps the bands in the right order on charts.
