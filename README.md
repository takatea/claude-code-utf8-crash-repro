# Claude Code UTF-8 Crash Reproduction

> **✅ Fixed in v2.1.6 (2026-01-13)**
>
> This bug appears to be fixed in Claude Code v2.1.6. The fix "Fixed text styling (bold, colors) getting progressively misaligned in multi-line responses" addressed the UTF-8 character boundary issue in syntax highlighting. Related issues [#16543](https://github.com/anthropics/claude-code/issues/16543), [#14133](https://github.com/anthropics/claude-code/issues/14133), [#14104](https://github.com/anthropics/claude-code/issues/14104) are now CLOSED.

This repository reproduces a crash bug in Claude Code related to UTF-8 multi-byte character processing. **Key finding: the crash is related to the number of changes in the diff — many changes crash, few changes succeed.**

## Related Issues

- [#16543](https://github.com/anthropics/claude-code/issues/16543) - Rust panic at UTF-8 character boundary
- [#14133](https://github.com/anthropics/claude-code/issues/14133) - Crash when processing Korean text
- [#14104](https://github.com/anthropics/claude-code/issues/14104) - UTF-8 Character Boundary Crash

## Key Findings

### Crash Conditions

| Operation | Occurrences | Result |
|-----------|-------------|--------|
| Replace "下人" with "浪人" | ~30+ times | **CRASH** |
| Replace "羅生門" with "羅城門" | 2-3 times | SUCCESS |
| Replace "夕冷えのする京都は" with "夕暮れの京都は" | 1 time | SUCCESS |

**Hypothesis:** When there are many changes, the diff display becomes longer, increasing the chance that the syntax highlighting code will attempt to slice at an invalid UTF-8 byte boundary.

### Workaround

```bash
export CLAUDE_CODE_SYNTAX_HIGHLIGHT=off
claude
```

## Reproduction

### Quick Steps

1. Clone this repository
2. Ensure `CLAUDE_CODE_SYNTAX_HIGHLIGHT=on` (default)
3. Run `claude`
4. Enter: `Replace all occurrences of "下人" with "浪人" in sample_data/rashomon.txt`
5. **Result:** CLI crashes after syntax highlighting shows the diff

### Detailed Log (2025-01-10)

**Environment:** macOS, Claude Code, `CLAUDE_CODE_SYNTAX_HIGHLIGHT=on`

**Operation:** Replace "下人" (servant) with "浪人" (ronin) in sample_data/rashomon.txt

```
❯ 「下人」を「浪人」に置換

⏺ Search(pattern: "下人")
  ⎿  Found 5 files

⏺ Read(rashomon.txt)
  ⎿  Read 45 lines

⏺ Update(rashomon.txt)

thread '<unnamed>' (16512894) panicked at /rustc/ed61e7d7e242494fb7057f2657300d9e77bb4fcb/library/core/src/str/mod.rs:833:21:
byte index 53 is not a char boundary; it is inside '、' (bytes 51..54) of `、いまだに上るけしきがない。そこで、下人は、何をおいても差`
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
fatal runtime error: failed to initiate panic, error 5, aborting
zsh: abort      claude
```

| Item | Value |
|------|-------|
| Crashed character | `、` (Japanese comma / ideographic comma) |
| UTF-8 byte range | 51-54 (3 bytes) |
| Invalid byte index | 53 (middle of character) |
| Crash timing | During diff display with syntax highlighting |

## Technical Details

### Root Cause

The Rust string slicing operation attempts to slice at a byte position that is not a valid UTF-8 character boundary. Multi-byte characters (like Japanese/Korean characters which are 3 bytes each in UTF-8) get split in the middle, causing a panic.

### Error Example

The character `、` (U+3001, IDEOGRAPHIC COMMA) is encoded in UTF-8 as 3 bytes: `E3 80 81`. When the syntax highlighting code tries to slice the string at byte index 53, it lands in the middle of this character (between bytes 51 and 54), which is not a valid character boundary in UTF-8. Rust's string safety checks detect this and panic.

```
thread '<unnamed>' panicked at /rustc/.../library/core/src/str/mod.rs:833:21:
byte index 53 is not a char boundary; it is inside '、' (bytes 51..54) of `...`
fatal runtime error: failed to initiate panic, error 5, aborting
```

## Sample Text: "Rashomon" by Ryunosuke Akutagawa

This repository uses the full text of "Rashomon" (羅生門, 1915).

**Why this text was chosen:**
- Sufficient length of Japanese text (~19KB)
- Contains a mix of Kanji, Hiragana, and Katakana characters
- Includes parenthetical ruby (furigana) annotations with special character patterns
- Real-world literary text, not synthetic test data

**Copyright Status:**
- Author: Ryunosuke Akutagawa (芥川龍之介, 1892-1927)
- This text is in the public domain worldwide (author passed away over 70 years ago)
- Available from [Aozora Bunko](https://www.aozora.gr.jp/)

## Repository Structure

```
.
├── README.md
├── REPRODUCE.md           # Detailed reproduction steps
├── sample_data/
│   ├── rashomon.txt       # Full text of "Rashomon" by Akutagawa
│   ├── works.csv          # Akutagawa's works list
│   └── author.md          # Author information
└── logs/
    ├── crash-many-changes-jp.md
    ├── crash-many-changes-jp-retry.md
    ├── crash-many-changes-en.md
    ├── crash-multifile.md
    └── success-highlight-off.md
```

## License

- Repository documentation: MIT License
- Sample text (rashomon.txt): Public Domain (via [Aozora Bunko](https://www.aozora.gr.jp/))
