# UI Component Design Guide

## Overview
This guide outlines the principles, patterns, and system design concepts for creating scalable and maintainable UI components.

## Component Design Principles

### 1. Single Responsibility Principle
- Each component should have one clear purpose
- Avoid components that handle multiple unrelated concerns
- Example: A `Button` component should only handle button behavior, not form validation

### 2. Composition over Inheritance
- Build complex components by combining simpler ones
- Use props and children for customization
- Prefer component composition patterns over class inheritance

### 3. Props Interface Design
- Keep props minimal and focused
- Use TypeScript/PropTypes for type safety
- Follow consistent naming conventions
- Provide sensible defaults

### 4. State Management
- Keep state as close to where it's needed as possible
- Lift state up only when necessary for sharing
- Use appropriate state management tools (Context, Redux, Zustand)

## System Design Concepts

### 1. Component Architecture Patterns

#### Atomic Design
- **Atoms**: Basic building blocks (buttons, inputs, labels)
- **Molecules**: Simple groups of atoms (search form, card header)
- **Organisms**: Complex UI sections (navigation, product grid)
- **Templates**: Page-level layouts
- **Pages**: Specific instances of templates

#### Container/Presentational Pattern
- **Presentational Components**: Focus on how things look
- **Container Components**: Focus on how things work
- Separation of concerns between UI and business logic

### 2. Design System Foundation

#### Design Tokens
- Colors, typography, spacing, shadows
- Consistent visual language across components
- Easy theming and customization

#### Component Library Structure
```
/components
  /atoms
    /Button
    /Input
    /Icon
  /molecules
    /SearchBox
    /Card
  /organisms
    /Header
    /ProductGrid
  /templates
    /PageLayout
```

### 3. Scalability Considerations

#### Performance
- Lazy loading for large component trees
- Memoization for expensive calculations
- Virtual scrolling for large lists
- Code splitting at component level

#### Accessibility
- Semantic HTML structure
- ARIA attributes for screen readers
- Keyboard navigation support
- Color contrast compliance

#### Responsive Design
- Mobile-first approach
- Flexible grid systems
- Breakpoint-based styling
- Progressive enhancement

### 4. Component Communication Patterns

#### Props Down, Events Up
- Pass data down through props
- Communicate changes up through callbacks
- Maintain unidirectional data flow

#### Context API
- For deeply nested prop drilling
- Theme providers
- Authentication state
- Global settings

#### Custom Hooks
- Reusable stateful logic
- Component lifecycle management
- External API integration

### 5. Testing Strategy

#### Unit Testing
- Component rendering
- Props handling
- Event handling
- State changes

#### Integration Testing
- Component interactions
- Data flow between components
- User workflows

#### Visual Regression Testing
- Screenshot comparisons
- Cross-browser compatibility
- Responsive design validation

### 6. Documentation Standards

#### Component Documentation
- Purpose and use cases
- Props interface
- Usage examples
- Accessibility notes

#### Storybook Integration
- Interactive component playground
- Visual documentation
- Design system showcase

## Best Practices

### 1. Naming Conventions
- Use PascalCase for component names
- Use descriptive, intention-revealing names
- Avoid abbreviations and acronyms

### 2. File Organization
- One component per file
- Co-locate related files (styles, tests, stories)
- Consistent folder structure

### 3. Error Handling
- Graceful degradation
- Error boundaries for component trees
- Meaningful error messages

### 4. Performance Optimization
- Avoid inline functions in render
- Use React.memo for pure components
- Optimize re-renders with useCallback and useMemo

### 5. Code Quality
- Consistent code formatting (Prettier)
- Linting rules (ESLint)
- Type checking (TypeScript)
- Code review processes

## Practical Examples: Component Design Process

### Example 1: Designing a Data Table Component

#### Step 1: Requirements Analysis
**Business Need**: Display tabular data with sorting, filtering, and pagination

**Questions to Ask**:
- What types of data will be displayed?
- What interactions are needed (sorting, filtering, selection)?
- How much data will be displayed at once?
- What devices will this be used on?

#### Step 2: Component Planning
```
DataTable (Organism)
├── TableHeader (Molecule)
│   ├── HeaderCell (Atom)
│   └── SortIcon (Atom)
├── TableBody (Molecule)
│   └── TableRow (Molecule)
│       └── TableCell (Atom)
├── TablePagination (Molecule)
│   ├── Button (Atom)
│   └── PageInfo (Atom)
└── TableFilters (Molecule)
    └── FilterInput (Atom)
```

