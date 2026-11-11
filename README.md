# money

Currency amounts on `Decimal`: exact arithmetic, ISO 4217 minor units, and
allocation that never loses a cent. Pure computation - no capabilities.

## Install

```bash
ecko get github.com/ecko-lang/money
```

```ecko
import money
```

## Usage

```ecko
import money

price = money.parse("19.99", "USD")
total = money.mul(price, 3)
print(money.to_string(total))                              # 59.97 USD
print(map(money.split(total, 4), money.to_string))         # [15.00 USD, 14.99 USD, 14.99 USD, 14.99 USD]
print(map(money.allocate(total, [3, 7]), money.to_string)) # [17.99 USD, 41.98 USD]
print(money.to_minor(money.parse("1050", "JPY")))          # 1050

tax = money.mul(money.parse("10.00", "USD"), 0.0825m)
print(money.to_string(tax))                                # 0.83 USD

reason = try {
    money.add(price, money.parse("1.00", "EUR"))
} catch (e) { get(e, "reason") }
print(reason)                                              # currency_mismatch
```

An amount is a map `{ amount, currency }` with a `Decimal` amount and an
uppercase ISO 4217 code. Build one with `of`, `parse`, `zero` or `from_minor`;
every other function refuses a value that is not one.

## API

### Building

| function | returns |
|---|---|
| `of(amount, code, rounding: false)` | an amount from a Decimal, an Int or a numeric string |
| `parse(text, code, rounding: false)` | an amount from a string such as `"19.99"` |
| `zero(code)` | zero in a currency |
| `from_minor(units, code)` | an amount from an Int count of minor units: `1050` USD is `10.50` |
| `to_minor(m)` | the amount as an Int count of minor units: `10.50` USD is `1050` |

### Arithmetic

| function | returns |
|---|---|
| `add(a, b)` | `a + b`, same currency |
| `sub(a, b)` | `a - b`, same currency |
| `mul(m, factor)` | the amount scaled by an Int or Decimal factor |
| `neg(m)` | the amount with its sign flipped |
| `abs_of(m)` | the amount without its sign |
| `sum(items, code)` | the total of a list of amounts, zero in `code` when empty |

### Comparison

| function | returns |
|---|---|
| `compare(a, b)` | `-1`, `0` or `1` |
| `eq(a, b)` | whether the amounts are equal |
| `lt(a, b)` | whether `a < b` |
| `gt(a, b)` | whether `a > b` |
| `is_zero(m)` | whether the amount is exactly zero |
| `is_negative(m)` | whether the amount is below zero |

### Allocation

| function | returns |
|---|---|
| `split(m, n)` | a list of `n` amounts that sum back exactly |
| `allocate(m, ratios)` | a list of shares by ratio that sum back exactly |

### Currencies and rendering

| function | returns |
|---|---|
| `minor_units(code)` | the currency's number of decimal places |
| `minor_unit_table()` | the map of every known code to its decimal places |
| `normalize_code(code)` | the code uppercased, once its shape is checked |
| `to_string(m)` | `"10.50 USD"`, with the currency's own number of decimals |

## Notes

### Errors

| kind | reason | when |
|---|---|---|
| `money` | `bad_currency` | a currency code is not three letters |
| `money` | `unknown_currency` | a code is not an active ISO 4217 currency |
| `money` | `bad_amount` | `parse` or `of` is given text that is not a number |
| `money` | `too_precise` | an amount has more decimal places than its currency, in `of`/`parse` without `rounding: true`, or in `split`/`allocate` |
| `money` | `currency_mismatch` | two amounts in different currencies are combined or compared |
| `bug` | `bad_argument` | a value that is not an amount, a Float amount, factor or ratio, a negative or all-zero ratio list, or a non-positive `n` |

### Amounts are Decimal, never float

`0.10 + 0.20` is exactly `0.30` here. In binary floating point it is not, and a
cent lost per transaction is a ledger that stops balancing. So a Float amount,
factor or ratio is refused with a message saying to write a Decimal (`0.1m`).
Conversions stay in decimal too: `to_minor` uses `round`, which on a Decimal is
exact and takes halves away from zero, and `to_string` uses `fmt.fixed`, which
rounds in decimal rather than through a float.

### How allocation hands out the remainder

`split` and `allocate` work in the currency's minor units. Each share gets the
whole units its ratio earns; the units left over go one at a time to the shares
with the largest remainders, ties to the earlier share (the largest-remainder
method). `59.97` as `3:7` is `17.99` and `41.98`, a zero ratio never receives
anything, and the shares always sum back to the original. A negative amount
allocates like its positive, sign carried through, so a refund splits the same
way as the charge it reverses.

### Not every currency has two decimal places

Yen has none, Kuwaiti dinar has three, the Chilean UF has four. Assuming two is
the classic bug - it makes yen a hundred times too small. The package knows
every active ISO 4217 code and refuses one it does not know.

### An amount never has more places than its currency, until you ask

`parse("12.345", "USD")` is refused instead of being kept and rounded later,
out of sight. When rounding is what you mean, say so: `parse("12.345", "USD",
rounding: true)` is `12.35`, half away from zero. Arithmetic in between (`mul`
by a tax rate) may carry extra places; `to_string` and `to_minor` round those
at the end. `split` and `allocate` refuse them, because the parts could not sum
back to an amount that is not a whole number of minor units: round it first
with `money.of(m.amount, m.currency, rounding: true)`.

### Mixing currencies raises rather than converting

Adding dollars to euros is not a rounding question, it is a missing exchange
rate. There are no rates here: getting one means talking to something, and
that is not a pure package.

## Testing

```bash
ecko test
```

## License

[MIT](LICENSE)
