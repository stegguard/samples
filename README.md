# StegGuard samples

Small files with things hidden in them, for trying out [StegGuard](https://github.com/stegguard/stegguard) and checking what it reports. The [demo page](https://stegguard.com/demo) is built from a run over these files.

**Every payload here is harmless.** Nothing in this repo executes, downloads or exfiltrates anything. The files named `evil_*` carry filler patterns, not code. They are named that way because that is what a reviewer would worry about, not because of what they do.

## What's in each file

Results are from StegGuard 0.3.0 with default settings.

| File | What's hidden | StegGuard result |
|---|---|---|
| `release_notes.md` | 296 zero-width characters at the end of line 4. U+200B is a 0 bit, U+200C is a 1, eight bits per ASCII byte. They spell `rotate the staging keys before friday`. | CRITICAL. `stegguard decode` recovers the sentence. |
| `calculator_2.py` | 3,340 copies of U+200C (zero-width non-joiner) after the last comment on line 11. One repeated character, so there is no message, only invisible bulk. | CRITICAL |
| `evil_hello.py` | 24,088 zero-width characters alternating U+200C and U+200D after a visible `→`. Read as bits, that is the byte `0x55` (`U`) over and over: filler with no message. | CRITICAL, **incomplete**: the default limit of 10,000 findings stops the scan. Use `--max-findings 30000` to finish it. |
| `.hidden_evil_hello.py` | Byte-for-byte copy of `evil_hello.py`, as a dotfile, to check that hidden files get scanned. | Same as `evil_hello.py` |
| `nothing_hidden.py` | Six zero-width characters (three U+200B, three U+200C) inside a comment that says nothing is hidden. Six bits is less than one byte, so nothing decodes. | MEDIUM |
| `sample.snowed_2.txt` | Three lines made only of trailing whitespace (spaces and tabs), the channel SNOW-style tools use. | LOW |
| `sample.snowed.txt` | Two empty lines. Nothing hidden. | Clean |
| `hello.py` | Nothing. Control file. | Clean |
| `calculator.py` | Nothing. The same calculator as `calculator_2.py` without the payload. | Clean |
| `example.txt` | Nothing. Plain text. | Clean |

## Try it

```sh
pip install git+https://github.com/stegguard/stegguard
git clone https://github.com/stegguard/samples
cd samples

stegguard detect . -r --html report.html
stegguard decode release_notes.md
stegguard sanitize release_notes.md   # writes release_notes.sanitized.md, leaves the original alone
```

`detect` exits with status 1 when it flags something and 2 when a scan is incomplete, so expect a non-zero exit here.

To see the hidden characters without StegGuard:

```sh
cat -A release_notes.md        # zero-width characters show up as M-bM-^@M-^K and similar
grep -nP '[\x{200B}-\x{200D}]' *.py *.md
```
