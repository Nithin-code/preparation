  # Corutines and Threads.

### Threads : 
1. Threads can run in parallel
2. creating too many threads is expensive (every thread uses memory and cpu).
   
### Executors : 
1. executors resue thread pools.
2. since using many threads leads to app unresponsive, to handel it developers have also used
   the executors which is a thread pool that reuses a fixed no of threads.

### Coroutines : 
 1. kotlin coroutines are language level feature designed to simplify asynchronous programming and concurrency.
 2. coroutines is a light-weight co-operative unit of work that can be paused and resumed later without blocking main thread.
 3. unlike thread, a coroutine does not necessarily run concurrently on a separate cpu core
 4. coroutines follow structured concurrency it means coroutines are launched in a specific scope and their lifecycle is unmanaged so that you can avoid leaks and unexpected behaviors.

### Coroutine Context : 
* it is an indexed set of elements, essentially it controls how and where the coroutine runs. there are four commonly used elements of a coroutine context.
  1. CoroutineDispatcher.
  2. Job.
  3. CoroutineName.
  4. CoroutineExceptionHandler.

### CoroutineDispacther : 
* Decides what thread or thread pool that coroutine runs on..
  1. Defalut - Optimised for cpu intensive tasks (heavy calculations like sorting lits, parsing json)
  2. IO - used for network calls, databases and file management
  3. Main - used for UI tasks
  4. Unconfired - starts in current thread, amy resume in another
 
### Job : 
* Represnts the coroutine's lifecycle.
* lets u cancle it, check if it's active and handle structured concurrency.
* Job is the only coroutine context element that is not inherited by a coroutine from a coroutine.
* Job is not inherited because each coroutine needs its own job to track its lifecycle, while still being linked to parent for structured concurrency.

### CoroutineName : 
* Useful for debugging/logging.
* you can give a coroutine a human-readable name like ("Downloader")

### CoroutineExceptionHandler: 
- Defines how uncaught exceptions should be handled inside the coroutine.
- The exception handler should be used only inside the root coroutine and not the child coroutines. (because this exceptiona handler only cacthes exception in parent coroutine)

## Difference between coroutine-scope and coroutuine-context : 

Coroutine-Scope  | Coroutine-Context
------------- | -------------
Defines lifetime of a coroutine. it determines when it started and when they are cancelled. | Defines behavior of a coroutine.
viewmodel scope cancels all its coroutines when the veiw Model is cleared.  | it is an indexed set of elements, essentially it controls how and where the coroutine runs

### Corutine Builders : 
 1. Launch() : 
    * The launch builder start a new coroutine and returns a job to manage its lifecycle, making it ideal for tasks that perform work but dont produce a result, ex : saving data, sending network request.
    * we cannot call launch outside of a coroutine with out a coroutine scope.

 2. Async() :
    * it is similar to lauch, but it is designed to produce a value.
    * the async fun returns a deffered object.
    * Deffered has a supending function called await(), which returns the value once it is ready.
   
      
### Structured Concurrency : 
* it is a design principle in kotlin that helps you manage coroutines in a safe predictable and maintainable way.
* structured concurrency means every coroutine has a clear life cycle tied to a scope.
* child coroutine inherit context from their parent.
* the parent coroutine waits for all child coroutine to complete before it finished.
* if a child fails, the failure can propagate to parent optionally cancelling other childern.
* if a child is destroyed, it also destroys the parent.

  ----------------------------------------------------------------------------------------------------------------

# Flows : 
- kotlin flow is a key building block for handeling asyncronous data stream in a structured and safe way.
- It is part of kotlin coroutines and is designed to model a sequence of values that computed asynchronously.(KMP ready)
- Flow can be suspended between emissions making it idel for operations like fecthing data from the network, reading database or emitting values over time.

## collect : 
- procress every emmission from the flow.
- each collector block completes before the next emmission is handelled.
- useful when everyvalue matters, and noting should be skipped.

## collectLatest :
- always procress only latest emmission.
- cancels the ongoing collector block if a new value is emmited.
- useful when only the most recent value matters, like search suggestions and UI updates.

## Imp points: 

- The producer can emit values independently of the consumer's procressing speed.
- The collector collects vaues whenever they arrive, potentially on a different thread, with potentially suspending the producer.
- This allows efficient handling of events, such as network responses, user inputs or sensor data, when values arrive over time.

## Cold Flows : 
- A cold stream meaning that the flow's block doesn't execute until it is collected.
- Each new collector triggers a fresh exection of the flow, it means every collect re-triggers the block.
- This allows flows to reusable, lazy and memory efficient, as data is produced only when needed.
- Garbage-Friendly, once a collector finishes, the emitted values can be garbage collected.

## Hot flows : 
- If you start listening late, you miss the songs that already played(unless the flow replayed them).
- The Producer is active all the time, independent of collectors.
  

