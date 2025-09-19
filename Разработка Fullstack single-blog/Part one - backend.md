# Создание Backend для Блога на Nest.js: Полный Пошаговый Гайд с Комментариями в Коде

## Введение

Дорогие читатели! Я рад приветствовать вас в обновленной версии этой книги. По вашим просьбам, я переписал весь гайд заново, добавив подробные комментарии прямо в код (используя // для TypeScript/JS и # для Prisma/Env). Это поможет лучше понять логику внутри файлов. Я также сохранил и расширил пояснения для каждой строки или блока, не пропустив ни один файл из структуры проекта. Для каждого файла: сначала описание, зачем он нужен, как установить/создать; затем код с комментариями; затем детальный разбор строк с ссылками на официальную документацию, где применимо.

Мы строим тот же backend для блога, но теперь с акцентом на читаемость кода. Если вы следуете шагам, соберите проект поэтапно. Уровень: для начинающих/средних разработчиков.

### Предварительные требования
- Node.js 18+ ([скачать](https://nodejs.org/)).
- npm.
- VS Code или аналог.
- Знания TS/JS.

Начинаем!

## Глава 1: Установка и Базовая Конфигурация

### Шаг 1.1: Установка Nest.js CLI
Nest.js — фреймворк для Node.js, использующий TS, DI и модули. Docs: [Nest.js Overview](https://docs.nestjs.com/).

Установите:
```
npm install -g @nestjs/cli
```

### Шаг 1.2: Создание Проекта
```
nest new backend
cd backend
```

Это создаст структуру, включая src/, package.json и т.д.

### Шаг 1.3: Установка Библиотек
Установите все зависимости (описание каждой):

- `@nestjs/config`: Для .env. Docs: [Configuration](https://docs.nestjs.com/techniques/configuration). `npm i @nestjs/config`
- `@nestjs/jwt`: Для JWT. Docs: [Authentication](https://docs.nestjs.com/security/authentication). `npm i @nestjs/jwt`
- `@nestjs/passport passport passport-jwt`: Для аутентификации. Docs: [Passport](https://docs.nestjs.com/security/authentication#implementing-passport-strategies). `npm i @nestjs/passport passport passport-jwt`
- `@nestjs/serve-static`: Для статических файлов. Docs: [Serve Static](https://docs.nestjs.com/techniques/serve-static). `npm i @nestjs/serve-static`
- `@nestjs/platform-express multer`: Для загрузки файлов. Docs: [File Upload](https://docs.nestjs.com/techniques/file-upload). `npm i @nestjs/platform-express multer`
- `prisma @prisma/client`: ORM. Docs: [Prisma](https://www.prisma.io/docs). `npx prisma init; npm i @prisma/client`
- `argon2`: Хэширование. Docs: [Argon2](https://github.com/ranisalt/node-argon2). `npm i argon2`
- `class-validator class-transformer`: Валидация. Docs: [Validation](https://docs.nestjs.com/pipes#class-validator). `npm i class-validator class-transformer`
- `slugify`: Слаги. Docs: [Slugify](https://github.com/simov/slugify). `npm i slugify`
- `uuid`: UUID. Docs: [UUID](https://github.com/uuidjs/uuid). `npm i uuid`
- `cookie-parser`: Куки. Docs: [Cookie-Parser](https://www.npmjs.com/package/cookie-parser). `npm i cookie-parser`
- Dev: `@types/multer @types/uuid` и т.д. `npm i -D @types/multer @types/uuid`

### Шаг 1.4: package.json
Этот файл — манифест проекта. Он генерируется, но мы добавим комментарии (хотя JSON не поддерживает, я покажу с // для ясности; в реальности используйте отдельный файл для docs).

Код (с реконструированными комментариями как в коде):
```json
{
  // Имя проекта, используется в импортах
  "name": "backend",
  // Версия для семвер
  "version": "0.0.1",
  // Описание проекта
  "description": "",
  // Автор
  "author": "",
  // Приватный, не публикуется в npm
  "private": true,
  // Лицензия
  "license": "UNLICENSED",
  // Скрипты для запуска/билда
  "scripts": {
    // Билд в dist/
    "build": "nest build",
    // Форматирование кода
    "format": "prettier --write \"src/**/*.ts\" \"test/**/*.ts\"",
    // Запуск
    "start": "nest start",
    // Dev с watch
    "start:dev": "nest start --watch",
    // Debug режим
    "start:debug": "nest start --debug --watch",
    // Prod запуск
    "start:prod": "node dist/main",
    // Линтинг с фиксом
    "lint": "eslint \"{src,apps,libs,test}/**/*.ts\" --fix",
    // Тесты
    "test": "jest",
    // Watch тесты
    "test:watch": "jest --watch",
    // Тесты с coverage
    "test:cov": "jest --coverage",
    // Debug тесты
    "test:debug": "node --inspect-brk -r tsconfig-paths/register -r ts-node/register node_modules/.bin/jest --runInBand",
    // E2E тесты
    "test:e2e": "jest --config ./test/jest-e2e.json"
  },
  // Основные зависимости
  "dependencies": {
    "@nestjs/common": "^10.0.0",
    "@nestjs/config": "^3.2.3",
    "@nestjs/core": "^10.0.0",
    "@nestjs/jwt": "^10.2.0",
    "@nestjs/passport": "^10.0.3",
    "@nestjs/platform-express": "^10.4.1",
    "@nestjs/serve-static": "^4.0.0",
    "@prisma/client": "^5.18.0",
    "argon2": "^0.40.3",
    "class-transformer": "^0.5.1",
    "class-validator": "^0.14.1",
    "cookie-parser": "^1.4.6",
    "passport": "^0.7.0",
    "passport-jwt": "^4.0.1",
    "reflect-metadata": "^0.2.0",
    "rxjs": "^7.8.1",
    "slugify": "^1.6.6",
    "uuid": "^10.0.0"
  },
  // Dev зависимости
  "devDependencies": {
    "@nestjs/cli": "^10.0.0",
    "@nestjs/schematics": "^10.0.0",
    "@nestjs/testing": "^10.0.0",
    "@types/express": "^4.17.17",
    "@types/jest": "^29.5.2",
    "@types/multer": "^1.4.11",
    "@types/node": "^20.3.3",
    "@types/supertest": "^6.0.0",
    "@types/uuid": "^9.0.8",
    "@typescript-eslint/eslint-plugin": "^6.0.0",
    "@typescript-eslint/parser": "^6.0.0",
    "eslint": "^8.42.0",
    "eslint-config-prettier": "^9.0.0",
    "eslint-plugin-prettier": "^5.0.0",
    "jest": "^29.5.0",
    "prettier": "^3.0.0",
    "prisma": "^5.18.0",
    "source-map-support": "^0.5.21",
    "supertest": "^6.3.3",
    "ts-jest": "^29.1.0",
    "ts-loader": "^9.4.3",
    "ts-node": "^10.9.1",
    "tsconfig-paths": "^4.2.0",
    "typescript": "^5.1.3"
  },
  // Конфиг Jest
  "jest": {
    // Расширения файлов
    "moduleFileExtensions": [
      "js",
      "json",
      "ts"
    ],
    // Корень тестов
    "rootDir": "src",
    // Regex для тестов
    "testRegex": ".*\\.spec\\.ts$",
    // Трансформер
    "transform": {
      "^.+\\.(t|j)s$": "ts-jest"
    },
    // Coverage из
    "collectCoverageFrom": [
      "**/*.(t|j)s"
    ],
    // Директория coverage
    "coverageDirectory": "../coverage",
    // Окружение
    "testEnvironment": "node"
  }
}
```

**Детальный разбор строк:**
- `"name": "backend"`: Имя проекта. Docs: [npm package.json](https://docs.npmjs.com/cli/v10/configuring-npm/package-json).
- `"scripts"`: Команды, например `"start:dev"`: Запуск с наблюдением. Docs: [npm scripts](https://docs.npmjs.com/cli/v10/using-npm/scripts).
- `"dependencies"`: Пакеты для runtime, каждый описан выше.
- `"devDependencies"`: Для dev, как Jest для тестов. Docs: [Nest.js Testing](https://docs.nestjs.com/fundamentals/testing).
- `"jest"`: Конфиг тестов. Docs: [Jest Config](https://jestjs.io/docs/configuration).

package-lock.json: Автоматически генерируется `npm install`, фиксирует версии. Не редактируйте вручную. Docs: [package-lock.json](https://docs.npmjs.com/cli/v10/configuring-npm/package-lock-json).

### Шаг 1.5: tsconfig.json
Конфиг TS.

Код с комментариями (JSON не поддерживает, но для гайда):
```json
{
  // Опции компилятора
  "compilerOptions": {
    // Модульная система
    "module": "commonjs",
    // Генерация деклараций
    "declaration": true,
    // Удаление комментариев
    "removeComments": true,
    // Метаданные декораторов
    "emitDecoratorMetadata": true,
    // Экспериментальные декораторы
    "experimentalDecorators": true,
    // Синтетические импорты
    "allowSyntheticDefaultImports": true,
    // Таргет JS
    "target": "ES2021",
    // Source maps
    "sourceMap": true,
    // Выходная директория
    "outDir": "./dist",
    // Базовый URL
    "baseUrl": "./",
    // Инкрементальный билд
    "incremental": true,
    // Пропуск lib проверок
    "skipLibCheck": true,
    // Строгие null
    "strictNullChecks": false,
    // Нет implicit any
    "noImplicitAny": false,
    // Строгие bind/call
    "strictBindCallApply": false,
    // Consistent casing
    "forceConsistentCasingInFileNames": false,
    // Нет fallthrough в switch
    "noFallthroughCasesInSwitch": false
  }
}
```

**Разбор:**
- `"experimentalDecorators": true`: Для Nest декораторов. Docs: [TS Decorators](https://www.typescriptlang.org/docs/handbook/decorators.html).
- `"outDir": "./dist"`: Билд в dist.

tsconfig.build.json: Extends tsconfig.json для prod, исключает тесты.

Код:
```json
{
  "extends": "./tsconfig.json",
  "exclude": ["node_modules", "test", "dist", "**/*spec.ts"]
}
```

**Разбор:**
- `"extends"`: Наследует от tsconfig.json. Docs: [TS Config Extends](https://www.typescriptlang.org/tsconfig#extends).
- `"exclude"`: Исключает файлы.

### Шаг 1.6: nest-cli.json
Конфиг CLI.

Код:
```json
{
  // Схема
  "$schema": "https://json.schemastore.org/nest-cli",
  // Коллекция схем
  "collection": "@nestjs/schematics",
  // Корень исходников
  "sourceRoot": "src"
}
```

**Разбор:**
- `"collection"`: Для генерации. Docs: [Nest CLI](https://docs.nestjs.com/cli/usages#nest-generate).

### Шаг 1.7: eslint.config.mjs
Линтинг.

Код с комментариями:
```js
// Импорт базового ESLint
import js from '@eslint/js';
// Импорт TS ESLint
import tseslint from 'typescript-eslint';

// Экспорт конфига
export default tseslint.config(
  // Рекомендованный JS
  js.configs.recommended, 
  // Рекомендованный TS
  ...tseslint.configs.recommended
);
```

**Разбор:**
- Импорты: Базовые конфиги. Docs: [ESLint Config](https://eslint.org/docs/latest/use/configure/), [TS-ESLint](https://typescript-eslint.io/getting-started).

### Шаг 1.8: README.md
Описание проекта.

Код:
```
# Backend Blog

## Installation
npm install

## Run
npm run start:dev

// Дополнительно: описание функционала
This is a Nest.js backend for a blog with auth, posts, categories.
```

**Разбор:**
- Просто markdown. Добавьте свои инструкции.

Создайте uploads/ для файлов.

## Глава 2: Prisma и БД

### Шаг 2.1: .env
Секреты.

Код с комментариями:
```
# Порт сервера
PORT=9000
# URL клиента для CORS
CLIENT_URL="http://localhost:3000"
# URL БД
DATABASE_URL="file:./dev.db"
# Директория uploads
UPLOAD_DIR=uploads

# JWT настройки
JWT_ACCESS_SECRET="access-secret-key"
JWT_REFRESH_SECRET="refresh-secret-key"
JWT_ACCESS_EXPIRES_IN="15m"
JWT_REFRESH_EXPIRES_IN="7d"

# Куки настройки
COOKIE_SECURE="false"
COOKIE_SAME_SITE="lax"
```

**Разбор:**
- Каждая переменная используется в коде. Docs: [Nest Config](https://docs.nestjs.com/techniques/configuration#using-the-configservice).

### Шаг 2.2: schema.prisma
Схема БД.

Код с комментариями:
```prisma
// Генератор клиента
generator client {
  provider = "prisma-client-js" // JS клиент
}

// Источник данных
datasource db {
  provider = "sqlite" // SQLite
  url      = env("DATABASE_URL") // Из .env
}

// Модель пользователя
model User {
  id        Int      @id @default(autoincrement()) // PK, авто
  createdAt DateTime @default(now()) @map("created_at") // Создание
  updatedAt DateTime @updatedAt @map("updated_at") // Обновление

  name         String // Имя
  email        String  @unique // Уникальный email
  password     String // Пароль хэш
  refreshToken String? @map("refresh_token") // Refresh хэш
  role         String  @default("user") // Роль

  posts     Post[] // Связь с постами
  favorites Favorite[] // С избранным

  @@map("users") // Имя таблицы
}

// Модель категории
model Category {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  name String @unique // Уникальное имя

  posts Post[] // Связь

  @@map("categories")
}

// Модель поста
model Post {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  title     String // Заголовок
  slug      String  @unique // Слаг
  content   String // Содержимое
  imagePath String? // Путь изображения

  favorites Favorite[] // Избранное

  category   Category? @relation(fields: [categoryId], references: [id]) // Связь категория
  categoryId Int?      @map("category_id")
  user       User?     @relation(fields: [userId], references: [id]) // Связь пользователь
  userId     Int?      @map("user_id")

  @@map("posts")
}

// Модель избранного
model Favorite {
  id        Int      @id @default(autoincrement())
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")

  post   Post? @relation(fields: [postId], references: [id]) // Связь пост
  postId Int?  @map("post_id")

  user   User? @relation(fields: [userId], references: [id]) // Связь пользователь
  userId Int?  @map("user_id")

  @@map("favorites")
}
```

**Разбор:**
- `generator client`: Клиент. Docs: [Generators](https://www.prisma.io/docs/concepts/components/prisma-schema/generators).
- `datasource db`: БД. Docs: [Data Sources](https://www.prisma.io/docs/concepts/components/prisma-schema/data-sources).
- Модели: Поля, атрибуты (@id, @default, @relation). Docs: [Models](https://www.prisma.io/docs/concepts/components/prisma-schema/models).

Запустите миграцию: `npx prisma migrate dev --name init`. Это создаст migrations/20250907123046_init/migration.sql (SQL скрипт), migration_lock.toml (лок) и dev.db (БД файл).

## Глава 3: Prisma Модуль

Генерируйте: `nest g module prisma`

### Шаг 3.1: prisma.module.ts
Код с комментариями:
```ts
import { Module } from '@nestjs/common'; // Импорт модуля
import { PrismaService } from './prisma.service'; // Сервис

@Module({
  providers: [PrismaService], // Провайдеры
  exports: [PrismaService], // Экспорты для других модулей
})
export class PrismaModule {} // Класс модуля
```

**Разбор:**
- `@Module({})`: Определяет модуль. Docs: [Modules](https://docs.nestjs.com/modules).

### Шаг 3.2: prisma.service.ts
Код с комментариями:
```ts
import { Injectable, OnModuleInit, OnModuleDestroy } from '@nestjs/common'; // Декораторы и интерфейсы
import { PrismaClient } from '@prisma/client'; // Клиент Prisma

@Injectable() // Инжектируемый
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy { // Расширение клиента
  async onModuleInit() { // Инициализация
    await this.$connect(); // Подключение к БД
  }
  async onModuleDestroy() { // Уничтожение
    await this.$disconnect(); // Отключение
  }
}
```

**Разбор:**
- `extends PrismaClient`: Наследует. Docs: [Prisma Client](https://www.prisma.io/docs/concepts/components/prisma-client).
- `onModuleInit`: Хук. Docs: [Lifecycle](https://docs.nestjs.com/fundamentals/lifecycle-events).

## Глава 4: Auth Модуль

Генерируйте: `nest g module auth; nest g controller auth; nest g service auth`

Создайте dto/ и strategies/.

### Шаг 4.1: auth.module.ts
Код с комментариями:
```ts
import { Module } from '@nestjs/common'; // Модуль
import { AuthService } from './auth.service'; // Сервис
import { AuthController } from './auth.controller'; // Контроллер
import { PrismaModule } from '../prisma/prisma.module'; // Prisma
import { JwtModule } from '@nestjs/jwt'; // JWT
import { ConfigModule, ConfigService } from '@nestjs/config'; // Конфиг
import { PassportModule } from '@nestjs/passport'; // Passport
import { JwtStrategy } from './strategies/jwt.strategy'; // Стратегия access
import { JwtRefreshStrategy } from './strategies/jwt-refresh.strategy'; // Стратегия refresh (исправил опечатку)

@Module({
  imports: [
    PrismaModule, // Для БД
    PassportModule, // Для аутентификации
    JwtModule.registerAsync({ // Асинхронная регистрация JWT
      imports: [ConfigModule], // Импорт конфига
      useFactory: async (configService: ConfigService) => ({ // Фабрика
        secret: configService.getOrThrow<string>('JWT_ACCESS_SECRET'), // Секрет
        signOptions: { expiresIn: configService.getOrThrow<string>('JWT_ACCESS_EXPIRES_IN') }, // Опции
      }),
      inject: [ConfigService], // Инъекция
    }),
  ],
  controllers: [AuthController], // Контроллер
  providers: [AuthService, JwtStrategy, JwtRefreshStrategy], // Провайдеры
})
export class AuthModule {} // Модуль
```

**Разбор:**
- `JwtModule.registerAsync`: Асинхронно. Docs: [Dynamic Modules](https://docs.nestjs.com/fundamentals/dynamic-modules).

### Шаг 4.2: login.dto.ts
Код с комментариями:
```ts
import { IsEmail, IsString, MinLength } from 'class-validator'; // Валидаторы

export class LoginDto { // DTO для логина
  @IsEmail() // Должен быть email
  email: string;

  @IsString() // Строка
  @MinLength(6, { message: "Минимальная длина пароля 6 символов" }) // Мин длина
  password: string;
}
```

**Разбор:**
- Декораторы валидации. Docs: [Class-Validator](https://github.com/typestack/class-validator#usage).

register.dto.ts: Аналогично.

Код:
```ts
import { IsEmail, IsString, MinLength } from 'class-validator'; // Валидаторы

export class RegisterDto { // DTO для регистрации
  @IsString() // Строка
  name: string;

  @IsEmail() // Email
  email: string;

  @IsString() // Строка
  @MinLength(6, { message: "Минимальная длина пароля 6 символов" }) // Мин длина
  password: string;
}
```

### Шаг 4.3: jwt.strategy.ts
Код с комментариями:
```ts
import { ExtractJwt, Strategy } from 'passport-jwt'; // JWT из Passport
import { PassportStrategy } from '@nestjs/passport'; // Nest Passport
import { Injectable } from '@nestjs/common'; // Injectable
import { ConfigService } from '@nestjs/config'; // Конфиг
import { PrismaService } from '../../prisma/prisma.service'; // Prisma
import { Request } from 'express'; // Request тип

@Injectable() // Инжектируемый
export class JwtStrategy extends PassportStrategy(Strategy, 'jwt') { // Стратегия JWT
  constructor(private configService: ConfigService, private prismaService: PrismaService) { // Конструктор
    super({ // Конфиг
      jwtFromRequest: ExtractJwt.fromExtractors([ // Извлечение из куки
        (req: Request) => req.cookies?.['access-token'] // Куки access-token
      ]),
      secretOrKey: configService.getOrThrow<string>('JWT_ACCESS_SECRET'), // Секрет
    });
  }

  async validate(payload: { sub: number, email: string, role: string }) { // Валидация payload
    const user = await this.prismaService.user.findUnique({ // Находим пользователя
      where: { id: payload.sub }, // По ID
    });

    if (!user) { // Если нет
      return null;
    }

    return { id: user.id, email: user.email, role: user.role }; // Возвращаем объект
  }
}
```

**Разбор:**
- `PassportStrategy`: Расширение. Docs: [Passport JWT](https://docs.nestjs.com/security/authentication#jwt-functionality).
- `ExtractJwt.fromExtractors`: Из куки.

jwt-refresh.strategy.ts: Аналогично, с проверкой хэша.

Код (исправил straregy на strategy):
```ts
import { ExtractJwt, Strategy } from 'passport-jwt'; // JWT
import { PassportStrategy } from '@nestjs/passport'; // Nest
import { Injectable } from '@nestjs/common'; // Injectable
import { ConfigService } from '@nestjs/config'; // Конфиг
import { PrismaService } from '../../prisma/prisma.service'; // Prisma
import { Request } from 'express'; // Request
import * as argon2 from 'argon2'; // Argon2

@Injectable() // Инжектируемый
export class JwtRefreshStrategy extends PassportStrategy(Strategy, 'jwt-refresh') { // Стратегия refresh
  constructor(private configService: ConfigService, private prismaService: PrismaService) { // Конструктор
    super({ // Конфиг
      jwtFromRequest: ExtractJwt.fromExtractors([(req: Request)=> req.cookies?.['refresh-token']]), // Из куки refresh
      passReqToCallback: true, // Передача req в validate
      secretOrKey: configService.getOrThrow<string>("JWT_REFRESH_SECRET"), // Секрет
    });
  }

  async validate(req: Request, payload: { sub: number, email: string, role: string }) { // Валидация
    const user = await this.prismaService.user.findUniqueOrThrow({ where: { id: payload.sub } }); // Находим пользователя
    if (!user || !user.refreshToken) return null; // Если нет токена

    const refreshTokenFromCookie = req.cookies?.['refresh-token']; // Куки
    if (!refreshTokenFromCookie) return null; // Если нет

    if(!(await argon2.verify(user.refreshToken, refreshTokenFromCookie))) return null; // Проверка хэша

    return { id: user.id, email: user.email, role: user.role }; // Возврат
  }
}
```

### Шаг 4.4: auth.controller.ts
Код с комментариями:
```ts
import { Body, Controller, Post, Req, Res, UseGuards } from '@nestjs/common'; // Декораторы
import { AuthService } from './auth.service'; // Сервис
import { RegisterDto } from './dto/register.dto'; // DTO
import { Response, Request } from 'express'; // Типы
import { LoginDto } from './dto/login.dto'; // DTO
import { AuthGuard } from '@nestjs/passport'; // Guard

@Controller('auth') // Контроллер /auth
export class AuthController {
  constructor(private readonly authService: AuthService) {} // Инъекция

  @Post('register') // POST /auth/register
  async register(@Body() dto: RegisterDto, @Res() res: Response) { // Регистрация
    return this.authService.register(dto, res); // Вызов сервиса
  }

  @Post('login') // POST /auth/login
  async login(@Body() dto: LoginDto, @Res() res: Response) { // Логин
    return this.authService.login(dto, res);
  }

  @Post('logout') // POST /auth/logout
  @UseGuards(AuthGuard('jwt')) // Guard JWT
  async logout(@Req() req: Request, @Res() res: Response) { // Логаут
    const user = req.user as { id: number }; // User из req
    return this.authService.logout(user.id, res);
  }

  @Post('refresh') // POST /auth/refresh
  @UseGuards(AuthGuard('jwt-refresh')) // Guard refresh
  async refresh(@Req() req: Request, @Res() res: Response) { // Refresh
    return this.authService.refresh(req.user, res);
  }
}
```

**Разбор:**
- `@Controller('auth')`: Префикс. Docs: [Controllers](https://docs.nestjs.com/controllers).
- `@UseGuards`: Guards. Docs: [Guards](https://docs.nestjs.com/guards).

### Шаг 4.5: auth.service.ts
Код с комментариями:
```ts
import { BadRequestException, Injectable, UnauthorizedException } from '@nestjs/common'; // Исключения
import { PrismaService } from '../prisma/prisma.service'; // Prisma
import { JwtService } from '@nestjs/jwt'; // JWT
import { ConfigService } from '@nestjs/config'; // Конфиг
import { Response } from 'express'; // Response
import * as argon2 from 'argon2'; // Argon2
import { RegisterDto } from './dto/register.dto'; // DTO
import { LoginDto } from './dto/login.dto'; // DTO

@Injectable() // Инжектируемый
export class AuthService {
  constructor( // Инъекции
    private readonly prismaService: PrismaService,
    private readonly jwtService: JwtService,
    private readonly configService: ConfigService,
  ) {}

  async register(dto: RegisterDto, res: Response) { // Регистрация
    const existingUser = await this.prismaService.user.findUnique({ where: { email: dto.email } }); // Проверка существования
    if (existingUser) throw new BadRequestException('Пользователь с таким Email уже существует!'); // Ошибка

    const hashPassword = await argon2.hash(dto.password); // Хэш пароля
    const user = await this.prismaService.user.create({ // Создание
      data: { name: dto.name, email: dto.email, password: hashPassword },
    });

    return this.generateTokenToCookie(user.id, user.role, user.email, res); // Генерация токенов
  }

  async login(dto: LoginDto, res: Response) { // Логин
    const user = await this.prismaService.user.findUnique({ where: { email: dto.email } }); // Поиск
    if (!user || !(await argon2.verify(user.password, dto.password))) { // Проверка
      throw new UnauthorizedException('Неверный логин и/или пароль');
    }

    return this.generateTokenToCookie(user.id, user.role, user.email, res); // Токены
  }

  async logout(userId: number, res: Response) { // Логаут
    await this.prismaService.user.update({ // Обновление
      where: { id: userId },
      data: { refreshToken: null },
    });

    res.clearCookie('access-token'); // Очистка куки
    res.clearCookie('refresh-token');
    return res.json({ message: 'Выход выполнен' }); // Ответ
  }

  async refresh(user: any, res: Response) { // Refresh
    return this.generateTokenToCookie(user.id, user.role, user.email, res); // Новые токены
  }

  private async generateTokenToCookie(userId: number, role: string, email: string, res: Response) { // Хелпер
    const accessToken = this.jwtService.sign({ sub: userId, role, email }, { // Access токен
      secret: this.configService.getOrThrow<string>('JWT_ACCESS_SECRET'),
      expiresIn: this.configService.getOrThrow<string>('JWT_ACCESS_EXPIRES_IN'),
    });

    const refreshToken = this.jwtService.sign({ sub: userId, role, email }, { // Refresh токен
      secret: this.configService.getOrThrow<string>('JWT_REFRESH_SECRET'),
      expiresIn: this.configService.getOrThrow<string>('JWT_REFRESH_EXPIRES_IN'),
    });

    const hashRefreshToken = await argon2.hash(refreshToken); // Хэш
    await this.prismaService.user.update({ where: { id: userId }, data: { refreshToken: hashRefreshToken } }); // Сохранение

    res.cookie('access-token', accessToken, { // Куки access
      httpOnly: true,
      maxAge: 15 * 60 * 1000,
      secure: this.configService.get<string>('COOKIE_SECURE') === "true",
      sameSite: this.configService.getOrThrow<string>('COOKIE_SAME_SITE') as "lax" | "strict",
    });

    res.cookie('refresh-token', refreshToken, { // Куки refresh
      httpOnly: true,
      maxAge: 7 * 24 * 60 * 60 * 1000,
      secure: this.configService.get<string>('COOKIE_SECURE') === "true",
      sameSite: this.configService.getOrThrow<string>('COOKIE_SAME_SITE') as "lax" | "strict",
    });

    return res.json({ accessToken, refreshToken }); // Ответ
  }
}
```

**Разбор:**
- `argon2.hash/verify`: Хэширование. Docs: [Argon2](https://github.com/ranisalt/node-argon2).
- `jwtService.sign`: Подпись. Docs: [JWT](https://docs.nestjs.com/security/authentication#jwt-token).
- `res.cookie`: Куки. Docs: [Express Res](https://expressjs.com/en/api.html#res.cookie).

## Глава 5: Common

### Шаг 5.1: roles.decorator.ts
Код:
```ts
import { SetMetadata } from '@nestjs/common'; // Метаданные

export const Roles = (...roles: string[]) => SetMetadata('roles', roles); // Декоратор ролей
```

**Разбор:**
- `SetMetadata`: Кастом декоратор. Docs: [Custom Decorators](https://docs.nestjs.com/custom-decorators).

### Шаг 5.2: roles.guard.ts
Код:
```ts
import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common'; // Guard интерфейсы
import { Observable } from 'rxjs'; // Observable
import { Reflector } from '@nestjs/core'; // Reflector

@Injectable() // Инжектируемый
export class RolesGuard implements CanActivate { // Guard ролей
  constructor(private reflector: Reflector) {} // Инъекция
  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> { // Метод
    const requiredRoles = this.reflector.getAllAndOverride<string[]>('roles', [ // Получение метаданных
      context.getHandler(),
      context.getClass(),
    ]);

    if(!requiredRoles) return true; // Если нет ролей

    const { user } = context.switchToHttp().getRequest(); // User из req
    return requiredRoles.some((role) => user?.role === role); // Проверка
  }
}
```

**Разбор:**
- `implements CanActivate`: Guard. Docs: [Guards](https://docs.nestjs.com/guards).

### Шаг 5.3: slugify.ts
Код:
```ts
import slugify from 'slugify'; // Slugify

export function slug(text: string): string { // Функция слага
  return slugify(text, { // Опции
    lower: true, // Нижний регистр
    strict: true, // Строгий
    locale: 'ru', // Русский
    trim: true, // Обрезка
  });
}
```

**Разбор:**
- Опции для русского. Docs: [Slugify Options](https://github.com/simov/slugify#options).

### Шаг 5.4: upload.utils.ts
Код:
```ts
import { diskStorage } from 'multer'; // Storage
import { extname } from 'path'; // Extname
import { v4 as uuid } from 'uuid'; // UUID

export const uploadOptions = { // Опции Multer
  storage: diskStorage({ // Дисковое хранение
    destination: './uploads', // Директория
    filename: (_, file, cb) => { // Имя файла
      const uniqueName = `${uuid()}${extname(file.originalname)}`; // UUID + ext
      cb(null, uniqueName); // Callback
    },
  }),
  fileFilter: (_, file, cb) => { // Фильтр
    const allowedTypes = /jpg|jpeg|png|gif/; // Типы
    const isValid = allowedTypes.test(file.mimetype); // Проверка
    if (isValid) cb(null, true);
    else cb(new Error('Разрешены только изображения'), false);
  },
  limits: { fileSize: 5 * 1024 * 1024 }, // Лимит 5MB
};
```

**Разбор:**
- `diskStorage`: Хранение. Docs: [Multer Storage](https://github.com/expressjs/multer#diskstorage).

## Глава 6: Categories Модуль

Генерируйте аналогично.

### Шаг 6.1: create-category.dto.ts
Код:
```ts
import { IsString } from 'class-validator'; // Валидатор

export class CreateCategoryDto { // DTO создания
  @IsString({ message: "Название категории должно быть строкой" }) // Строка
  name: string;
}
```

update-category.dto.ts:
```ts
import { IsOptional, IsString } from 'class-validator'; // Валидаторы

export class UpdateCategoryDto { // DTO обновления
  @IsOptional() // Опционально
  @IsString() // Строка
  name?: string;
}
```

### Шаг 6.2: categories.controller.ts
Код (исправил Put на Delete для delete):
```ts
import { Body, Controller, Get, Param, Post, Put, Delete, UseGuards } from '@nestjs/common'; // Декораторы
import { CategoriesService } from './categories.service'; // Сервис
import { AuthGuard } from '@nestjs/passport'; // Guard
import { RolesGuard } from '../common/guards/roles/roles.guard'; // Roles guard
import { Roles } from '../common/decorators/roles.decorator'; // Roles
import { CreateCategoryDto } from './dto/create-category.dto'; // DTO
import { UpdateCategoryDto } from './dto/update-category.dto'; // DTO

@Controller('categories') // /categories
export class CategoriesController {
  constructor(private readonly categoriesService: CategoriesService) {} // Инъекция

  @Get() // GET /
  async getCategories() { // Все категории
    return this.categoriesService.findAll();
  }

  @Get(':id') // GET /:id
  async getCategoryById(@Param('id') id: string) { // По ID
    return this.categoriesService.findOne(parseInt(id)); // Parse to number
  }

  @Post() // POST /
  @Roles('admin') // Только admin
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Guards
  async createCategory(@Body() dto: CreateCategoryDto) { // Создание
    return this.categoriesService.create(dto);
  }

  @Put(':id') // PUT /:id
  @Roles('admin')
  @UseGuards(AuthGuard('jwt'), RolesGuard)
  async updateCategory(@Param('id') id: string, @Body() dto: UpdateCategoryDto) { // Обновление
   return this.categoriesService.update(parseInt(id), dto);
  }

  @Delete(':id') // DELETE /:id (исправлено с Put)
  @Roles('admin')
  @UseGuards(AuthGuard('jwt'), RolesGuard)
  async deleteCategory(@Param('id') id: string) { // Удаление
    return this.categoriesService.delete(parseInt(id));
  }
}
```

**Разбор:**
- `@Delete`: Для удаления. Docs: [Controllers](https://docs.nestjs.com/controllers#request-mapping).

### Шаг 6.3: categories.module.ts
Код:
```ts
import { Module } from '@nestjs/common'; // Модуль
import { CategoriesService } from './categories.service'; // Сервис
import { CategoriesController } from './categories.controller'; // Контроллер
import { PrismaModule } from '../prisma/prisma.module'; // Prisma

@Module({
  imports: [PrismaModule], // Импорт
  controllers: [CategoriesController], // Контроллер
  providers: [CategoriesService], // Сервис
})
export class CategoriesModule {}
```

### Шаг 6.4: categories.service.ts
Код:
```ts
import { Injectable, NotFoundException } from '@nestjs/common'; // Injectable, исключение
import { PrismaService } from '../prisma/prisma.service'; // Prisma
import { CreateCategoryDto } from './dto/create-category.dto'; // DTO
import { UpdateCategoryDto } from './dto/update-category.dto'; // DTO

@Injectable() // Инжектируемый
export class CategoriesService {
  constructor(private readonly prismaService: PrismaService) {} // Инъекция

  async findAll() { // Все
    return this.prismaService.category.findMany({ include: { // С include
        posts: {
          include: {
            user: {
              select: { id: true, name: true, email: true } // Select полей
            },
          },
        },
      },
    });
  }

  async findOne(id: number) { // Один
    const category = await this.prismaService.category.findUnique({
      where: { id: id },
      include: { // Include
        posts: {
          include: {
            user: {
              select: { id: true, name: true, email: true },
            },
          },
        },
      },
    });
    if (!category) throw new NotFoundException('Категория не найдена'); // Ошибка
    return category;
  }

  async create(dto: CreateCategoryDto) { // Создание
    return this.prismaService.category.create({
      data: {
        name: dto.name, // Данные
      },
    });
  }

  async update(id: number, dto: UpdateCategoryDto) { // Обновление
    return this.prismaService.category.update({
      where: { id: id },
      data: {
        name: dto.name,
      },
    });
  }

  async delete(id: number) { // Удаление
    return this.prismaService.category.delete({ where: { id: id } });
  }
}
```

**Разбор:**
- Prisma методы. Docs: [CRUD](https://www.prisma.io/docs/concepts/components/prisma-client/crud).

## Глава 7: Posts Модуль

### Шаг 7.1: create-post.dto.ts
Код:
```ts
import { IsNumber, IsString } from 'class-validator'; // Валидаторы
import { Transform } from 'class-transformer'; // Трансформер

export class CreatePostDto { // DTO
  @IsString() // Строка
  title: string;

  @IsString() // Строка
  content: string;

  @Transform(({ value }) => parseInt(value, 10)) // Трансформ в number
  @IsNumber() // Число
  categoryId: number;
}
```

**Разбор:**
- `@Transform`: Из string в number. Docs: [Class-Transformer](https://github.com/typestack/class-transformer#basic-usage).

update-post.dto.ts:
```ts
import { IsNumber, IsOptional, IsString } from 'class-validator'; // Валидаторы
import { Transform } from 'class-transformer'; // Трансформер

export class UpdatePostDto { // DTO
  @IsString()
  @IsOptional() // Опционально
  title?: string;

  @IsString()
  @IsOptional()
  slug?: string;

  @IsString()
  @IsOptional()
  content?: string;

  @IsString()
  @IsOptional()
  imagePath?: string;

  @Transform(({ value }) => parseInt(value, 10))
  @IsNumber()
  @IsOptional()
  categoryId?: number;
}
```

### Шаг 7.2: posts.controller.ts
Код:
```ts
import {
  Body,
  Controller, Delete,
  Get,
  Param,
  Post,
  Put,
  Query,
  UploadedFile,
  UseGuards,
  UseInterceptors,
} from '@nestjs/common'; // Декораторы
import { FileInterceptor } from '@nestjs/platform-express'; // Интерсептор файла
import { PostsService } from './posts.service'; // Сервис
import { CreatePostDto } from './dto/create-post.dto'; // DTO
import { uploadOptions } from '../common/utils/upload.utils'; // Опции
import { AuthGuard } from '@nestjs/passport'; // Guard
import { RolesGuard } from '../common/guards/roles/roles.guard'; // Roles
import { Roles } from '../common/decorators/roles.decorator'; // Roles
import { UpdatePostDto } from './dto/update-post.dto'; // DTO

@Controller('posts') // /posts
export class PostsController {
  constructor(private readonly postsService: PostsService) {} // Инъекция

  @Get() // GET /
  async findAll(@Query('category') category?: string) { // Все, с query
    return this.postsService.findAll(category);
  }

  @Get(':slug') // GET /:slug
  async findOne(@Param('slug') slug: string) { // По слаг
    return this.postsService.findOne(slug);
  }

  @Post() // POST /
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Guards
  @Roles('admin') // Admin
  @UseInterceptors(FileInterceptor('image', uploadOptions)) // Интерсептор файла
  async create(@Body() dto: CreatePostDto, @UploadedFile() file: Express.Multer.File) { // Создание с файлом
    return this.postsService.create(dto, file);
  }

  @Put(':id') // PUT /:id
  @UseGuards(AuthGuard('jwt'), RolesGuard)
  @Roles('admin')
  @UseInterceptors(FileInterceptor('image', uploadOptions))
  async update(@Param('id') id: string, @Body() dto: UpdatePostDto, @UploadedFile() file?: Express.Multer.File) { // Обновление
    return this.postsService.update(parseInt(id), dto, file);
  }

  @Delete(':id') // DELETE /:id
  @UseGuards(AuthGuard('jwt'), RolesGuard)
  @Roles('admin')
  async delete(@Param('id') id: string) { // Удаление
    return this.postsService.delete(parseInt(id));
  }
}
```

**Разбор:**
- `@UseInterceptors(FileInterceptor)`: Для файлов. Docs: [File Upload](https://docs.nestjs.com/techniques/file-upload).

### Шаг 7.3: posts.module.ts
Код:
```ts
import { Module } from '@nestjs/common';
import { PostsService } from './posts.service';
import { PostsController } from './posts.controller';
import { PrismaModule } from '../prisma/prisma.module';

@Module({
  imports: [PrismaModule],
  controllers: [PostsController],
  providers: [PostsService],
})
export class PostsModule {}
```

### Шаг 7.4: posts.service.ts
Код:
```ts
import { Injectable, NotFoundException } from '@nestjs/common'; // Injectable
import { PrismaService } from '../prisma/prisma.service'; // Prisma
import { CreatePostDto } from './dto/create-post.dto'; // DTO
import { slug as gSlug } from '../common/utils/slugify'; // Slug
import { UpdatePostDto } from './dto/update-post.dto'; // DTO

@Injectable()
export class PostsService {
  constructor(private readonly prismaService: PrismaService) {}

  async findAll(category?: string) { // Все
    const where = category ? { category: { name: category } } : {}; // Where фильтр

    return this.prismaService.post.findMany({
      where,
      include: {
        category: true, // Include
        user: {
          select: {
            name: true,
            email: true,
          },
        },
      },
    });
  }

  async findOne(slug: string) { // Один
    const post = await this.prismaService.post.findUnique({
      where: { slug: slug },
      include: {
        category: true,
        user: {
          select: {
            name: true,
            email: true,
          },
        },
      },
    });
    if (!post) throw new NotFoundException('Запись не найдена');
    return post;
  }

  async create(dto: CreatePostDto, file?: Express.Multer.File) { // Создание
    return this.prismaService.post.create({
      data: {
        title: dto.title,
        slug: gSlug(dto.title), // Слаг
        content: dto.content,
        categoryId: Number(dto.categoryId),
        imagePath: file ? `/uploads/${file.filename}` : null, // Путь если файл
      },
    });
  }

  async update(id: number, dto: UpdatePostDto, file?: Express.Multer.File) { // Обновление
    const post = await this.prismaService.post.findUnique({ where: { id: id } });
    if (!post) throw new NotFoundException('Запись не найдена');
    return this.prismaService.post.update({
      where: { id: id },
      data: {
        title: dto.title,
        slug: dto.title ? gSlug(dto.title) : post.slug, // Условный слаг
        content: dto.content,
        categoryId: Number(dto.categoryId),
        imagePath: file ? `/uploads/${file.filename}` : post.imagePath, // Условный путь
      },
    });
  }

  async delete(id: number) { // Удаление
    const post = await this.prismaService.post.findUnique({ where: { id: id } });
    if (!post) throw new NotFoundException('Запись не найдена');

    return this.prismaService.post.delete({ where: { id: id } });
  }
}
```

## Глава 8: Users Модуль

### Шаг 8.1: create.dto.ts
Код:
```ts
import { IsEmail, IsString, MinLength } from 'class-validator';

export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @IsString()
  @MinLength(6, { message: "Минимальная длина пароля 6 символов" })
  password: string;
}
```

update.dto.ts:
```ts
import { IsEmail, IsOptional, IsString, MinLength } from 'class-validator';

export class UpdateUserDto {
  @IsString()
  @IsOptional()
  name?: string;

  @IsEmail()
  @IsOptional()
  email?: string;

  @IsString()
  @IsOptional()
  @MinLength(6, { message: "Минимальная длина пароля 6 символов" })
  password?: string;

  @IsOptional()
  @IsString()
  role?: string;
}
```

### Шаг 8.2: users.controller.ts
Код:
```ts
import { Controller, Get, UseGuards, Param, Post, Body, Put, Delete } from '@nestjs/common'; // Декораторы
import { UsersService } from './users.service'; // Сервис
import { AuthGuard } from '@nestjs/passport'; // Guard
import { RolesGuard } from '../common/guards/roles/roles.guard'; // Roles
import { Roles } from '../common/decorators/roles.decorator'; // Roles
import { CreateUserDto } from './dto/create.dto'; // DTO
import { UpdateUserDto } from './dto/update.dto'; // DTO

@Controller('users') // /users
export class UsersController {
  constructor(private readonly usersService: UsersService) {}

  @Get() // GET /
  @UseGuards(AuthGuard('jwt'), RolesGuard)
  @Roles('admin')
  async findAll() { // Все
    return this.usersService.findAll();
  }

  @Get(':id') // GET /:id
  @UseGuards(AuthGuard('jwt'), RolesGuard)
  @Roles('admin')
  async findOne(@Param('id') id: string) { // Один
    return this.usersService.findOne(parseInt(id));
  }

  @Post() // POST /
  @UseGuards(AuthGuard('jwt'), RolesGuard)
  @Roles('admin')
  async create(@Body() dto: CreateUserDto) { // Создание
    return this.usersService.create(dto);
  }

  @Put(':id') // PUT /:id
  @UseGuards(AuthGuard('jwt'), RolesGuard)
  @Roles('admin')
  async update(@Param('id') id: string, @Body() dto: UpdateUserDto) { // Обновление
    return this.usersService.update(parseInt(id), dto);
  }

  @Delete(':id') // DELETE /:id
  @UseGuards(AuthGuard('jwt'), RolesGuard)
  @Roles('admin')
  async delete(@Param('id') id: string) { // Удаление
    return this.usersService.delete(parseInt(id));
  }
}
```

### Шаг 8.3: users.module.ts
Код:
```ts
import { Module } from '@nestjs/common';
import { UsersService } from './users.service';
import { UsersController } from './users.controller';
import { PrismaModule } from '../prisma/prisma.module';

@Module({
  imports: [PrismaModule],
  controllers: [UsersController],
  providers: [UsersService],
})
export class UsersModule {}
```

### Шаг 8.4: users.service.ts
Код:
```ts
import { Injectable, NotFoundException, UnauthorizedException } from '@nestjs/common'; // Исключения
import { PrismaService } from '../prisma/prisma.service'; // Prisma
import { CreateUserDto } from './dto/create.dto'; // DTO
import * as argon2 from 'argon2'; // Argon2
import { UpdateUserDto } from './dto/update.dto'; // DTO

@Injectable()
export class UsersService {
  constructor(private readonly prismaService: PrismaService) {}

  async findAll() { // Все
    return this.prismaService.user.findMany();
  }

  async findOne(id: number) { // Один
    return this.prismaService.user.findUnique({ where: { id: id } });
  }

  async create(dto: CreateUserDto) { // Создание
    const existsUser = await this.prismaService.user.findUnique({ where: { email: dto.email } });
    if (existsUser) throw new UnauthorizedException("Пользователь с таким email уже существует!");

    return this.prismaService.user.create({
      data: {
        name: dto.name,
        email: dto.email,
        password: await argon2.hash(dto.password), // Хэш
      },
    });
  }

  async update(id: number, dto: UpdateUserDto) { // Обновление
    const existsUser = await this.prismaService.user.findUnique({ where: { id: id } });
    if (!existsUser) throw new NotFoundException("Пользователь не найден");

    return this.prismaService.user.update({
      where: { id: id },
      data: {
        name: dto.name,
        email: dto.email,
        password: dto.password ? await argon2.hash(dto.password) : existsUser.password, // Условный хэш
        role: dto.role,
      },
    });
  }

  async delete(id: number) { // Удаление
    const existsUser = await this.prismaService.user.findUnique({ where: { id: id } });
    if (!existsUser) throw new NotFoundException("Пользователь не найден");

    return this.prismaService.user.delete({ where: { id: id } });
  }
}
```

## Глава 9: App

### Шаг 9.1: app.controller.ts
Код:
```ts
import { Controller, Get } from '@nestjs/common'; // Декораторы
import { AppService } from './app.service'; // Сервис

@Controller() // Базовый
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get() // GET /
  getHello(): string { // Hello
    return this.appService.getHello();
  }
}
```

app.service.ts:
```ts
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Hello World!';
  }
}
```

### Шаг 9.2: app.module.ts
Код:
```ts
import { Module } from '@nestjs/common'; // Модуль
import { AppController } from './app.controller'; // Контроллер
import { AppService } from './app.service'; // Сервис
import { ConfigModule } from '@nestjs/config'; // Конфиг
import { ServeStaticModule } from '@nestjs/serve-static'; // Static
import { join } from 'path'; // Path
import { PrismaModule } from './prisma/prisma.module'; // Prisma
import { AuthModule } from './auth/auth.module'; // Auth
import { PostsModule } from './posts/posts.module'; // Posts
import { UsersModule } from './users/users.module'; // Users
import { CategoriesModule } from './categories/categories.module'; // Categories

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }), // Глобальный конфиг
    ServeStaticModule.forRoot({ // Static uploads
      rootPath: join(__dirname, '..', 'uploads'), // Путь
      serveRoot: '/uploads', // Роут
    }),
    PrismaModule,
    AuthModule,
    PostsModule,
    UsersModule,
    CategoriesModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
```

**Разбор:**
- `ServeStaticModule.forRoot`: Static. Docs: [Serve Static](https://docs.nestjs.com/techniques/serve-static).

## Глава 10: main.ts
Код:
```ts
import { NestFactory } from '@nestjs/core'; // Фабрика
import { AppModule } from './app.module'; // Модуль
import { ConfigService } from '@nestjs/config'; // Конфиг
import * as cookieParser from 'cookie-parser'; // Parser
import { ValidationPipe } from '@nestjs/common'; // Pipe

async function bootstrap() { // Bootstrap
  const app = await NestFactory.create(AppModule); // Создание app
  const configService = app.get(ConfigService); // Конфиг
  app.enableCors({ // CORS
    credentials: true,
    origin: configService.getOrThrow<string>('CLIENT_URL'),
  });
  app.useGlobalPipes(new ValidationPipe({ // Глобальный pipe
    transform: true, // Трансформ
    whitelist: true, // Whitelist
  }));
  app.setGlobalPrefix('api'); // Префикс /api
  app.use(cookieParser()); // Middleware куки
  await app.listen(configService.getOrThrow<number>('PORT')); // Запуск
}
bootstrap(); // Вызов
```

**Разбор:**
- `NestFactory.create`: App. Docs: [Bootstrap](https://docs.nestjs.com/first-steps#platform).
- `enableCors`: CORS. Docs: [CORS](https://docs.nestjs.com/security/cors).
- `useGlobalPipes`: Pipes. Docs: [Pipes](https://docs.nestjs.com/pipes).

## Заключение
Теперь проект с комментариями готов. Запустите `npm run start:dev`. Если нужно доработки, дайте знать! 

— Автор.