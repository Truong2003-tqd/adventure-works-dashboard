# DAX for Sample Dashboard

## DAX Expression Structure

This section provide a short explanation about the key components of DAX.

**Without VAR/RETURN**
```DAX
Total Sales =
    SUM ( 'Sales Data 2022'[Revenue] )
```

**With VAR/RETURN**
```DAX
Total Sales =
    VAR CurrentYear = YEAR ( TODAY() )
    RETURN
        CALCULATE ( SUM ( 'Sales Data 2022'[Revenue] ), 'Calendar Lookup'[Year] = CurrentYear )
```
- `VAR` defines a temporary value that can be reused within the measure, helping break complex logic into readable steps.
- `RETURN` marks the expression that produces the final result, often combining previously declared variables.
- Measures without `VAR`/`RETURN` evaluate directly in a single expression, which works for simple aggregations; using `VAR`/`RETURN` adds clarity and efficiency when intermediate calculations are needed.

## Build Calendar Lookup Table

### DAX Formula for Calendar Table

To create a Calendar Lookup table, click on the Modelling tab > New Table >  Insert the provided DAX.

```DAX
Calendar Lookup =
    -- Calendar table spanning data range between earliest stock and latest order dates
VAR _MinYear =
    -- Get the earliest year in the dataset
    -- Using StockDate because inventory typically starts before sales
    YEAR( MIN( 'Sales Data 2022'[StockDate] ) )
VAR _MaxYear =
    -- Get the latest year in the dataset
    -- Using OrderDate because sales activity usually extends further than stock
    YEAR( MAX( 'Sales Data 2022'[OrderDate] ) )
RETURN
    -- Generate a continuous list of daily dates
    -- The calendar starts from January 1st of the earliest year
    -- and ends on December 31st of the latest year
    CALENDAR(
        DATE( _MinYear, 1, 1 ),
        DATE( _MaxYear, 12, 31 )
    )
```

```DAX
EOY =
    -- Returns the last day of the year for each calendar date
    ENDOFYEAR ( 'Calendar Lookup'[Date] )
```

```DAX
'EOM (text)' =
    -- Returns the month-end date to support text labels
    ENDOFMONTH ( 'Calendar Lookup'[Date] )
```

```DAX
'EOM (number)' =
    -- Returns the month-end date for numerical sorting purposes
    ENDOFMONTH ( 'Calendar Lookup'[Date] )
```

```DAX
EOQ =
    -- Returns the quarter-end date for each calendar date
    ENDOFQUARTER ( 'Calendar Lookup'[Date] )
```

```DAX
Year =
    -- Extracts the calendar year from the date
    YEAR ( 'Calendar Lookup'[Date] )
```

```DAX
'Month (text)' =
    -- Formats the calendar month as a three-letter abbreviation
    FORMAT ( [Date], "mmm" )
```

```DAX
'Month (number)' =
    -- Extracts the month number from the date
    MONTH ( [Date] )
```

```DAX
Day =
    -- Extracts the day number from the date
    DAY ( [Date] )
```
			


### 1. Key Metric Measures
```DAX
1. Unit Sold =
    -- Calculates total units sold in the current filter context
    SUM ( 'Sales Data 2022'[OrderQuantity] )
```

```DAX
2. Revenue =
    -- Calculates total revenue in the current filter context
    SUM ( 'Sales Data 2022'[Revenue] )
```

```DAX
3. Profit =
    -- Calculates total profit in the current filter context
    SUM ( 'Sales Data 2022'[Profit] )
```

### 2. YoY Metric Measures

The `1.1 Unit Sold YoY %` measure subtracts last year's unit total from the current total and divides by last year's value, giving the percentage change. The revenue (`2.1`) and profit (`3.1`) YoY measures reuse the same structure, just referencing their respective base measures.

Mathematically:
```
YoY % = (UnitsCurrentYear - UnitsPreviousYear) / UnitsPreviousYear
```
`VAR _Current` captures the current-period unit total once so it can be reused without recalculating, while `VAR _Previous` evaluates the same measure in a shifted time context (one year back via `DATEADD`). `RETURN` then outputs the final percentage change using those stored values, keeping the measure readable and efficient.

