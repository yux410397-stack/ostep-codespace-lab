## Q1
- Prediction / 预测:总时间为 10 ticks，CPU 利用率为 100.00%。
- Reasoning / 理由:两个进程都只有 5 条 CPU 指令（无 I/O）。PID 0 先运行占用 CPU 5 个 tick，期间 PID 1 处于 READY 状态；PID 0 结束后，PID 1 占用 CPU 运行 5 个 tick，因此 CPU 全程无空闲。
- Verified result / 验证结果:
- Analysis / 分析:

## Q2
- Prediction / 预测:总时间为 11 ticks，CPU 利用率为 81.82%。
- Reasoning / 理由:PID 0 在 tick 1 发起 I/O（耗时 1 tick）后进入 BLOCKED 状态 5 个 tick（tick 2-6）。期间 CPU 空闲，PID 1 抢占 CPU 运行其 4 条 CPU 指令（tick 2-5）。tick 6 时 PID 1 已结束，CPU 空闲 1 个 tick。tick 7 时 PID 0 的 I/O 完成，运行 1 个 tick 的 `io_done`，接着运行剩下的 4 条 CPU 指令（tick 8-11）。总时间 11 ticks，CPU 在 tick 6 空闲，工作了 9 ticks（9/11 ≈ 81.82%）。
- Verified result / 验证结果:
- Analysis / 分析:

## Q3
- Prediction / 预测:总时间为 11 ticks，CPU 利用率为 81.82%
- Reasoning / 理由:改变进程顺序后，PID 0 先运行 4 条 CPU 指令（tick 1-4）。tick 5 时 PID 1 运行并发起 I/O（`RUN:io`）。接下来 tick 6-10，PID 1 处于 BLOCKED 状态，此时没有其他 READY 状态的进程，CPU 完全空闲（5 个 ticks）。tick 11 时 PID 1 处理 I/O 完成（`RUN:io_done`）并结束。CPU 总共运行了 4 + 1 + 1 = 6 个 ticks，利用率为 9/11 ≈ 81.82%（或 CPU 占用 9 ticks / 总 11 ticks）。
- Verified result / 验证结果:
- Analysis / 分析:

## Q4
- Prediction / 预测:总时间为 11 ticks，CPU 利用率为 81.82%。
- Reasoning / 理由:参数 `-S SWITCH_ON_END` 表示系统只有在当前进程完全结束时才切换进程。由于 PID 0 先运行且没有 I/O，PID 0 会一直运行直到结束，然后才轮到 PID 1。PID 1 发起 I/O 进入阻塞时，由于没有其他进程可运行，CPU 依然只能空闲等待，因此行为和 Q3 完全相同。
- Verified result / 验证结果:
- Analysis / 分析:

## Q5
- Prediction / 预测:总时间为 7 ticks，CPU 利用率为 100.00%。
- Reasoning / 理由:参数 `-S SWITCH_ON_IO` 表示当进程发起 I/O 阻塞时，系统会立即切换到另一个 READY 状态的进程。PID 0 在 tick 1 发起 I/O 后，系统立即切换到 PID 1 运行（tick 2-5）。当 PID 1 在 tick 5 结束时，PID 0 的 I/O 刚好在 tick 6 完成，PID 0 紧接着处理 `RUN:io_done` 和剩余指令，CPU 全程无重叠无空闲，效率最高。
- Verified result / 验证结果:
- Analysis / 分析:

## Q6
- Prediction / 预测:总时间为 11 ticks，CPU 利用率为 81.82%
- Reasoning / 理由:参数 `-I IO_RUN_LATER` 表示当 I/O 完成时，如果不立即抢占，而当前又没有其他进程在运行（PID 1 已在 tick 5 结束），系统会在下一个 tick（tick 7）调度该进程。PID 0 在 tick 1 发起 I/O，tick 6 完成 I/O，tick 7 恢复运行处理 `io_done` 并运行后续 4 条指令。
- Verified result / 验证结果:
- Analysis / 分析:

## Q7
- Prediction / 预测:总时间为 11 ticks，CPU 利用率为 81.82%。
- Reasoning / 理由:参数 `-I IO_RUN_IMMEDIATE` 表示一旦 I/O 完成，该进程将立即获得最高优先级恢复运行。在当前场景中，PID 1 在 tick 5 就已经运行完毕了，所以在 tick 6 时本就没有其他进程竞争 CPU，PID 0 在 I/O 完成后立刻在 tick 7 运行处理 `io_done`。
- Verified result / 验证结果:
- Analysis / 分析:

## Q8
- Prediction / 预测:随机种子 `-s 1`, `-s 2`, `-s 3` 会随机生成进程的 CPU 指令和 I/O 指令序列。不同参数设置对运行结果的影响如下：
1. 默认设置（`-S SWITCH_ON_IO -I IO_RUN_LATER`）：当进程发起 I/O 时会切换 CPU 给其他进程，但 I/O 完成后不会立刻抢占当前正在运行的进程，整体重叠利用率较高。
2. 立即运行设置（`-I IO_RUN_IMMEDIATE`）：I/O 完成后立刻抢占 CPU，能让 I/O 密集型进程更快发起下一次 I/O，从而提高整体 I/O 与 CPU 的并行度，减少总运行时间。
3. 顺序运行设置（`-S SWITCH_ON_END`）：不进行 I/O 切换，进程必须完全运行结束才换下一个，会导致 CPU 在 I/O 期间大量空闲，总运行时间最长，效率最低。
- Reasoning / 理由:不同的抢占与切换策略决定了 CPU 资源在等待 I/O 期间能否被充分利用。`-S SWITCH_ON_IO` 与 `-I IO_RUN_IMMEDIATE` 组合能最大程度发挥 CPU 和 I/O 设备并行工作的能力。
- Verified result / 验证结果:
- Analysis / 分析:

