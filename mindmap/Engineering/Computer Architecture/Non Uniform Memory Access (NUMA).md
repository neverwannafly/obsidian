NUMA is a computer memory design used in multi-CPU or multi-socket systems.
Instead of all CPUs sharing one big pool of memory with equal access speed, each CPU (or socket) has its own local memory. Access to local memory is fast. Access to another CPU’s memory is slower.

## Why NUMA exists?
As CPUs got more cores, one huge memory bus became a bottleneck.
NUMA reduces memory contention by giving each CPU socket its own dedicated memory.

Example:
Socket 0 has CPUs 0–7 and 64GB RAM.
Socket 1 has CPUs 8–15 and 64GB RAM.
CPU 2 accessing RAM from Socket 0: fast.
CPU 2 accessing RAM from Socket 1: slower.

This layout allows scalability to dozens or hundreds of cores.

## What is a NUMA Node?
A NUMA node is basically a group of CPUs + the memory attached to them.
If you have a 2-socket server, you usually have 2 NUMA nodes.

NUMA node = socket = local memory region (conceptually)

Some CPUs also subdivide nodes further internally.

## NUMA Awareness
If the OS or application is NUMA-aware, it will try to:
• Schedule a thread on a CPU near its memory
• Allocate memory from the closest node
• Keep work local to reduce remote traffic

Bad NUMA behavior destroys performance.

The OS knows about NUMA topology.
When you allocate memory, it usually comes from the node where the thread currently runs. If the thread later moves to a different node, memory becomes “remote.”

Some OS features:
- “first touch” policy → memory comes from the node whose CPU first writes to that page.
- NUMA balancing → OS may automatically migrate memory pages between nodes to maintain locality.

## NUMA in Virtualization
Hypervisors try to map vCPUs to the same physical NUMA node.
If a VM spans multiple physical NUMA nodes, performance drops (remote memory accesses).
This is why large VMs sometimes need NUMA pinning or tuning.

## CPU Pinning
It means forcing a process or thread to run only on specific CPU cores.

It prevents the OS from moving the process across the CPU cores — which would otherwise cause cache misses, NUMA issues, and inconsistent performance.
```cpp
#define _GNU_SOURCE
#include <pthread.h>
#include <sched.h>
#include <stdio.h>

int main() {
    cpu_set_t mask;
    CPU_ZERO(&mask);
    CPU_SET(3, &mask);   // pin to CPU 3

    if (sched_setaffinity(0, sizeof(mask), &mask) != 0) {
        perror("sched_setaffinity");
        return 1;
    }

    while (1) {
        // do work
    }
}

```

CPU pinning is also widely supported by docker and kubernetes. 
- `docker run --cpuset-cpus="0-3" myimage` forces the container to use cpus 0 to 3
- Kubernetes has a `single-numa-node` topology constraint, which enforces NUMA alignment