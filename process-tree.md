# process-tree: macOS Process Lineage Tool

A C++23 command-line tool with accompanying browser tool for vizualing "which program started which" on macOS. It combines a snapshot of existing processes with eslogger (Apple's Endpoint Security tool with no special entitlements). This is specifically for macOS, and is not useful for any other OS. MacOS has similar process behavior to linux, but with its own twists that this project takes into account.

| | |
|---|---|
| **Timeframe** | ~1 week active work |
| **Languages** | C++23, JavaScript, HTML/CSS |
| **Size** | ~1,000 lines C++; ~1,000 lines JS/HTML/CSS |
| **Platform / APIs** | macOS: Endpoint Security (`eslogger`), libproc, sysctl, Mach, Security.framework |
| **Libraries / tooling** | glaze (JSON), D3 v7, CMake, claude code |
| **Relevant to** | Endpoint security, detection engineering, incident response tooling, macOS/systems programming |

## The problem

Event and process logging is an essential part of IR and threat detection. macOS gives us some useful resources, but they don't paint a complete picture individually:

- A snapshot of existing processes via LibProc.
- Eslogger / Endpoint Security API only gives info on new processes.

This project combines those sources into a single tree. While this sounds simple, there are many surface-level challenges: pid reuse, skipped events, event volume, noise, etc.

## Skills demonstrated

- Modern C++: optional struct fields, custom hashing, compile-time JSON reflection, ...
- macOS internals: Endpoint Security, Mach ports, audit tokens, libproc, sysctl, code signing
- Data modeling for messy, partial, time-ordered event data
- Empirical validation: checking behavior against captures instead of only trusting documentation
- Defensive programming: race handling, graceful degradation (eslogger got a schema-breaking update mid-project), malformed-input guards 
- Quick data visualization: cheap web tool for demos
- AI collaboration: utilized Claude Code to accelerate development - testing, debugging, fully-generated unimportant components, research, planning


## Technical problems solved

- **Process identity.** The first version assumed spawning and exec behaved as documented. Testing against a real capture and probe programs showed otherwise: `exec` keeps the PID but bumps a version counter, and `posix_spawn` emits a fork event. The tracker was reworked so each program image is its own node keyed by (pid, pidversion), with exec chains as explicit links. This was the larger of the two feature commits (+398/−165 lines).
- **Orphaned processes.** When a parent exits, macOS reassigns the child to `launchd` and the kernel-reported parent is lost. The tool recovers the true creator from fork events (recorded) or a kernel unique-ID field (snapshot).
- **Incomplete data.** Events can be missing, processes can predate the capture, and timestamps come at mixed precision. The tool normalizes timestamps and, where a creator is unknown, infers it by matching a process's birth time against the lifetimes of candidate parents. Unresolvable references are kept and shown as "missing record" instead of being dropped.
- **Parent vs. responsible process.** These are separate relationships on macOS. Everything a shell launches is attributed to Terminal, not to the shell. The tool models both. The responsible-process call is a private libSystem symbol resolved at runtime with `dlsym`, so an OS change degrades to "unknown" instead of breaking the build or the run.
- **Low-level OS interfaces.** Uses libproc, binary parsing of `KERN_PROCARGS2` buffers, Mach audit tokens, the SecCode code-signing API, and a hand-declared kernel struct that is absent from the public SDK. It handles races such as a process exiting mid-scan or a PID being reused between lookups.

## Scope and limitations

- Personal project; no external users or production deployment.
- Batch, not live: it processes a capture file after the fact rather than monitoring in real time.
- macOS only; requires root.
- Relies on a few undocumented OS interfaces, which are flagged in the code as potentially unstable across OS versions.
- No automated test suite; validation so far has been against real captures.

## Repository layout

```
src/   C++ tool: snapshot (existingp), event parser (esparser), tree builder (ptracker)
viz/   Browser visualizer (index.html, app.js, style.css, vendored D3)
```

Build: `cmake -S . -B build && cmake --build build`
