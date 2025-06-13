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

#### What is Scalability?
Scalability is the ability of a system to handle increased load by adding resources to the system. In frontend development, this means designing applications that can handle growing user bases, increasing data volumes, and expanding feature sets without significant performance degradation.

#### Why is Scalability Important?
- **User Experience**: Maintains performance as your application grows
- **Business Growth**: Supports increasing user base without complete rewrites
- **Cost Efficiency**: Prevents over-engineering while ensuring future capacity
- **Team Productivity**: Enables multiple teams to work independently on different parts

#### Types of Scaling

##### Horizontal vs Vertical Scaling
- **Horizontal**: Multiple instances, load balancing, CDN distribution
- **Vertical**: Increasing server resources, database optimization
- **Frontend Focus**: Code splitting, lazy loading, micro-frontends

#### When to Consider Scalability?
- Application bundle size exceeds 1MB
- Initial page load time > 3 seconds
- Multiple teams working on the same codebase
- Planning for 10x user growth
- Complex feature requirements across different domains

#### Micro-Frontends Architecture

**What are Micro-Frontends?**
Micro-frontends extend the concept of microservices to frontend development. They break down a monolithic frontend application into smaller, more manageable pieces that can be developed, tested, and deployed independently by different teams.

**Why Use Micro-Frontends?**
- **Team Independence**: Different teams can work on separate features without conflicts
- **Technology Diversity**: Teams can choose different frameworks/libraries for their domain
- **Deployment Flexibility**: Independent deployment cycles reduce coordination overhead
- **Fault Isolation**: Issues in one micro-frontend don't bring down the entire application
- **Scalable Development**: Easier to scale development teams and maintain code quality

**When to Use Micro-Frontends?**
- Large applications with multiple business domains
- Multiple development teams (>3 teams)
- Different teams have different technology preferences
- Need for independent deployment cycles
- Legacy system modernization

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

**What is Bundle Optimization?**
Bundle optimization involves reducing the size and improving the loading performance of JavaScript bundles delivered to users. This includes techniques like code splitting, tree shaking, and lazy loading.

**Why is Bundle Optimization Important?**
- **Faster Load Times**: Smaller bundles download faster, improving user experience
- **Better Core Web Vitals**: Improves metrics like First Contentful Paint and Largest Contentful Paint
- **Reduced Data Usage**: Especially important for mobile users with limited data plans
- **Better SEO**: Search engines favor faster-loading websites
- **Increased Conversion**: Studies show that even 100ms delays can impact conversion rates

**When to Optimize Bundles?**
- Bundle size > 250KB compressed
- First load time > 2 seconds
- Mobile performance scores < 90
- High bounce rates correlating with slow load times

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

#### What is Caching?
Caching is the process of storing frequently accessed data in a location that allows for faster retrieval. In frontend applications, caching can occur at multiple levels: browser cache, application memory, service workers, and CDN.

#### Why is Caching Important?
- **Performance**: Reduces load times by serving cached content instead of re-fetching
- **Bandwidth Savings**: Reduces network requests and data transfer
- **Offline Capability**: Enables applications to work without internet connectivity
- **Server Load Reduction**: Fewer requests to backend services
- **Better User Experience**: Instant loading of previously accessed content

#### Types of Frontend Caching
1. **Browser Cache**: Automatic caching of static resources
2. **Application Cache**: In-memory storage of API responses and computed data
3. **Service Worker Cache**: Programmable cache for offline-first experiences
4. **CDN Cache**: Global distribution of static assets

#### When to Implement Caching?
- Frequent API calls to the same endpoints
- Large datasets that don't change often
- Static assets (images, fonts, stylesheets)
- Expensive computations that can be memoized
- Applications requiring offline functionality

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

#### What is State Management Architecture?
State management architecture refers to the systematic approach of organizing, storing, and manipulating application state in a frontend application. It encompasses the patterns, tools, and strategies used to manage data flow, component communication, and state persistence across the entire application lifecycle.

#### Why is State Management Important?
- **Predictable Data Flow**: Ensures consistent behavior and easier debugging
- **Component Communication**: Enables data sharing between components without prop drilling
- **Performance Optimization**: Prevents unnecessary re-renders and optimizes update cycles
- **Developer Experience**: Provides clear patterns for state mutations and side effects
- **Scalability**: Maintains code organization as applications grow in complexity
- **Time Travel Debugging**: Enables advanced debugging capabilities with tools like Redux DevTools

#### Types of State Management

##### Local State
- **What**: State contained within a single component
- **When to Use**: Simple component interactions, form inputs, UI toggles
- **Tools**: useState, useReducer, class component state

##### Global State  
- **What**: State shared across multiple components throughout the application
- **When to Use**: User authentication, theme settings, shopping cart, complex app state
- **Tools**: Context API, Redux, Zustand, Recoil

##### Server State
- **What**: Data fetched from and synchronized with external APIs
- **When to Use**: API data, caching, background updates, optimistic updates
- **Tools**: React Query, SWR, Apollo Client, RTK Query

#### When to Consider Different State Management Patterns?

##### Use Local State When:
- State is only needed within a single component
- Simple boolean flags or form inputs
- Component-specific UI state (expanded/collapsed)
- Temporary state that doesn't persist

##### Use Context API When:
- State needs to be shared across multiple components
- Avoiding prop drilling through many levels
- Theme providers or configuration settings
- Small to medium applications with simple state needs

##### Use Redux/Zustand When:
- Complex state interactions across many components
- Need for time-travel debugging
- Large applications with multiple teams
- Complex async logic and side effects

##### Use Server State Libraries When:
- Fetching data from APIs
- Need caching and background synchronization
- Optimistic updates and offline support
- Complex server state management requirements

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

#### What is Authentication & Authorization?
Authentication is the process of verifying who a user is, while authorization determines what an authenticated user is allowed to do. In frontend applications, this involves managing user credentials, tokens, permissions, and protecting routes and resources based on user identity and roles.

#### Why is Authentication & Authorization Necessary?
- **Security**: Protects sensitive data and functionality from unauthorized access
- **User Experience**: Provides personalized content and features based on user identity
- **Compliance**: Meets regulatory requirements for data protection (GDPR, HIPAA, etc.)
- **Business Logic**: Enables role-based features and premium content access
- **Data Integrity**: Prevents unauthorized modifications to user data
- **Audit Trail**: Tracks user actions for security and compliance purposes

#### Types of Authentication Approaches

##### Token-Based Authentication (JWT)
- **What**: Uses JSON Web Tokens to maintain user sessions
- **When to Use**: Stateless applications, microservices, mobile apps
- **Advantages**: Stateless, scalable, cross-domain support
- **Considerations**: Token expiration, secure storage, refresh mechanisms

##### Session-Based Authentication
- **What**: Server maintains session state with cookies for identification
- **When to Use**: Traditional web applications, same-origin requests
- **Advantages**: Automatic cookie handling, server-side session control
- **Considerations**: CSRF protection, cookie security, scalability

##### OAuth 2.0 / OpenID Connect
- **What**: Delegated authorization protocol for third-party authentication
- **When to Use**: Social login, enterprise SSO, API access delegation
- **Advantages**: No password handling, standardized protocol, trusted providers
- **Considerations**: Redirect flows, token management, provider dependencies

#### Authorization Patterns

##### Role-Based Access Control (RBAC)
- **What**: Users assigned roles with specific permissions
- **When to Use**: Clear organizational hierarchies, predefined user types
- **Example**: Admin, Manager, User roles with different capabilities

##### Attribute-Based Access Control (ABAC)
- **What**: Access decisions based on attributes (user, resource, environment)
- **When to Use**: Complex authorization rules, dynamic permissions
- **Example**: Access based on department, location, time of day

##### Resource-Based Authorization
- **What**: Permissions tied to specific resources or data objects
- **When to Use**: Multi-tenant applications, user-owned content
- **Example**: Users can only edit their own posts

#### When to Implement Different Authentication Strategies?

##### Implement JWT When:
- Building SPAs or mobile applications
- Need stateless authentication
- Microservices architecture
- Cross-domain authentication required
- API-first applications

##### Implement Session-Based When:
- Traditional server-rendered applications
- Same-origin requests predominantly
- Need server-side session control
- Simpler security model required

##### Implement OAuth When:
- Social login integration needed
- Enterprise SSO requirements
- Third-party API access
- Avoiding password management

#### Security Considerations for Frontend Auth:
- **Token Storage**: Secure storage in httpOnly cookies or secure storage
- **Token Expiration**: Implement proper refresh token rotation
- **HTTPS Only**: Always use secure connections for authentication
- **Input Validation**: Sanitize all authentication-related inputs
- **Error Handling**: Avoid exposing sensitive information in error messages

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

#### What is Performance Optimization?
Performance optimization in frontend development involves improving the speed, responsiveness, and efficiency of web applications. It encompasses various techniques to reduce load times, minimize resource usage, optimize rendering, and enhance user experience through faster interactions and smoother animations.

#### Why is Performance Optimization Critical?
- **User Experience**: Faster applications lead to better user satisfaction and engagement
- **Business Impact**: Studies show that even 100ms delays can reduce conversion rates by 7%
- **SEO Benefits**: Search engines favor faster websites in rankings
- **Mobile Performance**: Critical for mobile users with slower connections and limited resources
- **Accessibility**: Improved performance benefits users with disabilities and assistive technologies
- **Cost Efficiency**: Reduced server load and bandwidth usage lower operational costs
- **Competitive Advantage**: Faster applications outperform slower competitors

#### Types of Performance Optimization

##### Loading Performance
- **What**: Optimizing initial page load and resource delivery
- **Focus Areas**: Bundle size, code splitting, lazy loading, caching
- **Metrics**: First Contentful Paint (FCP), Largest Contentful Paint (LCP), Time to Interactive (TTI)

##### Runtime Performance  
- **What**: Optimizing application behavior during user interactions
- **Focus Areas**: Rendering optimization, memory management, efficient algorithms
- **Metrics**: Frame rate, memory usage, CPU utilization

##### Perceived Performance
- **What**: Making applications feel faster through UI/UX techniques
- **Focus Areas**: Loading states, progressive rendering, skeleton screens
- **Metrics**: User satisfaction, perceived speed, engagement metrics

#### When to Implement Performance Optimization?

##### Critical Performance Indicators:
- **Bundle Size**: > 250KB compressed requires immediate attention
- **Load Time**: > 3 seconds for initial page load
- **FCP**: > 1.8 seconds for first meaningful content
- **LCP**: > 2.5 seconds for largest content element
- **CLS**: > 0.1 for cumulative layout shift
- **FID**: > 100ms for first input delay

##### Performance Budget Approach:
- Set performance budgets for different metrics
- Monitor performance continuously in CI/CD pipeline
- Implement performance regression testing
- Regular performance audits and optimizations

#### Common Performance Optimization Strategies

##### Code Splitting and Lazy Loading
- **What**: Split code into chunks and load only when needed
- **When**: Large applications, route-based splitting, feature-based splitting
- **Impact**: Reduces initial bundle size, faster initial load

##### Memoization and Caching
- **What**: Cache expensive computations and API responses
- **When**: Complex calculations, frequent API calls, expensive renders
- **Tools**: React.memo, useMemo, useCallback, React Query

##### Virtual Scrolling
- **What**: Render only visible items in large lists
- **When**: Lists with > 100 items, infinite scrolling, data tables
- **Impact**: Reduces DOM nodes, improves scroll performance

##### Image Optimization
- **What**: Optimize images for web delivery
- **When**: Content-heavy applications, e-commerce, galleries
- **Techniques**: WebP format, responsive images, lazy loading, CDN

##### Tree Shaking and Dead Code Elimination
- **What**: Remove unused code from bundles
- **When**: Large applications, many dependencies
- **Tools**: Webpack, Rollup, ES6 modules

#### Web Vitals and Metrics

##### Core Web Vitals:
- **LCP (Largest Contentful Paint)**: Loading performance
- **FID (First Input Delay)**: Interactivity
- **CLS (Cumulative Layout Shift)**: Visual stability

##### Additional Metrics:
- **TTFB (Time to First Byte)**: Server response time
- **FCP (First Contentful Paint)**: Initial render
- **TTI (Time to Interactive)**: Full interactivity

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

#### What are Security Considerations?
Security considerations in frontend development encompass the practices, techniques, and protocols used to protect web applications and users from various security threats. This includes preventing attacks like XSS, CSRF, injection attacks, and ensuring secure data transmission and storage.

#### Why are Security Considerations Essential?
- **Data Protection**: Safeguards sensitive user information and business data
- **User Trust**: Maintains user confidence in your application and brand
- **Compliance**: Meets regulatory requirements (GDPR, HIPAA, PCI-DSS)
- **Business Continuity**: Prevents costly security breaches and downtime
- **Legal Liability**: Reduces legal risks associated with data breaches
- **Reputation Management**: Protects company reputation from security incidents
- **Financial Impact**: Prevents financial losses from fraud and cyber attacks

#### Common Frontend Security Threats

##### Cross-Site Scripting (XSS)
- **What**: Injection of malicious scripts into web applications
- **Types**: Reflected XSS, Stored XSS, DOM-based XSS
- **Impact**: Cookie theft, session hijacking, data exfiltration
- **Prevention**: Input sanitization, output encoding, CSP headers

##### Cross-Site Request Forgery (CSRF)
- **What**: Unauthorized actions performed on behalf of authenticated users
- **Impact**: Account takeover, unauthorized transactions, data modification
- **Prevention**: CSRF tokens, SameSite cookies, origin validation

##### Clickjacking
- **What**: Tricking users into clicking hidden elements
- **Impact**: Unauthorized actions, credential theft, malware installation
- **Prevention**: X-Frame-Options header, frame-ancestors CSP directive

##### Man-in-the-Middle (MITM)
- **What**: Interception of communication between client and server
- **Impact**: Data theft, credential harvesting, traffic manipulation
- **Prevention**: HTTPS enforcement, certificate pinning, HSTS

##### Insecure Data Storage
- **What**: Sensitive data stored in insecure client-side locations
- **Impact**: Credential exposure, personal data leakage
- **Prevention**: Secure storage APIs, encryption, minimal data retention

#### When to Implement Security Measures?

##### Always Implement (Baseline Security):
- HTTPS enforcement for all communications
- Input validation and output encoding
- Authentication and authorization
- Secure cookie settings
- Content Security Policy headers

##### Implement for Sensitive Applications:
- Multi-factor authentication
- Advanced threat detection
- Security monitoring and logging
- Regular security audits
- Penetration testing

##### Implement for High-Risk Scenarios:
- Financial transactions
- Healthcare applications
- Government systems
- Multi-tenant platforms
- Applications handling PII

#### Security Best Practices

##### Input Validation and Sanitization
- **Client-Side**: Basic validation for user experience
- **Server-Side**: Comprehensive validation and sanitization
- **Both Sides**: Never trust client-side validation alone

##### Secure Authentication
- **Password Requirements**: Strong password policies
- **Session Management**: Secure session handling and timeout
- **Token Security**: Proper JWT handling and rotation

##### Data Protection
- **Encryption**: Encrypt sensitive data in transit and at rest
- **Minimal Exposure**: Limit data sent to frontend
- **Secure Storage**: Use secure storage mechanisms

##### Error Handling
- **Information Disclosure**: Avoid revealing system details in errors
- **Logging**: Log security events without exposing sensitive data
- **User Feedback**: Provide helpful but secure error messages

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

#### What is Error Handling & Monitoring?
Error handling and monitoring in frontend applications involves implementing systems to gracefully handle unexpected errors, provide meaningful feedback to users, and collect detailed information about issues for debugging and improvement. This includes error boundaries, global error handlers, logging systems, and real-time monitoring tools.

#### Why is Error Handling & Monitoring Crucial?
- **User Experience**: Prevents application crashes and provides graceful degradation
- **Debugging Efficiency**: Helps developers quickly identify and fix issues
- **System Reliability**: Maintains application stability even when components fail
- **Business Intelligence**: Provides insights into user behavior and application performance
- **Proactive Maintenance**: Enables fixing issues before they affect users
- **Quality Assurance**: Helps maintain high application quality over time
- **Customer Support**: Provides detailed error information for support teams

#### Types of Error Handling

##### Client-Side Error Handling
- **What**: Handling errors that occur in the browser/frontend application
- **Types**: JavaScript errors, rendering errors, network failures, validation errors
- **Tools**: try-catch blocks, error boundaries, global error handlers

##### Network Error Handling
- **What**: Managing failures in API calls and data fetching
- **Types**: HTTP errors, timeout errors, connectivity issues, server errors
- **Strategies**: Retry logic, fallback data, offline support, user notifications

##### User Input Error Handling
- **What**: Managing invalid or malformed user inputs
- **Types**: Form validation errors, file upload errors, format errors
- **Approaches**: Real-time validation, clear error messages, input sanitization

#### Error Handling Strategies

##### Graceful Degradation
- **What**: Application continues to function with reduced functionality when errors occur
- **When**: Non-critical feature failures, third-party service outages
- **Example**: Display cached data when API is unavailable

##### Error Boundaries
- **What**: React components that catch errors in component tree
- **When**: Preventing entire application crashes from component errors
- **Scope**: Wrapping feature sections, page-level boundaries

##### Progressive Enhancement
- **What**: Building core functionality first, then adding enhanced features
- **When**: Ensuring basic functionality works even if advanced features fail
- **Benefits**: Broader compatibility, better fallback experience

#### When to Implement Different Error Handling Approaches?

##### Implement Error Boundaries When:
- Building complex React applications
- Want to prevent cascading failures
- Need to isolate error-prone components
- Require custom error fallback UIs

##### Implement Global Error Handling When:
- Need to catch all unhandled errors
- Want centralized error reporting
- Building production applications
- Need comprehensive error monitoring

##### Implement Retry Logic When:
- Dealing with unreliable network connections
- Transient failures are common
- User experience can benefit from automatic recovery
- API calls may fail temporarily

#### Monitoring and Logging Strategies

##### Error Monitoring Tools
- **Client-Side**: Sentry, Bugsnag, LogRocket, Rollbar
- **Performance**: New Relic, DataDog, Google Analytics
- **User Session**: FullStory, Hotjar, LogRocket

##### Logging Best Practices
- **Structured Logging**: Use consistent log formats
- **Log Levels**: Error, warning, info, debug levels
- **Context Information**: Include user ID, session ID, timestamp
- **Privacy Considerations**: Avoid logging sensitive data

##### Real-Time Monitoring
- **Application Performance**: Response times, error rates, throughput
- **User Experience**: Core Web Vitals, user flows, conversion funnels
- **Business Metrics**: Feature usage, user engagement, revenue impact

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

#### What is Real-time Data & WebSocket Management?
Real-time data management involves handling live, continuously updating information in frontend applications through persistent connections like WebSockets, Server-Sent Events (SSE), or polling mechanisms. WebSocket management specifically deals with maintaining bidirectional communication channels between clients and servers for instant data synchronization.

#### Why is Real-time Data Management Needed?
- **Enhanced User Experience**: Provides immediate feedback and live updates
- **Collaboration Features**: Enables real-time collaboration and multi-user interactions
- **Data Freshness**: Ensures users always see the most current information
- **Reduced Latency**: Eliminates need for constant polling and page refreshes
- **Interactive Applications**: Powers chat systems, gaming, trading platforms
- **Operational Efficiency**: Real-time monitoring and notifications improve response times
- **Competitive Advantage**: Real-time features differentiate applications in the market

