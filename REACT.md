# React.js Middle/Senior Dasturchilar uchun Interview Savollari va Javoblari

## 1. Virtual DOM va Reconciliation

**Savol:** Virtual DOM nima va React uni qanday ishlatadi? Reconciliation jarayonini tushuntiring.

**Javob:** 
Virtual DOM - bu real DOM ning JavaScript obyekti sifatidagi yengil nusxasi. React quyidagi jarayonni amalga oshiradi:

1. **Rendering:** State o'zgarganda, React yangi Virtual DOM daraxtini yaratadi
2. **Diffing:** React eski va yangi Virtual DOM larni taqqoslaydi (reconciliation)
3. **Batching:** O'zgarishlarni guruhlaydi va minimal DOM operatsiyalarini aniqlaydi
4. **Updating:** Faqat o'zgargan elementlarni real DOM da yangilaydi

React Fiber arxitekturasi reconciliation jarayonini yanada samarali qiladi va katta update larni kichik qismlarga bo'lib, prioritetlashtirish imkonini beradi.

## 2. useMemo vs useCallback

**Savol:** `useMemo` va `useCallback` hooklari orasidagi farq nima? Qachon ishlatish kerak?

**Javob:**

**useMemo:**
- Qimmat hisob-kitoblar natijasini memoizatsiya qiladi
- Qiymatni qaytaradi
```javascript
const expensiveValue = useMemo(() => {
  return computeExpensiveValue(a, b);
}, [a, b]);
```

**useCallback:**
- Funksiyaning o'zini memoizatsiya qiladi
- Funksiyani qaytaradi
```javascript
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

**Qachon ishlatish:**
- Child componentga prop sifatida funksiya yuborilganda
- Dependency array da funksiya ishlatilganda
- Performance optimizatsiya zarur bo'lganda

## 3. React Context vs Redux

**Savol:** React Context va Redux orasida qanday farq bor? Qachon qaysi birini tanlash kerak?

**Javob:**

**React Context:**
- React ga built-in
- Oddiy state managementga mos
- Kichik-o'rta loyihalar uchun yetarli
- Middleware yo'q
- DevTools cheklangan

**Redux:**
- Katta va murakkab loyihalar uchun
- Middleware (redux-thunk, redux-saga)
- Kuchli DevTools (time-travel debugging)
- Predictable state management
- Strict architectural patterns

**Tanlash mezonlari:**
- Global state hajmi kichik → Context
- Murakkab state logic → Redux
- Middleware kerak → Redux
- Oddiy prop drilling muammosi → Context

## 4. React Performance Optimization

**Savol:** React ilovasining performance ini yaxshilash uchun qanday usullar mavjud?

**Javob:**

1. **React.memo:** Component ni memoizatsiya qilish
```javascript
const MyComponent = React.memo(({ data }) => {
  return <div>{data}</div>;
});
```

2. **Code Splitting:**
```javascript
const LazyComponent = React.lazy(() => import('./LazyComponent'));
```

3. **Virtualization:** Katta ro'yxatlar uchun (react-window, react-virtualized)

4. **useCallback va useMemo:** Keraksiz re-renderlarni oldini olish

5. **Key prop:** Listlarda to'g'ri key ishlatish

6. **Debouncing/Throttling:** Input va scroll eventlarda

7. **Image Optimization:** Lazy loading, WebP format

8. **Bundle Size:** Tree shaking, code splitting

## 5. Custom Hooks

**Savol:** Custom hook yaratishning best practice lari nimalardan iborat?

**Javob:**

**Qoidalar:**
1. "use" prefiksi bilan boshlash
2. Reusable logic ajratish
3. Hooks qoidalariga rioya qilish

**Misol:**
```javascript
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(handler);
  }, [value, delay]);
  
  return debouncedValue;
}

