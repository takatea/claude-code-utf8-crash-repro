# Crash: Many Changes (Japanese Instruction)

**Date:** 2025-01-10
**Environment:** macOS, Claude Code, `CLAUDE_CODE_SYNTAX_HIGHLIGHT=on`
**Operation:** Replace "下人" with "浪人" in rashomon.txt only

## Session Log

```
❯ 「下人」を「浪人」に置換

⏺ Search(pattern: "下人")
  ⎿  Found 5 files

⏺ User answered Claude's questions:
  ⎿  rashomon.txtのみ

⏺ Read(rashomon.txt)
  ⎿  Read 45 lines

⏺ Update(rashomon.txt)

· Osmosing… (ctrl+c to interrupt · thinking)
```

## Crash Output

```
thread '<unnamed>' (16512894) panicked at /rustc/ed61e7d7e242494fb7057f2657300d9e77bb4fcb/library/core/src/str/mod.rs:833:21:
byte index 53 is not a char boundary; it is inside '、' (bytes 51..54) of `、いまだに上るけしきがない。そこで、下人は、何をおいても差`
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
fatal runtime error: failed to initiate panic, error 5, aborting
zsh: abort      claude
```

## Analysis

| Item | Value |
|------|-------|
| Problematic character | `、` (U+3001, Ideographic comma) |
| UTF-8 byte range | 51-54 (3 bytes) |
| Invalid byte index | 53 |
| Crash timing | During diff display with syntax highlighting |
