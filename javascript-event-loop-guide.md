# JavaScript Event Loop - Complete Visual Guide

## 🎯 What is the Event Loop?

The **Event Loop** is the heart of JavaScript's asynchronous execution model. It's what allows JavaScript to be **single-threaded** yet handle **multiple operations** simultaneously without blocking the main thread.

**Real-world analogy:** Think of it like a restaurant manager who coordinates between the kitchen (Web APIs), waiters (callback queue), and customers (main thread) to ensure everything runs smoothly without anyone waiting too long.

### 📝 Quick Overview

```
JavaScript Runtime Environment:
┌─────────────────────────────────────────────────────────────┐
│                    JAVASCRIPT ENGINE                        │
│  ┌─────────────────┐    ┌─────────────────┐                │
│  │   CALL STACK    │    │      HEAP       │                │
│  │                 │    │   (Memory)      │                │
│  │  [function3()]  │    │                 │                │
│  │  [function2()]  │    │   Objects &     │                │
│  │  [function1()]  │    │   Variables     │                │
│  │     [main()]    │    │                 │                │
│  └─────────────────┘    └─────────────────┘                │
└─────────────────────────────────────────────────────────────┘
         ↑                           ↑
    ┌────────────┐              ┌──────────────┐
    │ EVENT LOOP │              │   WEB APIs   │
    │            │              │              │
    │ Monitors & │              │ • setTimeout │
    │ Coordinates│              │ • HTTP Req   │
    └────────────┘              │ • DOM Events │
         ↑                      └──────────────┘
    ┌─────────────────────────────────┐
    │        CALLBACK QUEUE           │
    │  [callback3] [callback2] [cb1]  │
    └─────────────────────────────────┘
```

---

## 🧠 Core Components Explained

### 1. 📚 Call Stack
The **execution context** where your code runs. It's a LIFO (Last In, First Out) stack.

```
Call Stack Visualization:
┌─────────────────┐
│                 │ ← Top (currently executing)
│   setTimeout    │   
│   fetchData     │   
│   processUser   │   
│     main()      │ ← Bottom (first function called)
└─────────────────┘
```

**Rules:**
- **Single-threaded:** Only one function executes at a time
- **Synchronous:** Functions execute completely before moving to the next
- **Stack overflow:** Occurs when stack gets too deep

### 2. 🌐 Web APIs
**Browser-provided APIs** that handle asynchronous operations outside the main thread.

```
Web APIs Environment:
┌─────────────────────────────────┐
│           WEB APIs              │
│                                 │
│  ┌─────────────────────────┐    │
│  │     Timer APIs          │    │
│  │  • setTimeout()         │    │
│  │  • setInterval()        │    │
│  └─────────────────────────┘    │
│                                 │
│  ┌─────────────────────────┐    │
│  │     Network APIs        │    │
│  │  • fetch()              │    │
│  │  • XMLHttpRequest       │    │
│  └─────────────────────────┘    │
│                                 │
│  ┌─────────────────────────┐    │
│  │     DOM APIs            │    │
│  │  • addEventListener     │    │
│  │  • DOM manipulation    │    │
│  └─────────────────────────┘    │
└─────────────────────────────────┘
```

### 3. 📬 Callback Queue (Task Queue)
Where **completed async operations** wait to be executed.

```
Callback Queue (FIFO - First In, First Out):
┌─────────────────────────────────────────────────────────┐
│                   CALLBACK QUEUE                        │
│                                                         │
│  [First] ← [Second] ← [Third] ← [Fourth] ← [Last Added] │
│     ↑                                                   │
│   Next to                                               │
│   Execute                                               │
└─────────────────────────────────────────────────────────┘
```

### 4. 🔄 Event Loop
The **coordinator** that monitors the call stack and callback queue.