// Ishlatish
const searchTerm = useDebounce(inputValue, 500);
```

**Best Practices:**
- Bitta vazifani bajarishi kerak (Single Responsibility)
- Testga oson bo'lishi
- Dependency array ni to'g'ri boshqarish
- Cleanup funksiyalarini unutmaslik

## 6. Server-Side Rendering (SSR)

**Savol:** React da SSR nima va qanday afzalliklari bor?

**Javob:**

**SSR nima:**
HTML serverda generatsiya qilinib, tayyor holda browserga yuboriladi.

**Afzalliklari:**
1. **SEO:** Search engine crawlerlar content ni ko'radi
2. **Performance:** Birinchi sahifa tezroq yuklanadi (FCP)
3. **Social Media:** Meta taglari to'g'ri ishlaydi

**Kamchiliklari:**
- Server yuklamasi ortadi
- Murakkab deployment
- Hydration muammolari

**Framework lar:**
- Next.js (eng mashhur)
- Remix
- Gatsby

**Next.js misol:**
```javascript
export async function getServerSideProps() {
  const data = await fetchData();
  return { props: { data } };
}
```

## 7. Error Boundaries

**Savol:** Error Boundaries nima va qanday implement qilinadi?

**Javob:**

Error Boundaries - component daraxtidagi JavaScript xatolarini ushlash, logga yozish va fallback UI ko'rsatish uchun ishlatiladi.

**Class Component bilan:**
```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true };
  }

  componentDidCatch(error, errorInfo) {
    console.log('Error:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return <h1>Xatolik yuz berdi.</h1>;
    }
    return this.props.children;
  }
}

// Ishlatish
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

**Muhim:**
- Faqat class componentlarda ishlaydi
- Event handlerlar uchun ishlamaydi
- Async code xatolarini ushlamaydi

## 8. React Hooks Rules

**Savol:** React Hooks qoidalari nimalardan iborat va nima uchun muhim?

**Javob:**

**Asosiy qoidalar:**

1. **Faqat top level da chaqirish:**
```javascript
// ✅ To'g'ri
function Component() {
  const [state, setState] = useState(0);
  
  // ❌ Noto'g'ri
  if (condition) {
    const [state, setState] = useState(0);
  }
}
```

2. **Faqat React funksiyalarida chaqirish:**
- Function componentlarda
- Custom hooklarda

**Nima uchun muhim:**
- React hooklar ro'yxatiga tayanadi
- Har safar bir xil tartibda chaqirilishi kerak
- Shartli ravishda chaqirish tartibni buzadi

**ESLint plugin:**
```bash
npm install eslint-plugin-react-hooks
```

## 9. useEffect Dependencies

**Savol:** useEffect da dependency array qanday ishlaydi va keng tarqalgan xatolar qanday?

**Javob:**

**Dependency Array turlari:**

1. **Bo'sh array:** Faqat mount da ishlaydi
```javascript
useEffect(() => {
  // Faqat bir marta
}, []);
```

2. **Dependency bilan:** Dependency o'zgarganda
```javascript
useEffect(() => {
  // userId o'zgarganda
}, [userId]);
```

3. **Array yo'q:** Har render da
```javascript
useEffect(() => {
  // Har render da
});
```

**Keng tarqalgan xatolar:**

1. **Dependency ni unutish:**
```javascript
// ❌ Noto'g'ri
useEffect(() => {
  fetchData(userId); // userId dependency emas
}, []);

// ✅ To'g'ri
useEffect(() => {
  fetchData(userId);
}, [userId]);
```

2. **Object/Array dependency:**
```javascript
// ❌ Har render da yangi object
useEffect(() => {
  // ...
}, [{ id: userId }]);

// ✅ Primitive qiymat
useEffect(() => {
  // ...
}, [userId]);
```

## 10. React 18 yangiliklari

**Savol:** React 18 da qanday yangi features qo'shildi?

**Javob:**

**Asosiy features:**

1. **Automatic Batching:**
```javascript
// React 18 da avtomatik batch qilinadi
setTimeout(() => {
  setCount(c => c + 1);
  setFlag(f => !f);
  // Faqat bitta re-render
}, 1000);
```