#### Step 3: Props Interface Design
```javascript
interface DataTableProps {
  // Data
  data: Array<Record<string, any>>;
  columns: Column[];
  
  // Functionality
  sortable?: boolean;
  filterable?: boolean;
  selectable?: boolean;
  paginated?: boolean;
  
  // Customization
  className?: string;
  rowsPerPage?: number;
  
  // Events
  onSort?: (column: string, direction: 'asc' | 'desc') => void;
  onFilter?: (filters: Record<string, any>) => void;
  onSelect?: (selectedRows: any[]) => void;
}
```

#### Step 4: State Management Planning
```javascript
// Internal state
const [sortConfig, setSortConfig] = useState({ column: null, direction: 'asc' });
const [filters, setFilters] = useState({});
const [selectedRows, setSelectedRows] = useState([]);
const [currentPage, setCurrentPage] = useState(1);

// Derived state
const filteredData = useMemo(() => applyFilters(data, filters), [data, filters]);
const sortedData = useMemo(() => applySorting(filteredData, sortConfig), [filteredData, sortConfig]);
const paginatedData = useMemo(() => paginate(sortedData, currentPage, rowsPerPage), [sortedData, currentPage, rowsPerPage]);
```

### Example 2: Designing a Modal Component

#### Step 1: Requirements Analysis
**Business Need**: Display overlay content with various use cases (confirmation, forms, image preview)

**Thinking Process**:
- Modal should be flexible enough for different content types
- Need to handle focus management for accessibility
- Should support different sizes and animations
- Must handle escape key and backdrop clicks

#### Step 2: API Design Thinking
```javascript
// Option 1: Imperative API
const modal = useModal();
modal.open(<ConfirmDialog />);

// Option 2: Declarative API (chosen)
<Modal isOpen={isOpen} onClose={handleClose}>
  <ConfirmDialog />
</Modal>

// Why declarative? Better fits React's model and easier to reason about
```

#### Step 3: Implementation Strategy
```javascript
const Modal = ({ isOpen, onClose, size = 'medium', children }) => {
  // Focus management
  const modalRef = useRef();
  const previousFocusRef = useRef();
  
  useEffect(() => {
    if (isOpen) {
      previousFocusRef.current = document.activeElement;
      modalRef.current?.focus();
    } else {
      previousFocusRef.current?.focus();
    }
  }, [isOpen]);

  // Escape key handling
  useEffect(() => {
    const handleEscape = (event) => {
      if (event.key === 'Escape') onClose();
    };
    
    if (isOpen) {
      document.addEventListener('keydown', handleEscape);
      return () => document.removeEventListener('keydown', handleEscape);
    }
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return createPortal(
    <div className="modal-backdrop" onClick={onClose}>
      <div 
        ref={modalRef}
        className={`modal modal--${size}`}
        onClick={(e) => e.stopPropagation()}
        role="dialog"
        aria-modal="true"
        tabIndex={-1}
      >
        {children}
      </div>
    </div>,
    document.body
  );
};
```

### Example 3: Thinking Through a Form Input Component

#### Step 1: Analysis
**Requirements**: Reusable input with validation, different types, consistent styling

**Thinking Process**:
- Should it handle its own validation or receive validation from parent?
- How to handle different input types (text, email, password, etc.)?
- How to display error states and messages?
- Should it be controlled or uncontrolled?

#### Step 2: Design Decisions
```javascript
// Decision: Controlled component with external validation
// Reasoning: More flexible, follows React patterns, easier to test

interface InputProps {
  // Core functionality
  value: string;
  onChange: (value: string) => void;
  
  // Input configuration
  type?: 'text' | 'email' | 'password' | 'number';
  placeholder?: string;
  disabled?: boolean;
  required?: boolean;
  
  // Validation
  error?: string;
  valid?: boolean;
  
  // Accessibility
  label: string;
  id?: string;
  'aria-describedby'?: string;
  
  // Styling
  variant?: 'outlined' | 'filled' | 'standard';
  size?: 'small' | 'medium' | 'large';
}
```

