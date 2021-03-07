# Condensed notes for midterm

## Single processor machines

### Compiler optimization

compiler performs many optimizations

- improve register reuse/optimize away unneeded variables
- interchange loops
- unroll loops

### Memory hierarchy

![memory hierarchy.png](memory hierarchy.png)

### Cache and locality

- cold vs warm cache:
  - cold cache: data in the cache is stale, need to read from memory
  - warm cache: relevant data in cache
  - running code multiple times can help performance because it warms up cache
- **false sharing**
  - if `[x, y, z]` is an array, processor 1 accesses `x`, and processor 2 accesses `y`, then both have entire array in cache line
  - if processor 1 updates `x`, then processor 2 needs to fetch the cache line again even though it doesn't need `x`

## Parallel algorithm design

### Task decomposition

- divide the program into *tasks* that can be executed in parallel
- a **task** is a unit of computation in the program that can be assigned to a process and run concurrently with other tasks
  - i.e. independent chunks of work
- **granularity** is determined by how many tasks there are and how small these tasks are
- parallelism granularity is how much processing is performed before communication is necessary between processes
- the maximum/average **degree of concurrency** is the maximum/average number or weight of tasks that can be executed concurrently
  - the maximum degree is the maximum sum of nodes at a single level in a task dependency graph
- the **critical path** of a program is the longest path between any pair of start and finish nodes in a program's task dependency graph
  - the length of this path is the sum of the weights of the nodes in the path
  - the average degree of concurrency is the total amount of work divided by the length of the critical path

### Task mapping

- **task mapping** is assigning tasks to processes
- a good mapping
  - maximizes use of concurrency
    - minimize **idle time**
  - minimizes program completion time
  - minimizes interaction between processes
- **recursive decomposition** divides program into tasks by turning each step of program into divide and conquer problem, maps divided tasks to different processes
- **data decomposition** divides data into separate chunks, and this can induce a task mapping
  - could do this in different ways
    - partition input data
    - partition output data
    - partition both

### Static mapping

**static mapping** is when tasks are assigned to processes before execution

- e.g. by programmer
- must have good knowledge of program, data, interactions, programming paradigm for effective static mapping
- usually done with uniform partitioning of data
- NP-complete in general (so need good heuristics)

static mapping schemes:

1. data partitioning
   - arrays are generally regular and dense, so can create tasks by splitting array into chunks
     - map tasks to processes essentially by mapping data to them
   - graphs are generally sparse and irregular so more difficult
2. task partitioning
   - partition task dependency graph
   - map nodes of dependency graph to processes
   - optimal mapping is NP-complete
3. hierarchical mapping
   - if the task dependency graph is a tree, then there is a clear hierarchy but bad concurrency near/at the root
   - can solve this by dividing larger tasks into subtasks which are mapped to different processes, so each height of tree still has good concurrency

![static hierarchical mapping.png](static hierarchical mapping.png)

### Dynamic mapping

**dynamic mapping** is when tasks are assigned to processes during execution

- a common strategy is a **task pool**, a queue of tasks
  - processor takes a new task from queue whenever it is idle

### Decreasing interaction overhead

- maximizing data locality
  - minimize volume of shared data
  - use local data to store intermediate results, rather continuously updating global data
  - minimize interactions by using large chunks of shared data
- minimize contention and hot spots
  - **contention** is concurrent accesses to the same memory block
    - might require synchronization
    - might cause false sharing
  - can solve by reordering accesses
  - can also solve by decentralizing shared data
    ![minimize contention decentralization.png](minimize contention decentralization.png)
- overlapping computations with interactions
  - interactions require waiting, but processes don't have to idle while waiting
  - can do computations while waiting
  - one strategy is to initiate interaction earlier than necessary and continue computation while waiting
  - another strategy is to grab more tasks in advance, switch between computations while waiting
  - may be supported in software or hardware
  - more applicable to distributed/GPU architectures than shared memory models
- replicating data/computations
  - replicate shared data so that there is no interaction, so no contention
  - works well when shared data is read-only
  - increases overall memory usage due to replication
  - if shared data is not read-only, then upholding coherence might be difficult and negate benefits

### Parallel algorithm models

A model consists of a decomposition method, a task mapping method, and strategies to minimize interactions

- **data parallel model**
  - static uniform data partitioning
  - static mapping
    - make use of data parallelism - perform same operations on multiple data items
  - can optimize by
    - decomposing data in a way that preserves locality
    - overlapping computation with interaction
  - this model scales well when we add more processors
- **task graph model**
  - use knowledge of task dependencies from dependency graph to increase locality/reduce interaction
  - recursive or graph-based data partitioning
  - static or dynamic mapping, but try to keep tasks independent
  - can optimize by
    - maximizing locality by minimizing non-local accesses/interactions
    - overlapping computation with interaction
  - works much better in shared address space than distributed
  - e.g. merge sort
- **work pool model**
  - tasks are taken by processes from common queue
  - decomposition depends on problem
  - mapping is dynamic, by virtue of queue
  - main strategy for reducing interaction is adjusting granularity of tasks in work pool
  - this has tradeoff:
    - larger and fewer tasks means less interaction
    - smaller and more tasks means less idle time
- **manager-worker model**
  - used in distributed architectures
  - manager process generates work and allocates to workers
  - decomposition depends on problem
  - mapping generally dynamic
  - can optimize by:
    - choosing granularity carefully
      - too high granularity makes manager a bottleneck
      - manager generating tasks overlaps with workers doing computations

### Measuring performance

- common sources:
  - communication
  - idling
  - excess computation (sometimes added to reduce communication)
- speedup $S = T_s / T_p$
  - $T_s$ is time that serial program takes
  - $T_p$ is time that $p$ processors take
- linear speedup is when $S \sim p$
- superlinear speedup only happens when sequential algorithm is at a disadvantage