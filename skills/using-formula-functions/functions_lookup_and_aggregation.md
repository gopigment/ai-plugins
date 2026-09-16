# Lookup & Aggregation Functions

## Lookup Functions

### ITEM

`ITEM(ValueToFind, Dimension.'UniqueProperty')`

- Searches a **unique** Property; returns matching Dimension item or BLANK
- Property name can be omitted (uses Default Property)
- Faster than MATCH for unique lookups

```pigment
ITEM("ben@corp.com", 'Employees'.'Email')
ITEM('Transaction'.'ProductCode', 'Products'.'Code')
```

### MATCH

`MATCH(ValueToMatch, Expression)`

- Expression can be Metric, List, or List Property; **does not need to be unique**
- If Metric, must be on **only one** Dimension List
- Returns **first** match by item ordering; BLANK if none
- Both args must be same data type
- **Case-sensitive** (unlike Excel MATCH); use LOWER/TRIM for normalization

```pigment
MATCH('Order'.'ProductName', 'Products'.'Name')
MATCH(LOWER(TRIM('Import'.'Code')), LOWER(TRIM('Products'.'Code')))
```

### SHIFT

`SHIFT(Block, Offset)`

- Returns a **Dimension item**, not a value
- Positive offset = shift ahead N items; negative = shift behind
- Offset can be dynamic (Integer Metric)
- Out-of-range shifts → BLANK

```pigment
SHIFT('Employee'.'Start Month', 1)
SHIFT(Month, -3)
```

### TIMEDIM

`TIMEDIM(Date, TimeDimension)`

- Converts a Date into a Calendar Dimension item
- TimeDimension: Day, Week, Month, Quarter, Half, Year
- Respects fiscal calendar settings
- Belongs in a stored list property, not inline in a BY modifier (except if date is stored in Metrics)

```pigment
TIMEDIM('Transactions'.'Date', Month)
TIMEDIM('Employee'.'Start Date', Quarter)

// On a transaction list, this formula lives on a Month property (type: Dimension → Month).
// Metric formulas then reference the stored property:
'Orders'.'Amount'[BY: 'Orders'.'Month']
```

### ITEM vs MATCH

| | ITEM | MATCH |
| --- | --- | --- |
| Property | Must be unique | Any (first match) |
| Performance | Faster | Slower |
| Case sensitivity | Depends on property | **Case-sensitive** |
| Use when | You have a unique key | Non-unique or expression-based lookup |

---

## Aggregation Functions

All `*OF` functions collapse **all dimensions** to a single scalar. For dimension-aware aggregation, use modifiers (`[BY SUM:]`, `[REMOVE AVG:]`, etc.) instead.

### SUMOF

`SUMOF(Block)` → Number. Sums all numbers; blanks ignored.

```pigment
SUMOF('Revenue')
```

### AVGOF

`AVGOF(Block)` → Number. Average of non-blank values.

```pigment
AVGOF('Revenue')
```

### MINOF / MAXOF

`MINOF(Block)` / `MAXOF(Block)` → same type as input (Number/Integer/Date). Blanks ignored.

```pigment
MINOF('Price')
MAXOF('End Date')                 // latest date
```

### COUNTOF

`COUNTOF(Block)` → Integer. On a **List**, counts all items. On a **Property/Metric**, counts non-blank values only.

```pigment
COUNTOF('Revenue')                // items with revenue
COUNTOF(Employee)                 // all employees (list = all items)
```

### COUNTALLOF / COUNTBLANKOF / COUNTUNIQUEOF

- `COUNTALLOF(Block)` → Integer. Counts all items including blanks.
- `COUNTBLANKOF(Block)` → Integer. Counts blank values only. Lists always return 0.
- `COUNTUNIQUEOF(Block)` → Integer. Counts unique non-blank values.

```pigment
COUNTBLANKOF('Price')             // items missing price
COUNTUNIQUEOF('Product'.'Category')
```

---

## OF Functions vs Modifiers

| Scenario | Use | Example |
| --- | --- | --- |
| Single total | OF function | `SUMOF('Revenue')` |
| Aggregate by dimension | Modifier | `'Revenue'[REMOVE: Product]` |
| Non-SUM by dimension | Modifier | `'Price'[REMOVE AVG: Product]` |
| Min/Max by dimension | Modifier | `'Price'[REMOVE MIN: Product]` |
| Unique count by dimension | Modifier | `'Customer'[REMOVE COUNTUNIQUE: Order]` |

**Rule**: Prefer modifiers for dimensional control. Use OF functions only for a single aggregate value.

---

## Common Patterns

```pigment
// Missing data ratio
COUNTBLANKOF('Price') / COUNTALLOF('Price')

// Count active customers
COUNTOF('Revenue'[SELECT: 'Customer'.'IsActive'])

// Average by department (modifier, not OF)
'Salary'[BY AVG: Employee.Department]

// Unique customers by month (modifier)
'Order'.'Customer'[REMOVE COUNTUNIQUE: 'Order'][BY: Month]
```
