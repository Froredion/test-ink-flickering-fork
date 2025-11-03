# Ink Flickering Reproduction Test

This project is a minimal reproduction case for the screen flickering issue reported in [anthropics/claude-code#769](https://github.com/anthropics/claude-code/issues/769).

## The Issue

When Claude Code processes requests, the entire terminal buffer redraws with each status update instead of just refreshing the indicator line, causing screen flickering that can be problematic for users sensitive to flashing lights.

## Project Status

### Phase 1: Reproduction ✅ Complete
- Successfully reproduced Ink flickering with minimal test case
- Confirmed issue matches Claude Code behavior
- Established baseline for testing fixes

### Phase 2: Fix Testing ✅ Complete - **No Fixes Work**
Exhaustively tested all commonly suggested fixes:
- ❌ OSC133 sequences (inconclusive on GNOME, but fork with OSC133 fails)
- ❌ bcherny's Ink fork v5.0.24 (doesn't fix it)
- ❌ Ink 3.2.0 (still flickers)
- ❌ Ink 4.0.0 (still flickers)
- ❌ Ink 4.4.1 (still flickers)

**Conclusion**: Flickering is fundamental to Ink's rendering architecture across all versions.

### Phase 3: Source Code Investigation ✅ Complete - **Cannot Be Fixed**
Tested three different fixes by modifying Ink's source code (log-update.ts):
- ❌ cursorUp + overwrite (still flickers)
- ❌ Overwrite-only without clearing (still flickers)
- ❌ Line-by-line differential updates (still flickers, actually worse)

**Conclusion**: Flickering **cannot be fixed** at the log-update.ts level. The problem is architectural - Ink regenerates complete output on every React state change, losing information about what actually changed. Any cursor movement is visible in tmux.

📋 **See [FIX-TESTING.md](./FIX-TESTING.md) for all test results**
📋 **See [INK-ANALYSIS.md](./INK-ANALYSIS.md) for architectural analysis**
📋 **See [CONCLUSIONS.md](./CONCLUSIONS.md) for recommendations**

---

## 🎯 Key Findings

### The Problem is Architectural

Ink's flickering cannot be fixed with simple patches. Here's why:

1. **Full Tree Traversal**: Ink regenerates the complete output on EVERY React state change, even when only the status line updates
2. **Lost Information**: By the time the output reaches `log-update.ts`, it's just a string - no context about what changed
3. **Visible Cursor Movement**: Any ANSI cursor repositioning is visible in terminals like tmux
4. **No Atomic Updates**: Terminals don't have a "swap buffers" operation - every write is immediately visible

### What Would Actually Work

**Option 2 from our analysis**: Create a separate `<StatusLine>` component that:
- Uses absolute cursor positioning (`ESC[{row};{col}H`)
- Updates independently from main content
- Requires ~200-300 lines of code, 2-3 days effort
- Would work in tmux

### Recommendations for Users

**If using Claude Code:**
- Accept this is a limitation of Ink
- Reduce status update frequency if possible
- Consider terminals with better buffering (Kitty, Alacritty)
- Use tmux with detach/reattach to avoid constant visibility

**If building TUI apps:**
- Consider alternative libraries (Blessed has screen diffing)
- Or implement StatusLine component approach
- Or accept the trade-off (simplicity vs performance)

---

## What This Does

This test app mimics Claude Code's rendering behavior:
- Displays conversation history (scrollable text content)
- Shows a status line with an animated spinner
- Updates the timer every 100ms (triggering frequent re-renders)
- Uses ANSI colors and formatting similar to Claude Code

## How to Run

```bash
npm install
npm start
```

## What to Watch For

1. **Screen Flickering**: Does the entire screen flash when the timer updates?
2. **Text Reappearing**: Do you see text from earlier in the session briefly flash back?
3. **Scrollback Behavior**: Does scrolling make it worse?

## Testing Different Scenarios

### Baseline Test (Official Ink)
```bash
npm start
```

### With OSC133 Sequences
```bash
npm run start:osc133
```
**Note**: Requires terminal with OSC133 support (Kitty, Windows Terminal, VS Code)

### Stress Test (Aggressive Rendering)
```bash
npm run stress
```
⚠️ **Warning**: Updates every 50ms - may be uncomfortable!

## Terminal Compatibility

Test this on different terminals to compare behavior:
- GNOME Terminal
- Kitty (has OSC133 support)
- iTerm2 (macOS)
- Windows Terminal
- tmux
- VS Code integrated terminal

## Related Links

- Original Issue: https://github.com/anthropics/claude-code/issues/769
- Ink Library Fork PR: https://github.com/bcherny/ink/pull/8
- OSC133 Documentation: Search for "OSC 133 shell integration"

## Investigation Summary

### Phase 1: Reproduction ✅
- Successfully reproduced flickering with minimal test case
- Confirmed behavior matches Claude Code

### Phase 2: External Fix Testing ✅
- OSC133 sequences: ❌ No improvement
- bcherny's Ink fork: ❌ No improvement
- Ink v3.2.0, v4.0.0, v4.4.1: ❌ All flicker

### Phase 3: Source Code Testing ✅
- Built Ink from source
- Tested 3 different rendering approaches
- All failed to reduce flickering
- **Conclusion**: Problem is architectural, not patchable

### Investigation Status: ✅ COMPLETE

**Result**: Flickering is a fundamental architectural limitation of Ink that cannot be fixed with simple patches. See [CONCLUSIONS.md](./CONCLUSIONS.md) for full analysis and recommendations.

## Documentation

### Investigation Results
- [CONCLUSIONS.md](./CONCLUSIONS.md) - **START HERE**: Executive summary and recommendations
- [FIX-TESTING.md](./FIX-TESTING.md) - All three tested fixes with code and analysis
- [INK-ANALYSIS.md](./INK-ANALYSIS.md) - Complete architectural deep-dive

### Investigation Process
- [REPRODUCED.md](./REPRODUCED.md) - Initial reproduction confirmation
- [FINDINGS.md](./FINDINGS.md) - Chronological test log
- [TESTING-GUIDE.md](./TESTING-GUIDE.md) - How to reproduce and test
