# h1 Common Concurrency Java Patterns

Notes:
1. Runnable instance into Thread class constructor, difference between methods of thread instance:
- **thread.start** - create new thread;
- **thread.run** - not create new thread;
2. Static synchronized method get lock of hole class; other words: A synchronized static method uses the class as a synchronization object;
3. Synchronized method get lock of instance; other words: a synchronized non-static method uses the instance as a synchronized object;
4. Good practice is to hide an object used for synchronization;
5. If we've 2 synchronized methods: A, B in instance C, and thread T1 got lock on C.A, thread T2 trying to get same lock when invoke C.B and get state BLOCKED;
6. Pattern Producer/Consumer uses common object within synchronized block, which is waiting if full/empty queue and notify if push or pull;
7. To stop a thread needs to use interrupt method, it'll move thraed's state into TERMINATED;
8. Thread state list: NEW - RUNNABLE - INTERRUPTED, while it runs can be BLOACKED, WAIT, WAIT_TIMEOUT;


![CPU-Architecture](https://user-images.githubusercontent.com/5359109/79950934-80372780-8480-11ea-9aab-5e050534a2ae.png)


9. Synchronized and volatile give JVM visibility of variable between L1 cache of different cores;
10. Happens before link exists between all synchronized or volatile write operations and all synchronized or volatile read operations that follow;
11. False Caching. When a visible variable is modified in an L1 cache, all the line is marked "dirty" for the other caches;
12. It's a bad practic to use Double Check locking. Example:
![double-check-locking](https://user-images.githubusercontent.com/5359109/79951738-ec665b00-8481-11ea-8bba-8d1c508c1686.png)


Problem is that we've not "happens before link" between read and write operations, only write is synchronized;


13. ReentrantLock (implements Lock interface) can be use instead of "Object synchronized pattern" (see 4);
14. ReentrantLock offers the same guarantees as synchronized (execution, read and write ordering), and more functionalities;
15. ReentrantLock has methods: lock, lockInterruptible, tryLock;
16. Constructor of ReentrantLock(boolean fair); TRUE - lock is fair, costly (by sequence of threads); FALSE - non-fair(BY DEFAULT);
![reentrantlock](https://user-images.githubusercontent.com/5359109/79952170-ad84d500-8482-11ea-957f-4b03745b7e79.png)


17. ReentrantLock works in pair with an class Condition. To get it we call lock.newCondition(). We use it like: condition.await() and condition.signal();
18. For Producer/Consumer pattern:
![Condition-ProducerConsumer-Pattern](https://user-images.githubusercontent.com/5359109/79952235-c8574980-8482-11ea-89c8-9093a98ed3d2.png)


19. Reentrant could has as many conditions as it needs;
20. ReadWriteLock (extends Lock interface) interface with 2 methods: readLock(), writeLock(); It gives opportunity to create locking write operation and allow parallel reads;
21. RWLock has rules: 
- only 1 thread can hold WRITE lock; 
- when the write lock is held, NO ONE can held READ lock; 
- many thread can hold READ lock;

22. ReentrantReadWriteLock is an implementation;
23. Semaphore looks like a lock, but allows several threads (called permits, set in constructor) in the same block of code;
24. Semaphore can be fair or not (by default);
25. Semaphore has a number of permits, which can be acquired in different ways, and released;
26. CyclicBarrier is a tool to synchronize a several threads between them, and let them continue when they reach a common point;
27. A CyclicBarrier closes again once opened, allowing for cyclic computations, can also be reset;
28. The threads in CyclicBarrier can wait on each other on time outs;
29. CountDownLatch a tool to check that different threads did their tasks and synchronize the beginning of subsequent tasks on the last one completed;
30. Once open CountDownLatch cannot be closed again; (instead of CyclicBarrirer)
31. CASing -> Compare and Swap -> if the current value at this address is the expected value, then it's replaced by the new value and it returns true;
32. Atomic* classes help with CASing
33. CASing works fine when concurrency is not to high;
34. Atomic classes try again and again inside their methods if increment operation return false while it's not return true; 
35. LongAdder -  Adder classes increment without returning new/old value;
36. Copy on Write (CopyOnWriteArrayList, CopyOnWriteArraySet):
- exist for list and set;
- no locking for read operations;
- write operations create new structure;
- the new structure replaces the previous one;
37. Queue and Deque:
- ArrayBlockingQueue a bounded blocking queue build on array;
- ConccurentLinkedQueue an unbounded blocking queue;
38. ConcurrentHashMap:
- fully concurrent map;
- tailored to handle millions of key/value pairs;
- with build-in parallel operations;
- can be used to create concurrent set -> new ConcyrrentHashMap.<String>newKeySet();
39. ConcurrentSkipList (implementations: ConcurrentSkipListMap, ConcurrentSkipListSet):
- a skip list is used to implement a map;
- the keys are sorted;
- the skip list structure ensure fast access to any key;
- a skip list is not an array-based structure;
- and there are other ways than locking to guard it;
![concurrent-skiplist](https://user-images.githubusercontent.com/5359109/79952303-e0c76400-8482-11ea-8db5-9f744e08bf7c.png)
  
