## Введение

Добро пожаловать в руководство по созданию Fullstack ToDo-приложения! Это приложение позволяет пользователям регистрироваться, входить в систему, создавать, редактировать, удалять и просматривать задачи. Бэкенд построен на **NestJS** с использованием **Prisma** для работы с базой данных SQLite и **JWT** для аутентификации. Фронтенд реализован на **Next.js** с **TypeScript**, **React Hook Form**, **Zod** для валидации и компонентами **Shadcn/UI** для интерфейса.

В этом гайде мы шаг за шагом создадим приложение, начиная с настройки бэкенда, а затем перейдем к разработке фронтенда. Каждая глава начинается с установки зависимостей и инициализации проекта, включает подробные комментарии к коду и ссылки на официальную документацию. Гайд ориентирован на начинающих и опытных разработчиков, желающих изучить современный fullstack-разработку.

### Цели приложения

- **Бэкенд (NestJS)**:
  - Регистрация и аутентификация пользователей с использованием JWT и refresh-токенов.
  - CRUD-операции для задач (создание, получение, обновление, удаление).
  - Поддержка ролей (USER и ADMIN) для доступа к задачам.
  - Защита маршрутов с помощью JWT-аутентификации.
- **Фронтенд (Next.js)**:
  - Интерфейс для входа, регистрации и управления задачами.
  - Форма для создания задач.
  - Таблица для отображения задач с возможностью редактирования и удаления.
  - Middleware для защиты маршрутов и обработки токенов.

### Предварительные требования

- Установлены **Node.js** (версия 18 или выше) и **npm** (Node.js документация).
- База данных **SQLite** (или другой клиент, если вы хотите заменить SQLite, например, PostgreSQL).
- Базовые знания TypeScript, NestJS (NestJS документация), Next.js (Next.js документация) и React.
- Текстовый редактор (например, VS Code).
- Утилита для тестирования API, такая как **Postman** или **Insomnia**.

---

## Глава 1: Бэкенд (NestJS)

В этой главе мы создадим бэкенд для ToDo-приложения с использованием **NestJS**, **Prisma** для работы с SQLite и **JWT** для аутентификации. Мы начнем с установки и инициализации проекта, затем настроим Prisma, создадим модуль аутентификации, модуль задач, корневой модуль и основной файл приложения. Каждый файл будет создан в последовательности, с подробными комментариями и ссылками на документацию.

### 1.1 Установка и инициализация проекта

#### Установка зависимостей

1. **Установите NestJS CLI** для создания и управления проектом:

   ```bash
   npm install -g @nestjs/cli
   ```

   > **Комментарий**: NestJS CLI упрощает создание модулей, контроллеров и сервисов. Это инструмент командной строки, который генерирует шаблоны кода и запускает проект. Подробности: NestJS CLI документация.

2. **Создайте новый проект NestJS**:

   ```bash
   nest new backend
   cd backend
   ```

   > **Комментарий**: Команда `nest new` создает структуру проекта с TypeScript, ESLint и базовым модулем `AppModule`. Это стандартный способ старта NestJS-приложения. Подробности: NestJS Getting Started.

3. **Установите зависимости** для Prisma, JWT, аутентификации, хеширования паролей и других функций:

   ```bash
   npm install @nestjs/passport passport passport-jwt @nestjs/jwt prisma @prisma/client argon2 cookie-parser @nestjs/config
   npm install -D @types/passport-jwt @types/cookie-parser
   ```

   > **Комментарий**:
   >
   > - `@nestjs/passport` и `passport-jwt` используются для аутентификации через JWT. `passport-jwt` — стратегия Passport для проверки JWT-токенов. Подробности: Passport JWT документация.
   > - `@nestjs/jwt` для генерации и проверки JWT-токенов. Подробности: NestJS JWT документация.
   > - `prisma` и `@prisma/client` для работы с базой данных через ORM. Prisma упрощает запросы к БД и миграции. Подробности: Prisma документация.
   > - `argon2` для безопасного хеширования паролей. Это рекомендованный алгоритм для защиты паролей. Подробности: Argon2 NPM.
   > - `cookie-parser` для парсинга и установки cookies в Express (NestJS использует Express по умолчанию). Подробности: cookie-parser NPM.
   > - `@nestjs/config` для работы с переменными окружения из `.env`. Подробности: NestJS Config документация.