#### Types of Real-time Communication

##### WebSockets
- **What**: Full-duplex communication protocol over a single TCP connection
- **When to Use**: Bidirectional communication, low latency requirements, gaming, chat
- **Advantages**: Low overhead, persistent connection, supports binary data
- **Considerations**: Connection management, fallbacks, browser support

##### Server-Sent Events (SSE)
- **What**: Unidirectional communication from server to client
- **When to Use**: Live feeds, notifications, streaming data from server
- **Advantages**: Built-in reconnection, simple implementation, HTTP-based
- **Considerations**: One-way communication only, limited browser connections

##### WebRTC
- **What**: Peer-to-peer communication for real-time media and data
- **When to Use**: Video calls, file sharing, gaming, direct peer communication
- **Advantages**: Direct peer connection, multimedia support, low latency
- **Considerations**: Complex setup, NAT traversal, browser compatibility

##### Polling Strategies
- **Short Polling**: Regular HTTP requests at fixed intervals
- **Long Polling**: Hold requests open until data is available
- **When to Use**: Simple real-time needs, WebSocket fallbacks
- **Considerations**: Resource usage, server load, delay in updates

#### Common Use Cases for Real-time Data

##### Live Chat and Messaging
- **Requirements**: Instant message delivery, typing indicators, presence status
- **Challenges**: Message ordering, offline handling, scalability
- **Technologies**: WebSockets, Socket.IO, message queues

##### Collaborative Editing
- **Requirements**: Conflict resolution, operational transformation, cursor tracking
- **Challenges**: Concurrent edits, data consistency, performance
- **Technologies**: WebSockets, operational transform libraries, CRDTs

##### Live Data Dashboards
- **Requirements**: Real-time metrics, data visualization, alerting
- **Challenges**: Data volume, update frequency, rendering performance
- **Technologies**: WebSockets, SSE, data streaming protocols

##### Trading and Financial Data
- **Requirements**: Low latency, high frequency updates, reliability
- **Challenges**: Data accuracy, connection stability, regulatory compliance
- **Technologies**: WebSockets, binary protocols, dedicated connections

##### Gaming and Interactive Applications
- **Requirements**: Real-time state synchronization, low latency, smooth experience
- **Challenges**: Network lag, state consistency, cheating prevention
- **Technologies**: WebSockets, WebRTC, custom protocols

#### When to Implement Real-time Features?

##### Implement WebSockets When:
- Need bidirectional communication
- Low latency is critical (< 100ms)
- High frequency of updates
- Multiple concurrent users interacting
- Custom protocol requirements

##### Implement SSE When:
- Only need server-to-client updates
- Simple implementation preferred
- Built-in reconnection is valuable
- HTTP infrastructure is sufficient

##### Implement Polling When:
- Real-time requirements are relaxed
- Simple implementation needed
- Existing REST API infrastructure
- Sporadic updates are acceptable

#### Connection Management Strategies

##### Connection Lifecycle
- **Establishment**: Initial connection setup and authentication
- **Maintenance**: Keep-alive, heartbeat, connection monitoring
- **Reconnection**: Automatic reconnection on connection loss
- **Cleanup**: Proper connection closure and resource cleanup

##### Error Handling
- **Connection Failures**: Retry logic, exponential backoff
- **Network Issues**: Offline detection, queue messages
- **Server Errors**: Error categorization, fallback strategies

##### Performance Optimization
- **Connection Pooling**: Reuse connections when possible
- **Message Batching**: Group multiple updates together
- **Compression**: Reduce message size for better performance
- **Filtering**: Send only relevant updates to clients

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

#### What is SEO and Meta Management?
SEO (Search Engine Optimization) and meta management in frontend development involves optimizing web applications for search engine visibility and social media sharing. This includes managing meta tags, structured data, URL structure, content optimization, and ensuring proper indexing by search engines.

#### Why is SEO Important for Frontend Applications?
- **Organic Traffic**: Improves visibility in search engine results pages (SERPs)
- **User Discovery**: Helps users find your application through search
- **Brand Awareness**: Increases brand visibility and recognition
- **Cost-Effective Marketing**: Reduces dependency on paid advertising
- **Social Sharing**: Optimizes content appearance when shared on social platforms
- **Competitive Advantage**: Better SEO ranking than competitors drives more traffic
- **Business Growth**: More organic traffic typically leads to increased conversions

#### SEO Challenges in Frontend Applications

##### Single Page Applications (SPAs)
- **Challenge**: Dynamic content not visible to search engine crawlers
- **Solution**: Server-side rendering (SSR), pre-rendering, dynamic rendering
- **Impact**: Improved indexing of JavaScript-generated content

##### Client-Side Routing
- **Challenge**: URLs don't correspond to actual server pages
- **Solution**: History API, meta tag management, proper URL structure
- **Impact**: Each route can be indexed and ranked separately

##### Dynamic Content
- **Challenge**: Content that changes based on user interaction or data
- **Solution**: Static generation, incremental static regeneration
- **Impact**: Ensures all important content variations are indexed

#### Types of SEO Optimization

##### Technical SEO
- **What**: Optimizing technical aspects that affect search engine crawling
- **Focus Areas**: Site speed, mobile-friendliness, crawlability, indexability
- **Tools**: Google Search Console, Lighthouse, PageSpeed Insights

##### On-Page SEO
- **What**: Optimizing individual pages for specific keywords and topics
- **Focus Areas**: Title tags, meta descriptions, headings, content quality
- **Tools**: SEO analysis tools, keyword research tools

##### Structured Data
- **What**: Adding machine-readable data to help search engines understand content
- **Focus Areas**: Schema.org markup, JSON-LD, rich snippets
- **Impact**: Enhanced search results with rich snippets

#### When to Implement SEO Strategies?

##### Implement SSR/SSG When:
- Content needs to be indexed by search engines
- SEO is critical for business success
- Users frequently access content via search
- Social sharing is important
- Performance on slow networks matters

##### Implement Meta Management When:
- Building multi-page applications
- Content varies significantly between pages
- Social sharing is a key feature
- Brand representation in search results matters

##### Implement Structured Data When:
- Content fits common schema types (articles, products, events)
- Want enhanced search result appearance
- Building e-commerce or content sites
- Local business or organization pages

#### SEO Best Practices for Frontend

##### URL Structure
- **Clean URLs**: Use readable, keyword-rich URLs
- **Consistent Structure**: Maintain logical hierarchy
- **Canonical URLs**: Prevent duplicate content issues
- **URL Parameters**: Handle dynamic parameters properly

##### Content Optimization
- **Title Tags**: Unique, descriptive titles for each page
- **Meta Descriptions**: Compelling descriptions that encourage clicks
- **Heading Structure**: Proper H1-H6 hierarchy
- **Alt Text**: Descriptive alt text for images

##### Performance Optimization
- **Core Web Vitals**: Optimize LCP, FID, and CLS
- **Mobile Optimization**: Ensure mobile-first responsive design
- **Page Speed**: Minimize load times and resource sizes
- **Progressive Enhancement**: Ensure content works without JavaScript

#### Meta Management Strategies

##### Dynamic Meta Tags
- **Page-Specific**: Different meta tags for each route/page
- **Data-Driven**: Meta tags based on content or user data
- **Social Optimization**: Open Graph and Twitter Card tags
- **Localization**: Language and region-specific meta tags

##### Social Media Optimization
- **Open Graph**: Facebook and general social sharing
- **Twitter Cards**: Enhanced Twitter sharing experience
- **LinkedIn**: Professional network sharing optimization
- **Image Optimization**: Proper image sizes and formats for social sharing

#### Server-Side Rendering Considerations

##### SEO Benefits of SSR
- **Initial HTML**: Search engines see fully rendered content
- **Faster First Paint**: Improves perceived performance
- **Social Sharing**: Meta tags are immediately available
- **Accessibility**: Content is available even if JavaScript fails

##### Implementation Strategies
- **Full SSR**: Render everything on the server
- **Hybrid Approach**: SSR for public pages, CSR for authenticated areas
- **Static Generation**: Pre-render pages at build time
- **Incremental Static Regeneration**: Update static pages as needed

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

## Detailed Design Patterns & Implementation

This section provides comprehensive implementation guides for building production-ready systems from scratch, covering step-by-step design patterns, architectural decisions, and complete code examples.

### 1. State Management System Design

#### Architecture Overview
```
┌─────────────────────────────────────────────────────────────┐
│                    State Management Architecture            │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │   Actions   │  │  Reducers   │  │   Selectors │       │
│  └─────────────┘  └─────────────┘  └─────────────┘       │
│         │                 │                 │             │
│         ▼                 ▼                 ▼             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │              Central Store                           │  │
│  └─────────────────────────────────────────────────────┘  │
│         │                 │                 │             │
│         ▼                 ▼                 ▼             │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐       │
│  │ Middleware  │  │ DevTools    │  │  Persistence│       │
│  └─────────────┘  └─────────────┘  └─────────────┘       │
└─────────────────────────────────────────────────────────────┘
```

#### Step-by-Step Implementation

##### Step 1: Define State Structure
```javascript
// types/state.ts
interface AppState {
  auth: AuthState;
  ui: UIState;
  data: DataState;
  cache: CacheState;
}

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  permissions: string[];
  loading: boolean;
  error: string | null;
}

interface UIState {
  theme: 'light' | 'dark';
  sidebar: {
    isOpen: boolean;
    selectedItem: string | null;
  };
  modals: Record<string, boolean>;
  notifications: Notification[];
}

interface DataState {
  entities: {
    users: Record<string, User>;
    products: Record<string, Product>;
    orders: Record<string, Order>;
  };
  collections: {
    userIds: string[];
    productIds: string[];
    orderIds: string[];
  };
  loading: Record<string, boolean>;
  errors: Record<string, string | null>;
}
```

##### Step 2: Create Action System
```javascript
// actions/types.ts
enum ActionType {
  // Auth Actions
  AUTH_LOGIN_REQUEST = 'AUTH_LOGIN_REQUEST',
  AUTH_LOGIN_SUCCESS = 'AUTH_LOGIN_SUCCESS',
  AUTH_LOGIN_FAILURE = 'AUTH_LOGIN_FAILURE',
  AUTH_LOGOUT = 'AUTH_LOGOUT',
  
  // UI Actions
  UI_TOGGLE_SIDEBAR = 'UI_TOGGLE_SIDEBAR',
  UI_SET_THEME = 'UI_SET_THEME',
  UI_SHOW_MODAL = 'UI_SHOW_MODAL',
  UI_HIDE_MODAL = 'UI_HIDE_MODAL',
  
  // Data Actions
  DATA_FETCH_REQUEST = 'DATA_FETCH_REQUEST',
  DATA_FETCH_SUCCESS = 'DATA_FETCH_SUCCESS',
  DATA_FETCH_FAILURE = 'DATA_FETCH_FAILURE',
}

// actions/creators.ts
interface BaseAction {
  type: ActionType;
  payload?: any;
  meta?: any;
  error?: boolean;
}

class ActionCreator {
  static createAction<T = any>(
    type: ActionType,
    payload?: T,
    meta?: any
  ): BaseAction {
    return {
      type,
      payload,
      meta,
      error: payload instanceof Error,
    };
  }

  // Auth Actions
  static loginRequest(credentials: LoginCredentials) {
    return this.createAction(ActionType.AUTH_LOGIN_REQUEST, credentials);
  }

  static loginSuccess(user: User, token: string) {
    return this.createAction(ActionType.AUTH_LOGIN_SUCCESS, { user, token });
  }

  static loginFailure(error: string) {
    return this.createAction(ActionType.AUTH_LOGIN_FAILURE, error);
  }

  // Async Action Creator
  static login(credentials: LoginCredentials) {
    return async (dispatch: Dispatch, getState: () => AppState) => {
      dispatch(this.loginRequest(credentials));
      
      try {
        const response = await authService.login(credentials);
        dispatch(this.loginSuccess(response.user, response.token));
        
        // Side effects
        localStorage.setItem('token', response.token);
        analyticsService.track('login_success');
        
      } catch (error) {
        dispatch(this.loginFailure(error.message));
      }
    };
  }
}
```

##### Step 3: Build Reducer System
```javascript
// reducers/auth.ts
const initialAuthState: AuthState = {
  user: null,
  token: localStorage.getItem('token'),
  isAuthenticated: false,
  permissions: [],
  loading: false,
  error: null,
};

function authReducer(
  state = initialAuthState,
  action: BaseAction
): AuthState {
  switch (action.type) {
    case ActionType.AUTH_LOGIN_REQUEST:
      return {
        ...state,
        loading: true,
        error: null,
      };

    case ActionType.AUTH_LOGIN_SUCCESS:
      return {
        ...state,
        user: action.payload.user,
        token: action.payload.token,
        isAuthenticated: true,
        permissions: action.payload.user.permissions,
        loading: false,
        error: null,
      };

    case ActionType.AUTH_LOGIN_FAILURE:
      return {
        ...state,
        user: null,
        token: null,
        isAuthenticated: false,
        permissions: [],
        loading: false,
        error: action.payload,
      };

    case ActionType.AUTH_LOGOUT:
      return {
        ...initialAuthState,
        token: null,
      };

    default:
      return state;
  }
}

// reducers/index.ts
const rootReducer = combineReducers({
  auth: authReducer,
  ui: uiReducer,
  data: dataReducer,
  cache: cacheReducer,
});
```

##### Step 4: Create Selector System
```javascript
// selectors/auth.ts
class AuthSelectors {
  static getAuthState = (state: AppState) => state.auth;
  
  static getCurrentUser = createSelector(
    [this.getAuthState],
    (auth) => auth.user
  );
  
  static isAuthenticated = createSelector(
    [this.getAuthState],
    (auth) => auth.isAuthenticated
  );
  
  static hasPermission = (permission: string) => createSelector(
    [this.getAuthState],
    (auth) => auth.permissions.includes(permission)
  );
  
  static hasAnyPermission = (permissions: string[]) => createSelector(
    [this.getAuthState],
    (auth) => permissions.some(p => auth.permissions.includes(p))
  );
}

// selectors/data.ts with normalization
class DataSelectors {
  static getEntities = (state: AppState) => state.data.entities;
  static getCollections = (state: AppState) => state.data.collections;
  
  static getUserById = (id: string) => createSelector(
    [this.getEntities],
    (entities) => entities.users[id]
  );
  
  static getAllUsers = createSelector(
    [this.getEntities, this.getCollections],
    (entities, collections) => 
      collections.userIds.map(id => entities.users[id]).filter(Boolean)
  );
  
  static getUsersWithProducts = createSelector(
    [this.getAllUsers, this.getEntities],
    (users, entities) => 
      users.map(user => ({
        ...user,
        products: user.productIds?.map(id => entities.products[id]) || []
      }))
  );
}
```

##### Step 5: Middleware Implementation
```javascript
// middleware/logger.ts
const loggerMiddleware: Middleware = (store) => (next) => (action) => {
  if (process.env.NODE_ENV === 'development') {
    console.group(action.type);
    console.log('Previous State:', store.getState());
    console.log('Action:', action);
  }
  
  const result = next(action);
  
  if (process.env.NODE_ENV === 'development') {
    console.log('Next State:', store.getState());
    console.groupEnd();
  }
  
  return result;
};

// middleware/persistence.ts
const persistenceMiddleware: Middleware = (store) => (next) => (action) => {
  const result = next(action);
  
  // Persist specific state slices
  const persistableActions = [
    ActionType.AUTH_LOGIN_SUCCESS,
    ActionType.UI_SET_THEME,
    ActionType.UI_TOGGLE_SIDEBAR,
  ];
  
  if (persistableActions.includes(action.type)) {
    const state = store.getState();
    localStorage.setItem('appState', JSON.stringify({
      auth: { token: state.auth.token },
      ui: { theme: state.ui.theme }
    }));
  }
  
  return result;
};

// middleware/api.ts
const apiMiddleware: Middleware = (store) => (next) => (action) => {
  // Handle API actions
  if (action.meta?.api) {
    const { endpoint, method, body, onSuccess, onFailure } = action.meta.api;
    
    fetch(endpoint, { method, body: JSON.stringify(body) })
      .then(response => response.json())
      .then(data => {
        if (onSuccess) {
          store.dispatch(onSuccess(data));
        }
      })
      .catch(error => {
        if (onFailure) {
          store.dispatch(onFailure(error.message));
        }
      });
  }
  
  return next(action);
};
```

##### Step 6: Store Configuration
```javascript
// store/index.ts
class StoreManager {
  private store: Store<AppState>;
  
  constructor() {
    this.store = this.createStore();
  }
  
  private createStore(): Store<AppState> {
    const preloadedState = this.loadPersistedState();
    
    const middleware = [
      thunk,
      loggerMiddleware,
      persistenceMiddleware,
      apiMiddleware,
    ];
    
    if (process.env.NODE_ENV === 'development') {
      middleware.push(createLogger());
    }
    
    const enhancer = compose(
      applyMiddleware(...middleware),
      process.env.NODE_ENV === 'development' && 
      window.__REDUX_DEVTOOLS_EXTENSION__ 
        ? window.__REDUX_DEVTOOLS_EXTENSION__()
        : (f: any) => f
    );
    
    return createStore(rootReducer, preloadedState, enhancer);
  }
  
  private loadPersistedState(): Partial<AppState> {
    try {
      const serializedState = localStorage.getItem('appState');
      if (serializedState === null) return {};
      
      return JSON.parse(serializedState);
    } catch (error) {
      console.warn('Failed to load persisted state:', error);
      return {};
    }
  }
  
  getStore() {
    return this.store;
  }
  
  // Hot reloading support
  replaceReducer(nextReducer: Reducer<AppState>) {
    this.store.replaceReducer(nextReducer);
  }
}

export const storeManager = new StoreManager();
export const store = storeManager.getStore();
```

#### Testing Strategy
```javascript
// tests/store.test.ts
describe('State Management', () => {
  let store: Store<AppState>;
  
  beforeEach(() => {
    store = createTestStore();
  });
  
  describe('Auth Flow', () => {
    it('should handle login flow correctly', async () => {
      const credentials = { email: 'test@example.com', password: 'password' };
      
      // Dispatch login action
      await store.dispatch(ActionCreator.login(credentials));
      
      const state = store.getState();
      expect(state.auth.isAuthenticated).toBe(true);
      expect(state.auth.user).toBeDefined();
    });
    
    it('should handle login failure', async () => {
      const invalidCredentials = { email: 'test@example.com', password: 'wrong' };
      
      await store.dispatch(ActionCreator.login(invalidCredentials));
      
      const state = store.getState();
      expect(state.auth.isAuthenticated).toBe(false);
      expect(state.auth.error).toBeDefined();
    });
  });
  
  describe('Selectors', () => {
    it('should select user data correctly', () => {
      const state = createMockState();
      const user = AuthSelectors.getCurrentUser(state);
      
      expect(user).toMatchObject({
        id: '1',
        email: 'test@example.com',
      });
    });
  });
});
```

### 2. Authentication System Implementation

