# SCALABLE MULTITHREADED TASK MANAGER WITH THREAD POOL
## Operating Systems — College Project Report

**Technology:** Python 3.x · Tkinter · Threading · Queue · Matplotlib  
**Version:** 2.0 (Full GUI — 7 Advanced Features)  
**Subject:** Operating Systems  
**Submitted To:** Department of Computer Science  

---

## 1. Project Introduction

In modern computing, an Operating System (OS) is responsible for managing multiple tasks simultaneously. When many programs run at the same time, the OS must decide which task runs on which processor, for how long, and in what order. This process is called thread scheduling.

This project — **Scalable Multithreaded Task Manager with Thread Pool** — simulates how a real OS handles multithreaded task management. Instead of creating a new thread for every task (which is expensive in time and memory), we pre-create a fixed set of worker threads that reuse themselves to handle all incoming tasks.

The project is built with Python and a fully interactive Tkinter GUI, so you can visually see threads working, tasks completing in real time, performance being measured, and statistics being charted — exactly like a real OS scheduler would behave.

## 2. Objectives
- Understand and implement the Thread Pool design pattern used in OS design.
- Build a Priority Task Queue that orders tasks by importance (High > Medium > Low).
- Implement thread synchronization using `threading.Lock()` to prevent race conditions.
- Build a full GUI that shows live thread status, task progress, and execution logs.
- Compare performance between single-threaded and multi-threaded execution.
- Add 7 advanced features: live progress timers, pause/resume, drag reorder, category tags, activity graph, task history, and log export.

---

## 3. System Architecture

The project follows a layered architecture with three main layers:

### 3.1 Presentation Layer (GUI)
The Tkinter-based GUI provides the user interface. It has three tabs:
- **Manager Tab** — Task input, queue table, thread status, live graph, performance chart.
- **History Tab** — Table of all completed tasks with timestamps and speedup estimates.
- **Statistics Tab** — Bar charts by category/priority and thread utilization bars.

### 3.2 Logic Layer (Thread Pool Manager)
This layer manages all business logic:
- Creates and maintains `NUM_THREADS=4` worker threads.
- Maintains a `PriorityQueue` where High priority tasks are dequeued first.
- Uses `threading.Lock()` to protect shared state from race conditions.
- Schedules tasks to idle threads using a loop-based dispatcher.
- Communicates results back to the GUI via safe callback functions.

### 3.3 Data Layer (Task & State)
Tasks are represented as Python objects with these key attributes:

| Attribute | Type | Description |
| :--- | :--- | :--- |
| `id` | int | Auto-incremented unique task identifier |
| `name` | str | User-provided task name |
| `priority` | str | High / Medium / Low |
| `category` | str | CPU / IO / Network |
| `duration` | float | Simulated execution time (1.5–4.0 seconds) |
| `status` | str | Waiting / Running / Paused / Completed |
| `threadId` | int | Index of the thread executing this task |
| `progress` | float | Current completion percentage (0–100) |

### 3.4 Architecture Diagram
```text
┌──────────────────────────────────────────────────────────────────┐
│                   GUI LAYER (Tkinter)                            │
│  ┌─────────────┐  ┌──────────────────┐  ┌───────────────────┐   │
│  │ Task Input  │  │  Task Queue Table │  │  Thread Status    │   │
│  │ Panel       │  │  (with drag+pause)│  │  (4 thread cards) │   │
│  └──────┬──────┘  └────────┬─────────┘  └────────┬──────────┘   │
└─────────┼──────────────────┼─────────────────────┼──────────────┘
          │   add_task()     │  update_row()        │ set_thread_ui()
          ▼                  ▼                      ▼
┌──────────────────────────────────────────────────────────────────┐
│              THREAD POOL MANAGER (Logic Layer)                   │
│                                                                  │
│    ┌──────────────────────────────────────────────┐              │
│    │   PriorityQueue  [High=1 > Medium=2 > Low=3] │              │
│    └─────────────────────┬────────────────────────┘              │
│                          │  fetch task                           │
│         ┌────────────────┼───────────────────┐                  │
│         ▼                ▼                   ▼                   │
│    [Worker-1]       [Worker-2]          [Worker-4]               │
│    (Thread)         (Thread)            (Thread)                 │
│         │                                    │                   │
│         └────────── threading.Lock() ────────┘                  │
│                    (protects shared state)                        │
└──────────────────────────────────────────────────────────────────┘
          │
          ▼
   Execution Log  +  Activity Graph  +  Performance Chart
```