```
Event Loop Logic:
┌─────────────────────────────────────┐
│            EVENT LOOP               │
│                                     │
│  while (true) {                     │
│    if (callStack.isEmpty()) {       │
│      if (callbackQueue.hasItems()) {│
│        move from queue to stack     │
│      }                              │
│    }                                │
│    // Continue monitoring...        │
│  }                                  │
└─────────────────────────────────────┘
```

---

## 🎬 Step-by-Step Execution Example

Let's trace through this code:

```javascript
console.log('Start');

setTimeout(() => {
    console.log('Timeout callback');
}, 0);

console.log('End');
```

### 📊 Execution Timeline

#### **Step 1: Initial State**
```
Call Stack:        Callback Queue:     Web APIs:
┌─────────────┐    ┌─────────────┐     ┌─────────────┐
│             │    │             │     │             │
│             │    │             │     │             │
│   main()    │    │             │     │             │
└─────────────┘    └─────────────┘     └─────────────┘

Output: (nothing yet)
```

#### **Step 2: Execute `console.log('Start')`**
```
Call Stack:        Callback Queue:     Web APIs:
┌─────────────┐    ┌─────────────┐     ┌─────────────┐
│console.log  │    │             │     │             │
│('Start')    │    │             │     │             │
│   main()    │    │             │     │             │
└─────────────┘    └─────────────┘     └─────────────┘

Output: "Start"
```

#### **Step 3: Execute `setTimeout`**
```
Call Stack:        Callback Queue:     Web APIs:
┌─────────────┐    ┌─────────────┐     ┌─────────────┐
│setTimeout   │    │             │     │Timer(0ms)   │
│             │    │             │     │+ callback   │
│   main()    │    │             │     │             │
└─────────────┘    └─────────────┘     └─────────────┘

Output: "Start"
```

#### **Step 4: setTimeout moves to Web APIs**
```
Call Stack:        Callback Queue:     Web APIs:
┌─────────────┐    ┌─────────────┐     ┌─────────────┐
│             │    │             │     │Timer(0ms)   │
│             │    │             │     │+ callback   │
│   main()    │    │             │     │             │
└─────────────┘    └─────────────┘     └─────────────┘

Output: "Start"
```

#### **Step 5: Execute `console.log('End')`**
```
Call Stack:        Callback Queue:     Web APIs:
┌─────────────┐    ┌─────────────┐     ┌─────────────┐
│console.log  │    │             │     │Timer(0ms)   │
│('End')      │    │             │     │+ callback   │
│   main()    │    │             │     │             │
└─────────────┘    └─────────────┘     └─────────────┘

Output: "Start"
        "End"
```

#### **Step 6: Timer completes, callback moves to queue**
```
Call Stack:        Callback Queue:     Web APIs:
┌─────────────┐    ┌─────────────┐     ┌─────────────┐
│             │    │  callback   │     │             │
│             │    │function()   │     │             │
│   main()    │    │             │     │             │
└─────────────┘    └─────────────┘     └─────────────┘

Output: "Start"
        "End"
```

#### **Step 7: main() finishes, stack becomes empty**
```
Call Stack:        Callback Queue:     Web APIs:
┌─────────────┐    ┌─────────────┐     ┌─────────────┐
│             │    │  callback   │     │             │
│             │    │function()   │     │             │
│             │    │             │     │             │
└─────────────┘    └─────────────┘     └─────────────┘

Output: "Start"
        "End"
```

#### **Step 8: Event Loop moves callback to stack**
```
Call Stack:        Callback Queue:     Web APIs:
┌─────────────┐    ┌─────────────┐     ┌─────────────┐
│  callback   │    │             │     │             │
│ function()  │    │             │     │             │
│             │    │             │     │             │
└─────────────┘    └─────────────┘     └─────────────┘

Output: "Start"
        "End"
        "Timeout callback"
```

### 🎯 **Final Output:**
```
Start
End
Timeout callback
```

---

## 🚀 Advanced Example: Multiple Async Operations