#### Architecture Overview
```
┌─────────────────────────────────────────────────────────────┐
│                Authentication System Architecture          │
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │Auth Service │ │Token Manager│ │Route Guards │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│ ┌─────────────────────────────────────────────────────┐   │
│ │            Authentication Context                    │   │
│ └─────────────────────────────────────────────────────┘   │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │Login Forms  │ │Protected    │ │Session      │           │
│ │& Components │ │Routes       │ │Management   │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

#### Step-by-Step Implementation

##### Step 1: Token Management System
```javascript
// auth/tokenManager.ts
interface TokenData {
  accessToken: string;
  refreshToken: string;
  expiresAt: number;
  tokenType: string;
}

class TokenManager {
  private static instance: TokenManager;
  private refreshPromise: Promise<string> | null = null;
  
  static getInstance(): TokenManager {
    if (!TokenManager.instance) {
      TokenManager.instance = new TokenManager();
    }
    return TokenManager.instance;
  }
  
  setTokens(tokenData: TokenData): void {
    const expiresAt = Date.now() + (tokenData.expiresAt * 1000);
    
    // Use httpOnly cookies for production
    if (this.isSecureContext()) {
      document.cookie = `accessToken=${tokenData.accessToken}; httpOnly; secure; sameSite=strict`;
      document.cookie = `refreshToken=${tokenData.refreshToken}; httpOnly; secure; sameSite=strict`;
    } else {
      // Development fallback
      sessionStorage.setItem('accessToken', tokenData.accessToken);
      sessionStorage.setItem('refreshToken', tokenData.refreshToken);
    }
    
    localStorage.setItem('tokenExpiry', expiresAt.toString());
  }
  
  getAccessToken(): string | null {
    if (this.isSecureContext()) {
      return this.getCookieValue('accessToken');
    }
    return sessionStorage.getItem('accessToken');
  }
  
  getRefreshToken(): string | null {
    if (this.isSecureContext()) {
      return this.getCookieValue('refreshToken');
    }
    return sessionStorage.getItem('refreshToken');
  }
  
  isTokenExpired(): boolean {
    const expiry = localStorage.getItem('tokenExpiry');
    if (!expiry) return true;
    
    return Date.now() > parseInt(expiry) - 60000; // 1 minute buffer
  }
  
  async getValidToken(): Promise<string | null> {
    const token = this.getAccessToken();
    
    if (!token) return null;
    
    if (!this.isTokenExpired()) {
      return token;
    }
    
    // Prevent multiple refresh requests
    if (this.refreshPromise) {
      return this.refreshPromise;
    }
    
    this.refreshPromise = this.refreshAccessToken();
    
    try {
      const newToken = await this.refreshPromise;
      this.refreshPromise = null;
      return newToken;
    } catch (error) {
      this.refreshPromise = null;
      this.clearTokens();
      throw error;
    }
  }
  
  private async refreshAccessToken(): Promise<string> {
    const refreshToken = this.getRefreshToken();
    
    if (!refreshToken) {
      throw new Error('No refresh token available');
    }
    
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ refreshToken }),
    });
    
    if (!response.ok) {
      throw new Error('Token refresh failed');
    }
    
    const tokenData = await response.json();
    this.setTokens(tokenData);
    
    return tokenData.accessToken;
  }
  
  clearTokens(): void {
    if (this.isSecureContext()) {
      document.cookie = 'accessToken=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;';
      document.cookie = 'refreshToken=; expires=Thu, 01 Jan 1970 00:00:00 UTC; path=/;';
    } else {
      sessionStorage.removeItem('accessToken');
      sessionStorage.removeItem('refreshToken');
    }
    localStorage.removeItem('tokenExpiry');
  }
  
  private isSecureContext(): boolean {
    return location.protocol === 'https:' || location.hostname === 'localhost';
  }
  
  private getCookieValue(name: string): string | null {
    const value = `; ${document.cookie}`;
    const parts = value.split(`; ${name}=`);
    if (parts.length === 2) {
      return parts.pop()?.split(';').shift() || null;
    }
    return null;
  }
}
```

##### Step 2: Authentication Service
```javascript
// auth/authService.ts
interface LoginCredentials {
  email: string;
  password: string;
  rememberMe?: boolean;
}

interface User {
  id: string;
  email: string;
  name: string;
  roles: string[];
  permissions: string[];
  avatar?: string;
}

class AuthService {
  private tokenManager = TokenManager.getInstance();
  
  async login(credentials: LoginCredentials): Promise<{ user: User; tokens: TokenData }> {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(credentials),
    });
    
    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message || 'Login failed');
    }
    
    const data = await response.json();
    
    // Store tokens
    this.tokenManager.setTokens(data.tokens);
    
    // Track login event
    this.trackEvent('login_success', {
      userId: data.user.id,
      loginMethod: 'email',
    });
    
    return data;
  }
  
  async logout(): Promise<void> {
    const token = this.tokenManager.getAccessToken();
    
    if (token) {
      try {
        await fetch('/api/auth/logout', {
          method: 'POST',
          headers: { 
            'Authorization': `Bearer ${token}`,
            'Content-Type': 'application/json'
          },
        });
      } catch (error) {
        console.warn('Logout request failed:', error);
      }
    }
    
    this.tokenManager.clearTokens();
    this.trackEvent('logout');
    
    // Redirect to login page
    window.location.href = '/login';
  }
  
  async getCurrentUser(): Promise<User | null> {
    try {
      const token = await this.tokenManager.getValidToken();
      
      if (!token) return null;
      
      const response = await fetch('/api/auth/me', {
        headers: { 'Authorization': `Bearer ${token}` },
      });
      
      if (!response.ok) {
        throw new Error('Failed to fetch user');
      }
      
      return await response.json();
    } catch (error) {
      console.error('Failed to get current user:', error);
      return null;
    }
  }
  
  async resetPassword(email: string): Promise<void> {
    const response = await fetch('/api/auth/reset-password', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email }),
    });
    
    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message || 'Password reset failed');
    }
  }
  
  async changePassword(currentPassword: string, newPassword: string): Promise<void> {
    const token = await this.tokenManager.getValidToken();
    
    if (!token) {
      throw new Error('Not authenticated');
    }
    
    const response = await fetch('/api/auth/change-password', {
      method: 'PUT',
      headers: {
        'Authorization': `Bearer ${token}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ currentPassword, newPassword }),
    });
    
    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.message || 'Password change failed');
    }
  }
  
  private trackEvent(event: string, data?: any): void {
    // Analytics tracking
    if (window.gtag) {
      window.gtag('event', event, data);
    }
  }
}
```

##### Step 3: Authentication Context
```javascript
// auth/AuthContext.tsx
interface AuthContextType {
  user: User | null;
  isLoading: boolean;
  isAuthenticated: boolean;
  login: (credentials: LoginCredentials) => Promise<void>;
  logout: () => Promise<void>;
  resetPassword: (email: string) => Promise<void>;
  hasPermission: (permission: string) => boolean;
  hasRole: (role: string) => boolean;
}

const AuthContext = createContext<AuthContextType | null>(null);

export const useAuth = (): AuthContextType => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};

interface AuthProviderProps {
  children: React.ReactNode;
}

export const AuthProvider: React.FC<AuthProviderProps> = ({ children }) => {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const authService = useMemo(() => new AuthService(), []);
  
  const isAuthenticated = useMemo(() => !!user, [user]);
  
  // Initialize authentication state
  useEffect(() => {
    const initializeAuth = async () => {
      try {
        const currentUser = await authService.getCurrentUser();
        setUser(currentUser);
      } catch (error) {
        console.error('Failed to initialize auth:', error);
      } finally {
        setIsLoading(false);
      }
    };
    
    initializeAuth();
  }, [authService]);
  
  // Handle token refresh on app focus
  useEffect(() => {
    const handleFocus = async () => {
      if (user) {
        try {
          const currentUser = await authService.getCurrentUser();
          setUser(currentUser);
        } catch (error) {
          console.error('Failed to refresh user on focus:', error);
          setUser(null);
        }
      }
    };
    
    window.addEventListener('focus', handleFocus);
    return () => window.removeEventListener('focus', handleFocus);
  }, [user, authService]);
  
  const login = useCallback(async (credentials: LoginCredentials) => {
    setIsLoading(true);
    try {
      const { user: loggedInUser } = await authService.login(credentials);
      setUser(loggedInUser);
    } catch (error) {
      throw error;
    } finally {
      setIsLoading(false);
    }
  }, [authService]);
  
  const logout = useCallback(async () => {
    setIsLoading(true);
    try {
      await authService.logout();
      setUser(null);
    } catch (error) {
      console.error('Logout error:', error);
    } finally {
      setIsLoading(false);
    }
  }, [authService]);
  
  const resetPassword = useCallback(async (email: string) => {
    await authService.resetPassword(email);
  }, [authService]);
  
  const hasPermission = useCallback((permission: string): boolean => {
    return user?.permissions.includes(permission) ?? false;
  }, [user]);
  
  const hasRole = useCallback((role: string): boolean => {
    return user?.roles.includes(role) ?? false;
  }, [user]);
  
  const value: AuthContextType = {
    user,
    isLoading,
    isAuthenticated,
    login,
    logout,
    resetPassword,
    hasPermission,
    hasRole,
  };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
};
```

##### Step 4: Route Protection
```javascript
// auth/ProtectedRoute.tsx
interface ProtectedRouteProps {
  children: React.ReactNode;
  requiredPermissions?: string[];
  requiredRoles?: string[];
  redirectTo?: string;
  fallback?: React.ComponentType;
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({
  children,
  requiredPermissions = [],
  requiredRoles = [],
  redirectTo = '/login',
  fallback: Fallback,
}) => {
  const { user, isLoading, isAuthenticated, hasPermission, hasRole } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  
  useEffect(() => {
    if (!isLoading && !isAuthenticated) {
      // Store intended destination
      navigate(redirectTo, {
        state: { from: location.pathname },
        replace: true,
      });
    }
  }, [isLoading, isAuthenticated, navigate, redirectTo, location]);
  
  if (isLoading) {
    return <LoadingSpinner />;
  }
  
  if (!isAuthenticated) {
    return null;
  }
  
  // Check permissions
  if (requiredPermissions.length > 0) {
    const hasAllPermissions = requiredPermissions.every(hasPermission);
    if (!hasAllPermissions) {
      return Fallback ? <Fallback /> : <UnauthorizedPage />;
    }
  }
  
  // Check roles
  if (requiredRoles.length > 0) {
    const hasRequiredRole = requiredRoles.some(hasRole);
    if (!hasRequiredRole) {
      return Fallback ? <Fallback /> : <UnauthorizedPage />;
    }
  }
  
  return <>{children}</>;
};

// Higher-order component version
export const withAuth = <P extends object>(
  Component: React.ComponentType<P>,
  options: Omit<ProtectedRouteProps, 'children'> = {}
) => {
  return function AuthenticatedComponent(props: P) {
    return (
      <ProtectedRoute {...options}>
        <Component {...props} />
      </ProtectedRoute>
    );
  };
};
```

##### Step 5: HTTP Client with Auth
```javascript
// api/httpClient.ts
class HttpClient {
  private tokenManager = TokenManager.getInstance();
  private baseURL: string;
  
  constructor(baseURL: string = '/api') {
    this.baseURL = baseURL;
  }
  
  async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const url = `${this.baseURL}${endpoint}`;
    
    // Get valid token
    let token: string | null = null;
    try {
      token = await this.tokenManager.getValidToken();
    } catch (error) {
      // Token refresh failed, redirect to login
      window.location.href = '/login';
      throw error;
    }
    
    // Prepare headers
    const headers: HeadersInit = {
      'Content-Type': 'application/json',
      ...options.headers,
    };
    
    if (token) {
      headers['Authorization'] = `Bearer ${token}`;
    }
    
    const config: RequestInit = {
      ...options,
      headers,
    };
    
    try {
      const response = await fetch(url, config);
      
      if (response.status === 401) {
        // Unauthorized, clear tokens and redirect
        this.tokenManager.clearTokens();
        window.location.href = '/login';
        throw new Error('Unauthorized');
      }
      
      if (!response.ok) {
        const error = await response.json();
        throw new Error(error.message || `HTTP ${response.status}`);
      }
      
      const contentType = response.headers.get('content-type');
      if (contentType && contentType.includes('application/json')) {
        return await response.json();
      }
      
      return response.text() as any;
    } catch (error) {
      if (error instanceof TypeError) {
        throw new Error('Network error');
      }
      throw error;
    }
  }
  
  get<T>(endpoint: string, params?: Record<string, any>): Promise<T> {
    const url = params ? `${endpoint}?${new URLSearchParams(params)}` : endpoint;
    return this.request<T>(url, { method: 'GET' });
  }
  
  post<T>(endpoint: string, data?: any): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'POST',
      body: data ? JSON.stringify(data) : undefined,
    });
  }
  
  put<T>(endpoint: string, data?: any): Promise<T> {
    return this.request<T>(endpoint, {
      method: 'PUT',
      body: data ? JSON.stringify(data) : undefined,
    });
  }
  
  delete<T>(endpoint: string): Promise<T> {
    return this.request<T>(endpoint, { method: 'DELETE' });
  }
}

export const httpClient = new HttpClient();
```

### 3. Caching Layer Implementation

#### Architecture Overview
```
┌─────────────────────────────────────────────────────────────┐
│                    Caching Layer Architecture              │
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │Memory Cache │ │Storage Cache│ │Service      │           │
│ │(LRU)        │ │(IndexedDB)  │ │Worker Cache │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│ ┌─────────────────────────────────────────────────────┐   │
│ │              Cache Manager                           │   │
│ └─────────────────────────────────────────────────────┘   │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │Cache        │ │Invalidation │ │Analytics    │           │
│ │Strategies   │ │Policies     │ │& Metrics    │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

#### Step-by-Step Implementation

##### Step 1: Memory Cache with LRU
```javascript
// cache/memoryCache.ts
interface CacheItem<T> {
  value: T;
  expiresAt: number;
  accessCount: number;
  lastAccessed: number;
}

class LRUCache<T> {
  private cache = new Map<string, CacheItem<T>>();
  private maxSize: number;
  private defaultTTL: number;
  
  constructor(maxSize: number = 100, defaultTTL: number = 300000) { // 5 minutes
    this.maxSize = maxSize;
    this.defaultTTL = defaultTTL;
  }
  
  set(key: string, value: T, ttl?: number): void {
    const expiresAt = Date.now() + (ttl || this.defaultTTL);
    
    if (this.cache.has(key)) {
      // Update existing item
      this.cache.delete(key);
    } else if (this.cache.size >= this.maxSize) {
      // Remove least recently used item
      this.evictLRU();
    }
    
    this.cache.set(key, {
      value,
      expiresAt,
      accessCount: 0,
      lastAccessed: Date.now(),
    });
  }
  
  get(key: string): T | null {
    const item = this.cache.get(key);
    
    if (!item) return null;
    
    // Check expiration
    if (Date.now() > item.expiresAt) {
      this.cache.delete(key);
      return null;
    }
    
    // Update access statistics
    item.accessCount++;
    item.lastAccessed = Date.now();
    
    // Move to end (most recently used)
    this.cache.delete(key);
    this.cache.set(key, item);
    
    return item.value;
  }
  
  has(key: string): boolean {
    const item = this.cache.get(key);
    if (!item) return false;
    
    if (Date.now() > item.expiresAt) {
      this.cache.delete(key);
      return false;
    }
    
    return true;
  }
  
  delete(key: string): boolean {
    return this.cache.delete(key);
  }
  
  clear(): void {
    this.cache.clear();
  }
  
  private evictLRU(): void {
    const oldestKey = this.cache.keys().next().value;
    if (oldestKey) {
      this.cache.delete(oldestKey);
    }
  }
  
  // Analytics
  getStats(): {
    size: number;
    maxSize: number;
    hitRatio: number;
    totalAccess: number;
  } {
    let totalAccess = 0;
    for (const item of this.cache.values()) {
      totalAccess += item.accessCount;
    }
    
    return {
      size: this.cache.size,
      maxSize: this.maxSize,
      hitRatio: totalAccess > 0 ? this.cache.size / totalAccess : 0,
      totalAccess,
    };
  }
}
```

##### Step 2: Persistent Storage Cache
```javascript
// cache/storageCache.ts
interface StorageItem<T> {
  value: T;
  expiresAt: number;
  version: string;
  metadata?: Record<string, any>;
}

class StorageCache<T> {
  private dbName: string;
  private version: number;
  private db: IDBDatabase | null = null;
  
  constructor(dbName: string = 'AppCache', version: number = 1) {
    this.dbName = dbName;
    this.version = version;
  }
  
  async init(): Promise<void> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, this.version);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        this.db = request.result;
        resolve();
      };
      
      request.onupgradeneeded = (event) => {
        const db = (event.target as IDBOpenDBRequest).result;
        
        if (!db.objectStoreNames.contains('cache')) {
          const store = db.createObjectStore('cache', { keyPath: 'key' });
          store.createIndex('expiresAt', 'expiresAt');
        }
      };
    });
  }
  
  async set(
    key: string, 
    value: T, 
    ttl: number = 3600000, // 1 hour
    metadata?: Record<string, any>
  ): Promise<void> {
    if (!this.db) await this.init();
    
    const item: StorageItem<T> = {
      value,
      expiresAt: Date.now() + ttl,
      version: this.version.toString(),
      metadata,
    };
    
    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction(['cache'], 'readwrite');
      const store = transaction.objectStore('cache');
      const request = store.put({ key, ...item });
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve();
    });
  }
  
  async get(key: string): Promise<T | null> {
    if (!this.db) await this.init();
    
    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction(['cache'], 'readonly');
      const store = transaction.objectStore('cache');
      const request = store.get(key);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => {
        const result = request.result;
        
        if (!result) {
          resolve(null);
          return;
        }
        
        // Check expiration
        if (Date.now() > result.expiresAt) {
          this.delete(key);
          resolve(null);
          return;
        }
        
        resolve(result.value);
      };
    });
  }
  
  async delete(key: string): Promise<void> {
    if (!this.db) await this.init();
    
    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction(['cache'], 'readwrite');
      const store = transaction.objectStore('cache');
      const request = store.delete(key);
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve();
    });
  }
  
  async clear(): Promise<void> {
    if (!this.db) await this.init();
    
    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction(['cache'], 'readwrite');
      const store = transaction.objectStore('cache');
      const request = store.clear();
      
      request.onerror = () => reject(request.error);
      request.onsuccess = () => resolve();
    });
  }
  
  async cleanup(): Promise<number> {
    if (!this.db) await this.init();
    
    return new Promise((resolve, reject) => {
      const transaction = this.db!.transaction(['cache'], 'readwrite');
      const store = transaction.objectStore('cache');
      const index = store.index('expiresAt');
      
      const range = IDBKeyRange.upperBound(Date.now());
      const request = index.openCursor(range);
      
      let deletedCount = 0;
      
      request.onerror = () => reject(request.error);
      request.onsuccess = (event) => {
        const cursor = (event.target as IDBRequest).result;
        
        if (cursor) {
          cursor.delete();
          deletedCount++;
          cursor.continue();
        } else {
          resolve(deletedCount);
        }
      };
    });
  }
}
```

##### Step 3: Cache Manager
```javascript
// cache/cacheManager.ts
interface CacheConfig {
  memoryMaxSize: number;
  memoryTTL: number;
  persistentTTL: number;
  enablePersistent: boolean;
  enableServiceWorker: boolean;
}

