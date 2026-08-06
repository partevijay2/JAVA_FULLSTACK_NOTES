# Java Locking & Concurrency Patterns

## Overview
This note summarizes common locking and synchronization mechanisms in Java, when to use them, trade-offs, and short examples. It is intended as a practical reference for developers converting imperative multi-threaded code to safer, higher-performance constructs.

## Taxonomy
- Intrinsic locks (synchronized)
- Explicit locks (ReentrantLock, ReentrantReadWriteLock, StampedLock)
- Optimistic / CAS (Atomic*, VarHandle)
- Higher-level coordination (Semaphore, CountDownLatch, CyclicBarrier, Phaser)
- Volatile (visibility, ordering)
- Lockless utilities and concurrent collections (ConcurrentHashMap, LongAdder)

## Intrinsic locks (synchronized)
- Usage: synchronized(method) or synchronized(this) { ... }
- Reentrant, simple to use, no extra APIs (but no tryLock, no timeout, not interruptible)

Example:
```java
public class Counter {
    private int count = 0;

    public synchronized void increment() {
        count++;
    }

    public synchronized int get() {
        return count;
    }
}
```

## ReentrantLock
- Richer API: lockInterruptibly(), tryLock(timeout), newCondition(), fairness.
- Must always unlock in finally.

Example:
```java
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantCounter {
    private final ReentrantLock lock = new ReentrantLock();
    private int count = 0;

    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock();
        }
    }
}
```

When to use ReentrantLock: need tryLock, timeouts, interruptible lock acquisition, multiple Condition objects, or explicit fairness.

## ReadWrite locks (ReentrantReadWriteLock)
- Multiple readers allowed concurrently; writers exclusive.
- Good for read-heavy workloads; be careful of writer starvation.

Example:
```java
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadMostlyCache {
    private final ReentrantReadWriteLock rw = new ReentrantReadWriteLock();
    private volatile String data = "";

    public String read() {
        rw.readLock().lock();
        try {
            return data;
        } finally {
            rw.readLock().unlock();
        }
    }

    public void write(String s) {
        rw.writeLock().lock();
        try {
            data = s;
        } finally {
            rw.writeLock().unlock();
        }
    }
}
```

## StampedLock (Java 8)
- Supports optimistic reads for low-latency read-mostly access.
- Not reentrant; must validate optimistic reads.

Example (optimistic read):
```java
import java.util.concurrent.locks.StampedLock;

public class Point {
    private int x, y;
    private final StampedLock sl = new StampedLock();

    public int[] read() {
        long stamp = sl.tryOptimisticRead();
        int rx = x, ry = y;
        if (!sl.validate(stamp)) {
            stamp = sl.readLock();
            try {
                rx = x; ry = y;
            } finally {
                sl.unlockRead(stamp);
            }
        }
        return new int[] { rx, ry };
    }

    public void move(int nx, int ny) {
        long stamp = sl.writeLock();
        try {
            x = nx; y = ny;
        } finally {
            sl.unlockWrite(stamp);
        }
    }
}
```

## Atomic and CAS (non-blocking)
- AtomicInteger, AtomicReference, AtomicLong, LongAdder, VarHandle.
- Use for counters and single-variable updates. For compound invariants, prefer locks.

Example:
```java
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicCounter {
    private final AtomicInteger ai = new AtomicInteger();
    public void increment() { ai.getAndIncrement(); }
    public int get() { return ai.get(); }
}
```

## Volatile
- Ensures visibility and ordering for reads/writes of a variable.
- Does NOT provide atomicity for compound actions.

Use volatile for simple flags or immutable state references.

## Higher-level primitives
- Semaphore: limit concurrent access to N permits.
- CountDownLatch: one-shot waiting for N events.
- CyclicBarrier/Phaser: coordinate phases of threads.

## Best Practices
- Prefer higher-level concurrent collections (ConcurrentHashMap, ConcurrentLinkedQueue) over manual locking when possible.
- Keep critical sections small. Avoid holding locks during I/O or blocking operations.
- Document lock ordering if multiple locks are acquired; acquire in a consistent global order to avoid deadlocks.
- Use tryLock with timeouts for complex multi-lock interactions to avoid waiting forever.
- Use LongAdder for high-contention counters.

## Debugging and Diagnostics
- Thread dumps (jstack) show monitor ownership and blocked threads.
- Use ThreadMXBean to detect deadlocks programmatically.
- Profilers (Flight Recorder, VisualVM) can show contention hotspots.

## Exercises
1. Convert a synchronized counter to use ReentrantLock with tryLock and a timeout; compare behavior under contention.
2. Implement a read-heavy cache with ReentrantReadWriteLock and benchmark read vs write throughput.
3. Implement optimistic reads with StampedLock and simulate write frequency to observe validate() fallbacks.

---

Created by GitHub Copilot assistant — Java locking and concurrency patterns note.