#### Step 3: Component Structure Planning
```javascript
const Input = ({
  value,
  onChange,
  label,
  error,
  id,
  className,
  ...props
}) => {
  const inputId = id || useId();
  const errorId = error ? `${inputId}-error` : undefined;
  
  return (
    <div className={cn('input-wrapper', className, {
      'input-wrapper--error': error,
      'input-wrapper--disabled': props.disabled
    })}>
      <label htmlFor={inputId} className="input-label">
        {label}
        {props.required && <span aria-label="required">*</span>}
      </label>
      
      <input
        {...props}
        id={inputId}
        value={value}
        onChange={(e) => onChange(e.target.value)}
        className="input-field"
        aria-invalid={!!error}
        aria-describedby={errorId}
      />
      
      {error && (
        <div id={errorId} className="input-error" role="alert">
          {error}
        </div>
      )}
    </div>
  );
};
```

### Design Thinking Framework

#### 1. Start with User Needs
- What problem does this component solve?
- Who will use it and in what context?
- What are the edge cases and error scenarios?

#### 2. API Design First
- How should developers interact with your component?
- What props are essential vs. optional?
- How can you make the API intuitive and consistent?

#### 3. Break Down Complexity
- Can this be split into smaller components?
- What can be reused across different contexts?
- Where should state live?

#### 4. Consider Non-Functional Requirements
- Performance implications
- Accessibility requirements
- Browser compatibility
- Responsive behavior

#### 5. Plan for Change
- How will this component evolve?
- What customization points are needed?
- How can you maintain backward compatibility?

### Common Anti-Patterns to Avoid

#### 1. The "God Component"
```javascript
// ❌ Bad: Component doing too much
const UserProfile = ({ userId }) => {
  const [user, setUser] = useState();
  const [posts, setPosts] = useState();
  const [friends, setFriends] = useState();
  // ... handles data fetching, form submission, navigation, etc.
  
  return (
    <div>
      {/* 500+ lines of JSX */}
    </div>
  );
};

// ✅ Good: Single responsibility
const UserProfile = ({ user }) => (
  <div>
    <UserHeader user={user} />
    <UserPosts userId={user.id} />
    <UserFriends userId={user.id} />
  </div>
);
```

#### 2. Props Explosion
```javascript
// ❌ Bad: Too many individual props
const Button = ({
  backgroundColor,
  textColor,
  borderColor,
  hoverBackgroundColor,
  hoverTextColor,
  fontSize,
  fontWeight,
  padding,
  margin,
  borderRadius,
  // ... 20+ more props
}) => { /* ... */ };

// ✅ Good: Variant-based design
const Button = ({
  variant = 'primary',
  size = 'medium',
  children,
  ...props
}) => { /* ... */ };
```

## Implementation Checklist

### Before Building
- [ ] Define component requirements
- [ ] Identify reusable patterns
- [ ] Plan component hierarchy
- [ ] Design props interface

### During Development
- [ ] Follow established patterns
- [ ] Write tests alongside code
- [ ] Document as you build
- [ ] Consider accessibility

### After Implementation
- [ ] Performance review
- [ ] Accessibility audit
- [ ] Cross-browser testing
- [ ] Documentation update

## Frontend System Design Concepts

### 1. Scalability Architecture

#### Horizontal vs Vertical Scaling
- **Horizontal**: Multiple instances, load balancing, CDN distribution
- **Vertical**: Increasing server resources, database optimization
- **Frontend Focus**: Code splitting, lazy loading, micro-frontends

#### Micro-Frontends Architecture
```javascript
// Module Federation Example
const ModuleFederationPlugin = require('@module-federation/webpack');

module.exports = {
  plugins: [
    new ModuleFederationPlugin({
      name: 'host',
      remotes: {
        userModule: 'user@http://localhost:3001/remoteEntry.js',
        productModule: 'product@http://localhost:3002/remoteEntry.js',
      },
    }),
  ],
};

// Usage
const UserProfile = React.lazy(() => import('userModule/UserProfile'));
const ProductCatalog = React.lazy(() => import('productModule/ProductCatalog'));
```

#### Bundle Optimization Strategies
```javascript
// Code Splitting
const LazyComponent = React.lazy(() => import('./LazyComponent'));

// Route-based splitting
const routes = [
  {
    path: '/dashboard',
    component: React.lazy(() => import('./Dashboard')),
  },
  {
    path: '/profile',
    component: React.lazy(() => import('./Profile')),
  },
];

// Dynamic imports with chunks
const loadUtility = async () => {
  const { utility } = await import(/* webpackChunkName: "utility" */ './utility');
  return utility;
};
```