---

## 4. Algorithms Used

### 4.1 Thread Pool Algorithm
The Thread Pool avoids repeated thread creation/destruction by reusing pre-created threads:

**Algorithm: Thread Pool Execution**
1. At startup: create `N=4` worker threads (daemon threads)
2. Each worker runs an infinite loop:
   a. Call `queue.get(timeout=0.5)` ← blocks until task available
   b. If sentinel (`None`) received → EXIT loop
   c. Mark `task.status = 'Running'`
   d. Execute task (simulate with `time.sleep(duration)`)
   e. Mark `task.status = 'Completed'`
   f. Call `queue.task_done()`
   g. Go back to step a
3. On Stop: push `N` sentinel values to wake all blocked threads

### 4.2 Priority Queue Scheduling
Python's `queue.PriorityQueue` uses a min-heap internally. Tasks are inserted as tuples `(priority_value, task)`. Lower numbers are dequeued first, so High priority tasks always run before Medium or Low.

| Priority Level | Priority Value | Execution Order |
| :--- | :--- | :--- |
| High | 1 | First (lowest number = highest priority) |
| Medium | 2 | Second |
| Low | 3 | Last |

### 4.3 Synchronization — Mutual Exclusion
A race condition occurs when two threads modify shared data simultaneously, leading to corrupt state. We use `threading.Lock()` to create a critical section:

```python
# Only ONE thread can execute inside 'with self.lock' at a time
with self.lock:
    task.status = "Completed"     # protected write
    self.completed.append(task)   # protected list update

# Other threads wait at the lock boundary until it is released
```

### 4.4 Performance Measurement
The benchmark compares two execution models:

| Metric | Formula | Meaning |
| :--- | :--- | :--- |
| Single-Thread Time | Sum of all task durations | Total time if executed one by one |
| Multi-Thread Time | Max of all task durations | Bottleneck = longest task (parallel ideal) |
| Speedup Factor | Single ÷ Multi | How many times faster multithreading is |

---

## 5. Feature List — Version 2.0

| # | Feature | Description |
| :--- | :--- | :--- |
| 1 | Thread Pool (4 Workers) | 4 pre-created daemon threads fetch and execute tasks continuously without being re-created. |
| 2 | Priority Queue | Tasks are ordered High → Medium → Low using Python's PriorityQueue (min-heap). |
| 3 | Task Progress % + Live Timer | Each running task shows a real-time animated progress bar (0–100%) and countdown timer. |
| 4 | Pause / Resume Tasks | Any task — whether waiting or running mid-execution — can be individually paused and resumed. |
| 5 | Drag to Reorder | Waiting tasks can be dragged to change their position in the queue, immediately affecting execution order. |
| 6 | Task Categories | Tasks are tagged CPU / I/O / Network with color-coded badges and charted in the Statistics tab. |
| 7 | Live Activity Graph | A canvas-drawn line graph plots busy thread count and completed task count every 500ms. |
| 8 | Task History Tab | Every completed task is logged with name, category, priority, thread, duration, timestamp, and speedup estimate. |
| 9 | Statistics Dashboard | Bar charts show task distribution by category and priority; thread utilization shows each thread's workload. |
| 10 | Export Log / CSV | Execution log exports as .txt; task history exports as .csv with one click. |
| 11 | Dark / Light Theme | Full theme toggle between dark (hacker terminal) and light (clean professional) modes. |
| 12 | Benchmark Chart | Side-by-side bar chart compares single-thread vs multi-thread execution time with speedup factor. |