```DAX
1.1 Unit Sold YoY % =
    -- Computes year-over-year change for unit sales
    VAR _Current = [1. Unit Sold]
    VAR _Previous =
        CALCULATE(
            [1. Unit Sold],
            DATEADD ( 'Calendar Lookup'[Date], -1, YEAR )
        )
    RETURN
    DIVIDE ( _Current - _Previous, _Previous )
```

```DAX
2.1 Revenue YoY % =
    -- Computes year-over-year change for revenue
    VAR _Current = [2. Revenue]
    VAR _Previous =
        CALCULATE(
            [2. Revenue],
            DATEADD ( 'Calendar Lookup'[Date], -1, YEAR )
        )
    RETURN
    DIVIDE ( _Current - _Previous, _Previous )
```

```DAX
3.1 Profit YoY % =
    -- Computes year-over-year change for profit
    VAR _Current = [3. Profit]
    VAR _Previous =
        CALCULATE(
            [3. Profit],
            DATEADD ( 'Calendar Lookup'[Date], -1, YEAR )
        )
    RETURN
    DIVIDE ( _Current - _Previous, _Previous )
```

### 3. MoM Metric Measures

The `1.2 Unit Sold MoM %` measure compares the current month’s unit total with the previous month to express the change as a percentage. The revenue (`2.2`) and profit (`3.2`) MoM measures follow the same structure with their respective base metrics.

Mathematically:
```
MoM % = (UnitsCurrentMonth - UnitsPreviousMonth) / UnitsPreviousMonth
```
`VAR _CurrentMonth` captures the current-period value once, while `VAR _PreviousMonth` evaluates the same measure in the context shifted back one month via `DATEADD`. `RETURN` then feeds these stored values into `DIVIDE`, which safely handles division and keeps the calculation readable.
```DAX
1.2 Unit Sold MoM % =
    -- Computes month-over-month change for unit sales
    VAR _CurrentMonth = [1. Unit Sold]
    VAR _PreviousMonth =
        CALCULATE(
            [1. Unit Sold],
            DATEADD ( 'Calendar Lookup'[Date], -1, MONTH )
        )
    RETURN
    DIVIDE ( _CurrentMonth - _PreviousMonth, _PreviousMonth )
```

```DAX
2.2 Revenue MoM % =
    -- Computes month-over-month change for revenue
    VAR _CurrentMonth = [2. Revenue]
    VAR _PreviousMonth =
        CALCULATE(
            [2. Revenue],
            DATEADD ( 'Calendar Lookup'[Date], -1, MONTH )
        )
    RETURN
    DIVIDE ( _CurrentMonth - _PreviousMonth, _PreviousMonth )
```

```DAX
3.2 Profit MoM % =
    -- Computes month-over-month change for profit
    VAR _CurrentMonth = [3. Profit]
    VAR _PreviousMonth =
        CALCULATE(
            [3. Profit],
            DATEADD ( 'Calendar Lookup'[Date], -1, MONTH )
        )
    RETURN
    DIVIDE ( _CurrentMonth - _PreviousMonth, _PreviousMonth )
```

### 4. YoY Conditional Formating Color for Metrics 

The `1.3 Unit Sold % YoY Color` measure assigns display colors based on the sign of the YoY percentage change, yielding green for growth, red for decline, and grey otherwise. The revenue (`2.3`) and profit (`3.3`) color measures mirror this logic using their own YoY metrics.

`VAR _Metric` stores the YoY percentage for quick reuse, the color hex codes are declared once, and `RETURN` passes them into `SWITCH(TRUE())`, which evaluates each condition in order to output the correct color string.
```DAX
1.3 Unit Sold % YoY Color =
    -- Sets YoY color for unit sales metric
    VAR _Metric = [1.1 Unit Sold YoY %]
    VAR _Red = "#B2292E"
    VAR _Green = "#29B269"
    VAR _Grey = "#d1d1d1"
    RETURN
    SWITCH (
        TRUE (),
        _Metric > 0, _Green,
        _Metric < 0, _Red,
        _Grey
    )
```

