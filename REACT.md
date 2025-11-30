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