---

## 6. Code Explanation

### 6.1 Task Class
```python
class Task:
    def __init__(self, name, priority, category):
        self.id       = auto_increment()   # Unique ID
        self.name     = name
        self.priority = priority           # "High" | "Medium" | "Low"
        self.category = category           # "CPU" | "IO" | "Network"
        self.pval     = PMAP[priority]     # 1, 2, or 3 for queue ordering
        self.status   = "Waiting"          # Waiting→Running→Completed/Paused
        self.duration = random(1.5, 4.0)   # Simulated work duration
        self.threadId = None               # Set when thread picks up task
        self.progress = 0                  # 0–100 percent complete
```

### 6.2 Worker Thread Loop (Fixed Closure Bug)
The key bug fixed in v2 was a JavaScript closure issue inside a for-loop. We used an IIFE (Immediately Invoked Function Expression) to correctly capture the loop variable for each iteration:

```python
# Python equivalent — using a closure-safe approach:
def _worker(self, thread_index):
    while not self.stop_event.is_set():
        priority, task = self.task_queue.get(timeout=0.5)
        if task is None: break            # Sentinel → exit
        
        task.status   = "Running"
        task.threadId = thread_index
        
        time.sleep(task.duration)         # Simulate CPU work
        
        with self.lock:                   # Critical section
            task.status = "Completed"
            self.done_tasks.append(task)
        
        self.task_queue.task_done()
```

### 6.3 Scheduler — Closure Bug Fix (JavaScript)
```javascript
// BUG: variable 'i' captured by reference in all callbacks — all get i=4
for (let i = 0; i < 4; i++) {
    setTimeout(function() { console.log(i); }, 1000); // prints 4,4,4,4
}

// FIX: IIFE captures 'i' by value at call time
for (let i = 0; i < 4; i++) {
    (function(threadIdx) {
        setTimeout(function() { console.log(threadIdx); }, 1000); // 0,1,2,3
    })(i);
}

// Applied in our scheduler:
(function runTask(theTask, threadIdx) {
    // theTask and threadIdx are now safely captured per iteration
    taskTimers[theTask.id] = setTimeout(function() {
        setThreadIdle(threadIdx);   // correct thread index always
        doLog("Thread-" + (threadIdx+1) + " done");
    }, theTask.duration * 1000);
})(task, i);
```

### 6.4 Thread-Safe GUI Updates
Background threads cannot directly modify Tkinter widgets (causes crashes). All updates use `after(0, callback)` to schedule on the main thread:

```python
# Wrong (crashes):
def _worker(self):
    label.config(text="Done")   # Direct GUI update from thread = CRASH

# Correct:
def _worker(self):
    self.root.after(0, self._update_label, "Done")  # Schedules on main thread

def _update_label(self, text):
    label.config(text=text)     # Now runs safely on main thread
```

---

## 7. Advantages and Limitations

### 7.1 Advantages

| # | Advantage | Explanation |
| :--- | :--- | :--- |
| 1 | Thread Reuse | No overhead of creating/destroying threads per task — threads are pre-created once. |
| 2 | Priority Scheduling | Important tasks always execute first — same as OS process priorities. |
| 3 | Race Condition Safe | `threading.Lock()` ensures only one thread modifies shared data at a time. |
| 4 | Scalable | Change `NUM_THREADS = 8` and the pool instantly doubles capacity. |
| 5 | Real-time Visibility | GUI shows live progress, thread states, and logs — full observability. |
| 6 | Measurable Speedup | Benchmark proves multithreading is faster and shows the exact speedup factor. |

### 7.2 Limitations

| # | Limitation | Explanation |
| :--- | :--- | :--- |
| 1 | Python GIL | CPython's Global Interpreter Lock prevents true CPU parallelism. Use multiprocessing for CPU-bound tasks. |
| 2 | Simulated Tasks | Tasks use `sleep()` — real tasks would involve file I/O, network calls, or computations. |
| 3 | Fixed Pool Size | Thread count is set at startup. No dynamic scaling based on queue length. |
| 4 | No Persistence | Tasks are lost on reset. No database or file storage. |
| 5 | No Task Cancellation | Running tasks cannot be interrupted mid-execution (only queued/paused tasks). |