```DAX
2.3 Revenue % YoY Color =
    -- Sets YoY color for revenue metric
    VAR _Metric = [2.1 Revenue YoY %]
    VAR _Red = "#B2292E"
    VAR _Green = "#29B269"
    VAR _Grey = "#d1d1d1"
    RETURN
    SWITCH (
        TRUE (),
        _Metric > 0, _Green,
        _Metric < 0, _Red,
        _Grey
    )
```

```DAX
3.3 Profit % YoY Color =
    -- Sets YoY color for profit metric
    VAR _Metric = [3.1 Profit YoY %]
    VAR _Red = "#B2292E"
    VAR _Green = "#29B269"
    VAR _Grey = "#d1d1d1"
    RETURN
    SWITCH (
        TRUE (),
        _Metric > 0, _Green,
        _Metric < 0, _Red,
        _Grey
    )
```

### 5. MoM Conditional Formating Color for Metrics 

The `1.4 Unit Sold % MoM Color` measure applies the same color rules as the YoY variant, but based on month-over-month changes. The revenue (`2.4`) and profit (`3.4`) versions simply reference their respective MoM metrics.

`VAR _Metric` captures the MoM percentage once, the color palette is defined in local variables, and `RETURN` routes the values through `SWITCH(TRUE())` so the first matching condition determines the color.

```DAX
1.4 Unit Sold % MoM Color =
    -- Sets MoM color for unit sales metric
    VAR _Metric = [1.2 Unit Sold MoM %]
    VAR _Red = "#B2292E"
    VAR _Green = "#29B269"
    VAR _Grey = "#d1d1d1"
    RETURN
    SWITCH (
        TRUE (),
        _Metric > 0, _Green,
        _Metric < 0, _Red,
        _Grey
    )
```

```DAX
2.4 Revenue % MoM Color =
    -- Sets MoM color for revenue metric
    VAR _Metric = [2.2 Revenue MoM %]
    VAR _Red = "#B2292E"
    VAR _Green = "#29B269"
    VAR _Grey = "#d1d1d1"
    RETURN
    SWITCH (
        TRUE (),
        _Metric > 0, _Green,
        _Metric < 0, _Red,
        _Grey
    )
```

```DAX
3.4 Profit % MoM Color =
    -- Sets MoM color for profit metric
    VAR _Metric = [3.2 Profit MoM %]
    VAR _Red = "#B2292E"
    VAR _Green = "#29B269"
    VAR _Grey = "#d1d1d1"
    RETURN
    SWITCH (
        TRUE (),
        _Metric > 0, _Green,
        _Metric < 0, _Red,
        _Grey
    )
```

### 6. Placeholder for Column Chart

The `1.5 Upper Bound Time Series` measure creates a padded upper limit for visuals by taking the filtered maximum value and adding a 20% buffer. The revenue (`2.5`) and profit (`3.5`) counterparts behave the same way for their metrics.

Mathematically:
```
UpperBound = MaxValue + MaxValue * 0.2
```
`VAR _Max` stores the peak value once (using `MAXX` across the selected date granularity), and `RETURN` adds the buffer so charts have consistent headroom without repeated calculations.

```DAX
1.5 Upper Bound Time Series =
    -- Provides an upper bound buffer for unit time-series visuals
    VAR _Max =
        MAXX (
            ALLSELECTED ( 'Calendar Lookup'[EOM (number)] ),
            [1. Unit Sold]
        )
    RETURN
    _Max + _Max * 0.2
```

```DAX
2.5 Upper Bound Time Series =
    -- Provides an upper bound buffer for revenue time-series visuals
    VAR _Max =
        MAXX (
            ALLSELECTED ( 'Calendar Lookup'[EOM (number)] ),
            [2. Revenue]
        )
    RETURN
    _Max + _Max * 0.2
```

```DAX
3.5 Upper Bound Time Series =
    -- Provides an upper bound buffer for profit time-series visuals
    VAR _Max =
        MAXX (
            ALLSELECTED ( 'Calendar Lookup'[EOM (number)] ),
            [3. Profit]
        )
    RETURN
    _Max + _Max * 0.2
```
## Create Multiple DAX Quickly
Go to DAX Query view > In the Data tab on the right hand side, right click the Working Measure table > Quick Queries > Define new measure > The DAX Query view will appear this DAX, which is the template to create new measure.

