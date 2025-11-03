# Ink Flickering Reproduction Test

This project is a minimal reproduction case for the screen flickering issue reported in [anthropics/claude-code#769](https://github.com/anthropics/claude-code/issues/769).

## The Issue

When Claude Code processes requests, the entire terminal buffer redraws with each status update instead of just refreshing the indicator line, causing screen flickering that can be problematic for users sensitive to flashing lights.

## Status

- ✅ **Reproduction Confirmed** (2025-11-03) - Successfully reproduced the flickering
- ⏳ **OSC133 Testing** - Tested on GNOME Terminal (inconclusive - no OSC133 support)
- ⏳ **Fix Testing** - Testing potential solutions in progress

📋 **See [FINDINGS.md](./FINDINGS.md) for detailed test results**

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

## Next Steps

1. ✅ ~~Confirm flickering reproduces in this minimal app~~
2. ✅ ~~Compare with Claude Code's behavior~~
3. ⏳ Test with OSC133 on compatible terminal (Kitty, Windows Terminal, VS Code)
4. ⏳ Try bcherny's Ink fork
5. ⏳ Profile rendering to understand the issue

## Documentation

- [REPRODUCED.md](./REPRODUCED.md) - Reproduction confirmation and analysis
- [FINDINGS.md](./FINDINGS.md) - Detailed test results log
- [TESTING-GUIDE.md](./TESTING-GUIDE.md) - Comprehensive testing instructions
