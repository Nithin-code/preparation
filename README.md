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
  
## Flow Builders : 
- A flow builder is a  function that creates a flow,
- By Default, the builder runs on the same coroutine context as the collector, unless you use flowOn.

## Types of flow builders & when to use:
- flow :
  * when you need custom logic to generate values.
  * when values require computation, suspension or looping.
  * when you are bridging between suspending code and reactive streams.
- flowOf :
  * use when you already know the values in andvance.
  * used for testing as well.
- asFlow :
  * it is an extension function
  * use when any there is exesting sequence, or range can be turned into flow with asFlow().
  * when u want to transform or process collections reactively.
  * for converting existing data pipelines into flow without rewriting them.
- channelFlow :
  * This builder provides a coroutine-safe way to emit values from multiple coroutines.
  * It uses channel underthe hood.
  * Use it when you need to emit values concurrently from different coroutines.
  * when combining multiple sources inside one flow 
- callbackFlow :
  * This builder adapts callback-based api's into flows.
  * It is a powerful bridge when dealing with legacy or platform Api's.
  * Use it when you are working with Anroid Listeners (e.g., location updates, sensor events)
  * For Api's that push values via callbacks insted of returning them directly.

## Structured Concurrency with Flow : 
- Structured Concurrency ensures that coroutines are launched within a well-defined scope and are tied to that scope's life cycle.
- When the parent completes or cancelled, all its children are cleaned up automatically.
- since the flow is actually run inside a certain kotlin coroutine scope, it means that when sope is cancelled, the flow will be also canceled as well.

## Flows Cancellation : 
- If the collector is cancelled the upstream emission stops.
- This Prevents wasted work (no unnecessary network requests, no extra cpu cycles).
- It keeps resources safe - database queries, API calls or file I/O tied to the flow will be canceled as well.

## LifeCycle of Flows : 
- cold flow -> On demand computation (like youtube video starting when you press play).
- Hot Flow -> Always Active Stream(like a FM Radio).

## Efficiency : 
- A cold Flow is more performant when you have few collectors or need on-demand data.(since it runs only during collections).
- A state Flow(Hot Flow) is more performant when you have many collectors, because it shares emissions and avoids re-running upstream logic for each one.

## IMP point : 
- A kotlin flow is completely lifecyle-agnostic, means it does not know anything about android activity, fragments or composables.
- What determines how long it runs is the coroutuine scope in which its being collected.

## Flow Operators :
1. ## Terminal Flow Operators :
   - In kotlin Flow, a terminal operator is an operation that triggers the execution of the flow.
   - Until you call a Terminal Operator, a Flow is cold - it doesn't emit or do anything.
   - Terminal operators practically complete the flow and do not collect anything afterwards.
     1. collect / collectLatest()
     2. first/firstornull()
     3. toList()/toSet() - returns a list 
     4. single/singleOrNull() - returns only a single element from that flow.
     5. last/lastOrNull() 
     6. launchIn() -
## map() operator : 
 - It Allows you to transform each value emmited by flow into another value.
 - Think it as the equalent of map function in lists, but for asynchronous streams.
 - ## uses :
     - Use it for applying transformations to incoming data, such as formatting, converting types, or computing derived values.
     - converts every emission from the original type to other one.
     - only executes when flow is collected and can be chained with other operators.
     - the order of elements in the flow remains unchanged, making it predictable for sequential transformations.
## filter() operator : 
 - The Filter Operator allows you to only filter the values that sastify a condition to pass downstream.
 - ## uses :
     - Used for filtering user input, network responses, or sensor events before further processing.
     - Downstream operators or collectors won't receive irrelevant data.
     - Often paired with map or take to create consise, declarative pipelines.
## take() operator : 
- allows you to first n values from the flow.
- ## uses :
    - useful for timeout scenarios, limiting UI updates, or taking only few events from a large stream.
    - Only collects the first N elements, then cancels the upstream flow automatically.
    - Ideal for stopping long-running or infinate flows once enough data is collected.
## distintUntilChanged() opeator : 
- This operator ensures that consecutive duplicate values are ignored, emitting only when a new value differs from the prevoius one.
- #### note : it will not remove those duplicate values from the flow.
# Exception handling in flows: 
 - ## catch operator :
    - catches exceptions upstream of where catch is applied.
    - can emit fallback values to continue the flow after an error.
    - only catches exceptions in the flow builder or operatores above it.(not inside collect)
    - Example :
    - <img width="460" height="192" alt="{D1BCF09F-C55A-4766-8EE4-B1A3320AE57C}" src="https://github.com/user-attachments/assets/f5c8b323-84d0-4e6a-b5c8-72e1b7375dee" />
    - <img width="306" height="97" alt="{CAEBE947-C147-412D-B81A-983C17BEA683}" src="https://github.com/user-attachments/assets/d8ffd3cb-4a48-447d-bc14-b49fe5e71011" />
    - output is :
    - <img width="486" height="68" alt="{51D9BE52-06DF-48F8-BF00-C289F99905FD}" src="https://github.com/user-attachments/assets/2b393d6e-405d-4d57-97cb-5c07bf9ae79c" />


      
