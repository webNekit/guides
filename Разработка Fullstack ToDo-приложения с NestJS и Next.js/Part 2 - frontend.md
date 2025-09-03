# Учебник-гайд: Разработка Fullstack ToDo-приложения с NestJS и Next.js

## Глава 2: Фронтенд (Next.js)

В этой главе мы создадим фронтенд для ToDo-приложения с использованием **Next.js**, **TypeScript**, **React Hook Form**, **Zod** для валидации и **Shadcn/UI** для компонентов интерфейса. Фронтенд будет включать страницы для входа, регистрации, управления задачами и админской панели, а также middleware для защиты маршрутов. Мы начнем с установки и настройки проекта, затем создадим необходимые файлы в логическом порядке, добавляя подробные комментарии и ссылки на документацию. Константы из `lib/constans.ts` и переменные окружения из `.env` будут использованы для оптимизации кода.

---

### 2.1 Установка и инициализация проекта

#### Установка зависимостей

1. **Создайте новый проект Next.js** с TypeScript и ESLint:

   ```bash
   npx create-next-app@latest frontend --typescript --eslint --tailwind --app --src-dir --import-alias "@/*"
   cd frontend
   ```

   > **Комментарий**: Команда создает проект с TypeScript, Tailwind CSS и структурой App Router. Опция `--src-dir` добавляет папку `src`, а `--import-alias "@/*"` упрощает импорты. Подробности: [Next.js Create Next App](https://nextjs.org/docs/app/api-reference/create-next-app).

2. **Установите зависимости** для работы с API, формами, валидацией и UI:

   ```bash
   npm install axios react-hook-form @hookform/resolvers zod jose
   npm install -D @types/node
   ```

   > **Комментарий**:
   > - `axios` для выполнения HTTP-запросов к бэкенду. Подробности: [Axios](https://axios-http.com/docs/intro).
   > - `react-hook-form` для управления формами. Подробности: [React Hook Form](https://react-hook-form.com/).
   > - `@hookform/resolvers` для интеграции Zod с React Hook Form. Подробности: [Hookform Resolvers](https://github.com/react-hook-form/resolvers).
   > - `zod` для валидации данных. Подробности: [Zod](https://zod.dev/).
   > - `jose` для проверки JWT-токенов в middleware. Подробности: [Jose](https://github.com/panva/jose).
   > - `@types/node` для типизации переменных окружения.

3. **Установите Shadcn/UI** для компонентов интерфейса:

   ```bash
   npx shadcn-ui@latest init
   ```

   > **Комментарий**: Следуйте инструкциям CLI, выбрав TypeScript, Tailwind CSS и желаемые настройки. Shadcn/UI добавляет компоненты, такие как `Button`, `Input`, `Table`. Подробности: [Shadcn/UI](https://ui.shadcn.com/docs/installation).

4. **Добавьте компоненты Shadcn/UI**:

   ```bash
   npx shadcn-ui@latest add button input table
   ```

   > **Комментарий**: Команда добавляет компоненты в `src/components/ui/`. Они используются для форм и таблиц. Подробности: [Shadcn/UI Components](https://ui.shadcn.com/docs/components).

#### Настройка окружения

1. **Создайте файл** `frontend/.env`:

   ```env
   NEXT_PUBLIC_API_URL=http://localhost:9000/api
   JWT_ACCESS_SECRET=access-secret-key
   ```

   > **Комментарий**: `NEXT_PUBLIC_API_URL` доступен на клиенте и сервере для API-запросов. `JWT_ACCESS_SECRET` используется в middleware для проверки токенов. Используйте безопасные ключи в продакшене. Подробности: [Next.js Environment Variables](https://nextjs.org/docs/app/building-your-application/configuring/environment-variables).

---

### 2.2 Настройка констант и типов

1. **Создайте файл** `src/lib/constans.ts`:

   ```typescript
   // Определяем константы для API
   export const API_URL = process.env.NEXT_PUBLIC_API_URL || "http://localhost:9000/api"; // Базовый URL API
   export const API_HEADER = { 'Content-Type': 'application/json' }; // Заголовки для запросов
   ```

   > **Комментарий**: Файл содержит константы для URL API и заголовков, чтобы избежать хардкода. `API_URL` использует значение из `.env`. Подробности: [Next.js Environment Variables](https://nextjs.org/docs/app/building-your-application/configuring/environment-variables).

2. **Создайте файл** `src/lib/types.ts`:

   ```typescript
   // Определяем интерфейс задачи
   export interface Task {
     id: number; // Уникальный идентификатор
     title: string; // Название задачи
     description: string; // Описание задачи
     completed: boolean; // Статус выполнения
     userId: number; // ID владельца
     user: {
       id: number; // ID пользователя
       name: string; // Имя пользователя
       email: string; // Email пользователя
       role: string; // Роль (USER или ADMIN)
     };
   }
   ```

   > **Комментарий**: Интерфейс `Task` описывает структуру задачи, включая данные пользователя. Используется для типизации данных с бэкенда. Подробности: [TypeScript Interfaces](https://www.typescriptlang.org/docs/handbook/interfaces.html).

---

### 2.3 Настройка API-клиента

1. **Создайте файл** `src/api/config.api.ts`:

   ```typescript
   // Импортируем зависимости
   import axios from 'axios';
   import { API_URL, API_HEADER } from '@/lib/constans';
   
   // Создаем экземпляр axios для API-запросов
   const api = axios.create({
     baseURL: API_URL, // Базовый URL из констант
     headers: API_HEADER, // Заголовки из констант
     withCredentials: true, // Включаем отправку cookies
   });
   
   // Настраиваем перехватчик для обработки ошибок
   api.interceptors.response.use(
     (response) => response, // Успешный ответ возвращаем как есть
     async (error) => {
       if (!error.response) {
         return Promise.reject(error); // Сетевая ошибка
       }
   
       const originalRequest = error.config;
       // Если получили 401 и запрос не повторялся
       if (error.response.status === 401 && !originalRequest._retry) {
         originalRequest._retry = true; // Помечаем запрос как повторный
         try {
           await api.post('/auth/refresh'); // Пытаемся обновить токен
           return api(originalRequest); // Повторяем исходный запрос
         } catch (refreshError) {
           return Promise.reject(refreshError); // Ошибка при обновлении токена
         }
       }
       return Promise.reject(error); // Другие ошибки
     }
   );
   
   export default api;
   ```

   > **Комментарий**: Этот файл настраивает `axios` с базовым URL, заголовками и поддержкой cookies. Перехватчик обрабатывает 401 ошибки, автоматически обновляя токен через `/auth/refresh`. Подробности: [Axios Interceptors](https://axios-http.com/docs/interceptors).

2. **Создайте файл** `src/api/headers.api.ts`:

   ```typescript
   // Экспортируем функцию для получения заголовков с токеном
   export const getAuthHeaders = async () => {
     try {
       // Динамический импорт cookies (доступен только на сервере)
       const { cookies } = await import('next/headers');
       const cookieStore = cookies();
       const accessToken = cookieStore.get('accessToken')?.value;
       // Возвращаем заголовок с токеном, если он есть
       return accessToken ? { Cookie: `accessToken=${accessToken}` } : {};
     } catch (err) {
       // Если вызвано на клиенте, возвращаем пустой объект
       return {};
     }
   };
   ```

   > **Комментарий**: Функция извлекает `accessToken` из cookies на сервере для серверных запросов. Динамический импорт `next/headers` предотвращает ошибки на клиенте. Подробности: [Next.js Cookies](https://nextjs.org/docs/app/api-reference/functions/cookies).

---

### 2.4 Настройка схем валидации

1. **Создайте файл** `src/sсhemas/auth.ts`:

   ```typescript
   // Импортируем Zod для валидации
   import { z } from 'zod';
   
   // Схема для входа
   export const loginSchema = z.object({
     email: z.string().email('Введите корректный адрес email'), // Проверяем формат email
     password: z.string().min(6, 'Минимальная длина пароля 6 символов'), // Проверяем длину пароля
   });
   
   // Схема для регистрации
   export const registerSchema = z.object({
     name: z.string({ required_error: 'Поле должно быть строкой' }), // Проверяем имя
     email: z.string({ required_error: 'Поле должно быть строкой' }).email('Введите корректный адрес email'), // Проверяем email
     password: z.string({ required_error: 'Поле должно быть строкой' }).min(6, 'Минимальная длина пароля 6 символов'), // Проверяем пароль
   });
   ```

   > **Комментарий**: Схемы Zod определяют правила валидации для форм входа и регистрации. Ошибки отображаются в форме. Подробности: [Zod](https://zod.dev/).

2. **Создайте файл** `src/sсhemas/tasks.ts`:

   ```typescript
   // Импортируем Zod для валидации
   import { z } from 'zod';
   
   // Схема для создания задачи
   export const createTaskSchema = z.object({
     title: z.string({ required_error: 'Поле должно быть строкой' }), // Обязательное поле
     description: z.string({ required_error: 'Поле должно быть строкой' }).optional(), // Опциональное поле
   });
   
   // Схема для обновления задачи
   export const updateTaskSchema = z.object({
     title: z.string({ required_error: 'Поле должно быть строкой' }).optional(), // Опциональное поле
     description: z.string({ required_error: 'Поле должно быть строкой' }).optional(), // Опциональное поле
     completed: z.boolean(), // Обязательное поле для статуса
   });
   ```

   > **Комментарий**: Схемы определяют валидацию для форм создания и редактирования задач. Поле `completed` обязательно только при обновлении. Подробности: [Zod](https://zod.dev/).

---

### 2.5 Настройка сервисов API

1. **Создайте файл** `src/services/auth/auth.ts`:

   ```typescript
   // Импортируем зависимости
   import api from '@/api/config.api';
   import { loginSchema, registerSchema } from '@/sсhemas/auth';
   import { z } from 'zod';
   
   // Функция для входа
   export const login = async (data: z.infer<typeof loginSchema>) => {
     return api.post('/auth/login', data); // Отправляем POST-запрос на вход
   };
   
   // Функция для регистрации
   export const register = async (data: z.infer<typeof registerSchema>) => {
     return api.post('/auth/register', data); // Отправляем POST-запрос на регистрацию
   };
   
   // Функция для выхода
   export const logout = async () => {
     return api.post('/auth/logout'); // Отправляем POST-запрос на выход
   };
   ```

   > **Комментарий**: Сервис содержит функции для аутентификации, использующие `api` из `config.api.ts`. Данные валидируются схемами Zod. Подробности: [Axios](https://axios-http.com/docs/post_example).

2. **Создайте файл** `src/services/tasks/tasks.ts`:

   ```typescript
   // Импортируем зависимости
   import api from '@/api/config.api';
   import { getAuthHeaders } from '@/api/headers.api';
   import { Task } from '@/lib/types';
   import { createTaskSchema, updateTaskSchema } from '@/sсhemas/tasks';
   import { z } from 'zod';
   
   // Получение всех задач
   export const getAllTasks = async (): Promise<Task[]> => {
     const response = await api.get('/tasks', {
       headers: await getAuthHeaders(), // Добавляем заголовки с токеном
     });
     return response.data; // Возвращаем массив задач
   };
   
   // Получение одной задачи
   export const getOneTask = async (id: string) => {
     const response = await api.get(`/tasks/${id}`, {
       headers: await getAuthHeaders(), // Добавляем заголовки с токеном
     });
     return response; // Возвращаем задачу
   };
   
   // Создание задачи
   export const createTask = async (data: z.infer<typeof createTaskSchema>) => {
     return api.post('/tasks', data); // Отправляем POST-запрос
   };
   
   // Обновление задачи
   export const updateTask = async (id: string, data: z.infer<typeof updateTaskSchema>) => {
     return api.put(`/tasks/${id}`, data); // Отправляем PUT-запрос
   };
   
   // Удаление задачи
   export const deleteTask = async (id: string) => {
     return api.delete(`/tasks/${id}`); // Отправляем DELETE-запрос
   };
   ```

   > **Комментарий**: Сервис содержит функции для CRUD-операций с задачами. `getAuthHeaders` добавляет токен для защищенных запросов. Подробности: [Axios](https://axios-http.com/docs/api_intro).

---

### 2.6 Настройка middleware для защиты маршрутов

1. **Создайте файл** `src/middleware.ts`:

   ```typescript
   // Импортируем зависимости
   import { NextResponse } from 'next/server';
   import type { NextRequest } from 'next/server';
   import { jwtVerify } from 'jose';
   
   // Определяем публичные маршруты
   const publicRoutes = ['/login', '/register'];
   // Определяем маршруты для админов
   const adminRoutes = ['/dashboard', '/dashboard/(.*)'];
   // Определяем маршруты для пользователей
   const userRoutes = ['/', '/tasks'];
   
   // Middleware для проверки аутентификации
   export async function middleware(request: NextRequest) {
     const { pathname } = request.nextUrl;
     const accessToken = request.cookies.get('accessToken')?.value;
   
     // Проверка API-запросов
     if (pathname.startsWith('/api')) {
       if (!accessToken) {
         return new NextResponse('Unauthorized', { status: 401 }); // Нет токена
       }
       try {
         const secret = new TextEncoder().encode(process.env.JWT_ACCESS_SECRET); // Секрет для проверки
         await jwtVerify(accessToken, secret); // Проверяем токен
         return NextResponse.next(); // Продолжаем запрос
       } catch {
         return new NextResponse('Unauthorized', { status: 401 }); // Неверный токен
       }
     }
   
     // Авторизованные пользователи не должны попадать на login/register
     if (publicRoutes.includes(pathname) && accessToken) {
       try {
         const secret = new TextEncoder().encode(process.env.JWT_ACCESS_SECRET);
         const { payload } = await jwtVerify(accessToken, secret);
         const role = payload.role as string;
         const redirectPath = role === 'ADMIN' ? '/dashboard' : '/'; // Редирект по роли
         return NextResponse.redirect(new URL(redirectPath, request.url));
       } catch {
         return NextResponse.next(); // Если токен неверный, разрешаем доступ
       }
     }
   
     // Неавторизованные пользователи перенаправляются на login
     if (!accessToken) {
       return NextResponse.redirect(new URL('/login', request.url));
     }
   
     try {
       const secret = new TextEncoder().encode(process.env.JWT_ACCESS_SECRET);
       const { payload } = await jwtVerify(accessToken, secret);
       const role = payload.role as string;
   
       // Проверка ролей
       if (role === 'ADMIN' && userRoutes.includes(pathname)) {
         return NextResponse.redirect(new URL('/dashboard', request.url)); // Админы на /dashboard
       }
       if (role === 'USER' && adminRoutes.some((route) => pathname.startsWith(route.replace('/(.*)', '')))) {
         return NextResponse.redirect(new URL('/', request.url)); // Пользователи не могут на /dashboard
       }
   
       return NextResponse.next(); // Разрешаем доступ
     } catch {
       return NextResponse.redirect(new URL('/login', request.url)); // Неверный токен
     }
   }
   
   // Настраиваем маршруты, к которым применяется middleware
   export const config = {
     matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'], // Исключаем статические файлы
   };
   ```

   > **Комментарий**: Middleware проверяет `accessToken` и роль пользователя, перенаправляя на соответствующие страницы. `jose` используется для проверки JWT. Подробности: [Next.js Middleware](https://nextjs.org/docs/app/building-your-application/routing/middleware).

---

### 2.7 Настройка компонентов и страниц

1. **Создайте файл** `src/app/(auth)/layout.tsx`:

   ```typescript
   // Импортируем зависимости
   import type { Metadata } from 'next';
   import { GeistSans, GeistMono } from 'next/font/google';
   import './globals.css';
   
   // Настраиваем шрифты Geist
   const geistSans = GeistSans({
     variable: '--font-geist-sans',
     subsets: ['latin'],
   });
   
   const geistMono = GeistMono({
     variable: '--font-geist-mono',
     subsets: ['latin'],
   });
   
   // Метаданные для страниц аутентификации
   export const metadata: Metadata = {
     title: 'ToDo App',
     description: 'ToDo application with NestJS and Next.js',
   };
   
   // Определяем layout для страниц аутентификации
   export default function RootLayout({ children }: Readonly<{ children: React.ReactNode }>) {
     return (
       <html lang="en">
         <body className={`${geistSans.variable} ${geistMono.variable} antialiased`}>
           {children} {/* Рендерим дочерние компоненты */}
         </body>
       </html>
     );
   }
   ```

   > **Комментарий**: Layout задает шрифты и метаданные для страниц `/login` и `/register`. Подробности: [Next.js Layouts](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#layouts).

2. **Создайте файл** `src/app/(auth)/login/page.tsx`:

   ```typescript
   // Импортируем зависимости
   import LoginForm from '@/components/auth/login-form';
   import { Metadata } from 'next';
   
   // Метаданные для страницы входа
   export const metadata: Metadata = {
     title: 'Авторизация',
   };
   
   // Страница входа
   export default function LoginPage() {
     return (
       <div>
         <LoginForm /> {/* Рендерим форму входа */}
       </div>
     );
   }
   ```

   > **Комментарий**: Страница рендерит компонент `LoginForm`. Подробности: [Next.js Pages](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts).

3. **Создайте файл** `src/app/(auth)/register/page.tsx`:

   ```typescript
   // Импортируем зависимости
   import RegisterForm from '@/components/auth/register-form';
   import { Metadata } from 'next';
   
   // Метаданные для страницы регистрации
   export const metadata: Metadata = {
     title: 'Регистрация',
   };
   
   // Страница регистрации
   export default function RegisterPage() {
     return (
       <div>
         <RegisterForm /> {/* Рендерим форму регистрации */}
       </div>
     );
   }
   ```

   > **Комментарий**: Страница рендерит компонент `RegisterForm`. Подробности: [Next.js Pages](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts).

4. **Создайте файл** `src/components/auth/login-form.tsx`:

   ```typescript
   // Указываем, что компонент клиентский
   'use client';
   
   // Импортируем зависимости
   import React from 'react';
   import { Input } from '../ui/input';
   import { Button } from '../ui/button';
   import { useRouter } from 'next/navigation';
   import { useForm } from 'react-hook-form';
   import { zodResolver } from '@hookform/resolvers/zod';
   import { loginSchema } from '@/sсhemas/auth';
   import { login } from '@/services/auth/auth';
   import { z } from 'zod';
   import Link from 'next/link';
   
   // Компонент формы входа
   const LoginForm = () => {
     const router = useRouter();
     const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm({
       resolver: zodResolver(loginSchema), // Интеграция Zod для валидации
     });
   
     // Обработчик отправки формы
     const onSubmit = async (data: z.infer<typeof loginSchema>) => {
       try {
         await login(data); // Выполняем вход
         router.push('/'); // Перенаправляем на главную страницу
       } catch (err) {
         console.error(err);
       }
     };
   
     return (
       <div className="flex items-center justify-center min-h-screen bg-gray-100">
         <form onSubmit={handleSubmit(onSubmit)} className="p-6 bg-white rounded shadow-md space-y-4">
           <h2 className="text-2xl font-bold">Авторизация</h2>
           <Input
             {...register('email')}
             type="email"
             placeholder="Email"
             className={errors.email ? 'border-red-500' : ''} // Подсветка ошибок
           />
           {errors.email && <p className="text-red-500 text-sm">{errors.email.message}</p>}
           <Input
             {...register('password')}
             type="password"
             placeholder="Пароль"
             className={errors.password ? 'border-red-500' : ''} // Подсветка ошибок
           />
           {errors.password && <p className="text-red-500 text-sm">{errors.password.message}</p>}
           <Button type="submit" disabled={isSubmitting} className="w-full">
             {isSubmitting ? 'Вход...' : 'Войти'} {/* Статус отправки */}
           </Button>
           <Link href="/register">Нет аккаунта? Зарегистрируйтесь</Link>
         </form>
       </div>
     );
   };
   
   export default LoginForm;
   ```

   > **Комментарий**: Компонент использует `react-hook-form` и `zod` для валидации формы. После успешного входа пользователь перенаправляется на главную страницу. Подробности: [React Hook Form](https://react-hook-form.com/), [Zod](https://zod.dev/).

5. **Создайте файл** `src/components/auth/register-form.tsx`:

   ```typescript
   // Указываем, что компонент клиентский
   'use client';
   
   // Импортируем зависимости
   import React from 'react';
   import { Input } from '../ui/input';
   import { Button } from '../ui/button';
   import { useForm } from 'react-hook-form';
   import { z } from 'zod';
   import { zodResolver } from '@hookform/resolvers/zod';
   import { registerSchema } from '@/sсhemas/auth';
   import { useRouter } from 'next/navigation';
   import { register as registerService } from '@/services/auth/auth';
   
   // Компонент формы регистрации
   const RegisterForm = () => {
     const router = useRouter();
     const { register, handleSubmit, formState: { errors, isSubmitting } } = useForm({
       resolver: zodResolver(registerSchema), // Интеграция Zod для валидации
     });
   
     // Обработчик отправки формы
     const onSubmit = async (data: z.infer<typeof registerSchema>) => {
       try {
         await registerService(data); // Выполняем регистрацию
         router.push('/'); // Перенаправляем на главную страницу
       } catch (err) {
         console.error(err);
       }
     };
   
     return (
       <div className="flex items-center justify-center min-h-screen bg-gray-100">
         <form onSubmit={handleSubmit(onSubmit)} className="p-6 bg-white rounded shadow-md space-y-4">
           <h2 className="text-2xl font-bold">Регистрация</h2>
           <Input
             {...register('name')}
             type="text"
             placeholder="Имя"
             className={errors.name ? 'border-red-500' : ''} // Подсветка ошибок
           />
           {errors.name && <p className="text-red-500 text-sm">{errors.name.message}</p>}
           <Input
             {...register('email')}
             type="email"
             placeholder="Email"
             className={errors.email ? 'border-red-500' : ''} // Подсветка ошибок
           />
           {errors.email && <p className="text-red-500 text-sm">{errors.email.message}</p>}
           <Input
             {...register('password')}
             type="password"
             placeholder="Пароль"
             className={errors.password ? 'border-red-500' : ''} // Подсветка ошибок
           />
           {errors.password && <p className="text-red-500 text-sm">{errors.password.message}</p>}
           <Button type="submit" disabled={isSubmitting} className="w-full">
             {isSubmitting ? 'Регистрация...' : 'Регистрация'} {/* Статус отправки */}
           </Button>
         </form>
       </div>
     );
   };
   
   export default RegisterForm;
   ```

   > **Комментарий**: Компонент аналогичен `LoginForm`, но добавляет поле `name` для регистрации. Подробности: [React Hook Form](https://react-hook-form.com/), [Zod](https://zod.dev/).

6. **Создайте файл** `src/app/(main)/page.tsx`:

   ```typescript
   // Импортируем зависимости
   import TaskFormModal from '@/components/tasks/task-form';
   import TaskList from '@/components/tasks/task-list';
   import { getAllTasks } from '@/services/tasks/tasks';
   
   // Серверный компонент для страницы задач
   export default async function TasksPage() {
     const tasks = await getAllTasks(); // Получаем задачи с сервера
   
     return (
       <div className="max-w-7xl py-5 px-4 mx-auto">
         <TaskFormModal /> {/* Форма для создания задачи */}
         <TaskList tasks={tasks} /> {/* Список задач */}
       </div>
     );
   }
   ```

   > **Комментарий**: Серверный компонент загружает задачи через `getAllTasks` и рендерит форму и список. Подробности: [Next.js Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components).

7. **Создайте файл** `src/components/tasks/task-form.tsx`:

   ```typescript
   // Указываем, что компонент клиентский
   'use client';
   
   // Импортируем зависимости
   import React, { useState } from 'react';
   import { Button } from '@/components/ui/button';
   import { Input } from '@/components/ui/input';
   import { useForm } from 'react-hook-form';
   import { z } from 'zod';
   import { zodResolver } from '@hookform/resolvers/zod';
   import { createTaskSchema } from '@/sсhemas/tasks';
   import { createTask } from '@/services/tasks/tasks';
   import { useRouter } from 'next/navigation';
   
   // Компонент формы для создания задачи
   export default function TaskFormModal() {
     const [isOpen, setIsOpen] = useState(false); // Состояние модального окна
     const router = useRouter();
   
     const { register, handleSubmit, formState: { errors, isSubmitting }, reset } = useForm({
       resolver: zodResolver(createTaskSchema), // Интеграция Zod
       defaultValues: { title: '', description: '' }, // Начальные значения
     });
   
     // Обработчик отправки формы
     const onSubmit = async (data: z.infer<typeof createTaskSchema>) => {
       try {
         await createTask(data); // Создаем задачу
         reset(); // Сбрасываем форму
         setIsOpen(false); // Закрываем модальное окно
         router.refresh(); // Обновляем страницу
       } catch (err) {
         console.error(err);
       }
     };
   
     return (
       <div className="mb-6">
         <Button onClick={() => setIsOpen(true)}>Создать задачу</Button>
         {isOpen && (
           <div className="fixed inset-0 flex items-center justify-center bg-black/50 z-50">
             <div className="bg-white p-6 rounded-lg w-full max-w-md">
               <h3 className="text-lg font-semibold mb-4">Создать новую задачу</h3>
               <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
                 <div>
                   <Input
                     {...register('title')}
                     placeholder="Название задачи"
                     className={errors.title ? 'border-red-500' : ''} // Подсветка ошибок
                   />
                   {errors.title && <p className="text-red-500 text-sm">{errors.title.message}</p>}
                 </div>
                 <div>
                   <Input
                     {...register('description')}
                     placeholder="Описание (опционально)"
                     className={errors.description ? 'border-red-500' : ''} // Подсветка ошибок
                   />
                   {errors.description && <p className="text-red-500 text-sm">{errors.description.message}</p>}
                 </div>
                 <div className="flex justify-end space-x-2">
                   <Button type="button" variant="outline" onClick={() => setIsOpen(false)}>
                     Отмена
                   </Button>
                   <Button type="submit" disabled={isSubmitting}>
                     {isSubmitting ? 'Создание...' : 'Создать'} {/* Статус отправки */}
                   </Button>
                 </div>
               </form>
             </div>
           </div>
         )}
       </div>
     );
   }
   ```

   > **Комментарий**: Компонент рендерит модальное окно для создания задачи. Используется `react-hook-form` и `zod`. После создания задача страница обновляется. Подробности: [React Hook Form](https://react-hook-form.com/), [Next.js Navigation](https://nextjs.org/docs/app/api-reference/functions/use-router).

8. **Создайте файл** `src/components/tasks/task-list.tsx`:

   ```typescript
   // Указываем, что компонент клиентский
   'use client';
   
   // Импортируем зависимости
   import React, { useState } from 'react';
   import { Table, TableBody, TableCaption, TableCell, TableHead, TableHeader, TableRow } from '../ui/table';
   import { Task } from '@/lib/types';
   import { Button } from '../ui/button';
   import { deleteTask } from '@/services/tasks/tasks';
   import { useRouter } from 'next/navigation';
   import EditTaskModal from './edit-task-modal';
   
   // Компонент списка задач
   const TaskList = ({ tasks }: { tasks: Task[] }) => {
     const [editingTask, setEditingTask] = useState<Task | null>(null); // Текущая редактируемая задача
     const router = useRouter();
   
     // Обработчик удаления задачи
     const handleDelete = async (id: string) => {
       if (confirm('Удалить задачу?')) {
         await deleteTask(id); // Удаляем задачу
         router.refresh(); // Обновляем страницу
       }
     };
   
     return (
       <div>
         <Table>
           <TableCaption>Задачи</TableCaption>
           <TableHeader>
             <TableRow>
               <TableHead>#</TableHead>
               <TableHead>Задача</TableHead>
               <TableHead>Описание</TableHead>
               <TableHead>Статус</TableHead>
               <TableHead>Действия</TableHead>
             </TableRow>
           </TableHeader>
           <TableBody>
             {tasks.map((task) => (
               <TableRow key={task.id}>
                 <TableCell>{task.id}</TableCell>
                 <TableCell>{task.title}</TableCell>
                 <TableCell>{task.description}</TableCell>
                 <TableCell>{task.completed ? 'Выполнено' : 'Не выполнено'}</TableCell>
                 <TableCell className="space-x-2">
                   <Button variant="outline" onClick={() => setEditingTask(task)}>
                     Редактировать
                   </Button>
                   <Button variant="destructive" onClick={() => handleDelete(task.id)}>
                     Удалить
                   </Button>
                 </TableCell>
               </TableRow>
             ))}
           </TableBody>
         </Table>
         {editingTask && <EditTaskModal task={editingTask} onClose={() => setEditingTask(null)} />}
       </div>
     );
   };
   
   export default TaskList;
   ```

   > **Комментарий**: Компонент отображает таблицу задач с кнопками для редактирования и удаления. Используется `router.refresh()` для обновления данных. Подробности: [Shadcn/UI Table](https://ui.shadcn.com/docs/components/table), [Next.js Navigation](https://nextjs.org/docs/app/api-reference/functions/use-router).

9. **Создайте файл** `src/components/tasks/edit-task-modal.tsx`:

   ```typescript
   // Указываем, что компонент клиентский
   'use client';
   
   // Импортируем зависимости
   import React from 'react';
   import { Input } from '@/components/ui/input';
   import { Button } from '@/components/ui/button';
   import { useForm } from 'react-hook-form';
   import { z } from 'zod';
   import { zodResolver } from '@hookform/resolvers/zod';
   import { updateTaskSchema } from '@/sсhemas/tasks';
   import { updateTask } from '@/services/tasks/tasks';
   import { useRouter } from 'next/navigation';
   import { Task } from '@/lib/types';
   
   // Компонент модального окна для редактирования задачи
   export default function EditTaskModal({ task, onClose }: { task: Task; onClose: () => void }) {
     const router = useRouter();
   
     const { register, handleSubmit, formState: { errors, isSubmitting }, reset } = useForm({
       resolver: zodResolver(updateTaskSchema), // Интеграция Zod
       defaultValues: {
         title: task.title,
         description: task.description,
         completed: task.completed,
       },
     });
   
     // Обработчик отправки формы
     const onSubmit = async (data: z.infer<typeof updateTaskSchema>) => {
       try {
         await updateTask(task.id.toString(), data); // Обновляем задачу
         reset(); // Сбрасываем форму
         onClose(); // Закрываем модальное окно
         router.refresh(); // Обновляем страницу
       } catch (err) {
         console.error(err);
       }
     };
   
     return (
       <div className="fixed inset-0 flex items-center justify-center bg-black/50 z-50">
         <div className="bg-white p-6 rounded-lg w-full max-w-md">
           <h3 className="text-lg font-semibold mb-4">Редактировать задачу</h3>
           <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
             <div>
               <Input
                 {...register('title')}
                 placeholder="Название задачи"
                 className={errors.title ? 'border-red-500' : ''} // Подсветка ошибок
               />
               {errors.title && <p className="text-red-500 text-sm">{errors.title.message}</p>}
             </div>
             <div>
               <Input
                 {...register('description')}
                 placeholder="Описание (опционально)"
                 className={errors.description ? 'border-red-500' : ''} // Подсветка ошибок
               />
               {errors.description && <p className="text-red-500 text-sm">{errors.description.message}</p>}
             </div>
             <div>
               <label className="flex items-center space-x-2">
                 <input type="checkbox" {...register('completed')} defaultChecked={task.completed} />
                 <span>Выполнено</span>
               </label>
               {errors.completed && <p className="text-red-500 text-sm">{errors.completed.message}</p>}
             </div>
             <div className="flex justify-end space-x-2">
               <Button type="button" variant="outline" onClick={onClose}>
                 Отмена
               </Button>
               <Button type="submit" disabled={isSubmitting}>
                 {isSubmitting ? 'Сохранение...' : 'Сохранить'} {/* Статус отправки */}
               </Button>
             </div>
           </form>
         </div>
       </div>
     );
   }
   ```

   > **Комментарий**: Компонент рендерит модальное окно для редактирования задачи, включая поле `completed`. Подробности: [React Hook Form](https://react-hook-form.com/), [Shadcn/UI Input](https://ui.shadcn.com/docs/components/input).

10. **Создайте файл** `src/app/(dashboard)/dashboard/page.tsx`:

    ```typescript
    // Импортируем зависимости
    import TaskFormModal from '@/components/tasks/task-form';
    import TaskList from '@/components/tasks/task-list';
    import { getAllTasks } from '@/services/tasks/tasks';
    
    // Серверный компонент для админской панели
    export default async function DashboardPage() {
      const tasks = await getAllTasks(); // Получаем все задачи (для ADMIN)
    
      return (
        <div className="max-w-7xl py-5 px-4 mx-auto">
          <h2 className="text-2xl font-bold mb-4">Админская панель</h2>
          <TaskFormModal /> {/* Форма для создания задачи */}
          <TaskList tasks={tasks} /> {/* Список всех задач */}
        </div>
      );
    }
    ```

    > **Комментарий**: Страница админской панели отображает все задачи (доступно только для ADMIN). Используется тот же `TaskList` и `TaskFormModal`, что и на главной странице. Подробности: [Next.js Server Components](https://nextjs.org/docs/app/building-your-application/rendering/server-components).

11. **Создайте файл** `src/app/(main)/layout.tsx`:

    ```typescript
    // Импортируем зависимости
    import type { Metadata } from 'next';
    import { GeistSans, GeistMono } from 'next/font/google';
    import './globals.css';
    
    // Настраиваем шрифты Geist
    const geistSans = GeistSans({
      variable: '--font-geist-sans',
      subsets: ['latin'],
    });
    
    const geistMono = GeistMono({
      variable: '--font-geist-mono',
      subsets: ['latin'],
    });
    
    // Метаданные для основных страниц
    export const metadata: Metadata = {
      title: 'ToDo App',
      description: 'ToDo application with NestJS and Next.js',
    };
    
    // Определяем layout для основных страниц
    export default function RootLayout({ children }: Readonly<{ children: React.ReactNode }>) {
      return (
        <html lang="en">
          <body className={`${geistSans.variable} ${geistMono.variable} antialiased`}>
            {children} {/* Рендерим дочерние компоненты */}
          </body>
        </html>
      );
    }
    ```

    > **Комментарий**: Layout для основных страниц (`/`, `/tasks`, `/dashboard`) аналогичен `(auth)/layout.tsx`. Подробности: [Next.js Layouts](https://nextjs.org/docs/app/building-your-application/routing/pages-and-layouts#layouts).

---

### 2.8 Тестирование фронтенда

1. **Запустите фронтенд**:

   ```bash
   npm run dev
   ```

   > **Комментарий**: Сервер запустится на порту 3000 (или другом, указанном в настройках). Подробности: [Next.js CLI](https://nextjs.org/docs/app/api-reference/cli).

2. **Протестируйте функциональность**:

   - **Регистрация (`/register`)**:
     - Откройте `http://localhost:3000/register`.
     - Введите имя, email и пароль (не менее 6 символов).
     - После успешной регистрации перенаправляет на `/`.
   - **Вход (`/login`)**:
     - Откройте `http://localhost:3000/login`.
     - Введите email и пароль.
     - Перенаправляет на `/` (или `/dashboard` для ADMIN).
   - **Создание задачи (`/`)**:
     - Нажмите «Создать задачу», заполните форму, сохраните.
     - Задача появится в таблице.
   - **Редактирование/удаление задачи**:
     - Нажмите «Редактировать» или «Удалить» в таблице.
     - Подтвердите удаление или сохраните изменения.
   - **Админская панель (`/dashboard`)**:
     - Доступна только для ADMIN. Показывает все задачи.
   - **Выход**:
     - Вызовите `logout` через API (например, через Postman: `POST /api/auth/logout`).
     - Перенаправляет на `/login`.

   > **Комментарий**: Если получаете 401, проверьте токены в cookies (`accessToken`, `refreshToken`). Используйте DevTools браузера для отладки. Подробности: [Next.js Debugging](https://nextjs.org/docs/app/building-your-application/optimizing#debugging).

---