class CacheManager<T> {
  private memoryCache: LRUCache<T>;
  private storageCache: StorageCache<T> | null = null;
  private config: CacheConfig;
  private metrics = {
    hits: 0,
    misses: 0,
    sets: 0,
    deletes: 0,
  };
  
  constructor(config: Partial<CacheConfig> = {}) {
    this.config = {
      memoryMaxSize: 100,
      memoryTTL: 300000, // 5 minutes
      persistentTTL: 3600000, // 1 hour
      enablePersistent: true,
      enableServiceWorker: false,
      ...config,
    };
    
    this.memoryCache = new LRUCache<T>(
      this.config.memoryMaxSize,
      this.config.memoryTTL
    );
    
    if (this.config.enablePersistent) {
      this.storageCache = new StorageCache<T>();
    }
    
    // Periodic cleanup
    setInterval(() => this.cleanup(), 600000); // 10 minutes
  }
  
  async set(key: string, value: T, options: {
    memoryTTL?: number;
    persistentTTL?: number;
    persistToDisk?: boolean;
    metadata?: Record<string, any>;
  } = {}): Promise<void> {
    this.metrics.sets++;
    
    // Set in memory cache
    this.memoryCache.set(key, value, options.memoryTTL);
    
    // Set in persistent cache if enabled
    if (this.storageCache && options.persistToDisk !== false) {
      await this.storageCache.set(
        key,
        value,
        options.persistentTTL || this.config.persistentTTL,
        options.metadata
      );
    }
  }
  
  async get(key: string): Promise<T | null> {
    // Try memory cache first
    let value = this.memoryCache.get(key);
    
    if (value !== null) {
      this.metrics.hits++;
      return value;
    }
    
    // Try persistent cache
    if (this.storageCache) {
      value = await this.storageCache.get(key);
      
      if (value !== null) {
        this.metrics.hits++;
        // Promote to memory cache
        this.memoryCache.set(key, value);
        return value;
      }
    }
    
    this.metrics.misses++;
    return null;
  }
  
  async delete(key: string): Promise<void> {
    this.metrics.deletes++;
    
    this.memoryCache.delete(key);
    
    if (this.storageCache) {
      await this.storageCache.delete(key);
    }
  }
  
  async clear(): Promise<void> {
    this.memoryCache.clear();
    
    if (this.storageCache) {
      await this.storageCache.clear();
    }
  }
  
  async invalidatePattern(pattern: string): Promise<void> {
    const regex = new RegExp(pattern);
    const keysToDelete: string[] = [];
    
    // This is a simplified implementation
    // In production, you'd need to track keys separately
    for (const key of this.getAllKeys()) {
      if (regex.test(key)) {
        keysToDelete.push(key);
      }
    }
    
    await Promise.all(keysToDelete.map(key => this.delete(key)));
  }
  
  private getAllKeys(): string[] {
    // This would need to be implemented based on your storage strategy
    return [];
  }
  
  private async cleanup(): Promise<void> {
    if (this.storageCache) {
      const deletedCount = await this.storageCache.cleanup();
      console.log(`Cleaned up ${deletedCount} expired cache entries`);
    }
  }
  
  getMetrics() {
    return {
      ...this.metrics,
      hitRatio: this.metrics.hits / (this.metrics.hits + this.metrics.misses),
      memoryStats: this.memoryCache.getStats(),
    };
  }
}
```

##### Step 4: React Hook Integration
```javascript
// hooks/useCache.ts
interface UseCacheOptions<T> {
  key: string;
  fetcher: () => Promise<T>;
  dependencies?: any[];
  enabled?: boolean;
  ttl?: number;
  staleWhileRevalidate?: boolean;
  onError?: (error: Error) => void;
  onSuccess?: (data: T) => void;
}

export function useCache<T>({
  key,
  fetcher,
  dependencies = [],
  enabled = true,
  ttl = 300000,
  staleWhileRevalidate = true,
  onError,
  onSuccess,
}: UseCacheOptions<T>) {
  const [data, setData] = useState<T | null>(null);
  const [error, setError] = useState<Error | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [isValidating, setIsValidating] = useState(false);
  
  const cacheManager = useMemo(() => new CacheManager<T>(), []);
  
  const fetchData = useCallback(async (showLoading = true) => {
    if (!enabled) return;
    
    try {
      if (showLoading) setIsLoading(true);
      setIsValidating(true);
      setError(null);
      
      // Try cache first
      const cachedData = await cacheManager.get(key);
      
      if (cachedData && !isValidating) {
        setData(cachedData);
        if (showLoading) setIsLoading(false);
        
        if (!staleWhileRevalidate) {
          setIsValidating(false);
          return;
        }
      }
      
      // Fetch fresh data
      const freshData = await fetcher();
      
      // Update cache
      await cacheManager.set(key, freshData, { memoryTTL: ttl });
      
      setData(freshData);
      onSuccess?.(freshData);
      
    } catch (err) {
      const error = err instanceof Error ? err : new Error('Unknown error');
      setError(error);
      onError?.(error);
    } finally {
      setIsLoading(false);
      setIsValidating(false);
    }
  }, [key, fetcher, enabled, ttl, staleWhileRevalidate, onError, onSuccess]);
  
  // Initial fetch and dependency changes
  useEffect(() => {
    fetchData();
  }, [fetchData, ...dependencies]);
  
  const mutate = useCallback(async (newData?: T, revalidate = true) => {
    if (newData) {
      setData(newData);
      await cacheManager.set(key, newData, { memoryTTL: ttl });
    }
    
    if (revalidate) {
      await fetchData(false);
    }
  }, [key, ttl, fetchData]);
  
  const invalidate = useCallback(async () => {
    await cacheManager.delete(key);
    setData(null);
    await fetchData();
  }, [key, fetchData]);
  
  return {
    data,
    error,
    isLoading,
    isValidating,
    mutate,
    invalidate,
    refetch: () => fetchData(),
  };
}

// Usage example
const UserProfile = ({ userId }: { userId: string }) => {
  const {
    data: user,
    error,
    isLoading,
    mutate,
  } = useCache({
    key: `user-${userId}`,
    fetcher: () => httpClient.get(`/users/${userId}`),
    dependencies: [userId],
    ttl: 600000, // 10 minutes
    onError: (error) => {
      console.error('Failed to fetch user:', error);
    },
  });
  
  const updateUser = async (updates: Partial<User>) => {
    // Optimistic update
    const updatedUser = { ...user, ...updates };
    await mutate(updatedUser, false);
    
    try {
      const result = await httpClient.put(`/users/${userId}`, updates);
      await mutate(result);
    } catch (error) {
      // Revert on error
      await mutate(user);
      throw error;
    }
  };
  
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  if (!user) return <NotFoundMessage />;
  
  return (
    <div>
      <h1>{user.name}</h1>
      <button onClick={() => updateUser({ name: 'Updated Name' })}>
        Update Name
      </button>
    </div>
  );
};
```

### 4. Error Handling System Design

#### Architecture Overview
```
┌─────────────────────────────────────────────────────────────┐
│                Error Handling System Architecture          │
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │Error        │ │Global Error │ │Error        │           │
│ │Boundaries   │ │Handler      │ │Reporting    │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│ ┌─────────────────────────────────────────────────────┐   │
│ │              Error Manager                           │   │
│ └─────────────────────────────────────────────────────┘   │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │Retry Logic  │ │Fallback UI  │ │Analytics &  │           │
│ │& Recovery   │ │Components   │ │Monitoring   │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

#### Step-by-Step Implementation

##### Step 1: Error Classification System
```javascript
// errors/errorTypes.ts
enum ErrorSeverity {
  LOW = 'low',
  MEDIUM = 'medium',
  HIGH = 'high',
  CRITICAL = 'critical',
}

enum ErrorCategory {
  NETWORK = 'network',
  VALIDATION = 'validation',
  PERMISSION = 'permission',
  SYSTEM = 'system',
  USER_INPUT = 'user_input',
  BUSINESS_LOGIC = 'business_logic',
}

interface ErrorContext {
  userId?: string;
  sessionId: string;
  userAgent: string;
  url: string;
  timestamp: number;
  component?: string;
  props?: Record<string, any>;
  state?: Record<string, any>;
}

class AppError extends Error {
  public readonly code: string;
  public readonly severity: ErrorSeverity;
  public readonly category: ErrorCategory;
  public readonly context: ErrorContext;
  public readonly retry: boolean;
  public readonly userMessage?: string;
  
  constructor({
    message,
    code,
    severity = ErrorSeverity.MEDIUM,
    category,
    context,
    retry = false,
    userMessage,
    cause,
  }: {
    message: string;
    code: string;
    severity?: ErrorSeverity;
    category: ErrorCategory;
    context: Partial<ErrorContext>;
    retry?: boolean;
    userMessage?: string;
    cause?: Error;
  }) {
    super(message);
    this.name = 'AppError';
    this.code = code;
    this.severity = severity;
    this.category = category;
    this.retry = retry;
    this.userMessage = userMessage;
    
    this.context = {
      sessionId: generateSessionId(),
      userAgent: navigator.userAgent,
      url: window.location.href,
      timestamp: Date.now(),
      ...context,
    };
    
    if (cause) {
      this.stack = `${this.stack}\nCaused by: ${cause.stack}`;
    }
    
    // Maintain proper stack trace
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, AppError);
    }
  }
  
  toJSON() {
    return {
      name: this.name,
      message: this.message,
      code: this.code,
      severity: this.severity,
      category: this.category,
      context: this.context,
      retry: this.retry,
      userMessage: this.userMessage,
      stack: this.stack,
    };
  }
}

// Specific error types
class NetworkError extends AppError {
  constructor(message: string, context: Partial<ErrorContext> = {}) {
    super({
      message,
      code: 'NETWORK_ERROR',
      category: ErrorCategory.NETWORK,
      context,
      retry: true,
      userMessage: 'Connection problem. Please check your internet and try again.',
    });
  }
}

class ValidationError extends AppError {
  public readonly field?: string;
  
  constructor(
    message: string, 
    field?: string, 
    context: Partial<ErrorContext> = {}
  ) {
    super({
      message,
      code: 'VALIDATION_ERROR',
      category: ErrorCategory.VALIDATION,
      context,
      userMessage: message,
    });
    this.field = field;
  }
}

class PermissionError extends AppError {
  constructor(
    requiredPermission: string,
    context: Partial<ErrorContext> = {}
  ) {
    super({
      message: `Permission required: ${requiredPermission}`,
      code: 'PERMISSION_DENIED',
      category: ErrorCategory.PERMISSION,
      severity: ErrorSeverity.HIGH,
      context,
      userMessage: 'You do not have permission to perform this action.',
    });
  }
}
```

##### Step 2: Error Manager
```javascript
// errors/errorManager.ts
interface ErrorHandler {
  canHandle(error: Error): boolean;
  handle(error: Error, context?: any): Promise<void> | void;
}

interface RetryConfig {
  maxAttempts: number;
  delayMs: number;
  backoffMultiplier: number;
  shouldRetry: (error: Error, attempt: number) => boolean;
}

class ErrorManager {
  private static instance: ErrorManager;
  private handlers: ErrorHandler[] = [];
  private retryConfigs: Map<string, RetryConfig> = new Map();
  private metrics = {
    totalErrors: 0,
    errorsByCategory: new Map<ErrorCategory, number>(),
    errorsBySeverity: new Map<ErrorSeverity, number>(),
    retriesAttempted: 0,
    retriesSuccessful: 0,
  };
  
  static getInstance(): ErrorManager {
    if (!ErrorManager.instance) {
      ErrorManager.instance = new ErrorManager();
    }
    return ErrorManager.instance;
  }
  
  private constructor() {
    this.setupDefaultHandlers();
    this.setupGlobalHandlers();
  }
  
  private setupDefaultHandlers(): void {
    // Network error handler
    this.addHandler({
      canHandle: (error) => error instanceof NetworkError,
      handle: async (error) => {
        console.error('Network error:', error);
        this.showNotification({
          type: 'error',
          message: error.message,
          duration: 5000,
        });
      },
    });
    
    // Validation error handler
    this.addHandler({
      canHandle: (error) => error instanceof ValidationError,
      handle: (error) => {
        console.warn('Validation error:', error);
        // Handle form validation display
      },
    });
    
    // Permission error handler
    this.addHandler({
      canHandle: (error) => error instanceof PermissionError,
      handle: (error) => {
        console.error('Permission error:', error);
        this.showNotification({
          type: 'error',
          message: error.message,
          duration: 8000,
        });
      },
    });
  }
  
  private setupGlobalHandlers(): void {
    // Unhandled promise rejections
    window.addEventListener('unhandledrejection', (event) => {
      this.handleError(
        event.reason instanceof Error ? event.reason : new Error(event.reason)
      );
      event.preventDefault();
    });
    
    // Global JavaScript errors
    window.addEventListener('error', (event) => {
      this.handleError(event.error || new Error(event.message));
    });
  }
  
  addHandler(handler: ErrorHandler): void {
    this.handlers.push(handler);
  }
  
  removeHandler(handler: ErrorHandler): void {
    const index = this.handlers.indexOf(handler);
    if (index > -1) {
      this.handlers.splice(index, 1);
    }
  }
  
  async handleError(error: Error, context?: any): Promise<void> {
    this.updateMetrics(error);
    
    // Find appropriate handler
    const handler = this.handlers.find(h => h.canHandle(error));
    
    if (handler) {
      try {
        await handler.handle(error, context);
      } catch (handlerError) {
        console.error('Error handler failed:', handlerError);
      }
    } else {
      // Default handling
      console.error('Unhandled error:', error);
      this.reportError(error, context);
    }
  }
  
  async withRetry<T>(
    operation: () => Promise<T>,
    config: Partial<RetryConfig> = {}
  ): Promise<T> {
    const fullConfig: RetryConfig = {
      maxAttempts: 3,
      delayMs: 1000,
      backoffMultiplier: 2,
      shouldRetry: (error) => error instanceof NetworkError,
      ...config,
    };
    
    let lastError: Error;
    
    for (let attempt = 1; attempt <= fullConfig.maxAttempts; attempt++) {
      try {
        return await operation();
      } catch (error) {
        lastError = error instanceof Error ? error : new Error(String(error));
        
        this.metrics.retriesAttempted++;
        
        if (
          attempt === fullConfig.maxAttempts ||
          !fullConfig.shouldRetry(lastError, attempt)
        ) {
          break;
        }
        
        // Wait before retry with exponential backoff
        const delay = fullConfig.delayMs * Math.pow(fullConfig.backoffMultiplier, attempt - 1);
        await new Promise(resolve => setTimeout(resolve, delay));
      }
    }
    
    throw lastError!;
  }
  
  private updateMetrics(error: Error): void {
    this.metrics.totalErrors++;
    
    if (error instanceof AppError) {
      const categoryCount = this.metrics.errorsByCategory.get(error.category) || 0;
      this.metrics.errorsByCategory.set(error.category, categoryCount + 1);
      
      const severityCount = this.metrics.errorsBySeverity.get(error.severity) || 0;
      this.metrics.errorsBySeverity.set(error.severity, severityCount + 1);
    }
  }
  
  private async reportError(error: Error, context?: any): Promise<void> {
    try {
      const errorReport = {
        error: error instanceof AppError ? error.toJSON() : {
          name: error.name,
          message: error.message,
          stack: error.stack,
        },
        context,
        url: window.location.href,
        userAgent: navigator.userAgent,
        timestamp: Date.now(),
      };
      
      // Send to monitoring service
      await fetch('/api/errors', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(errorReport),
      });
    } catch (reportingError) {
      console.error('Failed to report error:', reportingError);
    }
  }
  
  private showNotification(notification: {
    type: 'error' | 'warning' | 'info';
    message: string;
    duration?: number;
  }): void {
    // Implementation depends on your notification system
    // Could use toast library, custom notification component, etc.
    console.log(`[${notification.type.toUpperCase()}] ${notification.message}`);
  }
  
  getMetrics() {
    return {
      ...this.metrics,
      errorsByCategory: Object.fromEntries(this.metrics.errorsByCategory),
      errorsBySeverity: Object.fromEntries(this.metrics.errorsBySeverity),
    };
  }
}
```

##### Step 3: Error Boundaries
```javascript
// components/ErrorBoundary.tsx
interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
  errorInfo: ErrorInfo | null;
  errorId: string | null;
}

interface ErrorBoundaryProps {
  children: React.ReactNode;
  fallback?: React.ComponentType<ErrorFallbackProps>;
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
  level?: 'page' | 'section' | 'component';
  resetKeys?: Array<string | number>;
  resetOnPropsChange?: boolean;
}

interface ErrorFallbackProps {
  error: Error;
  errorInfo: ErrorInfo;
  resetError: () => void;
  level: string;
}

class ErrorBoundary extends React.Component<ErrorBoundaryProps, ErrorBoundaryState> {
  private resetTimeoutId: number | null = null;
  private errorManager = ErrorManager.getInstance();
  
  constructor(props: ErrorBoundaryProps) {
    super(props);
    this.state = {
      hasError: false,
      error: null,
      errorInfo: null,
      errorId: null,
    };
  }
  
  static getDerivedStateFromError(error: Error): Partial<ErrorBoundaryState> {
    return {
      hasError: true,
      error,
      errorId: generateErrorId(),
    };
  }
  
  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    this.setState({ errorInfo });
    
    const appError = new AppError({
      message: error.message,
      code: 'COMPONENT_ERROR',
      category: ErrorCategory.SYSTEM,
      severity: ErrorSeverity.HIGH,
      context: {
        component: this.props.level || 'unknown',
        props: this.props,
        errorInfo: errorInfo.componentStack,
      },
    });
    
    this.errorManager.handleError(appError);
    this.props.onError?.(error, errorInfo);
  }
  
  componentDidUpdate(prevProps: ErrorBoundaryProps) {
    const { resetKeys, resetOnPropsChange } = this.props;
    const { hasError } = this.state;
    
    if (hasError && resetOnPropsChange && prevProps !== this.props) {
      this.resetError();
    }
    
    if (hasError && resetKeys) {
      const prevResetKeys = prevProps.resetKeys || [];
      const hasResetKeyChanged = resetKeys.some(
        (key, index) => key !== prevResetKeys[index]
      );
      
      if (hasResetKeyChanged) {
        this.resetError();
      }
    }
  }
  
  resetError = () => {
    this.setState({
      hasError: false,
      error: null,
      errorInfo: null,
      errorId: null,
    });
  };
  
  render() {
    const { hasError, error, errorInfo } = this.state;
    const { children, fallback: Fallback, level = 'component' } = this.props;
    
    if (hasError && error) {
      if (Fallback) {
        return (
          <Fallback
            error={error}
            errorInfo={errorInfo!}
            resetError={this.resetError}
            level={level}
          />
        );
      }
      
      return <DefaultErrorFallback {...{ error, errorInfo, resetError: this.resetError, level }} />;
    }
    
    return children;
  }
}

// Default error fallback component
const DefaultErrorFallback: React.FC<ErrorFallbackProps> = ({
  error,
  resetError,
  level,
}) => {
  const [showDetails, setShowDetails] = useState(false);
  
  return (
    <div className={`error-fallback error-fallback--${level}`}>
      <div className="error-fallback__content">
        <h2>Something went wrong</h2>
        <p>
          {error instanceof AppError && error.userMessage
            ? error.userMessage
            : 'An unexpected error occurred. Please try again.'}
        </p>
        
        <div className="error-fallback__actions">
          <button onClick={resetError} className="btn btn--primary">
            Try Again
          </button>
          
          <button
            onClick={() => setShowDetails(!showDetails)}
            className="btn btn--secondary"
          >
            {showDetails ? 'Hide' : 'Show'} Details
          </button>
        </div>
        
        {showDetails && (
          <details className="error-fallback__details">
            <summary>Error Details</summary>
            <pre>{error.stack}</pre>
          </details>
        )}
      </div>
    </div>
  );
};

// Usage wrapper
export const withErrorBoundary = <P extends object>(
  Component: React.ComponentType<P>,
  errorBoundaryProps?: Omit<ErrorBoundaryProps, 'children'>
) => {
  return function WrappedComponent(props: P) {
    return (
      <ErrorBoundary {...errorBoundaryProps}>
        <Component {...props} />
      </ErrorBoundary>
    );
  };
};
```

