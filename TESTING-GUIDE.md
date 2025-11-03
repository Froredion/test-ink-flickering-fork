# Testing Guide: Ink Flickering Investigation

This guide will help you systematically test and document the screen flickering issue.

## Quick Start

```bash
cd test-ink-flickering
npm install
npm start
```

## Test Scenarios

### 1. Baseline Test (minimal-repro.js)

**Run:** `npm start`

**What it does:**
- Updates status line every 100ms
- Shows static conversation history
- Moderate rendering load

**What to observe:**
- Does the entire screen flash?
- Does old text briefly reappear?
- Is the flickering constant or intermittent?

**Expected behavior:**
- If flickering occurs, it should be similar to Claude Code's behavior
- The entire terminal buffer may redraw instead of just the status line

---

### 2. OSC133 Test (with-osc133.js)

**Run:** `npm run start:osc133`

**What it does:**
- Same as baseline but emits OSC 133 control sequences
- Marks prompt boundaries to help terminal optimize rendering

**What to observe:**
- Is the flickering reduced compared to baseline?
- Does your terminal support OSC 133? (Kitty, iTerm2, Windows Terminal, VS Code)

**Terminal compatibility:**
- ✅ Kitty
- ✅ iTerm2 (macOS)
- ✅ Windows Terminal
- ✅ VS Code integrated terminal
- ❌ GNOME Terminal (no OSC 133 support)
- ❓ tmux (depends on inner terminal)

---

### 3. Stress Test (stress-test.js)

**Run:** `npm run stress`

⚠️ **WARNING:** Updates every 50ms - may be uncomfortable!

**What it does:**
- Very aggressive updates (50ms interval)
- Auto-scrolling content
- Multiple changing values (timer, CPU, message count)

**What to observe:**
- Is flickering more pronounced?
- Does scrolling make it worse?
- Performance/CPU usage

---

## Comparison Matrix

Testing matrix with actual results:

| Test Type | Terminal | Flickering? | Severity (1-5) | Notes | Date |
|-----------|----------|-------------|----------------|-------|------|
| Baseline  | GNOME    | ✅ Yes      | TBD            | In tmux | 2025-11-03 |
| OSC133    | GNOME    | ✅ Yes      | TBD            | No OSC133 support - test inconclusive | 2025-11-03 |
| Stress    | GNOME    | ⏳ Pending  | -              |       |      |
| Baseline  | Kitty    | ⏳ Pending  | -              | Needs Kitty installed |      |
| OSC133    | Kitty    | ⏳ Pending  | -              | **High priority** - has OSC133 support |      |
| Stress    | Kitty    | ⏳ Pending  | -              |       |      |

**Key Finding**: GNOME Terminal does not support OSC 133, making that test inconclusive. Need to test on Kitty, Windows Terminal, or VS Code terminal for valid OSC133 results.

📋 **See [FINDINGS.md](./FINDINGS.md) for detailed test results and analysis**

## What to Look For

### Signs of Flickering
1. **Full screen flash** - entire content area blinks
2. **Old text reappearing** - text from earlier momentarily visible
3. **Cursor jumping** - cursor position flickers
4. **Status line affecting whole screen** - updates cause global redraw

### Good Rendering (No Flicker)
1. Only the status line area updates
2. Text above stays stable
3. Smooth spinner animation
4. No visible screen refresh

## Advanced Testing

### Recording the Issue

**Video capture:**
```bash
# Use asciinema to record terminal session
asciinema rec flickering-test.cast
npm start
# Stop with Ctrl+C
```

**Screenshot comparison:**
```bash
# Take rapid screenshots during execution
# Look for differences in consecutive frames
```

### Performance Analysis

**Monitor redraws:**
```bash
# Use strace to see write() calls to terminal
strace -e write -s 1000 npm start 2>&1 | grep "write(1,"
```

**CPU usage:**
```bash
# Run with performance monitoring
/usr/bin/time -v npm start
```

### Testing Different Ink Versions

**Test bcherny's fork:**
```bash
# Edit package.json to use the fork
npm install bcherny/ink#master
npm start
```

**Compare versions:**
```bash
# Test official Ink
npm install ink@latest
npm start

# Test older versions
npm install ink@4.0.0
npm start
```

## Debugging Tips

### Enable Ink Debug Mode
Set environment variable:
```bash
DEBUG=* npm start
```

### Check Terminal Capabilities
```bash
# See what your terminal supports
echo $TERM
infocmp $TERM | grep -E "cup|csr|cud|clear"
```

### Raw Escape Sequence Inspection
```bash
# Capture raw output
script -c "npm start" output.txt
# Then examine output.txt in hex editor
```

## Next Steps

1. **Document findings** - Record which scenarios cause flickering
2. **Test fixes** - Try OSC133, different Ink versions, patches
3. **Compare with Claude Code** - Does it behave the same way?
4. **Report results** - Share findings on the GitHub issue

## Related Resources

- [Claude Code Issue #769](https://github.com/anthropics/claude-code/issues/769)
- [Ink Fork PR with OSC133](https://github.com/bcherny/ink/pull/8)
- [OSC 133 Specification](https://gitlab.freedesktop.org/Per_Bothner/specifications/-/blob/master/proposals/semantic-prompts.md)
- [Terminal Control Sequences](https://invisible-island.net/xterm/ctlseqs/ctlseqs.html)
