# Trap and Interrupt Handling Enhancements for xv6

## Description
This project implements kernel backtrace for debugging, and a `sigalarm`/`sigreturn` system call pair for periodic user-level timer callbacks (signals-like mechanism).

## Objectives
- Implement `backtrace()` to walk kernel stack frames using frame pointer (s0)
- Add `sigalarm(ticks, handler)` to request periodic timer callbacks
- Add `sigreturn()` to resume interrupted user code after handler runs
- Handle timer interrupts in `usertrap()` to invoke user alarm handlers

## Implementation
- `backtrace()` — Reads `s0` via `r_fp()`, walks stack frames using saved fp at offset -16, prints return address at offset -8; stops at stack page boundaries
- `sigalarm(interval, handler)` — Stores interval, handler, and tick counter in `struct proc`; initialize in `allocproc()`
- `sigreturn()` — Restores saved trapframe registers to resume interrupted user code
- `usertrap()` timer handling — On timer interrupt (which_dev==2), decrements tick counter; when zero, saves user registers in proc, sets `epc` to handler, re-arms counter

## Build and Run Instructions

To build and run xv6, you need a RISC-V toolchain and QEMU emulator

I used WSL and ran Ubuntu 24.04 specifically, so there may be differences with different distributions and versions. Once you have wsl installed this will install the version of Ubuntu I used, but removing "-24.04" will install the latest.

```bash
wsl.exe --install Ubuntu-24.04
```

Next update and install the packages I used

```bash
sudo apt-get update && sudo apt-get upgrade

sudo apt-get install git build-essential qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu  
```

Lastly clone this repository, change directory into it, and run QEMU.

```bash
git clone https://github.com/silasmhd/xv6Traps

cd xv6Traps

make qemu
```

To exit QEMU press `ctrl + a` then `x`. 
_This won't work with capslock on._

## Tests

Once you have QEMU running, you can run the test for this project and look at the them in `test-traps`.

```bash
make test
```

## Files Modified
- `kernel/printf.c` — `backtrace()`, call in `panic()`
- `kernel/riscv.h` — `r_fp()` inline asm to read `s0`
- `kernel/defs.h` — `backtrace()` prototype
- `kernel/sysproc.c` — `sys_sigalarm()`, `sys_sigreturn()`
- `kernel/syscall.h` — `SYS_sigalarm`, `SYS_sigreturn` numbers
- `kernel/syscall.c` — syscall dispatch
- `kernel/proc.h` — `alarm_interval`, `alarm_handler`, `alarm_ticks`, saved trapframe fields
- `kernel/proc.c` — initialize alarm fields in `allocproc()`
- `kernel/trap.c` — timer interrupt handling in `usertrap()` for alarms
- `user/usys.pl` — stubs for `sigalarm`, `sigreturn`
- `user/user.h` — prototypes for `sigalarm`, `sigreturn`
- `user/alarmtest.c` — test program
- `Makefile` — add `_alarmtest` to `UPROGS`

## References
- xv6 book, Chapter 4: Traps and System Calls
- RISC-V Privileged Architecture Specification (trap and interrupt behavior)
- Original xv6-riscv repository: https://github.com/mit-pdos/xv6-riscv