### 2. Caching Strategies

#### Browser Caching
```javascript
// Service Worker for Cache-First Strategy
self.addEventListener('fetch', (event) => {
  if (event.request.destination === 'image') {
    event.respondWith(
      caches.open('images-v1').then((cache) => {
        return cache.match(event.request).then((response) => {
          return response || fetch(event.request).then((fetchResponse) => {
            cache.put(event.request, fetchResponse.clone());
            return fetchResponse;
          });
        });
      })
    );
  }
});
```

#### Application-Level Caching
```javascript
// React Query for Server State Caching
import { useQuery, useQueryClient } from 'react-query';

const useUserData = (userId) => {
  return useQuery(
    ['user', userId],
    () => fetchUser(userId),
    {
      staleTime: 5 * 60 * 1000, // 5 minutes
      cacheTime: 10 * 60 * 1000, // 10 minutes
      refetchOnWindowFocus: false,
    }
  );
};

// Memory-based caching with Map
class MemoryCache {
  constructor(maxSize = 100) {
    this.cache = new Map();
    this.maxSize = maxSize;
  }

  get(key) {
    if (this.cache.has(key)) {
      const value = this.cache.get(key);
      // Move to end (LRU)
      this.cache.delete(key);
      this.cache.set(key, value);
      return value;
    }
    return null;
  }

  set(key, value) {
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    this.cache.set(key, value);
  }
}
```

#### CDN and Static Asset Caching
```javascript
// Webpack configuration for long-term caching
module.exports = {
  output: {
    filename: '[name].[contenthash].js',
    chunkFilename: '[name].[contenthash].chunk.js',
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          chunks: 'all',
        },
      },
    },
  },
};
```

### 3. State Management Architecture

#### Global State Management Patterns
```javascript
// Redux Toolkit with RTK Query
import { createSlice, createApi, fetchBaseQuery } from '@reduxjs/toolkit';

// API slice
export const apiSlice = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ baseUrl: '/api' }),
  tagTypes: ['User', 'Product'],
  endpoints: (builder) => ({
    getUsers: builder.query({
      query: () => '/users',
      providesTags: ['User'],
    }),
    updateUser: builder.mutation({
      query: ({ id, ...patch }) => ({
        url: `/users/${id}`,
        method: 'PATCH',
        body: patch,
      }),
      invalidatesTags: ['User'],
    }),
  }),
});

// Feature slice
const userSlice = createSlice({
  name: 'user',
  initialState: { currentUser: null, preferences: {} },
  reducers: {
    setCurrentUser: (state, action) => {
      state.currentUser = action.payload;
    },
    updatePreferences: (state, action) => {
      state.preferences = { ...state.preferences, ...action.payload };
    },
  },
});
```

#### State Normalization
```javascript
// Normalized state structure
const initialState = {
  users: {
    byId: {},
    allIds: [],
  },
  posts: {
    byId: {},
    allIds: [],
  },
  comments: {
    byId: {},
    allIds: [],
  },
};

// Selectors for denormalized data
const selectUserById = (state, userId) => state.users.byId[userId];
const selectPostsByUser = createSelector(
  [state => state.posts.byId, state => state.posts.allIds, (state, userId) => userId],
  (postsById, postIds, userId) => 
    postIds.filter(id => postsById[id].authorId === userId)
           .map(id => postsById[id])
);
```

#### Context API Optimization
```javascript
// Split contexts to prevent unnecessary re-renders
const UserContext = createContext();
const UserDispatchContext = createContext();

const UserProvider = ({ children }) => {
  const [user, dispatch] = useReducer(userReducer, initialUser);
  
  return (
    <UserContext.Provider value={user}>
      <UserDispatchContext.Provider value={dispatch}>
        {children}
      </UserDispatchContext.Provider>
    </UserContext.Provider>
  );
};

// Custom hooks
const useUser = () => {
  const context = useContext(UserContext);
  if (!context) throw new Error('useUser must be used within UserProvider');
  return context;
};

const useUserDispatch = () => {
  const context = useContext(UserDispatchContext);
  if (!context) throw new Error('useUserDispatch must be used within UserProvider');
  return context;
};
```

### 4. Authentication & Authorization