#### Инициализация Prisma

1. **Инициализируйте Prisma** для работы с SQLite:

   ```bash
   npx prisma init
   ```

   > **Комментарий**: Команда создает файл `prisma/schema.prisma` для определения модели данных и `.env` для настройки подключения к БД. Prisma — это ORM, которая генерирует клиент для взаимодействия с базой данных. Подробности: Prisma Init.

2. **Настройте** `prisma/schema.prisma`:

   ```prisma
   // Генератор клиента Prisma для TypeScript
   generator client {
     provider = "prisma-client-js" // Генерирует Prisma Client
   }
   
   // Источник данных — SQLite
   datasource db {
     provider = "sqlite" // Используем SQLite для локальной разработки
     url      = env("DATABASE_URL") // URL подключения из .env
   }
   
   // Перечисление ролей пользователей
   enum Role {
     USER // Обычный пользователь
     ADMIN // Администратор с расширенными правами
   }
   
   // Модель пользователя
   model User {
     id          Int      @id @default(autoincrement()) // Автоинкрементный ID
     createdAt   DateTime @default(now()) @map("created_at") // Время создания
     updatedAt   DateTime @updatedAt @map("updated_at") // Время обновления
     name        String // Имя пользователя
     email       String   @unique // Уникальный email
     password    String // Хешированный пароль
     refreshToken String?  @map("refresh_token") // Refresh-токен (опционально)
     role        Role     @default(USER) // Роль по умолчанию — USER
     tasks       Task[] // Связь с задачами (один ко многим)
     @@map("users") // Имя таблицы в БД
   }
   
   // Модель задачи
   model Task {
     id          Int      @id @default(autoincrement()) // Автоинкрементный ID
     createdAt   DateTime @default(now()) @map("created_at") // Время создания
     updatedAt   DateTime @updatedAt @map("updated_at") // Время обновления
     title       String // Заголовок задачи
     description String? // Описание (опционально)
     completed   Boolean  @default(false) // Статус выполнения
     userId      Int      @map("user_id") // ID владельца задачи
     user        User     @relation(fields: [userId], references: [id]) // Связь с пользователем
     @@map("tasks") // Имя таблицы в БД
   }
   ```

   > **Комментарий**: `schema.prisma` определяет модели данных и связи. Модель `User` включает поля для аутентификации, а `Task` — для задач. `@@map` задает имена таблиц, `enum Role` — роли. Подробности: Prisma Schema.

3. **Настройте** `.env`:

   ```env
   PORT=9000
   CLIENT_URL=http://localhost:3000
   DATABASE_URL="file:./dev.db"
   
   #JWT 
   JWT_ACCESS_SECRET="access-secret-key"
   JWT_REFRESH_SECRET="refresh-secret-key"
   JWT_ACCESS_EXPIRES_IN="15m"
   JWT_REFRESH_EXPIRES_IN="7d"
   ```

   > **Комментарий**: Файл содержит конфигурацию. `DATABASE_URL` указывает на SQLite файл `dev.db`. JWT-ключи используются для токенов. Подробности: NestJS Config.

4. **Выполните миграцию базы данных**:

   ```bash
   npx prisma migrate dev --name init
   ```

   > **Комментарий**: Команда применяет схему к базе данных, создавая таблицы. `--name init` задает имя миграции. Подробности: Prisma Migrate.

### 1.2 Настройка модуля Prisma

Модуль `Prisma` предоставляет сервис для взаимодействия с базой данных.

1. **Создайте файл** `src/prisma/prisma.module.ts`:

   ```typescript
   // Импортируем зависимости NestJS
   import { Module } from '@nestjs/common';
   import { PrismaService } from './prisma.service';
   
   // Определяем модуль Prisma
   @Module({
     providers: [PrismaService], // Регистрируем PrismaService
     exports: [PrismaService], // Экспортируем для использования в других модулях
   })
   export class PrismaModule {}
   ```

   > **Комментарий**: Модуль делает `PrismaService` доступным для других модулей. `exports` позволяет импортировать сервис. Подробности: NestJS Modules.