2. **Transitions:**
```javascript
import { useTransition } from 'react';

const [isPending, startTransition] = useTransition();

startTransition(() => {
  setSearchQuery(input); // Past prioritetli
});
```

3. **Suspense for SSR:**
```javascript
<Suspense fallback={<Spinner />}>
  <Comments />
</Suspense>
```

4. **Concurrent Rendering:**
- Interruptible rendering
- Background rendering
- Priority-based updates

5. **New Root API:**
```javascript
// React 18
import { createRoot } from 'react-dom/client';
const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

## 11. State Management Architecture

**Savol:** Katta React ilovasida state managementni qanday strukturalash kerak?

**Javob:**

**State turlari:**

1. **Local State:** Component ichidagi state
2. **Lifted State:** Parent componentga ko'tarilgan
3. **Global State:** Barcha componentlar uchun
4. **Server State:** Backend dan kelgan data

**Arxitektura pattern:**

```
src/
  store/
    slices/
      userSlice.js
      productsSlice.js
    middleware/
      api.js
    store.js
  hooks/
    useUser.js
    useProducts.js
  components/
```

**Best Practices:**
- State ni eng past darajada saqlash
- Server state uchun React Query/SWR
- Global state uchun Redux Toolkit/Zustand
- Immutable updates
- Normalized state structure

## 12. Testing React Components

**Savol:** React componentlarni test qilishning eng yaxshi usullari qanday?

**Javob:**

**Testing kutubxonalari:**
- Jest (test runner)
- React Testing Library (component testing)
- Cypress/Playwright (E2E)

**Misol:**
```javascript
import { render, screen, fireEvent } from '@testing-library/react';
import Counter from './Counter';

test('counter increments', () => {
  render(<Counter />);
  
  const button = screen.getByRole('button', { name: /increment/i });
  const count = screen.getByText(/count: 0/i);
  
  fireEvent.click(button);
  
  expect(screen.getByText(/count: 1/i)).toBeInTheDocument();
});
```

**Best Practices:**
- User perspective dan test yozish
- Implementation detaillardan qochish
- Integration testlarga e'tibor berish
- Accessibility testlari
- Mock larni minimal ishlatish

## Xulosa

Bu savollar React.js middle va senior darajadagi dasturchilar uchun eng muhim mavzularni qamrab oladi. Amaliy tajriba va bu kontseptsiyalarni chuqur tushunish interview jarayonida muvaffaqiyat kaliti hisoblanadi.

# React Middle/Senior - Qo'shimcha Interview Savollari

## 1. React Rendering Lifecycle

**Savol:** React component rendering lifecycle ni batafsil tushuntiring. Qaysi lifecycle metodlar qachon chaqiriladi?

**Javob:**

**Mounting (Component yaratilishi):**
```javascript
class MyComponent extends React.Component {
  // 1. constructor() - eng birinchi
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    // State initialization
    // Event handler binding
  }
  
  // 2. static getDerivedStateFromProps(props, state)
  static getDerivedStateFromProps(props, state) {
    // Props dan state yaratish
    // Return yangi state yoki null
    if (props.count !== state.count) {
      return { count: props.count };
    }
    return null;
  }
  
  // 3. render() - JSX qaytaradi
  render() {
    return <div>{this.state.count}</div>;
  }
  
  // 4. componentDidMount() - DOM tayyor
  componentDidMount() {
    // API calls
    // Subscriptions
    // DOM manipulations
    this.fetchData();
  }
}
```

**Updating (Props yoki State o'zgarganda):**
```javascript
class MyComponent extends React.Component {
  // 1. getDerivedStateFromProps(props, state)
  static getDerivedStateFromProps(props, state) {
    // Props o'zgarishiga javob
    return null;
  }
  
  // 2. shouldComponentUpdate(nextProps, nextState)
  shouldComponentUpdate(nextProps, nextState) {
    // Re-render kerakmi?
    return nextProps.id !== this.props.id;
  }
  
  // 3. render()
  render() {
    return <div>{this.state.count}</div>;
  }
  
