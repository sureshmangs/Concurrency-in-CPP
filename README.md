# Concurrency in C++


### Contents <a name="top"></a>

1. [Introduction](#1)       
    1.1 [Concurrency](#1.1)     
    1.2 [Parallelism](#1.2)     
    1.3 [Why do we need concurrency?](#1.3)     
    1.4 [How to achieve concurrency?](#1.4)     
2. [Thread References](#2)        
    2.1 [Program, Process and Thread](#2.1)     
    2.2 [Creating Threads](#2.2)     
            2.2.1 [Ways to create and manage threads](#2.2.1)     
    2.3 [Race Condition](#2.3)     
    2.4 [Mutex](#2.4)     
    2.5 [Lock Guard](#2.5)     
    2.6 [Deadlock](#2.6)     
    2.7 [Lock](#2.7)     
    2.8 [Unique Lock](#2.8)     
    2.9 [Condition Variable](#2.9)     
3. [Concurrency References](#3)        
    3.1 [Future](#3.1)        
    3.2 [Async](#3.2)        
    3.3 [Promise](#3.3)        
    3.4 [Shared Future](#3.4)        
    3.5 [Packaged Task](#3.5)        
    3.6 [Decide which to use: threads or task-based parallelism](#3.6)
4. [References](#4)        


## 1. Introduction <a name="1"></a>

### 1.1 Concurrency <a name="1.1"></a>

> Concurrency refers to the ability of a system to handle multiple tasks or processes in a way that they can make progress without necessarily being executed simultaneously.

It's about structuring a program so that it can manage multiple tasks that may be running at overlapping times. Concurrency involves managing the coordination and interleaving of tasks to ensure that they do not interfere with each other and that system resources are used efficiently.

**Key Points:**
- Overlapping Execution: Tasks or processes make progress in overlapping time periods.
- Task Management: Tasks may not be running at the exact same instant but are managed to give the illusion of simultaneous execution.
- Interleaving: On single-core systems, this often involves switching between tasks so quickly that it appears they are running simultaneously.

Example: An operating system that manages multiple applications, allowing users to switch between them while each application progresses independently, is leveraging concurrency.


### 1.2 Parallelism <a name="1.2"></a>

> Parallelism refers to the simultaneous execution of multiple tasks or processes. It involves physically performing multiple operations at the same time using multiple processing units, such as multiple CPU cores or processors.

Parallelism is a subset of concurrency and focuses on the actual simultaneous execution of tasks.

**Key Points:**
- Simultaneous Execution: Tasks are executed at the exact same time, utilizing multiple processing resources.
- Multi-Core Processing: Takes advantage of multiple cores or processors to perform computations simultaneously.
- Data Parallelism: Involves splitting data into chunks and processing each chunk concurrently.

Example: A multi-core processor running multiple threads of a program at the same time, where each core executes a different thread, is utilizing parallelism.


### 1.3 Why do we need concurrency? <a name="1.3"></a>

Concurrency is used to manage and optimize the execution of multiple tasks. It's commonly used for:

1. Improves Responsiveness

- User Interfaces: In applications with graphical user interfaces (GUIs), concurrency helps keep the interface responsive while performing background tasks. For example, a web browser can download files or process data while still allowing users to interact with the interface.

2. Efficient Resource Utilization

- CPU Utilization: Concurrency allows a system to better utilize the CPU. On single-core processors, it enables the CPU to switch between tasks, giving the impression that multiple tasks are running simultaneously. On multi-core systems, it helps distribute tasks across multiple cores, improving overall performance.

3. Better Throughput

- Handling Multiple Requests: Concurrency is crucial in server environments where multiple requests need to be handled at the same time. For instance, a web server can handle many incoming requests concurrently, reducing the waiting time for users and increasing the server's throughput.

4. Scalability

- Adapting to Load: Concurrency allows applications to scale with increasing workloads. By managing multiple tasks simultaneously, systems can handle more users, requests, or data without a linear increase in response time or resource consumption.

Example Scenarios:

- Web Servers: A web server uses concurrency to manage multiple client requests simultaneously, ensuring that each request is processed without waiting for others to finish.

- Real-Time Systems: Systems that require real-time processing, such as video streaming or online gaming, use concurrency to handle multiple data streams and user interactions effectively.

- Operating Systems: Modern operating systems use concurrency to handle multiple processes and threads, managing system resources efficiently and providing a smooth user experience.


### 1.4 How to achieve concurrency? <a name="1.4"></a>

Concurrency allows multiple tasks to execute simultaneously, enhancing the performance and responsiveness of applications. C++ offers several ways to achieve concurrency, from low-level thread management to high-level abstractions like tasks and futures.




## 2.Thread References <a name="2"></a>

### 2.1. Program, Process and Thread <a name="2.1"></a>

**1. Program**

> A program is a set of instructions written in a programming language that performs a specific task or solves a particular problem. It is essentially a static piece of code that doesn’t execute until it’s loaded into memory.

**Characteristics:**
- Static: A program is stored on disk as an executable file or script and does not execute until initiated.
- Code and Data: Contains the code (instructions) and, sometimes, data required to perform its functions.
- Development Stage: A program is created during the development phase and can be compiled or interpreted into a format that the operating system can execute.

Example: A text editor application (like Notepad or Vim) is a program. It contains the code that defines how the text editor should work.

**2. Process**

> A process is an instance of a program that is being executed. It is a dynamic entity that includes the program code, its current activity, and the resources it is using.

**Characteristics:**
- Execution Context: A process has its own memory space, file descriptors, and system resources. It is a running instance of a program.
- Isolation: Processes are isolated from each other, meaning one process cannot directly access the memory or resources of another process.
- Lifecycle: A process goes through various states including creation, running, waiting, and termination.

Example: When we launch a text editor on a computer, the operating system creates a process for that text editor. This process will manage the execution of the text editor’s code, handle input/output, and manage memory resources.

**3. Thread**

> A thread is the smallest unit of execution within a process. It represents a single sequence of instructions that can be scheduled and executed by the operating system.

**Characteristics:**
- Lightweight: Threads are often referred to as lightweight processes because they share the same memory space and resources of their parent process but can execute independently.
- Shared Resources: Threads within the same process share resources such as memory and file descriptors, which allows for efficient communication and resource sharing.
- Concurrency: Threads enable concurrent execution within a process, meaning multiple threads can run simultaneously to perform tasks like handling multiple user interactions, performing background operations, etc.

Example: Within the text editor process, there might be multiple threads handling different tasks, such as one thread for user interface updates, another for file operations, and another for spell-checking.


### 2.2. Creating Threads <a name="2.2"></a>

**Example:**

```cpp
#include <iostream>
#include <thread>
using namespace std;

void Greet()
{
    cout << "Hello World!\n";
}

int main()
{
    thread t1(Greet);   // Create and start a thread t1

    return 0;
}
```

Output:

```
terminate called without an active exception
Hello World!
```

Got an error, but why?

**Case 1. When `t1` Completes Before `main` Exits**

- Thread Completion: If `t1` completes before `main` exits, the thread has done its work and is no longer running.
- No Join or Detach: Because we didn't call `join()` or `detach()`, the `std::thread` object `t1` is still managing the completed thread when `main` exits.
- Destruction of `t1`: When `main` returns, `t1` is destructed. The C++ Standard Library requires that if a `std::thread` object is destructed while still representing an active thread (i.e., not joined or detached), the program will call `std::terminate()`.

**Case 2. When `main` Exits Before `t1` Completes**

- Thread Management: The thread `t1` is still running, and `main` has exited.
- Destruction of `t1`: When `main` returns, the `std::thread` object `t1` is destructed. Since `t1` is still managing an active thread, this will again result in a call to `std::terminate()`.

In both scenarios, whether `t1` completes before `main` exits or `main` exits before `t1` completes, the program will terminate abnormally because the thread `t1` is still active when it is destructed. 

To avoid this issue, we should always ensure that either `join` or `detach` the threads before the `std::thread` objects go out of scope.

- **`join()`:** Waits for the thread to finish its execution before the `std::thread` object is destroyed. This ensures that the thread has completed its work before the program ends.
  
- **`detach()`:** Separates the thread from the `std::thread` object, allowing it to run independently. The thread will continue executing even if the `std::thread` object is destroyed. However, once detached, we cannot join the thread or get its result, and we must ensure that the thread's execution is properly managed to avoid issues like accessing destroyed resources.

**Example:**

```cpp
#include <iostream>
#include <thread>
using namespace std;

void Greet()
{
    cout << "Hello World!\n";
}

int main()
{
    thread t1(Greet);

    t1.join();      // main thread waits for thread t1 to finish

    return 0;
}
```

Output:

```
Hello World!
```

**Understanding `std::thread::join`**

  - When we call `join()` on a `std::thread` object, the thread that calls `join()` (usually the main thread) will be blocked until the thread represented by the `std::thread` object completes its execution.
  - The thread that executes `join()` is the one that waits. If we call `join()` on a thread object `t1` in the main thread, then the main thread will be put to sleep until `t1` finishes its work. The thread represented by `t1` runs concurrently and completes its task independently of the main thread's execution until the `join()` is called.

**Explanation:**

1. Thread Creation:
   - `thread t1(Greet);` starts a new thread `t1` that runs the `Greet` function concurrently with the main thread.

2. Joining the Thread:
   - `t1.join();` is called in the main thread. This means the main thread will wait until `t1` has finished executing the `Greet` function.

3. Execution Flow:
   - The `Greet` function executes in the new thread `t1`.
   - Meanwhile, the main thread is blocked at `t1.join()` and does nothing until `t1` completes.

4. Completion:
   - Once `t1` has finished executing `Greet`, the `join()` call returns, allowing the main thread to continue.
   - The main thread then prints "Thread t1 has completed."

**Example:**

```cpp
#include <iostream>
#include <thread>
using namespace std;

void Greet()
{
    cout << "Hello World!\n";
}

int main()
{
    thread t1(Greet);

    t1.detach();        // t1 detached from main thread

    // Checking if the thread is joinable
    if (t1.joinable())
        cout << "Thread t1 is joinable\n";
    else
        cout << "Thread t1 is not joinable\n";

    return 0;
}
```

Output:

```
Hello World!
Thread t1 is not joinable
```

**Explanation**

1. Thread Creation:
   - `thread t1(Greet);` creates a new thread `t1` that runs the `Greet` function concurrently with the main thread.

2. Thread Detachment:
   - `t1.detach();` makes the thread `t1` run independently from the main thread. Once detached, `t1` will continue executing `Greet` in the background.

3. Main Thread Exits:
   - The main thread exits immediately after calling `t1.detach()`. Since the `main` function completes and returns, the main thread terminates.

4. Detached Thread Execution:
   - The detached thread `t1` will still be running, even though the main thread has finished. It will continue to execute `Greet` and print "Hello World!" unless the program terminates before it finishes.

5. Resource Management:
   - After a thread is detached, the system will eventually reclaim the resources used by the thread when it completes. The detached thread operates independently, and its execution is not directly managed or synchronized with the main thread anymore.

**Note:**

- Unpredictability: There is some unpredictability with detached threads. If the main thread exits very quickly, the detached thread might not have enough time to complete its work. However, most implementations of the C++ Standard Library will ensure that the detached thread gets enough time to finish, or the resources are properly reclaimed by the runtime system.

- Resource Management: When a thread is detached, we cannot join it later or get its return value. It's important to ensure that the thread's execution does not rely on resources that might be destroyed by the main thread or other parts of the program.

- Thread Joinability: After a thread is joined or detached, it becomes unjoinable. If we call `join()` or `detach()` on a thread that has already been joined or detached, we will get undefined behavior. We can call `joinable()` to check if a thread is available to be joined.


**Example:**

```cpp
#include <iostream>
#include <thread>
using namespace std;

void Greet(string name)
{
    cout << "Greet: Hello " << name << "!\n";
}

int main()
{
    string name = "World";

    thread t1(Greet, name);
    t1.join();

    cout << "Main: Hello " << name  << "\n";

    return 0;
}
```

Output:

```
Greet: Hello World!
Main: Hello World
```

When we start a thread in C++, arguments passed to the thread function are passed by value by default. This means that the thread function receives a copy of the arguments, not the original objects.

**Example:**

```cpp
#include <iostream>
#include <thread>
using namespace std;

void Greet(string &name)
{
    cout << "Greet: Hello " << name << "!\n";
    name = "Everyone";
}

int main()
{
    string name = "World";

    thread t1(Greet, ref(name));
    t1.join();

    cout << "Main: Hello " << name  << "\n";

    return 0;
}
```

Output:

```
Greet: Hello World!
Main: Hello Everyone
```

By default, arguments passed to functions or thread functions in C++ are passed by value. This means a copy of the argument is made. If we want to pass arguments by reference, we need to use `std::ref` to wrap the arguments. This allows the function to operate on the original object rather than a copy.

`std::ref` is a utility function provided by the C++ Standard Library in the `<utility>` header that allows us to pass arguments by reference to functions or thread functions.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <unistd.h>  // For POSIX systems
using namespace std;

void Greet()
{
    cout << "Hello World!\n";
}

int main()
{
    // Print the process ID
    std::cout << "Process ID: " << getpid() << "\n";

    // Prints the ID of the current thread (the main thread in this case)
    cout << "Main thread ID: " << this_thread::get_id() << "\n";

    // Prints the number of threads that can be executed concurrently by the hardware.
    // This is often equal to the number of CPU cores available
    cout << "Hardware concurrency: " << thread::hardware_concurrency() << "\n";

    thread t1(Greet);

    // Prints the ID of the newly created thread t1
    cout << "t1 thread ID: " << t1.get_id() << "\n";

    t1.join();

    return 0;
}
```

Output:

```
Process ID: 372
Main thread ID: 139636177962176
Hardware concurrency: 4
t1 thread ID: 139636177946304
Hello World!
```

**Process ID:**
> The unique identifier assigned by the operating system to the process currently running our program. It helps the OS manage and track processes.

**thread ID:**
> The unique identifier assigned to the thread of the program. Each thread in a process has a unique ID, which helps in distinguishing between different threads.

**Hardware concurrency:**
> The number of threads that the hardware (CPU) can handle simultaneously. This often corresponds to the number of CPU cores or logical processors available.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <utility>  // move
using namespace std;

void Greet()
{
    cout << "Hello World!\n";
}

int main()
{
    thread t1(Greet);   // Create and start the thread

    // Transfer ownership from t1 to t2
    thread t2 = move(t1);

    // t1 is now in a valid but unspecified state, so do not use it
    // Attempting to join t1 here would lead to undefined behavior

    // Join t2 to ensure the thread completes before exiting
    t2.join();

    return 0;
}
```

Output:

```
Hello World!
```

**Note:**

- Ownership Transfer:
  - `std::move`: A standard library function that casts an object to an rvalue reference, enabling the transfer of ownership or resources from one object to another.
  - `std::move` does not actually move the thread; it transfers ownership. After the move, `t1` no longer represents a thread and should not be used to perform operations related to threads.

- Valid but Unspecified State:
  - After moving, `t1` is still a valid `std::thread` object but does not own any thread. It should not be used to call `join()` or `detach()`. Accessing or using `t1` in any way related to threading operations would lead to undefined behavior.

- Joining and Detaching:
  - We must ensure that the thread is either joined or detached before the `std::thread` object goes out of scope to avoid resource leaks or undefined behavior.


#### 2.2.1. Ways to create and manage threads <a name="2.2.1"></a>

> The callable objects are entities that can be called or invoked like functions. They can take arguments and return values, much like regular functions, but they can also encapsulate additional state or behavior. Callable objects can be categorized into several types:

**Types of Callable Objects**

1. Function Pointers:
   - A pointer that points to a function. We can call the function through the pointer.
   ```cpp
   void func(int x) {
       std::cout << "Value: " << x << std::endl;
   }
   void (*funcPtr)(int) = func;
   funcPtr(10); // Calls func with 10
   ```

2. Function Objects (Functors):
   - Objects of classes that overload the `operator()`. They can maintain state and have additional methods.
   ```cpp
   class Functor {
   public:
       void operator()(int x) {
           std::cout << "Functor called with value: " << x << std::endl;
       }
   };
   Functor f;
   f(10); // Calls the functor
   ```

3. Lambda Expressions:
   - Anonymous functions that can capture variables from their enclosing scope.
   ```cpp
   auto lambda = [](int x) {
       std::cout << "Lambda called with value: " << x << std::endl;
   };
   lambda(10); // Calls the lambda
   ```

4. Member Function Pointers:
   - Pointers to member functions of a class. They require an object of the class to be called.
   ```cpp
   class MyClass {
   public:
       void memberFunc(int x) {
           std::cout << "Member function called with value: " << x << std::endl;
       }
   };
   MyClass obj;
   void (MyClass::*memberPtr)(int) = &MyClass::memberFunc;
   (obj.*memberPtr)(10); // Calls memberFunc on obj
   ```

5. Bind Expressions:
   - Created using `std::bind`, these allow us to create callable objects by binding arguments to a function.
   ```cpp
   #include <iostream>
   #include <functional>

   void printSum(int a, int b) {
       std::cout << "Sum: " << a + b << std::endl;
   }

   int main() {
       auto boundFunc = std::bind(printSum, 5, 10);
       boundFunc(); // Calls printSum with 5 and 10
       return 0;
   }
   ```

**Read more about**

- [Function Objects (Functors)](functor.md)
- [Lambda Expressions](lambda.md)
- [Bind Expressions](bind.md)
- [Chrono](chrono.md)

**Example:**

```cpp
#include <iostream>
#include <thread>
using namespace std;

class A {
    public:
        void f(int x)
        {
            cout << "f: " << x << "\n";
            return;
        }

        int operator()(int num) 
        {
            cout << "num: " << num << "\n";
            return 0;
        }
};

void foo(int x)
{
    cout << "foo: " << x << "\n";
    return;
}

int main()
{
    A a;  // Create an instance of class A

    // Start a thread t1 using the instance 'a' of class A
    // This will use A's callable operator() method
    thread t1(a, 1);  // Pass '1' to the callable operator() of 'a'
    t1.join();

    // Start a thread t2 using the reference to instance 'a' of class A
    // This will call the operator() method of 'a'
    thread t2(ref(a), 2);  // Pass '2' to the callable operator() of 'a'
    t2.join();

    // Start a thread t3 using a temporary instance of class A
    // This will call the operator() method of the temporary instance
    thread t3(A(), 3);  // Pass '3' to the callable operator() of a temporary A
    t3.join();

    // Start a thread t4 using a lambda function
    // The lambda function prints and returns the square of the integer passed
    thread t4([](int x) {
        cout << "x * x = " << x * x << "\n";
        return x * x;
    }, 4);  // Pass '4' to the lambda function
    t4.join();

    // Start a thread t5 using a function 'foo'
    thread t5(foo, 5);  // Pass '5' to the function 'foo'
    t5.join();

    // Start a thread t6 using a member function 'f' of class A
    // 'a' is used as the object for which the member function 'f' is called
    thread t6(&A::f, a, 6);  // Call A::f on 'a' with argument '6'
    t6.join();

    // Start a thread t7 using a member function 'f' of class A
    // '&a' is used as the object for which the member function 'f' is called
    thread t7(&A::f, &a, 7);  // Call A::f on 'a' with argument '7' (via pointer)
    t7.join();

    // Start a thread t8 using a moved instance of class A
    // The instance 'a' is moved to the thread, so it should not be used afterwards
    thread t8(move(a), 8);  // Pass '8' to the callable operator() of moved 'a'
    t8.join();

    return 0;
}
```

Output:

```
num: 1
num: 2
num: 3
x * x = 16
foo: 5
f: 6
f: 7
num: 8
```


### 2.3. Race Condition <a name="2.3"></a>

**Critical Section**

> A critical section refers to a part of the code where shared resources are accessed or modified. It is crucial to ensure that only one thread or process can execute within this section at a time to avoid data corruption, inconsistencies, or race conditions.

**Mutable Resource**

> A mutable resource refers to data that can be modified or changed during the execution of a program. Unlike immutable resources, which cannot be altered once created, mutable resources can be updated or changed by different threads, processes, or parts of a program.

**Sharable or Shared Resources**

> A sharable resource refers to entities that can be accessed by multiple threads, processes, or tasks concurrently. Sharable resources include variables, data structures, files, and memory that are shared across different execution contexts.

**Race Condition**

> A race condition is a situation in concurrent programming which occurs when multiple threads or processes access mutable shared resources concurrently, and the final outcome depends on the unpredictable timing or ordering of these accesses. This can lead to inconsistent or incorrect results, as the threads or processes interfere with each other's operations.

```cpp
#include <iostream>
#include <thread>
using namespace std;

int shared_counter = 0;  // Mutable Shared resource

void Increment()
{
    for (int i = 0; i < 100000; ++i)
    {
        ++shared_counter;  // Critical section: modifying shared resource
    }
}

int main()
{
    thread t1(Increment);
    thread t2(Increment);

    t1.join();
    t2.join();

    cout << "Final counter value: " << shared_counter << "\n";

    return 0;
}
```

Output:

```
Final counter value: 122413
```

**Note:**

- Since there is no synchronization mechanism, the `++shared_counter` operation is not atomic. This means that multiple threads can read and modify `shared_counter` simultaneously, leading to race conditions.
- For example, if both threads read the same value of `shared_counter` at the same time, they might both increment it and write back the same value, resulting in lost increments.
- Due to the race condition, the output of the final counter value may vary between runs. It might not always be `200000` (which is the expected value if there were no race conditions). Instead, it could be less than `200000` because of lost updates.
- To avoid the race condition, we should use a synchronization mechanism to ensure that only one thread accesses the critical section at a time.

### 2.4. Mutex <a name="2.4"></a>

> A mutex (short for "mutual exclusion") is a synchronization tool used to manage concurrent access to shared resources, ensuring that only one thread can access the resource at a time.

This prevents race conditions and data corruption when multiple threads attempt to modify or read the same resource simultaneously.

**Key Concepts of Mutex**

- Mutual Exclusion:
   - A mutex provides mutual exclusion by allowing only one thread to own the mutex and access the critical section of code that manipulates shared resources. Other threads attempting to acquire the same mutex will be blocked until the mutex is released.

- Locking and Unlocking:
   - Threads lock a mutex before entering a critical section and unlock it when they leave. If a mutex is already locked by another thread, the requesting thread will be blocked until the mutex becomes available.

- Avoiding Race Conditions:
   - By ensuring that only one thread can execute a critical section at a time, a mutex helps in preventing race conditions where multiple threads might otherwise interfere with each other’s operations.

**Example:**

```cpp
#include <iostream>
#include <thread>
using namespace std;

void Function()
{
    for (int i = 0; i > -5; i--)
        cout << "From t1: " << i << "\n";
}

int main()
{
    thread t1(Function);

    for (int i = 0; i < 5; i++)
        cout << "From main: " << i << "\n";

    t1.join();

    return 0;
}
```

Output:

```
From main: From t1: 0
From t1: -1
From t1: -2
From t1: -3
From t1: -4
0
From main: 1
From main: 2
From main: 3
From main: 4
```

Why are the print statements of the threads overlapping?

The overlapping of print statements from different threads is due to concurrent access to the shared resource, `std::cout`. When multiple threads write to `std::cout` at the same time, their outputs can interleave, leading to the mixed and sometimes jumbled output.

- Thread Interleaving: The `Greet` function and the `main` function are running concurrently because they are executed in different threads. The operating system schedules these threads to run at overlapping times, and since they both use `std::cout` to print messages, the output from both threads can be mixed.
  
- No Synchronization: By default, `std::cout` is not synchronized across different threads. This means that if two threads try to write to `std::cout` at the same time, the output can become interleaved or garbled.

To prevent the interleaving of output, we can synchronize the output between threads. This can be achieved using synchronization primitives such as mutexes.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

mutex mu; // Create a mutex

void shared_print(string msg, int i)
{
    mu.lock();
    cout << msg << i << "\n";
    mu.unlock();
}

void Function()
{
    for (int i = 0; i > -5; i--)
        shared_print("From t1: ", i);
}

int main()
{
    thread t1(Function);

    for (int i = 0; i < 5; i++)
        shared_print("From main: ", i);

    t1.join();

    return 0;
}
```

Output:

```
From main: 0
From main: 1
From t1: 0
From t1: -1
From t1: -2
From t1: -3
From t1: -4
From main: 2
From main: 3
From main: 4
```

Problem solved.


### 2.5 Lock Guard <a name="2.5"></a>

Issue with mutex: What if after putting the lock, the critical section code (i.e., here cout) fails, then we will not be able to unlock.

> A `lock_guard` is a convenient RAII (Resource Acquisition Is Initialization) wrapper around a mutex. It automatically locks the mutex when the `lock_guard` is created and unlocks it when the `lock_guard` goes out of scope. This ensures that the mutex is properly released, even if an exception occurs, which helps to prevent deadlocks and resource leaks.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
using namespace std;

mutex mu; // Create a mutex

void shared_print(string msg, int i)
{
    lock_guard <mutex> guard(mu); 
    cout << msg << i << "\n";
}

void Function()
{
    for (int i = 0; i > -5; i--)
        shared_print("From t1: ", i);
}

int main()
{
    thread t1(Function);

    for (int i = 0; i < 5; i++)
        shared_print("From main: ", i);

    t1.join();

    return 0;
}
```

Output:

```
From main: 0
From main: 1
From t1: 0
From t1: -1
From t1: -2
From t1: -3
From t1: -4
From main: 2
From main: 3
From main: 4
```

When lock guard goes out of scope it will relase its lock, even if an exception occurs.
Here in the above case, cout is a globally declared so other threads might still access it.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <fstream>
using namespace std;

class LogFile {
    private:
        mutex mu;
        ofstream f;
    public:
        LogFile()
        {
            f.open("logs.txt");
        }
        ~LogFile()
        {
            f.close();
        }
        void shared_print(string msg, int i)
        {
            lock_guard <mutex> guard(mu);
            f << msg << i << "\n";
        }
};

void Function(LogFile &log)
{
    for (int i = 0; i > -5; i--)
        log.shared_print("From t1: ", i);
}

int main()
{
    LogFile log;
    thread t1(Function, ref(log));

    for (int i = 0; i < 5; i++)
        log.shared_print("From main: ", i);

    t1.join();

    return 0;
}
```

Output:

```
From main: 0
From main: 1
From main: 2
From main: 3
From t1: 0
From t1: -1
From main: 4
From t1: -2
From t1: -3
From t1: -4
```

Note:
- Never return f to the outside world.
- Never paas f as an argument to user provided function.

**Key Features of lock_guard**

1. Automatic Locking and Unlocking:
   - When a `lock_guard` object is created, it locks the specified mutex.
   - When the `lock_guard` object is destroyed (typically when it goes out of scope), it automatically unlocks the mutex.

2. Simple Interface: 
   - The `lock_guard` is lightweight and provides a simple interface for managing mutex locks.

3. Non-Copyable:
   - `lock_guard` cannot be copied, which prevents multiple `lock_guard` instances from trying to manage the same mutex.


### 2.6 Deadlock <a name="2.6"></a>

> A deadlock is a situation in a multithreaded or multiprocess system where two or more threads (or processes) are unable to proceed because each is waiting for the other to release a resource. In essence, it's a standstill where no progress can be made.

**Key Characteristics of Deadlocks**

1. Mutual Exclusion: At least one resource must be held in a non-shareable mode. Only one thread can use the resource at any given time.

2. Hold and Wait: A thread holding at least one resource is waiting to acquire additional resources that are currently being held by other threads.

3. No Preemption: Resources cannot be forcibly taken from a thread holding them; they must be voluntarily released.

4. Circular Wait: There exists a set of threads where each thread is waiting for a resource that the next thread in the set holds, forming a cycle.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <fstream>
using namespace std;

class LogFile {
    private:
        mutex mu1;
        mutex mu2;
        ofstream f;
    public:
        LogFile()
        {
            f.open("logs.txt");
        }
        ~LogFile()
        {
            f.close();
        }
        void shared_print1(string msg, int i)
        {
            lock_guard <mutex> guard1(mu1);
            lock_guard <mutex> guard2(mu2);
            f << msg << i << "\n";
        }
        void shared_print2(string msg, int i)
        {
            lock_guard <mutex> guard2(mu2);
            lock_guard <mutex> guard1(mu1);
            f << msg << i << "\n";
        }
};

void Function(LogFile &log)
{
    for (int i = 0; i > -10; i--)
        log.shared_print1("From t1: ", i);
}

int main()
{
    LogFile log;
    thread t1(Function, ref(log));

    for (int i = 0; i < 10; i++)
        log.shared_print2("From main: ", i);

    t1.join();

    return 0;
}
```

Output:

```
From main: 0
From main: 1
From main: 2
From t1: 0
From t1: -1
From t1: -2
From t1: -3
```

Thread t1 and main thread are running cuncurrently, t1 thread will accquire lock on mutex mu1 and main thread will accquire lock on mutex mu2. Now thread t1 will wait to accquire lock on mutex mu2 and thread main will wait to accquire lock on mu1 thread. Both threads waits indefinitely and goes in deadlock.

**Deadlock Prevention and Resolution Strategies**

1. Resource Allocation Strategies:
   - Avoid Hold and Wait: Require threads to request all needed resources at once.
   - Request Resources in a Specific Order: Ensure all threads request resources in a predetermined order to prevent circular wait.

2. Detection and Recovery:
   - Implement algorithms to detect deadlocks and take action, such as terminating one of the threads or preempting resources.

3. Timeouts:
   - Implement timeouts for resource requests, causing a thread to back off and retry after a period if it cannot acquire a resource.

4. Using Higher-Level Concurrency Primitives:
   - Instead of using low-level mutexes, consider higher-level abstractions (like condition variables or thread-safe queues) that can help reduce the likelihood of deadlocks.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <fstream>
using namespace std;

class LogFile {
    private:
        mutex mu1;
        mutex mu2;
        ofstream f;
    public:
        LogFile()
        {
            f.open("logs.txt");
        }
        ~LogFile()
        {
            f.close();
        }
        void shared_print1(string msg, int i)
        {
            lock_guard <mutex> guard1(mu1);
            lock_guard <mutex> guard2(mu2);
            f << msg << i << "\n";
        }
        void shared_print2(string msg, int i)
        {
            lock_guard <mutex> guard1(mu1);
            lock_guard <mutex> guard2(mu2);
            f << msg << i << "\n";
        }
};

void Function(LogFile &log)
{
    for (int i = 0; i > -10; i--)
        log.shared_print1("From t1: ", i);
}

int main()
{
    LogFile log;
    thread t1(Function, ref(log));

    for (int i = 0; i < 10; i++)
        log.shared_print2("From main: ", i);

    t1.join();

    return 0;
}
```

Output:

```
From main: 0
From main: 1
From t1: 0
From main: 2
From main: 3
From main: 4
From main: 5
From main: 6
From main: 7
From main: 8
From main: 9
From t1: -1
From t1: -2
From t1: -3
From t1: -4
From t1: -5
From t1: -6
From t1: -7
From t1: -8
From t1: -9
```

Here, the threads request resources in a predetermined order to prevent circular wait, so no deadlock.


### 2.7 Lock <a name="2.7"></a>

> `std::lock` is a function that allows us to lock multiple mutexes simultaneously. It is designed to prevent deadlocks that can occur when threads attempt to acquire multiple mutexes in different orders.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <fstream>
using namespace std;

class LogFile {
    private:
        mutex mu1;
        mutex mu2;
        ofstream f;
    public:
        LogFile()
        {
            f.open("logs.txt");
        }
        ~LogFile()
        {
            f.close();
        }
        void shared_print1(string msg, int i)
        {
            lock(mu1, mu2);
            lock_guard <mutex> guard1(mu1, adopt_lock);
            lock_guard <mutex> guard2(mu2, adopt_lock);
            f << msg << i << "\n";
        }
        void shared_print2(string msg, int i)
        {
            lock(mu1, mu2);
            lock_guard <mutex> guard1(mu1, adopt_lock);
            lock_guard <mutex> guard2(mu2, adopt_lock);
            f << msg << i << "\n";
        }
};

void Function(LogFile &log)
{
    for (int i = 0; i > -10; i--)
        log.shared_print1("From t1: ", i);
}

int main()
{
    LogFile log;
    thread t1(Function, ref(log));

    for (int i = 0; i < 10; i++)
        log.shared_print2("From main: ", i);

    t1.join();

    return 0;
}
```

Output:

```
From main: 0
From main: 1
From t1: 0
From t1: -1
From t1: -2
From t1: -3
From t1: -4
From t1: -5
From t1: -6
From t1: -7
From t1: -8
From t1: -9
From main: 2
From main: 3
From main: 4
From main: 5
From main: 6
From main: 7
From main: 8
From main: 9
```

```cpp
lock(mu1, mu2);
```

- This line locks both `mu1` and `mu2` at the same time.
- Using `std::lock` prevents deadlocks by ensuring that both mutexes are locked without risk of one thread holding one mutex while waiting for another.
  
```cpp
lock_guard <mutex> guard1(mu1, std::adopt_lock);
lock_guard <mutex> guard2(mu2, std::adopt_lock);
```

- These lines create two `std::lock_guard` objects, `guard1` and `guard2`.
- The `std::adopt_lock` argument indicates that the mutexes are already locked (by `std::lock`), and `lock_guard` should take ownership of them without locking them again.
- The `lock_guard` ensures that the mutexes will be automatically released when the guard objects go out of scope, providing exception safety and preventing resource leaks.


### 2.8 Unique Lock <a name="2.8"></a>

> The `unique_lock` is a synchronization mechanism that manages a mutex for thread safety. It is part of the `<mutex>` header and is designed to provide a flexible way to lock and unlock a mutex.

**Key Features of `unique_lock`**

1. Automatic Resource Management: 
   - `unique_lock` automatically locks a mutex when it is created and unlocks it when the `unique_lock` goes out of scope. This helps prevent resource leaks and ensures proper cleanup.

2. Lock Ownership: 
   - It maintains exclusive ownership of the mutex, meaning only one `unique_lock` can own a mutex at a time. This enforces thread safety.

3. Deferred Locking: 
   - We can create a `unique_lock` without immediately locking the associated mutex. We can use the `lock()` member function later.

4. Unlocking: 
   - We can manually unlock the mutex before the `unique_lock` goes out of scope by calling the `unlock()` method, allowing for more granular control.

5. Move Semantics: 
   - `unique_lock` supports move semantics, allowing us to transfer ownership of the lock from one `unique_lock` instance to another. This is useful in situations like returning a lock from a function.

6. Try Locking: 
   - It provides a `try_lock()` function that attempts to lock the mutex without blocking. If the mutex is already locked by another thread, `try_lock()` returns false.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <fstream>
using namespace std;

class LogFile {
    private:
        mutex mu;
        ofstream f;
    public:
        LogFile()
        {
            f.open("logs.txt");
        }
        ~LogFile()
        {
            f.close();
        }
        void shared_print(string msg, int i)
        {
            unique_lock <mutex> locker(mu);
            f << msg << i << "\n";
        }
};

void Function(LogFile &log)
{
    for (int i = 0; i > -10; i--)
        log.shared_print("From t1: ", i);
}

int main()
{
    LogFile log;
    thread t1(Function, ref(log));

    for (int i = 0; i < 10; i++)
        log.shared_print("From main: ", i);

    t1.join();

    return 0;
}
```

Output:

```
From main: 0
From t1: 0
From t1: -1
From t1: -2
From t1: -3
From t1: -4
From t1: -5
From t1: -6
From t1: -7
From t1: -8
From t1: -9
From main: 1
From main: 2
From main: 3
From main: 4
From main: 5
From main: 6
From main: 7
From main: 8
From main: 9
```

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <fstream>
using namespace std;

class LogFile {
    private:
        mutex mu;
        ofstream f;
    public:
        LogFile()
        {
            f.open("logs.txt");
        }
        ~LogFile()
        {
            f.close();
        }
        void shared_print(string msg, int i)
        {
            unique_lock <mutex> locker(mu, defer_lock);

            // do something

            locker.lock();
            f << msg << i << "\n";
            locker.unlock();

            // do something

            // Acquire the lock again for some work
            locker.lock();
            // do something
            locker.unlock();

            // Transfer ownership
            unique_lock <mutex> locker2 = move(locker);

            // do something
        }
};

void Function(LogFile &log)
{
    for (int i = 0; i > -10; i--)
        log.shared_print("From t1: ", i);
}

int main()
{
    LogFile log;
    thread t1(Function, ref(log));

    for (int i = 0; i < 10; i++)
        log.shared_print("From main: ", i);

    t1.join();

    return 0;
}
```

Output:

```
From main: 0
From main: 1
From main: 2
From t1: 0  
From t1: -1 
From t1: -2 
From t1: -3 
From t1: -4 
From t1: -5 
From t1: -6 
From t1: -7 
From t1: -8 
From t1: -9 
From main: 3
From main: 4
From main: 5
From main: 6
From main: 7
From main: 8
From main: 9
```

```cpp
   unique_lock <mutex> locker(mu, defer_lock);
```

- This line creates a `unique_lock` called `locker` that is associated with the mutex `mu`. The `defer_lock` parameter means the mutex is not locked immediately; it allows us to control when the locking happens.


```cpp
   locker.lock();
   f << msg << i << "\n";
   locker.unlock();
```

   - Locking: The mutex is explicitly locked before accessing the shared resource (the log file).
   - Logging: Writes the message and the integer to the file.
   - Unlocking: The mutex is unlocked immediately after the logging is done, allowing other threads to acquire the lock.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <fstream>
using namespace std;

class LogFile {
    private:
        mutex mu;
        mutex mu_open;
        ofstream f;
    public:
        LogFile()
        {
        }
        ~LogFile()
        {
            f.close();
        }
        void shared_print(string msg, int i)
        {
            unique_lock <mutex> locker2(mu_open);
            if (!f.is_open())
                f.open("logs.txt");

            unique_lock <mutex> locker(mu);
            f << msg << i << "\n";
        }
};

void Function(LogFile &log)
{
    for (int i = 0; i > -10; i--)
        log.shared_print("From t1: ", i);
}

int main()
{
    LogFile log;
    thread t1(Function, ref(log));

    for (int i = 0; i < 10; i++)
        log.shared_print("From main: ", i);

    t1.join();

    return 0;
}
```

Output:

```
From main: 0
From main: 1
From main: 2
From main: 3
From main: 4
From main: 5
From main: 6
From main: 7
From t1: 0
From t1: -1
From t1: -2
From t1: -3
From t1: -4
From t1: -5
From t1: -6
From t1: -7
From t1: -8
From t1: -9
From main: 8
From main: 9
```

What if the shared_print was never called, then why are we even opening the file?
We will only open the file when the shared_print was actually called. But with this, now every thread will try to open the file. Even though the file will be only opened once but still the treads will try to acquire locks unnecessarly.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <fstream>
using namespace std;

class LogFile {
    private:
        mutex mu;
        once_flag flag;
        ofstream f;
    public:
        LogFile()
        {
        }
        ~LogFile()
        {
            f.close();
        }
        void shared_print(string msg, int i)
        {
            call_once(flag, [&](){ f.open("logs.txt"); });

            unique_lock <mutex> locker(mu);
            f << msg << i << "\n";
        }
};

void Function(LogFile &log)
{
    for (int i = 0; i > -10; i--)
        log.shared_print("From t1: ", i);
}

int main()
{
    LogFile log;
    thread t1(Function, ref(log));

    for (int i = 0; i < 10; i++)
        log.shared_print("From main: ", i);

    t1.join();

    return 0;
}
```

Output:

```
From main: 0
From main: 1
From t1: 0
From t1: -1
From t1: -2
From t1: -3
From t1: -4
From t1: -5
From main: 2
From main: 3
From main: 4
From main: 5
From main: 6
From main: 7
From main: 8
From t1: -6
From t1: -7
From t1: -8
From t1: -9
From main: 9
```

```cpp
   void shared_print(string msg, int i)
   {
       call_once(flag, [&]() { f.open("logs.txt"); });

       unique_lock<mutex> locker(mu);
       f << msg << i << "\n";
   }
```

- Opening the File:
     - `call_once(flag, [&]() { f.open("logs.txt"); });`: This line ensures that the log file is opened only once, regardless of how many times `shared_print` is called from different threads. The lambda function is executed only the first time `call_once` is invoked with the `flag`.
   
- Locking the Mutex:
     - `unique_lock<mutex> locker(mu);`: Locks the mutex to protect the critical section where the log file is written to.
   
- Logging: Writes the message and integer to the log file. The lock is automatically released when the `locker` goes out of scope.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <deque>
using namespace std;

mutex mu;
deque <int> q;

void Producer()
{
    int count = 10;
    while (count > 0)
    {
        unique_lock <mutex> locker(mu);
        q.push_front(count);
        locker.unlock();
        this_thread::sleep_for(chrono::seconds(1));
        count--;
    }
}

void Consumer()
{
    int data = 0;
    while (data != 1)
    {
        unique_lock <mutex> locker(mu);
        if (!q.empty())
        {
            data = q.back();
            q.pop_back();
            locker.unlock();
            cout << "t2 got a value from t1: " << data << endl;
        }
        else
        {
            locker.unlock();
        }
    }
}

int main()
{
    thread t1(Producer);
    thread t2(Consumer);
    t1.join();
    t2.join();

    return 0;
}
```

Output:

```
t2 got a value from t1: 10
t2 got a value from t1: 9
t2 got a value from t1: 8
t2 got a value from t1: 7
t2 got a value from t1: 6
t2 got a value from t1: 5
t2 got a value from t1: 4
t2 got a value from t1: 3
t2 got a value from t1: 2
t2 got a value from t1: 1
```

Here, the consumer keeps on checking whether there is data in the queue to be consumed even though the producer
has not produced new data.

We can make the Conusmer wait for 10 ms before checking again if there is data available to be consumed. In this
way we can reduce the frequency by which it checks whether there is any data to be consumed. But this is also an 
inefficient method.

```cpp
void Consumer()
{
    int data = 0;
    while (data != 1)
    {
        unique_lock <mutex> locker(mu);
        if (!q.empty())
        {
            data = q.back();
            q.pop_back();
            locker.unlock();
            cout << "t2 got a value from t1: " << data << endl;
        }
        else
        {
            locker.unlock();
            this_thread::sleep_for(chrono::milliseconds(10));
        }
    }
}
```


### 2.9 Condition Variable <a name="2.9"></a>

> A condition variable is a synchronization tool that enables threads to wait until a certain condition is met. It's part of the `<condition_variable>` header and is often used in conjunction with a mutex. Condition variables allow threads to communicate with each other about the state of shared data, facilitating coordination between producer and consumer threads or managing resource availability.

**Key Features of Condition Variables**

1. Blocking and Signaling:
   - Threads can block on a condition variable until they are notified that a condition they are waiting for has changed. This is done using `wait()`, which releases the associated mutex and puts the thread to sleep.
   - Other threads can signal this condition using `notify_one()` or `notify_all()`, waking up one or all threads waiting on the condition variable.

2. Mutex Integration:
   - Condition variables must be used with a mutex. Before a thread waits on a condition variable, it should hold the mutex, and when it wakes up, it must re-acquire the mutex to check the condition again.

3. Efficiency:
   - Condition variables help avoid busy waiting (where a thread continuously checks a condition), making them more efficient for inter-thread communication.

**Example:**

```cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>
#include <deque>
using namespace std;

mutex mu;
condition_variable cv;
deque <int> q;

void Producer()
{
    int count = 10;
    while (count > 0)
    {
        unique_lock <mutex> locker(mu);
        q.push_front(count);
        locker.unlock();
        cv.notify_one(); // Notify one waiting thread, if there is one
        this_thread::sleep_for(chrono::seconds(1));
        count--;
    }
}

void Consumer()
{
    int data = 0;
    while (data != 1)
    {
        unique_lock <mutex> locker(mu);
        cv.wait(locker, [](){ return !q.empty(); });
        data = q.back();
        q.pop_back();
        locker.unlock();
        cout << "t2 got a value from t1: " << data << endl;
    }
}

int main()
{
    thread t1(Producer);
    thread t2(Consumer);
    t1.join();
    t2.join();

    return 0;
}
```

Output:

```
t2 got a value from t1: 10
t2 got a value from t1: 9
t2 got a value from t1: 8
t2 got a value from t1: 7
t2 got a value from t1: 6
t2 got a value from t1: 5
t2 got a value from t1: 4
t2 got a value from t1: 3
t2 got a value from t1: 2
t2 got a value from t1: 1
```

```cpp
   void Producer()
   {
       int count = 10;
       while (count > 0)
       {
           unique_lock<mutex> locker(mu);
           q.push_front(count);
           locker.unlock();
           cv.notify_one(); // Notify one waiting thread, if there is one
           this_thread::sleep_for(chrono::seconds(1));
           count--;
       }
   }
```

- This function produces integers from `10` to `1` and pushes them to the front of the queue.
- It locks the mutex to ensure exclusive access to the queue while modifying it, then unlocks it immediately after the push.
- It uses `cv.notify_one()` to wake up a waiting consumer thread after adding a new item.
- The producer sleeps for 1 second between producing items to simulate some delay.


```cpp
   void Consumer()
   {
       int data = 0;
       while (data != 1)
       {
           unique_lock<mutex> locker(mu);
           cv.wait(locker, []() { return !q.empty(); }); // Wait until the queue is not empty
           data = q.back();
           q.pop_back();
           locker.unlock();
           cout << "t2 got a value from t1: " << data << endl; // Corrected output
       }
   }
```

- The consumer continuously checks for new items in the queue.
- It uses `cv.wait(locker, ...)` to wait until the queue is not empty, which releases the lock while waiting and reacquires it once notified.
- After getting an item from the back of the queue, it unlocks the mutex.
- Finally, it prints the consumed value.

Do we really need to check the condition for consumer, isn't the producer going to notify when there is data to be consumed?

While it's true that the producer will notify the consumer when new data is available, it's still essential to check the condition in the consumer for several reasons:

1. Spurious Wakeups:
    - A spurious wakeup is a phenomenon in multithreading where a thread that is waiting on a condition variable is awakened without an explicit notification (like `notify_one()` or `notify_all()`). This can happen due to various reasons, such as the operating system's scheduling or internal implementation details of the threading library.
   - A consumer thread can wake up without a notification due to spurious wakeups. Checking the condition ensures that the consumer only proceeds if there is actual data to consume.

2. Multiple Notifiers:
   - If there are multiple producers and consumers, it's possible that a notification may come from a different producer or that the producer may have sent multiple notifications in rapid succession. Without checking the condition, the consumer might wake up but find that the queue is still empty.

3. Race Conditions:
   - Even if the producer notifies the consumer, there's a chance that between the notification and the consumer acquiring the lock, the queue might have been modified by another thread. Checking the condition protects against this scenario.

4. Graceful Termination:
   - If we want the consumer to exit based on certain conditions (like when the producer has finished producing), checking the condition allows us to implement that logic safely. 




## 3. Concurrency References <a name="3"></a>

### 3.1 Future <a name="3.1"></a>

> A `std::future` is a mechanism that allows us to obtain a value that will be available at some point in the future, typically as the result of an asynchronous operation. It is part of the C++ Standard Library (introduced in C++11) and is primarily used for managing asynchronous tasks in a thread-safe manner.

**Key Features of `std::future`**

1. Asynchronous Result Retrieval:
   - A `std::future` object is linked to a computation that runs asynchronously, allowing us to retrieve the result once it's ready.
   
2. Blocking Behavior:
   - When we call `get()` on a `std::future`, it blocks the calling thread until the result is available. If the computation has already completed, `get()` returns immediately.

3. Exception Handling:
   - If the asynchronous operation encounters an exception, calling `get()` on the `std::future` will rethrow that exception, allowing us to handle errors gracefully.

4. Shared Ownership:
   - A `std::future` is designed to provide a single result. Once `get()` is called, the future is no longer valid, and further calls will result in an error.

**Example:**

```cpp
#include <iostream>
#include <future>
using namespace std;

int factorial(int N)
{
    int res = 1;
    for (int i = N; i > 1; i--)
        res *= i;

    return res;
}

int main()
{
    future <int> fu = async(factorial, 4);

    int x = fu.get();

    cout << "x: " << x << "\n";

    return 0;
}
```

Output:

```
x: 24
```

**Example:**

```cpp
int main()
{
    future <int> fu = async(launch::deferred, factorial, 4);

    int x = fu.get();

    cout << "x: " << x << "\n";

    int y = fu.get();

    cout << "y: " << x << "\n";

    return 0;
}
```

Output:

```
x: 24
terminate called after throwing an instance of 'std::future_error'
  what():  std::future_error: No associated state
```

- The second call to `fu.get()` fails because the future has already been consumed. A `std::future` object can only provide its result once. After the first call, the state of the future is no longer valid for retrieving the value, leading to the `std::future_error` exception.


### 3.2 Async <a name="3.2"></a>

> The `std::async` is a function that allows us to run a task asynchronously, which means it can execute in a separate thread without blocking the calling thread. It is part of the C++ Standard Library, introduced in C++11, and is designed to facilitate concurrent programming.

**Key Features of `std::async`**

1. Asynchronous Execution:
   - When we call `std::async`, it can execute a callable (like a function or lambda) in a separate thread, enabling our program to perform other tasks while waiting for the result.

2. Launch Policies:
   - `std::async` accepts a launch policy as its first argument, which determines how the task is executed:
     - `std::launch::async`: Ensures that the function runs in a new thread.
     - `std::launch::deferred`: Delays execution until `get()` is called on the returned future. In this case, the function runs in the context of the thread that calls `get()`.
     - We can also specify a combination of both launch policies.

3. Returns a `std::future`:
   - `std::async` returns a `std::future` object that represents the result of the asynchronous operation. We can call `get()` on the future to retrieve the result once the task is complete.

4. Exception Propagation:
   - If the asynchronous task throws an exception, calling `get()` on the future will rethrow that exception, allowing us to handle it appropriately.

```cpp
int main()
{
    future <int> fu = async(launch::deferred, factorial, 4);

    int x = fu.get();

    cout << "x: " << x << "\n";

    return 0;
}
```

**Key Points About `std::async` with Combined Launch Policies**

1. Combined Launch Policies:
   - The expression `launch::async | launch::deferred` indicates that the function can either be executed in a new thread or deferred until `get()` is called.
   - However, this combination does not guarantee that the function will run in a new thread. The actual behavior depends on the implementation and the state of the system when `get()` is called.

2. Behavior:
   - If the system is capable, it may run the task in a new thread (because of `launch::async`).
   - If `launch::deferred` is chosen, the task will not run until we call `get()`, and it will run in the context of the calling thread at that time.

3. Retrieving the Result:
   - Calling `fu.get()` will block until the result is available, regardless of whether it was run asynchronously or deferred.

```cpp
int main()
{
    future <int> fu = async(launch::async | launch::deferred, factorial, 4);

    int x = fu.get();

    cout << "x: " << x << "\n";

    return 0;
}
```


### 3.3 Promise <a name="3.3"></a>

> A `std::promise` is a synchronization tool that provides a way to set a value or an exception that can be retrieved asynchronously via a `std::future`. It is part of the C++ Standard Library, introduced in C++11, and is designed to facilitate communication between threads.

**Key Features of `std::promise`**

1. Setting Values:
   - A `std::promise` allows us to set a value that can later be accessed through a linked `std::future`. We do this using the `set_value()` method.

2. Exception Handling:
   - If an error occurs in the asynchronous operation, we can set an exception using the `set_exception()` method. When the `std::future` retrieves the value, it will throw the exception.

3. One-Time Use:
   - A `std::promise` is designed to be used once. After we set a value or an exception, the associated `std::future` can be used to retrieve that result, but the promise itself cannot be reused.

4. Thread Safety:
   - `std::promise` is thread-safe, meaning that it can be safely accessed from multiple threads without additional synchronization.

**Example:**

```cpp
#include <iostream>
#include <future>
#include <thread>
using namespace std;

int factorial(future <int> &f)
{
    int res = 1;
    int N = f.get(); // Get the value from the future
    for (int i = N; i > 1; i--)
        res *= i;

    return res;
}

int main()
{
    promise <int> p;                    // Create a promise
    future <int> f = p.get_future();    // Get the associated future

    future <int> fu = async(launch::async, factorial, ref(f));  // Launch factorial asynchronously

    // do something
    this_thread::sleep_for(chrono::milliseconds(10));

    p.set_value(4);                     // Set the value for the promise

    int x = fu.get();                   // Retrieve the result from the future

    cout << "x: " << x << "\n";

    return 0;
}
```

Output:

```
x: 24
```

**Execution**

1. Creating a Promise and Future:
   - We create a `std::promise<int>` object `p`, which will be used to set an integer value.
   - `future<int> f = p.get_future();` retrieves a `std::future<int>` associated with the promise. This future will be used to get the value in the `factorial` function.

2. Launching the Asynchronous Task:
   - `future<int> fu = async(launch::async, factorial, ref(f));` starts the `factorial` function asynchronously, passing a reference to the future `f`.
   - The `factorial` function will wait for `p.set_value(4)` to be called before it can proceed.

3. Simulating Work:
   - `this_thread::sleep_for(chrono::milliseconds(10));` simulates doing some other work while the factorial calculation is being prepared.

4. Setting the Value:
   - `p.set_value(4);` sets the value `4` in the promise. This unblocks the `f.get()` call in the `factorial` function, allowing it to proceed with the calculation.

5. Getting the Result:
   - `int x = fu.get();` retrieves the result of the factorial calculation, which has been computed asynchronously. Since `4! = 24`, the output will be `x: 24`.


What if we don't keep our promise and don't set a value for the promise? Will it keep waiting or throw an exception?

```cpp
int main()
{
    promise <int> p;
    future <int> f = p.get_future();

    future <int> fu = async(launch::async, factorial, ref(f));

    // No value is set for the promise
    return 0; // Exiting main without fulfilling the promise
}
```

- Since `main()` is returning without setting a value for the promise, the `promise` object `p` will go out of scope and be destroyed.
- As a result, when `f.get()` is called in the `factorial` function, it will throw a `std::future_error` with `std::future_errc::broken_promise` because the promise was destroyed without being fulfilled.
- So if we can't keep our promise then we should set an excepion.

```cpp
int main()
{
    promise <int> p;
    future <int> f = p.get_future();

    future <int> fu = async(launch::async, factorial, ref(f));

    p.set_exception(make_exception_ptr(runtime_error("Sorry, I couldn't keep my promise")));

    return 0;
}
```

**Example:**

```cpp
#include <iostream>
#include <future>
#include <thread>
using namespace std;

int factorial(future <int> &f)
{
    int res = 1;
    int N = f.get();
    for (int i = N; i > 1; i--)
        res *= i;

    return res;
}

int main()
{
    promise <int> p;
    future <int> f = p.get_future();

    future <int> fu = async(launch::async, factorial, ref(f));
    future <int> f1 = async(launch::async, factorial, ref(f));

    p.set_value(4);

    int x = fu.get();
    int y = f1.get();

    cout << "x: " << x << "\n";
    cout << "y: " << y << "\n";
}
```

Output:

```
terminate called after throwing an instance of 'std::future_error'
  what():  std::future_error: No associated state
```

Here, we are creating multiple future objects (fu, f1) from the same future (f) and passing this future to different tasks. This is problematic because a future can only be used once to retrieve a value (i.e., calling get()); after that, it becomes invalid. Each future should be used only once.


### 3.4 Shared Future <a name="3.4"></a>

> A `std::shared_future` is a special type of future that allows multiple threads to access the same result from an asynchronous operation. Unlike a regular `std::future`, which can only be accessed once, a `std::shared_future` can be copied and shared among multiple threads. This makes it useful for scenarios where multiple consumers need to retrieve the same result.

**Key Features of `std::shared_future`**

1. Multiple Access:
   - A `std::shared_future` can be shared among multiple threads, allowing each thread to call `get()` on it without affecting other threads' ability to retrieve the result.

2. Thread Safety:
   - It is safe to use `std::shared_future` across multiple threads, ensuring that the value can be retrieved concurrently.

3. Single Result:
   - The underlying promise can only be fulfilled once. Once the result is set, all copies of the `shared_future` will see the same result.

4. Exception Propagation:
   - If the underlying promise is fulfilled with an exception, all calls to `get()` on the `shared_future` will throw that exception.

**Example:**

```cpp
#include <iostream>
#include <future>
#include <thread>
using namespace std;

int factorial(shared_future <int> f)
{
    int res = 1;
    int N = f.get();
    for (int i = N; i > 1; i--)
        res *= i;

    return res;
}

int main()
{
    promise <int> p;
    future <int> f = p.get_future();
    shared_future <int> sf = f.share();

    future <int> fu = async(launch::async, factorial, sf);
    future <int> f1 = async(launch::async, factorial, sf);

    p.set_value(4);

    int x = fu.get();
    int y = f1.get();

    cout << "x: " << x << "\n";
    cout << "y: " << y << "\n";
}
```

Output:

```
x: 24
y: 24
```


### 3.5 Packaged Task <a name="3.5"></a>

> A "packaged task" refers to a way to encapsulate a callable object (like a function or a lambda) along with its associated data so that it can be executed asynchronously. This is commonly used in conjunction with `std::future` and `std::async` from the C++ Standard Library, enabling us to run tasks in a separate thread and retrieve the result later.

**Key Concepts of `std::packaged_task`**

1. `std::packaged_task`: A class template that wraps a callable object. It allows us to call the function and retrieve its result asynchronously.

2. `std::future`: When we create a `std::packaged_task`, it generates a `std::future` object that can be used to get the result of the task once it’s finished.

**Example:**

```cpp
#include <iostream>
#include <future>
#include <thread>
using namespace std;

// Function to be executed asynchronously
int compute(int x, int y)
{
    this_thread::sleep_for(chrono::seconds(2)); // Simulate long computation
    return x + y;
}

int main()
{
    // Create a packaged_task with a callable function
    // Here, int(int, int) specifies that the callable object compute returns an int and takes two int parameters
    packaged_task <int(int, int)> task(compute);

    // Retrieve the associated future
    future <int> result = task.get_future();

    // Launch the task asynchronously using a thread
    thread t(std::move(task), 5, 3);

    // Do other work while waiting for the result
    cout << "Doing other work while computation is in progress...\n";

    // Wait for the result from the task
    t.join(); // Ensure the thread finishes

    // Get and print the result
    cout << "Result: " << result.get() << "\n";

    return 0;
}
```

Output:

```
Doing other work while computation is in progress...
Result: 8
```
```cpp
   packaged_task <int(int, int)> task(compute);
   future <int> result = task.get_future();
```

- A `packaged_task` is created, wrapping the `compute` function. The type signature `int(int, int)` indicates that it takes two integers and returns an integer.
- `get_future()` retrieves a `future<int>` object associated with the task, which will hold the result once the task is executed.

 ```cpp
   thread t(std::move(task), 5, 3);
```
- A new thread `t` is created to execute the `packaged_task`. `std::move(task)` transfers ownership of the task to the thread. The arguments `5` and `3` are passed to the `compute` function.


```cpp
   t.join(); // Ensure the thread finishes
   cout << "Result: " << result.get() << "\n";
 ```
- The `join()` call blocks the main thread until the thread `t` finishes execution.
- Once the thread has finished, `result.get()` is called to retrieve the computed result from the `packaged_task`.

**Example:**

```cpp
#include <iostream>
#include <future>
#include <deque>
#include <functional>
#include <thread>
#include <mutex>
#include <condition_variable>
using namespace std;

mutex mu;                           // Mutex for synchronizing access to task_q
condition_variable cv;              // Condition variable for notifying thread_1
deque <packaged_task<int()>> task_q; // Queue to hold packaged_tasks

// Function to compute factorial of N
int factorial(int N)
{
    int res = 1;
    for (int i = N; i > 1; i--)
        res *= i;
    return res;
}

// Function that runs in a separate thread
void thread_1()
{
    packaged_task <int()> pt;  // Task object to hold the packaged_task

    {
        unique_lock <mutex> locker(mu);   // Lock the mutex to safely access task_q
        cv.wait(locker, []() { return !task_q.empty(); }); // Wait until task_q is not empty
        pt = move(task_q.front());  // Move the front task from the queue to pt
        task_q.pop_front();         // Remove the task from the queue
    }

    pt(); // Execute the task
}

int main()
{
    thread th(thread_1);  // Start thread_1 in a new thread

    // Create a packaged_task to compute factorial of 4
    packaged_task <int()> pt(bind(factorial, 4));

    // Get future associated with the packaged_task
    future <int> fu = pt.get_future();

    {
        unique_lock <mutex> locker(mu); // Lock the mutex to safely access task_q
        task_q.push_back(move(pt));     // Add the task to the queue
    }

    cv.notify_one(); // Notify thread_1 that a task is available

    cout << "Result: " << fu.get() << "\n";

    th.join(); // Wait for thread_1 to finish
    return 0;
}
```

Output:

```
Result: 24
```


### 3.6 Decide which to use: threads or task-based parallelism <a name="3.6"></a>

When deciding between using threads and task-based parallelism with constructs like `std::async`, `std::future`, and `std::promise`, consider the following factors:

**When to Use Threads**

1. Fine-Grained Control: If we need detailed control over thread management, such as setting thread priorities, managing lifetimes, or handling complex synchronization, manual thread handling may be preferable.

2. Long-Lived Tasks: For tasks that run for an extended period and require continuous execution, manually managing threads might be better.

3. High Customization: When our application requires a specific threading model or when we need to handle threads in a particular way (e.g., thread pools, specific scheduling).

4. Performance: In scenarios where we have a well-defined number of threads and a clear understanding of the workload distribution, threads can provide optimized performance.

**When to Use Tasks (e.g., `std::async`, `std::future`, `std::promise`)**

1. Simplicity and Readability: Tasks abstract away many complexities of manual thread management, making our code easier to read and maintain.

2. Return Values: If we need to retrieve results from concurrent operations, tasks simplify this with `std::future`, allowing us to easily obtain the return values.

3. Short-Lived or Frequent Tasks: For many short-lived tasks that might start and finish frequently, task-based parallelism is generally more efficient and reduces the overhead of managing threads.

4. Resource Management: The `std::async` mechanism manages threads for us, leveraging system resources more effectively and providing a cleaner way to handle thread pooling.

5. Error Handling: Tasks can provide easier mechanisms for handling exceptions and errors that occur during execution, allowing for more straightforward error propagation.




## 4. References <a name="4"></a>

1. [Concurrent Programming with C++ 11 by Bo Qian](https://www.youtube.com/playlist?list=PL5jc9xFGsL8E12so1wlMS0r0hTQoJL74M)
2. [Advanced C++ Crash Course (Threading and Concurrency)](https://github.com/methylDragon/coding-notes/blob/master/C%2B%2B/07%20C%2B%2B%20-%20Threading%20and%20Concurrency.md)