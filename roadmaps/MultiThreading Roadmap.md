# Multithreading Mastery Roadmap — Basics-First Edition

**For**: Java/Spring Boot developer (3+ yrs), learning multithreading from the ground up
**Approach**: Every topic builds on the previous one. Nothing is assumed — each concept is explained plainly before we write code for it.

---

## STAGE 1: What Even Is a Thread? (Day 1-2)

Build the mental model before touching code.

1. **Process vs Thread** — what a process is, what a thread is, why threads share heap memory but have their own stack
2. **The Main Thread** — every Java program already runs one thread even if you never create another
3. **Why threads exist** — parallelism (multi-core work) vs concurrency (responsiveness while waiting)
4. **Where this shows up in your job** — how Tomcat assigns a thread per incoming HTTP request

*No code yet. Just make sure this mental model is solid.*

---

## STAGE 2: Creating Your First Thread (Day 3-4)

1. **The `Thread` class** — what it represents in code
2. **`Runnable`** — what "a task" means, and why we separate "the task" from "the thread that runs it"
3. **`start()` vs `run()`** — the #1 beginner trap, and exactly why calling `run()` doesn't create a new thread
4. **Watching it happen** — print `Thread.currentThread().getName()` from multiple threads to *see* that they're genuinely different threads
5. **`Callable`** — like Runnable, but can return a value and throw checked exceptions (we'll use this properly once we reach Executors)

*Practice: create 3 threads that each print their name and a number in a loop. Run it multiple times — notice the output order is different every time. That's your first hands-on look at non-determinism.*

---

## STAGE 3: Thread Lifecycle (Day 5)

Now that you've created threads, understand the states a thread passes through:

1. **NEW** — created, not started
2. **RUNNABLE** — started, executing or eligible to execute
3. **BLOCKED** — waiting to grab a lock someone else is holding
4. **WAITING / TIMED_WAITING** — paused deliberately (`join()`, `sleep()`)
5. **TERMINATED** — done

*Practice: use `Thread.getState()` at different points and print it, so you literally watch a thread move through these states.*

---

## STAGE 4: The Real Problem — Shared Data (Day 6-8)

This is where multithreading gets interesting (and dangerous).

1. **What is a race condition?** — plain explanation with a real example: two threads incrementing the same counter, final result is wrong
2. **Why it happens** — `count++` is actually 3 steps (read, add 1, write back), and threads can interleave those steps
3. **What is a visibility problem?** — different from a race condition: one thread's write may never be *seen* by another thread due to CPU caching
4. **`volatile`** — what it fixes (visibility + ordering) and what it does NOT fix (atomicity of compound operations like `count++`)

*Practice: reproduce a race condition with a shared counter (no fix). Then reproduce a visibility bug with a flag (no fix). See both problems with your own eyes before learning the fix.*

---

## STAGE 5: Fixing It — Synchronization Basics (Day 9-11)

1. **`synchronized` keyword** — what a "lock" or "monitor" actually is, in plain terms
2. **`synchronized` method vs `synchronized` block** — when to use which
3. **Fixing the race condition** — take your buggy counter from Stage 4 and fix it with `synchronized`
4. **`wait()` / `notify()` / `notifyAll()`** — how threads can pause and signal each other, not just block
5. **Classic producer-consumer problem** — build it using `synchronized` + `wait/notify`

*Practice: the fixed counter, then a working producer-consumer queue built entirely from scratch (no library classes yet).*

---

## STAGE 6: Modern Locking Tools (Day 12-13)

Now that you understand *why* locks are needed and how the primitive version works:

1. **`ReentrantLock`** — same idea as `synchronized` but more control (tryLock, timeouts, fairness)
2. **`ReadWriteLock`** — separate locks for reading vs writing (many readers, one writer)
3. **When to use which** — practical guidance, not just API list

*Practice: rewrite your producer-consumer from Stage 5 using `ReentrantLock` + `Condition` instead of raw `wait/notify`. Compare the two.*

---

## STAGE 7: Stop Building Everything By Hand — Executors (Day 14-16)

You've now built things manually. Time to use the tools built for this.

1. **Why raw threads don't scale** — creating a new `Thread` per task is expensive; you need a pool
2. **`ExecutorService`** — what a thread pool actually is
3. **Types of pools** — fixed, cached, scheduled — what each is for
4. **Submitting tasks** — `submit()`, `execute()`, getting a `Future` back
5. **Shutting down properly** — `shutdown()` vs `shutdownNow()` (a very common production bug: forgetting to shut down executors)

*Practice: rewrite your producer-consumer using `ExecutorService` instead of manually created threads.*

---

## STAGE 8: Getting Results Back — Future & CompletableFuture (Day 17-19)

Directly relevant to your Spring Boot/microservices work.

1. **`Future`** — what it represents, why calling `.get()` blocks
2. **The problem with `Future`** — can't chain, can't combine easily, blocking `.get()` defeats the purpose
3. **`CompletableFuture`** — chaining (`thenApply`, `thenCompose`), combining (`thenCombine`), handling errors (`exceptionally`)
4. **Fan-out/fan-in pattern** — calling multiple services in parallel and combining results (exactly what you'd do calling multiple downstream APIs)

*Practice: build a mock "aggregator" that calls 3 fake downstream services in parallel and combines their results into one response.*

---

## STAGE 9: Concurrent Collections & Coordination Tools (Day 20-22)

1. **`ConcurrentHashMap`** — why a regular `HashMap` breaks under concurrent access, what this fixes
2. **`CopyOnWriteArrayList`** — when this makes sense (read-heavy, rarely-written lists)
3. **`BlockingQueue`** — the "proper" tool for producer-consumer (replaces what you hand-built in Stage 5/6)
4. **`CountDownLatch`, `CyclicBarrier`, `Semaphore`** — what each coordinates, with a simple real-world analogy for each
5. **`Atomic*` classes** — lock-free counters, when they're enough and when they're not

*Practice: rebuild producer-consumer a third time — this time using `BlockingQueue` — and compare against your Stage 5/6 versions. Build a simple rate limiter using `Semaphore`.*

---

## STAGE 10: Things That Go Wrong (Day 23-24)

1. **Deadlock** — what it is, a minimal 2-thread/2-lock example that hangs forever
2. **How to detect it** — thread dumps, `jstack`
3. **Livelock and starvation** — how these differ from deadlock
4. **Thread pool misconfiguration** — unbounded queues causing memory issues, undersized pools causing timeouts

*Practice: write code that deliberately deadlocks. Take a thread dump. Identify the deadlock from the dump. Then fix it via lock ordering.*

---

## STAGE 11: Where This Meets Spring Boot (Day 25-27)

Now everything connects to your actual job:

1. **`@Async`** — what it does under the hood (it's just submitting to an executor), configuring a custom `TaskExecutor`
2. **`@Scheduled`** — threading implications
3. **`@Transactional` and threads** — why a transaction does NOT propagate across an `@Async` call (classic gotcha)
4. **Optimistic vs pessimistic locking** — `@Version` in JPA vs `@Lock(PESSIMISTIC_WRITE)` — critical for banking/financial data like your ERM Portal
5. **Tomcat + HikariCP thread pool tuning** — how these settings relate to everything you just learned

*Practice: build a small Spring Boot endpoint using `@Async` + `CompletableFuture` to call 3 services in parallel. Add optimistic locking to a JPA entity and simulate a concurrent update conflict.*

---

## STAGE 12: Capstone Projects (Day 28-30)

Put it all together:

1. **Bank transfer simulator** — buggy version first (race condition), then fixed with proper locking, benchmarked against an `Atomic` version
2. **Rate limiter service** — production-style, using `Semaphore` + `ScheduledExecutorService`
3. **Load test + tune** — take a real (or mock) ERM-style endpoint, load test with JMeter/Gatling, tune thread pools based on actual observed behavior

---

## How We'll Work Through This

At each stage: I'll explain the concept in plain terms first (like we just did with "what is a thread"), then we write and run code together, then you do the practice exercise before moving to the next stage. No skipping ahead into jargon.

## Self-Check Milestones
- [ ] Can explain thread vs process to someone else without jargon
- [ ] Can explain the difference between a race condition and a visibility problem
- [ ] Has personally reproduced both a race condition and a visibility bug in code
- [ ] Can build producer-consumer 3 different ways (raw wait/notify, ReentrantLock, BlockingQueue) and explain trade-offs
- [ ] Can read a thread dump and spot a deadlock
- [ ] Can explain why `@Transactional` breaks across `@Async` boundaries
- [ ] Has built and load-tested a Spring Boot endpoint using async parallel calls
