# CSA04 – Operating Systems – Assignment 2 (CO2)
**Process management, scheduling, threads, IPC, synchronisation and deadlock handling**

| | |
|---|---|
| **Name** | Nithish G |
| **Register Number** | 192511478 |
| **Course** | CSA04 – Operating Systems |
| **Report** | [`REPORT.md`](REPORT.md) and `CSA04_Assignment2_192511478_NithishG.docx` |

## Environment used

| Item | Value |
|---|---|
| Machine | Apple M4 (10 cores: 4 performance + 6 efficiency), 16 GB RAM |
| OS used for the runs in `outputs/` | macOS 27.0.1 (Darwin / XNU, POSIX) |
| Compiler | Apple clang 21.0.0 (invoked as `gcc`) |
| Also builds on | Linux (Ubuntu 24.04, gcc 13) with the same command, without the macOS flags |

Exact compile command:
```
gcc -std=c11 -Wall -Wextra -O2 -pthread <file>.c -o <name>
```
On macOS, add `-D_DARWIN_C_SOURCE`. The Q5 semaphore programs also get `-include Q5/macos_sem.h`, because macOS does not implement unnamed POSIX semaphores (`sem_init` returns ENOSYS). The shim is force-included, so **the given source code is unchanged**. On Linux it is not used.

**Build and run everything:** `./run_all.sh` (outputs go to `Qn/outputs/`).

## Questions, programs and results

| Folder | Programs | Run | Result |
|---|---|---|---|
| Q1 | `fork_states.c`, `ctxswitch.c`, `sessions_bench.c` | `./fork_states`, `./ctxswitch`, `./sessions_bench` | Children seen with ps in states S (waiting), R (running) and Z (zombie). Process switch 1.6 µs vs thread 1.1 µs. 2,000 sessions: process-per-student 448 ms, thread-per-student 47 ms, pool of 64 86 ms |
| Q2 | `rr.c` (**Complete the Code**), `rr_quantum.c`, `sched_sim.c`, `ageing.c` | `./rr`, `./rr_quantum 1`, `./sched_sim`, `./ageing` | RR output matches expected. q=1: avg WT 12.5, 23 switches; q=20: avg WT 8.75, 3 switches. MLFQ gives short jobs 5.25 s avg wait |
| Q3 | `plates.c`, `plates_proc.c` | `./plates`, `./plates_proc 8` | 64 cameras: 7.7× speed-up with 16 threads on 10 cores. Unprotected counter lost ~1,000,000 of 2,000,000 updates |
| Q4 | `pipe_ipc.c` (**Complete the Code**), `pipe_noclose.c`, `ipc_bench.c` | `./pipe_ipc`, `./ipc_bench` | Output matches expected. Without `close(fd[1])` it hangs. 100 MB: pipe 17.9 ms vs shared memory 9.3 ms |
| Q5 | `bb.c` (**Complete the Code**), `bb_multi.c`, `booking.c`, `counter_bench.c`, `peterson.c`, `macos_sem.h` | `./bb`, `./bb_multi 4 4`, `./booking`, `./counter_bench`, `./peterson` | 100/100 runs give sum = 210. 4P/4C: every item consumed exactly once; without the mutex ~11,000 duplicates. Per-coach lock ≈ 10× the throughput of a global lock |
| Q6 | `banker.c` (**Complete the Code**), `banker_bench.c`, `deadlock_demo.c` | `./banker`, `./banker_bench`, `./deadlock_demo` | Output matches expected. Safety check grows as O(m·n²): 1000 processes take 0.78 ms worst case. Real deadlock detected and recovered |

## Complete-the-Code outputs (all identical to the expected output)

```
$ ./rr
Gantt: P1(0-3) P2(3-6) P3(6-9) P4(9-12) P1(12-15) P2(15-16) P3(16-19) P4(19-21) P1(21-23) P3(23-26)
Process  TAT  WT
P1        23  15
P2        15  11
P3        24  15
P4        18  13
Average TAT = 20.00, average WT = 13.50

$ ./pipe_ipc
analyser: 6 readings, average 35.5, max 41
reader: done

$ ./bb
items consumed: 20, sum = 210 (expected 210)

$ ./banker
initial state safe: P1 P3 P4 P0 P2
P1 request: granted, safe sequence P1 P3 P4 P0 P2
P4 request: must wait (not available)
P0 request: denied (unsafe)
```

On Linux, `Q1/fork_states` also prints each child's `/proc/<pid>/status` (State, VmRSS, context-switch counts).