  // 4. getSnapshotBeforeUpdate(prevProps, prevState)
  getSnapshotBeforeUpdate(prevProps, prevState) {
    // DOM o'zgarishidan oldin ma'lumot olish
    if (prevProps.list.length < this.props.list.length) {
      return this.listRef.scrollHeight;
    }
    return null;
  }
  
  // 5. componentDidUpdate(prevProps, prevState, snapshot)
  componentDidUpdate(prevProps, prevState, snapshot) {
    // DOM yangilangandan keyin
    if (snapshot !== null) {
      this.listRef.scrollTop += 
        this.listRef.scrollHeight - snapshot;
    }
  }
}
```

**Unmounting (Component o'chirilishi):**
```javascript
class MyComponent extends React.Component {
  componentWillUnmount() {
    // Cleanup
    // Unsubscribe
    // Cancel requests
    clearInterval(this.timer);
    this.subscription.unsubscribe();
  }
}
```

**Functional Components bilan (Hooks):**
```javascript
function MyComponent({ userId }) {
  const [data, setData] = useState(null);
  
  // componentDidMount + componentDidUpdate
  useEffect(() => {
    fetchData(userId).then(setData);
  }, [userId]);
  
  // componentWillUnmount
  useEffect(() => {
    const subscription = subscribe(userId);
    
    return () => {
      subscription.unsubscribe();
    };
  }, [userId]);
  
  return <div>{data?.name}</div>;
}
```

---

## 2. Higher-Order Components (HOC)

**Savol:** HOC nima va qanday yaratiladi? Qanday muammolarni hal qiladi?

**Javob:**

**HOC tushunchasi:**
HOC - bu componentni qabul qilib, yangi component qaytaruvchi funksiya.

```javascript
// HOC pattern
function withAuth(WrappedComponent) {
  return function AuthComponent(props) {
    const [isAuthenticated, setIsAuthenticated] = useState(false);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
      checkAuth().then(result => {
        setIsAuthenticated(result);
        setLoading(false);
      });
    }, []);
    
    if (loading) {
      return <Spinner />;
    }
    
    if (!isAuthenticated) {
      return <Redirect to="/login" />;
    }
    
    return <WrappedComponent {...props} />;
  };
}

// Ishlatish
const ProtectedDashboard = withAuth(Dashboard);
const ProtectedSettings = withAuth(Settings);
```

**Amaliy misollar:**

**1. Loading HOC:**
```javascript
function withLoading(WrappedComponent) {
  return function LoadingComponent({ isLoading, ...props }) {
    if (isLoading) {
      return <div>Yuklanmoqda...</div>;
    }
    
    return <WrappedComponent {...props} />;
  };
}

const UserListWithLoading = withLoading(UserList);

// Ishlatish
<UserListWithLoading 
  isLoading={loading} 
  users={users} 
/>
```

**2. Logger HOC:**
```javascript
function withLogger(WrappedComponent) {
  return function LoggerComponent(props) {
    useEffect(() => {
      console.log('Component mounted:', WrappedComponent.name);
      console.log('Props:', props);
      
      return () => {
        console.log('Component unmounted:', WrappedComponent.name);
      };
    }, [props]);
    
    return <WrappedComponent {...props} />;
  };
}
```

**3. Data Fetching HOC:**
```javascript
function withData(WrappedComponent, fetchFn) {
  return function DataComponent(props) {
    const [data, setData] = useState(null);
    const [error, setError] = useState(null);
    const [loading, setLoading] = useState(true);
    
    useEffect(() => {
      fetchFn(props)
        .then(setData)
        .catch(setError)
        .finally(() => setLoading(false));
    }, [props.id]);
    
    return (
      <WrappedComponent
        {...props}
        data={data}
        error={error}
        loading={loading}
      />
    );
  };
}

// Ishlatish
const UserProfile = withData(
  Profile,
  ({ userId }) => fetchUser(userId)
);
```

**HOC Best Practices:**
```javascript
// ✅ TO'G'RI
function withExample(WrappedComponent) {
  function WithExample(props) {
    // Logic
    return <WrappedComponent {...props} />;
  }
  
  // Display name debugging uchun
  WithExample.displayName = 
    `withExample(${WrappedComponent.displayName || WrappedComponent.name})`;
  
  return WithExample;
}

