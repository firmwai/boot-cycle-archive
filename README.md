# Boot Cycle Archive

Centralized store of boot cycle journals captured from QEMU-emulated firmware (ast2600-evb).
E2E test failures automatically dump their journals here. Interesting journals are then
curated into scenarios that drive improvements to Linux boot process parsing.

## What We Capture

Each snapshot is a timestamped directory containing:

| File | Purpose |
|------|---------|
| `journal.jsonl` | Full event stream (terminal output, state transitions, operations) |
| `console.txt` | Interleaved JSON events + raw boot/serial output |
| `summary.json` | Boot metadata: key events, timing, entry counts |
| `failure.txt` | (failures only) Assertion traceback or error message |

## What Makes a Journal Interesting

- **Timing oddities** — unusual delays between boot stages, slow login, sluggish network bring-up
- **Output variations** — unexpected kernel messages, different device enumeration order, missing expected output

## Directory Structure

```
cycles/
  passes/<machine>/     # Successful boots (1 representative snapshot per test)
    <test_name>/
      <timestamp>/      # e.g. 20260310T225142Z
  failures/<machine>/   # Failed boots (all snapshots kept for pattern analysis)
    <test_name>/
      <timestamp>/
  debug/<machine>/      # Manual debug sessions
```

## Maintenance

Redundant data is cleaned up by the `archive-cleanup` skill:

- **Passes**: Only 1 snapshot per test is kept (the latest). Identical boot structure
  across runs means extra snapshots add bulk without new signal.
- **Mock data**: Removed entirely — synthetic output is not useful for real boot parsing.
- **Failures**: All snapshots retained. Multiple runs of the same failure reveal
  flaky patterns and environmental variations.