#### JWT Token Management
```javascript
// Token storage and refresh logic
class AuthManager {
  constructor() {
    this.accessToken = localStorage.getItem('accessToken');
    this.refreshToken = localStorage.getItem('refreshToken');
    this.refreshPromise = null;
  }

  async getValidToken() {
    if (!this.accessToken) return null;
    
    if (this.isTokenExpired(this.accessToken)) {
      if (this.refreshPromise) {
        return this.refreshPromise;
      }
      
      this.refreshPromise = this.refreshAccessToken();
      return this.refreshPromise;
    }
    
    return this.accessToken;
  }

  async refreshAccessToken() {
    try {
      const response = await fetch('/auth/refresh', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ refreshToken: this.refreshToken }),
      });
      
      const { accessToken, refreshToken } = await response.json();
      
      this.setTokens(accessToken, refreshToken);
      this.refreshPromise = null;
      
      return accessToken;
    } catch (error) {
      this.clearTokens();
      window.location.href = '/login';
      throw error;
    }
  }

  isTokenExpired(token) {
    const payload = JSON.parse(atob(token.split('.')[1]));
    return payload.exp * 1000 < Date.now();
  }
}
```

#### Route Protection
```javascript
// Higher-order component for route protection
const withAuth = (WrappedComponent, requiredPermissions = []) => {
  return function AuthenticatedComponent(props) {
    const { user, isLoading } = useAuth();
    const navigate = useNavigate();

    useEffect(() => {
      if (!isLoading && !user) {
        navigate('/login');
        return;
      }

      if (user && requiredPermissions.length > 0) {
        const hasPermission = requiredPermissions.every(permission =>
          user.permissions.includes(permission)
        );
        
        if (!hasPermission) {
          navigate('/unauthorized');
          return;
        }
      }
    }, [user, isLoading, navigate]);

    if (isLoading) return <LoadingSpinner />;
    if (!user) return null;

    return <WrappedComponent {...props} />;
  };
};

// Usage
const AdminPanel = withAuth(AdminPanelComponent, ['admin', 'write']);
```

#### OAuth 2.0 / OIDC Implementation
```javascript
// OAuth flow with PKCE
class OAuthClient {
  constructor(config) {
    this.clientId = config.clientId;
    this.redirectUri = config.redirectUri;
    this.authUrl = config.authUrl;
    this.tokenUrl = config.tokenUrl;
  }

  generateCodeChallenge() {
    const codeVerifier = this.generateRandomString(128);
    const codeChallenge = btoa(
      String.fromCharCode(...new Uint8Array(
        crypto.subtle.digest('SHA-256', new TextEncoder().encode(codeVerifier))
      ))
    ).replace(/[+/]/g, c => c === '+' ? '-' : '_').replace(/=/g, '');
    
    sessionStorage.setItem('codeVerifier', codeVerifier);
    return codeChallenge;
  }

  async initiateLogin() {
    const codeChallenge = this.generateCodeChallenge();
    const state = this.generateRandomString(32);
    
    sessionStorage.setItem('oauthState', state);
    
    const params = new URLSearchParams({
      response_type: 'code',
      client_id: this.clientId,
      redirect_uri: this.redirectUri,
      scope: 'openid profile email',
      state: state,
      code_challenge: codeChallenge,
      code_challenge_method: 'S256',
    });

    window.location.href = `${this.authUrl}?${params}`;
  }
}
```

### 5. Performance Optimization

#### Rendering Optimization
```javascript
// Virtual scrolling for large lists
const VirtualList = ({ items, itemHeight, containerHeight }) => {
  const [scrollTop, setScrollTop] = useState(0);
  
  const visibleStart = Math.floor(scrollTop / itemHeight);
  const visibleEnd = Math.min(
    visibleStart + Math.ceil(containerHeight / itemHeight),
    items.length - 1
  );

  const visibleItems = items.slice(visibleStart, visibleEnd + 1);
  const totalHeight = items.length * itemHeight;
  const offsetY = visibleStart * itemHeight;

  return (
    <div
      style={{ height: containerHeight, overflow: 'auto' }}
      onScroll={(e) => setScrollTop(e.target.scrollTop)}
    >
      <div style={{ height: totalHeight, position: 'relative' }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map((item, index) => (
            <div key={visibleStart + index} style={{ height: itemHeight }}>
              {item}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
};

// Memoization strategies
const ExpensiveComponent = React.memo(({ data, onUpdate }) => {
  const processedData = useMemo(() => {
    return data.map(item => ({
      ...item,
      computed: expensiveCalculation(item),
    }));
  }, [data]);

  const handleUpdate = useCallback((id, changes) => {
    onUpdate(id, changes);
  }, [onUpdate]);

  return (
    <div>
      {processedData.map(item => (
        <Item key={item.id} data={item} onUpdate={handleUpdate} />
      ))}
    </div>
  );
}, (prevProps, nextProps) => {
  // Custom comparison function
  return prevProps.data.length === nextProps.data.length &&
         prevProps.data.every((item, index) => 
           item.id === nextProps.data[index].id &&
           item.version === nextProps.data[index].version
         );
});
```

