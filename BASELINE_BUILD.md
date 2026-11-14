# Baseline build (pre-AVX2 CPUs)

This fork publishes **baseline x86-64** builds of rtk for CPUs that **do not have AVX2**.

## Why

Upstream's official releases (v1.2.x) are compiled with `target-cpu=native`.
On a machine whose CPU has AVX but **not** AVX2 — e.g. Intel **i5-3337U / Ivy Bridge**
(`/proc/cpuinfo` flags show `avx` but no `avx2`) — the binary dies on the first call:

```console
$ rtk --version
Illegal instruction (core dumped)      # exit 132 (SIGILL)
```

`objdump` on the upstream binary confirms AVX2 instructions:

```console
$ objdump -d rtk | grep -c -- '%ymm'
23286
$ objdump -d rtk | grep -c -- 'vpermd'
28
```

## The fix

Build with the plain x86-64 baseline target, so the compiler never emits AVX/AVX2:

```bash
RUSTFLAGS="-C target-cpu=x86-64" \
  cargo build --release --target x86_64-unknown-linux-musl
```

Automated by [`.github/workflows/baseline-x86_64.yml`](.github/workflows/baseline-x86_64.yml),
which also prints an instruction scan of the produced binary.

## Download

- Release: [`baseline-v0.51.1-x86_64`](https://github.com/zng8418/rtk/releases/tag/baseline-v0.51.1-x86_64)
- Asset: `rtk-x86_64-unknown-linux-musl-baseline.tar.gz` (+ `.sha256`)

```bash
tar xzf rtk-x86_64-unknown-linux-musl-baseline.tar.gz
install -m 0755 rtk ~/.local/bin/rtk
rtk --version      # -> rtk 0.51.1
```

The static musl build runs on any glibc/musl x86-64 Linux, old hardware included.

## Verified on the real target machine

| check | result |
|---|---|
| CPU | Intel i5-3337U @ 1.80GHz (Ivy Bridge, `avx` yes / `avx2` no) |
| `rtk --version` | `rtk 0.51.1`, exit 0 |
| `rtk git status` | ok |
| `rtk rewrite "git diff --stat"` | ok |
| `rtk proxy "git --version"` | ok |
| `rtk gain` | ok |

## Notes for this fork

* Upstream release uploads are signed with a GitHub App that a fork cannot use, so
  this fork's release workflow uploads with the built-in `GITHUB_TOKEN`
  (already handled in `.github/workflows/release.yml`).
* The `stale` workflow uses `GITHUB_TOKEN` instead of the App token as well.
