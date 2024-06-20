---
title: Synchronizing data access in databases
type: page
description: Databases expose their data in a parallel way to multiple users. However, synchronization must occur so that in every moment of its lifespan, the database's state is consistent.
topic: Databases
---

## Latches vs. locks

Latches and locks are often interchangeably used to describe the synchronization scheme of a database. However, there are subtle differences in their usage. Locks are preferred when talking about users' interaction with the database through transactions. For example, locks as such are used in implementing waits-for-graph in deadlock detection, in transaction abort (full or partial) and in transaction timeout. On the other side, latches are used for ensuring consistency in parallel data structures which constitute the core of the database's storage engine. In most relational databases this is some variant of a B-Tree but choices are various, from deterministic data structures up to randomized ones like the skip list.

##  What are latches' preferred properties?

Although latches are primarily used to protect critical sections of the code to provide consistency of a data structure, the basic thing we would like from our latch is that it doesn't slow down the execution path in situations when threads don't create high contention in a system. This is very important because, in parallel systems, the number of execution paths that a system with multiple threads can take is very high. For example, I read in the book Clean Code by Uncle Bob that the following `getNextId` function can take 12870 execution paths when executed by only two threads (didn't get into details why).

```java
public class X {
    private int lastIdUsed;

    public int getNextId() {
        return ++lastIdUsed;
    }
};
```
The second thing we would like from latches is that it doesn't take a lot of memory because (for B-Tree) every node needs to have its latch. Now if `pthread_mutex_t` is used, which is of size 64B, this can be quite costly. (In c++ `std::mutex` is implemented using `pthread_mutex_t` ) And also as a side note, the latch is usually a part of the node struct (or class) to reduce the cost of ensuring cache coherence. 

### Spinlock

There are various implementations of a spinlock and many different optimizations and ideas that one can use to implement a spinlock but the core concept behind a spinlock is that it performs so-called "busy waiting". So if one thread acquired the right to use some object (node in a B-Tree example), the other thread spins in a loop testing whether the object is available. Such an approach is viable only when locks are held shortly by each thread since there is no OS involvement, otherwise, it is too costly to spend CPU time.

### Mutex

Mutex addresses the problem of a "busy loop" with the help of OS. The underlying operating system will deschedule the thread and avoid unnecessary CPU work time by waking the waiting thread once the resource becomes available. However, pure mutex has also a big performance cost since thread deschedulation and waking are expensive operations (around 25ns). However, if I am not wrong, most modern systems use some kind of a hybrid scheme which allows the thread to first spin for some time and then go to sleep if the resource is still not available therefore trying to make some kind of a compromise between two solutions.

### Adaptive Spinlock

Adaptive spinlock uses also a hybrid scheme but improves in a way that the thread doesn't get descheduled from OS but rather stored in a "parking lot". The trick is that such a "parking lot" is not managed by OS so it is much faster than a pure mutex. I know about Apple's WTF::ParkingLot but am not aware of anything else.

### Queue-based Spinlock

Queue-based Spinlock is used in the Linux kernel at multiple places. It improves the basic spinlock and the standard mutex in a way that when the thread cannot acquire the resource, it saves "the pointer" to the thread which currently holds the resource so that the resource is available, it can get notified very fast without big overhead.


In this blog post, I only scratched the surface locks and latches in the world of databases. Let me know about any comments, suggestions, corrections 🫡.

NOTE: The majority of materials are taken from Andy Pavlo's lectures from CMU.