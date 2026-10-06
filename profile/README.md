# IRIX 7

IRIX 7 rebuilds IRIX 6.5.7m with a modern GCC. Then it modernises the result into the same IRIX OS but written in RUST.

The first deliverable is the stock rebuild. It reproduces IRIX 6.5.7m from its own source tree with the new toolchain. The second deliverable is the modernised fork. It changes the rebuilt code to make a new IRIX-derived operating system.

## The plan

1. Build a modern cross compiler for `mips-sgi-irix6.5`. Use GCC 16.2 and binutils 2.47.
2. Run the compiler inside an emulated SGI Indy. Verify the output against the MIPSpro oracle.
3. Rebuild the whole tree with the new compiler. First the libraries, then the operating system.
4. Modernise the rebuild.

The work is private where IRIX itself is involved. The toolchain work is public.

## Repositories

| Repository | What it holds |
| --- | --- |
| [project](https://github.com/irix7/project) | The hub: the patch series, the build scripts, the documents |
| [gcc](https://github.com/irix7/gcc) | The upstream GCC mirror that the IRIX series applies to |
| [binutils-gdb](https://github.com/irix7/binutils-gdb) | The binutils mirror and the IRIX dynamic-link patch |
| [ghidra](https://github.com/irix7/ghidra) | The Ghidra fork with IRIX and SGI MIPS support |
| [iris2](https://github.com/irix7/iris2) | The Indy emulator fork that runs the IRIX guest |
| [harness](https://github.com/irix7/harness) | The agent harness for the transcription work (planned) |
| [irix6](https://github.com/irix7/irix6) | The IRIX 6.5.7m source tree (private) |

## Publication rule

The public repositories publish toolchain work only: patches, scripts and documents. They publish no IRIX source, headers, binaries or sysroot content. IRIX is proprietary SGI software.

## Documents

Start at the project repository. Its `CONTEXT.md` defines the vocabulary. Its `docs/` folder explains the toolchain, the rig, the oracle and the decisions.