``` 
// Write your formula below, select 'Run' to try it out. When ready, select 'Update model' to add the measure to your model.
DEFINE
	MEASURE ' Working Measure'[New Measure] = COUNTROWS(' Working Measure')

EVALUATE
	SUMMARIZECOLUMNS(
		"New Measure", [New Measure]
	)
```
**Explanation:** `DEFINE` lets you draft the measure against the chosen table before committing it, while `EVALUATE` runs a quick preview query (using `SUMMARIZECOLUMNS`) so you can verify the result in the DAX Query view prior to updating the model. `EVALUATE` is optional if you just want to update your measures.

Writing similar DAX logics multiple time is time-consuming, so the DAX Query acts as a code editor allowing updating and creating multiple all at once. Now, you can try to write 3 YoY measures by trying this provided DAX. Paste this DAX in the query pane, select **Update model with changes** and press **Update model** in the pop-up.

```
DEFINE
	MEASURE ' Working Measure'[4.1 Unit Sold YoY %] = -- Define current year total
		VAR _Current = [1. Unit Sold]

		-- Define previous year total
		VAR _Previous =
		CALCULATE(
			[1. Unit Sold],
			-- Set a condition that calculate the previous year total
			DATEADD(
				'Calendar Lookup'[Date],
				-1,
				YEAR
			)
		)
		RETURN
			DIVIDE(
				_Current - _Previous,
				_Previous
			)

	MEASURE ' Working Measure'[4.2 Revenue MoM %] = VAR _CurrentMonth = [2. Revenue]

		VAR _PreviousMonth =
		CALCULATE(
			[2. Revenue],
			DATEADD(
				'Calendar Lookup'[Date],
				-1,
				MONTH
			)
		)
		RETURN
			DIVIDE(
				_CurrentMonth - _PreviousMonth,
				_PreviousMonth
			)

	MEASURE ' Working Measure'[4.2 Profit MoM %] = VAR _CurrentMonth = [3. Profit]

		VAR _PreviousMonth =
		CALCULATE(
			[3. Profit],
			DATEADD(
				'Calendar Lookup'[Date],
				-1,
				MONTH
			)
		)
		RETURN
			DIVIDE(
				_CurrentMonth - _PreviousMonth,
				_PreviousMonth
			)
```

## Build Geographic and Product Paramenter for Drill Down
### Create Parameter

After inserting needed fields in the parameters, you will have a table with this DAX.

``` DAX
Geography and Product = {
    ("Continent", NAMEOF('Territory Lookup'[Continent]), 0),
    ("Country", NAMEOF('Territory Lookup'[Country]), 1),
    ("Region", NAMEOF('Territory Lookup'[Region]), 2),
    ("CategoryName", NAMEOF('Product Lookup'[CategoryName]), 3),
    ("SubcategoryName", NAMEOF('Product Lookup'[SubcategoryName]), 4),
    ("ModelName", NAMEOF('Product Lookup'[ModelName]), 5),
    ("ProductName", NAMEOF('Product Lookup'[ProductName]), 6)
}
```

This modified version adds a fourth argument that tags each entry as either `Geography` or `Product`, enabling the drill-down parameter to group fields by domain and let report viewers toggle between geographic and product hierarchies.


``` DAX
Geography and Product = {
    ("Continent", NAMEOF('Territory Lookup'[Continent]), 0, "Geography"),
    ("Country", NAMEOF('Territory Lookup'[Country]), 1, "Geography"),
    ("Region", NAMEOF('Territory Lookup'[Region]), 2, "Geography"),
    ("CategoryName", NAMEOF('Product Lookup'[CategoryName]), 3, "Product"),
    ("SubcategoryName", NAMEOF('Product Lookup'[SubcategoryName]), 4, "Product"),
    ("ModelName", NAMEOF('Product Lookup'[ModelName]), 5, "Product"),
    ("ProductName", NAMEOF('Product Lookup'[ProductName]), 6, "Product")
}
```





