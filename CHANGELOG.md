# Changelog

## 0.39.0

### Fixed

- `allocate` hands the leftover minor units to the shares with the largest
  remainders instead of to the first shares. `59.97` USD as `3:7` is now
  `17.99` and `41.98` (was `18.00` and `41.97`), `0.03` as `1:100` is `0.00`
  and `0.03`, and a zero ratio never receives money.
- `allocate` refuses a negative ratio. `allocate(1.00 USD, [2, -1])` used to
  return `[2.00, -1.00]`.
- `split` and `allocate` refuse an amount with more decimal places than its
  currency (from `mul`) instead of returning parts that do not sum back to it.
- A value that is not an amount (`{ amount: 1, currency: "USD" }`, `null`, a
  number) is refused by every function instead of being computed on.
- A Float amount, factor or ratio is refused with a message naming the function
  and saying to write a Decimal, instead of an error from inside `decimal()`.
- `parse` of text that is not a number raises a `money` error instead of a
  `parse` error.

### Added

- Every `money` error carries a `reason`: `bad_currency`, `unknown_currency`,
  `bad_amount`, `too_precise` or `currency_mismatch`.
- `allocate` takes Decimal ratios (`[0.3m, 0.7m]`).

### Breaking

- An empty or all-zero ratio list, and a `split` count that is not a positive
  Int, raise kind `bug` (reason `bad_argument`) instead of kind `money`.
  Migration: these are mistakes in the call; fix the arguments rather than
  catching them. Code matching `kind == "money"` for them must match `"bug"`.
- `split` and `allocate` refuse an amount finer than its currency.
  Migration: round it first with `money.of(m.amount, m.currency, rounding: true)`.
- Every function refuses a map whose `amount` is not a Decimal or whose
  `currency` is not a known uppercase code (`{ amount: 1, currency: "USD" }`
  used to be accepted and returned an Int amount).
  Migration: build amounts with `of`, `parse`, `zero` or `from_minor`.
- `normalize_code` raises kind `bug` for a code that is not a string (it used
  to convert it). Migration: pass the code as a string.
