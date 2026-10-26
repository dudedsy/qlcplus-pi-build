# Patches

Optional patches the workflow can apply to the QLC+ source before building. Pass their file names, space-separated, in the `patches` input. Patched builds get their own package version (`+p<date>.<names>` for a release tag, so they install over the plain release) and their own prerelease. `SOURCE.txt` in the release lists the patches and their hashes.

## universe-tick-direct.patch

Against QLC+ 5.3.0 (applies to the engine, so it affects both the v4 and v5 UIs).

**Problem.** Every 20 ms, the MasterTimer thread emits `tickReady()`. That signal was connected to each universe's `tick()` slot with `Qt::QueuedConnection`. The Universe objects are created on the main thread and never moved, so the queued call runs in the main (GUI) thread's event loop. `tick()` only releases a semaphore that wakes the Universe thread, which builds and sends the output. Whenever the GUI thread is busy, output stops; when it catches up, the backlog of ticks is delivered back to back.

On a Raspberry Pi 3B running QLC+ 5.3.0 with a large show, this produced output gaps of 100 ms to over 1 s, and command latencies of several hundred ms. Every gap lined up with the GUI thread running flat out, while the MasterTimer kept ticking normally.

**Change.**
- `MasterTimer::timerTick()` calls `tick()` on each universe directly. It does this at the end of the tick, while the universe list is still claimed (`claimUniverses()`), so no universe can be deleted during the call.
- The two queued `connect()` calls in `InputOutputMap::addUniverse()` are removed. `tickReady()` is still emitted; nothing else in the tree connects to it.
- **Load guard.** Ticks are skipped while `Doc::loadStatus() == Doc::Loading`. Loading reconfigures universes from the main thread without synchronising with the Universe threads. Under the old wiring the universes were idle during a load anyway, because the busy main thread couldn't deliver ticks, so this keeps load behaving as before.

**Tests.** The patch adds `engine/test/universetick`:

| Test | Without patch | With patch |
|---|---|---|
| Output passes during a 460 ms main-thread block | 0 (one 520 ms gap) | 23 (max gap 20 ms) |
| Universe passes concurrent with 100k fixture add/delete ops on the main thread | 0 | ~98 |
| Universe passes during `Doc::loadXML` of 3000 fixtures | 0 | 0 (14 without the load guard) |

The rest of the engine unit test suite gives identical results with and without the patch (x86_64, Qt 6.4).

**Known limitation.** Universe has no lock between its processing path and its configuration setters (channel capabilities, modifiers, default values, output/input patches). Those races exist without the patch too, during the few ms a universe pass runs. The patch removes the implicit idling that a busy GUI thread used to cause during long main-thread operations other than project loads, such as bulk fixture adds while the timer runs. The fixture-churn test exercises exactly that, with no crash seen so far. A proper fix is a per-universe lock, which is a separate change.
