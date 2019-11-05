# java12-concurrency-jcstress-happens-before

## reference
* https://www.youtube.com/watch?v=pS5dPQwgnYo
* https://github.com/amczarny/JMMPresentation
* https://www.amazon.com/Java-Concurrency-Practice-Brian-Goetz/dp/0321349601

## project
```
$ mvn clean install
$ java -jar target/jcstress.jar -v -t HappensBeforeExample
```
* reports in readable form: `results/jcstress.HappensBeforeExample`

## jcstress
* The Java Concurrency Stress tests (jcstress) is an experimental harness and a suite of tests to aid the research 
in the correctness of concurrency support in the JVM, class libraries, and hardware
* most of the tests are probabilistic, and require substantial time to catch all the cases
    * it is highly recommended to run tests longer to get reliable results
* most of the tests require at least 2 online CPUs
    * low CPU count machines could also use these tests, but harness will force yielding there
* samples: http://hg.openjdk.java.net/code-tools/jcstress/file/tip/jcstress-samples/src/main/java/org/openjdk/jcstress/samples
    * **APISample_** describe the API, 
    * **JMMSample_** describe the basics of Java Memory Model, 
    * **ConcurrencySample_** show the interesting concurrent behaviors of standard library
* API: http://hg.openjdk.java.net/code-tools/jcstress/file/tip/jcstress-core/src/main/java/org/openjdk/jcstress/annotations/

### overview
* `@JCStressTest`
    * marks the class that should be as the concurrency stress test
    * @Actor annotations are used to describe test behavior
    * @State and @Result annotations are used to describe the test state and results
* `@Outcome`
    *  * {@link Outcome} describes the test outcome, and how to deal with it.
       * It is usually the case that a {@link JCStressTest} has multiple outcomes,
       * each with its distinct {@link #id()}.
    *  * <p>{@link #id()} is cross-matched with {@link Result}-class' {@link #toString()}
       * value. {@link #id()} allows regular expressions.
    *  * <p>There can be a default outcome, which captures any non-captured result.
       * It is the one with the default {@link #id()}
    * fields
         * @return Observed result. Empty string or no parameter if the case is default.
         * Supports regular expressions.
         */
        String[] id() default { "" };
    
        /**
         * @return Expectation for the observed result.
         * @see Expect
         */
        Expect expect();
                /**
                 * Acceptable result. Acceptable results are not required to be present.
                 */
                ACCEPTABLE,
            
                /**
                 * Same as {@link #ACCEPTABLE}, but this result will be highlighted in reports.
                 */
                ACCEPTABLE_INTERESTING,
            
                /**
                 * Forbidden result. Should never be present.
                 */
                FORBIDDEN,
    
        /**
         * @return Human-readable description for a given result.
         */
        String desc() default "";
* `@State`
     * {@link State} is the central annotation for handling test state.
     * It annotates the class that holds the data mutated/read by the tests.
     
      * <p>Important properties for the class are:
      * <ol>
      *     <li>State class should be public, non-inner class.</li>
      *     <li>State class should have a default constructor.</li>
      * </ol>
      
       * <p>During the run, many {@link State} instances are created, and therefore
       * the tests should try to minimize state instance footprint.
* `@Actor`
     * {@link Actor} is the central test annotation. It marks the methods that hold the
     * actions done by the threads. The invariants that are maintained by the infrastructure
     * are as follows:
     *
     * <ol>
     *     <li>Each method is called only by one particular thread.</li>
     *     <li>Each method is called exactly once per {@link State} instance.</li>
     * </ol>
     *
     
     * <p>Note that the invocation order against other {@link Actor} methods is deliberately
      * not specified.
      
       * <p>Actor-annotated methods can have only the {@link State} or {@link Result}-annotated
       * classes as the parameters.
* `@Result`
    *  * {@link Result} annotation marks the result object. This annotation is seldom
       * useful for user code, because jcstress ships lots of pre-canned result classes,
       * see {@link org.openjdk.jcstress.infra.results} package.
    * one of many implementations: `I_Result` (one int holder)

## happens-before
* in computer science, the happened-before relation is a relation between the result of two events, 
such that if one event should happen before another event, the result must reflect that, even if those 
events are in reality executed out of order (usually to optimize program flow)
* compilers may generate instructions in a different order than in the source code
* variables could be stored in registers instead of memory
* processors may execute instructions in parallel or out of order
* caches may vary the order in which writes to variables are committed to main memory
* values stored in processor-local caches may not be visible to other processors
* Since most of the time threads are doing their own things, excessive inter-thread
  coordination will have no real benefit (only slows down the application)
* to guarantee that the thread executing action B can see the results of action A, there must
  be a happens-before relationship between A and B (partial ordering)
    * whether or not A and B occur in different threads
* in the absence of a happens-before ordering between two operations, the JVM is free to reorder them
* data race
    * a variable is read by more than one thread
    * and written by at least one thread
    * but the reads and writes are not ordered by happens-before
* rules for happens-before:
    * **program order rule**: each action in a thread happens-before every action
    in that thread that comes later in the program order
    * **monitor lock rule**: an unlock on a monitor lock happens-before every
    subsequent lock on that same monitor lock
    * **volatile variable rule**: a write to a volatile field happens-before every
    subsequent read of that same field
    * **thread start rule**: call to `Thread.start` on a thread happens-before
    every action in the started thread
    * **thread termination rule**: any action in a thread happens-before any
    other thread detects that thread has terminated (either by successfully
    return from `Thread.join` or by `Thread.isAlive` returning
    `false`)
    * **interruption rule**: thread calling interrupt on another thread
    happens-before the interrupted thread detects the interrupt (either
    by having `InterruptedException` thrown, or invoking `isInterrupted`
    or `interrupted`)
    * **finalizer rule**: the end of a constructor for an object happens-before
    the start of the finalizer for that object
    * **transitivity**: A happens-before B AND B happens-before C => A happens-before C
    * when two threads synchronize on different locks, we can’t say anything about the ordering
      of actions between them — there is no happens-before relation
  
## volatile
* Java Memory Model ensures that all threads see a consistent value for the variable
* every read of a volatile variable will be read from the computer's main memory, and not from the CPU cache
* every write to a volatile variable will be written to main memory, and not just to the CPU cache
* prevents compiler / CPU code reordering
* when we write to a volatile variable, it creates a **happens-before** relationship with each subsequent 
read of that same variable 
    * so any memory writes that have been done until that volatile variable write, will subsequently 
    be visible to any statements that follow the read of that volatile variable
* characterizing the situations under which volatile can be used safely involves determining whether 
each update operation can be performed as a single atomic update.

### remarks
* updates to non-volatile long or a double may not be atomic, and
* Java operators like ++ and += are not atomic.
* why an Object member variable can't be both final and volatile in Java?
    * JMM promises that after `ctor` is finished any thread will see the same (correct) value of final field