#### Bundle Analysis and Optimization
```javascript
// Webpack Bundle Analyzer configuration
const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;

module.exports = {
  plugins: [
    new BundleAnalyzerPlugin({
      analyzerMode: 'static',
      openAnalyzer: false,
      reportFilename: 'bundle-report.html',
    }),
  ],
  optimization: {
    splitChunks: {
      chunks: 'all',
      minSize: 20000,
      maxSize: 244000,
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendors',
          priority: 10,
          reuseExistingChunk: true,
        },
        common: {
          name: 'common',
          minChunks: 2,
          priority: 5,
          reuseExistingChunk: true,
        },
      },
    },
  },
};
```

### 6. Security Considerations

#### Content Security Policy (CSP)
```html
<!-- CSP Headers -->
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; 
               script-src 'self' 'unsafe-inline' https://apis.google.com;
               style-src 'self' 'unsafe-inline' https://fonts.googleapis.com;
               img-src 'self' data: https:;
               connect-src 'self' https://api.example.com;">
```

#### XSS Prevention
```javascript
// Input sanitization
import DOMPurify from 'dompurify';

const sanitizeHTML = (dirty) => {
  return DOMPurify.sanitize(dirty, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a'],
    ALLOWED_ATTR: ['href'],
  });
};

// Safe HTML rendering
const SafeHTML = ({ content }) => {
  const sanitizedContent = useMemo(() => sanitizeHTML(content), [content]);
  
  return (
    <div dangerouslySetInnerHTML={{ __html: sanitizedContent }} />
  );
};
```

#### CSRF Protection
```javascript
// CSRF token handling
const csrfToken = document.querySelector('meta[name="csrf-token"]')?.getAttribute('content');

const apiClient = axios.create({
  baseURL: '/api',
  headers: {
    'X-CSRF-TOKEN': csrfToken,
  },
});

// Request interceptor for dynamic token refresh
apiClient.interceptors.request.use(async (config) => {
  const token = await getCSRFToken();
  config.headers['X-CSRF-TOKEN'] = token;
  return config;
});
```

### 7. Error Handling & Monitoring

#### Error Boundaries and Fallbacks
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null, errorInfo: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    this.setState({
      error: error,
      errorInfo: errorInfo,
    });

    // Log to monitoring service
    if (this.props.onError) {
      this.props.onError(error, errorInfo);
    }
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback || <DefaultErrorFallback />;
    }

    return this.props.children;
  }
}

// Usage with error monitoring
const AppWithErrorHandling = () => (
  <ErrorBoundary
    onError={(error, errorInfo) => {
      // Send to monitoring service
      Sentry.captureException(error, {
        contexts: {
          react: {
            componentStack: errorInfo.componentStack,
          },
        },
      });
    }}
    fallback={<ErrorFallback />}
  >
    <App />
  </ErrorBoundary>
);
```

#### Global Error Handling
```javascript
// Unhandled promise rejections
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled promise rejection:', event.reason);
  
  // Report to monitoring service
  reportError({
    type: 'unhandledrejection',
    error: event.reason,
    timestamp: new Date().toISOString(),
  });
  
  // Prevent the default browser behavior
  event.preventDefault();
});

// Global error handler
window.addEventListener('error', (event) => {
  console.error('Global error:', event.error);
  
  reportError({
    type: 'javascript',
    message: event.message,
    filename: event.filename,
    lineno: event.lineno,
    colno: event.colno,
    error: event.error,
  });
});
```

### 8. Real-time Data & WebSocket Management

#### WebSocket Connection Management
```javascript
class WebSocketManager {
  constructor(url, options = {}) {
    this.url = url;
    this.options = {
      reconnectAttempts: 5,
      reconnectInterval: 1000,
      ...options,
    };
    this.ws = null;
    this.reconnectCount = 0;
    this.listeners = new Map();
  }

