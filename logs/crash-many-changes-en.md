# Crash: Many Changes (English Instruction)

**Date:** 2025-01-10
**Environment:** macOS, Claude Code, `CLAUDE_CODE_SYNTAX_HIGHLIGHT=on`
**Operation:** Replace "下人" with "浪人" in rashomon.txt (English instruction)

## Session Log

```
Replace all occurrences of "下人"
  with "浪人" in rashomon.txt

⏺ Read(rashomon.txt)
  ⎿  Read 45 lines

⏺ 「下人」を「浪人」に全て置換します。

⏺ Update(rashomon.txt)

✶ Canoodling… (thought for 10s)
```

## Crash Output

```
thread '<unnamed>' (16705791) panicked at /rustc/ed61e7d7e242494fb7057f2657300d9e77bb4fcb/library/core/src/str/mod.rs:833:21:
byte index 38 is not a char boundary; it is inside '、' (bytes 36..39) of `上るけしきがない。そこで、下人は、何`
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
fatal runtime error: failed to initiate panic, error 5, aborting
zsh: abort      claude
```

## Analysis

| Item | Value |
|------|-------|
| Problematic character | `、` (U+3001, Ideographic comma) |
| UTF-8 byte range | 36-39 (3 bytes) |
| Invalid byte index | 38 |
| Crash timing | During diff display with syntax highlighting |

## Notes

- **Instruction language does not matter** - English instruction causes same crash
- Different byte index (38) compared to Japanese instruction logs (53)
- Same character type (`、`) causes the crash
- Confirms the issue is in diff rendering, not instruction parsing
