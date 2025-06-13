# React Hooks Complete Guide - Detailed Beginner-Friendly Version

## Table of Contents
1. [What Are React Hooks? (Start Here!)](#what-are-hooks)
2. [Why Do We Need Hooks?](#why-hooks)
3. [Basic Hook Rules (Important!)](#hook-rules)
4. [Your First Hook - useState](#first-hook-usestate)
5. [Understanding useEffect](#understanding-useeffect)
6. [Step-by-Step Examples](#step-by-step-examples)
7. [Real-World Problems & Solutions](#real-world-problems)
8. [Building Real Applications](#real-applications)
9. [Advanced Concepts Made Simple](#advanced-concepts)
10. [Common Mistakes & How to Fix Them](#common-mistakes)
11. [Interview Questions with Detailed Explanations](#interview-questions)

---

## What Are React Hooks? (Start Here!) {#what-are-hooks}

### Simple Explanation with Real-World Analogy
Think of React Hooks as **special functions** that let you add superpowers to your React components.

**Real-World Analogy**: Imagine your React component is like a basic smartphone. Without hooks, it's just a phone that can make calls. But with hooks, you can install "apps" (useState, useEffect, etc.) that give your phone new abilities:
- useState = Camera app (lets you capture and store photos/memories)
- useEffect = Alarm app (runs tasks at specific times)
- useContext = GPS app (connects you to external services)
- Custom hooks = Custom apps you build yourself

Just like smartphone apps, hooks follow specific rules about how they can be installed and used.

### React Hooks Explained Like You're 5 Years Old

**What is State?**
Imagine you have a magic box 📦. You can put things in it, look at what's inside, and change what's inside. That's what "state" is in React - it's your magic box for storing information.

**What is useState?**
useState is like having a magic box with a label maker. You can:
1. Put something in the box (initial state)
2. Look at what's in the box anytime (current state)
3. Put something new in the box (update state)
4. The label always shows what's currently inside

```javascript
// It's like saying: "Give me a magic box with 0 cookies inside"
const [cookies, setCookies] = useState(0);

// cookies = how many cookies are in the box right now
// setCookies = the function to change how many cookies are in the box
```

**What is useEffect?**
useEffect is like having a robot friend 🤖 that watches your magic box and does things when:
1. You first get the box (componentDidMount)
2. Something in the box changes (componentDidUpdate)  
3. You throw the box away (componentWillUnmount)

```javascript
// "Robot friend, when the number of cookies changes, 
// please update the sign on my door"
useEffect(() => {
  document.title = `I have ${cookies} cookies!`;
}, [cookies]);
```

**What are Custom Hooks?**
Custom hooks are like teaching your friends how to make their own magic boxes that work exactly like yours. You write the instructions once, and they can use them in their own projects.

### Visual Representation of Hooks

```
🏠 React Component (Your House)
├── 📦 useState (Magic Storage Boxes)
│   ├── Box 1: Counter = 0
│   ├── Box 2: Name = "John"
│   └── Box 3: IsVisible = true
│
├── 🤖 useEffect (Robot Helpers)
│   ├── Robot 1: "Watch the counter box"
│   ├── Robot 2: "Fetch data when name changes"
│   └── Robot 3: "Clean up when leaving house"
│
└── 🎯 Custom Hooks (Special Tools You Made)
    ├── useOnlineStatus (Internet detector)
    ├── useLocalStorage (Memory saver)
    └── useAPI (Data fetcher)
```

**Before Hooks (Class Components):**
```javascript
// This is the OLD way (still works, but more complex)
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 }; // Setting up state
  }

  componentDidMount() {
    // Do something when component loads
    console.log('Component loaded!');
  }

  render() {
    return (
      <div>
        <p>Count: {this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Click me
        </button>
      </div>
    );
  }
}
```

**After Hooks (Function Components):**
```javascript
// This is the NEW way (much simpler!)
function MyComponent() {
  // useState hook gives us state in a function component
  const [count, setCount] = useState(0);

  // useEffect hook lets us do things when component loads
  useEffect(() => {
    console.log('Component loaded!');
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

### What Just Happened?
- **useState(0)** creates a piece of state called `count` starting at 0
- **setCount** is a function that updates the count
- **useEffect** runs code when the component loads (like componentDidMount)
- The component is now much shorter and easier to understand!

---

## Why Do We Need Hooks? {#why-hooks}

### Problem 1: Sharing Logic Between Components

**Imagine this scenario:** You have 3 different components that all need to track whether a user is online or offline.

**Without Hooks (The Hard Way):**
```javascript
// You'd need to create a Higher-Order Component (HOC) - very complex!
function withOnlineStatus(WrappedComponent) {
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.state = { isOnline: navigator.onLine };
    }

    componentDidMount() {
      window.addEventListener('online', this.handleOnline);
      window.addEventListener('offline', this.handleOffline);
    }

    componentWillUnmount() {
      window.removeEventListener('online', this.handleOnline);
      window.removeEventListener('offline', this.handleOffline);
    }

    handleOnline = () => {
      this.setState({ isOnline: true });
    };

    handleOffline = () => {
      this.setState({ isOnline: false });
    };

    render() {
      return <WrappedComponent {...this.props} isOnline={this.state.isOnline} />;
    }
  };
}

// Then you'd wrap each component
const MyComponentWithStatus = withOnlineStatus(MyComponent);
```

**With Hooks (The Easy Way):**
```javascript
// Create a custom hook once
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);

  useEffect(() => {
    // Function to update online status
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);

    // Add event listeners
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);

    // Cleanup function (runs when component unmounts)
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []); // Empty array means "run once when component mounts"

  return isOnline;
}

// Use it in any component - super simple!
function ChatBox() {
  const isOnline = useOnlineStatus(); // Just one line!
  
  return (
    <div>
      {isOnline ? '🟢 Online' : '🔴 Offline'}
      <p>Chat messages here...</p>
    </div>
  );
}

function UserProfile() {
  const isOnline = useOnlineStatus(); // Same hook, different component!
  
  return (
    <div>
      <h1>User Profile</h1>
      <p>Status: {isOnline ? 'Active' : 'Away'}</p>
    </div>
  );
}
```

### Problem 2: Class Components Are Confusing

**Class Component Problems:**
```javascript
class ConfusingComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = { data: null };
    
    // Why do we need to bind this?
    this.handleClick = this.handleClick.bind(this);
  }

  // What's the difference between all these lifecycle methods?
  componentDidMount() {
    this.fetchData();
  }

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchData();
    }
  }

  componentWillUnmount() {
    // Clean up
  }

  handleClick() {
    // Why doesn't 'this' work without binding?
    console.log(this.state.data);
  }

  fetchData() {
    // Fetch logic
  }

  render() {
    return <div onClick={this.handleClick}>{this.state.data}</div>;
  }
}
```

**Hook Component (Much Clearer):**
```javascript
function ClearComponent({ userId }) {
  const [data, setData] = useState(null);

  // One effect handles all data fetching logic
  useEffect(() => {
    async function fetchData() {
      const result = await fetch(`/api/user/${userId}`);
      const userData = await result.json();
      setData(userData);
    }

    fetchData();
  }, [userId]); // Runs when userId changes

  const handleClick = () => {
    console.log(data); // No binding needed!
  };

  return <div onClick={handleClick}>{data}</div>;
}
```

---

## Basic Hook Rules (Important!) {#hook-rules}

### Rule 1: Only Call Hooks at the Top Level

**❌ Wrong:**
```javascript
function BadComponent() {
  const [name, setName] = useState('');

  if (name === '') {
    // ❌ Don't do this! Hook inside condition
    const [error, setError] = useState('Name is required');
  }

  for (let i = 0; i < 5; i++) {
    // ❌ Don't do this! Hook inside loop
    const [item, setItem] = useState(null);
  }

  return <div>{name}</div>;
}
```

**✅ Correct:**
```javascript
function GoodComponent() {
  // ✅ All hooks at the top level
  const [name, setName] = useState('');
  const [error, setError] = useState('');
  const [items, setItems] = useState([]);

  // Use conditions for logic, not hook calls
  useEffect(() => {
    if (name === '') {
      setError('Name is required');
    } else {
      setError('');
    }
  }, [name]);

  return <div>{name}</div>;
}
```

### Rule 2: Only Call Hooks from React Functions

**❌ Wrong:**
```javascript
// ❌ Don't call hooks from regular JavaScript functions
function regularFunction() {
  const [state, setState] = useState(0); // This won't work!
}

// ❌ Don't call hooks from event handlers
function handleClick() {
  const [data, setData] = useState(null); // This won't work!
}
```

**✅ Correct:**
```javascript
// ✅ Call hooks from React components
function MyComponent() {
  const [state, setState] = useState(0);
  return <div>{state}</div>;
}

// ✅ Call hooks from custom hooks
function useMyCustomHook() {
  const [state, setState] = useState(0);
  return [state, setState];
}
```

### Why These Rules Exist

React keeps track of hooks by their **order of calling**. Here's a simplified view:

```javascript
// React internally creates something like this for each component:
const hooksArray = [];
let currentHookIndex = 0;

function useState(initialValue) {
  if (hooksArray[currentHookIndex] === undefined) {
    hooksArray[currentHookIndex] = initialValue;
  }
  
  const state = hooksArray[currentHookIndex];
  const setState = (newValue) => {
    hooksArray[currentHookIndex] = newValue;
    // Trigger re-render
  };
  
  currentHookIndex++;
  return [state, setState];
}
```

If you call hooks conditionally, the order changes, and React gets confused!

---

## Your First Hook - useState {#first-hook-usestate}

Let's start with the most basic hook: `useState`. This hook lets you add state to function components.

### Understanding State

**What is State?**
State is data that can change over time. When state changes, React re-renders your component to show the new data.

Examples of state:
- A counter number
- Whether a modal is open or closed  
- A list of items
- Form input values

### Basic useState Example

```javascript
import React, { useState } from 'react';

function Counter() {
  // useState returns an array with 2 things:
  // 1. The current state value (count)
  // 2. A function to update the state (setCount)
  const [count, setCount] = useState(0);
  //     ↑        ↑           ↑
  //  current  setter    initial value
  //   value  function

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

### Step-by-Step Breakdown

1. **Import useState:** `import { useState } from 'react'`
2. **Call useState:** `const [count, setCount] = useState(0)`
   - This creates a state variable called `count` starting at 0
   - It also creates a function called `setCount` to update the count
3. **Display the state:** `{count}` shows the current value
4. **Update the state:** `onClick={() => setCount(count + 1)}` increases count by 1

### Different Ways to Update State

```javascript
function StateExamples() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  const [isVisible, setIsVisible] = useState(false);

  // Method 1: Set to a new value
  const increment = () => {
    setCount(count + 1);
  };

  // Method 2: Use a function (better for state that depends on previous state)
  const incrementSafely = () => {
    setCount(prevCount => prevCount + 1);
  };

  // Method 3: Toggle boolean
  const toggleVisibility = () => {
    setIsVisible(!isVisible);
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>Add 1</button>
      <button onClick={incrementSafely}>Add 1 (Safe)</button>
      
      <p>Name: {name}</p>
      <input 
        value={name} 
        onChange={(e) => setName(e.target.value)} 
      />
      
      <button onClick={toggleVisibility}>
        {isVisible ? 'Hide' : 'Show'}
      </button>
      {isVisible && <p>I'm visible!</p>}
    </div>
  );
}
```

### Working with Objects and Arrays

```javascript
function ComplexState() {
  // Object state
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });

  // Array state
  const [todos, setTodos] = useState([]);

  // Update object state (must spread existing properties)
  const updateName = (newName) => {
    setUser({
      ...user,        // Keep all existing properties
      name: newName   // Update only the name
    });
  };

  // Add to array
  const addTodo = (todoText) => {
    const newTodo = {
      id: Date.now(),
      text: todoText,
      completed: false
    };
    setTodos([...todos, newTodo]);  // Create new array with new todo
  };

  // Remove from array
  const removeTodo = (todoId) => {
    setTodos(todos.filter(todo => todo.id !== todoId));
  };

  return (
    <div>
      <h3>User Info</h3>
      <input 
        placeholder="Name"
        value={user.name}
        onChange={(e) => updateName(e.target.value)}
      />
      <p>Hello, {user.name}!</p>

      <h3>Todo List</h3>
      <button onClick={() => addTodo('New task')}>
        Add Todo
      </button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            {todo.text}
            <button onClick={() => removeTodo(todo.id)}>
              Remove
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Common useState Mistakes

**❌ Mistake 1: Modifying state directly**
```javascript
function BadExample() {
  const [items, setItems] = useState([1, 2, 3]);

  const addItem = () => {
    items.push(4);        // ❌ Don't modify state directly
    setItems(items);      // ❌ React won't detect the change
  };
}
```

**✅ Correct Way:**
```javascript
function GoodExample() {
  const [items, setItems] = useState([1, 2, 3]);

  const addItem = () => {
    setItems([...items, 4]);  // ✅ Create new array
  };
}
```

**❌ Mistake 2: Not spreading objects**
```javascript
function BadObjectUpdate() {
  const [user, setUser] = useState({ name: 'John', age: 30 });

  const updateAge = () => {
    setUser({ age: 31 });  // ❌ This removes the name property!
  };
}
```

**✅ Correct Way:**
```javascript
function GoodObjectUpdate() {
  const [user, setUser] = useState({ name: 'John', age: 30 });

  const updateAge = () => {
    setUser({ ...user, age: 31 });  // ✅ Keep all properties, update age
  };
}
```

### Advanced useState Patterns and Examples

#### 1. Lazy Initial State
Sometimes calculating the initial state is expensive. You can pass a function to useState instead of a value:

```javascript
function ExpensiveComponent() {
  // ❌ This runs on every render (expensive)
  const [data, setData] = useState(calculateExpensiveValue());

  // ✅ This runs only once (efficient)
  const [data, setData] = useState(() => calculateExpensiveValue());

  // ✅ Also good for localStorage
  const [settings, setSettings] = useState(() => {
    const saved = localStorage.getItem('settings');
    return saved ? JSON.parse(saved) : { theme: 'light', language: 'en' };
  });

  return <div>{data}</div>;
}
```

#### 2. useState with Reducer Pattern
For complex state updates, you can use a reducer-like pattern with useState:

```javascript
function useStateWithReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);
  
  const dispatch = useCallback((action) => {
    setState(prevState => reducer(prevState, action));
  }, [reducer]);
  
  return [state, dispatch];
}

// Usage
function ShoppingCart() {
  const cartReducer = (state, action) => {
    switch (action.type) {
      case 'ADD_ITEM':
        return { ...state, items: [...state.items, action.payload] };
      case 'REMOVE_ITEM':
        return { 
          ...state, 
          items: state.items.filter(item => item.id !== action.payload) 
        };
      default:
        return state;
    }
  };

  const [cart, dispatch] = useStateWithReducer(cartReducer, { items: [] });

  return (
    <div>
      <button onClick={() => dispatch({ 
        type: 'ADD_ITEM', 
        payload: { id: 1, name: 'Apple' } 
      })}>
        Add Apple
      </button>
    </div>
  );
}
```

#### 3. Multiple Related State Variables
Here's how to handle multiple related state variables effectively:

```javascript
function UserProfileForm() {
  // ❌ Separate state for each field (harder to manage)
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);

  // ✅ Group related state together
  const [user, setUser] = useState({
    firstName: '',
    lastName: '',
    email: '',
    age: 0
  });

  // Helper function for updating nested state
  const updateUser = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };

  return (
    <form>
      <input
        value={user.firstName}
        onChange={(e) => updateUser('firstName', e.target.value)}
        placeholder="First Name"
      />
      <input
        value={user.lastName}
        onChange={(e) => updateUser('lastName', e.target.value)}
        placeholder="Last Name"
      />
      <input
        type="email"
        value={user.email}
        onChange={(e) => updateUser('email', e.target.value)}
        placeholder="Email"
      />
      <input
        type="number"
        value={user.age}
        onChange={(e) => updateUser('age', parseInt(e.target.value))}
        placeholder="Age"
      />
    </form>
  );
}
```

#### 4. State with Validation
Combining state with validation logic:

```javascript
function ValidatedInput() {
  const [email, setEmail] = useState('');
  const [emailError, setEmailError] = useState('');

  const validateEmail = (value) => {
    if (!value) {
      return 'Email is required';
    }
    if (!/\S+@\S+\.\S+/.test(value)) {
      return 'Email is invalid';
    }
    return '';
  };

  const handleEmailChange = (newEmail) => {
    setEmail(newEmail);
    setEmailError(validateEmail(newEmail));
  };

  return (
    <div>
      <input
        type="email"
        value={email}
        onChange={(e) => handleEmailChange(e.target.value)}
        style={{ borderColor: emailError ? 'red' : 'gray' }}
      />
      {emailError && <span style={{ color: 'red' }}>{emailError}</span>}
    </div>
  );
}
```

#### 5. Toggle States (Boolean Management)
Common patterns for managing boolean states:

```javascript
function ToggleExamples() {
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [isLoading, setIsLoading] = useState(false);
  const [isDarkMode, setIsDarkMode] = useState(false);

  // Simple toggle function
  const toggleModal = () => setIsModalOpen(!isModalOpen);
  
  // More explicit functions
  const openModal = () => setIsModalOpen(true);
  const closeModal = () => setIsModalOpen(false);
  
  // Toggle with callback pattern
  const toggleDarkMode = () => {
    setIsDarkMode(prevMode => {
      const newMode = !prevMode;
      // Save to localStorage
      localStorage.setItem('darkMode', newMode.toString());
      return newMode;
    });
  };

  return (
    <div>
      <button onClick={toggleModal}>
        {isModalOpen ? 'Close Modal' : 'Open Modal'}
      </button>
      
      <button onClick={toggleDarkMode}>
        Switch to {isDarkMode ? 'Light' : 'Dark'} Mode
      </button>
      
      {isLoading && <div>Loading...</div>}
      {isModalOpen && <Modal onClose={closeModal} />}
    </div>
  );
}
```

#### 6. Array State Management
Comprehensive patterns for managing arrays:

```javascript
function TodoListAdvanced() {
  const [todos, setTodos] = useState([]);
  const [newTodoText, setNewTodoText] = useState('');

  // Add item to array
  const addTodo = () => {
    if (newTodoText.trim()) {
      const newTodo = {
        id: Date.now(),
        text: newTodoText,
        completed: false,
        createdAt: new Date().toISOString()
      };
      setTodos(prevTodos => [...prevTodos, newTodo]);
      setNewTodoText('');
    }
  };

  // Remove item from array
  const removeTodo = (id) => {
    setTodos(prevTodos => prevTodos.filter(todo => todo.id !== id));
  };

  // Update item in array
  const toggleTodo = (id) => {
    setTodos(prevTodos =>
      prevTodos.map(todo =>
        todo.id === id ? { ...todo, completed: !todo.completed } : todo
      )
    );
  };

  // Edit item in array
  const editTodo = (id, newText) => {
    setTodos(prevTodos =>
      prevTodos.map(todo =>
        todo.id === id ? { ...todo, text: newText } : todo
      )
    );
  };

  // Reorder items (drag and drop)
  const moveTodo = (fromIndex, toIndex) => {
    setTodos(prevTodos => {
      const newTodos = [...prevTodos];
      const [movedTodo] = newTodos.splice(fromIndex, 1);
      newTodos.splice(toIndex, 0, movedTodo);
      return newTodos;
    });
  };

  // Filter/sort without modifying state
  const completedTodos = todos.filter(todo => todo.completed);
  const pendingTodos = todos.filter(todo => !todo.completed);
  const sortedTodos = [...todos].sort((a, b) => 
    new Date(b.createdAt) - new Date(a.createdAt)
  );

  return (
    <div>
      <input
        value={newTodoText}
        onChange={(e) => setNewTodoText(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
      />
      <button onClick={addTodo}>Add Todo</button>
      
      <div>
        <h3>Pending ({pendingTodos.length})</h3>
        {pendingTodos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={() => toggleTodo(todo.id)}
            onRemove={() => removeTodo(todo.id)}
            onEdit={(newText) => editTodo(todo.id, newText)}
          />
        ))}
      </div>
      
      <div>
        <h3>Completed ({completedTodos.length})</h3>
        {completedTodos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={() => toggleTodo(todo.id)}
            onRemove={() => removeTodo(todo.id)}
            onEdit={(newText) => editTodo(todo.id, newText)}
          />
        ))}
      </div>
    </div>
  );
}
```

#### 7. useState with TypeScript (if applicable)
If you're using TypeScript, here are type-safe useState patterns:

```typescript
// Basic types
const [count, setCount] = useState<number>(0);
const [name, setName] = useState<string>('');
const [isVisible, setIsVisible] = useState<boolean>(false);