// ❌ NOTO'G'RI - render ichida HOC yaratish
function MyComponent() {
  const EnhancedComponent = withExample(Component);
  return <EnhancedComponent />;
}
```

**HOC vs Hooks:**
```javascript
// HOC usuli
const EnhancedComponent = withAuth(Component);

// Hooks usuli (zamonaviy)
function Component() {
  const { isAuthenticated, loading } = useAuth();
  
  if (loading) return <Spinner />;
  if (!isAuthenticated) return <Redirect to="/login" />;
  
  return <div>Content</div>;
}
```

---

## 3. Render Props Pattern

**Savol:** Render Props pattern nima va qachon ishlatiladi?

**Javob:**

**Render Props tushunchasi:**
Component ni prop sifatida funksiya qabul qilib, nima render qilishni hal qilish.

```javascript
class MouseTracker extends React.Component {
  state = { x: 0, y: 0 };
  
  handleMouseMove = (event) => {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  };
  
  render() {
    return (
      <div onMouseMove={this.handleMouseMove}>
        {/* Render prop */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

// Ishlatish
<MouseTracker
  render={({ x, y }) => (
    <h1>Sichqoncha pozitsiyasi: ({x}, {y})</h1>
  )}
/>
```

**Children as Function:**
```javascript
class DataProvider extends React.Component {
  state = { data: null, loading: true };
  
  componentDidMount() {
    fetchData().then(data => {
      this.setState({ data, loading: false });
    });
  }
  
  render() {
    return this.props.children(this.state);
  }
}

// Ishlatish
<DataProvider>
  {({ data, loading }) => (
    loading ? <Spinner /> : <UserList data={data} />
  )}
</DataProvider>
```

**Amaliy misol - Dropdown:**
```javascript
class Dropdown extends React.Component {
  state = { isOpen: false };
  
  toggle = () => {
    this.setState(prev => ({ isOpen: !prev.isOpen }));
  };
  
  close = () => {
    this.setState({ isOpen: false });
  };
  
  render() {
    return this.props.render({
      isOpen: this.state.isOpen,
      toggle: this.toggle,
      close: this.close
    });
  }
}

// Ishlatish
<Dropdown
  render={({ isOpen, toggle, close }) => (
    <div>
      <button onClick={toggle}>Menyu</button>
      {isOpen && (
        <ul>
          <li onClick={close}>Home</li>
          <li onClick={close}>Profile</li>
          <li onClick={close}>Settings</li>
        </ul>
      )}
    </div>
  )}
/>
```

**Modern Hook yondashuvi:**
```javascript
// Render Props o'rniga custom hook
function useMousePosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMove);
    return () => window.removeEventListener('mousemove', handleMove);
  }, []);
  
  return position;
}

// Ishlatish
function Component() {
  const { x, y } = useMousePosition();
  return <h1>Pozitsiya: ({x}, {y})</h1>;
}
```

---

## 4. Code Splitting va Lazy Loading

**Savol:** React da code splitting qanday implement qilinadi va qanday foydalar beradi?

**Javob:**

**1. React.lazy va Suspense:**
```javascript
import React, { Suspense, lazy } from 'react';

// Lazy loading
const Dashboard = lazy(() => import('./Dashboard'));
const Profile = lazy(() => import('./Profile'));
const Settings = lazy(() => import('./Settings'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Yuklanmoqda...</div>}>
        <Routes>
          <Route path="/dashboard" element={<Dashboard />} />
          <Route path="/profile" element={<Profile />} />
          <Route path="/settings" element={<Settings />} />
        </Routes>
      </Suspense>
    </div>
  );
}
```

**2. Route-based code splitting:**
```javascript
import { BrowserRouter, Routes, Route } from 'react-router-dom';

