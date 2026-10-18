# Threat model

## What this project does and where untrusted input enters

A hardened malloc implementation, used as the default malloc implementation in GrapheneOS. The README's "Security
properties" section lists the security properties it is supposed to have. Untrusted input is any sequence of API
calls, with any pointers and from any threads, by a program with a memory corruption bug.

## How to exercise it

* Prebuilt: `/src/out/libhardened_malloc.so` (default), `/src/out-light/libhardened_malloc-light.so` (light) and
  `/src/out-android/libhardened_malloc.so` (`Android.bp` settings). Rebuild with `make CONFIG_NATIVE=false`.
* Run programs on it with `LD_PRELOAD`. `make CONFIG_NATIVE=false test` runs the tests in `test/`.
* A detection prints `fatal allocator error: <reason>` and aborts; a guard page hit is a silent SIGSEGV.

## How you rate severity

* Critical: correct API use corrupts memory, or a program bug lets the attacker write allocator metadata.
* High: a program bug defeats a deterministic protection (invalid or double free detection, zeroing, guards,
  quarantines, MTE tags), or correct use with regular inputs that real software sees causes an abort, crash or hang.
* Medium: a probabilistic protection (randomization, canaries) is weaker than documented.
* Low: an abort, crash or hang from a contrived edge case that the vast majority of software would not hit without
  memory corruption.

## Anything to leave alone

* Aborts and SIGSEGVs on detected misuse, and protections that a configuration turns off.
* Small overflows absorbed by the slack space or canary, and a wrong size for sized deallocation within one size class.
* A double free after the slot has been reused.
* `realloc` not enforcing more safety than the caller doing `malloc`, `memcpy` and `free` itself.
* The write-after-free check missing a write because the memory was purged (`memory_purge`, as in `malloc_trim`)
  and zeroed after it.
* Already known: MTE `free` not checking the tag, a stale MTE tag recurring on a slot's second or later reuse,
  non-random slab selection and guard slab placement, and pointer comparisons across mappings.
* Reported before and rejected: `init_slow_path` not being fork-safe, `errno` changes outside `free`, and ignored
  `malloc_info` write errors.
* Aborts on non-ENOMEM `mmap`, `munmap`, `mprotect` and `mremap` failures, on any `madvise` or `getrandom` failure,
  and on any failure during initialization: these are the README's aggressive sanity checks. After initialization,
  ENOMEM is handled the normal way; aborting on it would be incorrect except as an explicit opt-in, which doesn't
  exist yet.
* Anything the README lists as a non-goal, in progress or unsupported.
