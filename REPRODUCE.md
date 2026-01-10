# Reproduction Steps

## Environment Setup

### Enable Syntax Highlighting (Default)

```bash
# Explicitly enable (optional, this is the default)
export CLAUDE_CODE_SYNTAX_HIGHLIGHT=on

# Or unset to use default behavior
unset CLAUDE_CODE_SYNTAX_HIGHLIGHT
```

### Verify Environment

```bash
echo $CLAUDE_CODE_SYNTAX_HIGHLIGHT
# Should be empty (default) or "on"
```

## Test Cases

### Test 1: Replace Japanese Words in Text File

This is the primary reproduction case.

```bash
cd /path/to/claude-code-utf8-crash-repro
claude
```

Enter the following prompt in Claude Code:

```
Replace all occurrences of "下人" with "浪人" in sample_data/rashomon.txt
```

**Expected behavior:** Edit completes successfully
**Actual behavior:** Crash after diff is displayed with syntax highlighting

---

### Test 2: Edit CSV File with Japanese Content

```bash
claude
```

Enter the following prompt:

```
Change "羅生門" to "羅城門" in sample_data/works.csv
```

---

### Test 3: Edit Markdown File with Japanese Content

```bash
claude
```

Enter the following prompt:

```
Change "羅生門" to "羅城門" in sample_data/author.md
```

---

### Test 4: Replace Long Japanese String

```bash
claude
```

Enter the following prompt:

```
Replace "夕冷えのする京都は" with "夕暮れの京都は" in sample_data/rashomon.txt
```

---

## Crash Log Examples

### Actual Reproduction (2025-01-10)

**Environment:** macOS, Claude Code, `CLAUDE_CODE_SYNTAX_HIGHLIGHT=on`

**Operation:** Replace "下人" with "浪人" in sample_data/rashomon.txt

```
thread '<unnamed>' (16512894) panicked at /rustc/ed61e7d7e242494fb7057f2657300d9e77bb4fcb/library/core/src/str/mod.rs:833:21:
byte index 53 is not a char boundary; it is inside '、' (bytes 51..54) of `、いまだに上るけしきがない。そこで、下人は、何をおいても差`
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
fatal runtime error: failed to initiate panic, error 5, aborting
zsh: abort      claude
```

### Analysis

- **Problematic character:** `、` (Ideographic comma, U+3001)
- **UTF-8 encoding:** 3 bytes (E3 80 81) at positions 51-54
- **Invalid byte index:** 53 (lands inside the character)
- **Crash timing:** During syntax-highlighted diff rendering

### Similar Errors from Related Issues

From issue #16543 (Korean):
```
byte index 86 is not a char boundary; it is inside '요' (bytes 84..87) of `엔 달력 보면서...`
```

From issue #14133 (Korean):
```
byte index 13 is not a char boundary; it is inside '용' (bytes 11..14) of '필터 적용'
```

## Getting Debug Information

To get a full backtrace, run:

```bash
RUST_BACKTRACE=1 claude
```

Then reproduce the crash. The backtrace will show the exact code path leading to the panic.

## Workaround

Disable syntax highlighting to avoid the crash:

```bash
export CLAUDE_CODE_SYNTAX_HIGHLIGHT=off
claude
```

This prevents the problematic string slicing code from executing.

## Notes

- The crash occurs specifically during the **diff display** phase with syntax highlighting
- The crash happens **after** Claude successfully reads and processes the file
- The crash character varies depending on where the byte-slicing lands
- Common crash characters: `、` `。` `「` `」` and various Kanji/Kana