const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Contact = lazy(() => import('./pages/Contact'));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**3. Component-based splitting:**
```javascript
// Heavy component ni lazy loading qilish
const HeavyChart = lazy(() => import('./HeavyChart'));

function Dashboard() {
  const [showChart, setShowChart] = useState(false);
  
  return (
    <div>
      <button onClick={() => setShowChart(true)}>
        Grafikni ko'rsatish
      </button>
      
      {showChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

**4. Named exports bilan:**
```javascript
// components.js
export const ComponentA = () => <div>A</div>;
export const ComponentB = () => <div>B</div>;

// App.js
const ComponentA = lazy(() => 
  import('./components').then(module => ({ 
    default: module.ComponentA 
  }))
);
```

**5. Error Boundary bilan:**
```javascript
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  
  static getDerivedStateFromError(error) {
    return { hasError: true };
  }
  
  componentDidCatch(error, errorInfo) {
    console.log('Lazy loading error:', error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      return <h1>Xatolik yuz berdi. Qaytadan yuklang.</h1>;
    }
    
    return this.props.children;
  }
}

function App() {
  return (
    <ErrorBoundary>
      <Suspense fallback={<Loading />}>
        <LazyComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

**6. Preloading:**
```javascript
const OtherComponent = lazy(() => import('./OtherComponent'));

function MyComponent() {
  // Hover da preload qilish
  const handleMouseEnter = () => {
    // Webpack prefetch
    import(/* webpackPrefetch: true */ './OtherComponent');
  };
  
  return (
    <div>
      <button onMouseEnter={handleMouseEnter}>
        Hover qiling
      </button>
      <Suspense fallback={<div>Loading...</div>}>
        <OtherComponent />
      </Suspense>
    </div>
  );
}
```

**Foydalar:**
- Initial bundle size kichikroq
- Tez initial load
- Faqat kerakli kod yuklanadi
- Better user experience

---

## 5. React Context Advanced

**Savol:** React Context ni optimization qilish va to'g'ri ishlatish usullari.

**Javob:**

**1. Multiple Contexts:**
```javascript
// Auth Context
const AuthContext = createContext();

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    checkAuth().then(user => {
      setUser(user);
      setLoading(false);
    });
  }, []);
  
  const login = async (credentials) => {
    const user = await authService.login(credentials);
    setUser(user);
  };
  
  const logout = () => {
    authService.logout();
    setUser(null);
  };
  
  const value = { user, loading, login, logout };
  
  return (
    <AuthContext.Provider value={value}>
      {children}
    </AuthContext.Provider>
  );
}

// Theme Context
const ThemeContext = createContext();

function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Combine
function App() {
  return (
    <AuthProvider>
      <ThemeProvider>
        <YourApp />
      </ThemeProvider>
    </AuthProvider>
  );
}
```

**2. Context Split (Performance Optimization):**
```javascript
// ❌ NOTO'G'RI - Har safar barcha consumer lar re-render
const AppContext = createContext();

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);
  
  return (
    <AppContext.Provider value={{ 
      user, setUser, 
      theme, setTheme,
      notifications, setNotifications 
    }}>
      {children}
    </AppContext.Provider>
  );
}

// ✅ TO'G'RI - Alohida contextlar
const UserContext = createContext();
const ThemeContext = createContext();
const NotificationContext = createContext();

function Providers({ children }) {
  return (
    <UserProvider>
      <ThemeProvider>
        <NotificationProvider>
          {children}
        </NotificationProvider>
      </ThemeProvider>
    </UserProvider>
  );
}
```

**3. Context Selector Pattern:**
```javascript
// State va dispatch ni ajratish
const StateContext = createContext();
const DispatchContext = createContext();

function TodoProvider({ children }) {
  const [todos, dispatch] = useReducer(todoReducer, []);
  
  return (
    <StateContext.Provider value={todos}>
      <DispatchContext.Provider value={dispatch}>
        {children}
      </DispatchContext.Provider>
    </StateContext.Provider>
  );
}

// Custom hooks
function useTodoState() {
  const context = useContext(StateContext);
  if (!context) {
    throw new Error('useTodoState must be used within TodoProvider');
  }
  return context;
}

function useTodoDispatch() {
  const context = useContext(DispatchContext);
  if (!context) {
    throw new Error('useTodoDispatch must be used within TodoProvider');
  }
  return context;
}

// Ishlatish
function TodoList() {
  const todos = useTodoState(); // Faqat state o'zgarsa re-render
  return <ul>{todos.map(todo => <li key={todo.id}>{todo.text}</li>)}</ul>;
}

function AddTodo() {
  const dispatch = useTodoDispatch(); // dispatch o'zgarmaydi
  // State o'zgarsa bu component re-render bo'lmaydi
  
  return (
    <button onClick={() => dispatch({ type: 'ADD', text: 'New' })}>
      Add
    </button>
  );
}
```

**4. Memoized Context Value:**
```javascript
function UserProvider({ children }) {
  const [user, setUser] = useState(null);
  const [preferences, setPreferences] = useState({});
  
  // ❌ NOTO'G'RI - Har render da yangi object
  return (
    <UserContext.Provider value={{ user, setUser, preferences, setPreferences }}>
      {children}
    </UserContext.Provider>
  );
  
  // ✅ TO'G'RI - useMemo ishlatish
  const value = useMemo(
    () => ({ user, setUser, preferences, setPreferences }),
    [user, preferences]
  );
  
  return (
    <UserContext.Provider value={value}>
      {children}
    </UserContext.Provider>
  );
}
```

**5. Context Consumer Pattern:**
```javascript
// Zamonaviy usul
function Profile() {
  const { user } = useContext(UserContext);
  return <div>{user.name}</div>;
}

// Eski usul (hali ham foydali ba'zan)
function Profile() {
  return (
    <UserContext.Consumer>
      {({ user }) => (
        <div>{user.name}</div>
      )}
    </UserContext.Consumer>
  );
}
```

---

## 6. Custom Hooks Advanced

**Savol:** Murakkab custom hooklar yaratish va optimization.

**Javob:**

**1. useAsync - API calls uchun:**
```javascript
function useAsync(asyncFunction, immediate = true) {
  const [status, setStatus] = useState('idle');
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  
  const execute = useCallback(async (...params) => {
    setStatus('pending');
    setData(null);
    setError(null);
    
    try {
      const response = await asyncFunction(...params);
      setData(response);
      setStatus('success');
      return response;
    } catch (error) {
      setError(error);
      setStatus('error');
      throw error;
    }
  }, [asyncFunction]);
  
  useEffect(() => {
    if (immediate) {
      execute();
    }
  }, [execute, immediate]);
  
  return { execute, status, data, error };
}

// Ishlatish
function UserProfile({ userId }) {
  const { data: user, status, error } = useAsync(
    () => fetchUser(userId),
    true
  );
  
  if (status === 'pending') return <Spinner />;
  if (status === 'error') return <Error message={error.message} />;
  if (status === 'success') return <UserCard user={user} />;
}
```

**2. useLocalStorage - State va localStorage sync:**
```javascript
function useLocalStorage(key, initialValue) {
  // State initialization
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  // Set value
  const setValue = (value) => {
    try {
      // Function yoki value bo'lishi mumkin
      const valueToStore = 
        value instanceof Function ? value(storedValue) : value;
      
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [storedValue, setValue];
}

// Ishlatish
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [language, setLanguage] = useLocalStorage('language', 'uz');
  
  return (
    <div>
      <select value={theme} onChange={e => setTheme(e.target.value)}>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
      </select>
    </div>
  );
}
```

**3. useDebounce - Input optimization:**
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

// Ishlatish
function SearchComponent() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      // API call
      searchAPI(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <input
      value={searchTerm}
      onChange={e => setSearchTerm(e.target.value)}
      placeholder="Qidirish..."
    />
  );
}
```

**4. useIntersectionObserver - Lazy loading:**
```javascript
function useIntersectionObserver(ref, options = {}) {
  const [isIntersecting, setIsIntersecting] = useState(false);
  
  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      setIsIntersecting(entry.isIntersecting);
    }, options);
    
    if (ref.current) {
      observer.observe(ref.current);
    }
    
    return () => {
      observer.disconnect();
    };
  }, [ref, options]);
  
  return isIntersecting;
}