2. **Создайте файл** `src/prisma/prisma.service.ts`:

   ```typescript
   // Импортируем зависимости NestJS и Prisma
   import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common';
   import { PrismaClient } from '@prisma/client';
   
   // Определяем сервис Prisma, расширяющий PrismaClient
   @Injectable()
   export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy {
     // Подключаемся к базе данных при инициализации модуля
     async onModuleInit() {
       await this.$connect();
     }
     // Отключаемся от базы данных при завершении работы модуля
     async onModuleDestroy() {
       await this.$disconnect();
     }
   }
   ```

   > **Комментарий**: Сервис управляет подключением к БД, автоматически подключаясь при старте и отключаясь при завершении. Это предотвращает утечки ресурсов. Подробности: Prisma Client.

### 1.3 Настройка модуля аутентификации

Модуль `Auth` отвечает за регистрацию, вход, обновление токенов и выход пользователей.

1. **Создайте файл** `src/auth/auth.module.ts`:

   ```typescript
   // Импортируем зависимости NestJS
   import { Module } from '@nestjs/common';
   import { AuthService } from './auth.service';
   import { AuthController } from './auth.controller';
   import { PrismaModule } from '../prisma/prisma.module';
   import { PassportModule } from '@nestjs/passport';
   import { JwtModule } from '@nestjs/jwt';
   import { JwtStrategy } from './strategies/jwt.strategy';
   
   // Определяем модуль аутентификации
   @Module({
     imports: [
       PrismaModule, // Для доступа к базе данных
       PassportModule, // Для интеграции Passport
       JwtModule.register({}), // Регистрируем JwtModule
     ],
     controllers: [AuthController], // Контроллер для обработки запросов
     providers: [AuthService, JwtStrategy], // Сервис и стратегия JWT
   })
   export class AuthModule {}
   ```

   > **Комментарий**: Модуль объединяет сервисы, контроллеры и зависимости для аутентификации. `JwtModule.register({})` использует конфигурацию из `.env` (NestJS Modules, NestJS Passport).

2. **Создайте файл** `src/auth/strategies/jwt.strategy.ts`:

   ```typescript
   // Импортируем зависимости
   import { Injectable } from '@nestjs/common';
   import { PassportStrategy } from '@nestjs/passport';
   import { ExtractJwt, Strategy } from 'passport-jwt';
   
   // Определяем стратегию JWT для аутентификации
   @Injectable()
   export class JwtStrategy extends PassportStrategy(Strategy) {
     constructor() {
       super({
         // Извлекаем JWT из cookies
         jwtFromRequest: ExtractJwt.fromExtractors([(req) => req.cookies['accessToken']]),
         ignoreExpiration: false, // Проверяем срок действия токена
         secretOrKey: process.env.JWT_ACCESS_SECRET, // Секрет для проверки токена
       });
     }
   
     // Валидация полезной нагрузки токена
     async validate(payload: any) {
       // Возвращаем данные пользователя для использования в запросах
       return { userId: payload.sub, email: payload.email, role: payload.role };
     }
   }
   ```

   > **Комментарий**: Стратегия извлекает `accessToken` из cookies и проверяет его. Метод `validate` возвращает данные пользователя для `req.user`. Подробности: Passport JWT.

3. **Создайте файл** `src/auth/dto/register.dto.ts`:

   ```typescript
   // Импортируем зависимости для валидации
   import { IsEmail, IsString, MinLength } from 'class-validator';
   
   // Определяем DTO для регистрации
   export class RegisterDto {
     @IsString({ message: 'Поле должно быть строкой' }) // Валидируем имя
     name: string;
   
     @IsEmail() // Валидируем email
     email: string;
   
     @IsString()
     @MinLength(6, { message: 'Минимальная длина пароля 6 символов' }) // Валидируем пароль
     password: string;
   }
   ```

   > **Комментарий**: DTO определяет структуру и валидацию данных для регистрации. `@IsString` и `@MinLength` обеспечивают проверку на сервере. Подробности: Class Validator.

4. **Создайте файл** `src/auth/dto/login.dto.ts`:

   ```typescript
   // Импортируем зависимости для валидации
   import { IsEmail, IsString, MinLength } from 'class-validator';
   
   // Определяем DTO для входа
   export class LoginDto {
     @IsEmail() // Валидируем email
     email: string;
   
     @IsString()
     @MinLength(6, { message: 'Минимальная длина пароля 6 символов' }) // Валидируем пароль
     password: string;
   }
   ```

   > **Комментарий**: DTO определяет структуру и валидацию данных для входа. Подробности: Class Validator.

