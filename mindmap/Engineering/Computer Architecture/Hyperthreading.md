Hyperthreading (Intel) or SMT (AMD) is a microarchitectural technique that allows multiple hardware threads to share a single physical CPU core.
A physical core exposes 2 logical CPUs, each capable of accepting and issuing instructions.

## Why Hyperthreading Exists

Modern CPU cores frequently stall due to:
- L1/L2/L3 cache misses
- branch mispredictions
- memory latency
- instruction dependencies

During these stalls, large portions of the core’s execution pipeline remain idle.

Hyperthreading increases pipeline utilization by allowing another hardware thread to use the otherwise idle resources.

This improves throughput, not peak single-thread performance.

Because critical resources are shared, two hyperthreads never perform like two real cores.

Typical performance gain: 20%–35% depending on workload.

## Hyperthreading + CPU Pinning
[[Non Uniform Memory Access (NUMA)]]
When pinning workloads to specific CPUs, you must avoid placing critical workloads on sibling logical threads of the same core.

Example:

If CPU 0 and CPU 8 are siblings, then:
`taskset -c 0,8`
pins two tasks to the same physical core → both compete for execution units → increased latency, reduced performance.

Better:
`taskset -c 0,1`

Two separate physical cores → no resource contention.
This matters heavily in:
- real-time systems
- high-frequency trading systems
- consistent-latency services
- benchmarking
- database workloads