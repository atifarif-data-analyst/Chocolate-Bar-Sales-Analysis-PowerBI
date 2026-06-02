# 🧮 DAX Measures

These are the **actual measures authored in the `.pbix`**, extracted from the model. All measures live in a dedicated `Measurments` table. Field and table names follow the model in [`DATA_DICTIONARY.md`](DATA_DICTIONARY.md).

> Note on the model: `shipments[Sales]` is a stored column and `shipments[Cost]` is a **calculated column** defined as
> `RELATED(products[Cost_per_box]) * shipments[Boxes]`. Profit measures build on these.

---

## Core Measures

```dax
Total Sales = SUM ( shipments[Sales] )
```

```dax
Total Cost = SUM ( shipments[Cost] )
```

```dax
Total Profit = [Total Sales] - [Total Cost]
```

```dax
Profit % = DIVIDE ( [Total Profit], [Total Sales] )
```

```dax
Total Transactions = COUNTROWS ( shipments )
```

```dax
Total Boxes = SUM ( shipments[Boxes] )
```

---

## Low-Box-Size (LBS) Analysis

Flags small orders (fewer than 50 boxes) and computes their share of all transactions.

```dax
LBS Count =
CALCULATE ( [Total Transactions], shipments[Boxes] < 50 )
```

```dax
LBS % = [LBS Count] / [Total Transactions] * 100
```

---

## Time Intelligence

```dax
Previous Year Sales =
CALCULATE (
    [Total Sales],
    DATEADD ( 'calendar'[Date], -1, YEAR )
)
```

```dax
YoY Growth % =
VAR CurrentYTD =
    TOTALYTD ( [Total Sales], 'calendar'[Date] )
VAR PreviousYTD =
    CALCULATE (
        TOTALYTD ( [Total Sales], 'calendar'[Date] ),
        SAMEPERIODLASTYEAR ( 'calendar'[Date] )
    )
RETURN
    IF (
        PreviousYTD > 0,
        DIVIDE ( CurrentYTD - PreviousYTD, PreviousYTD ),
        BLANK ()
    )
```

```dax
MoM Growth % =
VAR CurrentMonth = [Total Sales]
VAR Previous_Month =
    CALCULATE (
        [Total Sales],
        DATEADD ( 'calendar'[Date], -1, MONTH ),
        ALL ( 'calendar' )
    )
RETURN
    IF (
        ISBLANK ( Previous_Month ) || Previous_Month = 0,
        BLANK (),
        DIVIDE ( CurrentMonth - Previous_Month, ABS ( Previous_Month ) )
    )
```

```dax
Latest Date = LASTDATE ( 'calendar'[Start of Month] )
```

```dax
Latest Month Sales =
VAR ld = [Latest Date]
RETURN
    CALCULATE ( [Total Sales Card], 'calendar'[Start of Month] = ld )
```

```dax
Latest MoM Sales Change % =
VAR ld = [Latest Date]
VAR this_month_sales = [Latest Month Sales]
VAR prev_month_sales =
    CALCULATE ( [Total Sales Card], 'calendar'[Start of Month] = EDATE ( ld, -1 ) )
RETURN
    DIVIDE ( this_month_sales - prev_month_sales, prev_month_sales )
```

```dax
Current Year Sales =
CALCULATE (
    [Total Sales Card],
    'calendar'[Year] = MAX ( 'calendar'[year] )
)
```

---

## Order-Status Measures

```dax
Total Delivered =
CALCULATE ( COUNTROWS ( shipments ), shipments[Order_Status] = "Delivered" )
```

```dax
Total Shipped =
CALCULATE ( COUNTROWS ( shipments ), shipments[Order_Status] = "Shipped" )
```

```dax
Total Placed =
CALCULATE ( COUNTROWS ( shipments ), shipments[Order_Status] = "Placed" )
```

```dax
Total Cancelled =
CALCULATE ( COUNTROWS ( shipments ), shipments[Order_Status] = "Cancelled" )
```

```dax
Cancellation % =
DIVIDE ( [Total Cancelled], [Total Transactions] )
```

```dax
Cancelled Boxes =
CALCULATE ( SUM ( shipments[Boxes] ), shipments[Order_Status] = "Cancelled" )
```

---

## Status Flag

```dax
Loss Flag =
IF ( [Profit %] < 0, "⚠️ Loss Making", "✅ Profitable" )
```

---

## Dynamic KPI Card Measures

The KPI banner uses emoji-prefixed text measures so each card renders an icon, label, and formatted value in a single string.

```dax
Total Sales Card =
"💰 Total Sales " & FORMAT ( SUM ( shipments[Sales] ), "#,0" )
```

```dax
Total Cost Card =
"💲 Total Cost " & FORMAT ( SUM ( shipments[Cost] ), "#,0" )
```

```dax
Total Profit Card =
"💎 Total Profit " & FORMAT ( [Total Profit], "#,0" )
```

```dax
Profit% Card =
"📊 Profit % " & FORMAT ( [Profit %], "0.0%" )
```

```dax
Transactions Card =
"🛒 Transactions " & FORMAT ( [Total Transactions], "#,0" )
```

```dax
YoY Growth % Card =
"📈 YoY Growth % " & FORMAT ( [YoY Growth %], "0.0%" )
```

```dax
Total Boxes Card =
"📦 Total Boxes " & FORMAT ( SUM ( shipments[Boxes] ), "0" )
```

```dax
KPICardWithEmoji =
"💰 Total Sales: " & FORMAT ( SUM ( shipments[Sales] ), "#,0" ) &
" 💲 Total Cost: " & FORMAT ( SUM ( shipments[Cost] ), "#,0" ) &
" 💎 Total Profit: " & FORMAT ( [Total Profit], "#,0" ) &
" 📊 Profit %: " & FORMAT ( [Profit %], "0.0%" ) &
" 🛒 Transactions: " & [Total Transactions] &
" 📈 YoY Growth %: " & FORMAT ( [YoY Growth %], "0.0%" ) &
" 📦 LBS %: " & FORMAT ( [LBS %], "0.0%" )
```

---

*Extracted directly from `Chocolate_Bar_Sales.pbix`. 31 measures total across core metrics, time intelligence, order-status splits, and dynamic card text.*