---

## 8. Future Enhancements

- **Dynamic Thread Scaling** — Automatically grow/shrink the pool based on current queue size.
- **Real Task Types** — Replace `sleep()` with actual File I/O, HTTP requests, or image processing.
- **Database Logging** — Store all task history persistently using SQLite.
- **Task Dependencies** — Task B only starts after Task A completes (DAG scheduling).
- **CPU Affinity** — Pin specific threads to specific CPU cores using multiprocessing.
- **Network Distribution** — Distribute tasks across multiple machines using sockets or Celery.
- **Deadline Scheduling** — Add task deadlines; mark overdue tasks in red.
- **Real-time CPU/Memory Monitor** — Show actual system resource usage per thread.

---

## 9. Viva Questions & Answers

**Q1. What is a Thread Pool and why is it used?**  
A Thread Pool is a set of pre-created worker threads that are reused for multiple tasks. It avoids the overhead of creating and destroying a new thread for every task. Thread creation involves OS-level memory allocation, stack setup, and scheduler registration — all expensive. Reusing threads makes the system faster and more efficient, especially under high task loads.

**Q2. What is the difference between a Thread and a Process?**  
A Process is a complete, independent program with its own memory address space. A Thread is a lightweight unit of execution within a process that shares the process's memory. Threads are cheaper to create and can communicate directly via shared memory, while processes must use IPC (pipes, sockets). A crash in one thread can affect the entire process, while processes are isolated from each other.

**Q3. What is a Race Condition and how did you prevent it?**  
A Race Condition occurs when two or more threads access and modify the same shared data concurrently, producing unpredictable results. For example, two threads both reading `task.status` as 'Running' and both trying to mark it 'Completed'. We prevent this using `threading.Lock()` — only one thread can hold the lock at a time. All critical sections (`task.status` updates, `completed[]` appends) are wrapped with: `with self.lock: # safe code here`

**Q4. What is Deadlock and how can it be avoided?**  
Deadlock occurs when two threads each hold a lock that the other needs, causing both to wait forever. The four Coffman Conditions are: (1) Mutual Exclusion, (2) Hold and Wait, (3) No Preemption, (4) Circular Wait. Prevention strategies: always acquire locks in the same order, use `lock.acquire(timeout=...)` to avoid infinite waiting, use `threading.RLock()` for reentrant scenarios, and minimize lock scope. In our project, we use only one lock, so circular wait between locks is impossible.

**Q5. What is the GIL and how does it affect our project?**  
The GIL (Global Interpreter Lock) is a mutex in CPython that allows only one thread to execute Python bytecode at a time. For CPU-bound tasks, this means Python threads cannot achieve true parallelism. However, for I/O-bound tasks (file reads, network calls, `sleep()`), the GIL is released during the wait, allowing real concurrent execution. Our simulation uses `time.sleep()` which releases the GIL, so our threads do run in parallel effectively.

**Q6. Explain the Producer-Consumer pattern in your project.**  
The Producer-Consumer pattern separates task creation (production) from task execution (consumption) using a shared buffer. In our project: Producer = the GUI user adds tasks via `add_task()` → inserted into PriorityQueue. Consumer = worker threads continuously call `queue.get()` → execute tasks. Buffer = the PriorityQueue acts as the shared, thread-safe buffer. This pattern prevents producers from overwhelming consumers and decouples task creation from execution.

**Q7. What is a Sentinel Value and why did you use it?**  
A Sentinel is a special signal value that tells consumers to stop. Worker threads block indefinitely on `queue.get()`. Simply setting a stop flag doesn't unblock a waiting thread. So when Stop is pressed, we push `(0, None)` into the queue for each worker thread. When a thread retrieves `task = None`, it recognizes the sentinel and exits its loop. This is the standard pattern for cleanly shutting down thread pools.