5. **Создайте файл** `src/auth/auth.service.ts`:

   ```typescript
   // Импортируем зависимости
   import { Injectable, UnauthorizedException } from '@nestjs/common';
   import { PrismaService } from '../prisma/prisma.service';
   import { JwtService } from '@nestjs/jwt';
   import * as argon2 from 'argon2';
   import { RegisterDto } from './dto/register.dto';
   import { LoginDto } from './dto/login.dto';
   
   // Определяем сервис для обработки аутентификации
   @Injectable()
   export class AuthService {
     constructor(private prisma: PrismaService, private jwt: JwtService) {}
   
     // Регистрация нового пользователя
     async register(dto: RegisterDto) {
       // Хешируем пароль с использованием argon2
       const hashedPassword = await argon2.hash(dto.password);
       // Создаем пользователя в базе данных
       const user = await this.prisma.user.create({
         data: { email: dto.email, password: hashedPassword, name: dto.name },
       });
       // Генерируем токены для пользователя
       return this.generateTokens(user.id, user.email, user.role);
     }
   
     // Вход пользователя
     async login(dto: LoginDto) {
       // Ищем пользователя по email
       const user = await this.prisma.user.findUnique({ where: { email: dto.email } });
       // Проверяем наличие пользователя и корректность пароля
       if (!user || !(await argon2.verify(user.password, dto.password))) {
         throw new UnauthorizedException('Invalid credentials');
       }
       // Генерируем токены для пользователя
       return this.generateTokens(user.id, user.email, user.role);
     }
   
     // Обновление токенов
     async refresh(refreshToken: string) {
       try {
         // Проверяем валидность refresh-токена
         const payload = this.jwt.verify(refreshToken, { secret: process.env.JWT_REFRESH_SECRET });
         // Ищем пользователя по ID из токена
         const user = await this.prisma.user.findUnique({ where: { id: payload.sub } });
         if (!user) throw new UnauthorizedException();
         // Генерируем новые токены
         return this.generateTokens(user.id, user.email, user.role);
       } catch {
         throw new UnauthorizedException('Invalid refresh token');
       }
     }
   
     // Генерация access и refresh токенов
     private generateTokens(userId: number, email: string, role: string) {
       // Создаем access-токен с временем жизни 15 минут
       const accessToken = this.jwt.sign({ sub: userId, email, role }, {
         secret: process.env.JWT_ACCESS_SECRET,
         expiresIn: process.env.JWT_ACCESS_EXPIRES_IN,
       });
       // Создаем refresh-токен с временем жизни 7 дней
       const refreshToken = this.jwt.sign({ sub: userId }, {
         secret: process.env.JWT_REFRESH_SECRET,
         expiresIn: process.env.JWT_REFRESH_EXPIRES_IN,
       });
       return { accessToken, refreshToken };
     }
   }
   ```

   > **Комментарий**: Сервис обрабатывает регистрацию, вход и обновление токенов. Пароли хешируются с помощью `argon2` для безопасности. `JwtService` используется для генерации токенов. Подробности: NestJS JWT, Argon2.

