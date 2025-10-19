# Kotlin-Coroutines
Revision for Coroutines



1. Threads:

⦁	Normal FLow of Work -> print(Start) -> count from (1 to 50 M) -> print(End). 

Will print start, then take some time to execute the count function, then after sometime will print End. 

⦁	But, if we put the count func in a thread, print(Start) -> thread {count from (1 to 50 M)} -> print(End)

Now the thread will make its own separate independent workflow. 

First start will be printed (main function workflow), and then the thread will separately start executing from the main work flow, parallelly (separate independent workflow). 

So End will be printed (main function workflow), then in separate workflow when the count from 1 to 50 M ends, then it is stopped.


⦁	Having a lot of work and funtions, will have to allocate a lot of differnt threads to work in parallel. 

Each thread take memory space and computation (CPU usage). 

So more threads will mean occupying more space and more computation, which will reduce performance.


To Handle this we have :

---

2. Kotlin Coroutines:

⦁	Kotlin are lightweight threads which are managed by Kotlin runtime, not by OS.


⦁	From a single thread, we can alot of kotlin coroutines. 

Like this we dont have to create or work on many threads, from a single thread we can create many kotlin coroutines which further have suspend functions.


⦁	Suspend functions are the functions that have the ability to suspend or work in the background if the function is taking its time to execute, and in the meanwhile some other work can be done (different coroutine) on the thread. 


⦁	Kotlin Builders are used to build Coroutines. 

2 types of Kotlin Builders:  
(1.) StandAlone Kotlin Builder  
(2.) Scoped Kotlin Builders


---

⦁	StandAlone Builder:


1. **runBlocking:**  
   Will block the main thread and will execute whatever is mentioned in the runBlocking function and then resume the main thread.


2. **Launch:**  
   Extension of Coroutine Scope.  
   Follows the 'launch and forget' mechanism.  
   Just launch any work function inside it, without caring about the results.


3. **Async Await:**  
   If we are interested about the results, unlike Launch, Async Await is used.  
   Also an extension of Coroutine scope.  

   When we use  
   `val funResult = async { any working function }`  
   It will start running the work function mentioned under the hood with Coroutine, and will store the result internally inside it.  

   It will only show the result when we use await/onComplete like this ->  
   `println(funResult.await())`  

   `await` -> will wait for the function to fully complete and then return.  
   `onComlete` -> will not wait and return at exact moment.  
   So await is preferred.


---

⦁	Coroutine Start:

1. **default:** Start immediately  
2. **lazy:** we can write the function under lazy, then call it after.  
3. **Atomic:** atomic is only cancellled when we reach the first suspended function, unlike default which can be cancelled immediately.  
4. **Undispatched:** it starts immediately, but the feature is that it can change the thread after the first suspended function, if another thread is mentioned. 


---

⦁	Scoped Builder:

2 Types (Difference is ability to handle the exception):

1. **CoroutineScope Builder:**  
   It stops the function immediately when it encounters an exception.

2. **SupervisorScope Builder:**  
   It shows the exception but still performs the tasks appearing after the exception.


---

⦁	Coroutine Context -> Group of elements of Coroutine:

1. **Dispacthers:** on which thread we want to use the Coroutines -> Default, Main, IO, Unconfined. (Discussed later)  
2. **exceptionHandler**


---

⦁	Dispatchers -> on which thread we want to use the Coroutines:

1. **Dispatchers.Main:**  
   Used to update UI constantly.  
   If we try to use this thread for some other logic (like network calls) which takes time, the application may crash or not respond, because UI will not be updated as Main thread is busy doing other tasks.  
   By default, every suspend function works in Main thread.

2. **Dispatchers.IO:**  
   Best for network calls and database calls (either local database or API calls), due to large number of coroutines available in this thread.

3. **Dispatchers.Default:**  
   Used for heavy computations. (example - for audio transformations, video transformations)

4. **Dispatchers.Unconfined:**  
   Will start with thread with which it is called, then after reaching first suspend function, it will change the thread.


---

⦁	withContext -> used to switch between the threads.


---

⦁	Jobs:

Job is simply a coroutine ->  
`job = launch(start = CourotineStart.Lazy) { work funtion }`


Now, we can do:

1. **job.start():**  
   Start the workflow in normal way.  
   If current work takes more time, it will suspend, and start working on outside of this launch function (Coroutine).

2. **job.join():**  
   Unlike job.start(), it will first complete the launch function, then move ahead, even if it takes time.

3. **job.cancel():**  
   Simply cancel/stop/terminate the coroutine.

4. **job.invokeOnCompletion():**  
   For exception handling.


⦁	State of Jobs:  

When we create a job, by default, it is in the NEW State.
⦁	job.isActive(), job.isCompleted(), job.isCancelled()

<img width="700" height="350" alt="Screenshot 2025-10-19 220507" src="https://github.com/user-attachments/assets/fdf8884d-223c-4ad9-ac35-a9aa2bc83c31" />



⦁	Parent-Children Relation:  
We can create many children of Coroutine like -> `launch { launch { launch {} } }`  

Whenever any children throws exception then the all the parent hierarchy is stopped and the exception is thrown.  

To handle this we can create **supervisor Jobs** which can still perform the tasks even if any children throws exceptions.


---

⦁	Coroutine Scope:

It is the context that manage the lifecycle of the Coroutine.  
All the Coroutine gets cancelled when it's lifecycle gets cancelled.


5 types:

1. **GlobalScope:**  
   Its lifecycle is tied to the application.  
   Not recommended because it can cause memory leaks.

2. **MainScope:**  
   Used for UI as it uses Main thread.

3. **lifeCycleScope:**  
   Tied with the activity/fragment lifecycle.  
   When activity/fragment lifecycle is destroyed, all coroutines with this CoroutineScope also get destroyed.  
   If anything not related to UI, can use this.  
   Also, we can switch anytime we want by using `withContext()`.

4. **viewModelScope:**  
   Tied to the ViewModel.  
   When ViewModel lifecycle is destroyed, all coroutines with this CoroutineScope also get destroyed.

5. **Custom Scope:**  
   We can create our own custom Coroutine scope.  
   Create the job, then define  
   `scope = CoroutineScope(job + Dispatcher.Main)`  
   Just simply do `scope.launch {}` and also `scope.cancel()`.



