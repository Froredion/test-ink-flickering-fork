# Comment for anthropics/claude-code#769

## Investigation Complete: Ink Flickering is an Architectural Limitation

Hey everyone, I've completed a comprehensive investigation of the terminal flickering issue. **TL;DR: It's a fundamental architectural limitation of Ink that cannot be fixed with simple patches.**

### What I Tested

Over 3 days (Nov 1-3, 2025), I tested **6 different approaches** across 3 phases:

**Phase 1 - Reproduction** ✅
- Created minimal Ink app reproducing the exact flickering
- Confirmed it matches Claude Code's behavior
- Repository: https://github.com/atxtechbro/test-ink-flickering

**Phase 2 - External Fixes** ❌
- OSC 133 sequences → Still flickers
- bcherny's Ink fork (with OSC133) → Still flickers
- Ink v3.2.0, v4.0.0, v4.4.1 → All flicker

**Phase 3 - Source Code Modifications** ❌
- Built Ink from source
- Tested 3 different rendering approaches by modifying `log-update.ts`:
  1. cursorUp + overwrite → Still flickers
  2. Overwrite-only (no clearing) → Still flickers
  3. Line-by-line differential → Still flickers (actually worse)

**Success rate: 0/6 (0%)**

### Why All Fixes Failed

The problem is **architectural**:

1. **Ink regenerates the complete output** on every React state change
2. **Full component tree traversal** even when only status line changes
3. **By the time `log-update.ts` receives output**, it's just a string with no context about what changed
4. **Any cursor movement is visible** in terminals like tmux

The flickering pattern you see (`[2K[1A` repeated 60+ times) is Ink moving the cursor up through all 60 lines, erasing and rewriting everything.

### What Would Actually Work

From my analysis, **only architectural changes would fix this**:

**Option 2 - Separate StatusLine Component** (Most Practical) ✅
- Create special `<StatusLine>` component with separate rendering path
- Use absolute cursor positioning (`ESC[{row};{col}H`)
- Update status independently from main content
- **Effort**: 200-300 lines of code, 2-3 days
- **Works in tmux**: Yes!

This would require changes to Ink itself or a fork.

### For Claude Code Team

**Short Term:**
- Accept this as a known limitation of Ink
- Document it for users
- Consider adding photosensitivity disclaimer

**Medium Term:**
- Reduce status update frequency (200ms instead of 100ms)
- Suggest terminals with better buffering (Kitty, Alacritty)

**Long Term:**
- Implement StatusLine component approach (2-3 days)
- Or migrate to alternative TUI library (Blessed has screen diffing)

### Complete Documentation

I've created comprehensive documentation of the entire investigation:

📦 **Repository**: https://github.com/atxtechbro/test-ink-flickering

📄 **Key Documents**:
- [CONCLUSIONS.md](https://github.com/atxtechbro/test-ink-flickering/blob/main/CONCLUSIONS.md) - Executive summary and recommendations
- [FIX-TESTING.md](https://github.com/atxtechbro/test-ink-flickering/blob/main/FIX-TESTING.md) - All tested fixes with full source code
- [INK-ANALYSIS.md](https://github.com/atxtechbro/test-ink-flickering/blob/main/INK-ANALYSIS.md) - Complete architectural deep-dive

### Bottom Line

**The flickering cannot be fixed without significant changes to Ink's architecture.** Options are:

1. **Accept the limitation** (easiest)
2. **Implement StatusLine component** (2-3 days, universal fix)
3. **Switch to alternative TUI library** (Blessed, Textual, etc.)

Happy to discuss further or help if Claude Code team decides to pursue Option 2!

---

**Investigation Date**: November 1-3, 2025
**Test Environment**: tmux + GNOME Terminal, Linux Mint, Node v24.11.0
**Approaches Tested**: 6 (all failed)
