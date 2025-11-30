# JavaScript Middle/Senior Dasturchilar uchun Interview Savollari va Javoblari

## 1. Event Loop va Asynchronous JavaScript

**Savol:** JavaScript da Event Loop qanday ishlaydi? Call Stack, Task Queue va Microtask Queue orasidagi farqni tushuntiring.

**Javob:**

**Event Loop jarayoni:**

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

console.log('4');

// Natija: 1, 4, 3, 2
```

**Komponentlar:**

1. **Call Stack:** Funksiyalar ketma-ketligi
2. **Web APIs:** setTimeout, fetch, DOM events
3. **Task Queue (Macrotask):** setTimeout, setInterval callbacks
4. **Microtask Queue:** Promises, queueMicrotask
5. **Event Loop:** Queue lardan stackga o'tkazadi

**Bajarilish tartibi:**
1. Call Stack bo'shagunga qadar barcha sync kod
2. Microtask Queue (Promises)
3. Macrotask Queue (setTimeout)
4. Render (keyin yana Microtasks)

**Amaliy misol:**
```javascript
async function test() {
  console.log('A');
  
  setTimeout(() => console.log('B'), 0);
  
  await Promise.resolve();
  console.log('C');
  
  setTimeout(() => console.log('D'), 0);
}

test();
console.log('E');

