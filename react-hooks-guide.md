# React Hooks Complete Guide

## Table of Contents
1. [Introduction to React Hooks](#introduction)
2. [Hook Lifecycle Diagram](#lifecycle-diagram)
3. [Core Hooks Overview](#core-hooks)
4. [Problem Statements & Solutions](#problems-solutions)
5. [Real-time Scenarios](#real-time-scenarios)
6. [Advanced Hook Patterns](#advanced-patterns)
7. [Interview Questions & Answers](#interview-questions)

## Introduction to React Hooks {#introduction}

React Hooks are functions that let you "hook into" React state and lifecycle features from function components. They were introduced in React 16.8 to solve several problems:

- **Stateful Logic Reuse**: Share stateful logic between components without wrapper hell
- **Complex Components**: Break down complex components into smaller functions
- **Classes Confusion**: Eliminate confusion around `this` binding and lifecycle methods

### Hook Rules
1. Only call hooks at the top level (not inside loops, conditions, or nested functions)
2. Only call hooks from React function components or custom hooks

## Hook Lifecycle Diagram {#lifecycle-diagram}

```
Component Lifecycle with Hooks
┌─────────────────────────────────────────────────────────────┐
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

---

This guide covers the essential concepts, patterns, and best practices for React Hooks. Practice these examples and patterns to master React Hooks development!