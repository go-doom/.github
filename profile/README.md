<p align="center"><img src="https://raw.githubusercontent.com/go-doom/brand/main/social/go-doom.png" alt="go-doom" width="720"></p>

# go-doom

**Pure-Go DOOM (id Tech 1, 1993) — bare-metal, no cgo.**

`go-doom` is the pure-Go port of the original [DOOM](https://en.wikipedia.org/wiki/Doom_(1993_video_game))
engine, derived from [AndreRenaud/gore](https://github.com/AndreRenaud/gore)
(itself a hand-port from the [id Software 1997 source release](https://github.com/id-Software/DOOM)),
with a [TamaGo](https://github.com/usbarmory/tamago) bare-metal backend
+ a 4-gate **provable-test protocol** validating engine determinism /
GPU fidelity / audio events / audio waveform.

Sibling of the [`go-quake1` family](https://github.com/go-quake1)
(`go-quake1` / `go-quake2` / `go-quake3` — the id Tech 1 / 2 / 3
ports). Both engines share the same TamaGo backend conventions,
the same `CGO_ENABLED=0` discipline, and the same 4-gate provable-
test harness.

## Repositories

| Repo | Latest | Role |
|---|---|---|
| [`engine`](https://github.com/go-doom/engine) | — | The DOOM engine, TamaGo backend, harvest-reference oracle, demos |
| [`brand`](https://github.com/go-doom/brand) | — | Logos, social previews, favicons |
| [`.github`](https://github.com/go-doom/.github) | — | This profile + shared org workflows |

## How it works

- **Engine** — pure-Go DOOM (`doom.go`, `seed.go`, etc., the AndreRenaud/gore
  fork hand-cleaned for bare-metal use)
- **`backend/tamago/`** — bare-metal adapters: frontend (run loop) +
  gpu (framebuffer through [go-virtio](https://github.com/go-virtio) GPU) +
  sound (PCM into virtio-snd) + input (virtio-input keyboard)
- **`embedwad/`** — shareware `DOOM1.WAD` in-tree (id Software's freely-
  redistributable shareware grant) so CI is reproducible
- **`cmd/harvest-reference/`** — reference oracle that runs a canonical
  demo and records the per-tic frame + audio digests the 4 provable-test
  gates compare against
- **`example/`** — host-side demos (SDL, terminal, web, Ebitengine) that
  share the same engine + a different frontend

## Project standards

- **Pure Go.** `CGO_ENABLED=0` on the engine + TamaGo backend. The host-side
  demos under `example/` are allowed to use cgo (SDL, Ebitengine) because
  they run in user-space, not on bare metal.
- **Reference-mirror traceability.** Every Go file links to the upstream
  C/Go function it derives from.
- **4-gate provable-test protocol.**
  - **GATE A** — engine determinism, BYTE-EQUAL frames at checkpoint tics
  - **GATE B** — guest virtio-gpu fidelity, χ² ≤ tolerance
  - **GATE C-1** — audio event stream, BYTE-EQUAL CacheSound/PlaySound log
  - **GATE C-2** — guest WAV bounded tolerance (per-second RMS envelope)
- **6-arch CI.** All six 64-bit Go targets — `amd64`, `arm64`, `riscv64`,
  `loong64`, `ppc64le`, `s390x` — green on each PR (engine itself; the
  TamaGo + cgo-frontend demos run only on the relevant subset).
- **BSD-3-Clause** wrapper + **GPL-2.0-or-later** carve-out on the engine
  subtree (inherited from the id Software source release).

## Why pure-Go DOOM?

The same reason as [go-quake1](https://github.com/go-quake1) and
[go-virtio](https://github.com/go-virtio): **one self-contained Go binary,
zero cgo, runs on bare metal** — the TamaGo target the
[cloud-boot](https://github.com/cloud-boot) loader hands control to. A C
engine drags glibc + SDL + the cgo runtime; pure-Go drags exactly
`runtime` + `math`. DOOM's CPU-only software renderer fits the bare-metal
model: no GPU shaders, all CPU rasterisation — exactly the workload
pure-Go + go-asmgen SIMD is built for.

## Who uses it

[cloud-boot](https://github.com/cloud-boot) — the bare-metal TamaGo +
UEFI demo target. The 4-gate provable-test discipline ships with this
engine and is carried forward into [go-quake1](https://github.com/go-quake1)
unchanged: same gates, same reference recordings, against the same
go-virtio + go-asmgen substrate.

## Landing page

Project landing page: <https://go-doom.github.io>.
