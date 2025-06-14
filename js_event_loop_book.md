# The JavaScript Event Loop: A Complete Guide

## Table of Contents

- [Foreword](#foreword)
- [Chapter 1: Understanding JavaScript's Single-Threaded Nature](#chapter-1-understanding-javascripts-single-threaded-nature)
- [Chapter 2: The JavaScript Runtime Environment](#chapter-2-the-javascript-runtime-environment)
- [Chapter 3: The Event Loop Fundamentals](#chapter-3-the-event-loop-fundamentals)
- [Chapter 4: Call Stack Deep Dive](#chapter-4-call-stack-deep-dive)
- [Chapter 5: Task Queues and Microtasks](#chapter-5-task-queues-and-microtasks)
- [Chapter 6: Timers and Scheduling](#chapter-6-timers-and-scheduling)
- [Chapter 7: Promises and the Event Loop](#chapter-7-promises-and-the-event-loop)
- [Chapter 8: Node.js Event Loop Specifics](#chapter-8-nodejs-event-loop-specifics)
- [Chapter 9: Performance Implications](#chapter-9-performance-implications)
- [Chapter 10: Common Pitfalls and Debugging](#chapter-10-common-pitfalls-and-debugging)
- [Appendix: Practical Examples and Exercises](#appendix-practical-examples-and-exercises)

---

## Foreword

The JavaScript Event Loop is one of the most fundamental yet misunderstood concepts in JavaScript development. Whether you're building web applications in the browser or server-side applications with Node.js, understanding how JavaScript handles asynchronous operations is crucial for writing efficient, responsive code.

This guide will take you on a comprehensive journey through the Event Loop, from basic concepts to advanced patterns. By the end, you'll have a deep understanding of how JavaScript executes code, handles asynchronous operations, and maintains responsiveness in a single-threaded environment.

---

## Chapter 1: Understanding JavaScript's Single-Threaded Nature

### The Foundation of JavaScript

JavaScript is fundamentally a **single-threaded** programming language. This means that JavaScript can only execute one piece of code at a time in its main execution thread. Unlike languages such as Java or C++, which can create multiple threads to run code in parallel, JavaScript operates with a single call stack.

```javascript
console.log("First");
console.log("Second");
console.log("Third");
```

**Execution Flow:**
```
┌─────────────┐
│   "First"   │  ← Executes
└─────────────┘
┌─────────────┐
│   "Second"  │  ← Then executes
└─────────────┘
┌─────────────┐
│   "Third"   │  ← Finally executes
└─────────────┘
```

In this simple example, the statements execute sequentially, one after another. There's no parallelism—each `console.log` must complete before the next one begins.

### Why Single-Threaded?

JavaScript was originally designed to run in web browsers, where it needed to interact with the DOM (Document Object Model). Having multiple threads accessing and modifying the DOM simultaneously would create race conditions and make the language much more complex.

**Multi-threaded Problems (Avoided in JavaScript):**
```
Thread 1: ───────► Change element color to red
Thread 2: ───────► Change same element color to blue
Thread 3: ───────► Remove the element

Result: Race condition! Unpredictable behavior!
```

**JavaScript's Solution:**
```
Single Thread: ──► Change color ──► Change color ──► Remove element
Result: Predictable, sequential execution!
```

### The Challenge of Asynchronous Operations

The single-threaded nature of JavaScript presents a significant challenge: how do we handle operations that take time without blocking the entire program?

Consider this scenario:

```javascript
console.log("Start");

// This would block for 5 seconds if JavaScript waited
fetchDataFromServer(); // Takes 5 seconds

console.log("End");
```

**Without Event Loop (Blocking):**
```
Time: 0s     1s     2s     3s     4s     5s     6s
      │      │      │      │      │      │      │
      Start  ████████████████████████████ End
             (Blocked waiting for server)
```

**With Event Loop (Non-blocking):**
```
Time: 0s     1s     2s     3s     4s     5s     6s
      │      │      │      │      │      │      │
      Start  End    ░░░░░░░░░░░░░░░░ Server Response
      (Immediate)   (Background)   (Callback)
```

---

## Chapter 2: The JavaScript Runtime Environment

To understand the Event Loop, we first need to understand the JavaScript runtime environment and its components.

### JavaScript Runtime Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    JavaScript Runtime Environment               │
├─────────────────┬───────────────────────────────────────────────┤
│                 │                                               │
│   JavaScript    │              Browser/Node.js APIs            │
│     Engine      │                                               │
│                 │  ┌─────────────┐ ┌─────────────┐            │
│  ┌───────────┐  │  │   setTimeout │ │    fetch    │            │
│  │   Heap    │  │  │ setInterval  │ │   DOM APIs  │            │
│  │ (Memory)  │  │  │              │ │   fs (Node) │            │
│  └───────────┘  │  └─────────────┘ └─────────────┘            │
│                 │                                               │
│  ┌───────────┐  │  ┌─────────────────────────────────────────┐ │
│  │   Call    │  │  │           Event Loop                    │ │
│  │   Stack   │  │  │                                         │ │
│  └───────────┘  │  │  ┌─────────────────┐ ┌─────────────────┐│ │
│                 │  │  │  Microtask      │ │   Macrotask     ││ │
│                 │  │  │    Queue        │ │     Queue       ││ │
│                 │  │  │  (Promises)     │ │  (setTimeout)   ││ │
│                 │  │  └─────────────────┘ └─────────────────┘│ │
│                 │  └─────────────────────────────────────────┘ │
└─────────────────┴───────────────────────────────────────────────┘
```

### Memory Heap

The **memory heap** is where memory allocation happens. When you declare variables or create objects, memory is allocated in the heap:

```javascript
let myString = "Hello World"; // Allocated in heap
let myObject = { name: "John" }; // Allocated in heap
```

**Heap Structure:**
```
┌─────────────────────────────────────┐
│              Memory Heap            │
├─────────────┬─────────────┬─────────┤
│ myString    │ myObject    │  ...    │
│"Hello World"│{name:"John"}│         │
└─────────────┴─────────────┴─────────┘
```

### Call Stack

The **call stack** is where JavaScript keeps track of function calls. It's a LIFO (Last In, First Out) data structure:

```javascript
function first() {
    console.log("Inside first");
    second();
    console.log("Back in first");
}

function second() {
    console.log("Inside second");
    third();
    console.log("Back in second");
}

function third() {
    console.log("Inside third");
}

first();
```

**Call Stack Progression:**

```
Step 1:          Step 2:          Step 3:          Step 4:
┌─────────┐      ┌─────────┐      ┌─────────┐      ┌─────────┐
│         │      │ second()│      │ third() │      │ second()│
├─────────┤      ├─────────┤      ├─────────┤      ├─────────┤
│ first() │      │ first() │      │ second()│      │ first() │
├─────────┤      ├─────────┤      ├─────────┤      ├─────────┤
│ global()│      │ global()│      │ first() │      │ global()│
└─────────┘      └─────────┘      └─────────┘      └─────────┘
    ↓                ↓                ↓                ↓
first() called   second() called  third() called  third() returns
```

---

## Chapter 3: The Event Loop Fundamentals

### What is the Event Loop?

The **Event Loop** is the mechanism that allows JavaScript to perform non-blocking operations despite being single-threaded.

**Event Loop Core Components:**

```
┌─────────────────────────────────────────────────────────────────┐
│                         Event Loop                             │
│                                                                 │
│    1. Check Call Stack                                          │
│       ┌─────────────┐                                          │
│       │ Call Stack  │ ← Is it empty?                          │
│       └─────────────┘                                          │
│              │                                                  │
│              ▼                                                  │
│    2. Process Microtasks                                        │
│       ┌─────────────┐                                          │
│       │ Microtask   │ ← Process ALL microtasks                │
│       │   Queue     │                                          │
│       └─────────────┘                                          │
│              │                                                  │
│              ▼                                                  │
│    3. Process One Macrotask                                     │
│       ┌─────────────┐                                          │
│       │ Macrotask   │ ← Process ONE macrotask                 │
│       │   Queue     │                                          │
│       └─────────────┘                                          │
│              │                                                  │
│              ▼                                                  │
│    4. Repeat (Back to step 1)                                  │
└─────────────────────────────────────────────────────────────────┘
```

### Event Loop Algorithm

```
while (eventLoop.isRunning()) {
    // 1. Execute all synchronous code in call stack
    while (!callStack.isEmpty()) {
        callStack.executeNext();
    }
    
    // 2. Process ALL microtasks
    while (!microtaskQueue.isEmpty()) {
        microtaskQueue.dequeue().execute();
    }
    
    // 3. Process ONE macrotask
    if (!macrotaskQueue.isEmpty()) {
        macrotaskQueue.dequeue().execute();
    }
    
    // 4. Render (in browser)
    if (needsRendering) {
        render();
    }
}
```

### Simple Event Loop Example

```javascript
console.log("Start");

setTimeout(() => {
    console.log("Timer callback");
}, 0);

console.log("End");

// Output:
// Start
// End
// Timer callback
```

**Execution Timeline:**

```
Time  │ Call Stack      │ Macrotask Queue     │ Output
──────┼─────────────────┼─────────────────────┼──────────
  1   │ console.log()   │                     │ "Start"
  2   │ setTimeout()    │                     │
  3   │                 │ [timer callback]    │
  4   │ console.log()   │ [timer callback]    │ "End"
  5   │                 │ [timer callback]    │
  6   │ timer callback  │                     │ "Timer callback"
```

---

## Chapter 4: Call Stack Deep Dive

### How the Call Stack Works

The call stack operates on a simple principle: **Last In, First Out (LIFO)**.

**Stack Operations:**

```
Push (function call):     Pop (function return):
┌─────────┐              ┌─────────┐
│ new()   │ ← pushed     │         │ ← popped
├─────────┤              ├─────────┤
│ func2() │              │ func2() │
├─────────┤              ├─────────┤
│ func1() │              │ func1() │
└─────────┘              └─────────┘
```

### Stack Frame Contents

Each stack frame contains:

```
┌─────────────────────────────────────┐
│           Stack Frame               │
├─────────────────────────────────────┤
│ Function Name: multiply             │
│ Parameters: a = 4, b = 4            │
│ Local Variables: result             │
│ Return Address: line 8              │
│ Scope Chain: [global, multiply]     │
└─────────────────────────────────────┘
```

### Complex Call Stack Example

```javascript
function multiply(a, b) {
    return a * b;
}

function square(n) {
    return multiply(n, n);
}

function printSquare(num) {
    let result = square(num);
    console.log(result);
}

printSquare(4);
```

**Detailed Stack Progression:**

```
Step 1: printSquare(4) called
┌─────────────────────────┐
│ printSquare(4)          │
│ - num: 4                │
│ - result: undefined     │
├─────────────────────────┤
│ global execution        │
└─────────────────────────┘

Step 2: square(4) called
┌─────────────────────────┐
│ square(4)               │
│ - n: 4                  │
├─────────────────────────┤
│ printSquare(4)          │
│ - num: 4                │
│ - result: undefined     │
├─────────────────────────┤
│ global execution        │
└─────────────────────────┘

Step 3: multiply(4, 4) called
┌─────────────────────────┐
│ multiply(4, 4)          │
│ - a: 4, b: 4            │
├─────────────────────────┤
│ square(4)               │
│ - n: 4                  │
├─────────────────────────┤
│ printSquare(4)          │
│ - num: 4                │
│ - result: undefined     │
├─────────────────────────┤
│ global execution        │
└─────────────────────────┘

Step 4: multiply returns 16
┌─────────────────────────┐
│ square(4)               │
│ - n: 4                  │
│ - return value: 16      │
├─────────────────────────┤
│ printSquare(4)          │
│ - num: 4                │
│ - result: undefined     │
├─────────────────────────┤
│ global execution        │
└─────────────────────────┘
```

### Stack Overflow Visualization

```javascript
function infiniteRecursion(depth = 0) {
    console.log(`Depth: ${depth}`);
    infiniteRecursion(depth + 1); // This will cause stack overflow
}
```

**Stack Growth:**

```
Normal Stack:              Stack Overflow:
┌─────────┐               ┌─────────┐ ← Stack limit reached!
│ func3() │               │ recur() │   RangeError: Maximum
├─────────┤               ├─────────┤   call stack size exceeded
│ func2() │               │ recur() │
├─────────┤               ├─────────┤
│ func1() │               │ recur() │
└─────────┘               ├─────────┤
                          │ recur() │
                          ├─────────┤
                          │   ...   │
                          └─────────┘
```

---

## Chapter 5: Task Queues and Microtasks

### Queue Priority System

```
┌─────────────────────────────────────────────────────────────────┐
│                    Event Loop Priority                         │
│                                                                 │
│ 1. Call Stack (Highest Priority)                               │
│    ┌─────────────────┐                                         │
│    │ Synchronous     │ ← Execute immediately                   │
│    │ Code            │                                         │
│    └─────────────────┘                                         │
│                                                                 │
│ 2. Microtask Queue                                              │
│    ┌─────────────────┐                                         │
│    │ Promise.then()  │ ← Process ALL before macrotasks        │
│    │ queueMicrotask  │                                         │
│    │ process.nextTick│                                         │
│    └─────────────────┘                                         │
│                                                                 │
│ 3. Macrotask Queue (Lowest Priority)                           │
│    ┌─────────────────┐                                         │
│    │ setTimeout      │ ← Process ONE at a time                │
│    │ setInterval     │                                         │
│    │ I/O callbacks   │                                         │
│    └─────────────────┘                                         │
└─────────────────────────────────────────────────────────────────┘
```

### Microtask vs Macrotask Example

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0);          // Macrotask
Promise.resolve().then(() => console.log("3")); // Microtask
setTimeout(() => console.log("4"), 0);          // Macrotask
Promise.resolve().then(() => console.log("5")); // Microtask

console.log("6");
```

**Execution Flow:**

```
Phase 1 - Synchronous Code:
┌─────────────────┐
│ Call Stack      │ → Output: "1", "6"
│ console.log("1")│
│ console.log("6")│
└─────────────────┘

Phase 2 - Microtasks (ALL processed):
┌─────────────────┐
│ Microtask Queue │ → Output: "3", "5"
│ then(() => "3") │
│ then(() => "5") │
└─────────────────┘

Phase 3 - Macrotasks (ONE at a time):
┌─────────────────┐
│ Macrotask Queue │ → Output: "2"
│ timeout(() =>"2")│ ← First macrotask
│ timeout(() =>"4")│
└─────────────────┘

Phase 4 - Next Macrotask:
┌─────────────────┐
│ Macrotask Queue │ → Output: "4"
│ timeout(() =>"4")│ ← Second macrotask
└─────────────────┘

Final Output: 1, 6, 3, 5, 2, 4
```

### Complex Queue Interaction

```javascript
console.log("Start");

setTimeout(() => {
    console.log("Timer 1");
    Promise.resolve().then(() => console.log("Promise in Timer 1"));
}, 0);

Promise.resolve().then(() => {
    console.log("Promise 1");
    setTimeout(() => console.log("Timer in Promise 1"), 0);
});

setTimeout(() => console.log("Timer 2"), 0);

console.log("End");
```

**Detailed Timeline:**

```
Step │ Call Stack      │ Microtask Queue │ Macrotask Queue      │ Output
─────┼─────────────────┼─────────────────┼──────────────────────┼─────────────
  1  │ console.log()   │                 │                      │ "Start"
  2  │ setTimeout()    │                 │ [Timer 1 callback]   │
  3  │ Promise.then()  │ [Promise 1]     │ [Timer 1 callback]   │
  4  │ setTimeout()    │ [Promise 1]     │ [Timer 1, Timer 2]   │
  5  │ console.log()   │ [Promise 1]     │ [Timer 1, Timer 2]   │ "End"
  6  │                 │                 │ [Timer 1, Timer 2]   │
  7  │ Promise 1       │                 │ [Timer 1, Timer 2]   │ "Promise 1"
  8  │ setTimeout()    │                 │ [Timer 1, Timer 2,   │
     │                 │                 │  Timer in Promise 1] │
  9  │ Timer 1         │                 │ [Timer 2,            │ "Timer 1"
     │                 │                 │  Timer in Promise 1] │
 10  │ Promise.then()  │ [Promise in     │ [Timer 2,            │
     │                 │  Timer 1]       │  Timer in Promise 1] │
 11  │ Promise in      │                 │ [Timer 2,            │ "Promise in
     │ Timer 1         │                 │  Timer in Promise 1] │  Timer 1"
 12  │ Timer 2         │                 │ [Timer in Promise 1] │ "Timer 2"
 13  │ Timer in        │                 │                      │ "Timer in
     │ Promise 1       │                 │                      │  Promise 1"
```

---

## Chapter 6: Timers and Scheduling

### Timer Functions Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                       Timer Functions                          │
├─────────────────┬─────────────────┬─────────────────┬───────────┤
│   setTimeout    │   setInterval   │  setImmediate   │ reqAnimFr │
│                 │                 │   (Node.js)     │ (Browser) │
├─────────────────┼─────────────────┼─────────────────┼───────────┤
│ Runs once after │ Runs repeatedly │ Runs in next    │ Runs before│
│ minimum delay   │ every interval  │ event loop tick │ repaint   │
├─────────────────┼─────────────────┼─────────────────┼───────────┤
│ Macrotask Queue │ Macrotask Queue │ Check Phase     │ RAF Queue │
│                 │                 │ (Node.js)       │           │
└─────────────────┴─────────────────┴─────────────────┴───────────┘
```

### setTimeout Behavior

```javascript
console.log("Start");

setTimeout(() => {
    console.log("Timer 1 (0ms)");
}, 0);

setTimeout(() => {
    console.log("Timer 2 (100ms)");
}, 100);

console.log("End");
```

**Timeline Visualization:**

```
Time:  0ms      100ms     200ms
       │        │         │
       │        │         │
Start──┘        │         │
End─────────────┘         │
Timer 1 (0ms)─────────────┘
Timer 2 (100ms)───────────────────────┘

Actual execution:
0ms: "Start", "End" (synchronous)
~4ms: "Timer 1 (0ms)" (minimum 4ms delay in browsers)
100ms: "Timer 2 (100ms)"
```

### setInterval vs setTimeout Patterns

**setInterval (Fixed Intervals):**
```javascript
let count = 0;
const intervalId = setInterval(() => {
    console.log(`Count: ${++count}`);
    if (count >= 5) {
        clearInterval(intervalId);
    }
}, 1000);
```

**Visualization:**
```
Time: 0s    1s    2s    3s    4s    5s
      │     │     │     │     │     │
      Start Count Count Count Count Count
            1     2     3     4     5
```

**setTimeout Chain (Adaptive Intervals):**
```javascript
let count = 0;
function scheduleNext() {
    setTimeout(() => {
        console.log(`Count: ${++count}`);
        if (count < 5) {
            scheduleNext();
        }
    }, 1000);
}
scheduleNext();
```

**Key Difference:**
```
setInterval:     setTimeout chain:
┌─Timer─┐        ┌─Timer─┐
│ 1000ms│        │ 1000ms│
└───┬───┘        └───┬───┘
    │                │
    ▼                ▼
Execute          Execute ──┐
    │                     │
    │            ┌─Timer─┐ │
    │            │ 1000ms│ │
    │            └───┬───┘ │
    │                │     │
    │                ▼     │
    │            Execute ◄─┘
    │
Fixed timing     Adaptive timing
(can drift)      (accounts for execution time)
```

### requestAnimationFrame (Browser)

```javascript
function animate() {
    // Animation logic here
    console.log("Frame rendered");
    requestAnimationFrame(animate);
}

requestAnimationFrame(animate);
```

**Browser Frame Timeline:**
```
Browser Refresh Rate: 60 FPS (16.67ms per frame)

Frame: 1     2     3     4     5     6
Time:  0ms   16ms  33ms  50ms  66ms  83ms
       │     │     │     │     │     │
       │     │     │     │     │     │
       ▼     ▼     ▼     ▼     ▼     ▼
    Render Render Render Render Render Render
```

---

## Chapter 7: Promises and the Event Loop

### Promise Lifecycle and Microtasks

```
┌─────────────────────────────────────────────────────────────────┐
│                    Promise Lifecycle                           │
│                                                                 │
│ 1. Promise Created                                              │
│    ┌─────────────────┐                                         │
│    │ new Promise()   │ ← Executor runs immediately            │
│    │ Promise.resolve │                                         │
│    └─────────────────┘                                         │
│            │                                                    │
│            ▼                                                    │
│ 2. Promise Settled                                              │
│    ┌─────────────────┐                                         │
│    │ resolve(value)  │ ← .then() callbacks → Microtask Queue  │
│    │ reject(error)   │ ← .catch() callbacks → Microtask Queue │
│    └─────────────────┘                                         │
│            │                                                    │
│            ▼                                                    │
│ 3. Callbacks Queued                                             │
│    ┌─────────────────┐                                         │
│    │ Microtask Queue │ ← Higher priority than macrotasks      │
│    └─────────────────┘                                         │
└─────────────────────────────────────────────────────────────────┘
```

### Promise Execution Flow

```javascript
console.log("1");

Promise.resolve().then(() => {
    console.log("2");
    return Promise.resolve();
}).then(() => {
    console.log("3");
});

Promise.resolve().then(() => {
    console.log("4");
});

console.log("5");
```

**Step-by-Step Execution:**

```
Step 1 - Synchronous Code:
Call Stack: [console.log("1"), Promise.resolve(), console.log("5")]
Output: "1", "5"

Microtask Queue: [then(() => "2"), then(() => "4")]

Step 2 - First Microtask Batch:
Process: then(() => "2")
Output: "2"
New Promise created, adds: then(() => "3") to microtask queue

Process: then(() => "4")
Output: "4"

Microtask Queue: [then(() => "3")]

Step 3 - Second Microtask Batch:
Process: then(() => "3")
Output: "3"

Final Output: 1, 5, 2, 4, 3
```

### async/await and Event Loop

```javascript
async function example() {
    console.log("1");
    
    await Promise.resolve();
    console.log("2"); // This is equivalent to .then(() => console.log("2"))
    
    await Promise.resolve();
    console.log("3");
}

console.log("Start");
example();
console.log("End");
```

**async/await Transformation:**
```javascript
// async/await version:
async function example() {
    console.log("1");
    await Promise.resolve();
    console.log("2");
}

// Equivalent Promise version:
function example() {
    console.log("1");
    return Promise.resolve().then(() => {
        console.log("2");
    });
}
```

**Execution Flow:**
```
Synchronous Phase:
┌─────────────────┐
│ "Start"         │ ← Output
│ "1"             │ ← Output (from async function)
│ "End"           │ ← Output
└─────────────────┘

Microtask Phase:
┌─────────────────┐
│ await resolved  │ ← "2" output
│ await resolved  │ ← "3" output
└─────────────────┘

Output: Start, 1, End, 2, 3
```

### Promise Error Handling

```javascript
Promise.reject("Error!")
    .catch(error => {
        console.log("Caught:", error);
        return "Recovered";
    })
    .then(value => {
        console.log("Success:", value);
    });
```

**Error Flow Diagram:**
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ Promise.reject  │───►│ .catch()        │───►│ .then()         │
│ "Error!"        │    │ Handle error    │    │ Handle success  │
│ (rejected)      │    │ Return recovery │    │ "Recovered"     │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

---

## Chapter 8: Node.js Event Loop Specifics

### Node.js Event Loop Phases

```
┌─────────────────────────────────────────────────────────────────┐
│                    Node.js Event Loop Phases                   │
│                                                                 │
│   ┌─────────────────┐                                          │
│   │  1. Timer       │ ← setTimeout, setInterval callbacks      │
│   │     Phase       │                                          │
│   └─────────────────┘                                          │
│            │                                                    │
│            ▼                                                    │
│   ┌─────────────────┐                                          │
│   │  2. Pending     │ ← I/O callbacks deferred from previous   │
│   │     Callbacks   │   iteration                              │
│   └─────────────────┘                                          │
│            │                                                    │
│            ▼                                                    │
│   ┌─────────────────┐                                          │
│   │  3. Idle,       │ ← Internal use only                     │
│   │     Prepare     │                                          │
│   └─────────────────┘                                          │
│            │                                                    │
│            ▼                                                    │
│   ┌─────────────────┐                                          │
│   │  4. Poll        │ ← Fetch new I/O events                  │
│   │     Phase       │   Execute I/O callbacks                 │
│   └─────────────────┘                                          │
│            │                                                    │
│            ▼                                                    │
│   ┌─────────────────┐                                          │
│   │  5. Check       │ ← setImmediate callbacks                │
│   │     Phase       │                                          │
│   └─────────────────┘                                          │
│            │                                                    │
│            ▼                                                    │
│   ┌─────────────────┐                                          │
│   │  6. Close       │ ← close event callbacks                 │
│   │     Callbacks   │   (socket.on('close', ...))             │
│   └─────────────────┘                                          │
│            │                                                    │
│            └─────────────┐                                     │
│                          │                                     │
│   Between each phase:    ▼                                     │
│   ┌─────────────────┐                                          │
│   │   Microtasks    │ ← process.nextTick, Promises            │
│   │     Queue       │                                          │
│   └─────────────────┘                                          │
└─────────────────────────────────────────────────────────────────┘
```

### Node.js Microtask Priorities

```
┌─────────────────────────────────────────────────────────────────┐
│                    Node.js Microtask Order                     │
│                                                                 │
│ 1. process.nextTick() Queue (Highest Priority)                 │
│    ┌─────────────────┐                                         │
│    │ nextTick Queue  │ ← Always processed first               │
│    │                 │                                         │
│    └─────────────────┘                                         │
│            │                                                    │
│            ▼                                                    │
│ 2. Promise Microtask Queue                                      │
│    ┌─────────────────┐                                         │
│    │ Promise Queue   │ ← .then(), .catch(), .finally()        │
│    │                 │                                         │
│    └─────────────────┘                                         │
└─────────────────────────────────────────────────────────────────┘
```

### setImmediate vs setTimeout(0)

```javascript
// Inside I/O callback:
const fs = require('fs');

fs.readFile(__filename, () => {
    setTimeout(() => console.log('setTimeout'), 0);
    setImmediate(() => console.log('setImmediate'));
});

// Output: setImmediate, setTimeout
```

**Execution Context Matters:**

```
Context 1 - Main Thread (Order may vary):
┌─────────────────┐    ┌─────────────────┐
│ setTimeout(0)   │ ?  │ setImmediate    │
│ (Timer Phase)   │ ←→ │ (Check Phase)   │
└─────────────────┘    └─────────────────┘

Context 2 - I/O Callback (Predictable):
┌─────────────────┐    ┌─────────────────┐
│ setImmediate    │───►│ setTimeout(0)   │
│ (Check Phase)   │    │ (Timer Phase)   │
│ Executes first  │    │ Next iteration  │
└─────────────────┘    └─────────────────┘
```

### File System Operations

```javascript
const fs = require('fs');

console.log('Start');

// Asynchronous file read
fs.readFile(__filename, (err, data) => {
    console.log('File read complete');
    
    // These will execute in predictable order within the callback
    setImmediate(() => console.log('setImmediate'));
    setTimeout(() => console.log('setTimeout'), 0);
    process.nextTick(() => console.log('nextTick'));
    Promise.resolve().then(() => console.log('Promise'));
});

console.log('End');
```

**Thread Pool Visualization:**

```
┌─────────────────────────────────────────────────────────────────┐
│                       libuv Thread Pool                        │
│                                                                 │
│ Main Thread          │  Thread Pool (4 threads by default)     │
│                      │                                         │
│ ┌─────────────────┐  │  ┌───────┐ ┌───────┐ ┌───────┐ ┌───────┐ │
│ │ JavaScript      │  │  │Thread │ │Thread │ │Thread │ │Thread │ │
│ │ Event Loop      │◄─┼──┤   1   │ │   2   │ │   3   │ │   4   │ │
│ │                 │  │  │       │ │       │ │       │ │       │ │
│ └─────────────────┘  │  └───────┘ └───────┘ └───────┘ └───────┘ │
│                      │      │         │         │         │     │
│ fs.readFile() ───────┼──────┘         │         │         │     │
│ crypto.pbkdf2() ─────┼────────────────┘         │         │     │
│ dns.lookup() ────────┼──────────────────────────┘         │     │
│ some I/O ops ────────┼────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────┘
```

---

## Chapter 9: Performance Implications

### Blocking vs Non-blocking Operations

**The Blocking Problem:**

```javascript
// ❌ BAD: Blocks the event loop
function blockingFunction() {
    const start = Date.now();
    while (Date.now() - start < 5000) {
        // Busy waiting for 5 seconds
    }
    console.log('Blocking operation complete');
}

console.log('Before blocking');
blockingFunction();
console.log('After blocking');
```

**Visual Impact:**

```
Time: 0s    1s    2s    3s    4s    5s    6s
      │     │     │     │     │     │     │
      │     │     │     │     │     │     │
Before│████████████████████████████After
      │←──── Event Loop Blocked ────→│
      │                              │
      No other code can execute during this time!
      User interface becomes unresponsive!
```

**Non-blocking Solution:**

```javascript
// ✅ GOOD: Non-blocking approach
function nonBlockingFunction(callback) {
    setTimeout(() => {
        console.log('Non-blocking operation complete');
        callback();
    }, 5000);
}

console.log('Before non-blocking');
nonBlockingFunction(() => {
    console.log('Callback executed');
});
console.log('After non-blocking');
```

**Visual Improvement:**

```
Time: 0s    1s    2s    3s    4s    5s    6s
      │     │     │     │     │     │     │
      │     │     │     │     │     │     │
Before│After│░░░░░░░░░░░░░░░░░░░░░░░░Callback
      │     │                       │
      │     Other code can run      │
      │     Event loop responsive   │
```

### CPU-Intensive Task Optimization

**Problem: Large Array Processing**

```javascript
// ❌ BAD: Blocks event loop
function processLargeArray(array) {
    console.log('Processing started');
    const result = array.map(item => {
        // Simulated heavy computation
        let sum = 0;
        for (let i = 0; i < 1000000; i++) {
            sum += Math.random();
        }
        return item * sum;
    });
    console.log('Processing complete');
    return result;
}
```

**Solution: Chunked Processing**

```javascript
// ✅ GOOD: Process in chunks
function processLargeArrayAsync(array, chunkSize = 1000, callback) {
    let index = 0;
    const result = [];
    
    function processChunk() {
        const endIndex = Math.min(index + chunkSize, array.length);
        
        // Process chunk
        for (let i = index; i < endIndex; i++) {
            let sum = 0;
            for (let j = 0; j < 1000000; j++) {
                sum += Math.random();
            }
            result.push(array[i] * sum);
        }
        
        index = endIndex;
        
        if (index < array.length) {
            // Yield control to event loop
            setImmediate(processChunk);
        } else {
            callback(result);
        }
    }
    
    processChunk();
}
```

**Chunked Processing Visualization:**

```
┌─────────────────────────────────────────────────────────────────┐
│                    Event Loop Timeline                         │
│                                                                 │
│ Chunk 1   │ Other  │ Chunk 2   │ Other  │ Chunk 3   │ Other    │
│ Process   │ Tasks  │ Process   │ Tasks  │ Process   │ Tasks    │
│ 1000 items│ Execute│ 1000 items│ Execute│ 1000 items│ Execute  │
│           │        │           │        │           │          │
│ ▼         ▼        ▼           ▼        ▼           ▼          │
│setImmediate     setImmediate        setImmediate              │
│(yield control)  (yield control)    (yield control)           │
└─────────────────────────────────────────────────────────────────┘
```

### Memory Management and Garbage Collection

**Memory Leak Example:**

```javascript
// ❌ BAD: Memory leak
let cache = {};

function addToCache(key, data) {
    cache[key] = data; // Cache grows indefinitely
}

setInterval(() => {
    addToCache(Date.now(), new Array(1000000).fill('data'));
}, 100);
```

**Memory Usage Over Time:**

```
Memory Usage (MB)
     ▲
 800 │                              ●
 700 │                         ●
 600 │                    ●
 500 │               ●
 400 │          ●
 300 │     ●
 200 │●
 100 │
   0 └──────────────────────────────────► Time
     0   1   2   3   4   5   6   7   8
```

**Fixed Version:**

```javascript
// ✅ GOOD: Bounded cache
const MAX_CACHE_SIZE = 1000;
let cache = new Map();

function addToCache(key, data) {
    if (cache.size >= MAX_CACHE_SIZE) {
        // Remove oldest entry
        const firstKey = cache.keys().next().value;
        cache.delete(firstKey);
    }
    cache.set(key, data);
}
```

---

## Chapter 10: Common Pitfalls and Debugging

### Event Loop Misconceptions

**Misconception 1: setTimeout(0) executes immediately**

```javascript
console.log('1');
setTimeout(() => console.log('2'), 0);
console.log('3');

// Expected by beginners: 1, 2, 3
// Actual output: 1, 3, 2
```

**Why this happens:**

```
┌─────────────────────────────────────────────────────────────────┐
│ setTimeout(0) doesn't mean "execute immediately"               │
│ It means "execute as soon as possible after current code"      │
│                                                                 │
│ Call Stack: [console.log('1'), setTimeout, console.log('3')]   │
│ Macrotask Queue: [() => console.log('2')]                      │
│                                                                 │
│ Event loop only processes macrotasks when call stack is empty  │
└─────────────────────────────────────────────────────────────────┘
```

**Misconception 2: Promises execute immediately**

```javascript
console.log('1');

new Promise((resolve) => {
    console.log('2'); // This executes immediately!
    resolve();
}).then(() => {
    console.log('3'); // This is queued as microtask
});

console.log('4');

// Output: 1, 2, 4, 3
```

**Promise Constructor vs .then():**

```
┌─────────────────┐    ┌─────────────────┐
│ Promise         │    │ .then()         │
│ Constructor     │    │ Callback        │
│                 │    │                 │
│ Executes        │    │ Queued as       │
│ IMMEDIATELY     │───►│ MICROTASK       │
│ (synchronous)   │    │ (asynchronous)  │
└─────────────────┘    └─────────────────┘
```

### Debugging Asynchronous Code

**Problem: Lost Stack Traces**

```javascript
function problematicAsync() {
    setTimeout(() => {
        throw new Error('Something went wrong');
    }, 1000);
}

try {
    problematicAsync();
} catch (error) {
    // This won't catch the error!
    console.log('Caught:', error);
}
```

**Why try/catch doesn't work:**

```
Time 0ms:                    Time 1000ms:
┌─────────────────┐         ┌─────────────────┐
│ Call Stack      │         │ Call Stack      │
│                 │         │                 │
│ try/catch       │         │ Error thrown    │
│ problematicAsync│         │ (no try/catch   │
│                 │  ──►    │  in scope)      │
│                 │ 1000ms  │                 │
└─────────────────┘         └─────────────────┘
Error occurs in different execution context!
```

**Solution: Proper Error Handling**

```javascript
// ✅ GOOD: Handle errors in async context
function properAsyncError(callback) {
    setTimeout(() => {
        try {
            // Risky operation
            throw new Error('Something went wrong');
        } catch (error) {
            callback(error);
        }
    }, 1000);
}

properAsyncError((error) => {
    if (error) {
        console.log('Properly caught:', error.message);
    }
});

// ✅ BETTER: Use Promises
function promiseBasedAsync() {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            reject(new Error('Something went wrong'));
        }, 1000);
    });
}

promiseBasedAsync().catch(error => {
    console.log('Promise caught:', error.message);
});
```

### Performance Debugging Tools

**Measuring Event Loop Lag:**

```javascript
class EventLoopMonitor {
    constructor(threshold = 10) {
        this.threshold = threshold; // milliseconds
        this.start();
    }
    
    start() {
        const startTime = process.hrtime.bigint();
        
        setImmediate(() => {
            const endTime = process.hrtime.bigint();
            const lag = Number(endTime - startTime) / 1000000; // Convert to ms
            
            if (lag > this.threshold) {
                console.warn(`Event loop lag detected: ${lag.toFixed(2)}ms`);
            }
            
            this.start(); // Continue monitoring
        });
    }
}

const monitor = new EventLoopMonitor(5);
```

**Visual Event Loop Lag:**

```
Normal Event Loop:
Time: 0ms    16ms   32ms   48ms
      │      │      │      │
      Task───Task───Task───Task
      ↕16ms  ↕16ms  ↕16ms

Event Loop Lag:
Time: 0ms    16ms   45ms   61ms
      │      │      │      │
      Task───Heavy──Task───Task
      ↕16ms  ↕29ms  ↕16ms
             (Lag!)
```

### Common Anti-patterns

**Anti-pattern 1: Callback Hell**

```javascript
// ❌ BAD: Nested callbacks
getData(function(a) {
    getMoreData(a, function(b) {
        getEvenMoreData(b, function(c) {
            getFinalData(c, function(d) {
                // Finally, use the data
                console.log(d);
            });
        });
    });
});
```

**Callback Hell Visualization:**

```
getData ──┐
          │
          ▼
    getMoreData ──┐
                  │
                  ▼
           getEvenMoreData ──┐
                             │
                             ▼
                      getFinalData ──┐
                                     │
                                     ▼
                               console.log
```

**Solution: Promise Chain**

```javascript
// ✅ GOOD: Promise chain
getData()
    .then(a => getMoreData(a))
    .then(b => getEvenMoreData(b))
    .then(c => getFinalData(c))
    .then(d => console.log(d))
    .catch(error => console.error(error));

// ✅ BETTER: async/await
async function processData() {
    try {
        const a = await getData();
        const b = await getMoreData(a);
        const c = await getEvenMoreData(b);
        const d = await getFinalData(c);
        console.log(d);
    } catch (error) {
        console.error(error);
    }
}
```

**Anti-pattern 2: Mixed Callback/Promise Patterns**

```javascript
// ❌ BAD: Inconsistent async patterns
function mixedAsync(callback) {
    if (someCondition) {
        callback(null, 'result');
    } else {
        return Promise.resolve('promise result');
    }
}
```

**Solution: Consistent Patterns**

```javascript
// ✅ GOOD: Always return promises
function consistentAsync() {
    if (someCondition) {
        return Promise.resolve('result');
    } else {
        return Promise.resolve('promise result');
    }
}

// Or use promisify for callback-based functions
const { promisify } = require('util');
const readFile = promisify(require('fs').readFile);
```

---

## Appendix: Practical Examples and Exercises

### Exercise 1: Predict the Output

**Challenge:** What will this complex code output?

```javascript
console.log('A');

setTimeout(() => console.log('B'), 0);

Promise.resolve().then(() => {
    console.log('C');
    setTimeout(() => console.log('D'), 0);
});

setTimeout(() => {
    console.log('E');
    Promise.resolve().then(() => console.log('F'));
}, 0);

Promise.resolve().then(() => console.log('G'));

console.log('H');
```

**Step-by-step Analysis:**

```
Phase 1 - Synchronous Execution:
┌─────────────────┐
│ Call Stack      │ → Output: A, H
│ console.log('A')│
│ setTimeout(B)   │ → Queues B
│ Promise.then(C) │ → Queues C
│ setTimeout(E)   │ → Queues E
│ Promise.then(G) │ → Queues G
│ console.log('H')│
└─────────────────┘

Queues after Phase 1:
Microtasks: [C, G]
Macrotasks: [B, E]

Phase 2 - Process ALL Microtasks:
Process C: Output C, setTimeout(D) → Queues D
Process G: Output G

Queues after Phase 2:
Microtasks: []
Macrotasks: [B, E, D]

Phase 3 - Process ONE Macrotask:
Process B: Output B

Queues after Phase 3:
Microtasks: []
Macrotasks: [E, D]

Phase 4 - Process ONE Macrotask:
Process E: Output E, Promise.then(F) → Queues F

Queues after Phase 4:
Microtasks: [F]
Macrotasks: [D]

Phase 5 - Process ALL Microtasks:
Process F: Output F

Phase 6 - Process ONE Macrotask:
Process D: Output D

Final Output: A, H, C, G, B, E, F, D
```

### Exercise 2: Build a Task Queue System

**Challenge:** Create a priority-based task queue that respects the event loop.

```javascript
class PriorityTaskQueue {
    constructor() {
        this.queues = {
            immediate: [],    // Highest priority
            normal: [],       // Normal priority
            idle: []         // Lowest priority
        };
        this.isProcessing = false;
    }

    add(task, priority = 'normal') {
        if (!this.queues[priority]) {
            throw new Error(`Invalid priority: ${priority}`);
        }
        
        this.queues[priority].push(task);
        
        if (!this.isProcessing) {
            this.process();
        }
    }

    async process() {
        this.isProcessing = true;
        
        while (this.hasWork()) {
            const task = this.getNextTask();
            
            if (task) {
                try {
                    await this.executeTask(task);
                } catch (error) {
                    console.error('Task failed:', error);
                }
                
                // Yield to event loop after each task
                await this.yieldToEventLoop();
            }
        }
        
        this.isProcessing = false;
    }

    hasWork() {
        return Object.values(this.queues).some(queue => queue.length > 0);
    }

    getNextTask() {
        // Check queues in priority order
        for (const priority of ['immediate', 'normal', 'idle']) {
            if (this.queues[priority].length > 0) {
                return this.queues[priority].shift();
            }
        }
        return null;
    }

    async executeTask(task) {
        if (typeof task === 'function') {
            return await task();
        } else if (task && typeof task.execute === 'function') {
            return await task.execute();
        }
        throw new Error('Invalid task format');
    }

    yieldToEventLoop() {
        return new Promise(resolve => setImmediate(resolve));
    }

    // Get queue status for debugging
    getStatus() {
        return {
            immediate: this.queues.immediate.length,
            normal: this.queues.normal.length,
            idle: this.queues.idle.length,
            isProcessing: this.isProcessing
        };
    }
}

// Usage example:
const taskQueue = new PriorityTaskQueue();

// Add various priority tasks
taskQueue.add(() => console.log('Normal task 1'), 'normal');
taskQueue.add(() => console.log('Urgent task!'), 'immediate');
taskQueue.add(() => console.log('Background task'), 'idle');
taskQueue.add(() => console.log('Normal task 2'), 'normal');

console.log('Queue status:', taskQueue.getStatus());
```

### Exercise 3: Rate Limiter with Token Bucket

**Challenge:** Implement a token bucket rate limiter that plays nicely with the event loop.

```javascript
class TokenBucketRateLimiter {
    constructor(capacity, refillRate, refillInterval = 1000) {
        this.capacity = capacity;           // Maximum tokens
        this.tokens = capacity;             // Current tokens
        this.refillRate = refillRate;       // Tokens per interval
        this.refillInterval = refillInterval; // Milliseconds
        this.queue = [];                    // Waiting requests
        
        this.startRefill();
    }

    startRefill() {
        setInterval(() => {
            this.refill();
            this.processQueue();
        }, this.refillInterval);
    }

    refill() {
        this.tokens = Math.min(this.capacity, this.tokens + this.refillRate);
    }

    async acquire(tokensNeeded = 1) {
        return new Promise((resolve, reject) => {
            if (tokensNeeded > this.capacity) {
                reject(new Error('Requested tokens exceed bucket capacity'));
                return;
            }

            if (this.tokens >= tokensNeeded) {
                this.tokens -= tokensNeeded;
                resolve();
            } else {
                this.queue.push({ tokensNeeded, resolve, reject });
            }
        });
    }

    processQueue() {
        while (this.queue.length > 0) {
            const request = this.queue[0];
            
            if (this.tokens >= request.tokensNeeded) {
                this.tokens -= request.tokensNeeded;
                this.queue.shift();
                
                // Resolve in next tick to avoid blocking
                setImmediate(() => request.resolve());
            } else {
                break; // Not enough tokens, wait for next refill
            }
        }
    }

    getStatus() {
        return {
            tokens: this.tokens,
            capacity: this.capacity,
            queueLength: this.queue.length,
            refillRate: this.refillRate
        };
    }
}

// Usage example:
async function demonstrateRateLimiter() {
    const limiter = new TokenBucketRateLimiter(5, 2, 1000); // 5 capacity, 2/sec refill

    console.log('Initial status:', limiter.getStatus());

    // Simulate API calls
    for (let i = 1; i <= 10; i++) {
        limiter.acquire().then(() => {
            console.log(`API call ${i} executed at ${new Date().toISOString()}`);
        });
    }

    // Monitor status over time
    const monitor = setInterval(() => {
        console.log('Status:', limiter.getStatus());
    }, 500);

    setTimeout(() => clearInterval(monitor), 10000);
}

demonstrateRateLimiter();
```

**Token Bucket Visualization:**

```
Time:  0s    1s    2s    3s    4s    5s
       │     │     │     │     │     │
Tokens:5 ──► 3 ──► 5 ──► 3 ──► 5 ──► 3
       │     │     │     │     │     │
Calls: ██    ██    ██    ██    ██    ██
       (2)   (2)   (2)   (2)   (2)   (2)
       
Bucket: [●●●●●] → [●●●] → [●●●●●] → [●●●]
        (full)   (-2)   (+2)     (-2)
```

### Exercise 4: Event Loop Visualization Tool

**Challenge:** Create a tool to visualize event loop execution order.

```javascript
class EventLoopVisualizer {
    constructor() {
        this.steps = [];
        this.stepCounter = 0;
        this.isRecording = false;
    }

    start() {
        this.steps = [];
        this.stepCounter = 0;
        this.isRecording = true;
        this.log('🚀 Event Loop Visualization Started');
    }

    stop() {
        this.isRecording = false;
        this.log('🏁 Event Loop Visualization Stopped');
        this.printSummary();
    }

    log(message, type = 'sync') {
        if (!this.isRecording) return;

        const step = {
            id: ++this.stepCounter,
            timestamp: Date.now(),
            message,
            type,
            stackDepth: this.getStackDepth()
        };

        this.steps.push(step);
        
        const icon = this.getTypeIcon(type);
        const indent = '  '.repeat(step.stackDepth);
        console.log(`${icon} ${indent}[${step.id}] ${message}`);
    }

    getTypeIcon(type) {
        const icons = {
            sync: '⚡',
            async: '🔄',
            microtask: '🔸',
            macrotask: '🔹',
            timer: '⏰',
            promise: '📝'
        };
        return icons[type] || '❓';
    }

    getStackDepth() {
        const error = new Error();
        const stack = error.stack.split('\n');
        return Math.max(0, stack.length - 10); // Approximate depth
    }

    printSummary() {
        console.log('\n📊 Event Loop Execution Summary:');
        console.log('=' .repeat(50));
        
        const typeGroups = this.groupBy(this.steps, 'type');
        
        Object.entries(typeGroups).forEach(([type, steps]) => {
            console.log(`${this.getTypeIcon(type)} ${type}: ${steps.length} operations`);
        });

        console.log('\n📋 Execution Order:');
        this.steps.forEach(step => {
            const duration = step.id < this.steps.length ? 
                this.steps[step.id].timestamp - step.timestamp : 0;
            console.log(`  ${step.id}. ${step.message} (${step.type})`);
        });
    }

    groupBy(array, key) {
        return array.reduce((groups, item) => {
            const group = item[key];
            groups[group] = groups[group] || [];
            groups[group].push(item);
            return groups;
        }, {});
    }

    // Wrapper methods for common operations
    setTimeout(callback, delay = 0) {
        this.log(`setTimeout scheduled (${delay}ms)`, 'timer');
        return setTimeout(() => {
            this.log('setTimeout callback executing', 'macrotask');
            callback();
        }, delay);
    }

    setImmediate(callback) {
        this.log('setImmediate scheduled', 'timer');
        return setImmediate(() => {
            this.log('setImmediate callback executing', 'macrotask');
            callback();
        });
    }

    promise(executor) {
        this.log('Promise created', 'promise');
        return new Promise((resolve, reject) => {
            executor(
                (value) => {
                    this.log('Promise resolved', 'promise');
                    resolve(value);
                },
                (error) => {
                    this.log('Promise rejected', 'promise');
                    reject(error);
                }
            );
        });
    }

    promiseThen(promise, callback) {
        this.log('Promise .then() scheduled', 'promise');
        return promise.then((value) => {
            this.log('Promise .then() callback executing', 'microtask');
            return callback(value);
        });
    }
}

// Usage example:
function demonstrateEventLoop() {
    const viz = new EventLoopVisualizer();
    
    viz.start();
    
    viz.log('Synchronous operation 1', 'sync');
    
    viz.setTimeout(() => {
        viz.log('Timer callback 1', 'macrotask');
    }, 0);
    
    const promise = viz.promise((resolve) => {
        viz.log('Promise executor', 'sync');
        resolve('Promise value');
    });
    
    viz.promiseThen(promise, (value) => {
        viz.log(`Promise then: ${value}`, 'microtask');
    });
    
    viz.setTimeout(() => {
        viz.log('Timer callback 2', 'macrotask');
        viz.stop();
    }, 10);
    
    viz.log('Synchronous operation 2', 'sync');
}

demonstrateEventLoop();
```

---

## Conclusion

Understanding the JavaScript Event Loop is essential for becoming a proficient JavaScript developer. The Event Loop enables JavaScript to handle asynchronous operations efficiently while maintaining its single-threaded nature.

### Key Takeaways

```
┌─────────────────────────────────────────────────────────────────┐
│                     Essential Concepts                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│ 1. 🧵 JavaScript is single-threaded                            │
│    └─ Uses Event Loop for asynchronous operations              │
│                                                                 │
│ 2. ⚡ Microtasks have higher priority than macrotasks          │
│    └─ Promises, queueMicrotask vs setTimeout, I/O              │
│                                                                 │
│ 3. 📚 Call stack must be empty before queued tasks execute    │
│    └─ Synchronous code always runs first                       │
│                                                                 │
│ 4. 📝 Promises and async/await use microtask queue            │
│    └─ .then(), .catch(), await are microtasks                  │
│                                                                 │
│ 5. ⏰ Timer functions use macrotask queue                      │
│    └─ setTimeout, setInterval minimum delays                    │
│                                                                 │
│ 6. 🔍 Understanding execution order helps debug               │
│    └─ Predict async behavior, trace execution flow             │
│                                                                 │
│ 7. 🚫 Avoid blocking the event loop                           │
│    └─ Use chunking, workers for CPU-intensive tasks            │
└─────────────────────────────────────────────────────────────────┘
```

### The Event Loop Mental Model

Remember this core pattern:

```
while (true) {
    // 1. Execute all synchronous code
    executeSynchronousCode();
    
    // 2. Process ALL microtasks
    while (microtaskQueue.length > 0) {
        microtaskQueue.shift()();
    }
    
    // 3. Process ONE macrotask
    if (macrotaskQueue.length > 0) {
        macrotaskQueue.shift()();
    }
    
    // 4. Render (browser only)
    if (needsRender) {
        render();
    }
}
```

### Performance Principles

1. **Keep the main thread responsive**
2. **Break up long-running tasks**
3. **Use appropriate async patterns**
4. **Monitor event loop health**
5. **Understand your runtime (Browser vs Node.js)**

### Final Visualization: The Complete Picture

```
┌─────────────────────────────────────────────────────────────────┐
│                 JavaScript Runtime Ecosystem                   │
│                                                                 │
│ ┌─────────────────┐  ┌─────────────────────────────────────────┐ │
│ │ Your JavaScript │  │            Event Loop                   │ │
│ │      Code       │  │                                         │ │
│ │                 │  │ 1. Call Stack (Sync Code)              │ │
│ │ function main() │─►│ 2. Microtask Queue (Promises)          │ │
│ │ async/await     │  │ 3. Macrotask Queue (Timers, I/O)       │ │
│ │ promises        │  │ 4. Render (Browser)                    │ │
│ │ timers          │  │                                         │ │
│ └─────────────────┘  └─────────────────────────────────────────┘ │
│                                     │                            │
│                                     ▼                            │
│ ┌─────────────────────────────────────────────────────────────────┐ │
│ │                    Platform APIs                              │ │
│ │                                                               │ │
│ │ Browser:              │  Node.js:                            │ │
│ │ • DOM APIs            │  • File System                       │ │
│ │ • Fetch/XHR           │  • Network                           │ │
│ │ • setTimeout          │  • Process                           │ │
│ │ • requestAnimFrame    │  • Crypto                            │ │
│ └─────────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

By mastering these concepts, you'll be able to write more efficient, responsive JavaScript applications and debug complex asynchronous scenarios with confidence.

**Remember:** The Event Loop is always watching, always coordinating, and always ensuring that JavaScript remains responsive even in a single-threaded world. 🚀