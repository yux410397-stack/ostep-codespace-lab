## Q1
- Prediction / 预测:总时间为 10 ticks，CPU 利用率为 100.00%。
- Reasoning / 理由:两个进程都只有 5 条 CPU 指令（无 I/O）。PID 0 先运行占用 CPU 5 个 tick，期间 PID 1 处于 READY 状态；PID 0 结束后，PID 1 占用 CPU 运行 5 个 tick，因此 CPU 全程无空闲。
- Verified result / 验证结果:总时间 10 ticks，CPU利用率100.00%，与预测完全一致。
- Analysis / 分析:两个进程全部是CPU计算指令，不存在I/O阻塞。PID0连续跑完5个tick，接着PID1连续跑5个tick。CPU全程都有进程在运行，没有空闲周期，CPU利用率满。

## Q2
- Prediction / 预测:总时间为 11 ticks，CPU 利用率为 81.82%。
- Reasoning / 理由:PID 0 在 tick 1 发起 I/O（耗时 1 tick）后进入 BLOCKED 状态 5 个 tick（tick 2-6）。期间 CPU 空闲，PID 1 抢占 CPU 运行其 4 条 CPU 指令（tick 2-5）。tick 6 时 PID 1 已结束，CPU 空闲 1 个 tick。tick 7 时 PID 0 的 I/O 完成，运行 1 个 tick 的 `io_done`，接着运行剩下的 4 条 CPU 指令（tick 8-11）。总时间 11 ticks，CPU 在 tick 6 空闲，工作了 9 ticks（9/11 ≈ 81.82%）。
- Verified result / 验证结果:总时间11 ticks，CPU利用率81.82%，和预测相符。
- Analysis / 分析:PID0执行RUN:io后进入BLOCKED状态等待I/O。此时CPU切换到PID1执行。PID1在tick6运行完毕，但PID0的I/O还没有完成，CPU出现1tick空闲。I/O与CPU计算只能部分重叠，因此CPU无法100%利用。

## Q3
- Prediction / 预测:总时间为 11 ticks，CPU 利用率为 81.82%
- Reasoning / 理由:改变进程顺序后，PID 0 先运行 4 条 CPU 指令（tick 1-4）。tick 5 时 PID 1 运行并发起 I/O（`RUN:io`）。接下来 tick 6-10，PID 1 处于 BLOCKED 状态，此时没有其他 READY 状态的进程，CPU 完全空闲（5 个 ticks）。tick 11 时 PID 1 处理 I/O 完成（`RUN:io_done`）并结束。CPU 总共运行了 4 + 1 + 1 = 6 个 ticks，利用率为 9/11 ≈ 81.82%（或 CPU 占用 9 ticks / 总 11 ticks）。
- Verified result / 验证结果:总时间11 ticks，CPU利用率81.82%，预测正确。
- Analysis / 分析:交换了进程启动顺序，PID1先运行并发起I/O阻塞。阻塞期间系统没有其他就绪进程，CPU只能空转等待I/O结束。虽然进程顺序调换，但CPU空闲tick数量不变，总耗时和利用率与Q2完全相同。

## Q4
- Prediction / 预测:总时间为 11 ticks，CPU 利用率为 81.82%。
- Reasoning / 理由:参数 `-S SWITCH_ON_END` 表示系统只有在当前进程完全结束时才切换进程。由于 PID 0 先运行且没有 I/O，PID 0 会一直运行直到结束，然后才轮到 PID 1。PID 1 发起 I/O 进入阻塞时，由于没有其他进程可运行，CPU 依然只能空闲等待，因此行为和 Q3 完全相同。
- Verified result / 验证结果:总时间11 ticks，CPU利用率81.82%，预测一致。
- Analysis / 分析:-S SWITCH_ON_END策略：只有进程全部执行结束，才会切换进程。PID0必须完整跑完，才轮到PID1；PID1阻塞I/O时没有就绪进程，CPU空闲。最终运行轨迹、总时间、利用率和Q3完全一样。

## Q5
- Prediction / 预测:总时间为 7 ticks，CPU 利用率为 100.00%。
- Reasoning / 理由:参数 `-S SWITCH_ON_IO` 表示当进程发起 I/O 阻塞时，系统会立即切换到另一个 READY 状态的进程。PID 0 在 tick 1 发起 I/O 后，系统立即切换到 PID 1 运行（tick 2-5）。当 PID 1 在 tick 5 结束时，PID 0 的 I/O 刚好在 tick 6 完成，PID 0 紧接着处理 `RUN:io_done` 和剩余指令，CPU 全程无重叠无空闲，效率最高。
- Verified result / 验证结果:总时间7 ticks，CPU利用率100.00%，预测正确。
- Analysis / 分析:-S SWITCH_ON_IO策略：进程发起I/O，立刻切换就绪进程。PID0等待I/O的同时，CPU并行运行PID1；当PID1执行结束，PID0的I/O刚好完成，CPU无缝继续执行PID0剩余指令。计算与I/O完全并行，CPU全程无空闲，效率最高。

