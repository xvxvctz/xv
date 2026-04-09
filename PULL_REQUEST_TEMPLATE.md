## Problem

The `read_game_state` example fails to read any game state despite successfully attaching to the CS2 process and discovering offsets. The root cause is a chain of issues:

### 1. Sanity check threshold too low (`offsets_discovery.rs`)
The `scan_local_player` function had a hardcoded sanity check of `0xA00000` (10 MiB), but modern CS2 builds have `libclient.so` text sections exceeding that. The scanned offset `0x39999d8` (~60 MiB) is valid but was rejected, causing the fallback to use stale hardcoded offsets.

### 2. Stale hardcoded fallback offsets (`offsets.rs`)
The fallback `Offsets::load()` had old values that don't match the current CS2 build. When the scanner rejected its own results, these stale values were used, pointing to garbage memory.

### 3. `MAX_SCAN_BYTES` too small
`MAX_SCAN_BYTES` was 32 MiB but the offsets being discovered are beyond that range on current builds. Increased to 64 MiB.

### 4. Compiler warnings
Unused parameters, constants, functions, and imports.

## Changes

### `src/process/offsets_discovery.rs`
- Raise sanity check from `0xA00000` to `MAX_SCAN_BYTES` (64 MiB)
- Increase `MAX_SCAN_BYTES` from 32 MiB to 64 MiB
- Prefix unused `module_base` params with underscore
- Add `#[allow(dead_code)]` on unused constants and functions

### `src/process/offsets.rs`
- Update hardcoded fallback offsets to match current CS2 build

### `examples/read_game_state.rs`
- Remove unused import `offsets_discovery::discover_offsets`