##### Step 4: Hook for Error Handling
```javascript
// hooks/useErrorHandler.ts
interface UseErrorHandlerOptions {
  onError?: (error: Error) => void;
  throwOnError?: boolean;
  retryConfig?: Partial<RetryConfig>;
}

export function useErrorHandler(options: UseErrorHandlerOptions = {}) {
  const [error, setError] = useState<Error | null>(null);
  const [isRetrying, setIsRetrying] = useState(false);
  const errorManager = useMemo(() => ErrorManager.getInstance(), []);
  
  const handleError = useCallback(async (error: Error) => {
    setError(error);
    
    if (options.onError) {
      options.onError(error);
    } else {
      await errorManager.handleError(error);
    }
    
    if (options.throwOnError) {
      throw error;
    }
  }, [errorManager, options]);
  
  const clearError = useCallback(() => {
    setError(null);
  }, []);
  
  const withRetry = useCallback(async <T>(
    operation: () => Promise<T>
  ): Promise<T> => {
    setIsRetrying(true);
    
    try {
      const result = await errorManager.withRetry(operation, options.retryConfig);
      clearError();
      return result;
    } catch (error) {
      await handleError(error instanceof Error ? error : new Error(String(error)));
      throw error;
    } finally {
      setIsRetrying(false);
    }
  }, [errorManager, handleError, clearError, options.retryConfig]);
  
  const tryCatch = useCallback(async <T>(
    operation: () => Promise<T>,
    fallback?: () => T
  ): Promise<T | undefined> => {
    try {
      const result = await operation();
      clearError();
      return result;
    } catch (error) {
      await handleError(error instanceof Error ? error : new Error(String(error)));
      return fallback?.();
    }
  }, [handleError, clearError]);
  
  return {
    error,
    isRetrying,
    handleError,
    clearError,
    withRetry,
    tryCatch,
  };
}

// Usage example
const DataComponent = () => {
  const { error, isRetrying, withRetry, clearError } = useErrorHandler({
    onError: (error) => {
      console.error('Component error:', error);
    },
  });
  
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const fetchData = useCallback(async () => {
    setLoading(true);
    
    try {
      const result = await withRetry(async () => {
        const response = await httpClient.get('/api/data');
        if (!response.ok) {
          throw new NetworkError('Failed to fetch data');
        }
        return response.json();
      });
      
      setData(result);
    } finally {
      setLoading(false);
    }
  }, [withRetry]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  if (loading || isRetrying) {
    return <LoadingSpinner message={isRetrying ? "Retrying..." : "Loading..."} />;
  }
  
  if (error) {
    return (
      <ErrorMessage
        error={error}
        onRetry={fetchData}
        onDismiss={clearError}
      />
    );
  }
  
  return <DataDisplay data={data} />;
};
```

## Tools and Technologies

### 5. Performance Monitoring System

#### Architecture Overview
```
┌─────────────────────────────────────────────────────────────┐
│              Performance Monitoring Architecture           │
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │Metrics      │ │Real-time    │ │Performance  │           │
│ │Collection   │ │Monitoring   │ │Analytics    │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│ ┌─────────────────────────────────────────────────────┐   │
│ │           Performance Manager                        │   │
│ └─────────────────────────────────────────────────────┘   │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │Web Vitals   │ │Custom       │ │Reporting &  │           │
│ │Tracking     │ │Metrics      │ │Alerting     │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

#### Step-by-Step Implementation

##### Step 1: Core Metrics Collection
```javascript
// performance/metricsCollector.ts
interface PerformanceMetric {
  name: string;
  value: number;
  timestamp: number;
  unit: string;
  tags?: Record<string, string>;
}

interface WebVitalsMetrics {
  CLS: number | null;
  FID: number | null;
  FCP: number | null;
  LCP: number | null;
  TTFB: number | null;
  INP: number | null;
}

class MetricsCollector {
  private metrics: PerformanceMetric[] = [];
  private observers: PerformanceObserver[] = [];
  private webVitals: Partial<WebVitalsMetrics> = {};
  
  constructor() {
    this.initializeWebVitalsObservers();
    this.initializePerformanceObservers();
    this.startMemoryMonitoring();
  }
  
  private initializeWebVitalsObservers(): void {
    // Cumulative Layout Shift (CLS)
    this.observeLayoutShifts();
    
    // First Input Delay (FID) / Interaction to Next Paint (INP)
    this.observeInputDelays();
    
    // First Contentful Paint (FCP) & Largest Contentful Paint (LCP)
    this.observePaintMetrics();
    
    // Time to First Byte (TTFB)
    this.observeNavigationTiming();
  }
  
  private observeLayoutShifts(): void {
    if ('LayoutShiftAttribution' in window) {
      const observer = new PerformanceObserver((list) => {
        let cumulativeScore = 0;
        
        for (const entry of list.getEntries()) {
          if (!(entry as any).hadRecentInput) {
            cumulativeScore += (entry as any).value;
          }
        }
        
        this.webVitals.CLS = cumulativeScore;
        this.recordMetric('web_vitals_cls', cumulativeScore, 'score');
      });
      
      observer.observe({ type: 'layout-shift', buffered: true });
      this.observers.push(observer);
    }
  }
  
  private observeInputDelays(): void {
    if ('PerformanceEventTiming' in window) {
      const observer = new PerformanceObserver((list) => {
        for (const entry of list.getEntries()) {
          const eventEntry = entry as PerformanceEventTiming;
          
          // First Input Delay
          if (eventEntry.processingStart && eventEntry.startTime) {
            const fid = eventEntry.processingStart - eventEntry.startTime;
            this.webVitals.FID = fid;
            this.recordMetric('web_vitals_fid', fid, 'ms');
          }
          
          // Interaction to Next Paint
          if (eventEntry.processingEnd && eventEntry.startTime) {
            const inp = eventEntry.processingEnd - eventEntry.startTime;
            this.webVitals.INP = Math.max(this.webVitals.INP || 0, inp);
            this.recordMetric('web_vitals_inp', inp, 'ms');
          }
        }
      });
      
      observer.observe({ type: 'event', buffered: true });
      this.observers.push(observer);
    }
  }
  
  private observePaintMetrics(): void {
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        switch (entry.name) {
          case 'first-contentful-paint':
            this.webVitals.FCP = entry.startTime;
            this.recordMetric('web_vitals_fcp', entry.startTime, 'ms');
            break;
          case 'largest-contentful-paint':
            this.webVitals.LCP = entry.startTime;
            this.recordMetric('web_vitals_lcp', entry.startTime, 'ms');
            break;
        }
      }
    });
    
    observer.observe({ type: 'paint', buffered: true });
    observer.observe({ type: 'largest-contentful-paint', buffered: true });
    this.observers.push(observer);
  }
  
  private observeNavigationTiming(): void {
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        const navEntry = entry as PerformanceNavigationTiming;
        const ttfb = navEntry.responseStart - navEntry.requestStart;
        
        this.webVitals.TTFB = ttfb;
        this.recordMetric('web_vitals_ttfb', ttfb, 'ms');
        
        // Additional navigation metrics
        this.recordMetric('navigation_dom_load', navEntry.domContentLoadedEventEnd - navEntry.navigationStart, 'ms');
        this.recordMetric('navigation_load_complete', navEntry.loadEventEnd - navEntry.navigationStart, 'ms');
      }
    });
    
    observer.observe({ type: 'navigation', buffered: true });
    this.observers.push(observer);
  }
  
  private startMemoryMonitoring(): void {
    if ('memory' in performance) {
      setInterval(() => {
        const memory = (performance as any).memory;
        
        this.recordMetric('memory_used', memory.usedJSHeapSize, 'bytes');
        this.recordMetric('memory_total', memory.totalJSHeapSize, 'bytes');
        this.recordMetric('memory_limit', memory.jsHeapSizeLimit, 'bytes');
      }, 30000); // Every 30 seconds
    }
  }
  
  recordMetric(name: string, value: number, unit: string, tags?: Record<string, string>): void {
    const metric: PerformanceMetric = {
      name,
      value,
      timestamp: Date.now(),
      unit,
      tags,
    };
    
    this.metrics.push(metric);
    
    // Emit to listeners
    this.emit('metric', metric);
    
    // Keep only recent metrics in memory
    const oneHourAgo = Date.now() - 3600000;
    this.metrics = this.metrics.filter(m => m.timestamp > oneHourAgo);
  }
  
  recordCustomMetric(name: string, value: number, tags?: Record<string, string>): void {
    this.recordMetric(`custom_${name}`, value, 'count', tags);
  }
  
  recordTiming(name: string, startTime: number, endTime?: number): void {
    const duration = (endTime || performance.now()) - startTime;
    this.recordMetric(`timing_${name}`, duration, 'ms');
  }
  
  getWebVitals(): WebVitalsMetrics {
    return { ...this.webVitals } as WebVitalsMetrics;
  }
  
  getMetrics(filter?: { name?: string; since?: number }): PerformanceMetric[] {
    let filtered = this.metrics;
    
    if (filter?.name) {
      filtered = filtered.filter(m => m.name.includes(filter.name!));
    }
    
    if (filter?.since) {
      filtered = filtered.filter(m => m.timestamp >= filter.since!);
    }
    
    return filtered;
  }
  
  private listeners = new Map<string, Function[]>();
  
  private emit(event: string, data: any): void {
    const eventListeners = this.listeners.get(event) || [];
    eventListeners.forEach(listener => listener(data));
  }
  
  on(event: string, listener: Function): void {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event)!.push(listener);
  }
  
  destroy(): void {
    this.observers.forEach(observer => observer.disconnect());
    this.observers = [];
    this.listeners.clear();
  }
}
```

##### Step 2: Performance Manager
```javascript
// performance/performanceManager.ts
interface PerformanceConfig {
  enableWebVitals: boolean;
  enableResourceTiming: boolean;
  enableUserTiming: boolean;
  reportingInterval: number;
  samplingRate: number;
  endpoint?: string;
}

interface PerformanceReport {
  timestamp: number;
  url: string;
  userAgent: string;
  connection?: string;
  webVitals: WebVitalsMetrics;
  customMetrics: PerformanceMetric[];
  resourceTiming: PerformanceEntry[];
  errors: any[];
}

class PerformanceManager {
  private static instance: PerformanceManager;
  private metricsCollector: MetricsCollector;
  private config: PerformanceConfig;
  private reportQueue: PerformanceReport[] = [];
  private reportingTimer: number | null = null;
  
  static getInstance(config?: Partial<PerformanceConfig>): PerformanceManager {
    if (!PerformanceManager.instance) {
      PerformanceManager.instance = new PerformanceManager(config);
    }
    return PerformanceManager.instance;
  }
  
  private constructor(config: Partial<PerformanceConfig> = {}) {
    this.config = {
      enableWebVitals: true,
      enableResourceTiming: true,
      enableUserTiming: true,
      reportingInterval: 30000, // 30 seconds
      samplingRate: 1.0, // 100% sampling
      ...config,
    };
    
    this.metricsCollector = new MetricsCollector();
    this.setupReporting();
    this.setupUnloadHandler();
  }
  
  private setupReporting(): void {
    if (this.config.endpoint) {
      this.reportingTimer = window.setInterval(() => {
        this.sendReport();
      }, this.config.reportingInterval);
    }
  }
  
  private setupUnloadHandler(): void {
    // Send final report on page unload
    window.addEventListener('beforeunload', () => {
      this.sendReport(true);
    });
    
    // Use Page Visibility API for better reliability
    document.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'hidden') {
        this.sendReport(true);
      }
    });
  }
  
  private shouldSample(): boolean {
    return Math.random() < this.config.samplingRate;
  }
  
  recordPageLoad(): void {
    if (!this.shouldSample()) return;
    
    const startTime = performance.now();
    
    // Wait for page to be fully loaded
    if (document.readyState === 'complete') {
      this.collectPageLoadMetrics();
    } else {
      window.addEventListener('load', () => {
        this.collectPageLoadMetrics();
      });
    }
  }
  
  private collectPageLoadMetrics(): void {
    const navigation = performance.getEntriesByType('navigation')[0] as PerformanceNavigationTiming;
    
    if (navigation) {
      this.metricsCollector.recordMetric('page_load_time', navigation.loadEventEnd - navigation.navigationStart, 'ms');
      this.metricsCollector.recordMetric('dom_interactive', navigation.domInteractive - navigation.navigationStart, 'ms');
      this.metricsCollector.recordMetric('dom_complete', navigation.domComplete - navigation.navigationStart, 'ms');
    }
  }
  
  recordUserInteraction(interaction: string, duration?: number): void {
    if (!this.shouldSample()) return;
    
    this.metricsCollector.recordCustomMetric(`interaction_${interaction}`, duration || 1, {
      type: 'user_interaction',
      interaction,
    });
  }
  
  recordAPICall(endpoint: string, duration: number, status: number): void {
    if (!this.shouldSample()) return;
    
    this.metricsCollector.recordMetric('api_call_duration', duration, 'ms', {
      endpoint,
      status: status.toString(),
    });
    
    // Track API errors
    if (status >= 400) {
      this.metricsCollector.recordCustomMetric('api_error', 1, {
        endpoint,
        status: status.toString(),
      });
    }
  }
  
  recordComponentRender(componentName: string, duration: number): void {
    if (!this.shouldSample()) return;
    
    this.metricsCollector.recordMetric('component_render', duration, 'ms', {
      component: componentName,
    });
  }
  
  startTiming(name: string): number {
    const startTime = performance.now();
    performance.mark(`${name}_start`);
    return startTime;
  }
  
  endTiming(name: string, startTime: number): number {
    const endTime = performance.now();
    performance.mark(`${name}_end`);
    performance.measure(name, `${name}_start`, `${name}_end`);
    
    const duration = endTime - startTime;
    this.metricsCollector.recordTiming(name, startTime, endTime);
    
    return duration;
  }
  
  private generateReport(): PerformanceReport {
    const now = Date.now();
    
    return {
      timestamp: now,
      url: window.location.href,
      userAgent: navigator.userAgent,
      connection: this.getConnectionInfo(),
      webVitals: this.metricsCollector.getWebVitals(),
      customMetrics: this.metricsCollector.getMetrics({ since: now - this.config.reportingInterval }),
      resourceTiming: this.config.enableResourceTiming ? performance.getEntriesByType('resource') : [],
      errors: [], // Would be populated by error tracking
    };
  }
  
  private getConnectionInfo(): string {
    const connection = (navigator as any).connection || (navigator as any).mozConnection || (navigator as any).webkitConnection;
    
    if (connection) {
      return `${connection.effectiveType || 'unknown'}_${connection.downlink || 'unknown'}mbps`;
    }
    
    return 'unknown';
  }
  
  private async sendReport(immediate = false): Promise<void> {
    const report = this.generateReport();
    
    if (!this.config.endpoint) {
      console.log('Performance Report:', report);
      return;
    }
    
    if (immediate) {
      // Use sendBeacon for immediate sending during unload
      if (navigator.sendBeacon) {
        navigator.sendBeacon(
          this.config.endpoint,
          JSON.stringify(report)
        );
      }
    } else {
      try {
        await fetch(this.config.endpoint, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(report),
        });
      } catch (error) {
        console.error('Failed to send performance report:', error);
        // Queue for retry
        this.reportQueue.push(report);
      }
    }
  }
  
  getPerformanceScore(): number {
    const vitals = this.metricsCollector.getWebVitals();
    let score = 100;
    
    // Deduct points based on Web Vitals thresholds
    if (vitals.LCP && vitals.LCP > 2500) score -= 20;
    if (vitals.FID && vitals.FID > 100) score -= 20;
    if (vitals.CLS && vitals.CLS > 0.1) score -= 20;
    if (vitals.FCP && vitals.FCP > 1800) score -= 15;
    if (vitals.TTFB && vitals.TTFB > 800) score -= 15;
    
    return Math.max(0, score);
  }
  
  getInsights(): {
    score: number;
    recommendations: string[];
    webVitals: WebVitalsMetrics;
  } {
    const vitals = this.metricsCollector.getWebVitals();
    const recommendations: string[] = [];
    
    if (vitals.LCP && vitals.LCP > 2500) {
      recommendations.push('Optimize Largest Contentful Paint: Consider image optimization, preloading critical resources, or reducing server response times');
    }
    
    if (vitals.FID && vitals.FID > 100) {
      recommendations.push('Improve First Input Delay: Reduce JavaScript execution time, split long tasks, or use web workers');
    }
    
    if (vitals.CLS && vitals.CLS > 0.1) {
      recommendations.push('Fix Cumulative Layout Shift: Set dimensions for images and embeds, avoid inserting content above existing content');
    }
    
    if (vitals.FCP && vitals.FCP > 1800) {
      recommendations.push('Optimize First Contentful Paint: Eliminate render-blocking resources, minimize main thread work');
    }
    
    if (vitals.TTFB && vitals.TTFB > 800) {
      recommendations.push('Improve Time to First Byte: Optimize server performance, use CDN, minimize redirects');
    }
    
    return {
      score: this.getPerformanceScore(),
      recommendations,
      webVitals: vitals,
    };
  }
  
  destroy(): void {
    if (this.reportingTimer) {
      clearInterval(this.reportingTimer);
    }
    
    this.metricsCollector.destroy();
    this.sendReport(true); // Final report
  }
}
```

##### Step 3: React Hook Integration
```javascript
// hooks/usePerformance.ts
interface UsePerformanceOptions {
  trackRenders?: boolean;
  trackInteractions?: boolean;
  componentName?: string;
}

export function usePerformance(options: UsePerformanceOptions = {}) {
  const performanceManager = useMemo(() => PerformanceManager.getInstance(), []);
  const renderStartTime = useRef<number>();
  
  // Track component renders
  useEffect(() => {
    if (options.trackRenders && options.componentName) {
      renderStartTime.current = performance.now();
    }
  });
  
  useLayoutEffect(() => {
    if (options.trackRenders && options.componentName && renderStartTime.current) {
      const renderDuration = performance.now() - renderStartTime.current;
      performanceManager.recordComponentRender(options.componentName, renderDuration);
    }
  });
  
  const trackInteraction = useCallback((interaction: string, duration?: number) => {
    if (options.trackInteractions) {
      performanceManager.recordUserInteraction(interaction, duration);
    }
  }, [performanceManager, options.trackInteractions]);
  
  const trackAPICall = useCallback((endpoint: string, duration: number, status: number) => {
    performanceManager.recordAPICall(endpoint, duration, status);
  }, [performanceManager]);
  
  const startTiming = useCallback((name: string) => {
    return performanceManager.startTiming(name);
  }, [performanceManager]);
  
  const endTiming = useCallback((name: string, startTime: number) => {
    return performanceManager.endTiming(name, startTime);
  }, [performanceManager]);
  
  const withTiming = useCallback(async <T>(
    name: string,
    operation: () => Promise<T>
  ): Promise<T> => {
    const start = startTiming(name);
    try {
      return await operation();
    } finally {
      endTiming(name, start);
    }
  }, [startTiming, endTiming]);
  
  const getInsights = useCallback(() => {
    return performanceManager.getInsights();
  }, [performanceManager]);
  
  return {
    trackInteraction,
    trackAPICall,
    startTiming,
    endTiming,
    withTiming,
    getInsights,
  };
}