// Natija: A, E, C, B, D
```

## 2. Closures va Lexical Scope

**Savol:** Closure nima va qanday ishlaydi? Amaliy qo'llanilishiga misollar keltiring.

**Javob:**

**Closure:** Funksiya o'z scope idagi o'zgaruvchilarga kirish huquqini saqlab qoladi.

```javascript
function createCounter() {
  let count = 0;
  
  return {
    increment() {
      count++;
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    getCount() {
      return count;
    }
  };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount()); // 2
```

**Amaliy qo'llanilishi:**

1. **Private o'zgaruvchilar:**
```javascript
function bankAccount(initialBalance) {
  let balance = initialBalance; // private
  
  return {
    deposit(amount) {
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (amount <= balance) {
        balance -= amount;
        return balance;
      }
      throw new Error('Mablag yetarli emas');
    }
  };
}
```

2. **Function Factory:**
```javascript
function multiplyBy(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = multiplyBy(2);
const triple = multiplyBy(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

3. **Memoization:**
```javascript
function memoize(fn) {
  const cache = {};
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (key in cache) {
      return cache[key];
    }
    
    const result = fn.apply(this, args);
    cache[key] = result;
    return result;
  };
}

const factorial = memoize((n) => {
  return n <= 1 ? 1 : n * factorial(n - 1);
});
```

## 3. Prototypal Inheritance

**Savol:** JavaScript da prototype inheritance qanday ishlaydi? `__proto__`, `prototype` va `Object.create()` ni tushuntiring.

**Javob:**

**Prototype Chain:**

```javascript
function Animal(name) {
  this.name = name;
}

Animal.prototype.speak = function() {
  console.log(`${this.name} ovoz chiqaradi`);
};

function Dog(name, breed) {
  Animal.call(this, name);
  this.breed = breed;
}

// Inheritance o'rnatish
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

Dog.prototype.bark = function() {
  console.log(`${this.name} vov-vov deydi`);
};

const dog = new Dog('Rex', 'Labrador');
dog.speak(); // Rex ovoz chiqaradi
dog.bark();  // Rex vov-vov deydi
```

**Tushuncha:**

- **`prototype`:** Constructor funksiyaning xususiyati
- **`__proto__`:** Obyektning prototypeiga havola
- **`Object.create()`:** Yangi obyekt yaratib, prototypeini o'rnatadi

**Modern ES6 class syntax:**
```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    console.log(`${this.name} ovoz chiqaradi`);
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }
  
  bark() {
    console.log(`${this.name} vov-vov deydi`);
  }
}
```

**Prototype Chain tekshirish:**
```javascript
console.log(dog instanceof Dog);     // true
console.log(dog instanceof Animal);  // true
console.log(Dog.prototype.isPrototypeOf(dog)); // true
```

## 4. this Keyword

**Savol:** JavaScript da `this` qanday ishlaydi? Turli kontekstlarda qanday o'zgaradi?

**Javob:**

**this ning 4 ta binding qoidasi:**

1. **Default Binding:**
```javascript
function show() {
  console.log(this); // strict mode: undefined, non-strict: window
}
show();
```

2. **Implicit Binding:**
```javascript
const obj = {
  name: 'Ali',
  greet() {
    console.log(this.name); // Ali
  }
};
obj.greet();
```

3. **Explicit Binding:**
```javascript
function greet() {
  console.log(`Salom, ${this.name}`);
}

const person = { name: 'Vali' };

greet.call(person);  // Salom, Vali
greet.apply(person); // Salom, Vali

const boundGreet = greet.bind(person);
boundGreet(); // Salom, Vali
```

4. **new Binding:**
```javascript
function Person(name) {
  this.name = name;
}

const person = new Person('Jamol');
console.log(person.name); // Jamol
```

**Arrow Functions:**
```javascript
const obj = {
  name: 'Test',
  regularFunc() {
    console.log(this.name); // Test
  },
  arrowFunc: () => {
    console.log(this.name); // undefined (lexical this)
  }
};
```

**Keng tarqalgan xato:**
```javascript
const obj = {
  name: 'Ali',
  delays: [1000, 2000],
  
  // ❌ Noto'g'ri
  start() {
    this.delays.forEach(function(delay) {
      setTimeout(function() {
        console.log(this.name); // undefined
      }, delay);
    });
  },
  
  // ✅ To'g'ri - Arrow function
  startCorrect() {
    this.delays.forEach(delay => {
      setTimeout(() => {
        console.log(this.name); // Ali
      }, delay);
    });
  }
};
```

## 5. Promises va Async/Await

**Savol:** Promise chaining, error handling va async/await ni qanday to'g'ri ishlatish kerak?

**Javob:**

**Promise states:**
- Pending (kutilmoqda)
- Fulfilled (muvaffaqiyatli)
- Rejected (xato)

**Promise chaining:**
```javascript
fetch('/api/user')
  .then(response => response.json())
  .then(user => fetch(`/api/posts/${user.id}`))
  .then(response => response.json())
  .then(posts => console.log(posts))
  .catch(error => console.error(error))
  .finally(() => console.log('Tugadi'));
```

**Async/Await:**
```javascript
async function getUserPosts() {
  try {
    const userResponse = await fetch('/api/user');
    const user = await userResponse.json();
    
    const postsResponse = await fetch(`/api/posts/${user.id}`);
    const posts = await postsResponse.json();
    
    return posts;
  } catch (error) {
    console.error('Xato:', error);
    throw error;
  } finally {
    console.log('Tugadi');
  }
}
```

**Parallel execution:**
```javascript
// ❌ Ketma-ket (sekin)
const user = await fetchUser();
const posts = await fetchPosts();
const comments = await fetchComments();

// ✅ Parallel (tez)
const [user, posts, comments] = await Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
]);

// Promise.allSettled - hamma natijalarni olish
const results = await Promise.allSettled([
  fetchUser(),
  fetchPosts(),
  fetchComments()
]);

// Promise.race - birinchi tugagan
const fastest = await Promise.race([
  fetch('/api/server1'),
  fetch('/api/server2')
]);
```

**Custom Promise:**
```javascript
function delay(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}

await delay(1000);
console.log('1 soniyadan keyin');
```

## 6. Functional Programming Concepts

**Savol:** JavaScript da functional programming asoslari: Pure functions, Higher-order functions, Immutability.

**Javob:**

**1. Pure Functions:**
```javascript
// ❌ Impure - tashqi o'zgaruvchini o'zgartiradi
let count = 0;
function increment() {
  count++;
  return count;
}

// ✅ Pure - side effect yo'q
function increment(num) {
  return num + 1;
}
```

**2. Higher-Order Functions:**
```javascript
// Funksiyani qaytaradi
function multiplier(factor) {
  return num => num * factor;
}

// Funksiyani argument sifatida oladi
function operate(a, b, operation) {
  return operation(a, b);
}

const add = (a, b) => a + b;
console.log(operate(5, 3, add)); // 8
```

**3. Immutability:**
```javascript
// ❌ Mutable
const arr = [1, 2, 3];
arr.push(4); // Original arrayni o'zgartiradi

// ✅ Immutable
const arr2 = [1, 2, 3];
const newArr = [...arr2, 4]; // Yangi array

// Object bilan
const user = { name: 'Ali', age: 25 };
const updatedUser = { ...user, age: 26 };
```

**4. Function Composition:**
```javascript
const compose = (...fns) => x => 
  fns.reduceRight((acc, fn) => fn(acc), x);

const addOne = x => x + 1;
const double = x => x * 2;
const square = x => x * x;

const calculate = compose(square, double, addOne);
console.log(calculate(3)); // (3 + 1) * 2 = 8, 8² = 64
```

**5. Currying:**
```javascript
// Oddiy funksiya
function add(a, b, c) {
  return a + b + c;
}

// Curried
const curriedAdd = a => b => c => a + b + c;

console.log(curriedAdd(1)(2)(3)); // 6

const add5 = curriedAdd(5);
const add5and10 = add5(10);
console.log(add5and10(2)); // 17
```

## 7. Memory Management va Garbage Collection

**Savol:** JavaScript da memory management qanday ishlaydi? Memory leak larning sabablari va oldini olish usullari.

**Javob:**

**Garbage Collection:**
JavaScript avtomatik memory management ishlatadi (Mark-and-Sweep algoritmi).

**Memory Leak sabablari:**

1. **Global o'zgaruvchilar:**
```javascript
// ❌ Memory leak
function createLeak() {
  leak = 'Bu global o'zgaruvchi'; // var/let/const yo'q
}
```

2. **Event Listener lar:**
```javascript
// ❌ Tozalanmagan listener
const element = document.getElementById('btn');
element.addEventListener('click', onClick);

// ✅ To'g'ri
element.removeEventListener('click', onClick);
```

3. **Timer lar:**
```javascript
// ❌ Bekor qilinmagan timer
const interval = setInterval(() => {
  console.log('Running...');
}, 1000);

// ✅ To'g'ri
clearInterval(interval);
```

4. **Closure da keraksiz referencelar:**
```javascript
// ❌ Memory leak
function outer() {
  const bigData = new Array(1000000);
  
  return function inner() {
    // bigData ishlatilmasa ham xotirada qoladi
    console.log('Hello');
  };
}

// ✅ To'g'ri
function outer() {
  const bigData = new Array(1000000);
  const smallData = bigData[0];
  
  return function inner() {
    console.log(smallData);
  };
}
```

5. **DOM references:**
```javascript
const elements = [];

function addElement() {
  const div = document.createElement('div');
  document.body.appendChild(div);
  elements.push(div); // Reference saqlanib qoladi
}

// DOM dan olib tashlangandan keyin ham xotirada
document.body.innerHTML = '';
// elements hali ham referencelarni saqlaydi
```

**Memory profiling:**
```javascript
// Chrome DevTools Memory Profiler ishlatish
// Performance Monitor
// Heap Snapshot
```

## 8. Design Patterns

**Savol:** JavaScript da eng muhim design patterns larni tushuntiring.

**Javob:**

**1. Module Pattern:**
```javascript
const Module = (function() {
  // Private
  let privateVar = 'private';
  
  function privateMethod() {
    console.log(privateVar);
  }
  
  // Public
  return {
    publicVar: 'public',
    publicMethod() {
      privateMethod();
    }
  };
})();
```

**2. Singleton Pattern:**
```javascript
const Singleton = (function() {
  let instance;
  
  function createInstance() {
    return {
      name: 'Singleton',
      getData() {
        return this.name;
      }
    };
  }
  
  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

const instance1 = Singleton.getInstance();
const instance2 = Singleton.getInstance();
console.log(instance1 === instance2); // true
```

**3. Observer Pattern:**
```javascript
class EventEmitter {
  constructor() {
    this.events = {};
  }
  
  on(event, listener) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(listener);
  }
  
  emit(event, ...args) {
    if (this.events[event]) {
      this.events[event].forEach(listener => {
        listener(...args);
      });
    }
  }
  
  off(event, listenerToRemove) {
    if (this.events[event]) {
      this.events[event] = this.events[event].filter(
        listener => listener !== listenerToRemove
      );
    }
  }
}

const emitter = new EventEmitter();
emitter.on('login', user => console.log(`${user} logged in`));
emitter.emit('login', 'Ali');
```

**4. Factory Pattern:**
```javascript
class Car {
  constructor(options) {
    this.doors = options.doors || 4;
    this.color = options.color || 'silver';
  }
}

class Truck {
  constructor(options) {
    this.wheels = options.wheels || 6;
    this.color = options.color || 'red';
  }
}

class VehicleFactory {
  createVehicle(type, options) {
    switch(type) {
      case 'car':
        return new Car(options);
      case 'truck':
        return new Truck(options);
      default:
        throw new Error('Unknown vehicle type');
    }
  }
}

const factory = new VehicleFactory();
const car = factory.createVehicle('car', { color: 'blue' });
```

**5. Decorator Pattern:**
```javascript
function readonly(target, key, descriptor) {
  descriptor.writable = false;
  return descriptor;
}

class Person {
  @readonly
  name = 'Ali';
}

// Yoki funksiya bilan
function logExecutionTime(fn) {
  return function(...args) {
    console.time('Execution');
    const result = fn.apply(this, args);
    console.timeEnd('Execution');
    return result;
  };
}
```

## 9. ES6+ Features

**Savol:** ES6 dan keyingi eng muhim yangiliklar va ularning qo'llanilishi.

**Javob:**

**1. Destructuring:**
```javascript
// Array
const [first, second, ...rest] = [1, 2, 3, 4, 5];

// Object
const { name, age, ...others } = user;

// Nested
const { address: { city } } = user;

// Default values
const { status = 'active' } = user;

// Function parameters
function greet({ name, age = 25 }) {
  console.log(`${name} - ${age} yosh`);
}
```

**2. Spread/Rest:**
```javascript
// Array spread
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];

// Object spread
const obj1 = { a: 1, b: 2 };
const obj2 = { ...obj1, c: 3 };

// Rest parameters
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
```

**3. Optional Chaining:**
```javascript
const user = {
  address: {
    city: 'Tashkent'
  }
};

// Eski usul
const city = user && user.address && user.address.city;

// Yangi usul
const city2 = user?.address?.city;
const method = obj?.someMethod?.();
const item = arr?.[index];
```

**4. Nullish Coalescing:**
```javascript
// || vs ??
const value1 = 0 || 'default'; // 'default'
const value2 = 0 ?? 'default'; // 0

const name = null ?? 'Guest'; // 'Guest'
const name2 = undefined ?? 'Guest'; // 'Guest'
```

**5. Private Fields:**
```javascript
class BankAccount {
  #balance = 0; // private
  
  deposit(amount) {
    this.#balance += amount;
  }
  
  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount();
account.deposit(100);
console.log(account.getBalance()); // 100
// console.log(account.#balance); // SyntaxError
```

**6. Dynamic Import:**
```javascript
async function loadModule() {
  const module = await import('./module.js');
  module.doSomething();
}

// Conditional import
if (condition) {
  const { feature } = await import('./feature.js');
}
```

## 10. Error Handling

**Savol:** JavaScript da error handling ning best practice lari.

**Javob:**

**1. Try-Catch:**
```javascript
try {
  const data = JSON.parse(jsonString);
  processData(data);
} catch (error) {
  console.error('Parsing xatosi:', error.message);
} finally {
  cleanup();
}
```

**2. Custom Error:**
```javascript
class ValidationError extends Error {
  constructor(message) {
    super(message);
    this.name = 'ValidationError';
  }
}

function validateUser(user) {
  if (!user.email) {
    throw new ValidationError('Email majburiy');
  }
}

try {
  validateUser({});
} catch (error) {
  if (error instanceof ValidationError) {
    console.log('Validation xatosi:', error.message);
  } else {
    console.log('Boshqa xato:', error);
  }
}
```

**3. Async Error Handling:**
```javascript
// Promise
fetch('/api/data')
  .then(response => response.json())
  .catch(error => {
    console.error('Fetch xatosi:', error);
  });

// Async/Await
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return await response.json();
  } catch (error) {
    console.error('Xato:', error);
    throw error; // Re-throw if needed
  }
}
```

**4. Global Error Handler:**
```javascript
// Synchronous errors
window.onerror = function(message, source, lineno, colno, error) {
  console.error('Global error:', error);
  return true; // Prevent default
};

// Unhandled promise rejections
window.addEventListener('unhandledrejection', event => {
  console.error('Unhandled rejection:', event.reason);
  event.preventDefault();
});
```

**5. Error Recovery:**
```javascript
async function fetchWithRetry(url, retries = 3) {
  for (let i = 0; i < retries; i++) {
    try {
      const response = await fetch(url);
      return await response.json();
    } catch (error) {
      if (i === retries - 1) throw error;
      await delay(1000 * Math.pow(2, i)); // Exponential backoff
    }
  }
}
```

## 11. Performance Optimization

**Savol:** JavaScript kodning performance ini yaxshilash usullari.

**Javob:**

**1. Debouncing:**
```javascript
function debounce(func, delay) {
  let timeoutId;
  
  return function(...args) {
    clearTimeout(timeoutId);
    
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

const searchInput = document.getElementById('search');
const debouncedSearch = debounce(search, 300);
searchInput.addEventListener('input', debouncedSearch);
```

**2. Throttling:**
```javascript
function throttle(func, limit) {
  let inThrottle;
  
  return function(...args) {
    if (!inThrottle) {
      func.apply(this, args);
      inThrottle = true;
      
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

window.addEventListener('scroll', throttle(handleScroll, 100));
```

**3. Memoization:**
```javascript
function memoize(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const expensiveOperation = memoize((n) => {
  // Qimmat hisoblash
  return n * n;
});
```

**4. Lazy Loading:**
```javascript
// Intersection Observer
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      observer.unobserve(img);
    }
  });
});

document.querySelectorAll('img[data-src]').forEach(img => {
  observer.observe(img);
});
```

**5. Web Workers:**
```javascript
// main.js
const worker = new Worker('worker.js');

worker.postMessage({ data: largeArray });

worker.onmessage = (event) => {
  console.log('Result:', event.data);
};

// worker.js
self.onmessage = (event) => {
  const result = heavyComputation(event.data);
  self.postMessage(result);
};
```

## 12. Security Best Practices

**Savol:** JavaScript da security muammolari va ularning yechimlari.

**Javob:**

**1. XSS (Cross-Site Scripting):**
```javascript
// ❌ Xavfli
element.innerHTML = userInput;

// ✅ Xavfsiz
element.textContent = userInput;

// yoki sanitize qilish
function escapeHtml(text) {
  const div = document.createElement('div');
  div.textContent = text;
  return div.innerHTML;
}
```

**2. CSRF Protection:**
```javascript
// CSRF token qo'shish
fetch('/api/data', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'X-CSRF-Token': getCsrfToken()
  },
  body: JSON.stringify(data)
});
```

**3. Content Security Policy:**
```javascript
// HTTP header yoki meta tag
<meta http-equiv="Content-Security-Policy" 
      content="default-src 'self'; script-src 'self' 'unsafe-inline'">
```

**4. Input Validation:**
```javascript
function validateEmail(email) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

function sanitizeInput(input) {
  return input
    .trim()
    .replace(/[<>]/g, '')
    .slice(0, 100); // max length
}
```

**5. Secure Storage:**
```javascript
// ❌ Sensitive data localStorage da
localStorage.setItem('password', password);

// ✅ Secure cookie yoki memory da
// HTTPOnly, Secure, SameSite cookies
document.cookie = "token=abc123; Secure; HttpOnly; SameSite=Strict";
```

## Xulosa

Ushbu savollar JavaScript middle va senior dasturchilar uchun eng muhim mavzularni qamrab oladi. Har bir mavzu chuqur tushunilishi va amaliy qo'llanilishi interview jarayonida katta ustunlik beradi.

**Qo'shimcha o'rganish uchun mavzular:**
- TypeScript
- Testing (Jest, Mocha)
- Build tools (Webpack, Vite)
- Node.js fundamentals
- Browser APIs (Storage, Fetch, WebSockets)
- Module systems (CommonJS, ES Modules)
- 
# Middle/Senior JavaScript Developer Interview Savollari

## 1. Higher-Order Functions va Callbacks

### Savol: Higher-order function nima va misol keltiring?
**Javob:**
Higher-order function - bu boshqa funksiyani argument sifatida qabul qiladigan yoki funksiyani qaytaradigan funksiya.

```javascript
// Function argument sifatida
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(num => num * 2);

// Function qaytarish
function multiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = multiplier(2);
const triple = multiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

**Amaliy foydalanish:**
- Array methods (map, filter, reduce)
- Event handlers
- Middleware patterns
- Function composition

---

## 2. Currying va Partial Application

### Savol: Currying nima va qanday ishlaydi?
**Javob:**
Currying - ko'p parametrli funksiyani bir parametrli funksiyalar ketma-ketligiga aylantirish.

```javascript
// Oddiy funksiya
function add(a, b, c) {
  return a + b + c;
}

// Curried versiya
function curriedAdd(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

console.log(curriedAdd(1)(2)(3)); // 6

// ES6 arrow functions bilan
const curriedAdd2 = a => b => c => a + b + c;

// Partial application
const add5 = curriedAdd(5);
const add5And3 = add5(3);
console.log(add5And3(2)); // 10
```

**Amaliy misol:**
```javascript
// Logger function
const log = level => message => time => {
  console.log(`[${level}] ${time}: ${message}`);
};

const errorLog = log('ERROR');
const errorLogNow = errorLog(new Date().toISOString());
errorLogNow('Database connection failed');
```

---

## 3. Module Pattern va Encapsulation

### Savol: JavaScript da private o'zgaruvchilarni qanday yaratish mumkin?
**Javob:**
```javascript
// IIFE bilan Module Pattern
const BankAccount = (function() {
  // Private o'zgaruvchilar
  let balance = 0;
  const transactions = [];
  
  // Private funksiya
  function recordTransaction(type, amount) {
    transactions.push({
      type,
      amount,
      date: new Date(),
      balance: balance
    });
  }
  
  // Public API
  return {
    deposit(amount) {
      if (amount > 0) {
        balance += amount;
        recordTransaction('deposit', amount);
        return balance;
      }
    },
    
    withdraw(amount) {
      if (amount > 0 && amount <= balance) {
        balance -= amount;
        recordTransaction('withdraw', amount);
        return balance;
      }
      throw new Error('Insufficient funds');
    },
    
    getBalance() {
      return balance;
    },
    
    getTransactionHistory() {
      return [...transactions]; // Copy qaytarish
    }
  };
})();

// ES6 WeakMap bilan
const BankAccount2 = (function() {
  const privateData = new WeakMap();
  
  class Account {
    constructor(initialBalance = 0) {
      privateData.set(this, {
        balance: initialBalance,
        transactions: []
      });
    }
    
    deposit(amount) {
      const data = privateData.get(this);
      data.balance += amount;
      data.transactions.push({ type: 'deposit', amount });
      return data.balance;
    }
    
    getBalance() {
      return privateData.get(this).balance;
    }
  }
  
  return Account;
})();
```

---

## 4. Generators va Iterators

### Savol: Generator function nima va qachon ishlatiladi?
**Javob:**
Generator - bu funksiya bajarilishini to'xtatib turish va davom ettirish imkonini beradi.

```javascript
// Oddiy generator
function* numberGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = numberGenerator();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }

// Infinite generator
function* idGenerator() {
  let id = 1;
  while (true) {
    yield id++;
  }
}

const ids = idGenerator();
console.log(ids.next().value); // 1
console.log(ids.next().value); // 2

// Amaliy misol: Pagination
function* paginate(array, pageSize) {
  for (let i = 0; i < array.length; i += pageSize) {
    yield array.slice(i, i + pageSize);
  }
}

const data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const pages = paginate(data, 3);

for (const page of pages) {
  console.log(page); // [1,2,3], [4,5,6], [7,8,9], [10]
}

// Async generator
async function* fetchUsers(userIds) {
  for (const id of userIds) {
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    yield user;
  }
}

// Foydalanish
for await (const user of fetchUsers([1, 2, 3])) {
  console.log(user);
}
```

---

## 5. Symbols va Well-Known Symbols

### Savol: Symbol nima va qachon ishlatiladi?
**Javob:**
Symbol - bu unique va immutable primitive data type.

```javascript
// Oddiy symbol
const id = Symbol('id');
const id2 = Symbol('id');
console.log(id === id2); // false

// Object property sifatida
const user = {
  name: 'John',
  [id]: 123 // Symbol property
};

console.log(user[id]); // 123
console.log(Object.keys(user)); // ['name'] - Symbol ko'rinmaydi

// Well-known symbols
const myArray = {
  data: [1, 2, 3, 4, 5],
  [Symbol.iterator]() {
    let index = 0;
    return {
      next: () => {
        if (index < this.data.length) {
          return { value: this.data[index++], done: false };
        }
        return { done: true };
      }
    };
  }
};

for (const item of myArray) {
  console.log(item); // 1, 2, 3, 4, 5
}

// Symbol.toStringTag
class MyClass {
  get [Symbol.toStringTag]() {
    return 'MyClass';
  }
}

console.log(Object.prototype.toString.call(new MyClass())); 
// [object MyClass]
```

---

## 6. Proxy va Reflect

### Savol: Proxy object nima va qanday ishlatiladi?
**Javob:**
Proxy - bu object operatsiyalarini to'sib olish (intercept) va qayta aniqlash imkonini beradi.

```javascript
// Validation proxy
const validator = {
  set(target, property, value) {
    if (property === 'age') {
      if (typeof value !== 'number' || value < 0 || value > 120) {
        throw new TypeError('Age must be a number between 0 and 120');
      }
    }
    target[property] = value;
    return true;
  }
};

const person = new Proxy({}, validator);
person.age = 25; // OK
// person.age = -5; // Error!

// Logging proxy
function createLoggingProxy(obj, name) {
  return new Proxy(obj, {
    get(target, property) {
      console.log(`Reading ${name}.${property}`);
      return Reflect.get(target, property);
    },
    set(target, property, value) {
      console.log(`Writing ${name}.${property} = ${value}`);
      return Reflect.set(target, property, value);
    }
  });
}

const user = createLoggingProxy({ name: 'John' }, 'user');
user.name; // Log: Reading user.name
user.age = 30; // Log: Writing user.age = 30

// Default values proxy
function withDefaults(target, defaults) {
  return new Proxy(target, {
    get(obj, prop) {
      return prop in obj ? obj[prop] : defaults[prop];
    }
  });
}

const config = withDefaults(
  { port: 3000 },
  { host: 'localhost', port: 8080 }
);

console.log(config.port); // 3000
console.log(config.host); // localhost
```

---

## 7. WeakMap va WeakSet

### Savol: WeakMap va Map orasidagi farq nima?
**Javob:**
```javascript
// Map - kuchli reference
const map = new Map();
let obj = { data: 'important' };
map.set(obj, 'value');
obj = null; // obj hali ham Map da saqlanadi (memory leak)

// WeakMap - zaif reference
const weakMap = new WeakMap();
let obj2 = { data: 'important' };
weakMap.set(obj2, 'value');
obj2 = null; // obj2 garbage collection orqali o'chiriladi

// Amaliy misol: Private data storage
const privateData = new WeakMap();

class Person {
  constructor(name, ssn) {
    this.name = name;
    privateData.set(this, { ssn }); // SSN ni yashirish
  }
  
  getSSN() {
    return privateData.get(this).ssn;
  }
}

// Cache implementation
const cache = new WeakMap();

function processData(obj) {
  if (cache.has(obj)) {
    console.log('Returning cached result');
    return cache.get(obj);
  }
  
  console.log('Computing result');
  const result = expensiveOperation(obj);
  cache.set(obj, result);
  return result;
}

// WeakSet - unique objects
const visitedNodes = new WeakSet();

function traverse(node) {
  if (visitedNodes.has(node)) return;
  
  visitedNodes.add(node);
  // Process node
  node.children?.forEach(traverse);
}
```

---

## 8. Composition vs Inheritance

### Savol: JavaScript da composition yaxshiroqmi yoki inheritance?
**Javob:**
```javascript
// Inheritance muammosi
class Animal {
  eat() { return 'eating'; }
  sleep() { return 'sleeping'; }
}

class Dog extends Animal {
  bark() { return 'barking'; }
  swim() { return 'swimming'; }
}

class Cat extends Animal {
  meow() { return 'meowing'; }
  // Cat suzolmaydi, lekin Dog swim() ga ega
}

// Composition yechimi
const canEat = (state) => ({
  eat() {
    console.log(`${state.name} is eating`);
  }
});

const canSleep = (state) => ({
  sleep() {
    console.log(`${state.name} is sleeping`);
  }
});

const canBark = (state) => ({
  bark() {
    console.log(`${state.name} is barking`);
  }
});

const canSwim = (state) => ({
  swim() {
    console.log(`${state.name} is swimming`);
  }
});

// Dog composition
const createDog = (name) => {
  const state = { name };
  return Object.assign(
    {},
    canEat(state),
    canSleep(state),
    canBark(state),
    canSwim(state)
  );
};

// Cat composition
const createCat = (name) => {
  const state = { name };
  return Object.assign(
    {},
    canEat(state),
    canSleep(state),
    { meow() { console.log(`${state.name} is meowing`); } }
  );
};

const dog = createDog('Rex');
dog.swim(); // Rex is swimming

const cat = createCat('Whiskers');
cat.eat(); // Whiskers is eating
// cat.swim(); // Yo'q!
```

---

## 9. Microtasks vs Macrotasks

### Savol: Event Loop da microtask va macrotask farqi nima?
**Javob:**
```javascript
console.log('1: Sync');

setTimeout(() => {
  console.log('2: setTimeout (macrotask)');
}, 0);

Promise.resolve().then(() => {
  console.log('3: Promise (microtask)');
});

console.log('4: Sync');

// Natija:
// 1: Sync
// 4: Sync
// 3: Promise (microtask)
// 2: setTimeout (macrotask)

// Murakkab misol
console.log('Start');

setTimeout(() => {
  console.log('setTimeout 1');
  Promise.resolve().then(() => {
    console.log('Promise in setTimeout 1');
  });
}, 0);

Promise.resolve().then(() => {
  console.log('Promise 1');
  setTimeout(() => {
    console.log('setTimeout in Promise 1');
  }, 0);
});

Promise.resolve().then(() => {
  console.log('Promise 2');
});

console.log('End');

// Natija:
// Start
// End
// Promise 1
// Promise 2
// setTimeout 1
// Promise in setTimeout 1
// setTimeout in Promise 1
```

---

## 10. Object Destructuring va Rest/Spread

### Savol: Advanced destructuring patternlarni tushuntiring?
**Javob:**
```javascript
// Nested destructuring
const user = {
  id: 1,
  name: 'John',
  address: {
    city: 'New York',
    coordinates: {
      lat: 40.7128,
      lng: -74.0060
    }
  }
};

const {
  name,
  address: {
    city,
    coordinates: { lat, lng }
  }
} = user;

// Default values
const { age = 18, country = 'USA' } = user;

// Renaming
const { name: userName, id: userId } = user;

// Rest operator
const { address, ...userWithoutAddress } = user;

// Function parameters
function displayUser({ name, age = 18, ...rest }) {
  console.log(name, age, rest);
}

// Array destructuring with rest
const [first, second, ...others] = [1, 2, 3, 4, 5];
console.log(others); // [3, 4, 5]

// Swapping variables
let a = 1, b = 2;
[a, b] = [b, a];

// Computed property names
const key = 'dynamicKey';
const { [key]: value } = { dynamicKey: 'value' };

// Amaliy misol: API response handling
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  const {
    data: {
      user: {
        firstName,
        lastName,
        email,
        profile: { avatar, bio } = {}
      } = {}
    } = {}
  } = await response.json();
  
  return { firstName, lastName, email, avatar, bio };
}
```

---

## 11. Optional Chaining va Nullish Coalescing

### Savol: ?. va ?? operatorlari qanday ishlaydi?
**Javob:**
```javascript
const user = {
  name: 'John',
  address: {
    city: 'NYC'
  }
};

// Optional chaining (?.)
console.log(user.address?.city); // NYC
console.log(user.address?.country?.code); // undefined (error emas!)
console.log(user.contact?.phone); // undefined

// Function chaining
const result = user.getAddress?.(); // function mavjud bo'lsa chaqiradi

// Array chaining
const firstItem = user.items?.[0];

// Nullish coalescing (??)
const port = config.port ?? 3000; // null yoki undefined bo'lsa 3000
const host = config.host ?? 'localhost';

// || vs ?? farqi
const value1 = 0 || 10; // 10 (0 falsy)
const value2 = 0 ?? 10; // 0 (0 null/undefined emas)

const empty = '' || 'default'; // 'default'
const empty2 = '' ?? 'default'; // ''

// Amaliy misol
function getUserCity(user) {
  return user?.address?.city ?? 'Unknown';
}

// Complex example
const config = {
  server: {
    port: process.env.PORT ?? 3000,
    host: process.env.HOST ?? 'localhost',
    timeout: process.env.TIMEOUT ?? 30000
  },
  database: {
    url: process.env.DB_URL ?? 'mongodb://localhost:27017',
    options: {
      useNewUrlParser: true,
      maxPoolSize: process.env.DB_POOL_SIZE ?? 10
    }
  }
};
```

---

## 12. BigInt va Number Precision

### Savol: JavaScript da katta sonlar bilan qanday ishlash kerak?
**Javob:**
```javascript
// Number limitations
const maxSafeInt = Number.MAX_SAFE_INTEGER; // 9007199254740991
console.log(maxSafeInt + 1); // 9007199254740992
console.log(maxSafeInt + 2); // 9007199254740992 (noto'g'ri!)

// BigInt
const bigNumber = 9007199254740991n;
const bigNumber2 = BigInt('9007199254740991');

console.log(bigNumber + 1n); // 9007199254740992n
console.log(bigNumber + 2n); // 9007199254740993n (to'g'ri!)

// BigInt operations
const a = 100n;
const b = 200n;
console.log(a + b); // 300n
console.log(a * b); // 20000n
console.log(b / a); // 2n (integer division)

// BigInt bilan Number aralashtirish mumkin emas
// console.log(100n + 50); // TypeError!
console.log(100n + BigInt(50)); // 150n
console.log(Number(100n) + 50); // 150

// Amaliy misol: Cryptocurrency calculations
function calculateSatoshis(btc) {
  const satoshisPerBTC = 100000000n;
  return BigInt(Math.floor(btc * 100000000)) * 1n;
}

// Precision masalasi
console.log(0.1 + 0.2); // 0.30000000000000004
console.log(0.1 + 0.2 === 0.3); // false

// Yechim: Decimal library yoki integer arithmetic
function addDecimals(a, b, decimals = 2) {
  const multiplier = Math.pow(10, decimals);
  return (Math.round(a * multiplier) + Math.round(b * multiplier)) / multiplier;
}

console.log(addDecimals(0.1, 0.2)); // 0.3
```

Men qo'shimcha mavzularni ham qo'shishim mumkin:
- Web Workers va Multithreading
- IndexedDB va LocalStorage bilan ishlash
- Regular Expressions (RegExp) advanced patterns
- Temporal API (yangi Date API)
- AbortController va Request cancellation

Qaysi mavzularni ko'proq o'rganishni xohlaysiz?