  connect() {
    this.ws = new WebSocket(this.url);
    
    this.ws.onopen = () => {
      console.log('WebSocket connected');
      this.reconnectCount = 0;
      this.emit('connect');
    };

    this.ws.onmessage = (event) => {
      try {
        const data = JSON.parse(event.data);
        this.emit(data.type, data.payload);
      } catch (error) {
        console.error('Failed to parse WebSocket message:', error);
      }
    };

    this.ws.onclose = () => {
      console.log('WebSocket disconnected');
      this.emit('disconnect');
      this.attemptReconnect();
    };

    this.ws.onerror = (error) => {
      console.error('WebSocket error:', error);
      this.emit('error', error);
    };
  }

  attemptReconnect() {
    if (this.reconnectCount < this.options.reconnectAttempts) {
      this.reconnectCount++;
      setTimeout(() => {
        console.log(`Reconnecting... (${this.reconnectCount}/${this.options.reconnectAttempts})`);
        this.connect();
      }, this.options.reconnectInterval * this.reconnectCount);
    }
  }

  on(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);
  }

  emit(event, data) {
    if (this.listeners.has(event)) {
      this.listeners.get(event).forEach(callback => callback(data));
    }
  }

  send(type, payload) {
    if (this.ws && this.ws.readyState === WebSocket.OPEN) {
      this.ws.send(JSON.stringify({ type, payload }));
    }
  }
}

// React hook for WebSocket
const useWebSocket = (url) => {
  const [socket, setSocket] = useState(null);
  const [isConnected, setIsConnected] = useState(false);

  useEffect(() => {
    const ws = new WebSocketManager(url);
    
    ws.on('connect', () => setIsConnected(true));
    ws.on('disconnect', () => setIsConnected(false));
    
    ws.connect();
    setSocket(ws);

    return () => {
      ws.ws?.close();
    };
  }, [url]);

  return { socket, isConnected };
};
```

### 9. SEO and Meta Management

#### Dynamic Meta Tags
```javascript
// React Helmet for meta management
import { Helmet } from 'react-helmet-async';

const SEOComponent = ({ title, description, image, url }) => {
  return (
    <Helmet>
      <title>{title}</title>
      <meta name="description" content={description} />
      
      {/* Open Graph */}
      <meta property="og:title" content={title} />
      <meta property="og:description" content={description} />
      <meta property="og:image" content={image} />
      <meta property="og:url" content={url} />
      <meta property="og:type" content="website" />
      
      {/* Twitter Cards */}
      <meta name="twitter:card" content="summary_large_image" />
      <meta name="twitter:title" content={title} />
      <meta name="twitter:description" content={description} />
      <meta name="twitter:image" content={image} />
      
      {/* Structured Data */}
      <script type="application/ld+json">
        {JSON.stringify({
          "@context": "https://schema.org",
          "@type": "WebPage",
          "name": title,
          "description": description,
          "url": url,
        })}
      </script>
    </Helmet>
  );
};
```

#### Server-Side Rendering (SSR) Considerations
```javascript
// Next.js SSR example
export async function getServerSideProps(context) {
  const { params, req, res } = context;
  
  try {
    const data = await fetchDataForPage(params.id);
    
    return {
      props: {
        data,
        timestamp: Date.now(),
      },
    };
  } catch (error) {
    return {
      notFound: true,
    };
  }
}

// Static generation with ISR
export async function getStaticProps({ params }) {
  const data = await fetchStaticData(params.slug);
  
  return {
    props: { data },
    revalidate: 60, // Regenerate page every 60 seconds
  };
}
```

## Tools and Technologies

### Development
- React/Vue/Angular for component frameworks
- TypeScript for type safety
- Styled-components/CSS Modules for styling
- Storybook for component development

### Testing
- Jest for unit testing
- React Testing Library for component testing
- Cypress for end-to-end testing
- Chromatic for visual regression testing

### Design
- Figma/Sketch for design specifications
- Design tokens for consistent styling
- Component libraries (Material-UI, Ant Design)

## Conclusion

Effective UI component design requires balancing immediate needs with long-term maintainability. By following these principles and patterns, you can create components that are reusable, testable, and scalable across your application.