## Q6
- Prediction / 预测:总时间为 11 ticks，CPU 利用率为 81.82%
- Reasoning / 理由:参数 `-I IO_RUN_LATER` 表示当 I/O 完成时，如果不立即抢占，而当前又没有其他进程在运行（PID 1 已在 tick 5 结束），系统会在下一个 tick（tick 7）调度该进程。PID 0 在 tick 1 发起 I/O，tick 6 完成 I/O，tick 7 恢复运行处理 `io_done` 并运行后续 4 条指令。
- Verified result / 验证结果:总时间11 ticks，CPU利用率81.82%，预测结果匹配。
- Analysis / 分析:-I IO_RUN_LATER：I/O完成不会立刻抢占CPU，需要等到当前正在运行的进程结束，才调度被I/O唤醒的进程。在本场景中PID0的I/O在tick6完成，但要等到下一个tick才可以运行，造成CPU空闲，无法做到硬件完全并行。

## Q7
- Prediction / 预测:总时间为 11 ticks，CPU 利用率为 81.82%。
- Reasoning / 理由:参数 `-I IO_RUN_IMMEDIATE` 表示一旦 I/O 完成，该进程将立即获得最高优先级恢复运行。在当前场景中，PID 1 在 tick 5 就已经运行完毕了，所以在 tick 6 时本就没有其他进程竞争 CPU，PID 0 在 I/O 完成后立刻在 tick 7 运行处理 `io_done`。
- Verified result / 验证结果:总时间11 ticks，CPU利用率81.82%，预测正确。
- Analysis / 分析:-I IO_RUN_IMMEDIATE，I/O一旦完成立刻抢占CPU。但在本例子中，PID0的I/O完成时PID1已经运行结束，CPU处于空闲状态，抢占机制不会改变运行结果。所以总时间、CPU利用率和Q6保持一致。

## Q8
- Prediction / 预测:随机种子 `-s 1`, `-s 2`, `-s 3` 会随机生成进程的 CPU 指令和 I/O 指令序列。不同参数设置对运行结果的影响如下：
1. 默认设置（`-S SWITCH_ON_IO -I IO_RUN_LATER`）：当进程发起 I/O 时会切换 CPU 给其他进程，但 I/O 完成后不会立刻抢占当前正在运行的进程，整体重叠利用率较高。
2. 立即运行设置（`-I IO_RUN_IMMEDIATE`）：I/O 完成后立刻抢占 CPU，能让 I/O 密集型进程更快发起下一次 I/O，从而提高整体 I/O 与 CPU 的并行度，减少总运行时间。
3. 顺序运行设置（`-S SWITCH_ON_END`）：不进行 I/O 切换，进程必须完全运行结束才换下一个，会导致 CPU 在 I/O 期间大量空闲，总运行时间最长，效率最低。
- Reasoning / 理由:不同的抢占与切换策略决定了 CPU 资源在等待 I/O 期间能否被充分利用。`-S SWITCH_ON_IO` 与 `-I IO_RUN_IMMEDIATE` 组合能最大程度发挥 CPU 和 I/O 设备并行工作的能力。
- Verified result / 验证结果:1. 默认配置 (-S SWITCH_ON_IO -I IO_RUN_LATER)：I/O阻塞时切换进程，I/O完成不抢占；CPU与I/O中等并行。2.  -S SWITCH_ON_IO -I IO_RUN_IMMEDIATE：I/O完成立刻抢占CPU，硬件并行度最高，总运行时间最短。3. -S SWITCH_ON_END：进程不中途切换，必须等进程跑完才切换；I/O等待期间CPU全程空闲，总耗时最长，CPU利用率最低。三个种子（-s 1、-s 2、-s3）会生成不同随机指令序列，但调度策略带来的性能趋势不变。
- Analysis / 分析:调度策略决定了CPU和I/O设备能否并行工作。• SWITCH_ON_IO：进程发起I/O就切换，把CPU交给其他就绪进程，避免CPU空等。• IO_RUN_IMMEDIATE：I/O完成立刻抢占CPU，适合I/O密集型任务，最大化I/O与CPU重叠。• SWITCH_ON_END 完全不做中途切换，I/O等待时CPU浪费，性能最差。

