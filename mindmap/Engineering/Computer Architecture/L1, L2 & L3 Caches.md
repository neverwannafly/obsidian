CPU caches exist because the CPU is extremely fast and RAM is extremely slow in comparison. To avoid waiting on RAM, the CPU stores frequently used data in small, fast memory units called caches. These caches come in a hierarchy: L1, L2, and L3.

## L1 Cache
This is the fastest and smallest cache, located closest to the CPU core. Each core has its own L1 cache. L1 is typically split into two parts: L1d (data) and L1i (instructions). Accessing L1 takes around 1 nanosecond. It stores the most immediately needed data. If data is here, the CPU runs at full speed. If not, it needs to check L2.

## L2 Cache
L2 is larger but slower than L1. It is also private to each core. Latency is around 3–4 nanoseconds. It stores data that is likely to be used soon but didn’t fit into L1. When L1 misses, the CPU checks L2. If L2 hits, it fills L1 and continues. If it misses, it moves on to L3.

## L3 Cache
L3 is the largest and slowest of the cache levels, but still far faster than RAM. It is shared across all cores. Latency ranges from 10–15 nanoseconds. L3’s purpose is to reduce RAM access and allow cores to share data efficiently. If something isn’t in L3 either, the CPU must go to RAM, which is roughly 60–120 nanoseconds away. A RAM access is about 100x slower than an L1 hit.

### Why L3 is shared?
If one core computes something, another core might need that data. Without a shared cache, the second core would fetch from RAM. With L3, it gets the data quickly, helping multi-core performance.

## Why Multiple levels exist?
Small memory is fast. Large memory is slow. The CPU needs both. L1 is tiny but extremely fast. L3 is huge but slower. L2 sits in between. The hierarchy gives the CPU a balance: very fast access for immediate data, and larger storage for data used slightly later.

```
Registers: 0.3 ns
L1 cache: ~1 ns
L2 cache: ~4 ns
L3 cache: ~12 ns
RAM: ~60–120 ns
SSD: ~50,000 ns
HDD: ~5,000,000 ns
```

## Cache lines
Caches move data in fixed chunks called cache lines (usually 64 bytes). If you access array\[i], the CPU also loads the nearby values array\[i+1], array\[i+2], etc. Loops over contiguous arrays are extremely fast because of this. Linked lists, however, often scatter nodes in memory and destroy cache locality, making them slower.

## Locality
- Spatial locality: accessing memory near recently accessed memory is faster because it’s probably in the same cache line.
- Temporal locality: accessing the same memory repeatedly is fast because it stays in the cache.

## Inclusive vs Exclusive Cache Architectures
### INCLUSIVE caches (Intel for many years)

Rule:
If a line exists in L1 or L2, it MUST also exist in L3.

So:
- Evict from L2 => it can remain in L3.
- Evict from L3 => it must also be evicted from L2 (and L1).

Hence, L3 cache would be a superset of L1+l2 Cache.
Pros:
- simplifies coherence between cores
- easier for multi-core systems

Downside:
- L3 gets filled with data just because it exists in L1/L2
- reduces effective usable size of L3

### Exclusive Caches (AMD Zen Series)
Rule:
Data is stored either in L2 or in L3, but never both.

So:
- Evict from L2 => it is moved to L3, not deleted.
- Evict from L3 => it is gone from the cache hierarchy.

Why AMD did this:
- improves effective cache size
- less duplication
- great for gaming and memory-heavy workloads

Downside:
- slightly more complex move logic
- misses may be more expensive in certain patterns