Let's trace a more complex example:

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');
```

### 🔄 **Important:** Microtask Queue vs Callback Queue

JavaScript has **two types of queues**:

```
Priority System:
┌─────────────────────────────────────────────────────────┐
│                   PRIORITY ORDER                        │
│                                                         │
│  1. Call Stack (currently executing)                   │
│  2. Microtask Queue (Promises, queueMicrotask)         │
│  3. Callback Queue (setTimeout, setInterval, events)   │
└─────────────────────────────────────────────────────────┘
```

### 📊 Advanced Execution Timeline

#### **Step 1-3: Synchronous execution**
```
Call Stack:        Microtask Queue:   Callback Queue:    Web APIs:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│console.log  │    │             │    │             │    │             │
│('1')        │    │             │    │             │    │             │
│   main()    │    │             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘

Output: "1"
```

#### **Step 4: setTimeout processed**
```
Call Stack:        Microtask Queue:   Callback Queue:    Web APIs:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│             │    │             │    │             │    │Timer(0ms)   │
│             │    │             │    │             │    │+ callback   │
│   main()    │    │             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘

Output: "1"
```

#### **Step 5: Promise resolves immediately**
```
Call Stack:        Microtask Queue:   Callback Queue:    Web APIs:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│             │    │Promise.then │    │             │    │Timer(0ms)   │
│             │    │callback     │    │             │    │+ callback   │
│   main()    │    │             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘

Output: "1"
```

#### **Step 6: Execute console.log('4')**
```
Call Stack:        Microtask Queue:   Callback Queue:    Web APIs:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│console.log  │    │Promise.then │    │             │    │Timer(0ms)   │
│('4')        │    │callback     │    │             │    │+ callback   │
│   main()    │    │             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘

Output: "1"
        "4"
```

#### **Step 7: Timer completes**
```
Call Stack:        Microtask Queue:   Callback Queue:    Web APIs:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│             │    │Promise.then │    │setTimeout   │    │             │
│             │    │callback     │    │callback     │    │             │
│   main()    │    │             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘

Output: "1"
        "4"
```

#### **Step 8: main() finishes, Event Loop processes Microtasks FIRST**
```
Call Stack:        Microtask Queue:   Callback Queue:    Web APIs:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│Promise.then │    │             │    │setTimeout   │    │             │
│callback     │    │             │    │callback     │    │             │
│             │    │             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘

Output: "1"
        "4"
        "3"
```

#### **Step 9: Finally, process Callback Queue**
```
Call Stack:        Microtask Queue:   Callback Queue:    Web APIs:
┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│setTimeout   │    │             │    │             │    │             │
│callback     │    │             │    │             │    │             │
│             │    │             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘

Output: "1"
        "4"
        "3"
        "2"
```

### 🎯 **Final Output:**
```
1
4
3
2
```

**Key Insight:** Promises (microtasks) have **higher priority** than setTimeout (macrotasks)!

---

## 🛠️ Practical Examples

### Example 1: DOM Event Handling

```javascript
button.addEventListener('click', () => {
    console.log('Button clicked!');
    
    setTimeout(() => {
        console.log('Timeout after click');
    }, 0);
    
    Promise.resolve().then(() => {
        console.log('Promise after click');
    });
    
    console.log('Click handler done');
});

// When button is clicked:
// Output: 
// "Button clicked!"
// "Click handler done"
// "Promise after click"  ← Microtask (higher priority)
// "Timeout after click"  ← Macrotask (lower priority)
```

### Example 2: Fetch API

```javascript
console.log('Start fetching');

fetch('https://api.example.com/data')
    .then(response => response.json())
    .then(data => {
        console.log('Data received:', data);
    });

console.log('Fetch initiated');

// Output:
// "Start fetching"
// "Fetch initiated"
// ... (when response arrives) ...
// "Data received: [data]"
```

### Example 3: Complex Nested Scenarios

```javascript
console.log('Start');

