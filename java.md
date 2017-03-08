## Java Basics
- java is always pass-by-value
- except the primitive types, all objects value are pointer types (but unlike c++, no * needed to decode the pointer)
- when passing object around, pointer value are actually passed around, thus reassignment will only change the pointer itself, not the contents the pointer pointed to
- keyword 'final' only cast the constness of the pointer iself, rather than the contents the pointer pointing to


## String Pool
- String is immutable in java, which makes intern pool possible
- String pool is a pool of constant strings, whose memory are in Heap
- using "" to assign a string, will return a reference of the "string" from the heap
- using 'new' operator to create a string, will force to create a string in the heap, but outside of the string pool
- using String.intern() method, will put the underline string into string pool, and reference to the string in the string pool, and free extra memory from the heap

## Immutable Class
- immutable class is good for caching, and is inherently thread-safe
- how to create immutable class? steps
    - declare class as final, thus cannot be extended
    - mark all data member as private, thus no direct access
    - do not provide any setter methods, thus cannot change from outside
    - make all mutable fields final, thus can only assign once
    - initialize all fileds via constructor performing deep copy
    - perform cloning of objects in getter method to return a copy of the object, instead of a reference to the actual object

## Java References
- Strong Reference
    - used everywhere, and GC will not be able to collect the memory refered by strong reference
- Soft Reference
    - declared soft reference variable -> soft reference object -> the object itself
    - GC will collect the memory refered by soft reference only when it thinks JVM is out of memory
    - using get() to retrieve a strong reference to the object
- Weak Reference
    - declared weak reference variable -> weak reference object -> the object itself
    - GC will collect the memory refered by weak reference whenever GC is running
    - using get() to retrieve a strong reference to the object
- Phantom Reference
    - get() will always  return null
    - every java object has a finalize() method, which is supposed to be called to do some cleanup work before GC actually reclaim the memory
    - however, when GC will run is totaly unpredicable, thus we donot know when these cleanup work is performed
    - phantom reference is the rescue
    - phantom reference is declared together with a reference queue (rq), whenever GC has called the finalize() method, the phantom reference will be put into the reference queue
    - by checking the contents of reference queue (non-blocking: poll(), blocking: remove()), one can know if finalize() method is called

## Java Memory Model

|   Edegn      | S0 | S1 |   Old Memory | Perm | 
|<- Minor GC ->| <       |<- Major GC ->|
|<- Young Gen (-Xmn)   ->|
|<- JVM Heap (-Xms, -Xmx)             ->|

### Composition
- Young Generation
- Old Generation
- Permanent Generation

#### Young Generation
- consist of three regions: Eden Memory and two Survivor Memory
- at any time, one of the two Survivor Memory is empty
- all new objects are created in Edgen Memory, when it is full, Minor GC will be performed to move survived object in Eden Memory and one of the Survivor Memory to the other Survivor Memory
- objects survived after a few rounds of Minor GC will be moved to Old Generation
- Tunning
    - -Xms (initial heap size)
    - -Xmx (max heap size)
    - -Xmn (size of Young Gen, rest will be for Old Gen)

#### Old Generation
- Contains long lived objects that survived many rounds of Minor GC
- Major GC will be performed when Old Generation is full

#### Permanent Generation
- contains application metadata required by JVM to describe class and methods used in the application
- is is not part of the Heap Memory
- populated by JVM at run time based on classes used by the application
- objects here are garbage collected in a full GC
- Method Area
    - in Perm Gen used to store class structure (runtime constants/static variables), code for methods
- Memory Pool
    - can belong to either Heap or Perm Gen
    - pool of immutable objects created by JVM memory manager (e.g., string pool)
- Tunning
    - -XX:PermGen (initial size)
    - XX:MaxPermGen (max size)

### Gabarge Collection Type
- Serial GC (-XX:+UseSerialGC)
    - mark-sweep-compact procedure for Young/Old Gen GC
    - single thread GC
- Parallel GC (-XX:+UseParallelGC)
    - N threads for Young Gen GC (-XX:ParallelGCThreads=N)
    - single thred for Old Gen GC
- Parallel Old GC (-XX:+UseParallelOldGC)
    - same with Paralel GC, but use N threads for Old Gen GC
- Concurrent Mark Sweep (CMS) Collector (-XX:+UseConcMarkSweepGC)
    - minimize the pause, and do most of Old Gen GC work concurrent with the application thread
    - Young Gen GC is same with Parallel GC
    - -XX:ParallelCMSThreads=N to config N threads for Old Gen GC
- G1 Garbage Collector (-XX:+UseG1GC)
    - G1 (Garbage First) collector: long term replacement for CMS collector
    - heap is partitioned into a set of equal-sized heap regionsk (No distinguishing between Young/Old Gen)
    - G1 performs a concurrent global marking phase to determine the liveness of objects throughout the heap
    - G1 knows which regions are mostly empty. It collects in these regions first, which usually yields a large amount of free space.
    - G1 concentrates its collection and compaction activity on the areas of the heap that are likely to be full of reclaimable objects
    - G1 uses a pause prediction model to meet a user-defined pause time target and selects the number of regions to collect based on the specified pause time target.

