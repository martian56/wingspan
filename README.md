# wingspan

Display width for terminal text, in pure Raven.

A String's `length()` counts UTF-8 bytes, and that number lies the moment
text leaves ASCII: CJK characters render two columns wide, combining marks
render zero, and ANSI color codes render nothing at all. Line up columns by
byte count and your tables drift. wingspan counts what the terminal actually
draws.

## Install

```toml
[dependencies]
"github.com/martian56/wingspan" = "v0.1.0"
```

## Usage

```raven
import "github.com/martian56/wingspan" { width, visible_width, truncate, pad_right }

width("hello")     // 5
width("你好")      // 4, two columns per character
width("é")         // 1, the combining accent takes no column

// Styled text measures at its printed size:
visible_width("\u{1b}[32mok\u{1b}[0m")   // 2

// Fit text into a column:
truncate("你好吗", 4)   // "你好"
pad_right("名前", 8)    // "名前    " (padded to 8 columns, not 8 bytes)
```

## API

- `width(s) -> Int`: columns the string takes when printed.
- `visible_width(s) -> Int`: same, but ANSI escape sequences (CSI and OSC)
  count as zero, so colored text measures correctly.
- `char_width(cp) -> Int`: columns for a single codepoint, 0, 1, or 2.
- `decode(s, i) -> CharInfo`: the UTF-8 character at byte offset `i`, as
  `{ cp, size, width }`. Invalid bytes decode as U+FFFD with size 1, and an
  offset past the end returns size 0, so a scan always terminates.
- `truncate(s, max) -> String`: cut to at most `max` columns on a character
  boundary. A wide character never straddles the limit.
- `pad_left(s, w)`, `pad_right(s, w)`, `center(s, w)`: pad with spaces to
  `w` columns. Strings already at or past `w` come back unchanged.

Walking a string character by character:

```raven
let i = 0
while i < s.length() {
    let c = decode(s, i)
    // use c.cp and c.width
    i = i + c.size
}
```

## What it covers

The wide set follows Unicode East Asian Width (W and F) plus the common emoji
blocks. The zero set covers the frequent combining blocks (Latin, Greek,
Cyrillic, Hebrew, Arabic, Thai), zero width spaces and joiners, bidi controls,
variation selectors, and the BOM.

Known simplifications: there is no grapheme clustering, so a ZWJ emoji
sequence measures as the sum of its parts. Ambiguous-width characters count
as one column. Terminals disagree with each other on these edges; wingspan
sides with the common case.

## Development

```
rvpm build       # type-check
rvpm test        # run the test suite
rvpm fmt         # format
raven build demo.rv -o demo && ./demo
```

## License

MIT
