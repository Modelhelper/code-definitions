# Code definitions for ModelHelper

Language definitions for the [ModelHelper CLI](https://github.com/Modelhelper/cli) (`mh`). A definition tells `mh` how to turn a database type into a type in your programming language. When a template writes `{{ .DataType | datatype }}` for a column of type `varchar`, the definition for the template's `language` decides that the result is `string` in Go and in TypeScript, and that the nullable form in Go is `sql.NullString`.

| Language | File | Notes |
|----------|------|-------|
| Go | `definitions/go.yaml` | Uses only the standard library (`database/sql` null types) |
| C# | `definitions/cs.yaml` | Nullable types use `?`. Also has default imports and keys |
| TypeScript | `definitions/ts.yaml` | |
| Dart | `definitions/dart.yaml` | Null safety (`T?`). `Uint8List` needs `dart:typed_data` |
| Rust | `definitions/rust.yaml` | `Option<T>`. Uses the crates `chrono`, `rust_decimal`, `uuid` and `serde_json` |
| Protocol Buffers | `definitions/proto.yaml` | `language: proto`. Nullable is `optional`. Dates are `google.protobuf.Timestamp` |
| SQL | `definitions/sql.yaml` | For the SQL templates |
| C++ | `definitions/cpp.yaml` | A starting point, the types are not checked |

The type names in the files are database types. The Dart, Rust and proto files cover the SQL Server names (`varchar`, `bigint`, `datetime2`, `uniqueidentifier`) and the PostgreSQL names (`int4`, `int8`, `bool`, `timestamptz`, `uuid`, `bytea`, `jsonb` and so on). The other files cover the SQL Server names only, so a PostgreSQL type such as `int4` is printed as it is for them.

One name means different things in the two databases: `timestamp` is a row version (bytes) in SQL Server and a date and time in PostgreSQL. The files follow SQL Server.

## Use the definitions

`mh` has copies of these files built in, so it works out of the box. Use this repository when you want to change a mapping or add a language without rebuilding `mh`.

1. Clone the repository:

   ```bash
   git clone https://github.com/Modelhelper/code-definitions.git
   ```

2. Open the `mh` config (`mh config open`, or edit `~/.modelhelper/config.yaml`) and set the path to the `definitions` folder:

   ```yaml
   languages:
     definitions: /path/to/code-definitions/definitions
   ```

3. Check the result:

   ```bash
   mh language list
   ```

   The `Source` column shows `built in`, or the path of the file that is used.

The files in your folder are laid **over** the built-in definitions. A file with `language: go` replaces the built-in Go definition, a file for a new language adds it, and a language you have no file for keeps its built-in definition. A folder that does not exist is ignored.

## Change a mapping

Edit the `notNull` and `nullable` values of a type, then run a template. No rebuild is needed:

```yaml
datatypes:
  decimal:
    notNull: float64
    nullable: sql.NullFloat64
```

```bash
mh code generate -t go-struct --demo -s order --screen
```

The Go file maps `decimal`, `numeric` and `money` to `float64`, because it only uses the standard library. A `float64` can round amounts. If you need exact values, change them to `string` or to the type of a decimal package, and add the import in your templates.

## Add a language

Create `definitions/<language>.yaml`. The `language` must be the same as the `language` in the templates that use it. [default-layout.yaml](default-layout.yaml) explains every option. The smallest useful file is:

```yaml
version: 3.0
language: rust
short: Language definition for Rust

datatypes:
  int:
    notNull: i32
    nullable: Option<i32>
  bigint:
    notNull: i64
    nullable: Option<i64>
  varchar:
    notNull: String
    nullable: Option<String>
```

A database type that is not in `datatypes` is printed as it is.

Quote values that start with `[`, `{`, `*` or `&`, YAML reads them as something else: `notNull: "[]byte"`.

## What `mh` uses today

| Property | Effect |
|----------|--------|
| `language`, `version`, `short`, `description` | Shown by `mh language list` |
| `datatypes` with `notNull` and `nullable` | **Used.** This is what the `datatype` and `nullable` template functions read |
| `datatypes` with `defaultValue` and `nullableAlternative` | Read, but no template uses them yet |
| `keys`, `inject`, `defaultImports`, `global` | Counted by `mh language list`. Namespace, prefix, postfix and inject for generated code come from the project's `.modelhelper/config.yaml` |
| `usesNamespace`, `canInject`, `moduleLevelVariablePrefix` | Not read |

## Repository layout

```
definitions/          the language files that mh reads
default-layout.yaml   a documented example of every option
dev/                  empty placeholders for languages without a definition yet
```

## Contributing

Fixes to a mapping and new languages are welcome. Check a mapping by generating code with `mh code generate --demo`.