**Q8. Why can't background threads directly update Tkinter widgets?**  
Tkinter is not thread-safe. Its internal event loop and widget tree are not protected by locks. Updating widgets from a background thread causes unpredictable crashes, visual glitches, or silent data corruption. The safe solution is to use `root.after(0, callback)`, which schedules the callback to run on the main thread at the next event loop iteration. This ensures all GUI updates happen on exactly one thread.

**Q9. What is the difference between threading.Lock and threading.Event?**  
`threading.Lock` provides mutual exclusion — only one thread enters a critical section at a time. It is used to protect shared data. `threading.Event` is a signaling mechanism — one thread sets the event, others wait for it. It is used for coordination and notification. In our project: Lock protects `task.status` and the `completed[]` list. `stop_event` (Event) signals all worker threads to shut down cleanly.

**Q10. What is the IIFE pattern and why was it critical in fixing the closure bug?**  
IIFE stands for Immediately Invoked Function Expression. In JavaScript, variables declared with `var` inside a for-loop are shared across all iterations due to closure-by-reference. This caused all `setTimeout` callbacks to use the same (final) value of `i`. The fix is to wrap each iteration in an IIFE: `(function(capturedI) { ... })(i);` — this creates a new scope per iteration and captures `i` by value. In our project, this ensured each task's completion callback referenced the correct thread index instead of always using the last value.

---

## 10. Key Concepts Explained

**Thread vs Process**  
A process is a full independent program with its own memory space. A thread is a lighter unit inside a process sharing its memory. Threads communicate via shared variables (fast but needs synchronization). Processes communicate via IPC (slower but isolated). Use threads for concurrent I/O tasks; use processes for true CPU parallelism in Python (bypasses GIL).

**Thread Pool Pattern**  
Pre-create `N` worker threads at startup. Maintain a task queue. Workers loop forever: fetch task → execute → fetch next. On shutdown, send `N` sentinel values to unblock all workers. Benefits: eliminates repeated thread creation cost, gives fine-grained control over concurrency level, prevents resource exhaustion from unbounded thread creation.

**Synchronization**  
Coordination of concurrent threads accessing shared resources. Tools: Lock/Mutex (one thread at a time), Semaphore (up to N threads), Event (signal-based wait/notify), Condition (wait until predicate is true), RLock (reentrant lock for recursive calls). Without synchronization, race conditions corrupt shared state silently.

**Deadlock**  
Permanent blocking of two or more threads waiting for resources held by each other. Requires all four Coffman Conditions simultaneously: Mutual Exclusion (resource can't be shared), Hold and Wait (thread holds one, waits for another), No Preemption (can't forcibly take resources), Circular Wait (A waits for B, B waits for A). Breaking any one condition prevents deadlock. Our project uses a single lock, making circular wait impossible.

**Priority Scheduling**  
Tasks are assigned priority values and a min-heap (PriorityQueue) ensures the highest-priority task is always dequeued first. This mirrors OS scheduling where real-time or interactive processes receive higher priority than background batch jobs. Priority inversion (a low-priority task blocking a high-priority one) is a known problem — solved by priority inheritance in real OS schedulers.

---

## 11. Conclusion

This project successfully demonstrates how an Operating System efficiently manages multiple concurrent tasks using a Thread Pool and Priority Queue. The implementation covers all key OS concepts — thread lifecycle, synchronization, priority scheduling, and performance analysis — in a visual, interactive, and beginner-friendly way.

The GUI makes abstract concepts concrete: you can literally see Thread-1 go orange and busy, a task's progress bar fill from 0% to 100%, and the activity graph spike as all 4 threads run in parallel. The benchmark chart shows in numbers why multithreading matters — a speedup of 3× or 4× over sequential execution.

Version 2.0 adds 7 advanced features (pause/resume, drag reorder, categories, live graph, history, statistics, export) making this project suitable not just as a college submission, but as a foundation for more advanced OS scheduler simulations.
