### Java Basics
- java is always pass-by-value
- except the primitive types, all objects value are pointer types (but unlike c++, no * needed to decode the pointer)
- when passing object around, pointer value are actually passed around, thus reassignment will only change the pointer itself, not the contents the pointer pointed to
- keyword 'final' only cast the constness of the pointer iself, rather than the contents the pointer pointing to


### Heap vs Stack
- Heap is used by all parts of the application  vs  Stack is only used for thread execution
- Heap stores the contents of all the objects vs Stack only stores the pointer of the object (object variable), as well as primitive types
- Heap contents is globally accessible vs Stack contents is only available within its scope
- Heap is where Java Garbage Collection is performed vs Stack is managed in a LIFO manner
- Heap memory lives from start to the end of the application vs Stack memory is short lived
- Heap memory full -> OutOfMemoryException vs Stack memory full -> StackOverflowException
- Heap mmemory use -Xms/-Xmm to control start/maximal size vs Stack memory size use -Xss to control
- Stack is faster than heap memory

### String Pool
- String is immutable in java, which makes intern pool possible
- String pool is a pool of constant strings, whose memory are in Heap
- using "" to assign a string, will return a reference of the "string" from the heap
- using 'new' operator to create a string, will force to create a string in the heap, but outside of the string pool
- using String.intern() method, will put the underline string into string pool, and reference to the string in the string pool, and free extra memory from the heap
 

### Immutable Class
- immutable class is good for caching, and is inherently thread-safe
- how to create immutable class? steps
    - declare class as final, thus cannot be extended
    - mark all data member as private, thus no direct access
    - do not provide any setter methods, thus cannot change from outside
    - make all mutable fields final, thus can only assign once
    - initialize all fileds via constructor performing deep copy
    - perform cloning of objects in getter method to return a copy of the object, instead of a reference to the actual object