setTimeout(() => {
    console.log('Timeout 1');
    Promise.resolve().then(() => console.log('Promise in Timeout 1'));
}, 0);

Promise.resolve().then(() => {
    console.log('Promise 1');
    setTimeout(() => console.log('Timeout in Promise 1'), 0);
});

setTimeout(() => console.log('Timeout 2'), 0);

Promise.resolve().then(() => console.log('Promise 2'));

console.log('End');

// Output:
// "Start"
// "End"
// "Promise 1"      ← Microtasks first
// "Promise 2"      ← Microtasks first  
// "Timeout 1"      ← Then macrotasks
// "Timeout 2"      ← Then macrotasks
// "Promise in Timeout 1"  ← Microtask created inside macrotask
// "Timeout in Promise 1"  ← Macrotask created inside microtask
```

---

## 🧪 Visual Memory Palace

### The Restaurant Analogy

```
🏪 RESTAURANT ANALOGY:
┌─────────────────────────────────────────────────────────┐
│                    RESTAURANT                           │
│                                                         │
│  👨‍🍳 KITCHEN (Web APIs)                               │
│  ┌─────────────────────────────────────────────────┐   │
│  │ • Timer Orders (setTimeout)                     │   │
│  │ • Online Orders (fetch/AJAX)                    │   │
│  │ • Special Requests (DOM events)                 │   │
│  └─────────────────────────────────────────────────┘   │
│                         ↓                               │
│  📋 ORDER QUEUE (Callback Queue)                       │
│  ┌─────────────────────────────────────────────────┐   │
│  │ [Order 1] → [Order 2] → [Order 3] → [Order 4]  │   │
│  └─────────────────────────────────────────────────┘   │
│                         ↓                               │
│  🍽️ SERVING TABLE (Call Stack)                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │ Currently serving customer...                   │   │
│  │ (Only one customer at a time!)                  │   │
│  └─────────────────────────────────────────────────┘   │
│                                                         │
│  👨‍💼 MANAGER (Event Loop)                            │
│  "When table is empty, serve next order from queue"    │
└─────────────────────────────────────────────────────────┘
```

---

## 🚨 Common Misconceptions

### ❌ **Myth 1:** "setTimeout(fn, 0) executes immediately"
**✅ Reality:** It executes after the current call stack is empty

### ❌ **Myth 2:** "JavaScript is multithreaded"  
**✅ Reality:** JavaScript is single-threaded, but the browser provides multithreaded Web APIs

### ❌ **Myth 3:** "Promises and setTimeout have the same priority"
**✅ Reality:** Promises (microtasks) always execute before setTimeout (macrotasks)

### ❌ **Myth 4:** "Event Loop runs inside JavaScript engine"
**✅ Reality:** Event Loop is part of the JavaScript runtime environment (browser/Node.js)

---

## 🎯 Key Takeaways

1. **🔄 Single Thread:** JavaScript executes one thing at a time
2. **⚡ Non-blocking:** Async operations don't block the main thread  
3. **📋 Priority System:** Microtasks > Macrotasks
4. **🎪 Coordination:** Event Loop orchestrates everything
5. **🌐 Browser Magic:** Web APIs handle the heavy lifting

### 📚 Mental Model Summary

```
🧠 REMEMBER THIS FLOW:
┌─────────────────────────────────────────────────────────┐
│  1. Synchronous code executes first (call stack)       │
│  2. Async operations go to Web APIs                    │
│  3. Completed operations go to queues                  │
│  4. Event Loop waits for empty call stack              │
│  5. Microtasks execute before macrotasks               │
│  6. Process repeats infinitely                         │
└─────────────────────────────────────────────────────────┘
```

## 🏆 Mastery Quiz

Try to predict the output:

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

console.log('G');

// What's the output order?
// Answer: A, G, C, B, E, F, D
```

Now you understand the JavaScript Event Loop! 🎉 This knowledge is crucial for writing efficient, non-blocking JavaScript applications and avoiding common async pitfalls.