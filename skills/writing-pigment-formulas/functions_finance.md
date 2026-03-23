# Financial Functions

Financial calculations for investment analysis and valuation.

**Covers**: NPV, XNPV, IRR, XIRR

---

## Quick Reference

| Function | Purpose                                   | Syntax Example                         |
| -------- | ----------------------------------------- | -------------------------------------- |
| **NPV**  | Net present value (periodic)              | `NPV(Rate, CashFlows, Period)`         |
| **XNPV** | Net present value (irregular dates)       | `XNPV(Rate, CashFlows, Dates)`         |
| **IRR**  | Internal rate of return (periodic)        | `IRR(CashFlows, Period, InitialGuess)` |
| **XIRR** | Internal rate of return (irregular dates) | `XIRR(CashFlows, Dates, InitialGuess)` |

---

## Net Present Value Functions

### NPV

Calculate NPV for periodic cash flows.

**Syntax**: `NPV(DiscountRate, CashFlows, Period)`

**Parameters**:

- **DiscountRate**: Annual discount rate (e.g., 0.1 for 10%)
- **CashFlows**: Metric with cash flow values
- **Period**: Time dimension (must be periodic: Month, Quarter, Year)

**Examples**:

```pigment
// Project NPV with 10% discount rate
NPV(0.10, 'Cash Flows', Year)

// Monthly cash flows with annual rate
NPV(0.12 / 12, 'Monthly Cash Flows', Month)

// Compare projects
IF(NPV(0.10, 'Project A Cash Flow', Year) > NPV(0.10, 'Project B Cash Flow', Year), "Project A", "Project B")
```

**Key Points**:

- Cash flows must be on a regular time dimension
- The initial investment (typically negative) must be included in the cash flows for the first period if it occurs at the same time
- Discount rate is per period (annual rate / periods per year)

---

### XNPV

Calculate NPV for cash flows on irregular dates.

**Syntax**: `XNPV(DiscountRate, CashFlows, Dates)`

**Parameters**:

- **DiscountRate**: Annual discount rate
- **CashFlows**: Metric or transaction list property with cash amounts
- **Dates**: Date or DateTime dimension/property

**Examples**:

```pigment
// Transaction-based NPV
XNPV(0.10, 'Transactions'.'Amount', 'Transactions'.'Date')

// Irregular project cash flows
XNPV(0.12, 'Cash Flow', 'Payment Date')

// NPV of investment portfolio
XNPV(0.08, 'Portfolio'.'Cash Flow', 'Portfolio'.'Transaction Date')
```

**When to Use**: Cash flows on irregular dates (not periodic time dimension).

---

## Internal Rate of Return Functions

### IRR

Calculate IRR for periodic cash flows.

**Syntax**: `IRR(CashFlows, Period, InitialGuess)`

**Parameters**:

- **CashFlows**: Metric with cash flow values
- **Period**: Time dimension (periodic)
- **InitialGuess**: Optional starting guess (default: 0.1)

**Examples**:

```pigment
// Project IRR
IRR('Cash Flows', Year, 0.1)

// Monthly cash flows
IRR('Monthly Cash Flows', Month, 0.01)

// Compare IRR to hurdle rate
IF(IRR('Project Cash Flows', Year, 0.1) > 0.15, "Accept", "Reject")
```

**Key Points**:

- Returns the discount rate where NPV = 0
- Cash flows must include at least one negative (investment) and one positive (return) value
- InitialGuess parameter is the starting value for the iterative calculation
- Returns BLANK if no solution is found after 200 iterations

---

### XIRR

Calculate IRR for cash flows on irregular dates.

**Syntax**: `XIRR(CashFlows, Dates, InitialGuess)`

**Parameters**:

- **CashFlows**: Metric or transaction property with cash amounts
- **Dates**: Date or DateTime dimension/property
- **InitialGuess**: Optional starting guess (default: 0.1)

**Examples**:

```pigment
// Transaction-based IRR
XIRR('Transactions'.'Amount', 'Transactions'.'Date', 0.1)

// Irregular investment returns
XIRR('Investment'.'Cash Flow', 'Investment'.'Date', 0.15)

// Portfolio IRR
XIRR('Portfolio'.'Cash Flow', 'Portfolio'.'Transaction Date', 0.12)
```

**When to Use**: Cash flows on irregular dates.

---

## Function Comparison

### NPV vs XNPV

| Aspect               | NPV                             | XNPV              |
| -------------------- | ------------------------------- | ----------------- |
| **Cash Flow Timing** | Periodic (Month, Quarter, Year) | Irregular dates   |
| **Use Case**         | Regular intervals               | Transaction-based |
| **Performance**      | Faster                          | Slower            |

### IRR vs XIRR

| Aspect               | IRR               | XIRR              |
| -------------------- | ----------------- | ----------------- |
| **Cash Flow Timing** | Periodic          | Irregular dates   |
| **Use Case**         | Regular intervals | Transaction-based |
| **Performance**      | Faster            | Slower            |

---

## Common Patterns

### Pattern 1: Project Evaluation

```pigment
// Calculate NPV and compare to threshold
IF(NPV(0.10, 'Project Cash Flows', Year) > 0, "Accept", "Reject")
```

### Pattern 2: Investment Decision

```pigment
// Compare IRR to hurdle rate
IF(IRR('Investment Cash Flows', Year, 0.1) > 'Hurdle Rate', "Invest", "Pass")
```

### Pattern 3: Portfolio Analysis

```pigment
// NPV of transaction list
XNPV(0.12, 'Transactions'.'Amount', 'Transactions'.'Date')[BY: Portfolio]
```

### Pattern 4: Monthly to Annual Rate Conversion

```pigment
// Monthly cash flows with annual discount rate
NPV('Annual Discount Rate' / 12, 'Monthly Cash Flows', Month)
```

---

## Critical Rules

- **First cash flow is typically negative** - Initial investment
- **NPV > 0 = positive return** - Above discount rate
- **IRR > hurdle rate = accept** - Project meets requirements
- **Discount rate is per period** - Adjust for time period
- **XNPV/XIRR for irregular dates** - More flexible but slower
- **InitialGuess helps convergence** - Use reasonable estimate
- **Cash flows must change sign** - At least one negative and one positive for IRR
- **Blank if no solution** - IRR/XIRR return BLANK if can't converge