// Object types
interface User {
  id: number;
  name: string;
  email: string;
}

const [user, setUser] = useState<User | null>(null);
const [users, setUsers] = useState<User[]>([]);

// Union types
type Status = 'idle' | 'loading' | 'success' | 'error';
const [status, setStatus] = useState<Status>('idle');

// Optional with default
const [settings, setSettings] = useState<Partial<Settings>>({});
```

---

## Understanding useEffect {#understanding-useeffect}

`useEffect` is like a Swiss Army knife for handling "side effects" in your components.

### What are Side Effects?

Side effects are things that happen "outside" of rendering:
- Fetching data from an API
- Setting up event listeners
- Updating the document title
- Starting/stopping timers
- Cleaning up resources

### Basic useEffect

```javascript
import React, { useState, useEffect } from 'react';

function BasicEffect() {
  const [count, setCount] = useState(0);

  // This runs after every render
  useEffect(() => {
    document.title = `Count: ${count}`;
  });

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  );
}
```

### useEffect with Dependencies

```javascript
function EffectWithDependencies() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // Runs after every render
  useEffect(() => {
    console.log('This runs after every render');
  });

  // Runs only once after the first render (like componentDidMount)
  useEffect(() => {
    console.log('This runs only once');
  }, []); // Empty dependency array

  // Runs only when count changes
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]); // Only re-run when count changes

  // Runs when either count OR name changes
  useEffect(() => {
    console.log(`Count: ${count}, Name: ${name}`);
  }, [count, name]); // Re-run when count or name changes

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      
      <input 
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter name"
      />
    </div>
  );
}
```

### useEffect Dependency Array Explained

```javascript
// Visual representation of how dependencies work:

