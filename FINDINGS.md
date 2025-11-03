# Test Findings Log

This document tracks all test results as we investigate potential fixes for the Ink terminal flickering issue.

## Test Entry Format

Each test entry includes:
- **Date**: When the test was conducted
- **Test Type**: What was being tested
- **Environment**: Terminal, OS, multiplexer, etc.
- **Command**: How to reproduce
- **Result**: What happened
- **Conclusion**: What we learned

---

## 2025-11-03: OSC133 Sequences on GNOME Terminal

### Test Details
- **Date**: November 3, 2025
- **Test Type**: OSC133 prompt marking sequences
- **Tester**: atxtechbro
- **Commit**: e0088bc

### Environment
- **OS**: Linux Mint (Ubuntu-based)
- **Terminal Emulator**: GNOME Terminal (`gnome-terminal-server`)
- **Multiplexer**: tmux
- **Node.js**: v24.11.0
- **Ink Version**: 4.4.1

### Command
```bash
npm run start:osc133
```

### Result
❌ **Still flickers** - No improvement over baseline test

The flickering behavior was identical to the vanilla test (`npm start`):
- Entire screen redraws on each status update
- Visual flashing every 100ms
- Full component tree redraw observed

### Analysis

**Why this result was expected:**

GNOME Terminal **does not support OSC 133** sequences. From the [GNOME Terminal documentation](https://gitlab.gnome.org/GNOME/gnome-terminal), OSC 133 (semantic prompt marking) is not implemented.

**What this means:**
- GNOME Terminal silently ignores the OSC 133 escape sequences
- The test effectively ran the same code as the baseline
- This is an **inconclusive test** for OSC 133 efficacy

### Conclusion

✅ **Confirmed**: Flickering reproduces on GNOME Terminal
❌ **Inconclusive**: Whether OSC 133 helps (terminal doesn't support it)
📋 **Next Step**: Test on OSC 133-compatible terminal (Kitty, Windows Terminal, VS Code)

### Terminal OSC 133 Support Matrix

| Terminal | OSC 133 Support | Tested |
|----------|----------------|---------|
| GNOME Terminal | ❌ No | ✅ Yes (this test) |
| Kitty | ✅ Yes | ⏳ Pending |
| iTerm2 | ✅ Yes | ⏳ Pending |
| Windows Terminal | ✅ Yes | ⏳ Pending |
| VS Code Terminal | ✅ Yes | ⏳ Pending |
| Alacritty | ❌ No | ⏳ Pending |
| Konsole | ❌ No | ⏳ Pending |

---

## Template for Future Tests

### 2025-MM-DD: [Test Name]

### Test Details
- **Date**:
- **Test Type**:
- **Tester**:
- **Commit**:

### Environment
- **OS**:
- **Terminal Emulator**:
- **Multiplexer**:
- **Node.js**:
- **Ink Version**:

### Command
```bash

```

### Result


### Analysis


### Conclusion


---

## Summary of All Tests

| Date | Test | Environment | Result | Conclusive? |
|------|------|-------------|--------|-------------|
| 2025-11-03 | OSC133 | GNOME Terminal + tmux | Still flickers | ❌ No (no OSC133 support) |
| TBD | OSC133 | Kitty | ? | ⏳ Pending |
| TBD | bcherny/ink fork | GNOME Terminal | ? | ⏳ Pending |

## Next Tests to Run

### High Priority
1. **OSC133 on Kitty** - Proper OSC133 support test
   - Install: `sudo apt install kitty` or use installer script
   - Run: `npm run start:osc133`
   - Compare with: `npm start`

2. **bcherny's Ink fork** - Test the fork mentioned in the issue
   - Modify: `package.json` to use `github:bcherny/ink#master`
   - Run: `npm install && npm start`
   - Compare with: vanilla Ink

### Medium Priority
3. **Different Ink versions** - Check if version matters
4. **Alternate screen buffer** - Test if that helps
5. **Update throttling** - Reduce update frequency

### Low Priority
6. **Profile with strace** - See actual syscalls
7. **Test on different distros** - Check OS-specific behavior
