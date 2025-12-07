# FSD (Feature-Sliced Design) - To'liq qo'llanma

FSD - bu frontend loyihalarni modulli va scalable qilish uchun arxitektura metodologiyasi.

## 📐 Asosiy printsiplar

1. **Layerlar** (qatlamlar) - vertikal ajratish
2. **Slicelar** (bo'laklar) - gorizontal ajratish  
3. **Segmentlar** - slice ichidagi papkalar

## 🗂️ Layerlar (yuqoridan pastga)

```
src/
├── app/           # 1-layer: Ilovani ishga tushirish
├── processes/     # 2-layer: Biznes jarayonlar (kam ishlatiladi)
├── pages/         # 3-layer: Sahifalar
├── widgets/       # 4-layer: Katta UI bloklari
├── features/      # 5-layer: Biznes funksionallik
├── entities/      # 6-layer: Biznes entitylar
└── shared/        # 7-layer: Qayta ishlatiladigan kod
```

### ⚠️ Muhim qoida: **Import restrictions**
Har bir layer faqat o'zidan **pastdagi** layerlardan import qilishi mumkin:

```
app → pages → widgets → features → entities → shared
```

❌ **Noto'g'ri:**
```typescript
// entities dan features ga import - XATO!
import { loginUser } from '@/features/auth';
```

✅ **To'g'ri:**
```typescript
// features dan entities ga import - TO'G'RI
import { userApi } from '@/entities/user';
```

---

## 1️⃣ **APP Layer** - Ilovani ishga tushirish

Ilova konfiguratsiyasi, routing, global providerlar.

```
app/
├── providers/              # React providerlar
│   ├── StoreProvider.tsx
│   ├── RouterProvider.tsx
│   ├── ThemeProvider.tsx
│   └── index.tsx
├── store/                  # Redux store
│   ├── store.ts
│   ├── rootReducer.ts
│   └── index.ts
├── routes/                 # Routing konfiguratsiya
│   ├── index.tsx
│   └── ProtectedRoute.tsx
├── styles/                 # Global stillar
│   ├── index.css
│   └── globals.css
├── index.tsx              # Entry point
└── App.tsx
```

**providers/index.tsx:**
```typescript
import { StoreProvider } from './StoreProvider';
import { RouterProvider } from './RouterProvider';
import { ThemeProvider } from './ThemeProvider';

export const Providers = ({ children }: { children: React.ReactNode }) => {
  return (
    <StoreProvider>
      <ThemeProvider>
        <RouterProvider>
          {children}
        </RouterProvider>
      </ThemeProvider>
    </StoreProvider>
  );
};
```

**store/store.ts:**
```typescript
import { configureStore } from '@reduxjs/toolkit';
import { baseApi } from '@/shared/api/base-api';

export const store = configureStore({
  reducer: {
    [baseApi.reducerPath]: baseApi.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(baseApi.middleware),
});
```

---

## 2️⃣ **PAGES Layer** - Sahifalar

Har bir route uchun alohida sahifa.

```
pages/
├── home/
│   ├── ui/
│   │   └── HomePage.tsx
│   └── index.ts
├── vacancies/
│   ├── ui/
│   │   ├── VacanciesPage.tsx
│   │   └── VacancyDetailsPage.tsx
│   └── index.ts
├── quiz/
│   ├── ui/
│   │   └── QuizPage.tsx
│   └── index.ts
└── not-found/
    ├── ui/
    │   └── NotFoundPage.tsx
    └── index.ts
```

**pages/vacancies/ui/VacanciesPage.tsx:**
```typescript
import { VacancyList } from '@/widgets/vacancy-list';
import { VacancyFilters } from '@/features/vacancy-filters';

export const VacanciesPage = () => {
  return (
    <div>
      <h1>Vakansiyalar</h1>
      <VacancyFilters />
      <VacancyList />
    </div>
  );
};
```

**pages/vacancies/index.ts:**
```typescript
export { VacanciesPage } from './ui/VacanciesPage';
export { VacancyDetailsPage } from './ui/VacancyDetailsPage';
```

---

## 3️⃣ **WIDGETS Layer** - Katta UI bloklari

Bir nechta features va entities dan tashkil topgan murakkab komponentlar.

```
widgets/
├── header/
│   ├── ui/
│   │   ├── Header.tsx
│   │   └── Header.module.css
│   └── index.ts
├── vacancy-list/
│   ├── ui/
│   │   ├── VacancyList.tsx
│   │   └── VacancyCard.tsx
│   ├── model/
│   │   └── useVacancyList.ts
│   └── index.ts
└── quiz-results/
    ├── ui/
    │   └── QuizResults.tsx
    └── index.ts
```

**widgets/vacancy-list/ui/VacancyList.tsx:**
```typescript
import { useGetActiveVacanciesQuery } from '@/entities/applicant';
import { VacancyCard } from '@/entities/vacancy';
import { ApplyButton } from '@/features/apply-to-vacancy';

export const VacancyList = () => {
  const { data: vacancies, isLoading } = useGetActiveVacanciesQuery({
    branch: 'tashkent',
    department: 'it'
  });

  if (isLoading) return <div>Yuklanmoqda...</div>;

  return (
    <div className="vacancy-list">
      {vacancies?.map((vacancy) => (
        <div key={vacancy.id}>
          <VacancyCard vacancy={vacancy} />
          <ApplyButton vacancyId={vacancy.id} />
        </div>
      ))}
    </div>
  );
};
```

---

## 4️⃣ **FEATURES Layer** - Biznes funksionallik

Foydalanuvchi o'zaro ta'siri (user actions).

```
features/
├── auth/
│   ├── ui/
│   │   ├── LoginForm.tsx
│   │   └── RegisterForm.tsx
│   ├── model/
│   │   ├── types.ts
│   │   └── useAuth.ts
│   ├── api/
│   │   └── auth-api.ts
│   └── index.ts
├── apply-to-vacancy/
│   ├── ui/
│   │   ├── ApplyButton.tsx
│   │   └── ApplicationForm.tsx
│   ├── model/
│   │   ├── types.ts
│   │   └── useApply.ts
│   └── index.ts
└── vacancy-filters/
    ├── ui/
    │   └── VacancyFilters.tsx
    ├── model/
    │   ├── types.ts
    │   └── useFilters.ts
    └── index.ts
```

**features/apply-to-vacancy/ui/ApplyButton.tsx:**
```typescript
import { useState } from 'react';
import { useAddApplicantMutation } from '@/entities/applicant';
import { Button } from '@/shared/ui/button';

export const ApplyButton = ({ vacancyId }: { vacancyId: number }) => {
  const [isOpen, setIsOpen] = useState(false);
  const [apply, { isLoading }] = useAddApplicantMutation();

  const handleApply = async () => {
    await apply({ vacancyId, userId: 123 });
    setIsOpen(false);
  };

  return (
    <>
      <Button onClick={() => setIsOpen(true)}>
        Ariza yuborish
      </Button>
      {/* Modal with form */}
    </>
  );
};
```

---

## 5️⃣ **ENTITIES Layer** - Biznes entitylar

Domenga oid ma'lumotlar va API.

```
entities/
├── user/
│   ├── api/
│   │   └── user-api.ts
│   ├── model/
│   │   ├── types.ts
│   │   └── selectors.ts
│   ├── ui/
│   │   ├── UserCard.tsx
│   │   └── UserAvatar.tsx
│   └── index.ts
├── vacancy/
│   ├── api/
│   │   └── vacancy-api.ts
│   ├── model/
│   │   └── types.ts
│   ├── ui/
│   │   └── VacancyCard.tsx
│   └── index.ts
├── applicant/
│   ├── api/
│   │   └── applicant-api.ts
│   ├── model/
│   │   └── types.ts
│   └── index.ts
└── quiz/
    ├── api/
    │   └── quiz-api.ts
    ├── model/
    │   └── types.ts
    ├── ui/
    │   └── QuestionCard.tsx
    └── index.ts
```

**entities/vacancy/api/vacancy-api.ts:**
```typescript
import { baseApi } from '@/shared/api/base-api';
import { IVacancy } from '../model/types';

export const vacancyApi = baseApi.injectEndpoints({
  endpoints: (builder) => ({
    getVacancies: builder.query<IVacancy[], void>({
      query: () => '/vacancies',
      providesTags: ['Vacancies'],
    }),
    getVacancyById: builder.query<IVacancy, number>({
      query: (id) => `/vacancy/${id}`,
      providesTags: (result, error, id) => [{ type: 'Vacancies', id }],
    }),
  }),
});

export const { useGetVacanciesQuery, useGetVacancyByIdQuery } = vacancyApi;
```

**entities/vacancy/model/types.ts:**
```typescript
export interface IVacancy {
  id: number;
  title: string;
  department: string;
  branch: string;
  description: string;
  requirements: string[];
  salary?: string;
}
```

**entities/vacancy/ui/VacancyCard.tsx:**
```typescript
import { IVacancy } from '../model/types';
import { Card } from '@/shared/ui/card';

export const VacancyCard = ({ vacancy }: { vacancy: IVacancy }) => {
  return (
    <Card>
      <h3>{vacancy.title}</h3>
      <p>{vacancy.department}</p>
      <p>{vacancy.branch}</p>
    </Card>
  );
};
```

**entities/vacancy/index.ts:**
```typescript
export * from './api/vacancy-api';
export * from './model/types';
export { VacancyCard } from './ui/VacancyCard';
```

---

## 6️⃣ **SHARED Layer** - Umumiy kod

Loyiha bo'ylab qayta ishlatiladigan kod.

```
shared/
├── ui/                    # UI komponentlar (Design System)
│   ├── button/
│   │   ├── Button.tsx
│   │   ├── Button.module.css
│   │   └── index.ts
│   ├── input/
│   │   ├── Input.tsx
│   │   └── index.ts
│   ├── card/
│   ├── modal/
│   └── index.ts
├── lib/                   # Utility funksiyalar
│   ├── apis.ts
│   ├── utils.ts
│   ├── constants.ts
│   └── hooks/
│       ├── useDebounce.ts
│       └── useLocalStorage.ts
├── api/                   # Base API konfiguratsiya
│   ├── base-api.ts
│   └── index.ts
├── config/               # Konfiguratsiya
│   ├── env.ts
│   └── routes.ts
├── types/                # Global types
│   └── common.ts
└── locales/              # i18n
    ├── en.json
    ├── uz.json
    └── ru.json
```

**shared/ui/button/Button.tsx:**
```typescript
import { ButtonHTMLAttributes } from 'react';
import styles from './Button.module.css';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary';
  size?: 'sm' | 'md' | 'lg';
}

export const Button = ({ 
  variant = 'primary', 
  size = 'md',
  children,
  ...props 
}: ButtonProps) => {
  return (
    <button 
      className={`${styles.button} ${styles[variant]} ${styles[size]}`}
      {...props}
    >
      {children}
    </button>
  );
};
```

**shared/lib/hooks/useDebounce.ts:**
```typescript
import { useEffect, useState } from 'react';

export const useDebounce = <T,>(value: T, delay: number = 500): T => {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(handler);
  }, [value, delay]);

  return debouncedValue;
};
```

**shared/api/base-api.ts:**
```typescript
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import { API } from '@/shared/lib/apis';

export const baseApi = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({ 
    baseUrl: API,
    prepareHeaders: (headers) => {
      const token = localStorage.getItem('token');
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    }
  }),
  tagTypes: ['Vacancies', 'Applications', 'Exam', 'Questions'],
  endpoints: () => ({}),
});
```

---

## 📦 Segmentlar (har bir slice ichida)

### **UI** - Foydalanuvchi interfeysi
```typescript
// entities/user/ui/UserCard.tsx
export const UserCard = ({ user }) => { ... };
```

### **MODEL** - Biznes logika, state, types
```typescript
// entities/user/model/types.ts
export interface User {
  id: number;
  name: string;
  email: string;
}

// entities/user/model/selectors.ts
export const selectCurrentUser = (state) => state.user.current;
```

### **API** - Backend bilan ishlash
```typescript
// entities/user/api/user-api.ts
export const userApi = baseApi.injectEndpoints({ ... });
```

### **LIB** - Utility funksiyalar (slice uchun)
```typescript
// features/auth/lib/validation.ts
export const validateEmail = (email: string) => { ... };
```

---

## 🌍 Lokalizatsiya (i18n)

**shared/locales/** papkasida:

```
shared/
└── locales/
    ├── en.json
    ├── uz.json
    └── ru.json
```

**shared/locales/uz.json:**
```json
{
  "common": {
    "save": "Saqlash",
    "cancel": "Bekor qilish",
    "loading": "Yuklanmoqda..."
  },
  "vacancy": {
    "title": "Vakansiya",
    "apply": "Ariza yuborish",
    "requirements": "Talablar"
  }
}
```

**Ishlatish:**
```typescript
import { useTranslation } from 'react-i18next';

export const ApplyButton = () => {
  const { t } = useTranslation();
  
  return <button>{t('vacancy.apply')}</button>;
};
```

---

## ✅ FSD afzalliklari

1. **Modulli** - Har bir qism mustaqil
2. **Scalable** - Oson kengaytirish
3. **Maintainable** - Oson maintenance
4. **Team-friendly** - Jamoa uchun qulay
5. **Clear dependencies** - Aniq bog'lanishlar

## ❌ Keng tarqalgan xatolar

```typescript
// ❌ Features dan entities ga bog'lanish
// features/apply-to-vacancy/ui/ApplyButton.tsx
import { VacancyCard } from '@/entities/vacancy'; // XATO!

// ✅ Faqat pastga bog'lanish
// widgets/vacancy-list/ui/VacancyList.tsx
import { VacancyCard } from '@/entities/vacancy'; // TO'G'RI
import { ApplyButton } from '@/features/apply-to-vacancy'; // TO'G'RI
```

# FSD (Feature-Sliced Design) - To'liq qo'llanma

FSD - bu frontend loyihalarni tashkil qilish uchun arxitektura metodologiyasi. Keling, har bir qatlamni batafsil ko'rib chiqamiz:

## 📚 FSD Qatlamlari (yuqoridan pastga)

### 1. **App** - Ilovaning asosi
Bu ilova uchun global sozlamalar va konfiguratsiyalar joyi.

**Nima joylashadi:**
- `providers/` - Global provayderlar (Redux, Router, Theme)
- `styles/` - Global stillar va CSS o'zgaruvchilar
- `config/` - Ilova konfiguratsiyalari
- `index.tsx` - Ilova kirish nuqtasi

**Misol:**
```
app/
├── providers/
│   ├── StoreProvider.tsx
│   ├── RouterProvider.tsx
│   └── ThemeProvider.tsx
├── styles/
│   ├── global.css
│   └── variables.css
└── index.tsx
```

### 2. **Pages** - Sahifalar
Har bir marshrut (route) uchun alohida sahifa. Bu foydalanuvchi ko'radigan to'liq ekranlar.

**Qoidalar:**
- Faqat sahifa darajasidagi komponentlar
- Widgets va Features dan foydalanadi
- Boshqa Pages dan import qilmaydi

**Misol:**
```
pages/
├── home/
│   ├── ui/
│   │   └── HomePage.tsx
│   └── index.ts
├── profile/
│   ├── ui/
│   │   └── ProfilePage.tsx
│   └── index.ts
└── products/
    ├── ui/
    │   ├── ProductsPage.tsx
    │   └── ProductDetail.tsx
    └── index.ts
```

### 3. **Widgets** - Murakkab komponentlar
Bu sahifaning katta qismlari - header, sidebar, footer kabi murakkab bloklar.

**Xususiyatlari:**
- Features va Entities dan tuziladi
- O'z ichida biznes logika bo'lishi mumkin
- Qayta ishlatilishi mumkin, lekin majburiy emas

**Misol:**
```
widgets/
├── header/
│   ├── ui/
│   │   ├── Header.tsx
│   │   └── UserMenu.tsx
│   ├── model/
│   │   └── useHeader.ts
│   └── index.ts
├── sidebar/
│   ├── ui/
│   │   └── Sidebar.tsx
│   └── index.ts
└── product-card/
    ├── ui/
    │   ├── ProductCard.tsx
    │   └── ProductPrice.tsx
    ├── model/
    │   └── useProductCard.ts
    └── index.ts
```

### 4. **Features** - Biznes funksiyalar
Bu foydalanuvchi harakatlari va biznes funksionallik (login, qo'shish, o'chirish va h.k.).

**Qoidalar:**
- Bitta feature = bitta foydalanuvchi harakati
- Entities dan foydalanadi
- Boshqa Features dan import qilmaydi

**Misol:**
```
features/
├── auth/
│   ├── login/
│   │   ├── ui/
│   │   │   └── LoginForm.tsx
│   │   ├── model/
│   │   │   ├── loginSlice.ts
│   │   │   └── useLogin.ts
│   │   ├── api/
│   │   │   └── loginApi.ts
│   │   └── index.ts
│   └── register/
│       └── ...
├── add-to-cart/
│   ├── ui/
│   │   └── AddToCartButton.tsx
│   ├── model/
│   │   └── useAddToCart.ts
│   └── index.ts
└── product-filter/
    ├── ui/
    │   └── ProductFilter.tsx
    ├── model/
    │   └── filterSlice.ts
    └── index.ts
```

### 5. **Entities** - Biznes ob'ektlar
Bu dasturingizning asosiy ma'lumot ob'ektlari (User, Product, Order va h.k.).

**Xususiyatlari:**
- Faqat ma'lumotlar va ularning mantiq logikasi
- Boshqa qatlamlardan mustaqil
- Qayta ishlatiladi

**Misol:**
```
entities/
├── user/
│   ├── model/
│   │   ├── types.ts
│   │   ├── userSlice.ts
│   │   └── selectors.ts
│   ├── api/
│   │   └── userApi.ts
│   ├── ui/
│   │   ├── UserCard.tsx
│   │   └── UserAvatar.tsx
│   └── index.ts
├── product/
│   ├── model/
│   │   ├── types.ts
│   │   └── productSlice.ts
│   ├── api/
│   │   └── productApi.ts
│   ├── ui/
│   │   └── ProductImage.tsx
│   └── index.ts
└── order/
    └── ...
```

### 6. **Shared** - Umumiy kod
Bu loyiha bo'ylab qayta ishlatiladigan, biznes logikaga bog'liq bo'lmagan kod.

**Tuzilishi:**
```
shared/
├── ui/
│   ├── Button/
│   ├── Input/
│   ├── Modal/
│   └── Card/
├── lib/
│   ├── hooks/
│   │   ├── useDebounce.ts
│   │   └── useLocalStorage.ts
│   ├── utils/
│   │   ├── formatDate.ts
│   │   └── validators.ts
│   └── helpers/
├── api/
│   ├── axios.ts
│   └── endpoints.ts
├── config/
│   └── constants.ts
├── types/
│   └── common.ts
└── locales/
    ├── en.json
    ├── uz.json
    └── ru.json
```

## 📁 Har bir slice ichidagi segment tuzilishi

### **ui/** - UI komponentlar
```typescript
// features/login/ui/LoginForm.tsx
export const LoginForm = () => {
  return <form>...</form>
}
```

### **model/** - Ma'lumotlar va holat boshqaruvi
```typescript
// entities/user/model/types.ts
export interface User {
  id: string;
  name: string;
  email: string;
}

// entities/user/model/userSlice.ts
export const userSlice = createSlice({
  name: 'user',
  initialState,
  reducers: {...}
})
```

### **api/** - Backend bilan aloqa
```typescript
// entities/user/api/userApi.ts
export const userApi = {
  getUser: (id: string) => axios.get(`/users/${id}`),
  updateUser: (data: User) => axios.put('/users', data)
}
```

### **lib/** - Yordamchi funksiyalar
```typescript
// features/auth/lib/validation.ts
export const validateEmail = (email: string) => {
  return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)
}
```

### **config/** - Konfiguratsiya
```typescript
// features/payment/config/constants.ts
export const PAYMENT_METHODS = {
  CARD: 'card',
  CASH: 'cash'
}
```

## 🔗 Import qoidalari

**Ruxsat etilgan importlar (yuqoridan pastga):**
```
App → Pages → Widgets → Features → Entities → Shared
```

**Taqiqlangan:**
- Pastdan yuqoriga import (masalan: Shared → Features)
- Bir xil darajadagi slice'lar o'rtasida (masalan: Feature A → Feature B)

## 🌍 Locales va Translations

### Tuzilish:
```
shared/
└── locales/
    ├── en/
    │   ├── common.json
    │   ├── auth.json
    │   └── products.json
    ├── uz/
    │   ├── common.json
    │   ├── auth.json
    │   └── products.json
    └── index.ts
```

### Misol fayllar:
```json
// shared/locales/uz/auth.json
{
  "login": "Kirish",
  "email": "Email",
  "password": "Parol",
  "submit": "Yuborish"
}

// shared/locales/en/auth.json
{
  "login": "Login",
  "email": "Email",
  "password": "Password",
  "submit": "Submit"
}
```

### Ishlatish:
```typescript
// features/login/ui/LoginForm.tsx
import { useTranslation } from 'react-i18next'

export const LoginForm = () => {
  const { t } = useTranslation('auth')
  
  return (
    <form>
      <h1>{t('login')}</h1>
      <input placeholder={t('email')} />
    </form>
  )
}
```

## ✅ FSD afzalliklari:

1. **Tushunarli tuzilma** - Har kim tezda yo'naladi
2. **Oson kengaytirish** - Yangi feature qo'shish oson
3. **Qayta ishlatish** - Kodning ko'p qismi universal
4. **Test qilish** - Har bir qism alohida test qilinadi
5. **Jamoa ishi** - Bir vaqtda turli qatlamlarda ishlash mumkin

Bu FSD arxitekturasi loyihangizni toza, tushunarli va kengaytirilishi oson qiladi!
