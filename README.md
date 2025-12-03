# DAX for Sample Dashboard
## Build Calendar Lookup Table
### DAX Formula for Calendar Table

To create a Calendar Lookup table, click on the Modelling tab > New Table >  Insert the provided DAX.

```DAX
// Calendar table spanning data range between earliest stock and latest order dates
Calendar Lookup = 
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

### DAX to Create Year, Month Columns


```DAX
// Returns the last day of the year for each calendar date
EOY =
    ENDOFYEAR ( 'Calendar Lookup'[Date] )
```

```DAX
// Returns the month-end date to support text labels
'EOM (text)' =
    ENDOFMONTH ( 'Calendar Lookup'[Date] )
```

```DAX
// Returns the month-end date for numerical sorting purposes
'EOM (number)' =
    ENDOFMONTH ( 'Calendar Lookup'[Date] )
```

```DAX
// Returns the quarter-end date for each calendar date
EOQ =
    ENDOFQUARTER ( 'Calendar Lookup'[Date] )
```

```DAX
// Extracts the calendar year from the date
Year =
    YEAR ( 'Calendar Lookup'[Date] )
```

```DAX
// Formats the calendar month as a three-letter abbreviation
'Month (text)' =
    FORMAT ( [Date], "mmm" )
```

```DAX
// Extracts the month number from the date
'Month (number)' =
    MONTH ( [Date] )
```

```DAX
// Extracts the day number from the date
Day =
    DAY ( [Date] )
```
			


### 1. Key Metric Measures
```DAX
// Calculates total units sold in the current filter context
1. Unit Sold =
    SUM ( 'Sales Data 2022'[OrderQuantity] )
```

```DAX
// Calculates total revenue in the current filter context
2. Revenue =
    SUM ( 'Sales Data 2022'[Revenue] )
```

```DAX
// Calculates total profit in the current filter context
3. Profit =
    SUM ( 'Sales Data 2022'[Profit] )
```

## 2. YoY Metric Measures
```DAX
// Computes year-over-year change for unit sales
1.1 Unit Sold YoY % =
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
// Computes year-over-year change for revenue
2.1 Revenue YoY % =
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
// Computes year-over-year change for profit
3.1 Profit YoY % =
    VAR _Current = [3. Profit]
    VAR _Previous =
        CALCULATE(
            [3. Profit],
            DATEADD ( 'Calendar Lookup'[Date], -1, YEAR )
        )
    RETURN
    DIVIDE ( _Current - _Previous, _Previous )
```

## 3. MoM Metric Measures
```DAX
// Computes month-over-month change for unit sales
1.2 Unit Sold MoM % =
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
// Computes month-over-month change for revenue
2.2 Revenue MoM % =
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
// Computes month-over-month change for profit
3.2 Profit MoM % =
    VAR _CurrentMonth = [3. Profit]
    VAR _PreviousMonth =
        CALCULATE(
            [3. Profit],
            DATEADD ( 'Calendar Lookup'[Date], -1, MONTH )
        )
    RETURN
    DIVIDE ( _CurrentMonth - _PreviousMonth, _PreviousMonth )
```

## 4. YoY Conditional Formating Color for Metrics 
```DAX
// Sets YoY color for unit sales metric
1.3 Unit Sold % YoY Color =
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
// Sets YoY color for revenue metric
2.3 Revenue % YoY Color =
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
// Sets YoY color for profit metric
3.3 Profit % YoY Color =
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

## 5. MoM Conditional Formating Color for Metrics 

```DAX
// Sets MoM color for unit sales metric
1.4 Unit Sold % MoM Color =
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
// Sets MoM color for revenue metric
2.4 Revenue % MoM Color =
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
// Sets MoM color for profit metric
3.4 Profit % MoM Color =
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

## 6. Placeholder for Column Chart

```DAX
// Provides an upper bound buffer for unit time-series visuals
1.5 Upper Bound Time Series =
    VAR _Max =
        MAXX (
            ALLSELECTED ( 'Calendar Lookup'[EOM (number)] ),
            [1. Unit Sold]
        )
    RETURN
    _Max + _Max * 0.2
```

```DAX
// Provides an upper bound buffer for revenue time-series visuals
2.5 Upper Bound Time Series =
    VAR _Max =
        MAXX (
            ALLSELECTED ( 'Calendar Lookup'[EOM (number)] ),
            [2. Revenue]
        )
    RETURN
    _Max + _Max * 0.2
```

```DAX
// Provides an upper bound buffer for profit time-series visuals
3.5 Upper Bound Time Series =
    VAR _Max =
        MAXX (
            ALLSELECTED ( 'Calendar Lookup'[EOM (number)] ),
            [3. Profit]
        )
    RETURN
    _Max + _Max * 0.2
```