### Heap vs Stack
- Heap is used by all parts of the application  vs  Stack is only used for thread execution
- Heap stores the contents of all the objects vs Stack only stores the pointer of the object (object variable), as well as primitive types
- Heap contents is globally accessible vs Stack contents is only available within its scope
- Heap is where Java Garbage Collection is performed vs Stack is managed in a LIFO manner
- Heap memory lives from start to the end of the application vs Stack memory is short lived
- Heap memory full -> OutOfMemoryException vs Stack memory full -> StackOverflowException
- Heap mmemory use -Xms/-Xmm to control start/maximal size vs Stack memory size use -Xss to control
- Stack is faster than heap memory


## Design Pattern in Java
### Creational Design Pattern
- Singleton Pattern
- Factory Pattern
- Abstract Factory Pattern
- Builder Pattern
- Prototype Pattern
- Object Pool Pattern

#### Static Factory Method
- How?
    - disable all constructors, thus cannot create object using constructor
    - declare static create() function to create object, thus can be called before object is created
- Why?
    - this concept is widly use in different design patterns
    - create() can add extra layer to manager the creation of the objects
        - depending on different parameters, create subclasses
        - limit the number of object instances created (singleton/pool of objects)
    - create() function signatures/parameters can be renamed, to improve readability

#### Singleton Pattern
- When?
    - only one instantiation of the class is allowed in the JVM, like logging/driver/caches/thread pool, and etc
- How?
    - mark constructor as private to disable constructors
    - declare private static instance variable of the same class
    - declare public static getInstance() method to return the instance
    - different implementations
        - eager initialization: initialize the static instance variable when declaration (cannot handle creation exception, object created before used, might never used)
        - static block initialization: add a static block of codes after instance declaration, in the block, add exception handling, but still object created too early, might never used
        - lazy initialization: only when getInstance() method is called, create instance if null, otherwise return the instance
        - thread-safe initialization: 
            - mark getInstance() method synchronized -> too much overhead if the instance is already created
            - use synchronized block with double 'instance == null' check, if first check is null, put another check in synchronized block; if not, directly return

#### Builder Pattern
- when? 
    - where there are lots of attribute (required and optional)
- why? 
    - error prone to pass all the arguments in the right order
    - factory method will force to pass all the arguments, even if it is optional, have to pass 'null'
    - if the object creation is complex, all the complexity will go up to factory method
- how?
    - for class X, create a 'nested static class' called XBuilder, with all the data members from class X
    - XBuilder class should provide constructor with required data members as parameters
    - XBuilder class should provide setter methods for all optional data members, and the setter should return 'this' referencing to XBuilder, thus can chaining all setter to construct the object
    - XBuilder class should provide build() method to return new X(XBuilder), for this class X itself needs to provide private constructor taking XBuilder as a parameter

#### Object Pool Pattern
- When?
    - when the resource stateless object is costly to create, and is limited, but multiple clients need to access the resource, thus use object pool pattern to reuse the resource
- How?
    - create the reusable object
    - main a pool of resusable object, which manages allocate reusable object to client, mark as available when client finishes using it
    - client request object from the pool as if creating a new object, and inform the pool after using it
- Examples?
    - database connections

#### Dependency Injection Pattern (a.k.a. Inversion of Control)
- When?
    - when declaring a member object inside a class, rather than using new to create a member object, it is better passed by reference, and the reference would be better a interface, thus can be extended without changing the client code
- How?
    - constructor injection
    - setter injection
    - interface injection

### Structural Design Pattern
- Adapter Pattern
- Composite Pattern
- Proxy Pattern
- Flyweight Pattern
- Facade Pattern
- Bridge Pattern
- Decorator Pattern

#### Flyweight Pattern
- When?
    - when need to create tons of object and memory is limited, thus use this pattern to share the objects
    - object same object's properties can be divided into intrinsic and extrinsice
    - intrinsic properties are those that come with the object, and won't change
    - extrinsic properties are those that come from the client
- How?
    - flyweight interface: defines the interface that manipulate the shared object by the extrinsic properties
    - concrete flyweight: stores the intrinsic properties and implements the flyweight interface
    - flyweight factory: maintain and manage the flyweight object. if the flyweight object is not created, create it; if created, return reference to the flyweight object
    - client: maintains reference to the shared object, and computes and manages the extrinsic properties of the shared object
- Examples?
    - Java string pool
    - In the game where tons of soilder need to be drawn, flyweight interface defints how to move soilders, and concreate flyweight object defines all intrinsic properties needed to draw the soilder; client controls how to move the soilder around


### Behavioral Design Pattern
- Template Method Pattern
- Mediator Pattern
- Chain of Responsibility Pattern
- Observer Pattern
- Strategy Pattern
- Command Pattern
- State Pattern
- Visitor Pattern
- Interpreter Pattern
- Iterator Pattern
- Memento Pattern
