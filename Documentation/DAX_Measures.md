# DAX Measures

## Total Revenue
Total Revenue = SUM('Online Retail Data Set'[Revenue])

## Total Customers
Total Customers = DISTINCTCOUNT('Online Retail Data Set'[CustomerID])

## Total Orders
Total Orders = DISTINCTCOUNT('Online Retail Data Set'[InvoiceNo])

## Total Units Sold
Total Units Sold = SUM('Online Retail Data Set'[Quantity])

## Average Order value
Average Order value = DIVIDE('Online Retail Data Set'[Total Revenue],'Online Retail Data Set'[Total Orders])

## Cumulative Product Revenue % 
Cumulative Product Revenue % = 
VAR CurrentProductRevenue = [Total Revenue]
VAR CurrentProduct = SELECTEDVALUE('Online Retail Data Set'[Description])

-- 1. Total revenue across ALL products (respects external slicers like Date/Region)
VAR TotalRevenueAllProducts = 
    CALCULATE(
        [Total Revenue], 
        REMOVEFILTERS('Online Retail Data Set'[Description])
    )

-- 2. Running total across all products ranked higher
VAR SumOfHeavierProducts = 
    CALCULATE(
        [Total Revenue],
        FILTER(
            ALLSELECTED('Online Retail Data Set'[Description]),
            [Total Revenue] > CurrentProductRevenue || 
            ([Total Revenue] = CurrentProductRevenue && 'Online Retail Data Set'[Description] <= CurrentProduct)
        ),
        REMOVEFILTERS('Online Retail Data Set'[Description])
    )

RETURN
    DIVIDE(SumOfHeavierProducts, TotalRevenueAllProducts, 0)

## Cumulative Revenue %
Cumulative Revenue % = 
VAR TotalRevenue = CALCULATE([Total Revenue], ALL('Online Retail Data Set'[Country]))
VAR CurrentCountryRevenue = [Total Revenue]
VAR SumOfHeavierMarkets = 
    CALCULATE(
        [Total Revenue],
        FILTER(
            ALL('Online Retail Data Set'[Country]),
            [Total Revenue] >= CurrentCountryRevenue
        )
    )
RETURN
DIVIDE(SumOfHeavierMarkets, TotalRevenue, 0)

## MoM Growth %
MoM Growth % = 
VAR CurrentMonthRevenue = [Total Revenue]
VAR PreviousMonthRevenue = 
    CALCULATE(
        [Total Revenue], 
        DATEADD('Calendar'[Date], -1, MONTH)
    )
RETURN 
    DIVIDE(CurrentMonthRevenue - PreviousMonthRevenue, PreviousMonthRevenue)

One-Time Customer Revenue = [Total Revenue] - [Repeat Customer Revenue]

One-Time Customers = [Customers] - [Repeat Customers]

## Repeat Customer Rate
Repeat Customer Rate = 
DIVIDE([Repeat Customers], DISTINCTCOUNT('Online Retail Data Set'[CustomerID]),0)

## Repeat Customer Revenue
Repeat Customer Revenue = 
CALCULATE(
    [Total Revenue],
    FILTER(
        VALUES('Online Retail Data Set'[CustomerID]),
        CALCULATE(DISTINCTCOUNT('Online Retail Data Set'[InvoiceNo])) > 1
    )
)

## Repeat Customers
Repeat Customers = 
COUNTROWS(
    FILTER(
        VALUES('Online Retail Data Set'[CustomerID]),
        CALCULATE(DISTINCTCOUNT('Online Retail Data Set'[InvoiceNo])) > 1
    )
)
