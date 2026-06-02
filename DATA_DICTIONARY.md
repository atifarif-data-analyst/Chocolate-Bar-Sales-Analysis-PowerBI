# 📖 Data Dictionary

The source workbook feeds a star schema in the `.pbix`: one fact table (`shipments`), three dimension tables (`products`, `locations`, `people`), a `calendar` date table, and a `Measurments` table holding all DAX measures.

---

## Fact Table — `shipments`

25,076 rows of transactional shipment data.

| Field | Type | Description |
|---|---|---|
| `SPID` | Text (FK) | Salesperson ID → links to `people` dimension |
| `PID` | Text (FK) | Product ID → links to `products` dimension |
| `GID` | Text (FK) | Geography ID → links to `locations` dimension |
| `Shipdate` | Date | Date of shipment → links to `calendar[Date]` |
| `Sales` | Decimal | Sale amount (revenue) for the shipment |
| `Boxes` | Integer | Number of boxes shipped |
| `Order_Status` | Text | `Delivered`, `Shipped`, `Placed`, or `Cancelled` |
| `Cost` | Decimal | **Calculated column:** `RELATED(products[Cost_per_box]) * shipments[Boxes]` |

**Order status distribution:** Delivered 21,215 · Shipped 2,174 · Cancelled 1,215 · Placed 472

---

## Dimension — `products`

| Field | Type | Description |
|---|---|---|
| `PID` | Text (PK) | Product ID (`P01`–`P22`) |
| `Product` | Text | Product name (e.g. *Milk Bars*, *85% Dark Bars*) |
| `Category` | Text | Product category: `Bars`, `Bites`, or `Other` |
| `Cost_per_box` | Decimal | Unit cost per box (drives the `Cost` calculated column) |

22 distinct products across 3 categories.

---

## Dimension — `locations`

| Field | Type | Description |
|---|---|---|
| `GID` | Text (PK) | Geography ID (`G1`–`G6`) |
| `Geo` | Text | Country (India, USA, Canada, New Zealand, Australia, UK) |
| `Region` | Text | Region grouping: `APAC`, `Americas`, `Europe` |

---

## Dimension — `people`

| Field | Type | Description |
|---|---|---|
| `SPID` | Text (PK) | Salesperson ID (`SP01`–`SP25`) |
| `Sales_person` | Text | Salesperson full name |
| `Team` | Text | Sales team: `Yummies`, `Delish`, `Jucies`, `Tempo` |
| `Picture` | URL | Avatar image URL (used for visuals) |

25 salespeople across 4 teams.

---

## Date Table — `calendar`

| Field | Type | Description |
|---|---|---|
| `Date` | Date | Calendar date (one row per day); model's marked date table |
| `Start of Month` | Date | First day of the month (used by month-level measures) |
| `Month_num` | Integer | Month number (1–12) |
| `month_name` | Text | Month name (January … December) |
| `year` | Integer | Calendar year |
| `weekday_num` | Integer | Day-of-week number (1 = Monday) |
| `weekday_name` | Text | Day-of-week name |

Covers the full reporting window (Jan 2023 – Mar 2025) and is marked as the model's date table to enable time-intelligence DAX.

---

## Measures Table — `Measurments`

A dedicated table holding all 31 DAX measures (no data columns). See [`DAX_MEASURES.md`](DAX_MEASURES.md) for the full list.
