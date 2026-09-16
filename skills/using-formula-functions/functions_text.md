# Text Functions

---

## Conversion

### TEXT

`TEXT(Value)` → converts Number/Integer to Text.

```pigment
TEXT(123.456)       // "123.456"
TEXT(-44.23)        // "-44.23"
```

### VALUE (alias: NUMBER)

`VALUE(TextToConvert)` or `NUMBER(TextToConvert)` → converts Text to Number. Returns BLANK if not numeric or BLANK input.

```pigment
VALUE("123.45")     // 123.45
VALUE("abc")        // BLANK
```

---

## Length & Extraction

### LEN

`LEN(Text)` → Integer character count (including spaces).

### LEFT / RIGHT

`LEFT(Text, Count)` / `RIGHT(Text, Count)`

- Count exceeds length → returns all characters
- Negative count → BLANK

```pigment
LEFT('Product'.'SKU', 2)
RIGHT('Product'.'SKU', 3)
LEFT("abc", 5)               // "abc"
```

### MID

`MID(Text, StartPosition, Count)`

- **1-based** indexing
- Position 0 or negative → BLANK
- Negative count → BLANK

```pigment
MID("ABC123DEF", 4, 3)       // "123"
```

---

## Case Transformation

| Function | Syntax | Example |
| --- | --- | --- |
| **LOWER** | `LOWER(Text)` | `LOWER("TEXT")` → "text" |
| **UPPER** | `UPPER(Text)` | `UPPER("text")` → "TEXT" |
| **PROPER** | `PROPER(Text)` | `PROPER("c@ss n1te")` → "C@Ss N1Te" |

PROPER capitalizes the first letter and any letter after a non-alphabetic character. All others lowercase.

---

## TRIM

`TRIM(Text)` → removes leading/trailing spaces; replaces multiple internal spaces with single space.

```pigment
TRIM("  hello   world  ")    // "hello world"
```

---

## Search Functions

**Default: NOT case sensitive** (except SUBSTITUTE).

### CONTAINS

`CONTAINS(TextToFind, TextToSearch [, StartPosition] [, IsCaseSensitive])`

- Parameter order: **substring first, haystack second**
- Default case insensitive
- Invalid start position (0 or negative) → BLANK

```pigment
CONTAINS("premium", 'Product'.'Description')
CONTAINS("A", "abc", 1, TRUE)    // FALSE (case sensitive)
```

### STARTSWITH

`STARTSWITH(StartText, TextToSearch [, IsCaseSensitive])`

```pigment
STARTSWITH("PRD", 'Product'.'SKU')
```

### ENDSWITH

`ENDSWITH(EndText, TextToSearch [, IsCaseSensitive])`

```pigment
ENDSWITH(".com", User.Email)
```

### FIND

`FIND(TextToFind, TextToSearch [, StartPosition] [, IsCaseSensitive])`

- Returns **1-based position** or **BLANK** if not found (not 0)
- Default case insensitive

```pigment
FIND("World", "Hello World")     // 7
FIND("x", "Hello")              // BLANK
```

---

## SUBSTITUTE

`SUBSTITUTE(TextToSearch, TextToReplace, ReplaceWith [, OccurrenceNumber])`

- **Case sensitive** (only text function that is)
- Default replaces **all** occurrences
- Specific occurrence: 4th arg
- Negative occurrence → BLANK

```pigment
SUBSTITUTE("aaa", "a", "b")     // "bbb" (all)
SUBSTITUTE("aaa", "a", "b", 2)  // "aba" (2nd only)
SUBSTITUTE("abc", "A", "d")     // "abc" (no match, case sensitive)
```

---

## & (Concatenation Operator)

`text1 & text2 & ...`

- Each operand must be **Text type**; wrap numbers with `TEXT()` explicitly
- Does NOT auto-convert numbers (unlike some contexts)

```pigment
'First Name' & " " & 'Last Name'
'Product'.'Code' & "-" & TEXT('Product'.'Version')
Country.Name & " (" & Country.Region & ")"
```

---

## Case Sensitivity Summary

| Function | Default | Configurable |
| --- | --- | --- |
| CONTAINS | Not case sensitive | Yes (4th arg) |
| STARTSWITH | Not case sensitive | Yes (3rd arg) |
| ENDSWITH | Not case sensitive | Yes (3rd arg) |
| FIND | Not case sensitive | Yes (4th arg) |
| **SUBSTITUTE** | **Case sensitive** | **No** |

---

## Common Patterns

```pigment
// Composite key
'Customer'.'Country' & "-" & TEXT('Customer'.'ID')

// Extract domain from email
MID(User.Email, FIND("@", User.Email) + 1, LEN(User.Email) - FIND("@", User.Email))

// Clean and uppercase
UPPER(TRIM('User'.'Email'))

// SKU prefix extraction
LEFT('Product'.'SKU', 3)
```
