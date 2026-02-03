# The Four Fundamental OS Latency Sources

This section summarizes the four fundamental sources of latency introduced by an operating system. These sources are inherent to multitasking, shared hardware, and interrupt-driven execution, and they dominate tail latency behavior in real systems.

---

## 1. Scheduler Wait Time

**What it is**  
A task is runnable, but the scheduler does not run it immediately.

**Why it’s fundamental**  
There is exactly one CPU per core. If another task is running, your task must wait.

**Contribution**
- Adds direct delay before execution
- Grows with system load and priority inversion
- Primary source of tail latency

**Mental model**  
> “I’m ready, but someone else is using the CPU.”

---

## 2. Context Switches

**What it is**  
The OS stops one task and starts another.

**Why it’s fundamental**  
Multitasking requires saving and restoring execution state. This cost cannot be eliminated.

**Contribution**
- Consumes CPU cycles
- Pollutes CPU caches and TLBs
- Amplifies scheduler wait time and latency jitter

**Mental model**  
> “The CPU is busy switching people instead of doing work.”

---

## 3. CPU Migration

**What it is**  
A task resumes execution on a different CPU core.

**Why it’s fundamental**  
Load balancing and fairness require moving tasks across CPUs.

**Contribution**
- Cache cold-start penalties
- NUMA memory access penalties
- Non-deterministic latency spikes

**Mental model**  
> “I was moved to a different desk with none of my papers.”

---

## 4. Interrupt Handling Latency

**What it is**  
Hardware or kernel interrupts preempt task execution.

**Why it’s fundamental**  
Hardware events must be serviced; interrupts cannot be eliminated.

**Contribution**
- Preempts critical execution paths
- Adds unpredictable delay
- Introduces jitter even under low system load

**Mental model**  
> “I was interrupted mid-task and had to deal with something else.”

---
## In Simpler Terms

At the OS level, latency only comes from the kernel preventing your task from running.

That happens in four ways:

- You **wait** (scheduler wait time)
- You **get switched** (context switches)
- You **get moved** (CPU migration)
- You **get interrupted** (interrupt handling)

Everything else is a side effect of one of these.


## Summary

All four latency sources are unavoidable consequences of:
- Shared CPUs
- Preemptive multitasking
- Hardware interrupts
- Fairness and load balancing

Low-latency system design is primarily about **minimizing exposure** to these costs, not eliminating them.

# End-to-End Latency, Jitter, and Determinism

This section explains how **four OS-level parameters** combine to determine **end-to-end latency**, **jitter**, and **determinism** in a system. The focus is strictly OS-centric, avoiding hand-waving and separating application work from operating-system effects.

---

## OS-Level Parameters

The following four parameters are introduced by the operating system:

- **Scheduler wait time**
- **Context switch cost**
- **CPU migration penalty**
- **Interrupt preemption time**

---

## 1. End-to-End Latency (Single Execution)

For one request, period, or control cycle:

```text
End-to-End Latency =
    scheduler_wait_time
  + context_switch_cost
  + migration_penalty
  + interrupt_preemption_time
  + execution_time
```

### Component Definitions

- **Scheduler wait time**  
  Time spent waiting in the run queue before the task is scheduled.

- **Context switch cost**  
  CPU time consumed while saving and restoring task state.

- **CPU migration penalty**  
  Cache, TLB, and NUMA penalties incurred when a task resumes on a different CPU.

- **Interrupt preemption time**  
  Time during which the task is paused due to interrupt handling.

- **Execution time**  
  Actual application work performed by the task.

> **Note:** Only the first four components are introduced by the OS.  
> Execution time is application-defined but included for full end-to-end measurement.

---

## 2. Jitter (Variation Across Runs)

Jitter is **not** the sum of delays. Instead, it represents the **variation** of OS-introduced components across executions.

```text
Jitter ≈ variance(
    scheduler_wait_time,
    context_switch_cost,
    migration_penalty,
    interrupt_preemption_time
)
```

### Sources of Jitter

| Source | Reason for Variation |
|------|----------------------|
| Scheduler wait time | Run-queue depth, competing tasks |
| Context switch cost | Cache state, switch frequency |
| CPU migration | Unpredictable migration events |
| Interrupts | External hardware timing |

### Key Observations

- If all four components were constant → **zero jitter**
- In real systems:
  - Scheduler wait time and interrupts dominate jitter
  - CPU migration introduces long-tail latency spikes
  - Context switches amplify variability through cache effects

---

## 3. Determinism (Worst-Case Predictability)

Determinism concerns **bounding latency**, not minimizing average latency.

A system is deterministic if:

```text
max(
    scheduler_wait_time
  + context_switch_cost
  + migration_penalty
  + interrupt_preemption_time
) ≤ deadline
```

### Causes of Non-Determinism

- Unbounded scheduler wait time (e.g., CFS under load)
- Interrupts preempting tasks at arbitrary points
- CPU migration causing unbounded cache and memory penalties

### Improving Determinism

- Use a scheduler with bounded latency (real-time scheduler)
- Thread, isolate, or otherwise control interrupts
- Disable or constrain CPU migration
- Minimize context switches

---

This model cleanly separates **average latency**, **jitter**, and **worst-case guarantees**, making OS behavior explicit, measurable, and tunable.