6. **Создайте файл** `src/auth/auth.controller.ts`:

   ```typescript
   // Импортируем зависимости
   import { Body, Controller, Post, Res, Req, UseGuards, UnauthorizedException } from '@nestjs/common';
   import { AuthService } from './auth.service';
   import { Response, Request } from 'express';
   import { AuthGuard } from '@nestjs/passport';
   import { RegisterDto } from './dto/register.dto';
   import { LoginDto } from './dto/login.dto';
   
   // Определяем контроллер для маршрутов аутентификации
   @Controller('auth')
   export class AuthController {
     constructor(private authService: AuthService) {}
   
     // Регистрация пользователя
     @Post('register')
     async register(@Body() dto: RegisterDto, @Res() res: Response) {
       // Вызываем сервис для регистрации и получения токенов
       const tokens = await this.authService.register(dto);
       // Устанавливаем токены в cookies
       this.setCookies(res, tokens);
       // Возвращаем сообщение об успешной регистрации
       return res.json({ message: 'Registered' });
     }
   
     // Вход пользователя
     @Post('login')
     async login(@Body() dto: LoginDto, @Res() res: Response) {
       // Вызываем сервис для входа и получения токенов
       const tokens = await this.authService.login(dto);
       // Устанавливаем токены в cookies
       this.setCookies(res, tokens);
       // Возвращаем сообщение об успешном входе
       return res.json({ message: 'Logged in' });
     }
   
     // Обновление токенов
     @Post('refresh')
     async refresh(@Req() req: Request, @Res() res: Response) {
       // Извлекаем refresh-токен из cookies
       const refreshToken = req.cookies['refreshToken'];
       if (!refreshToken) throw new UnauthorizedException();
       // Вызываем сервис для обновления токенов
       const tokens = await this.authService.refresh(refreshToken);
       // Устанавливаем новые токены в cookies
       this.setCookies(res, tokens);
       // Возвращаем сообщение об успешном обновлении
       return res.json({ message: 'Tokens refreshed' });
     }
   
     // Выход пользователя
     @Post('logout')
     @UseGuards(AuthGuard('jwt')) // Защищаем маршрут JWT-аутентификацией
     async logout(@Res() res: Response) {
       // Очищаем cookies с токенами
       res.clearCookie('accessToken');
       res.clearCookie('refreshToken');
       // Возвращаем сообщение об успешном выходе
       return res.json({ message: 'Logged out' });
     }
   
     // Устанавливаем токены в cookies
     private setCookies(res: Response, tokens: { accessToken: string; refreshToken: string }) {
       res.cookie('accessToken', tokens.accessToken, { httpOnly: true, secure: false, sameSite: 'lax' });
       res.cookie('refreshToken', tokens.refreshToken, { httpOnly: true, secure: false, sameSite: 'lax' });
     }
   }
   ```

   > **Комментарий**: Контроллер обрабатывает запросы для регистрации, входа, обновления токенов и выхода. Токены сохраняются в `httpOnly` cookies для безопасности. Параметр `secure: false` используется для локальной разработки; в продакшене установите `secure: true`. Подробности: NestJS Controllers, Express Cookies.

### 1.4 Настройка модуля задач

1. **Создайте файл** `src/tasks/tasks.module.ts`:

   ```typescript
   // Импортируем зависимости
   import { Module } from '@nestjs/common';
   import { TasksService } from './tasks.service';
   import { TasksController } from './tasks.controller';
   import { PrismaModule } from '../prisma/prisma.module';
   import { PassportModule } from '@nestjs/passport';
   import { JwtModule } from '@nestjs/jwt';
   
   // Определяем модуль задач
   @Module({
     imports: [
       PrismaModule, // Для доступа к базе данных
       PassportModule, // Для аутентификации
       JwtModule, // Для работы с JWT
     ],
     controllers: [TasksController], // Контроллер задач
     providers: [TasksService], // Сервис задач
   })
   export class TasksModule {}
   ```

   > **Комментарий**: Модуль объединяет сервисы и контроллеры для задач, подключая зависимости для Prisma и JWT. Подробности: NestJS Modules.

2. **Создайте файл** `src/tasks/dto/create.dto.ts`:

   ```typescript
   // Импортируем зависимости для валидации
   import { IsNotEmpty, IsOptional, IsString } from 'class-validator';
   
   // Определяем DTO для создания задачи
   export class CreateDto {
     @IsString() @IsNotEmpty() // Валидируем заголовок
     title: string;
   
     @IsString() @IsOptional() // Описание опционально
     description?: string;
   }
   ```

   > **Комментарий**: DTO определяет структуру и валидацию данных для создания задачи. Подробности: Class Validator.

3. **Создайте файл** `src/tasks/dto/update.dto.ts`:

   ```typescript
   // Импортируем зависимости для валидации
   import { IsBoolean, IsOptional, IsString } from 'class-validator';
   
   // Определяем DTO для обновления задачи
   export class UpdateDto {
     @IsString() @IsOptional() // Заголовок опционален
     title?: string;
   
     @IsString() @IsOptional() // Описание опционально
     description?: string;
   
     @IsBoolean() @IsOptional() // Статус выполнения опционален
     completed?: boolean;
   }
   ```

   > **Комментарий**: DTO определяет структуру и валидацию данных для обновления задачи. Подробности: Class Validator.