// Ishlatish
function LazyImage({ src, alt }) {
  const imgRef = useRef();
  const isVisible = useIntersectionObserver(imgRef, {
    threshold: 0.1,
    rootMargin: '100px'
  });
  
  return (
    <img
      ref={imgRef}
      src={isVisible ? src : placeholder}
      alt={alt}
    />
  );
}
```

**5. usePrevious - Oldingi qiymatni saqlash:**
```javascript
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

// Ishlatish
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Hozirgi: {count}</p>
      <p>Oldingi: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  );
}
```

**6. useWindowSize - Responsive:**
```javascript
function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    // Throttle
    let timeoutId;
    const throttledResize = () => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(handleResize, 150);
    };
    
    window.addEventListener('resize', throttledResize);
    return () => window.removeEventListener('resize', throttledResize);
  }, []);
  
  return windowSize;
}

// Ishlatish
function ResponsiveComponent() {
  const { width } = useWindowSize();
  
  return (
    <div>
      {width < 768 ? <MobileView /> : <DesktopView />}
    </div>
  );
}
```

---

## 7. React Performance Patterns

**Savol:** React da performance optimization uchun qanday patternlar mavjud?

**Javob:**

**1. Virtualization (React Window):**
```javascript
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  );
  
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {Row}
    </FixedSizeList>
  );
}

