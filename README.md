# java12-concurrency-jcstress-happens-before
# Presentation
* https://www.youtube.com/watch?v=pS5dPQwgnYo
* https://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601
# Build
```
$ mvn clean install
$ java -jar target/jcstress.jar -v -t AmISynchronized
```

# happens-before
* Compilers may generate instructions in a different order than the “obvious”
  one suggested by the source code
* store variables in registers instead of in
  memory
* processors may execute instructions in parallel or out of order
* caches
  may vary the order in which writes to variables are committed to main memory
* values stored in processor-local caches may not be visible to other processors
* Since most of the time threads within
  a concurrent application are each “doing their own thing”, excessive inter-thread
  coordination would only slow down the application to no real benefit
* In order to shield the Java developer from the differences between
  memory models across architectures, Java provides its own memory model,
  and the JVM deals with the differences between the JMMand the underlying platform’s
  memory model by inserting memory barriers at the appropriate places
* To guarantee that the thread executing action B can see the
  results of action A (whether or not A and B occur in different threads), there must
  be a happens-before relationship between A and B (partial ordering)
    * In the absence of a happens-before
      ordering between two operations, the JVM is free to reorder them as it pleases.
* A data race occurs when a variable is read by more than one thread, and written
  by at least one thread, but the reads and writes are not ordered by happens-before.
* A correctly synchronized program is one with no data races
* The rules for happens-before are:
  * Program order rule. Each action in a thread happens-before every action
  in that thread that comes later in the program order.
  * Monitor lock rule. An unlock on a monitor lock happens-before every
  subsequent lock on that same monitor lock.3
  * Volatile variable rule. A write to a volatile field happens-before every
  subsequent read of that same field.4
  * Thread start rule. A call to Thread.start on a thread happens-before
  every action in the started thread.
  * Thread termination rule. Any action in a thread happens-before any
  other thread detects that thread has terminated, either by successfully
  return from Thread.join or by Thread.isAlive returning
  false.
  * Interruption rule. A thread calling interrupt on another thread
  happens-before the interrupted thread detects the interrupt (either
  by having InterruptedException thrown, or invoking isInterrupted
  or interrupted).
  * Finalizer rule. The end of a constructor for an object happens-before
  the start of the finalizer for that object.
  * Transitivity. If A happens-before B, and B happens-before C, then A
  happens-before C.
* When two
  threads synchronize on different locks, we can’t say anything about the ordering
  of actions between them—there is no happens-before relation between the actions
  in the two threads
  
# volatile
A field may be declared **volatile**, in which case the Java Memory Model ensures that all threads see 
a consistent value for the variable. More precisely that means, that every read of a volatile variable 
will be read from the computer's main memory, and not from the CPU cache, and that every write to a 
volatile variable will be written to main memory, and not just to the CPU cache.

It prevents compiler / CPU code reordering.

**Happens-before** - In computer science, the happened-before relation is a relation between the result 
of two events, such that if one event should happen before another event, the result must reflect that, 
even if those events are in reality executed out of order (usually to optimize program flow).

When we write to a volatile variable, it creates a **happens-before** relationship with each subsequent 
read of that same variable. So any memory writes that have been done until that volatile variable write, 
will subsequently be visible to any statements that follow the read of that volatile variable.

Characterizing the situations under which volatile can be used safely involves determining whether 
each update operation can be performed as a single atomic update.

_Remark_:
* updates to non-volatile long or a double may not be atomic, and
* Java operators like ++ and += are not atomic.

1. Why an Object member variable can't be both final and volatile in Java?
    Essentially, when you declare object field as final you need to initialize it in object's 
    constructor and then final field won't change it's value. And JMM promises that after ctor is 
    finished any thread will see the same (correct) value of final field.
