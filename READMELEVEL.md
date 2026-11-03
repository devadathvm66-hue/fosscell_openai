# Level 3: Algorithm Optimization

This level examines two algorithms with very different performance characteristics: Fibonacci number generation and Bellman–Ford shortest paths. The goal is to replace avoidable repeated work, explain the resulting complexity, and manage memory and invalid inputs safely.

## Fibonacci

A straightforward recursive definition calls itself twice for most inputs:

```python
fib(n) = fib(n - 1) + fib(n - 2)
```

That implementation takes exponential time, approximately `O(φ^n)` (often loosely described as `O(2^n)`), because it recalculates the same Fibonacci values many times. Its recursion depth is `O(n)` and sufficiently large inputs can exceed Python's recursion limit.

Use an iterative two-value recurrence instead:

```python
def fibonacci(n: int) -> int:
    if isinstance(n, bool) or not isinstance(n, int):
        raise TypeError("n must be an integer")
    if n < 0:
        raise ValueError("n must be non-negative")

    previous, current = 0, 1
    for _ in range(n):
        previous, current = current, previous + current
    return previous
```

This performs `O(n)` additions and uses `O(1)` auxiliary variables, with no recursion. Python integers are arbitrary precision, so the bit cost of each addition grows with `n`; `O(n)` describes the number of recurrence steps.

**Source status:** `fibonacci.py` was not present in this `level3` directory when this README was written. The function above documents the recommended implementation; it is not a claim that the project already contains that file.

## Bellman–Ford

`bellman_ford.c` calculates shortest distances from a source vertex in a weighted directed graph. It supports negative edge weights and detects negative-weight cycles reachable from the source.

The current implementation stores edges in an edge list and performs up to `V - 1` passes over all `E` edges. Its worst-case time is **`O(VE)`**, not exponential. The final edge scan for a reachable negative cycle is `O(E)`. The edge list uses `O(E)` space; the distance array uses `O(V)` space.

A Fibonacci heap is generally not the right replacement here: Dijkstra's algorithm can use a heap for nonnegative edge weights, but it cannot correctly handle negative edges. Bellman–Ford's repeated relaxations are required to support the input domain.

### Recommended improvements

- Stop after a pass that makes no distance changes. This preserves the worst-case `O(VE)` bound while making already-converged inputs finish sooner; a no-change pass costs `O(E)`.
- Skip edges whose source is currently unreachable, as the current implementation already does.
- Use `long long` distances and guard addition against overflow; `int` distance arithmetic can overflow before comparison.
- Validate the graph, source index, and every edge endpoint before indexing arrays.
- Check allocations for failure and free every allocated object on all return paths.
- Allocate the distance array dynamically when graph size is not known at compile time, rather than using a variable-length stack array.

The edge list is a reasonable and compact representation for Bellman–Ford. An adjacency list can skip scanning outgoing edges of unreachable vertices, but still has `O(VE)` worst-case time and needs more bookkeeping. The appropriate choice depends on the graph and implementation goals.

## Build and run

From this directory, a C compiler supporting C99 or newer can compile the current source with warnings enabled:

```sh
gcc -std=c99 -Wall -Wextra -Wpedantic bellman_ford.c -o bellman_ford
./bellman_ford
```

The sample graph is expected to produce distances from vertex `0`:

```text
Vertex Distance from Source
0      0
1      -1
2      2
3      -2
4      1
```

Actual spacing may differ because of the source's tab formatting. A compiler is required only for the C program; Fibonacci uses Python 3 and its standard library.

## Files

- `bellman_ford.c` — Bellman–Ford C implementation and sample graph.
- `fibonacci.py` — expected by the task brief, but absent from this directory at the time this README was prepared.