useEffect(() => {
  // Your code here
}, [dependency1, dependency2]);

/*
How React checks dependencies:

Render 1: [1, 'hello']
Render 2: [1, 'hello']  → Same values, effect doesn't run
Render 3: [2, 'hello']  → dependency1 changed, effect runs
Render 4: [2, 'world']  → dependency2 changed, effect runs
*/
```

### Cleanup with useEffect

Some effects need cleanup to prevent memory leaks:

```javascript
function TimerComponent() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    // Set up the timer
    const timer = setInterval(() => {
      setSeconds(prevSeconds => prevSeconds + 1);
    }, 1000);

    // Cleanup function (very important!)
    return () => {
      clearInterval(timer);  // Clear the timer when component unmounts
    };
  }, []); // Empty array means this effect runs once

  return <div>Seconds: {seconds}</div>;
}
```

### Data Fetching with useEffect

```javascript
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    // Reset states when userId changes
    setLoading(true);
    setError(null);

    // Async function inside useEffect
    async function fetchUser() {
      try {
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
          throw new Error('Failed to fetch user');
        }
        
        const userData = await response.json();
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    }

    fetchUser();
  }, [userId]); // Re-run when userId changes

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!user) return <div>No user found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>Email: {user.email}</p>
    </div>
  );
}
```

### Event Listeners with useEffect

```javascript
function WindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });

  useEffect(() => {
    // Function to update size
    function updateSize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    }

    // Add event listener
    window.addEventListener('resize', updateSize);

    // Cleanup: remove event listener
    return () => {
      window.removeEventListener('resize', updateSize);
    };
  }, []); // Empty dependency array

  return (
    <div>
      <p>Width: {windowSize.width}px</p>
      <p>Height: {windowSize.height}px</p>
    </div>
  );
}
```

### Advanced useEffect Patterns and Common Use Cases

#### 1. Conditional Effects
Sometimes you want effects to run only under certain conditions:

```javascript
function ConditionalEffect({ shouldFetch, userId }) {
  const [data, setData] = useState(null);

  useEffect(() => {
    // Only run effect if shouldFetch is true
    if (!shouldFetch) return;

    const fetchData = async () => {
      try {
        const response = await fetch(`/api/user/${userId}`);
        const userData = await response.json();
        setData(userData);
      } catch (error) {
        console.error('Failed to fetch:', error);
      }
    };

    fetchData();
  }, [shouldFetch, userId]);

  return <div>{data ? data.name : 'No data'}</div>;
}
```

#### 2. Multiple Effects for Different Concerns
Separate different concerns into different effects:

```javascript
function UserDashboard({ userId }) {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [notifications, setNotifications] = useState([]);

  // Effect 1: Fetch user data
  useEffect(() => {
    const fetchUser = async () => {
      const response = await fetch(`/api/users/${userId}`);
      const userData = await response.json();
      setUser(userData);
    };

    if (userId) {
      fetchUser();
    }
  }, [userId]);

  // Effect 2: Fetch posts
  useEffect(() => {
    const fetchPosts = async () => {
      const response = await fetch(`/api/users/${userId}/posts`);
      const postsData = await response.json();
      setPosts(postsData);
    };

    if (userId) {
      fetchPosts();
    }
  }, [userId]);

  // Effect 3: Set up real-time notifications
  useEffect(() => {
    if (!userId) return;

    const socket = new WebSocket(`ws://localhost/notifications/${userId}`);
    
    socket.onmessage = (event) => {
      const notification = JSON.parse(event.data);
      setNotifications(prev => [notification, ...prev]);
    };

    return () => {
      socket.close();
    };
  }, [userId]);

  // Effect 4: Update document title
  useEffect(() => {
    if (user) {
      document.title = `${user.name}'s Dashboard`;
    }

    return () => {
      document.title = 'My App';
    };
  }, [user]);

  return (
    <div>
      {user && <h1>Welcome, {user.name}!</h1>}
      <div>Posts: {posts.length}</div>
      <div>Notifications: {notifications.length}</div>
    </div>
  );
}
```

#### 3. Debounced Effects (Search as you type)
Delay effect execution until user stops typing:

```javascript
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [isLoading, setIsLoading] = useState(false);

  // Debounced search effect
  useEffect(() => {
    if (!query.trim()) {
      setResults([]);
      return;
    }

    setIsLoading(true);

    // Create a timeout to delay the search
    const timeoutId = setTimeout(async () => {
      try {
        const response = await fetch(`/api/search?q=${encodeURIComponent(query)}`);
        const searchResults = await response.json();
        setResults(searchResults);
      } catch (error) {
        console.error('Search failed:', error);
        setResults([]);
      } finally {
        setIsLoading(false);
      }
    }, 500); // 500ms delay

    // Cleanup: cancel previous timeout
    return () => {
      clearTimeout(timeoutId);
    };
  }, [query]);

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Search..."
      />
      
      {isLoading && <div>Searching...</div>}
      
      <ul>
        {results.map(result => (
          <li key={result.id}>{result.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

#### 4. Polling/Interval Effects
Fetch data at regular intervals:

```javascript
function LiveData() {
  const [data, setData] = useState(null);
  const [isPolling, setIsPolling] = useState(true);

  useEffect(() => {
    if (!isPolling) return;

    const fetchData = async () => {
      try {
        const response = await fetch('/api/live-data');
        const newData = await response.json();
        setData(newData);
      } catch (error) {
        console.error('Failed to fetch live data:', error);
      }
    };

    // Fetch immediately
    fetchData();

    // Set up polling
    const intervalId = setInterval(fetchData, 5000); // Every 5 seconds

    return () => {
      clearInterval(intervalId);
    };
  }, [isPolling]);

  return (
    <div>
      <button onClick={() => setIsPolling(!isPolling)}>
        {isPolling ? 'Stop Polling' : 'Start Polling'}
      </button>
      
      <div>
        Last updated: {data ? new Date(data.timestamp).toLocaleTimeString() : 'Never'}
      </div>
      
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
}
```

#### 5. Cleanup Complex Subscriptions
Handle complex cleanup scenarios:

```javascript
function ComplexSubscription({ topic }) {
  const [messages, setMessages] = useState([]);
  const [connectionStatus, setConnectionStatus] = useState('disconnected');

  useEffect(() => {
    let socket;
    let heartbeatInterval;
    let reconnectTimeout;

    const connect = () => {
      setConnectionStatus('connecting');
      socket = new WebSocket(`ws://localhost/topics/${topic}`);

      socket.onopen = () => {
        setConnectionStatus('connected');
        
        // Start heartbeat
        heartbeatInterval = setInterval(() => {
          if (socket.readyState === WebSocket.OPEN) {
            socket.send(JSON.stringify({ type: 'ping' }));
          }
        }, 30000);
      };

      socket.onmessage = (event) => {
        const message = JSON.parse(event.data);
        if (message.type !== 'pong') {
          setMessages(prev => [...prev, message]);
        }
      };

      socket.onclose = () => {
        setConnectionStatus('disconnected');
        
        // Clear heartbeat
        if (heartbeatInterval) {
          clearInterval(heartbeatInterval);
        }

        // Attempt reconnection after 5 seconds
        reconnectTimeout = setTimeout(connect, 5000);
      };

      socket.onerror = (error) => {
        console.error('WebSocket error:', error);
        setConnectionStatus('error');
      };
    };

    connect();

    // Cleanup function
    return () => {
      if (socket) {
        socket.close();
      }
      if (heartbeatInterval) {
        clearInterval(heartbeatInterval);
      }
      if (reconnectTimeout) {
        clearTimeout(reconnectTimeout);
      }
    };
  }, [topic]);

  return (
    <div>
      <div>Status: {connectionStatus}</div>
      <div>Messages: {messages.length}</div>
      <ul>
        {messages.slice(-10).map((msg, index) => (
          <li key={index}>{msg.content}</li>
        ))}
      </ul>
    </div>
  );
}
```

#### 6. Effect with Async/Await Patterns
Proper async handling in useEffect:

```javascript
function AsyncDataComponent({ id }) {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);

  useEffect(() => {
    let isCancelled = false; // Flag to prevent state updates if component unmounts

    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);

        // Multiple async operations
        const [userData, userPosts, userSettings] = await Promise.all([
          fetch(`/api/users/${id}`).then(res => res.json()),
          fetch(`/api/users/${id}/posts`).then(res => res.json()),
          fetch(`/api/users/${id}/settings`).then(res => res.json())
        ]);

        // Only update state if component is still mounted
        if (!isCancelled) {
          setData({
            user: userData,
            posts: userPosts,
            settings: userSettings
          });
        }
      } catch (err) {
        if (!isCancelled) {
          setError(err.message);
        }
      } finally {
        if (!isCancelled) {
          setLoading(false);
        }
      }
    };

    if (id) {
      fetchData();
    }

    // Cleanup function
    return () => {
      isCancelled = true;
    };
  }, [id]);

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  if (!data) return <div>No data</div>;

  return (
    <div>
      <h2>{data.user.name}</h2>
      <p>Posts: {data.posts.length}</p>
      <p>Theme: {data.settings.theme}</p>
    </div>
  );
}
```

#### 7. Local Storage Sync Effect
Sync state with localStorage:

```javascript
function useLocalStorage(key, initialValue) {
  // Get value from localStorage or use initialValue
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error('Error reading from localStorage:', error);
      return initialValue;
    }
  });

  // Effect to sync with localStorage
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error('Error writing to localStorage:', error);
    }
  }, [key, storedValue]);

  // Listen for localStorage changes from other tabs
  useEffect(() => {
    const handleStorageChange = (e) => {
      if (e.key === key && e.newValue !== null) {
        try {
          setStoredValue(JSON.parse(e.newValue));
        } catch (error) {
          console.error('Error parsing localStorage value:', error);
        }
      }
    };

    window.addEventListener('storage', handleStorageChange);
    return () => window.removeEventListener('storage', handleStorageChange);
  }, [key]);

  return [storedValue, setStoredValue];
}