4. **Создайте файл** `src/tasks/tasks.service.ts`:

   ```typescript
   // Импортируем зависимости
   import { ForbiddenException, Injectable } from '@nestjs/common';
   import { PrismaService } from '../prisma/prisma.service';
   import { CreateDto } from './dto/create.dto';
   import { UpdateDto } from './dto/update.dto';
   
   // Определяем сервис для работы с задачами
   @Injectable()
   export class TasksService {
     constructor(private readonly prismaService: PrismaService) {}
   
     // Создание новой задачи
     async create(userId: number, dto: CreateDto) {
       // Создаем задачу, связывая ее с пользователем
       return this.prismaService.task.create({
         data: { ...dto, userId },
       });
     }
   
     // Получение всех задач
     async findAll(userId: number, role: string) {
       // Если пользователь — ADMIN, возвращаем все задачи
       if (role === 'ADMIN') return this.prismaService.task.findMany();
       // Иначе возвращаем только задачи пользователя
       return this.prismaService.task.findMany({ where: { userId }, include: { user: true } });
     }
   
     // Получение одной задачи
     async findOne(id: number, userId: number, role: string) {
       // Ищем задачу по ID
       const task = await this.prismaService.task.findUnique({ where: { id } });
       // Проверяем, существует ли задача и имеет ли пользователь доступ
       if (!task || (task.userId !== userId && role !== 'ADMIN')) {
         throw new ForbiddenException('Access denied');
       }
       return task;
     }
   
     // Обновление задачи
     async update(id: number, userId: number, role: string, dto: UpdateDto) {
       // Проверяем доступ к задаче
       await this.findOne(id, userId, role);
       // Обновляем задачу
       return this.prismaService.task.update({ where: { id }, data: dto });
     }
   
     // Удаление задачи
     async delete(id: number, userId: number, role: string) {
       // Проверяем доступ к задаче
       await this.findOne(id, userId, role);
       // Удаляем задачу
       return this.prismaService.task.delete({ where: { id } });
     }
   }
   ```

   > **Комментарий**: Сервис реализует CRUD-операции с учетом ролей. ADMIN имеет доступ ко всем задачам, USER — только к своим. `findOne` используется для проверки доступа перед обновлением и удалением. Подробности: Prisma Client.

5. **Создайте файл** `src/tasks/tasks.controller.ts`:

   ```typescript
   // Импортируем зависимости
   import { Body, Controller, Delete, Get, Param, Post, Put, Req, UseGuards } from '@nestjs/common';
   import { TasksService } from './tasks.service';
   import { CreateDto } from './dto/create.dto';
   import { UpdateDto } from './dto/update.dto';
   import { AuthGuard } from '@nestjs/passport';
   
   // Определяем контроллер для маршрутов задач
   @Controller('tasks')
   @UseGuards(AuthGuard('jwt')) // Защищаем все маршруты JWT-аутентификацией
   export class TasksController {
     constructor(private readonly tasksService: TasksService) {}
   
     // Создание задачи
     @Post()
     create(@Req() req, @Body() dto: CreateDto) {
       return this.tasksService.create(req.user.userId, dto);
     }
   
     // Получение всех задач
     @Get()
     findAll(@Req() req) {
       return this.tasksService.findAll(req.user.userId, req.user.role);
     }
   
     // Получение одной задачи
     @Get(':id')
     findOne(@Req() req, @Param('id') id: string) {
       return this.tasksService.findOne(+id, req.user.userId, req.user.role);
     }
   
     // Обновление задачи
     @Put(':id')
     update(@Req() req, @Param('id') id: string, @Body() dto: UpdateDto) {
       return this.tasksService.update(+id, req.user.userId, req.user.role, dto);
     }
   
     // Удаление задачи
     @Delete(':id')
     delete(@Req() req, @Param('id') id: string) {
       return this.tasksService.delete(+id, req.user.userId, req.user.role);
     }
   }
   ```

   > **Комментарий**: Контроллер обрабатывает запросы для задач, используя `AuthGuard('jwt')` для защиты. Данные пользователя (`userId`, `role`) извлекаются из `req.user`. Подробности: NestJS Controllers, NestJS Guards.

### 1.5 Настройка основного приложения

