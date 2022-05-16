# Interprocess Communication

Comments and corrections to [J. M. F. Tsang](j.m.f.tsang@cantab.net).

---

According to _The Pragmatic Programmer_ (Thomas & Hunt, 2020):
>*Concurrency* is when the execution of two or more pieces of code act as if they run at the same time. *Parallelism* is when they *do* run at the same time. [...] Concurrency is a requirement if you want your application to be able to deal with [situations where] where things are asynchronous. [...] Why is writing concurrent and parallel code so difficult? [...] One of the biggest culprits here is *shared state*. [...] 
>[One way to write better concurrent code] is using the *actor model*, where independent processes, which share no data, communicate over channels using defined, simple, semantics.

Python has plenty of support for writing concurrent code. 

- multiprocessing — Process-based parallelism
- threading — Thread-based concurrency
- asyncio — 'Javascript-style' `async` and `await` (concurrency)
- subprocess — Subprocess management
- Multiprocess synchronisation
    -   **_queue_** — A synchronized queue class
    -   **_socket_** — Low-level networking interface
    -   **_mmap_** — Memory-mapped file support

Python's multiprocessing and threading support have very similar interfaces to each other. However, they address different types of problems depending on the limiting factor. Multiprocessing is good for *CPU-bound* tasks, such as intensive calculations; while threading is preferred for *I/O bound* tasks where speed is limited by some external factor such as disk I/O or network communication, and other code can be run while waiting on this external factor.


## Amdahl's law
The degree to which parallelism will speed up a task is limited by the proportion $p$ of the task that can be parallelised. If this portion of the task is split between $n$ workers, then the speed of the entire task will be improved by a factor of $S$, given by
$$
S(n) = \frac{1}{1 - p + \frac{p}{n}},
$$
with an upper bound
$$ S(n) \leq \frac{1}{1 - p} = S(\infty). $$
Thus if $p \ll 1$ there cannot be much benefit from concurrency, regardless of the number of workers available. In such cases, another approach may be more suitable, depending on what the limiting factor is. (For example, if the limiting factor is I/O then threading may be useful.) On the other hand, as $p \rightarrow 1$ the speedup scales with the number of workers,
$$ S \approx n, $$and so additional resources can be effectively used.


## Threading


## Subprocess


## Multiprocess synchronisation

Allen B. Downey's _The Little Book of Semaphores_ ([link](https://greenteapress.com/wp/semaphores/)) is an excellent practical introduction to the principles of synchronisation.

### `queue`: a synchronised queue class

### `socket`: low-level networking interface

### `mmap`: memory-mapped file support