// Usage
function UserPreferences() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [language, setLanguage] = useLocalStorage('language', 'en');

  return (
    <div>
      <select value={theme} onChange={(e) => setTheme(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
      
      <select value={language} onChange={(e) => setLanguage(e.target.value)}>
        <option value="en">English</option>
        <option value="es">Spanish</option>
        <option value="fr">French</option>
      </select>
    </div>
  );
}
```

#### 8. Effect Dependencies Deep Dive
Understanding when effects run:

```javascript
function EffectDependencies() {
  const [count, setCount] = useState(0);
  const [user, setUser] = useState({ name: 'John', age: 30 });
  const [items, setItems] = useState([1, 2, 3]);

  // ❌ This will run on every render (infinite loop potential)
  useEffect(() => {
    console.log('Runs every render');
  });

  // ✅ Runs only once after mount
  useEffect(() => {
    console.log('Component mounted');
  }, []);

  // ✅ Runs when count changes
  useEffect(() => {
    console.log('Count changed:', count);
  }, [count]);

  // ❌ Object/array dependencies cause issues
  useEffect(() => {
    console.log('User changed:', user);
  }, [user]); // user object changes reference on every render

  // ✅ Depend on specific properties
  useEffect(() => {
    console.log('User name or age changed');
  }, [user.name, user.age]);

  // ❌ Array dependency issue
  useEffect(() => {
    console.log('Items changed:', items);
  }, [items]); // items array changes reference

  // ✅ Use useMemo for stable references
  const itemsString = useMemo(() => JSON.stringify(items), [items]);
  useEffect(() => {
    console.log('Items actually changed');
  }, [itemsString]);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <button onClick={() => setUser({ ...user, age: user.age + 1 })}>
        Age: {user.age}
      </button>
      <button onClick={() => setItems([...items, items.length + 1])}>
        Add Item
      </button>
    </div>
  );
}
```

---

## Step-by-Step Examples {#step-by-step-examples}

Let's build real components step by step to see how hooks work in practice.

### Example 1: Building a Todo App

**Step 1: Basic Structure**
```javascript
function TodoApp() {
  return (
    <div>
      <h1>My Todo App</h1>
      <input placeholder="Enter a task..." />
      <button>Add Task</button>
      <ul>
        <li>Sample task</li>
      </ul>
    </div>
  );
}
```

**Step 2: Add State for Todos**
```javascript
function TodoApp() {
  // State to store all todos
  const [todos, setTodos] = useState([]);
  
  return (
    <div>
      <h1>My Todo App</h1>
      <input placeholder="Enter a task..." />
      <button>Add Task</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Step 3: Add State for Input**
```javascript
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState(''); // State for input field

  return (
    <div>
      <h1>My Todo App</h1>
      <input 
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="Enter a task..." 
      />
      <button>Add Task</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Step 4: Add Functionality to Add Todos**
```javascript
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState('');

  // Function to add a new todo
  const addTodo = () => {
    if (inputValue.trim() !== '') { // Check if input is not empty
      const newTodo = {
        id: Date.now(), // Simple ID generation
        text: inputValue,
        completed: false
      };
      
      setTodos([...todos, newTodo]); // Add new todo to existing list
      setInputValue(''); // Clear input field
    }
  };

  return (
    <div>
      <h1>My Todo App</h1>
      <input 
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="Enter a task..." 
      />
      <button onClick={addTodo}>Add Task</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </div>
  );
}
```

**Step 5: Add Delete Functionality**
```javascript
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState('');

  const addTodo = () => {
    if (inputValue.trim() !== '') {
      const newTodo = {
        id: Date.now(),
        text: inputValue,
        completed: false
      };
      setTodos([...todos, newTodo]);
      setInputValue('');
    }
  };

  // Function to delete a todo
  const deleteTodo = (todoId) => {
    setTodos(todos.filter(todo => todo.id !== todoId));
  };

  return (
    <div>
      <h1>My Todo App</h1>
      <input 
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="Enter a task..." 
      />
      <button onClick={addTodo}>Add Task</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            {todo.text}
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

**Step 6: Add Toggle Complete Functionality**
```javascript
function TodoApp() {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState('');

  const addTodo = () => {
    if (inputValue.trim() !== '') {
      const newTodo = {
        id: Date.now(),
        text: inputValue,
        completed: false
      };
      setTodos([...todos, newTodo]);
      setInputValue('');
    }
  };

  const deleteTodo = (todoId) => {
    setTodos(todos.filter(todo => todo.id !== todoId));
  };

  // Function to toggle completion status
  const toggleTodo = (todoId) => {
    setTodos(todos.map(todo => 
      todo.id === todoId 
        ? { ...todo, completed: !todo.completed }
        : todo
    ));
  };

  return (
    <div>
      <h1>My Todo App</h1>
      <input 
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        placeholder="Enter a task..." 
      />
      <button onClick={addTodo}>Add Task</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input 
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span style={{ 
              textDecoration: todo.completed ? 'line-through' : 'none' 
            }}>
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### Example 2: Building a Weather App with API

**Step 1: Basic Structure**
```javascript
function WeatherApp() {
  return (
    <div>
      <h1>Weather App</h1>
      <input placeholder="Enter city name..." />
      <button>Get Weather</button>
      <div>Weather info will appear here</div>
    </div>
  );
}
```

**Step 2: Add State**
```javascript
function WeatherApp() {
  const [city, setCity] = useState('');
  const [weather, setWeather] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  return (
    <div>
      <h1>Weather App</h1>
      <input 
        value={city}
        onChange={(e) => setCity(e.target.value)}
        placeholder="Enter city name..." 
      />
      <button>Get Weather</button>
      
      {loading && <div>Loading...</div>}
      {error && <div>Error: {error}</div>}
      {weather && (
        <div>
          <h2>{weather.name}</h2>
          <p>Temperature: {weather.main.temp}°C</p>
          <p>Description: {weather.weather[0].description}</p>
        </div>
      )}
    </div>
  );
}
```

**Step 3: Add API Fetch Function**
```javascript
function WeatherApp() {
  const [city, setCity] = useState('');
  const [weather, setWeather] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  // Function to fetch weather data
  const fetchWeather = async () => {
    if (!city.trim()) return; // Don't fetch if city is empty

    setLoading(true);
    setError(null);
    setWeather(null);

    try {
      const API_KEY = 'your-api-key-here';
      const response = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric`
      );

      if (!response.ok) {
        throw new Error('City not found');
      }

      const data = await response.json();
      setWeather(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <h1>Weather App</h1>
      <input 
        value={city}
        onChange={(e) => setCity(e.target.value)}
        placeholder="Enter city name..." 
      />
      <button onClick={fetchWeather}>Get Weather</button>
      
      {loading && <div>Loading...</div>}
      {error && <div>Error: {error}</div>}
      {weather && (
        <div>
          <h2>{weather.name}</h2>
          <p>Temperature: {weather.main.temp}°C</p>
          <p>Description: {weather.weather[0].description}</p>
        </div>
      )}
    </div>
  );
}
```

**Step 4: Add Enter Key Support**
```javascript
function WeatherApp() {
  const [city, setCity] = useState('');
  const [weather, setWeather] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchWeather = async () => {
    if (!city.trim()) return;

    setLoading(true);
    setError(null);
    setWeather(null);

    try {
      const API_KEY = 'your-api-key-here';
      const response = await fetch(
        `https://api.openweathermap.org/data/2.5/weather?q=${city}&appid=${API_KEY}&units=metric`
      );

      if (!response.ok) {
        throw new Error('City not found');
      }

      const data = await response.json();
      setWeather(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  // Handle Enter key press
  const handleKeyPress = (e) => {
    if (e.key === 'Enter') {
      fetchWeather();
    }
  };

  return (
    <div>
      <h1>Weather App</h1>
      <input 
        value={city}
        onChange={(e) => setCity(e.target.value)}
        onKeyPress={handleKeyPress}
        placeholder="Enter city name..." 
      />
      <button onClick={fetchWeather} disabled={loading}>
        {loading ? 'Loading...' : 'Get Weather'}
      </button>
      
      {error && <div style={{ color: 'red' }}>Error: {error}</div>}
      {weather && (
        <div style={{ marginTop: '20px', padding: '20px', border: '1px solid #ccc' }}>
          <h2>{weather.name}, {weather.sys.country}</h2>
          <p><strong>Temperature:</strong> {Math.round(weather.main.temp)}°C</p>
          <p><strong>Feels like:</strong> {Math.round(weather.main.feels_like)}°C</p>
          <p><strong>Description:</strong> {weather.weather[0].description}</p>
          <p><strong>Humidity:</strong> {weather.main.humidity}%</p>
          <p><strong>Wind Speed:</strong> {weather.wind.speed} m/s</p>
        </div>
      )}
    </div>
  );
}
```

### Visual Component Lifecycle

```
React Component Lifecycle with Hooks

MOUNT (Component appears on screen)
┌─────────────────────────────────────┐
│ 1. Component function runs          │
│ 2. useState sets initial state      │
│ 3. Component renders (returns JSX)  │
│ 4. useEffect with [] runs (mount)   │
└─────────────────────────────────────┘
                 │
                 ▼
UPDATE (State or props change)
┌─────────────────────────────────────┐
│ 1. State/props change               │
│ 2. Component function runs again    │
│ 3. useState returns current state   │
│ 4. Component re-renders             │
│ 5. useEffect with deps runs         │
└─────────────────────────────────────┘
                 │
                 ▼
UNMOUNT (Component removed from screen)
┌─────────────────────────────────────┐
│ 1. useEffect cleanup functions run  │
│ 2. Component is removed from DOM    │
└─────────────────────────────────────┘
```

### Hook Execution Order

```javascript
function ExampleComponent() {
  console.log('1. Component function starts');
  
  const [count, setCount] = useState(0);
  console.log('2. useState called');
  
  const [name, setName] = useState('');
  console.log('3. Second useState called');
  
  useEffect(() => {
    console.log('5. useEffect (no deps) runs after render');
  });
  
  useEffect(() => {
    console.log('6. useEffect (empty deps) runs once after mount');
  }, []);
  
  useEffect(() => {
    console.log('7. useEffect (with deps) runs when count changes');
  }, [count]);
  
  console.log('4. About to return JSX');
  
  return <div>Count: {count}</div>;
}

/*
Console output on first render:
1. Component function starts
2. useState called
3. Second useState called
4. About to return JSX
5. useEffect (no deps) runs after render
6. useEffect (empty deps) runs once after mount
7. useEffect (with deps) runs when count changes

Console output on subsequent renders (when count changes):
1. Component function starts
2. useState called
3. Second useState called
4. About to return JSX
5. useEffect (no deps) runs after render
7. useEffect (with deps) runs when count changes
*/
```
│                    MOUNTING PHASE                           │
├─────────────────────────────────────────────────────────────┤
│ 1. useState() - Initialize state                            │
│ 2. useEffect(() => {}, []) - Mount effect (componentDidMount)│
│ 3. useContext() - Subscribe to context                     │
│ 4. Custom hooks - Initialize custom logic                  │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                    UPDATING PHASE                           │
├─────────────────────────────────────────────────────────────┤
│ 1. State changes trigger re-render                         │
│ 2. useEffect(() => {}, [deps]) - Effect with dependencies  │
│ 3. useMemo() - Memoized computations                       │
│ 4. useCallback() - Memoized functions                      │
│ 5. Re-run custom hooks                                     │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                   UNMOUNTING PHASE                          │
├─────────────────────────────────────────────────────────────┤
│ 1. useEffect cleanup functions                             │
│ 2. Custom hook cleanup                                     │
│ 3. Context unsubscription                                  │
└─────────────────────────────────────────────────────────────┘
```

## Core Hooks Overview {#core-hooks}

### State Hook Flow
```
useState Hook Flow
┌──────────────┐    ┌──────────────┐    ┌──────────────┐
│ Initial State│───▶│Current State │───▶│ State Setter │
│   (or lazy)  │    │   (value)    │    │ (function)   │
└──────────────┘    └──────────────┘    └──────────────┘
                            │                    │
                            │                    ▼
                            │            ┌──────────────┐
                            │            │ Trigger      │
                            │◀───────────│ Re-render    │
                            │            └──────────────┘
                            ▼
                    ┌──────────────┐
                    │ Updated State│
                    └──────────────┘
```

### Effect Hook Dependencies
```
useEffect Dependency Comparison
┌─────────────────┐
│ Previous Deps   │
│ [a, b, c]       │
└─────────────────┘
         │
         ▼
┌─────────────────┐    ┌─────────────┐    ┌─────────────┐
│ Current Deps    │───▶│ Compare     │───▶│ Run Effect? │
│ [a, b, c']      │    │ Each Dep    │    │ Yes/No      │
└─────────────────┘    └─────────────┘    └─────────────┘
```

## Problem Statements & Solutions {#problems-solutions}

### Problem 1: Sharing Stateful Logic Between Components

**Problem**: Multiple components need the same stateful logic (e.g., form validation, API calls).

**Class Component Solution** (Complex):
```javascript
// Higher-Order Component approach
const withCounter = (WrappedComponent) => {
  return class extends React.Component {
    state = { count: 0 };
    increment = () => this.setState({ count: this.state.count + 1 });
    render() {
      return <WrappedComponent {...this.props} {...this.state} increment={this.increment} />;
    }
  };
};
```

**Hook Solution** (Simple):
```javascript
// Custom Hook
const useCounter = (initialValue = 0) => {
  const [count, setCount] = useState(initialValue);
  const increment = useCallback(() => setCount(c => c + 1), []);
  const decrement = useCallback(() => setCount(c => c - 1), []);
  const reset = useCallback(() => setCount(initialValue), [initialValue]);
  
  return { count, increment, decrement, reset };
};

// Usage in components
const Counter1 = () => {
  const { count, increment } = useCounter(0);
  return <button onClick={increment}>Count: {count}</button>;
};

const Counter2 = () => {
  const { count, increment } = useCounter(10);
  return <button onClick={increment}>Count: {count}</button>;
};
```

### Problem 2: Complex Component Lifecycle Management

**Problem**: Components with multiple lifecycle methods become hard to maintain.

**Class Component Solution**:
```javascript
class UserProfile extends React.Component {
  state = { user: null, posts: [], loading: true };

  componentDidMount() {
    this.fetchUser();
    this.fetchPosts();
    window.addEventListener('resize', this.handleResize);
  }

  componentDidUpdate(prevProps) {
    if (prevProps.userId !== this.props.userId) {
      this.fetchUser();
      this.fetchPosts();
    }
  }

  componentWillUnmount() {
    window.removeEventListener('resize', this.handleResize);
  }

  fetchUser = async () => {
    // fetch logic
  };

  fetchPosts = async () => {
    // fetch logic
  };

  handleResize = () => {
    // resize logic
  };

  render() {
    // render logic
  }
}
```

**Hook Solution**:
```javascript
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState(null);
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(true);

  // User data effect
  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      try {
        const userData = await api.getUser(userId);
        setUser(userData);
      } catch (error) {
        console.error('Failed to fetch user:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  // Posts data effect
  useEffect(() => {
    const fetchPosts = async () => {
      try {
        const postsData = await api.getUserPosts(userId);
        setPosts(postsData);
      } catch (error) {
        console.error('Failed to fetch posts:', error);
      }
    };

    if (userId) {
      fetchPosts();
    }
  }, [userId]);

  // Resize event effect
  useEffect(() => {
    const handleResize = () => {
      // resize logic
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  if (loading) return <div>Loading...</div>;
  return (
    <div>
      <UserDetails user={user} />
      <PostsList posts={posts} />
    </div>
  );
};
```

### Problem 3: Performance Optimization

**Problem**: Unnecessary re-renders and expensive calculations.

**Solution with useMemo and useCallback**:
```javascript
const ExpensiveComponent = ({ items, filter, onItemClick }) => {
  // Memoize expensive calculation
  const filteredItems = useMemo(() => {
    console.log('Filtering items...'); // This will only run when items or filter change
    return items.filter(item => 
      item.name.toLowerCase().includes(filter.toLowerCase())
    );
  }, [items, filter]);

  // Memoize event handler to prevent child re-renders
  const handleItemClick = useCallback((item) => {
    onItemClick(item.id);
  }, [onItemClick]);

  return (
    <div>
      {filteredItems.map(item => (
        <ExpensiveItem 
          key={item.id} 
          item={item} 
          onClick={handleItemClick}
        />
      ))}
    </div>
  );
};

// Child component wrapped with React.memo
const ExpensiveItem = React.memo(({ item, onClick }) => {
  console.log('Rendering item:', item.id);
  return (
    <div onClick={() => onClick(item)}>
      {item.name}
    </div>
  );
});
```

## Real-time Scenarios {#real-time-scenarios}

### Scenario 1: Real-time Chat Application

```javascript
const useChatRoom = (roomId) => {
  const [messages, setMessages] = useState([]);
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    const socket = io(`/chat/${roomId}`);
    
    socket.on('connect', () => {
      setIsConnected(true);
    });

    socket.on('disconnect', () => {
      setIsConnected(false);
    });

    socket.on('message', (message) => {
      setMessages(prev => [...prev, message]);
    });

    return () => {
      socket.disconnect();
    };
  }, [roomId]);

  const sendMessage = useCallback((text) => {
    if (isConnected) {
      socket.emit('message', { text, timestamp: Date.now() });
    }
  }, [isConnected]);

  return { messages, isConnected, sendMessage };
};

const ChatRoom = ({ roomId }) => {
  const { messages, isConnected, sendMessage } = useChatRoom(roomId);
  const [inputValue, setInputValue] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    if (inputValue.trim()) {
      sendMessage(inputValue);
      setInputValue('');
    }
  };

  return (
    <div>
      <div>Status: {isConnected ? 'Connected' : 'Disconnected'}</div>
      <div>
        {messages.map((msg, index) => (
          <div key={index}>{msg.text}</div>
        ))}
      </div>
      <form onSubmit={handleSubmit}>
        <input 
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          disabled={!isConnected}
        />
        <button type="submit">Send</button>
      </form>
    </div>
  );
};
```

### Scenario 2: Shopping Cart with Local Storage

```javascript
const useShoppingCart = () => {
  const [items, setItems] = useState(() => {
    const saved = localStorage.getItem('cart');
    return saved ? JSON.parse(saved) : [];
  });

  // Persist to localStorage whenever items change
  useEffect(() => {
    localStorage.setItem('cart', JSON.stringify(items));
  }, [items]);

  const addItem = useCallback((product) => {
    setItems(prev => {
      const existing = prev.find(item => item.id === product.id);
      if (existing) {
        return prev.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        );
      }
      return [...prev, { ...product, quantity: 1 }];
    });
  }, []);

  const removeItem = useCallback((productId) => {
    setItems(prev => prev.filter(item => item.id !== productId));
  }, []);

  const updateQuantity = useCallback((productId, quantity) => {
    if (quantity <= 0) {
      removeItem(productId);
      return;
    }
    setItems(prev =>
      prev.map(item =>
        item.id === productId ? { ...item, quantity } : item
      )
    );
  }, [removeItem]);

  const total = useMemo(() => {
    return items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }, [items]);

  const itemCount = useMemo(() => {
    return items.reduce((sum, item) => sum + item.quantity, 0);
  }, [items]);

  return {
    items,
    addItem,
    removeItem,
    updateQuantity,
    total,
    itemCount
  };
};
```

### Scenario 3: Form Validation with Custom Hook

```javascript
const useFormValidation = (initialValues, validationRules) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const validate = useCallback((fieldName, value) => {
    const rules = validationRules[fieldName];
    if (!rules) return null;

    for (const rule of rules) {
      const error = rule(value);
      if (error) return error;
    }
    return null;
  }, [validationRules]);

  const handleChange = useCallback((name, value) => {
    setValues(prev => ({ ...prev, [name]: value }));
    
    if (touched[name]) {
      const error = validate(name, value);
      setErrors(prev => ({ ...prev, [name]: error }));
    }
  }, [touched, validate]);

  const handleBlur = useCallback((name) => {
    setTouched(prev => ({ ...prev, [name]: true }));
    const error = validate(name, values[name]);
    setErrors(prev => ({ ...prev, [name]: error }));
  }, [validate, values]);

  const isValid = useMemo(() => {
    return Object.keys(validationRules).every(field => !errors[field]);
  }, [errors, validationRules]);

  return {
    values,
    errors,
    touched,
    isValid,
    handleChange,
    handleBlur
  };
};

// Usage
const LoginForm = () => {
  const validationRules = {
    email: [
      (value) => !value ? 'Email is required' : null,
      (value) => !/\S+@\S+\.\S+/.test(value) ? 'Email is invalid' : null
    ],
    password: [
      (value) => !value ? 'Password is required' : null,
      (value) => value.length < 6 ? 'Password must be at least 6 characters' : null
    ]
  };

  const {
    values,
    errors,
    touched,
    isValid,
    handleChange,
    handleBlur
  } = useFormValidation({ email: '', password: '' }, validationRules);

  return (
    <form>
      <input
        type="email"
        value={values.email}
        onChange={(e) => handleChange('email', e.target.value)}
        onBlur={() => handleBlur('email')}
      />
      {touched.email && errors.email && <span>{errors.email}</span>}
      
      <input
        type="password"
        value={values.password}
        onChange={(e) => handleChange('password', e.target.value)}
        onBlur={() => handleBlur('password')}
      />
      {touched.password && errors.password && <span>{errors.password}</span>}
      
      <button type="submit" disabled={!isValid}>Login</button>
    </form>
  );
};
```

## Advanced Hook Patterns {#advanced-patterns}

### Pattern 1: useReducer for Complex State Logic

```javascript
const useShoppingCartReducer = () => {
  const cartReducer = (state, action) => {
    switch (action.type) {
      case 'ADD_ITEM':
        return {
          ...state,
          items: [...state.items, action.payload]
        };
      case 'REMOVE_ITEM':
        return {
          ...state,
          items: state.items.filter(item => item.id !== action.payload.id)
        };
      case 'UPDATE_QUANTITY':
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: action.payload.quantity }
              : item
          )
        };
      case 'CLEAR_CART':
        return { ...state, items: [] };
      default:
        return state;
    }
  };

  const [state, dispatch] = useReducer(cartReducer, { items: [] });

  return { state, dispatch };
};
```

### Pattern 2: Custom Hook with Ref

```javascript
const useClickOutside = (callback) => {
  const ref = useRef();

  useEffect(() => {
    const handleClickOutside = (event) => {
      if (ref.current && !ref.current.contains(event.target)) {
        callback();
      }
    };

    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, [callback]);

  return ref;
};

// Usage
const Dropdown = () => {
  const [isOpen, setIsOpen] = useState(false);
  const dropdownRef = useClickOutside(() => setIsOpen(false));

  return (
    <div ref={dropdownRef}>
      <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
      {isOpen && <div>Dropdown content</div>}
    </div>
  );
};
```

## Interview Questions & Answers {#interview-questions}

### Q1: What are React Hooks and why were they introduced?

**Answer**: React Hooks are functions that allow you to use state and other React features in functional components. They were introduced in React 16.8 to solve several problems:

1. **Stateful Logic Reuse**: Before hooks, sharing stateful logic between components required complex patterns like HOCs or render props
2. **Complex Components**: Class components with multiple lifecycle methods became hard to understand and maintain
3. **Confusing Classes**: `this` binding and lifecycle methods were confusing for developers

Hooks allow you to:
- Use state in functional components
- Perform side effects
- Access context
- Create reusable stateful logic

### Q2: What are the rules of hooks?

**Answer**: There are two main rules:

1. **Only call hooks at the top level**: Don't call hooks inside loops, conditions, or nested functions. This ensures hooks are called in the same order every time.

2. **Only call hooks from React functions**: Call hooks from React function components or from custom hooks, not from regular JavaScript functions.

These rules ensure that React can correctly associate hook calls with component instances and maintain their state across re-renders.

### Q3: Explain the difference between useState and useReducer. When would you use each?

**Answer**:

**useState**:
- Simple state management
- Good for primitive values or simple objects
- State updates are straightforward

```javascript
const [count, setCount] = useState(0);
const increment = () => setCount(count + 1);
```

**useReducer**:
- Complex state management
- Good for state with multiple sub-values
- State updates involve complex logic
- Similar to Redux pattern

```javascript
const [state, dispatch] = useReducer(reducer, initialState);
const increment = () => dispatch({ type: 'INCREMENT' });
```

**When to use useReducer**:
- State has complex update logic
- Next state depends on previous state
- Want to optimize performance (dispatch is always stable)
- Need to manage related pieces of state together

### Q4: How does useEffect work and what are its different patterns?

**Answer**: useEffect lets you perform side effects in functional components. It serves the same purpose as componentDidMount, componentDidUpdate, and componentWillUnmount combined.

**Patterns**:

1. **Effect without dependencies** (runs after every render):
```javascript
useEffect(() => {
  document.title = `Count: ${count}`;
});
```

2. **Effect with empty dependencies** (runs once after mount):
```javascript
useEffect(() => {
  fetchData();
}, []);
```

3. **Effect with dependencies** (runs when dependencies change):
```javascript
useEffect(() => {
  fetchUser(userId);
}, [userId]);
```

4. **Effect with cleanup** (cleanup when component unmounts or before next effect):
```javascript
useEffect(() => {
  const timer = setInterval(() => {
    setCount(c => c + 1);
  }, 1000);
  
  return () => clearInterval(timer);
}, []);
```

### Q5: What is the purpose of useMemo and useCallback? How do they differ?

**Answer**:

**useMemo**: Memoizes the result of a computation
- Use when you have expensive calculations
- Returns a memoized value

```javascript
const expensiveValue = useMemo(() => {
  return items.filter(item => item.category === 'electronics');
}, [items]);
```

**useCallback**: Memoizes a function
- Use to prevent unnecessary re-renders of child components
- Returns a memoized callback

```javascript
const handleClick = useCallback((id) => {
  onItemClick(id);
}, [onItemClick]);
```

**Key Differences**:
- useMemo memoizes values, useCallback memoizes functions
- useMemo runs the function and returns its result, useCallback returns the function itself
- Both help with performance optimization by preventing unnecessary recalculations/recreations

### Q6: How would you create a custom hook for API data fetching?

**Answer**:

```javascript
const useApi = (url, options = {}) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(url, options);
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const result = await response.json();
        setData(result);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url, JSON.stringify(options)]);

  const refetch = useCallback(() => {
    fetchData();
  }, [url, options]);

  return { data, loading, error, refetch };
};

// Usage
const UserProfile = ({ userId }) => {
  const { data: user, loading, error } = useApi(`/api/users/${userId}`);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;
  return <div>{user.name}</div>;
};
```

### Q7: Explain the concept of "stale closures" in React hooks and how to avoid them.

**Answer**: Stale closures occur when a function captures variables from its lexical scope, but those variables become outdated by the time the function executes.

**Problem Example**:
```javascript
const Counter = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setCount(count + 1); // This captures the initial value of count (0)
    }, 1000);

    return () => clearInterval(timer);
  }, []); // Empty dependency array causes stale closure

  return <div>{count}</div>;
};
```

**Solutions**:

1. **Use functional state updates**:
```javascript
setCount(prevCount => prevCount + 1);
```

2. **Include dependencies**:
```javascript
useEffect(() => {
  const timer = setInterval(() => {
    setCount(count + 1);
  }, 1000);
  return () => clearInterval(timer);
}, [count]); // Include count in dependencies
```

3. **Use useRef for mutable values**:
```javascript
const countRef = useRef(count);
countRef.current = count;

useEffect(() => {
  const timer = setInterval(() => {
    setCount(countRef.current + 1);
  }, 1000);
  return () => clearInterval(timer);
}, []);
```

### Q8: How would you optimize a component that renders a large list?

**Answer**: Several optimization strategies:

1. **React.memo for preventing unnecessary re-renders**:
```javascript
const ListItem = React.memo(({ item, onDelete }) => {
  return (
    <div>
      {item.name}
      <button onClick={() => onDelete(item.id)}>Delete</button>
    </div>
  );
});
```

2. **useCallback for stable function references**:
```javascript
const List = ({ items }) => {
  const handleDelete = useCallback((id) => {
    setItems(items => items.filter(item => item.id !== id));
  }, []);

  return (
    <div>
      {items.map(item => (
        <ListItem key={item.id} item={item} onDelete={handleDelete} />
      ))}
    </div>
  );
};
```

3. **Virtualization for very large lists**:
```javascript
import { FixedSizeList as List } from 'react-window';

const VirtualizedList = ({ items }) => {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );

  return (
    <List
      height={600}
      itemCount={items.length}
      itemSize={35}
    >
      {Row}
    </List>
  );
};
```

### Q9: What are some common mistakes when using useEffect?

**Answer**:

1. **Missing dependencies**:
```javascript
// Wrong - missing dependency
useEffect(() => {
  fetchUser(userId);
}, []);

// Correct
useEffect(() => {
  fetchUser(userId);
}, [userId]);
```

2. **Infinite loops**:
```javascript
// Wrong - object/array dependency causes infinite loop
const [user, setUser] = useState({});
useEffect(() => {
  fetchUser().then(setUser);
}, [user]); // user object changes on every render

// Correct
useEffect(() => {
  fetchUser().then(setUser);
}, [user.id]); // depend on specific property
```

3. **Not cleaning up subscriptions**:
```javascript
// Wrong - memory leak
useEffect(() => {
  const subscription = subscribeTo(something);
}, []);

// Correct
useEffect(() => {
  const subscription = subscribeTo(something);
  return () => subscription.unsubscribe();
}, []);
```

### Q10: How do you test components that use hooks?

**Answer**: Use React Testing Library with special considerations for hooks:

```javascript
import { render, screen, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';

// Testing custom hooks
import { renderHook, act } from '@testing-library/react';

test('useCounter hook', () => {
  const { result } = renderHook(() => useCounter(0));

  expect(result.current.count).toBe(0);

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});

// Testing components with hooks
test('Counter component', async () => {
  render(<Counter />);
  
  const button = screen.getByText(/increment/i);
  const count = screen.getByText(/count: 0/i);

  expect(count).toBeInTheDocument();

  await userEvent.click(button);

  await waitFor(() => {
    expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
  });
});

// Mocking custom hooks
jest.mock('./useApi', () => ({
  useApi: jest.fn()
}));

test('component with mocked hook', () => {
  useApi.mockReturnValue({
    data: { name: 'John' },
    loading: false,
    error: null
  });

  render(<UserProfile userId="1" />);
  expect(screen.getByText('John')).toBeInTheDocument();
});
```

## Advanced Custom Hooks Library {#custom-hooks-library}

Here are more comprehensive custom hooks for common use cases:

### 1. useToggle - Boolean State Management
```javascript
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);

  return [value, { toggle, setTrue, setFalse }];
}

// Usage
function Modal() {
  const [isOpen, { toggle, setTrue, setFalse }] = useToggle(false);

  return (
    <div>
      <button onClick={setTrue}>Open Modal</button>
      {isOpen && (
        <div className="modal">
          <button onClick={setFalse}>Close</button>
          <button onClick={toggle}>Toggle</button>
        </div>
      )}
    </div>
  );
}
```

### 2. usePrevious - Track Previous Values
```javascript
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  });
  
  return ref.current;
}

// Usage
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### 3. useDebounce - Debounce Values
```javascript
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchInput() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);

  useEffect(() => {
    if (debouncedSearchTerm) {
      // Perform search
      console.log('Searching for:', debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);

  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### 4. useInterval - Declarative Intervals
```javascript
function useInterval(callback, delay) {
  const savedCallback = useRef();

  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);

  useEffect(() => {
    function tick() {
      savedCallback.current();
    }
    
    if (delay !== null) {
      const id = setInterval(tick, delay);
      return () => clearInterval(id);
    }
  }, [delay]);
}

// Usage
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(true);

  useInterval(() => {
    setSeconds(seconds => seconds + 1);
  }, isRunning ? 1000 : null);

  return (
    <div>
      <p>Seconds: {seconds}</p>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? 'Pause' : 'Start'}
      </button>
    </div>
  );
}
```

### 5. useAsync - Async Operation Management
```javascript
function useAsync(asyncFunction, dependencies = []) {
  const [status, setStatus] = useState('idle');
  const [value, setValue] = useState(null);
  const [error, setError] = useState(null);

  const execute = useCallback(() => {
    setStatus('pending');
    setValue(null);
    setError(null);

    return asyncFunction()
      .then(response => {
        setValue(response);
        setStatus('success');
      })
      .catch(error => {
        setError(error);
        setStatus('error');
      });
  }, dependencies);

  useEffect(() => {
    execute();
  }, [execute]);

  return {
    execute,
    status,
    value,
    error,
    isLoading: status === 'pending',
    isError: status === 'error',
    isSuccess: status === 'success'
  };
}

// Usage
function UserProfile({ userId }) {
  const {
    value: user,
    isLoading,
    isError,
    error,
    execute: refetch
  } = useAsync(
    () => fetch(`/api/users/${userId}`).then(res => res.json()),
    [userId]
  );

  if (isLoading) return <div>Loading...</div>;
  if (isError) return <div>Error: {error.message}</div>;

  return (
    <div>
      <h1>{user.name}</h1>
      <button onClick={refetch}>Refresh</button>
    </div>
  );
}
```

### 6. useKeyboard - Keyboard Event Handler
```javascript
function useKeyboard(targetKey, callback, options = {}) {
  const { 
    event = 'keydown',
    target = window,
    condition = () => true 
  } = options;

  useEffect(() => {
    const handler = (e) => {
      if (e.key === targetKey && condition(e)) {
        callback(e);
      }
    };

    target.addEventListener(event, handler);
    return () => target.removeEventListener(event, handler);
  }, [targetKey, callback, event, target, condition]);
}

// Usage
function App() {
  const [count, setCount] = useState(0);

  useKeyboard('ArrowUp', () => setCount(c => c + 1));
  useKeyboard('ArrowDown', () => setCount(c => c - 1));
  useKeyboard('Escape', () => setCount(0));

  return (
    <div>
      <p>Count: {count}</p>
      <p>Use arrow keys to increment/decrement, ESC to reset</p>
    </div>
  );
}
```

### 7. useMediaQuery - Responsive Design
```javascript
function useMediaQuery(query) {
  const [matches, setMatches] = useState(false);

  useEffect(() => {
    const media = window.matchMedia(query);
    if (media.matches !== matches) {
      setMatches(media.matches);
    }

    const listener = () => setMatches(media.matches);
    media.addEventListener('change', listener);
    
    return () => media.removeEventListener('change', listener);
  }, [matches, query]);

  return matches;
}

// Usage
function ResponsiveComponent() {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isDesktop = useMediaQuery('(min-width: 1024px)');

  return (
    <div>
      {isMobile && <div>Mobile View</div>}
      {isDesktop && <div>Desktop View</div>}
      {!isMobile && !isDesktop && <div>Tablet View</div>}
    </div>
  );
}
```

## Debugging and Troubleshooting React Hooks {#debugging-hooks}

### Common Hook Problems and Solutions

#### 1. Infinite Re-render Loops

**Problem**: Component keeps re-rendering infinitely.

```javascript
// ❌ Problematic code
function ProblematicComponent() {
  const [count, setCount] = useState(0);

  // This causes infinite loop!
  useEffect(() => {
    setCount(count + 1);
  }, [count]); // count changes, effect runs, count changes again...

  return <div>{count}</div>;
}
```

**Solutions**:
```javascript
// ✅ Solution 1: Remove the dependency
useEffect(() => {
  setCount(c => c + 1); // Use functional update
}, []); // Run only once

// ✅ Solution 2: Use a different approach
useEffect(() => {
  const timer = setTimeout(() => {
    setCount(c => c + 1);
  }, 1000);
  return () => clearTimeout(timer);
}, []); // Run only once
```

#### 2. Stale Closures

**Problem**: Hook captures old values.

```javascript
// ❌ Problematic code
function StaleClosureExample() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      console.log(count); // Always logs 0!
      setCount(count + 1); // Always adds 1 to 0!
    }, 1000);

    return () => clearInterval(timer);
  }, []); // Empty deps cause stale closure

  return <div>{count}</div>;
}
```

**Solutions**:
```javascript
// ✅ Solution 1: Use functional updates
useEffect(() => {
  const timer = setInterval(() => {
    setCount(prevCount => {
      console.log(prevCount); // Logs current value
      return prevCount + 1;
    });
  }, 1000);

  return () => clearInterval(timer);
}, []);

// ✅ Solution 2: Use useRef for mutable values
function FixedWithRef() {
  const [count, setCount] = useState(0);
  const countRef = useRef(count);
  countRef.current = count;

  useEffect(() => {
    const timer = setInterval(() => {
      console.log(countRef.current); // Always current value
      setCount(c => c + 1);
    }, 1000);

    return () => clearInterval(timer);
  }, []);

  return <div>{count}</div>;
}
```

#### 3. Missing Dependencies

**Problem**: ESLint warnings about missing dependencies.

```javascript
// ❌ Problematic code
function MissingDeps({ userId, onUserLoaded }) {
  const [user, setUser] = useState(null);

  useEffect(() => {
    fetchUser(userId).then(userData => {
      setUser(userData);
      onUserLoaded(userData); // Missing from deps!
    });
  }, [userId]); // ESLint warning: missing 'onUserLoaded'

  return <div>{user?.name}</div>;
}
```

**Solutions**:
```javascript
// ✅ Solution 1: Add all dependencies
useEffect(() => {
  fetchUser(userId).then(userData => {
    setUser(userData);
    onUserLoaded(userData);
  });
}, [userId, onUserLoaded]); // Include all dependencies

// ✅ Solution 2: Use useCallback in parent component
function ParentComponent() {
  const handleUserLoaded = useCallback((user) => {
    console.log('User loaded:', user);
  }, []);

  return <MissingDeps userId="123" onUserLoaded={handleUserLoaded} />;
}

// ✅ Solution 3: Use useCallback inside component
function FixedComponent({ userId, onUserLoaded }) {
  const [user, setUser] = useState(null);

  const handleUserLoaded = useCallback(onUserLoaded, [onUserLoaded]);

  useEffect(() => {
    fetchUser(userId).then(userData => {
      setUser(userData);
      handleUserLoaded(userData);
    });
  }, [userId, handleUserLoaded]);

  return <div>{user?.name}</div>;
}
```

#### 4. Memory Leaks

**Problem**: Component updates state after unmounting.

```javascript
// ❌ Problematic code
function MemoryLeakExample() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(result => {
      setData(result); // Might happen after unmount!
    });
  }, []);

  return <div>{data}</div>;
}
```

**Solutions**:
```javascript
// ✅ Solution 1: Use cleanup flag
function FixedComponent() {
  const [data, setData] = useState(null);

  useEffect(() => {
    let isCancelled = false;

    fetchData().then(result => {
      if (!isCancelled) {
        setData(result);
      }
    });

    return () => {
      isCancelled = true;
    };
  }, []);

  return <div>{data}</div>;
}

// ✅ Solution 2: Use AbortController
function AbortControllerExample() {
  const [data, setData] = useState(null);

  useEffect(() => {
    const abortController = new AbortController();

    fetch('/api/data', { signal: abortController.signal })
      .then(response => response.json())
      .then(result => setData(result))
      .catch(error => {
        if (error.name !== 'AbortError') {
          console.error('Fetch error:', error);
        }
      });

    return () => {
      abortController.abort();
    };
  }, []);

  return <div>{data}</div>;
}
```

### Debugging Tools and Techniques

#### 1. React DevTools
- Install React DevTools browser extension
- Use "Profiler" tab to identify performance issues
- Use "Components" tab to inspect hook values

#### 2. Custom Debug Hook
```javascript
function useDebug(name, value) {
  const prevValue = usePrevious(value);
  
  useEffect(() => {
    if (prevValue !== value) {
      console.log(`${name} changed:`, {
        from: prevValue,
        to: value
      });
    }
  });
}

// Usage
function DebuggableComponent({ userId }) {
  const [user, setUser] = useState(null);
  
  useDebug('userId', userId);
  useDebug('user', user);

  // Rest of component...
}
```

#### 3. useWhyDidYouUpdate Hook
```javascript
function useWhyDidYouUpdate(name, props) {
  const previous = useRef();
  
  useEffect(() => {
    if (previous.current) {
      const allKeys = Object.keys({ ...previous.current, ...props });
      const changedProps = {};
      
      allKeys.forEach(key => {
        if (previous.current[key] !== props[key]) {
          changedProps[key] = {
            from: previous.current[key],
            to: props[key]
          };
        }
      });
      
      if (Object.keys(changedProps).length) {
        console.log('[why-did-you-update]', name, changedProps);
      }
    }
    
    previous.current = props;
  });
}

// Usage
function ExpensiveComponent(props) {
  useWhyDidYouUpdate('ExpensiveComponent', props);
  
  return <div>Expensive rendering...</div>;
}
```

#### 4. Performance Monitoring
```javascript
function useRenderCount() {
  const count = useRef(1);
  useEffect(() => {
    count.current++;
  });
  return count.current;
}

function useRenderTime(name) {
  const start = useRef();
  const renderCount = useRenderCount();
  
  if (!start.current) {
    start.current = performance.now();
  }
  
  useEffect(() => {
    const end = performance.now();
    console.log(`${name} render #${renderCount} took ${end - start.current}ms`);
    start.current = performance.now();
  });
}

// Usage
function MonitoredComponent() {
  useRenderTime('MonitoredComponent');
  const renderCount = useRenderCount();
  
  return <div>Render count: {renderCount}</div>;
}
```

---

This comprehensive guide covers React Hooks from beginner to advanced levels. Practice these examples and patterns to master React Hooks development!