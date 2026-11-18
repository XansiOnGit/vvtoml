# vvtoml - Toml for Vivi
a small toy I have been making for vivi.

A TOML reader and writer for [Vivi](https://github.com/djxnzi/Vivi).

## Installing the lib
installing it is pretty easy.

Put `vvtoml.vivi` in Vivi's `lib/core` folder, next to `io.vivi` and `sys.vivi`. Then import it by name
Or other places that vivi can read however if using another folder that is not core you may have to use "Vvtoml.vivi"
```
#import "Vvtoml"
```
## NixOS
or installing on NixOS

Add the flake input

```nix
vvtoml-src = {
  url = "github:XansiOnGit/vvtoml";
  flake = false;
};
```

Add it to your Vivi package:

```nix
(inputs.vivi.packages.${pkgs.system}.default.overrideAttrs (old: {
  installPhase = (old.installPhase or "") + ''
    mkdir -p $out/lib/vivi/core
    cp ${inputs.vvtoml-src}/vvtoml.vivi $out/lib/vivi/core/vvtoml.vivi
  '';
}))
```

Don't use `as` with it. The library calls its own helper functions, so import it plain.

## Functions

| Function | What it does |
|---|---|
| `toml_parse(text)` | Turns TOML text into a value. Returns `null` if the text is bad. |
| `toml_parse_strict(text)` | Same, but calls `panic()` if the text is bad. |
| `toml_error()` | The error message from the last parse, or `null` if it worked. |
| `toml_stringify(value)` | Turns a value into TOML text. |

## Reading TOML

```
#import "Io"
#import "Vvtoml"

doc := toml_parse(file_read("config.toml"))
if toml_error() != null {
    eprintln(toml_error())
    exit(1)
}

println(doc["server"]["host"])
println(doc["server"]["port"])
println(doc["tags"][0])
```

Always check `toml_error()`, not the result a empty table and a failed parse both come back as `null`.

## Writing TOML

```
config := [
    "title": "my app",
    "server": [
        "host": "localhost",
        "port": 8080
    ]
]

file_write("config.toml", toml_stringify(config))
```

This writes:

```
title = "thingy"

[server]
host = "localhost"
port = 8080
```
this is how types map.

| TOML | Vivi |
|---|---|
| table | array with string keys |
| array | array |
| string | string |
| integer | int |
| float | f64 |
| true / false | bool |

## Errors

Error messages say where the problem is:

```
toml: unexpected character in value (line 1, column 8)
```

this is what i know works.

- Comments (`# like this`)
- `[tables]` and `[dotted.tables]`
- Dotted keys (`a.b = 1`) and quoted keys (`"my key" = 1`)
- Strings with `"double"` or `'single'` quotes, plus triple-quoted strings
- Escapes: `\n` `\t` `\r` `\b` `\f` `\\` `\"`
- Integers (with `_` allowed, like `1_000`) and floats (like `1.5` or `2e3`)
- Arrays, including multi-line and nested ones
- Inline tables (`point = { x = 1, y = 2 }`)

What doesn't work yet :(

- `[[array of tables]]` gives an error
- `\uXXXX` escapes
- Hex, octal and binary numbers
- Dates and times
- `inf` and `nan` when reading (they are written correctly)
- Integers longer than 18 digits give an error

sum things to watch out for

- **Braces in strings.** Vivi treats `{...}` inside a string as code. If you write TOML by hand in a Vivi string, build the braces with `rune(123)` and `rune(125)`.
- **Empty tables and arrays.** `{}` and `[]` both become an empty array, so an empty table is written back as `[]`.
- **One-letter keys.** In Vivi, a one-character string like `"x"` is a rune, so use keys with two or more letters when you build a value by hand.
- **One parse at a time.** The parser keeps its state in global variables, so don't call it from inside another parse.
- **Null values.** TOML has no null, so `toml_stringify` skips any key whose value is `null`.
