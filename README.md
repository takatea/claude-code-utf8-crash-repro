# Claude Code UTF-8 Crash Reproduction

This repository provides sample files to reproduce a crash bug in Claude Code related to UTF-8 multi-byte character processing.

## Related Issues

- [#16543](https://github.com/anthropics/claude-code/issues/16543) - Rust panic at UTF-8 character boundary
- [#14133](https://github.com/anthropics/claude-code/issues/14133) - Crash when processing Korean text

## Problem Overview

When `CLAUDE_CODE_SYNTAX_HIGHLIGHT=on` (default), Claude Code crashes with a Rust panic when attempting to edit files containing multi-byte UTF-8 characters (Japanese, Korean, Chinese, etc.).

### Root Cause

The Rust string slicing operation attempts to slice at a byte position that is not a valid UTF-8 character boundary. Multi-byte characters (like Japanese/Korean characters which are 3 bytes each in UTF-8) get split in the middle, causing a panic.

### Error Example

```
thread '<unnamed>' panicked at /rustc/.../library/core/src/str/mod.rs:833:21:
byte index 53 is not a char boundary; it is inside '、' (bytes 51..54) of `...`
fatal runtime error: failed to initiate panic, error 5, aborting
```

## Sample Text

This repository uses the full text of "Rashomon" (羅生門) by Ryunosuke Akutagawa (1915).

**Why this text was chosen:**
- Sufficient length of Japanese text (~19KB)
- Contains a mix of Kanji, Hiragana, and Katakana characters
- Includes parenthetical ruby (furigana) annotations with special character patterns
- Real-world literary text, not synthetic test data

**Copyright Status:**
- Author: Ryunosuke Akutagawa (芥川龍之介, 1892-1927)
- The author passed away in 1927, over 70 years ago
- Under Japanese copyright law, works enter the public domain 70 years after the author's death
- This text is freely available from [Aozora Bunko](https://www.aozora.gr.jp/) (Japanese digital library of public domain works)
- **This work is in the public domain worldwide and can be freely used, modified, and distributed**

## Repository Structure

```
.
├── README.md              # This file
├── REPRODUCE.md           # Detailed reproduction steps
├── sample_data/           # Sample files with Japanese content
│   ├── rashomon.txt       # Full text of "Rashomon" by Akutagawa
│   ├── works.csv          # Akutagawa's works list
│   └── author.md          # Author information
└── logs/                  # Crash reproduction logs
    ├── crash-many-changes-jp.md       # Japanese instruction crash
    ├── crash-many-changes-jp-retry.md # Reproducibility confirmation
    ├── crash-many-changes-en.md       # English instruction crash
    ├── crash-multifile.md             # Multiple file edit crash
    └── success-highlight-off.md       # Workaround verification
```

## Quick Reproduction

### Prerequisites

- Claude Code CLI installed
- `CLAUDE_CODE_SYNTAX_HIGHLIGHT=on` (this is the default setting)

### Steps

1. Clone this repository
2. Open terminal and navigate to the repository directory
3. Run Claude Code:
   ```bash
   claude
   ```
4. Ask Claude to replace a Japanese word:
   ```
   Replace all occurrences of "下人" with "浪人" in sample_data/rashomon.txt
   ```
5. **Result:** After syntax highlighting shows the diff, the CLI crashes

## Actual Reproduction Log (2025-01-10)

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

### Analysis

| Item | Value |
|------|-------|
| Crashed character | `、` (Japanese comma / ideographic comma) |
| UTF-8 byte range | 51-54 (3 bytes) |
| Invalid byte index | 53 (middle of character) |
| Crash timing | During diff display with syntax highlighting |

### Technical Explanation

The character `、` (U+3001, IDEOGRAPHIC COMMA) is encoded in UTF-8 as 3 bytes: `E3 80 81`. When the syntax highlighting code tries to slice the string at byte index 53, it lands in the middle of this character (between bytes 51 and 54), which is not a valid character boundary in UTF-8. Rust's string safety checks detect this and panic.

## Crash Conditions

Based on testing, the crash appears to be related to the **number of changes** in the diff:

| Operation | Occurrences | Result |
|-----------|-------------|--------|
| Replace "下人" with "浪人" | ~30+ times | **CRASH** |
| Replace "羅生門" with "羅城門" | 2-3 times | SUCCESS |
| Replace "夕冷えのする京都は" with "夕暮れの京都は" | 1 time | SUCCESS |

**Hypothesis:** When there are many changes, the diff display becomes longer, increasing the chance that the syntax highlighting code will attempt to slice at an invalid UTF-8 byte boundary.

## Workaround

Disable syntax highlighting to avoid the crash:

```bash
export CLAUDE_CODE_SYNTAX_HIGHLIGHT=off
claude
```

## License

- Repository documentation: MIT License
- Sample text (rashomon.txt): Public Domain (via [Aozora Bunko](https://www.aozora.gr.jp/))
