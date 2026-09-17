---
name: rtk
description: >
  Enforces RTK (Rust Token Killer) CLI proxy for shell commands to cut up to 90%
  of terminal output token consumption. Auto-triggers when executing shell commands,
  building code, running tests, or checking git status.
---

# RTK - Rust Token Killer

High-performance CLI proxy that filters and compresses command outputs before they reach your LLM context window.

## Usage Rule

Always prefix terminal commands with `rtk` to minimize token consumption.

```bash
rtk git status
rtk git diff
rtk git log -n 10
rtk cargo test
rtk cargo check
rtk npm test
rtk pytest
rtk grep "pattern" src/
rtk find "*.rs" .
```

## Meta Commands

```bash
rtk gain              # Display token savings dashboard
rtk gain --history    # Command execution history with savings
rtk discover          # Detect un-proxied command execution opportunities
rtk proxy <cmd>       # Run raw command without output filtering (debugging)
rtk init -g           # Initialize global hooks and RTK.md reference
```

## Why Use RTK

RTK reduces bash output by 60–90% through intelligent grouping, truncation, deduplication, and structured formatting, preserving context window budget and improving response latency.