// Usage example
const ExpensiveComponent = ({ data }: { data: any[] }) => {
  const { trackInteraction, withTiming } = usePerformance({
    trackRenders: true,
    trackInteractions: true,
    componentName: 'ExpensiveComponent',
  });
  
  const handleSort = useCallback(async () => {
    trackInteraction('sort_data');
    
    await withTiming('data_sort', async () => {
      // Expensive sorting operation
      await new Promise(resolve => setTimeout(resolve, 100));
    });
  }, [trackInteraction, withTiming]);
  
  return (
    <div>
      <button onClick={handleSort}>Sort Data</button>
      <DataList data={data} />
    </div>
  );
};
```

### 6. Real-time WebSocket Management System

#### Architecture Overview
```
┌─────────────────────────────────────────────────────────────┐
│              WebSocket Management Architecture             │
├─────────────────────────────────────────────────────────────┤
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │Connection   │ │Message      │ │Reconnection │           │
│ │Manager      │ │Handler      │ │Strategy     │           │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│ ┌─────────────────────────────────────────────────────┐   │
│ │              WebSocket Manager                       │   │
│ └─────────────────────────────────────────────────────┘   │
│         │               │               │                 │
│         ▼               ▼               ▼                 │
│ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│ │Event        │ │State        │ │Health       │           │
│ │Management   │ │Synchronization│ │Monitoring   │         │
│ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────┘
```

#### Step-by-Step Implementation

##### Step 1: WebSocket Connection Manager
```javascript
// websocket/connectionManager.ts
interface ConnectionConfig {
  url: string;
  protocols?: string[];
  reconnectAttempts: number;
  reconnectInterval: number;
  maxReconnectInterval: number;
  heartbeatInterval: number;
  messageTimeout: number;
}

interface Message {
  id: string;
  type: string;
  payload: any;
  timestamp: number;
  requiresAck?: boolean;
}

enum ConnectionState {
  DISCONNECTED = 'disconnected',
  CONNECTING = 'connecting',
  CONNECTED = 'connected',
  RECONNECTING = 'reconnecting',
  FAILED = 'failed',
}

class WebSocketConnectionManager {
  private ws: WebSocket | null = null;
  private config: ConnectionConfig;
  private state: ConnectionState = ConnectionState.DISCONNECTED;
  private reconnectAttempts = 0;
  private reconnectTimer: number | null = null;
  private heartbeatTimer: number | null = null;
  private lastHeartbeat = 0;
  private messageQueue: Message[] = [];
  private pendingAcks = new Map<string, { resolve: Function; reject: Function; timer: number }>();
  private eventListeners = new Map<string, Set<Function>>();
  
  constructor(config: ConnectionConfig) {
    this.config = config;
    this.setupVisibilityHandling();
  }
  
  private setupVisibilityHandling(): void {
    document.addEventListener('visibilitychange', () => {
      if (document.visibilityState === 'visible' && this.state === ConnectionState.DISCONNECTED) {
        this.connect();
      } else if (document.visibilityState === 'hidden') {
        this.disconnect();
      }
    });
  }
  
  async connect(): Promise<void> {
    if (this.state === ConnectionState.CONNECTING || this.state === ConnectionState.CONNECTED) {
      return;
    }
    
    this.setState(ConnectionState.CONNECTING);
    
    try {
      this.ws = new WebSocket(this.config.url, this.config.protocols);
      this.setupEventHandlers();
      
      await this.waitForConnection();
      this.setState(ConnectionState.CONNECTED);
      this.reconnectAttempts = 0;
      this.startHeartbeat();
      this.processMessageQueue();
      
    } catch (error) {
      this.setState(ConnectionState.FAILED);
      this.scheduleReconnect();
      throw error;
    }
  }
  
  private waitForConnection(): Promise<void> {
    return new Promise((resolve, reject) => {
      const timeout = setTimeout(() => {
        reject(new Error('Connection timeout'));
      }, 10000);
      
      const onOpen = () => {
        clearTimeout(timeout);
        resolve();
      };
      
      const onError = (error: Event) => {
        clearTimeout(timeout);
        reject(error);
      };
      
      this.ws!.addEventListener('open', onOpen, { once: true });
      this.ws!.addEventListener('error', onError, { once: true });
    });
  }
  
  private setupEventHandlers(): void {
    if (!this.ws) return;
    
    this.ws.onopen = () => {
      this.emit('connected');
    };
    
    this.ws.onclose = (event) => {
      this.setState(ConnectionState.DISCONNECTED);
      this.stopHeartbeat();
      this.emit('disconnected', { code: event.code, reason: event.reason });
      
      if (!event.wasClean) {
        this.scheduleReconnect();
      }
    };
    
    this.ws.onerror = (error) => {
      this.emit('error', error);
    };
    
    this.ws.onmessage = (event) => {
      this.handleMessage(event);
    };
  }
  
  private handleMessage(event: MessageEvent): void {
    try {
      const message = JSON.parse(event.data);
      
      // Handle heartbeat pong
      if (message.type === 'pong') {
        this.lastHeartbeat = Date.now();
        return;
      }
      
      // Handle acknowledgments
      if (message.type === 'ack' && message.messageId) {
        const pending = this.pendingAcks.get(message.messageId);
        if (pending) {
          clearTimeout(pending.timer);
          this.pendingAcks.delete(message.messageId);
          pending.resolve(message);
        }
        return;
      }
      
      // Send acknowledgment if required
      if (message.requiresAck) {
        this.sendAck(message.id);
      }
      
      // Emit message event
      this.emit('message', message);
      this.emit(`message:${message.type}`, message);
      
    } catch (error) {
      this.emit('error', new Error('Failed to parse message'));
    }
  }
  
  private sendAck(messageId: string): void {
    this.send({
      type: 'ack',
      messageId,
      timestamp: Date.now(),
    }, false);
  }
  
  send(data: any, requiresAck = false): Promise<any> {
    const message: Message = {
      id: this.generateMessageId(),
      type: data.type || 'message',
      payload: data,
      timestamp: Date.now(),
      requiresAck,
    };
    
    if (this.state !== ConnectionState.CONNECTED) {
      this.messageQueue.push(message);
      return Promise.reject(new Error('Not connected'));
    }
    
    try {
      this.ws!.send(JSON.stringify(message));
      
      if (requiresAck) {
        return this.waitForAck(message.id);
      }
      
      return Promise.resolve();
    } catch (error) {
      return Promise.reject(error);
    }
  }
  
  private waitForAck(messageId: string): Promise<any> {
    return new Promise((resolve, reject) => {
      const timer = window.setTimeout(() => {
        this.pendingAcks.delete(messageId);
        reject(new Error('Message acknowledgment timeout'));
      }, this.config.messageTimeout);
      
      this.pendingAcks.set(messageId, { resolve, reject, timer });
    });
  }
  
  private processMessageQueue(): void {
    const queue = [...this.messageQueue];
    this.messageQueue = [];
    
    queue.forEach(message => {
      this.send(message.payload, message.requiresAck);
    });
  }
  
  private startHeartbeat(): void {
    this.stopHeartbeat();
    
    this.heartbeatTimer = window.setInterval(() => {
      if (this.state === ConnectionState.CONNECTED) {
        // Check if we've received a recent heartbeat response
        const timeSinceLastHeartbeat = Date.now() - this.lastHeartbeat;
        
        if (timeSinceLastHeartbeat > this.config.heartbeatInterval * 2) {
          // Connection might be dead
          this.disconnect();
          this.scheduleReconnect();
          return;
        }
        
        // Send heartbeat ping
        this.send({ type: 'ping', timestamp: Date.now() }, false);
      }
    }, this.config.heartbeatInterval);
  }
  
  private stopHeartbeat(): void {
    if (this.heartbeatTimer) {
      clearInterval(this.heartbeatTimer);
      this.heartbeatTimer = null;
    }
  }
  
  private scheduleReconnect(): void {
    if (this.reconnectAttempts >= this.config.reconnectAttempts) {
      this.setState(ConnectionState.FAILED);
      this.emit('failed');
      return;
    }
    
    this.setState(ConnectionState.RECONNECTING);
    this.reconnectAttempts++;
    
    const delay = Math.min(
      this.config.reconnectInterval * Math.pow(2, this.reconnectAttempts - 1),
      this.config.maxReconnectInterval
    );
    
    this.reconnectTimer = window.setTimeout(() => {
      this.connect().catch(() => {
        // Reconnection failed, will try again
      });
    }, delay);
    
    this.emit('reconnecting', { attempt: this.reconnectAttempts, delay });
  }
  
  disconnect(): void {
    this.stopHeartbeat();
    
    if (this.reconnectTimer) {
      clearTimeout(this.reconnectTimer);
      this.reconnectTimer = null;
    }
    
    if (this.ws) {
      this.ws.close(1000, 'Client disconnect');
      this.ws = null;
    }
    
    this.setState(ConnectionState.DISCONNECTED);
  }
  
  private setState(newState: ConnectionState): void {
    const oldState = this.state;
    this.state = newState;
    this.emit('stateChange', { oldState, newState });
  }
  
  getState(): ConnectionState {
    return this.state;
  }
  
  isConnected(): boolean {
    return this.state === ConnectionState.CONNECTED;
  }
  
  private generateMessageId(): string {
    return `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`;
  }
  
  on(event: string, listener: Function): void {
    if (!this.eventListeners.has(event)) {
      this.eventListeners.set(event, new Set());
    }
    this.eventListeners.get(event)!.add(listener);
  }
  
  off(event: string, listener?: Function): void {
    if (!listener) {
      this.eventListeners.delete(event);
    } else {
      this.eventListeners.get(event)?.delete(listener);
    }
  }
  
  private emit(event: string, data?: any): void {
    const listeners = this.eventListeners.get(event);
    if (listeners) {
      listeners.forEach(listener => {
        try {
          listener(data);
        } catch (error) {
          console.error('Event listener error:', error);
        }
      });
    }
  }
  
  destroy(): void {
    this.disconnect();
    this.eventListeners.clear();
    
    // Clear any pending acknowledgments
    this.pendingAcks.forEach(({ reject, timer }) => {
      clearTimeout(timer);
      reject(new Error('Connection destroyed'));
    });
    this.pendingAcks.clear();
  }
}
```

##### Step 2: React Hook Integration
```javascript
// hooks/useWebSocket.ts
interface UseWebSocketOptions {
  url: string;
  protocols?: string[];
  reconnectAttempts?: number;
  reconnectInterval?: number;
  heartbeatInterval?: number;
  onMessage?: (message: any) => void;
  onError?: (error: any) => void;
  onStateChange?: (state: ConnectionState) => void;
}

export function useWebSocket(options: UseWebSocketOptions) {
  const [connectionState, setConnectionState] = useState<ConnectionState>(ConnectionState.DISCONNECTED);
  const [lastMessage, setLastMessage] = useState<any>(null);
  const [error, setError] = useState<any>(null);
  
  const connectionManager = useMemo(() => {
    return new WebSocketConnectionManager({
      url: options.url,
      protocols: options.protocols,
      reconnectAttempts: options.reconnectAttempts || 5,
      reconnectInterval: options.reconnectInterval || 1000,
      maxReconnectInterval: 30000,
      heartbeatInterval: options.heartbeatInterval || 30000,
      messageTimeout: 5000,
    });
  }, [options.url, options.protocols, options.reconnectAttempts, options.reconnectInterval, options.heartbeatInterval]);
  
  useEffect(() => {
    const handleStateChange = ({ newState }: { newState: ConnectionState }) => {
      setConnectionState(newState);
      options.onStateChange?.(newState);
    };
    
    const handleMessage = (message: any) => {
      setLastMessage(message);
      options.onMessage?.(message);
    };
    
    const handleError = (error: any) => {
      setError(error);
      options.onError?.(error);
    };
    
    connectionManager.on('stateChange', handleStateChange);
    connectionManager.on('message', handleMessage);
    connectionManager.on('error', handleError);
    
    // Auto-connect
    connectionManager.connect().catch(handleError);
    
    return () => {
      connectionManager.off('stateChange', handleStateChange);
      connectionManager.off('message', handleMessage);
      connectionManager.off('error', handleError);
      connectionManager.destroy();
    };
  }, [connectionManager, options]);
  
  const sendMessage = useCallback(async (message: any, requiresAck = false) => {
    try {
      return await connectionManager.send(message, requiresAck);
    } catch (error) {
      setError(error);
      throw error;
    }
  }, [connectionManager]);
  
  const connect = useCallback(() => {
    return connectionManager.connect();
  }, [connectionManager]);
  
  const disconnect = useCallback(() => {
    connectionManager.disconnect();
  }, [connectionManager]);
  
  return {
    connectionState,
    isConnected: connectionState === ConnectionState.CONNECTED,
    lastMessage,
    error,
    sendMessage,
    connect,
    disconnect,
  };
}

// Usage example with real-time chat
const ChatComponent = () => {
  const [messages, setMessages] = useState<any[]>([]);
  const [inputValue, setInputValue] = useState('');
  
  const { isConnected, sendMessage, lastMessage } = useWebSocket({
    url: 'wss://api.example.com/chat',
    onMessage: (message) => {
      if (message.type === 'chat_message') {
        setMessages(prev => [...prev, message.payload]);
      }
    },
    onError: (error) => {
      console.error('WebSocket error:', error);
    },
  });
  
  const handleSendMessage = useCallback(async () => {
    if (!inputValue.trim() || !isConnected) return;
    
    try {
      await sendMessage({
        type: 'chat_message',
        content: inputValue,
        timestamp: Date.now(),
      }, true); // Require acknowledgment
      
      setInputValue('');
    } catch (error) {
      console.error('Failed to send message:', error);
    }
  }, [inputValue, isConnected, sendMessage]);
  
  return (
    <div className="chat-container">
      <div className="connection-status">
        Status: {isConnected ? 'Connected' : 'Disconnected'}
      </div>
      
      <div className="messages">
        {messages.map((message, index) => (
          <div key={index} className="message">
            {message.content}
          </div>
        ))}
      </div>
      
      <div className="message-input">
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && handleSendMessage()}
          disabled={!isConnected}
          placeholder="Type a message..."
        />
        <button onClick={handleSendMessage} disabled={!isConnected}>
          Send
        </button>
      </div>
    </div>
  );
};
```

### 7. Design Decision Framework

#### Architecture Decision Records (ADRs)

##### Template for Decision Making
```markdown
# ADR-001: [Decision Title]

## Status
[Proposed | Accepted | Deprecated | Superseded]

## Context
What is the issue that we're seeing that is motivating this decision or change?

## Decision
What is the change that we're proposing and/or doing?

## Consequences
What becomes easier or more difficult to do because of this change?

### Positive Consequences
- [Positive consequence 1]
- [Positive consequence 2]

### Negative Consequences
- [Negative consequence 1]
- [Negative consequence 2]

### Neutral Consequences
- [Neutral consequence 1]

## Alternatives Considered
- [Alternative 1]: [Brief description and why it was rejected]
- [Alternative 2]: [Brief description and why it was rejected]

## Implementation Notes
Any specific implementation details or considerations.
```

#### Decision Framework Process

##### Step 1: Problem Identification
```javascript
// decision/problemFramework.ts
interface Problem {
  id: string;
  title: string;
  description: string;
  stakeholders: string[];
  urgency: 'low' | 'medium' | 'high' | 'critical';
  impact: 'low' | 'medium' | 'high' | 'critical';
  constraints: Constraint[];
  requirements: Requirement[];
}

interface Constraint {
  type: 'technical' | 'business' | 'legal' | 'time' | 'budget';
  description: string;
  mandatory: boolean;
}

interface Requirement {
  id: string;
  description: string;
  priority: 'must-have' | 'should-have' | 'could-have' | 'wont-have';
  measurable: boolean;
  criteria: string;
}

class ProblemAnalyzer {
  analyzeComplexity(problem: Problem): {
    score: number;
    factors: string[];
    recommendation: string;
  } {
    let complexity = 0;
    const factors: string[] = [];
    
    // Stakeholder complexity
    if (problem.stakeholders.length > 5) {
      complexity += 2;
      factors.push('Multiple stakeholders require coordination');
    }
    
    // Technical complexity
    const technicalConstraints = problem.constraints.filter(c => c.type === 'technical');
    complexity += technicalConstraints.length;
    
    // Business impact
    if (problem.impact === 'high' || problem.impact === 'critical') {
      complexity += 2;
      factors.push('High business impact requires careful consideration');
    }
    
    // Requirements complexity
    const mustHaveRequirements = problem.requirements.filter(r => r.priority === 'must-have');
    if (mustHaveRequirements.length > 3) {
      complexity += 1;
      factors.push('Multiple critical requirements increase implementation complexity');
    }
    
    let recommendation = '';
    if (complexity <= 3) {
      recommendation = 'Simple decision - can be made quickly with basic analysis';
    } else if (complexity <= 6) {
      recommendation = 'Moderate complexity - requires structured analysis and stakeholder input';
    } else {
      recommendation = 'High complexity - requires extensive analysis, prototyping, and phased approach';
    }
    
    return { score: complexity, factors, recommendation };
  }
}
```

##### Step 2: Alternative Generation
```javascript
// decision/alternativeGenerator.ts
interface Alternative {
  id: string;
  name: string;
  description: string;
  pros: string[];
  cons: string[];
  effort: 'low' | 'medium' | 'high';
  risk: 'low' | 'medium' | 'high';
  cost: 'low' | 'medium' | 'high';
  timeToImplement: string;
  maintainability: 'low' | 'medium' | 'high';
  scalability: 'low' | 'medium' | 'high';
  implementation: ImplementationPlan;
}

interface ImplementationPlan {
  phases: Phase[];
  dependencies: string[];
  risks: Risk[];
  successCriteria: string[];
}

interface Phase {
  name: string;
  duration: string;
  deliverables: string[];
  resources: string[];
}

interface Risk {
  description: string;
  probability: 'low' | 'medium' | 'high';
  impact: 'low' | 'medium' | 'high';
  mitigation: string;
}

class AlternativeGenerator {
  generateAlternatives(problem: Problem): Alternative[] {
    const alternatives: Alternative[] = [];
    
    // Generate based on common patterns
    alternatives.push(this.generateBuyVsBuild(problem));
    alternatives.push(this.generateIncrementalVsBigBang(problem));
    alternatives.push(this.generateTechnologyAlternatives(problem));
    
    return alternatives;
  }
  
