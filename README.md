# money - Ecko Std Lib Package

Currency amounts on `Decimal`: exact arithmetic, ISO 4217 minor units, and
allocation that never loses a cent.

Pure computation - no capabilities.

## Install

```bash
ecko get github.com/ecko-lang/money
```

```ecko
import money
```

## Usage

```ecko
price = money.parse("19.99", "USD")
total = money.mul(price, 3)          # 59.97 USD

money.to_string(total)               # "59.97 USD"
money.split(total, 4)                # four parts that sum back exactly
money.allocate(total, [3, 7])        # a 3:7 share, to the cent
```

## API

### Building

| function | what it does |
|---|---|
| `of(decimal, code)` | an amount from a Decimal |
| `parse(text, code)` | an amount from a string |
| `zero(code)` | zero in a currency |
| `from_minor(units, code)` | `1050` with USD becomes `10.50` |
| `to_minor(m)` | `10.50 USD` becomes `1050` |

### Arithmetic

`add`, `sub`, `mul(m, factor)`, `neg`, `abs_of`, `sum(items, code)`.

### Comparison

`compare`, `eq`, `lt`, `gt`, `is_zero`, `is_negative`.

### Allocation

| function | what it does |
|---|---|
| `split(m, n)` | `n` parts summing back exactly |
| `allocate(m, ratios)` | shares by ratio, summing back exactly |

### Currencies and rendering

`minor_units(code)`, `minor_unit_table()`, `normalize_code(code)`,
`to_string(m)`.

## Notes

**Amounts are Decimal, never float.** `0.10 + 0.20` is exactly `0.30` here. In
binary floating point it is not, and a cent lost per transaction is a ledger
that stops balancing. Conversions stay in decimal too: `to_minor` uses `round`,
which on a Decimal is exact and takes halves away from zero, and `to_string`
uses `fmt.fixed`, which rounds in decimal rather than through a float.

**Division hands out the remainder rather than rounding it away.** Splitting
`$0.10` three ways gives `0.04, 0.03, 0.03` - not three of `0.033`. The parts of
a split always sum back to what you started with, which is the property invoices,
refunds and revenue shares depend on. `allocate($0.05, [3, 7])` is `0.02` and
`0.03`: as close to 3:7 as whole cents allow, and still five cents in total.

**Not every currency has two decimal places.** Yen has none, Kuwaiti dinar has
three. Assuming two is the classic bug - it makes yen a hundred times too small.
Unknown codes default to two.

**Mixing currencies raises rather than converting.** Adding dollars to euros is
not a rounding question, it is a missing exchange rate. There are no rates here:
getting one means talking to something, and that is not a pure package.

**Negative amounts allocate like positive ones**, sign carried through, so a
refund splits the same way as the charge it reverses.

## Testing

```bash
ecko test
```

Offline and deterministic. The allocation tests assert that every split sums
back to the original across a range of awkward amounts and divisors.

## License

MIT
