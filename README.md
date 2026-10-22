# fin-toml

A TOML v1.0 parser and encoder for Fin. Pure Fin, zero dependencies.

## Why

Every Fin project already ships a `finn.toml`, and every Fin app needs
config files — but the ecosystem had no TOML parser. `fin-toml` fills
that gap: parse app configs, read typed values with defaults, and write
configs back out in canonical TOML.

## Install

With `finn` (once the registry lands) or by vendoring `src/toml.fin`:

```toml
[packages]
fin-toml = "M1778/fin-toml@0.1.0"
```

Then import:

```fin
import { toml_parse, toml_get_string, toml_get_int } from toml::fin;
```

## Quick example

```fin
import { toml_parse, toml_get_string, toml_get_int, toml_encode, TomlResult } from toml::fin;

fun main() <noret> {
    let r <TomlResult> = toml_parse("[server]\nhost = \"localhost\"\nport = 8080\n");
    if (!r.ok) { /* r.err.line, r.err.message */ return; }
    let host <string> = toml_get_string(r.root, "server.host", "localhost");
    let port <long> = toml_get_int(r.root, "server.port", 8080);
    let back <string> = toml_encode(r.root); // canonical TOML
}
```

See `examples/basic.fin` (parse + read + encode) and
`examples/config.fin` (load a file, read with defaults).

## API reference

### Parsing

| Function | Description |
|---|---|
| `toml_parse(text: string) <TomlResult>` | Parse a document. Never aborts: on failure `ok=false` with `err.line` / `err.message`. |
| `toml_parse_file(path: string) <TomlResult>` | Parse a file via `fs::std`. Unreadable file is an `ok=false` result. |
| `toml_free(v)` / `toml_free_result(r)` | Release a tree / result. |

### Values

`TomlValue` is a tagged struct tree (same shape as finson's `JsonValue`):
`kind` is `0=string, 1=int, 2=float, 3=bool, 4=array, 5=table,
6=datetime`. Use `toml_kind_name(kind)` for a readable name.

Constructors: `toml_str`, `toml_int`, `toml_float`, `toml_bool`,
`toml_datetime`, `toml_array`, `toml_table`.

### Getters (dotted paths, e.g. `"server.tls.enabled"`)

| Function | Description |
|---|---|
| `toml_get(root, path) <&TomlValue>` | Raw node, or `null` when missing. Quoted path segments (`"\"weird.key\".value"`) work for keys containing dots. |
| `toml_get_string(root, path, default)` | String value, or `default`. Datetimes read through here (raw text). |
| `toml_get_int(root, path, default: long)` | Integer value, or `default`. |
| `toml_get_float(root, path, default: double)` | Float value, or `default`. |
| `toml_get_bool(root, path, default: bool)` | Boolean value, or `default`. |
| `toml_table_get/has/len/keys`, `toml_table_set` | Direct table access. |
| `toml_array_len/get/push` | Direct array access. |

Wrong types fall back to `default` (no coercion: an int is not a
string, a datetime is not a float).

### Encoding

`toml_encode(root) <string>` emits canonical TOML: scalar/array entries
first, then sub-tables under `[path]` headers and arrays-of-tables under
`[[path]]` headers. Parse → encode → parse round-trips preserve
semantics (not formatting or comments).

## What it parses

- Key/value pairs, dotted keys (`a.b.c = 1`), quoted keys
- `[table]` and `[[array-of-tables]]` headers (including `[a.b]` after
  `[[a]]`, which targets the last element, per spec)
- Basic `"..."` strings with `\b \t \n \f \r \" \\ \uXXXX \UXXXXXXXX`
  escapes, multiline `"""..."""` (first-newline trim, line-ending
  backslash continuation), literal `'...'` and `'''...'''` strings
- Integers: decimal, `0x`/`0o`/`0b`, underscores, signs (leading zeros
  rejected, per spec)
- Floats: fractions, exponents, `inf`/`nan` (signed)
- Booleans, datetimes (`YYYY-MM-DD`, `HH:MM:SS`, RFC 3339 with `T`/space
  separator, fractional seconds, `Z`/offset) — stored as raw text
- Arrays: nested, multiline, trailing commas, `#` comments inside
- Inline tables (single-line, dotted keys inside)

## Redefinition rules (enforced, per spec)

- Duplicate keys are errors — including inside inline tables.
- A `[table]` header over an already header-defined table is an error.
- A `[table]` header over a table created by a dotted key is an error.
- Inline tables cannot be extended from outside.
- Implicit tables created as header-path intermediates *can* be defined
  later by a header.

## Limitations (v0.1.0)

- Datetime validation is structural, not calendrical: month/day ranges
  are not checked (that is a calendar's job, not a parser's).
- No comment preservation in the encoder; encoding is canonical, not
  formatting-preserving.
- Integer overflow is not diagnosed (accumulation wraps, as in C).

## Dependencies

None. Standard library only (`strings`, `collection`, `types`, `fs`).

## Development

```sh
finc src/toml.fin                                    # frontend check
finc tests/test_toml.fin -I src -o test_toml && ./test_toml
finc examples/basic.fin -I src -o basic && ./basic
```

Verified on finc 0.4.0 (contract 3): 85-assertion suite passes, both
examples run with correct output.

## License

MIT — see LICENSE.