  private generateBuyVsBuild(problem: Problem): Alternative {
    return {
      id: 'buy-vs-build',
      name: 'Buy vs Build Analysis',
      description: 'Evaluate whether to purchase existing solution or build custom',
      pros: [
        'Faster time to market with existing solutions',
        'Lower upfront development costs',
        'Proven technology with support'
      ],
      cons: [
        'Less customization flexibility',
        'Ongoing licensing costs',
        'Vendor dependency'
      ],
      effort: 'medium',
      risk: 'medium',
      cost: 'medium',
      timeToImplement: '3-6 months',
      maintainability: 'medium',
      scalability: 'medium',
      implementation: {
        phases: [
          {
            name: 'Vendor Evaluation',
            duration: '4 weeks',
            deliverables: ['Vendor comparison matrix', 'Cost analysis'],
            resources: ['Technical lead', 'Business analyst']
          },
          {
            name: 'Proof of Concept',
            duration: '2 weeks',
            deliverables: ['Working prototype', 'Integration assessment'],
            resources: ['Developer', 'Technical lead']
          }
        ],
        dependencies: ['Budget approval', 'Vendor availability'],
        risks: [
          {
            description: 'Vendor solution may not meet all requirements',
            probability: 'medium',
            impact: 'high',
            mitigation: 'Conduct thorough evaluation and pilot testing'
          }
        ],
        successCriteria: ['All requirements met', 'Cost within budget', 'Smooth integration']
      }
    };
  }
  
  private generateIncrementalVsBigBang(problem: Problem): Alternative {
    return {
      id: 'incremental-vs-bigbang',
      name: 'Incremental vs Big Bang Implementation',
      description: 'Compare phased rollout vs complete system replacement',
      pros: [
        'Reduced risk with incremental approach',
        'Early feedback and course correction',
        'Continuous value delivery'
      ],
      cons: [
        'Longer overall timeline',
        'Complex integration management',
        'Potential inconsistencies during transition'
      ],
      effort: 'high',
      risk: 'low',
      cost: 'high',
      timeToImplement: '6-12 months',
      maintainability: 'high',
      scalability: 'high',
      implementation: {
        phases: [
          {
            name: 'Core Foundation',
            duration: '8 weeks',
            deliverables: ['Base architecture', 'Core components'],
            resources: ['Senior developer', 'Architect']
          },
          {
            name: 'Feature Increments',
            duration: '16 weeks',
            deliverables: ['Feature releases', 'User feedback'],
            resources: ['Development team', 'Product owner']
          }
        ],
        dependencies: ['Stakeholder buy-in', 'Resource allocation'],
        risks: [
          {
            description: 'Integration complexity may cause delays',
            probability: 'medium',
            impact: 'medium',
            mitigation: 'Invest in strong integration testing and monitoring'
          }
        ],
        successCriteria: ['Smooth incremental rollouts', 'User adoption', 'System stability']
      }
    };
  }
  
  private generateTechnologyAlternatives(problem: Problem): Alternative {
    return {
      id: 'technology-stack',
      name: 'Technology Stack Comparison',
      description: 'Evaluate different technology options for implementation',
      pros: [
        'Choose best-fit technology for requirements',
        'Consider team expertise and learning curve',
        'Balance performance with maintainability'
      ],
      cons: [
        'Technology choices are hard to reverse',
        'Learning curve for new technologies',
        'Ecosystem maturity varies'
      ],
      effort: 'medium',
      risk: 'medium',
      cost: 'medium',
      timeToImplement: '1-3 months',
      maintainability: 'high',
      scalability: 'high',
      implementation: {
        phases: [
          {
            name: 'Technology Assessment',
            duration: '2 weeks',
            deliverables: ['Technology comparison matrix', 'Prototype implementations'],
            resources: ['Senior developers', 'Architect']
          },
          {
            name: 'Decision and Setup',
            duration: '1 week',
            deliverables: ['Technology decision', 'Development environment'],
            resources: ['Technical lead', 'DevOps engineer']
          }
        ],
        dependencies: ['Team skill assessment', 'Infrastructure requirements'],
        risks: [
          {
            description: 'Chosen technology may not scale as expected',
            probability: 'low',
            impact: 'high',
            mitigation: 'Conduct performance testing and have migration plan'
          }
        ],
        successCriteria: ['Technology supports all requirements', 'Team can effectively use technology', 'Performance meets expectations']
      }
    };
  }
}
```

##### Step 3: Decision Matrix
```javascript
// decision/decisionMatrix.ts
interface DecisionCriteria {
  name: string;
  weight: number; // 0-1, sum should equal 1
  scoreFunction: (alternative: Alternative) => number; // 0-10
}

interface ScoredAlternative extends Alternative {
  criteriaScores: Record<string, number>;
  weightedScore: number;
  rank: number;
}

class DecisionMatrix {
  private criteria: DecisionCriteria[];
  
  constructor(criteria: DecisionCriteria[]) {
    this.criteria = criteria;
    this.validateCriteria();
  }
  
  private validateCriteria(): void {
    const totalWeight = this.criteria.reduce((sum, criterion) => sum + criterion.weight, 0);
    if (Math.abs(totalWeight - 1) > 0.001) {
      throw new Error('Criteria weights must sum to 1');
    }
  }
  
  evaluateAlternatives(alternatives: Alternative[]): ScoredAlternative[] {
    const scoredAlternatives: ScoredAlternative[] = alternatives.map(alternative => {
      const criteriaScores: Record<string, number> = {};
      let weightedScore = 0;
      
      this.criteria.forEach(criterion => {
        const score = criterion.scoreFunction(alternative);
        criteriaScores[criterion.name] = score;
        weightedScore += score * criterion.weight;
      });
      
      return {
        ...alternative,
        criteriaScores,
        weightedScore,
        rank: 0, // Will be set after sorting
      };
    });
    
    // Sort by weighted score and assign ranks
    scoredAlternatives.sort((a, b) => b.weightedScore - a.weightedScore);
    scoredAlternatives.forEach((alt, index) => {
      alt.rank = index + 1;
    });
    
    return scoredAlternatives;
  }
  
  generateReport(scoredAlternatives: ScoredAlternative[]): {
    summary: string;
    recommendations: string[];
    detailedAnalysis: any;
  } {
    const winner = scoredAlternatives[0];
    const runnerUp = scoredAlternatives[1];
    
    const summary = `Analysis of ${scoredAlternatives.length} alternatives shows "${winner.name}" as the top choice with a score of ${winner.weightedScore.toFixed(2)}.`;
    
    const recommendations = [
      `Recommended: ${winner.name} (Score: ${winner.weightedScore.toFixed(2)})`,
      `Alternative: ${runnerUp?.name} (Score: ${runnerUp?.weightedScore.toFixed(2)})`,
      'Consider conducting a pilot implementation to validate assumptions',
      'Plan for regular review points to reassess the decision'
    ];
    
    const detailedAnalysis = {
      alternatives: scoredAlternatives,
      criteria: this.criteria,
      sensitivityAnalysis: this.performSensitivityAnalysis(scoredAlternatives),
    };
    
    return { summary, recommendations, detailedAnalysis };
  }
  
  private performSensitivityAnalysis(scoredAlternatives: ScoredAlternative[]): any {
    // Analyze how changes in criteria weights affect the ranking
    const baseRanking = scoredAlternatives.map(alt => alt.rank);
    const sensitivityResults: any[] = [];
    
    this.criteria.forEach((criterion, index) => {
      // Increase this criterion's weight by 20% and decrease others proportionally
      const modifiedCriteria = [...this.criteria];
      const increase = criterion.weight * 0.2;
      modifiedCriteria[index] = { ...criterion, weight: criterion.weight + increase };
      
      // Adjust other weights proportionally
      const totalOtherWeight = 1 - modifiedCriteria[index].weight;
      const currentOtherWeight = 1 - criterion.weight;
      const adjustmentFactor = totalOtherWeight / currentOtherWeight;
      
      modifiedCriteria.forEach((c, i) => {
        if (i !== index) {
          c.weight *= adjustmentFactor;
        }
      });
      
      // Recalculate scores with modified criteria
      const tempMatrix = new DecisionMatrix(modifiedCriteria);
      const reranked = tempMatrix.evaluateAlternatives(scoredAlternatives);
      const newRanking = reranked.map(alt => alt.rank);
      
      sensitivityResults.push({
        criterion: criterion.name,
        rankingChange: this.calculateRankingDifference(baseRanking, newRanking),
        newTopChoice: reranked[0].name,
      });
    });
    
    return sensitivityResults;
  }
  
  private calculateRankingDifference(ranking1: number[], ranking2: number[]): number {
    return ranking1.reduce((diff, rank, index) => {
      return diff + Math.abs(rank - ranking2[index]);
    }, 0);
  }
}

// Usage example
const createDecisionMatrix = (problem: Problem): DecisionMatrix => {
  const criteria: DecisionCriteria[] = [
    {
      name: 'Implementation Speed',
      weight: 0.25,
      scoreFunction: (alt) => {
        switch (alt.effort) {
          case 'low': return 9;
          case 'medium': return 6;
          case 'high': return 3;
          default: return 5;
        }
      }
    },
    {
      name: 'Risk Level',
      weight: 0.20,
      scoreFunction: (alt) => {
        switch (alt.risk) {
          case 'low': return 9;
          case 'medium': return 6;
          case 'high': return 3;
          default: return 5;
        }
      }
    },
    {
      name: 'Long-term Maintainability',
      weight: 0.30,
      scoreFunction: (alt) => {
        switch (alt.maintainability) {
          case 'high': return 9;
          case 'medium': return 6;
          case 'low': return 3;
          default: return 5;
        }
      }
    },
    {
      name: 'Cost Effectiveness',
      weight: 0.25,
      scoreFunction: (alt) => {
        switch (alt.cost) {
          case 'low': return 9;
          case 'medium': return 6;
          case 'high': return 3;
          default: return 5;
        }
      }
    }
  ];
  
  return new DecisionMatrix(criteria);
};
```

## Class Hierarchies & Architectural Patterns

### 1. Factory Pattern Implementation

#### Step-by-Step Factory Design
```javascript
// factories/componentFactory.ts
interface ComponentConfig {
  type: string;
  props: Record<string, any>;
  children?: ComponentConfig[];
}

abstract class ComponentFactory {
  abstract createComponent(config: ComponentConfig): React.ComponentType;
  
  // Template method pattern
  buildComponent(config: ComponentConfig): React.ComponentType {
    const component = this.createComponent(config);
    return this.applyMiddleware(component, config);
  }
  
  protected applyMiddleware(
    component: React.ComponentType, 
    config: ComponentConfig
  ): React.ComponentType {
    // Apply common enhancements
    let enhancedComponent = component;
    
    if (config.props.trackAnalytics) {
      enhancedComponent = withAnalytics(enhancedComponent);
    }
    
    if (config.props.errorBoundary) {
      enhancedComponent = withErrorBoundary(enhancedComponent);
    }
    
    if (config.props.loading) {
      enhancedComponent = withLoadingState(enhancedComponent);
    }
    
    return enhancedComponent;
  }
}

class FormComponentFactory extends ComponentFactory {
  createComponent(config: ComponentConfig): React.ComponentType {
    switch (config.type) {
      case 'input':
        return this.createInput(config);
      case 'textarea':
        return this.createTextarea(config);
      case 'select':
        return this.createSelect(config);
      case 'form':
        return this.createForm(config);
      default:
        throw new Error(`Unknown form component type: ${config.type}`);
    }
  }
  
  private createInput(config: ComponentConfig): React.ComponentType {
    return (props: any) => (
      <Input
        {...config.props}
        {...props}
        validation={this.createValidation(config.props.validation)}
      />
    );
  }
  
  private createTextarea(config: ComponentConfig): React.ComponentType {
    return (props: any) => (
      <Textarea
        {...config.props}
        {...props}
        autoResize={config.props.autoResize}
      />
    );
  }
  
  private createSelect(config: ComponentConfig): React.ComponentType {
    return (props: any) => (
      <Select
        {...config.props}
        {...props}
        options={this.processOptions(config.props.options)}
      />
    );
  }
  
  private createForm(config: ComponentConfig): React.ComponentType {
    return (props: any) => {
      const childComponents = config.children?.map((childConfig, index) => {
        const ChildComponent = this.createComponent(childConfig);
        return <ChildComponent key={index} />;
      });
      
      return (
        <Form
          {...config.props}
          {...props}
          onSubmit={this.createSubmitHandler(config.props.submitConfig)}
        >
          {childComponents}
        </Form>
      );
    };
  }
  
  private createValidation(validationConfig: any) {
    if (!validationConfig) return undefined;
    
    return (value: any) => {
      const errors: string[] = [];
      
      if (validationConfig.required && !value) {
        errors.push('This field is required');
      }
      
      if (validationConfig.minLength && value?.length < validationConfig.minLength) {
        errors.push(`Minimum length is ${validationConfig.minLength}`);
      }
      
      if (validationConfig.pattern && !new RegExp(validationConfig.pattern).test(value)) {
        errors.push('Invalid format');
      }
      
      return errors.length > 0 ? errors : undefined;
    };
  }
  
  private processOptions(options: any[]): any[] {
    return options?.map(option => ({
      ...option,
      value: option.value,
      label: option.label || option.value,
    })) || [];
  }
  
  private createSubmitHandler(submitConfig: any) {
    return async (formData: any) => {
      if (submitConfig.validate) {
        const validation = await submitConfig.validate(formData);
        if (!validation.isValid) {
          throw new Error(validation.errors.join(', '));
        }
      }
      
      if (submitConfig.transform) {
        formData = submitConfig.transform(formData);
      }
      
      if (submitConfig.endpoint) {
        return await fetch(submitConfig.endpoint, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(formData),
        });
      }
      
      if (submitConfig.onSubmit) {
        return await submitConfig.onSubmit(formData);
      }
    };
  }
}

// Usage
const formFactory = new FormComponentFactory();

const LoginForm = formFactory.buildComponent({
  type: 'form',
  props: {
    submitConfig: {
      endpoint: '/api/auth/login',
      validate: async (data) => ({ isValid: !!data.email && !!data.password }),
    },
    trackAnalytics: true,
    errorBoundary: true,
  },
  children: [
    {
      type: 'input',
      props: {
        name: 'email',
        type: 'email',
        label: 'Email',
        validation: { required: true, pattern: '^[^@]+@[^@]+\.[^@]+$' },
      },
    },
    {
      type: 'input',
      props: {
        name: 'password',
        type: 'password',
        label: 'Password',
        validation: { required: true, minLength: 8 },
      },
    },
  ],
});
```

### 2. Observer Pattern for Component Communication

#### Step-by-Step Observer Implementation
```javascript
// patterns/observer.ts
interface Observer<T = any> {
  update(data: T): void;
}

interface Observable<T = any> {
  subscribe(observer: Observer<T>): () => void;
  unsubscribe(observer: Observer<T>): void;
  notify(data: T): void;
}

class EventBus<T = any> implements Observable<T> {
  private observers = new Set<Observer<T>>();
  private history: T[] = [];
  private maxHistorySize = 100;
  
  subscribe(observer: Observer<T>): () => void {
    this.observers.add(observer);
    
    // Return unsubscribe function
    return () => {
      this.unsubscribe(observer);
    };
  }
  
  unsubscribe(observer: Observer<T>): void {
    this.observers.delete(observer);
  }
  
  notify(data: T): void {
    // Store in history
    this.history.push(data);
    if (this.history.length > this.maxHistorySize) {
      this.history.shift();
    }
    
    // Notify all observers
    this.observers.forEach(observer => {
      try {
        observer.update(data);
      } catch (error) {
        console.error('Observer update failed:', error);
      }
    });
  }
  
  getHistory(): T[] {
    return [...this.history];
  }
  
  clear(): void {
    this.observers.clear();
    this.history = [];
  }
}

// Typed event bus for different event types
interface AppEvents {
  userLogin: { userId: string; timestamp: number };
  dataUpdate: { entity: string; data: any };
  notification: { type: 'info' | 'warning' | 'error'; message: string };
  navigation: { from: string; to: string };
}

class TypedEventBus {
  private eventBuses = new Map<keyof AppEvents, EventBus>();
  
  private getEventBus<K extends keyof AppEvents>(eventType: K): EventBus<AppEvents[K]> {
    if (!this.eventBuses.has(eventType)) {
      this.eventBuses.set(eventType, new EventBus<AppEvents[K]>());
    }
    return this.eventBuses.get(eventType) as EventBus<AppEvents[K]>;
  }
  
  subscribe<K extends keyof AppEvents>(
    eventType: K,
    observer: Observer<AppEvents[K]>
  ): () => void {
    return this.getEventBus(eventType).subscribe(observer);
  }
  
  emit<K extends keyof AppEvents>(eventType: K, data: AppEvents[K]): void {
    this.getEventBus(eventType).notify(data);
  }
  
  unsubscribe<K extends keyof AppEvents>(
    eventType: K,
    observer: Observer<AppEvents[K]>
  ): void {
    this.getEventBus(eventType).unsubscribe(observer);
  }
}

// React hook for using event bus
function useEventBus() {
  const eventBus = useMemo(() => new TypedEventBus(), []);
  
  const subscribe = useCallback(<K extends keyof AppEvents>(
    eventType: K,
    handler: (data: AppEvents[K]) => void,
    deps: any[] = []
  ) => {
    const observer: Observer<AppEvents[K]> = { update: handler };
    
    const unsubscribe = eventBus.subscribe(eventType, observer);
    
    useEffect(() => {
      return unsubscribe;
    }, deps);
  }, [eventBus]);
  
  const emit = useCallback(<K extends keyof AppEvents>(
    eventType: K,
    data: AppEvents[K]
  ) => {
    eventBus.emit(eventType, data);
  }, [eventBus]);
  
  return { subscribe, emit };
}

// Usage example
const UserProfile = ({ userId }: { userId: string }) => {
  const { subscribe, emit } = useEventBus();
  const [user, setUser] = useState(null);
  
  // Subscribe to data updates
  subscribe('dataUpdate', (event) => {
    if (event.entity === 'user' && event.data.id === userId) {
      setUser(event.data);
    }
  }, [userId]);
  
  // Subscribe to user login events
  subscribe('userLogin', (event) => {
    console.log('User logged in:', event.userId);
  }, []);
  
  const handleUpdate = useCallback((updatedData: any) => {
    // Emit data update event
    emit('dataUpdate', {
      entity: 'user',
      data: { ...user, ...updatedData },
    });
    
    // Emit notification
    emit('notification', {
      type: 'info',
      message: 'Profile updated successfully',
    });
  }, [user, emit]);
  
  return (
    <div>
      <h1>{user?.name}</h1>
      <button onClick={() => handleUpdate({ name: 'Updated Name' })}>
        Update Profile
      </button>
    </div>
  );
};
```

### 3. Composition Pattern for Flexible Components

#### Step-by-Step Composition Design
```javascript
// patterns/composition.ts
interface ComposableComponent {
  render(): React.ReactElement;
  compose(other: ComposableComponent): ComposableComponent;
}

abstract class BaseComponent implements ComposableComponent {
  abstract render(): React.ReactElement;
  
  compose(other: ComposableComponent): ComposableComponent {
    return new CompositeComponent([this, other]);
  }
}

class CompositeComponent implements ComposableComponent {
  constructor(private components: ComposableComponent[]) {}
  
