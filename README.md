# Ingert

A decompiler for *Trails through Daybreak*, *Trails through Daybreak 2*, *Trails beyond the Horizon*, and *Ys X: Nordics*. It can decompile and recompile every script[^mon9996_c00] with perfect roundtripping.

To use, simply drag a file or folder onto the executable.

[^mon9996_c00]: Except *Daybreak*'s `mon9996_c00.dat`, which has a different format from all the other scripts.

## About this fork

This is a personal fork of [Aureole-Suite/Ingert](https://github.com/Aureole-Suite/Ingert), maintained while working on text swaps between the XSeed and EVO voice-mod scripts for the *Trails in the Sky 1st Chapter* remake (Sora no Kiseki FC remake — hence the repo name `ingert-sora1`). It adds:

- **A fix for decompiling modded/repacked 1st-chapter scripts.** The EVO voice mod's `mp0010_05.dat` failed to decompile with `could not read code for LP_CHECKED_BOARD (54): invalid function id 65535`. Root cause was in `ingert/src/scp/io/code.rs`: when a function had an explicit `end` upper bound, the read loop ignored the natural-end heuristic (`Op::Return` past all forward labels) and kept consuming orphan bytes the mod had left between functions, eventually decoding garbage as a `CallLocal`. The fix (commit [`2a410e0`](../../commit/2a410e0)) makes the natural-end heuristic apply unconditionally and treats `end` purely as a safety net:

  ```rust
  let is_end = (op == Op::Return && pos >= extent.get())
      || end.is_some_and(|e| f.pos() >= e);
  ```

  Verified on the EVO + XSeed corpora (151/151 each, including the previously-failing `mp0010_05.dat`). I'll send this upstream once it's had more mileage on other 1st-chapter scripts.

- **Prebuilt Windows binaries via CI.** Upstream doesn't currently ship releases, so this fork builds `ingert.exe` on every push and tag using the MSYS2 UCRT64 toolchain. Grab the latest from the [Actions tab](../../actions) or the [Releases page](../../releases).

- **MSRV pinned to 1.95** in `Cargo.toml` to match the toolchain used here.

Bugfixes and portability tweaks are intended for upstream; only release plumbing and in-progress experiments live here long-term.