1. **Создайте файл** `src/app.module.ts`:

   ```typescript
   // Импортируем зависимости NestJS
   import { Module } from '@nestjs/common';
   import { AppController } from './app.controller';
   import { AppService } from './app.service';
   import { PrismaModule } from './prisma/prisma.module';
   import { AuthModule } from './auth/auth.module';
   import { TasksModule } from './tasks/tasks.module';
   import { ConfigModule } from '@nestjs/config';
   
   // Определяем корневой модуль приложения
   @Module({
     imports: [
       ConfigModule.forRoot({ isGlobal: true }), // Загружаем конфигурацию из .env
       PrismaModule, // Модуль Prisma
       AuthModule, // Модуль аутентификации
       TasksModule, // Модуль задач
     ],
     controllers: [AppController], // Пустой контроллер по умолчанию
     providers: [AppService], // Пустой сервис по умолчанию
   })
   export class AppModule {}
   ```

   > **Комментарий**: `AppModule` объединяет все модули приложения. `ConfigModule.forRoot({ isGlobal: true })` делает переменные из `.env` доступными во всём приложении. Подробности: NestJS Modules, NestJS Config).

2. **Настройте файл** `src/main.ts`:

   ```typescript
   // Импортируем основные зависимости NestJS
   import { NestFactory } from '@nestjs/core';
   import { AppModule } from './app.module';
   import * as cookieParser from 'cookie-parser';
   import { ValidationPipe } from '@nestjs/common';
   
   // Определяем асинхронную функцию для запуска приложения
   async function bootstrap() {
     // Создаем экземпляр приложения
     const app = await NestFactory.create(AppModule);
     // Включаем CORS для взаимодействия с фронтендом
     app.enableCors({
       origin: process.env.CLIENT_URL || 'http://localhost:3000', // Разрешаем запросы с фронтенда
       credentials: true, // Разрешаем отправку cookies
     });
     // Применяем ValidationPipe для валидации входящих данных
     app.useGlobalPipes(new ValidationPipe());
     // Устанавливаем глобальный префикс 'api' для всех маршрутов
     app.setGlobalPrefix('api');
     // Подключаем cookie-parser для обработки cookies
     app.use(cookieParser());
     // Запускаем сервер на порту из .env или 3000
     await app.listen(process.env.PORT ?? 3000);
   }
   // Запускаем приложение
   bootstrap();
   ```

   > **Комментарий**: Этот файл — точка входа приложения. Он настраивает CORS, валидацию, префикс маршрутов и cookies. `ValidationPipe` автоматически проверяет DTO. Подробности: NestJS Pipes, NestJS Middleware.

### 1.6 Тестирование бэкенда

1. **Запустите бэкенд**:

   ```bash
   npm run start:dev
   ```

   > **Комментарий**: Сервер запустится на порту 9000 (или другом, указанном в `.env`) с горячей перезагрузкой для разработки. Подробности: NestJS CLI.

2. **Протестируйте API** с помощью **Postman** или **Insomnia**:

   - **POST /api/auth/register**: `{ "name": "Test User", "email": "test@example.com", "password": "password123" }`
     - Ожидаемый ответ: `{ "message": "Registered" }`, cookies с `accessToken` и `refreshToken`.
   - **POST /api/auth/login**: `{ "email": "test@example.com", "password": "password123" }`
     - Ожидаемый ответ: `{ "message": "Logged in" }`, cookies с токенами.
   - **POST /api/auth/refresh**: Отправьте запрос с `refreshToken` в cookies.
     - Ожидаемый ответ: `{ "message": "Tokens refreshed" }`, новые cookies.
   - **POST /api/auth/logout**: Отправьте запрос с `accessToken` в cookies.
     - Ожидаемый ответ: `{ "message": "Logged out" }`, cookies очищены.
   - **POST /api/tasks**: `{ "title": "Test Task", "description": "Description" }`
     - Ожидаемый ответ: объект задачи.
   - **GET /api/tasks**: Получите список задач пользователя (или всех для ADMIN).
   - **GET /api/tasks/:id**: Получите задачу по ID.
   - **PUT /api/tasks/:id**: `{ "title": "Updated Task", "completed": true }`
   - **DELETE /api/tasks/:id**: Удалите задачу.

   > **Комментарий**: Убедитесь, что `accessToken` отправляется в cookies для защищенных маршрутов. Если вы получаете 401, проверьте токен. Подробности: Postman.

---