  render(): React.ReactElement {
    return (
      <React.Fragment>
        {this.components.map((component, index) => (
          React.cloneElement(component.render(), { key: index })
        ))}
      </React.Fragment>
    );
  }
  
  compose(other: ComposableComponent): ComposableComponent {
    return new CompositeComponent([...this.components, other]);
  }
}

// Decorator pattern for component enhancement
abstract class ComponentDecorator implements ComposableComponent {
  constructor(protected component: ComposableComponent) {}
  
  render(): React.ReactElement {
    return this.component.render();
  }
  
  compose(other: ComposableComponent): ComposableComponent {
    return new CompositeComponent([this, other]);
  }
}

class LoadingDecorator extends ComponentDecorator {
  constructor(
    component: ComposableComponent,
    private isLoading: boolean,
    private loadingComponent?: React.ComponentType
  ) {
    super(component);
  }
  
  render(): React.ReactElement {
    if (this.isLoading) {
      const LoadingComponent = this.loadingComponent || DefaultLoader;
      return <LoadingComponent />;
    }
    
    return this.component.render();
  }
}

class ErrorBoundaryDecorator extends ComponentDecorator {
  constructor(
    component: ComposableComponent,
    private fallbackComponent?: React.ComponentType<{ error: Error }>
  ) {
    super(component);
  }
  
  render(): React.ReactElement {
    const FallbackComponent = this.fallbackComponent || DefaultErrorFallback;
    
    return (
      <ErrorBoundary fallback={FallbackComponent}>
        {this.component.render()}
      </ErrorBoundary>
    );
  }
}

class AnalyticsDecorator extends ComponentDecorator {
  constructor(
    component: ComposableComponent,
    private trackingConfig: {
      event: string;
      properties?: Record<string, any>;
    }
  ) {
    super(component);
  }
  
  render(): React.ReactElement {
    const originalElement = this.component.render();
    
    return React.cloneElement(originalElement, {
      onClick: (e: any) => {
        // Track analytics
        analytics.track(this.trackingConfig.event, {
          ...this.trackingConfig.properties,
          timestamp: Date.now(),
        });
        
        // Call original onClick if exists
        if (originalElement.props.onClick) {
          originalElement.props.onClick(e);
        }
      },
    });
  }
}

// Builder pattern for complex component construction
class ComponentBuilder {
  private component: ComposableComponent;
  
  constructor(baseComponent: ComposableComponent) {
    this.component = baseComponent;
  }
  
  withLoading(
    isLoading: boolean, 
    loadingComponent?: React.ComponentType
  ): ComponentBuilder {
    this.component = new LoadingDecorator(this.component, isLoading, loadingComponent);
    return this;
  }
  
  withErrorBoundary(
    fallbackComponent?: React.ComponentType<{ error: Error }>
  ): ComponentBuilder {
    this.component = new ErrorBoundaryDecorator(this.component, fallbackComponent);
    return this;
  }
  
  withAnalytics(
    event: string, 
    properties?: Record<string, any>
  ): ComponentBuilder {
    this.component = new AnalyticsDecorator(this.component, { event, properties });
    return this;
  }
  
  compose(other: ComposableComponent): ComponentBuilder {
    this.component = this.component.compose(other);
    return this;
  }
  
  build(): React.ComponentType {
    return () => this.component.render();
  }
}

// Usage example
class ButtonComponent extends BaseComponent {
  constructor(
    private props: {
      children: React.ReactNode;
      onClick?: () => void;
      variant?: 'primary' | 'secondary';
    }
  ) {
    super();
  }
  
  render(): React.ReactElement {
    return (
      <button
        className={`btn btn--${this.props.variant || 'primary'}`}
        onClick={this.props.onClick}
      >
        {this.props.children}
      </button>
    );
  }
}

class IconComponent extends BaseComponent {
  constructor(private iconName: string) {
    super();
  }
  
  render(): React.ReactElement {
    return <Icon name={this.iconName} />;
  }
}

// Build a complex component
const EnhancedButton = new ComponentBuilder(
  new ButtonComponent({
    children: 'Click me',
    onClick: () => console.log('Button clicked'),
    variant: 'primary',
  })
)
  .withLoading(false)
  .withErrorBoundary()
  .withAnalytics('button_click', { buttonType: 'primary' })
  .compose(new IconComponent('arrow-right'))
  .build();
```

### 4. Plugin Architecture for Extensible Systems

#### Step-by-Step Plugin System
```javascript
// plugins/pluginSystem.ts
interface Plugin {
  name: string;
  version: string;
  dependencies?: string[];
  init(context: PluginContext): Promise<void> | void;
  destroy?(): Promise<void> | void;
}

interface PluginContext {
  registerHook(name: string, handler: Function): void;
  executeHook(name: string, ...args: any[]): any[];
  registerComponent(name: string, component: React.ComponentType): void;
  getComponent(name: string): React.ComponentType | undefined;
  registerService(name: string, service: any): void;
  getService<T = any>(name: string): T | undefined;
  config: Record<string, any>;
}

class PluginManager {
  private plugins = new Map<string, Plugin>();
  private hooks = new Map<string, Function[]>();
  private components = new Map<string, React.ComponentType>();
  private services = new Map<string, any>();
  private config: Record<string, any>;
  
  constructor(config: Record<string, any> = {}) {
    this.config = config;
  }
  
  async registerPlugin(plugin: Plugin): Promise<void> {
    // Check dependencies
    if (plugin.dependencies) {
      for (const dep of plugin.dependencies) {
        if (!this.plugins.has(dep)) {
          throw new Error(`Plugin ${plugin.name} requires ${dep} but it's not registered`);
        }
      }
    }
    
    // Check for name conflicts
    if (this.plugins.has(plugin.name)) {
      throw new Error(`Plugin ${plugin.name} is already registered`);
    }
    
    this.plugins.set(plugin.name, plugin);
    
    // Initialize plugin
    const context: PluginContext = {
      registerHook: (name, handler) => this.registerHook(name, handler),
      executeHook: (name, ...args) => this.executeHook(name, ...args),
      registerComponent: (name, component) => this.components.set(name, component),
      getComponent: (name) => this.components.get(name),
      registerService: (name, service) => this.services.set(name, service),
      getService: (name) => this.services.get(name),
      config: this.config,
    };
    
    await plugin.init(context);
  }
  
  async unregisterPlugin(name: string): Promise<void> {
    const plugin = this.plugins.get(name);
    if (!plugin) return;
    
    // Check if other plugins depend on this one
    for (const [, otherPlugin] of this.plugins) {
      if (otherPlugin.dependencies?.includes(name)) {
        throw new Error(`Cannot unregister ${name} because ${otherPlugin.name} depends on it`);
      }
    }
    
    // Destroy plugin
    if (plugin.destroy) {
      await plugin.destroy();
    }
    
    this.plugins.delete(name);
    
    // Clean up hooks registered by this plugin
    this.hooks.forEach((handlers, hookName) => {
      this.hooks.set(hookName, handlers.filter(h => h.pluginName !== name));
    });
  }
  
  private registerHook(name: string, handler: Function): void {
    if (!this.hooks.has(name)) {
      this.hooks.set(name, []);
    }
    this.hooks.get(name)!.push(handler);
  }
  
  executeHook(name: string, ...args: any[]): any[] {
    const handlers = this.hooks.get(name) || [];
    return handlers.map(handler => {
      try {
        return handler(...args);
      } catch (error) {
        console.error(`Hook ${name} handler failed:`, error);
        return null;
      }
    }).filter(result => result !== null);
  }
  
  getPlugins(): Plugin[] {
    return Array.from(this.plugins.values());
  }
  
  getComponent(name: string): React.ComponentType | undefined {
    return this.components.get(name);
  }
  
  getService<T = any>(name: string): T | undefined {
    return this.services.get(name);
  }
}

// Example plugins
class ThemePlugin implements Plugin {
  name = 'theme';
  version = '1.0.0';
  
  init(context: PluginContext): void {
    // Register theme service
    const themeService = {
      currentTheme: 'light',
      setTheme: (theme: string) => {
        this.currentTheme = theme;
        context.executeHook('theme:changed', theme);
      },
      getTheme: () => this.currentTheme,
    };
    
    context.registerService('theme', themeService);
    
    // Register theme provider component
    const ThemeProvider = ({ children }: { children: React.ReactNode }) => (
      <div data-theme={themeService.currentTheme}>
        {children}
      </div>
    );
    
    context.registerComponent('ThemeProvider', ThemeProvider);
    
    // Register hook handler
    context.registerHook('app:init', () => {
      console.log('Theme plugin initialized');
    });
  }
}

class AnalyticsPlugin implements Plugin {
  name = 'analytics';
  version = '1.0.0';
  dependencies = ['theme'];
  
  init(context: PluginContext): void {
    const analyticsService = {
      track: (event: string, properties?: Record<string, any>) => {
        console.log('Analytics:', { event, properties });
        // Send to analytics provider
      },
    };
    
    context.registerService('analytics', analyticsService);
    
    // Listen to theme changes
    context.registerHook('theme:changed', (newTheme: string) => {
      analyticsService.track('theme_changed', { theme: newTheme });
    });
    
    // Register analytics HOC
    const withAnalytics = (WrappedComponent: React.ComponentType, eventName: string) => {
      return (props: any) => {
        useEffect(() => {
          analyticsService.track(`component_mounted`, { component: eventName });
        }, []);
        
        return <WrappedComponent {...props} />;
      };
    };
    
    context.registerComponent('withAnalytics', withAnalytics as any);
  }
}

// Usage
const pluginManager = new PluginManager({
  apiEndpoint: 'https://api.example.com',
  debugMode: true,
});

// Register plugins
await pluginManager.registerPlugin(new ThemePlugin());
await pluginManager.registerPlugin(new AnalyticsPlugin());

// Execute initialization hooks
pluginManager.executeHook('app:init');

// Use plugin services and components
const App = () => {
  const ThemeProvider = pluginManager.getComponent('ThemeProvider');
  const themeService = pluginManager.getService('theme');
  const analyticsService = pluginManager.getService('analytics');
  
  return (
    <ThemeProvider>
      <div>
        <button onClick={() => themeService.setTheme('dark')}>
          Toggle Theme
        </button>
        <button onClick={() => analyticsService.track('button_click')}>
          Track Event
        </button>
      </div>
    </ThemeProvider>
  );
};
```

## Testing Strategies for Complex Systems

### 1. Integration Testing for System Architecture
```javascript
// tests/integration/systemIntegration.test.ts
describe('State Management Integration', () => {
  let store: Store;
  let errorManager: ErrorManager;
  let performanceManager: PerformanceManager;
  
  beforeEach(() => {
    store = createTestStore();
    errorManager = ErrorManager.getInstance();
    performanceManager = PerformanceManager.getInstance();
  });
  
  it('should handle authentication flow with error recovery', async () => {
    const mockCredentials = { email: 'test@test.com', password: 'password' };
    
    // Simulate network error on first attempt
    const mockAuthService = {
      login: jest.fn()
        .mockRejectedValueOnce(new NetworkError('Connection failed'))
        .mockResolvedValueOnce({ user: { id: '1', email: 'test@test.com' }, token: 'abc123' })
    };
    
    // Test retry mechanism
    await expect(errorManager.withRetry(
      () => mockAuthService.login(mockCredentials),
      { maxAttempts: 2, shouldRetry: (error) => error instanceof NetworkError }
    )).resolves.toBeDefined();
    
    expect(mockAuthService.login).toHaveBeenCalledTimes(2);
  });
  
  it('should track performance metrics during data operations', async () => {
    const performanceTracker = jest.spyOn(performanceManager, 'recordAPICall');
    
    // Simulate API call
    const startTime = performance.now();
    await new Promise(resolve => setTimeout(resolve, 100));
    const endTime = performance.now();
    
    performanceManager.recordAPICall('/api/users', endTime - startTime, 200);
    
    expect(performanceTracker).toHaveBeenCalledWith('/api/users', expect.any(Number), 200);
  });
});

describe('WebSocket Integration', () => {
  let connectionManager: WebSocketConnectionManager;
  let mockWebSocket: any;
  
  beforeEach(() => {
    mockWebSocket = {
      send: jest.fn(),
      close: jest.fn(),
      addEventListener: jest.fn(),
      removeEventListener: jest.fn(),
    };
    
    global.WebSocket = jest.fn(() => mockWebSocket);
    
    connectionManager = new WebSocketConnectionManager({
      url: 'ws://test',
      reconnectAttempts: 2,
      reconnectInterval: 100,
      maxReconnectInterval: 500,
      heartbeatInterval: 1000,
      messageTimeout: 1000,
    });
  });
  
  it('should handle connection lifecycle correctly', async () => {
    const connectPromise = connectionManager.connect();
    
    // Simulate successful connection
    const openHandler = mockWebSocket.addEventListener.mock.calls
      .find(([event]) => event === 'open')[1];
    openHandler();
    
    await connectPromise;
    
    expect(connectionManager.isConnected()).toBe(true);
    
    // Test message sending
    await connectionManager.send({ type: 'test', data: 'hello' });
    expect(mockWebSocket.send).toHaveBeenCalled();
  });
});
```

### 2. Performance Testing
```javascript
// tests/performance/performanceTests.test.ts
describe('Performance Benchmarks', () => {
  it('should render large lists efficiently', async () => {
    const data = Array.from({ length: 10000 }, (_, i) => ({ id: i, name: `Item ${i}` }));
    
    const startTime = performance.now();
    
    const { container } = render(
      <VirtualList
        items={data}
        itemHeight={50}
        containerHeight={500}
        renderItem={({ item }) => <div>{item.name}</div>}
      />
    );
    
    const endTime = performance.now();
    const renderTime = endTime - startTime;
    
    // Should render in under 100ms
    expect(renderTime).toBeLessThan(100);
    
    // Should only render visible items
    const renderedItems = container.querySelectorAll('[data-testid="list-item"]');
    expect(renderedItems.length).toBeLessThanOrEqual(12); // ~10 visible + buffer
  });
  
  it('should handle cache operations efficiently', async () => {
    const cacheManager = new CacheManager({ memoryMaxSize: 1000 });
    
    // Benchmark cache set operations
    const setStartTime = performance.now();
    for (let i = 0; i < 1000; i++) {
      await cacheManager.set(`key-${i}`, { data: `value-${i}` });
    }
    const setEndTime = performance.now();
    
    // Benchmark cache get operations
    const getStartTime = performance.now();
    for (let i = 0; i < 1000; i++) {
      await cacheManager.get(`key-${i}`);
    }
    const getEndTime = performance.now();
    
    expect(setEndTime - setStartTime).toBeLessThan(100); // 100ms for 1000 sets
    expect(getEndTime - getStartTime).toBeLessThan(50);  // 50ms for 1000 gets
  });
});
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

### State Management & Architecture
- Redux/Zustand for state management
- React Query/SWR for server state
- Context API for local state sharing
- WebSocket libraries for real-time communication

### Performance & Monitoring
- Web Vitals API for performance metrics
- Sentry/Bugsnag for error monitoring
- Performance Observer API for custom metrics
- Lighthouse for performance auditing

### Development Tools
- Webpack/Vite for bundling
- ESLint/Prettier for code quality
- Husky for git hooks
- GitHub Actions/Jenkins for CI/CD

## Implementation Roadmap

### Phase 1: Foundation (Weeks 1-4)
1. **Week 1-2**: Set up basic architecture
   - Create project structure
   - Set up build tools and development environment
   - Implement basic state management
   - Create initial component library structure

2. **Week 3-4**: Core Components & Patterns
   - Implement atomic design system
   - Create base components (Button, Input, etc.)
   - Set up testing framework
   - Implement error boundaries

### Phase 2: Advanced Features (Weeks 5-8)
1. **Week 5-6**: Authentication & Authorization
   - Implement token management
   - Create authentication context
   - Set up route protection
   - Build login/signup flows

2. **Week 7-8**: Performance & Caching
   - Implement caching layer
   - Add performance monitoring
   - Set up lazy loading
   - Optimize bundle sizes

### Phase 3: Production Features (Weeks 9-12)
1. **Week 9-10**: Real-time & Advanced Features
   - Implement WebSocket management
   - Add real-time updates
   - Create plugin architecture
   - Build advanced patterns

2. **Week 11-12**: Testing & Deployment
   - Comprehensive testing strategy
   - Performance testing
   - Production deployment
   - Documentation and training

### Phase 4: Maintenance & Evolution (Ongoing)
1. **Monthly Reviews**: 
   - Performance analysis
   - User feedback integration
   - Security updates
   - Feature enhancements

2. **Quarterly Planning**:
   - Architecture reviews
   - Technology stack evaluation
   - Team training
   - Process improvements

## Best Practices Summary

### Architecture Principles
1. **Separation of Concerns**: Keep business logic separate from UI components
2. **Single Responsibility**: Each component/module should have one clear purpose
3. **Dependency Inversion**: Depend on abstractions, not concrete implementations
4. **Open/Closed Principle**: Open for extension, closed for modification

### Development Guidelines
1. **Code Quality**:
   - Use TypeScript for type safety
   - Follow consistent naming conventions
   - Write self-documenting code
   - Maintain high test coverage

2. **Performance**:
   - Implement lazy loading for large components
   - Use memoization strategically
   - Optimize bundle sizes
   - Monitor Core Web Vitals

3. **Accessibility**:
   - Follow WCAG guidelines
   - Use semantic HTML
   - Implement keyboard navigation
   - Test with screen readers

4. **Security**:
   - Validate all inputs
   - Use secure authentication
   - Implement proper error handling
   - Follow OWASP guidelines

### Team Practices
1. **Code Reviews**:
   - Review all code before merging
   - Focus on architecture and maintainability
   - Share knowledge and best practices
   - Document decisions and rationale

2. **Documentation**:
   - Keep documentation up to date
   - Include examples and use cases
   - Document architectural decisions
   - Create onboarding guides

3. **Continuous Improvement**:
   - Regular retrospectives
   - Performance monitoring
   - User feedback integration
   - Technology evaluation

## Conclusion

This comprehensive guide provides the foundation for building scalable, maintainable, and production-ready frontend systems. The detailed implementation patterns and architectural decisions covered here enable development teams to:

1. **Build Robust Systems**: With proper error handling, performance monitoring, and testing strategies
2. **Scale Effectively**: Through modular architecture, caching strategies, and performance optimization
3. **Maintain Quality**: With comprehensive testing, code review processes, and documentation
4. **Evolve Continuously**: Through plugin architectures, decision frameworks, and iterative improvement

The key to success lies in:
- **Starting Simple**: Begin with core patterns and gradually add complexity
- **Measuring Everything**: Implement monitoring and analytics from day one
- **Planning for Change**: Design flexible architectures that can evolve
- **Focusing on Users**: Always prioritize user experience and accessibility

Remember that these patterns and implementations should be adapted to your specific needs, team size, and project requirements. The goal is not to implement every pattern, but to choose the right ones for your context and apply them consistently.

By following these guidelines and implementing these patterns thoughtfully, you'll create frontend systems that not only meet today's requirements but can grow and adapt to future needs while maintaining high quality and performance standards.

### Design
- Figma/Sketch for design specifications
- Design tokens for consistent styling
- Component libraries (Material-UI, Ant Design)

## Conclusion

Effective UI component design requires balancing immediate needs with long-term maintainability. By following these principles and patterns, you can create components that are reusable, testable, and scalable across your application.