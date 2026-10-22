# Changelog — fin-toml

All notable changes to the fin-toml TOML parser/encoder.

## [0.1.0] — 2026-10-22

### Added
- TOML v1.0 parser (`toml::fin`): key/value pairs, dotted keys, `[table]`
  and `[[array-of-tables]]` headers, basic `"..."` (incl. multiline
  `"""..."""` with escapes and line-ending-backslash continuation) and
  literal `'...'` (incl. multiline) strings, integers (decimal, `0x`/`0o`/
  `0b`, underscores, signs), floats (exponents, `inf`/`nan`), booleans,
  datetimes (kept as raw text — Fin has no date type in std), arrays
  (nested, multiline, trailing commas, comments), and inline tables.
- Duplicate-key, redefined-table, and dotted-key-table redefinition
  rules enforced (TOML forbids redefinition); errors carry line numbers
  and never abort parsing (`TomlResult` with `ok=false`).
- Typed getters by dotted path (`toml_get_string/int/float/bool`, with
  defaults; datetimes read through the string getter), table/array
  accessors, and a canonical TOML encoder (`toml_encode`) with verified
  parse → encode → parse round-trips.
- `toml_parse_file` via `fs::std`.
- 85-assertion test suite (all passing on finc 0.4.0, contract 3) and
  two runnable examples.

### Notes
- Datetimes are kind 6 with the raw text in `s`: no silent loss of
  offset/fraction, and no fake date arithmetic. `toml_get_string` reads
  them (a string getter over raw text); `toml_get_int/float/bool` do not
  coerce them.
- A space between date and time (`1979-05-27 07:32:00`) is supported:
  the bare-token reader extends a date-shaped token across spaces when a
  time follows.
- Inline tables (flag bit 4) cannot be extended from outside; tables
  created by dotted keys (bit 2) cannot be redefined by a `[header]`;
  implicit header-path tables (bit 0) can be defined later.
