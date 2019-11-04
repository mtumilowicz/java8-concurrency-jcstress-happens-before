# A practical approach to Java Memory Model 
# Presentation
 * https://www.youtube.com/watch?v=pS5dPQwgnYo
# Build
```
$ mvn clean install
$ java -jar target/jcstress.jar -v -t AmISynchronized
```

 To guarantee that the thread executing action B can see the
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
