# IRIX 7

IRIX 7 is two related but distinct tracks:

1. **IRIX 6.5.7m source and toolchain.** Assemble and transcribe the IRIX 6.5.7m
   source, use MIPSpro as the behavioural oracle, and build a modern GCC and
   binutils 2.47 cross-toolchain for `mips-sgi-irix6.5`. The source rebuild
   stays private where IRIX code is involved.
2. **IRIX 7 OS.** An independent, clean-room, GPL-3.0 Rust-native operating
   system for IRIX-derived SGI MIPS machines, starting with the Indy (IP24).
   It has its own assembly PROM, n32 Rust kernel, loadable modules and a
   POSIX/userland effort. It is not a rewritten IRIX 6.5.7m or a line-for-line
   port, and it does not yet provide IRIX 6.5 binary compatibility.

The tracks share the hardware target and the reference evidence, not a
codebase. The private track provides the MIPSpro oracle and build reference;
the public OS is an independent clean-room implementation, with an assembly
PROM and a Rust kernel and modules. The OS is in bring-up: M1 is partial and
M2 is still ahead.

> **[The Tree of IRIX 7](https://irix7.github.io/os/tree/)** — the living
> roadmap for the Rust-native OS, from its foundations to the v1.0 canopy.

## Current State

### IRIX 6.5.7m source and toolchain

The private [`irix6`](https://github.com/irix7/irix6) tree is assembled, and
transcription continues where the source is incomplete. SGI did not ship source
for the graphics and audio code, so the
[Ghidra fork](https://github.com/irix7/ghidra) and decompilation fill those
gaps. The stock rebuild is in progress, not complete: the selected IP22 kernel
object set, the `lboot` link contract and a rebuilt-kernel boot remain open
([project #1](https://github.com/irix7/project/issues/1),
[#13](https://github.com/irix7/project/issues/13)).

On the public toolchain side:

- The modern GCC cross and binutils 2.47 target `mips-sgi-irix6.5`. The binutils
  series passes the controlled o32 and n32 guest smokes.
- A native `hinv` reference from MIPSpro and a GCC `hinv` build match.
- An initial o32 libc build from the tree compiles the selected C and assembly
  sources; a static hello links against it, runs in the guest, and matches the
  MIPSpro oracle output. This is a bounded proof, not stock libc/libm completion
  ([project #9](https://github.com/irix7/project/issues/9)).
- Building the kernel with GCC and booting the rebuilt IP22 image are the next
  source-rebuild milestones
  ([project #39](https://github.com/irix7/project/issues/39)).

### IRIX 7 OS

The current release is [v0.1.3](https://github.com/irix7/os/releases/tag/v0.1.3).
Its assembly PROM sizes, maps and tests DRAM, brings up the serial console,
LZ4-decompresses the n32 kernel and jumps to it. The Rust kernel provides Seeq
networking, DHCP/DNS/NTP, HTTP/HTTPS and TLS, the REX3 boot log, telnet and
SHA-256-checked module autoload. It loads a static Rust `std` user image from
SCSI target 2.

- **Modules.** A flat module image loads into a fixed 2 MiB module area, is
  SHA-256 checked, and is called through a versioned C service table. The
  [module SDK](https://github.com/irix7/os-sdk) owns the ABI, targets, linkers
  and examples. The A/B slots are fixed; a general module multiplexer is still
  missing.
- **M1 single-node.** The kernel starts a local namespace/CAS, actor table and
  capability policy at boot. An optional slot-B Wasm module reproduces the
  local policy as a boot-time proof; the M1 line reports zero mesh traffic and
  the boot CI asserts it. M1 is a partial, iris-only software gate; there is no
  resident Wasm core or mesh, and M2 is still ahead.
- **POSIX and userland.** The native user path has a static Rust `std` image
  with a small ramfs, `stat`, `read_dir` and `chdir`; the native hello also
  exercises an in-kernel pipe ring and `tcgetwinsize`/`tcsetwinsize`. A
  synthetic o32 probe exercises classic `open`/`read`/`write`/`close`; it is not
  an IRIX 6.5 application. The 42-symbol C shim is a bring-up shim, not the
  final libc. The [relibc fork](https://github.com/irix7/relibc)'s n32
  Linux/QEMU gate passes TLS, `setjmp` and pthreads; its native IRIX 7 backend
  has not run.

**What runs where.** All current boot evidence is on the
[iris](https://github.com/irix7/iris) Indy emulator. No real Indy IP24 has been
characterised or booted, and there is no recorded real-hardware pass; the lab
follow-up is [os #16](https://github.com/irix7/os/issues/16). The OS is not a
finished IRIX replacement and does not yet provide IRIX 6.5 binary
compatibility.

### Known limits

- No process table, threads/TLS, `posix_spawn`, signal delivery, termios or
  socket API. The user path is a single-address-space bring-up, not a process
  isolation boundary.
- No resident Wasm core or mesh; M1 is partial and M2 is ahead.
- No real-Indy validation. The full graphics stack, XFS compatibility and
  broader driver coverage remain ahead.
- The compressed PROM is nearly full (about 3 KiB free in the current image),
  so policy, modules and userland ship outside the bootstrap.

## Repositories

| Repository | What it holds |
| --- | --- |
| [os](https://github.com/irix7/os) | The clean-room Rust OS, its docs and [the living roadmap](https://irix7.github.io/os/tree/) |
| [project](https://github.com/irix7/project) | The toolchain hub: GCC and binutils patches, the rig, the MIPSpro oracle, rebuild scripts and documents |
| [os-sdk](https://github.com/irix7/os-sdk) | The module ABI, module and user SDK, targets, linkers and examples |
| [relibc](https://github.com/irix7/relibc) | The IRIX 7 n32 big-endian relibc fork |
| [gcc](https://github.com/irix7/gcc) | The `rust-lang/gcc` fork with libgccjit; the IRIX patch series lives in `project` |
| [binutils-gdb](https://github.com/irix7/binutils-gdb) | The binutils 2.47 fork and the IRIX dynamic-linking patch |
| [ghidra](https://github.com/irix7/ghidra) | The Ghidra fork with IRIX and SGI MIPS support |
| [iris](https://github.com/irix7/iris) | The Indy emulator fork used for the guest and IRIX 7 gates |
| [harness](https://github.com/irix7/harness) | Agent orchestration, evidence and verification for the transcription workflow (planned) |
| [irix6](https://github.com/irix7/irix6) | The IRIX 6.5.7m source tree — **private, never published** |
| [reference](https://github.com/irix7/reference) | SGI/IRIX reference documentation — **private, never published** |

## Publication rule

The public repositories publish toolchain work and clean-room project material
only: GCC and binutils patches, scripts, documents, the harness, the module and
user SDK, the relibc fork and the IRIX 7 OS. They publish no IRIX source,
headers, binaries, disk images or sysroot content. IRIX is proprietary SGI
software. The `irix6` source tree and the `reference` documentation are private
and are never distribution channels.

## Documents

- The current stock-rebuild scope and status:
  [project #1](https://github.com/irix7/project/issues/1) and
  [`CONTEXT.md`](https://github.com/irix7/project/blob/main/CONTEXT.md).
- The toolchain, rig, oracle and publication boundary:
  [project docs](https://github.com/irix7/project/tree/main/docs) and the
  [publication note](https://github.com/irix7/project/blob/main/docs/publication.md).
- The OS plan, gates and limits:
  [milestones](https://github.com/irix7/os/blob/main/docs/plan/milestones.md),
  [system architecture](https://github.com/irix7/os/blob/main/docs/spec/system.md),
  [module ABI](https://github.com/irix7/os/blob/main/docs/spec/modules.md) and
  [POSIX baseline](https://github.com/irix7/os/blob/main/docs/spec/posix.md).
- The first Rust `std` result:
  [std-hello-bringup](https://github.com/irix7/os/blob/main/docs/research/std-hello-bringup.md).
- The roadmap: [The Tree of IRIX 7](https://irix7.github.io/os/tree/) and the
  [OS documentation](https://irix7.github.io/os/).