// 10,000 ta element - faqat ko'ringanlar render qilinadi
```

**2. Memoization Strategy:**
```javascript
// Component memoization
const ExpensiveComponent = React.memo(
  ({ data, onUpdate }) => {
    // Complex rendering logic
    return <div>{/* ... */}</div>;
  },
  // Custom comparison
  (prevProps, nextProps) => {
    return prevProps.data.id === nextProps.data.id;
  }
);

// Callback memoization
function Parent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);
  
  // ❌ Har render da yangi funksiya
  const handleClick = () => {
    console.log('clicked');
  };
  
  // ✅ Memoized callback
  const handleClickMemo = useCallback(() => {
    console.log('clicked');
  }, []); // Dependencies
  
  // ✅ Memoized value
  const sortedItems = useMemo(() => {
    return items.sort((a, b) => a.value - b.value);
  }, [items]);
  
  return (
    <div>
      <ExpensiveComponent onClick={handleClickMemo} />
    </div>
  );
}
```

**3. Code Splitting Pattern:**
```javascript
// Feature-based splitting
const Dashboard = lazy(() => import('./features/Dashboard'));
const Analytics = lazy(() => import('./features/Analytics'));
const Reports = lazy(() => import('./features/Reports'));

// Component-based splitting
function DashboardPage() {
  const [showHeavyChart, setShowHeavyChart] = useState(false);
  
  const HeavyChart = lazy(() => import('./HeavyChart'));
  
  return (
    <div>
      <button onClick={() => setShowHeavyChart(true)}>
        Show Chart
      </button>
      
      {showHeavyChart && (
        <Suspense fallback={<ChartSkeleton />}>
          <HeavyChart />
        </Suspense>
      )}
    </div>
  );
}
```

**4. Prevent Unnecessary Re-renders:**
```javascript
// Children as props pattern
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      
      {/* ExpensiveChild re-render bo'lmaydi */}
      <Wrapper>
        <ExpensiveChild />
      </Wrapper>
    </div>
  );
}

function Wrapper({ children }) {
  // Wrapper re-render bo'lsa ham, children o'zgarmaydi
  return <div className="wrapper">{children}</div>;
}
```

**5. Batch State Updates:**
```javascript
// React 18 - Automatic batching
function handleClick() {
  setCount(c => c + 1);
  setFlag(f => !f);
  // Faqat bitta re-render
}

// React 17
