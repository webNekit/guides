// Краткое описание функционала сайта

// Новостной портал с базовым функционалом: неавторизованные пользователи могут просматривать список статей, фильтровать по категориям и читать статьи. Авторизованные пользователи могут добавлять/удалять статьи в избранное. Администраторы имеют полный CRUD над пользователями, постами и категориями. Вся аутентификация (регистрация, логин, refresh, logout) реализована в модуле auth.

// Это полный backend проект на NestJS без заглушек. Все модули, файлы и логика реализованы полностью. Комментарии на русском языке с объяснениями и ссылками на документацию. Структура соответствует рекомендациям: тонкие контроллеры, логика в сервисах, Prisma не глобальный, JWT с refresh, RBAC, централизованные uploads, cookie handling в сервисе. Добавлена раздача статических файлов через ServeStaticModule. Для избранного использована many-to-many связь в schema, эндпоинты в PostsController (для авторизованных), userId извлекается из req.user в контроллере и передаётся в сервис. Фильтр по категориям в findAll постов через @Query.

// Установите зависимости, примените миграции (npx prisma migrate dev), запустите npm run start:dev. Тестируйте в Insomnia.

#### README.md содержание
```
// Это README файл для backend проекта.
// Содержит инструкции по установке и использованию.
// https://docs.nestjs.com/ для документации NestJS.

# Новостной портал Backend

## Требования
- Node.js >= 18 // Минимальная версия Node для совместимости с NestJS и зависимостями. https://nodejs.org/en/docs
- npm или yarn // Менеджеры пакетов для установки зависимостей. https://docs.npmjs.com/

## Установка
1. Склонируйте репозиторий или создайте новую директорию: `mkdir news-portal-backend && cd news-portal-backend` // Создаёт директорию проекта. https://git-scm.com/docs/git-clone
2. Инициализируйте проект NestJS: `npx @nestjs/cli new . --skip-git --skip-install` // Создаёт базовую структуру NestJS. https://docs.nestjs.com/cli/overview
3. Установите зависимости: `npm install @nestjs/common @nestjs/core @nestjs/jwt @nestjs/passport @nestjs/config cookie-parser class-validator class-transformer multer argon2 prisma @prisma/client passport-jwt @nestjs/serve-static` // Устанавливает пакеты для аутентификации, валидации, загрузки файлов и т.д. https://www.npmjs.com/
4. Создайте `.env` из `.env.example` и заполните значения. // Конфигурация переменных окружения. https://docs.nestjs.com/techniques/configuration
5. Настройте Prisma: `npx prisma generate` и `npx prisma migrate dev --name init` // Генерирует клиент и применяет миграции. https://www.prisma.io/docs/reference/api-reference/command-reference
6. Запустите приложение: `npm run start:dev` // Запускает сервер в режиме разработки. https://docs.nestjs.com/cli/scripts

## Миграции Prisma
- Генерация клиента: `npx prisma generate` // Создаёт Prisma клиент. https://www.prisma.io/docs/reference/api-reference/prisma-client-reference
- Миграция: `npx prisma migrate dev --name [migration-name]` // Создаёт и применяет миграции базы данных. https://www.prisma.io/docs/concepts/components/prisma-migrate
- Сидинг (опционально): Реализуйте в prisma/seed.ts и выполните `npx prisma db seed` // Заполняет базу начальными данными. https://www.prisma.io/docs/guides/other/seed-database

## Тестирование
Используйте Insomnia или Postman для тестирования эндпоинтов. Базовый URL: http://localhost:9000/api // Базовый путь API, установленный в main.ts.
- Аутентификация: POST /api/auth/register, /api/auth/login и т.д. // Эндпоинты аутентификации.
- Защищённые маршруты требуют заголовок Authorization: Bearer <accessToken> // JWT токен для авторизации. https://docs.nestjs.com/security/authentication
- Посты: GET /api/posts (список, ?category= для фильтра), GET /api/posts/:slug (чтение), POST/PUT/DELETE для админа.
- Избранное: POST /api/posts/:id/favorite (добавить), DELETE /api/posts/:id/favorite (удалить) для авторизованных.
- Пользователи/Категории: CRUD для админа.

## Примечания
- Загрузка файлов локальная (диск). Для облака (например, S3) расширьте src/common/uploads.ts — см. комментарии. // Локальное хранилище для простоты. https://docs.nestjs.com/techniques/file-upload
- Refresh токен хранится в виде хеша в базе для безопасности: https://auth0.com/blog/refresh-token-rotation-and-reuse-detection-in-node-js/ // Повышает защиту от кражи токенов.
- Куки: HttpOnly для безопасности — https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#security // Защищает от XSS атак.
- Изображения в постах: Хранятся как путь в базе после загрузки, обслуживаются через /uploads. // Добавлено по запросу.
- Статические файлы: Обслуживаются через ServeStaticModule в app.module.ts. // Для доступа к uploads. https://docs.nestjs.com/recipes/serve-static
```

#### .env.example (и .env)
```
// Файл пример переменных окружения.
// Скопируйте в .env и заполните значения.
// https://docs.nestjs.com/techniques/configuration для использования ConfigModule.

PORT=9000 // Порт сервера. По умолчанию 9000, если не указано.
CLIENT_URL=http://localhost:3000 // URL фронтенда для CORS. https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
DATABASE_URL="file:./dev.db" // URL базы данных для Prisma (SQLite). https://www.prisma.io/docs/reference/database-reference/connection-urls
UPLOAD_DIR=uploads // Директория для загруженных файлов. Путь для локального хранения.

#JWT 
JWT_ACCESS_SECRET="access-secret-key" // Секрет для подписи access JWT. Храните в тайне. https://github.com/auth0/node-jsonwebtoken
JWT_REFRESH_SECRET="refresh-secret-key" // Секрет для подписи refresh JWT.
JWT_ACCESS_EXPIRES_IN="15m" // Время жизни access токена.
JWT_REFRESH_EXPIRES_IN="7d" // Время жизни refresh токена.
```

#### prisma/schema.prisma
```
// Файл схемы Prisma, определяющий модели базы данных.
// Использует SQLite по умолчанию.
// https://www.prisma.io/docs/concepts/components/prisma-schema для синтаксиса схемы.

datasource db { // Определяет источник данных.
  provider = "sqlite" // Провайдер SQLite для локальной разработки. Прост в настройке, не требует сервера.
  url      = env("DATABASE_URL") // URL из .env. Позволяет легко переключиться на другие базы, например PostgreSQL.
}

generator client { // Генерирует Prisma Client.
  provider = "prisma-client-js" // Генератор для JavaScript.
}

model User { // Модель пользователя для аутентификации.
  id           Int      @id @default(autoincrement()) // Автоинкрементный ID.
  email        String   @unique // Уникальный email для входа.
  password     String // Хешированный пароль.
  role         String   @default("user") // Роль для RBAC, по умолчанию 'user'.
  refreshToken String? // Хеш refresh токена для безопасности.
  createdAt    DateTime @default(now()) // Временная метка создания.
  favorites    Post[]   @relation("favorites") // Связь многие-ко-многим для избранного. Добавлено для функционала избранного.
  posts        Post[]   // Связь с постами автора.
}

model Category { // Модель категории для постов.
  id    Int    @id @default(autoincrement()) // Автоинкрементный ID.
  name  String @unique // Уникальное имя категории.
  posts Post[] // Связь с постами.
}

model Post { // Модель поста для статей.
  id          Int      @id @default(autoincrement()) // Автоинкрементный ID.
  title       String // Заголовок поста.
  slug        String   @unique // Уникальный slug для URL.
  content     String // Содержимое поста.
  imagePath   String? // Путь к загруженному изображению (добавлено по запросу). Необязательное поле.
  authorId    Int // Внешний ключ автора.
  author      User     @relation(fields: [authorId], references: [id]) // Связь с пользователем.
  categoryId  Int // Внешний ключ категории.
  category    Category @relation(fields: [categoryId], references: [id]) // Связь с категорией.
  createdAt   DateTime @default(now()) // Временная метка создания.
  updatedAt   DateTime @updatedAt // Временная метка обновления.
  favoritedBy User[]   @relation("favorites") // Связь многие-ко-многим для избранного. Добавлено для функционала.
}
```

#### src/main.ts
```typescript
// Точка входа приложения.
// Запускает приложение NestJS с конфигурацией.
// https://docs.nestjs.com/ для основ NestJS.

import { NestFactory } from '@nestjs/core'; // Импортирует фабрику для создания приложения. https://docs.nestjs.com/fundamentals/custom-providers
import { AppModule } from './app.module'; // Импортирует корневой модуль.
import { ConfigService } from '@nestjs/config'; // Сервис для доступа к переменным окружения. https://docs.nestjs.com/techniques/configuration
import { ValidationPipe } from '@nestjs/common'; // Пайп для валидации DTO. https://docs.nestjs.com/pipes#built-in-pipes
import * as cookieParser from 'cookie-parser'; // Middleware для парсинга кук. https://www.npmjs.com/package/cookie-parser

async function bootstrap() { // Асинхронная функция для запуска приложения.
  const app = await NestFactory.create(AppModule); // Создаёт приложение Nest из корневого модуля.
  const configService = app.get(ConfigService); // Получает экземпляр ConfigService.
  app.useGlobalPipes(new ValidationPipe({ whitelist: true })); // Устанавливает глобальный пайп валидации для удаления неизвестных свойств. https://docs.nestjs.com/pipes#global-scoped-pipes
  app.enableCors({ // Включает CORS с настройками.
    origin: configService.get<string>('CLIENT_URL'), // Разрешает запросы с URL фронтенда.
    credentials: true, // Разрешает куки в CORS запросах. https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS
  });
  app.setGlobalPrefix('api'); // Устанавливает глобальный префикс для всех маршрутов. https://docs.nestjs.com/controllers#routing
  app.use(cookieParser()); // Использует cookie-parser глобально. https://www.npmjs.com/package/cookie-parser
  await app.listen(configService.get<number>('PORT') || 9000); // Запускает сервер на порту из env или 9000.
}
bootstrap(); // Вызывает функцию запуска.
```

#### src/app.module.ts
```typescript
// Корневой модуль приложения.
// Импортирует другие модули, включая ServeStaticModule для статических файлов.
// https://docs.nestjs.com/modules для системы модулей.

import { Module } from '@nestjs/common'; // Декоратор модуля. https://docs.nestjs.com/modules
import { ConfigModule } from '@nestjs/config'; // Модуль для конфигурации. https://docs.nestjs.com/techniques/configuration
import { PrismaModule } from './prisma/prisma.module'; // Пользовательский модуль Prisma.
import { AuthModule } from './auth/auth.module'; // Модуль аутентификации.
import { UsersModule } from './users/users.module'; // Модуль пользователей.
import { PostsModule } from './posts/posts.module'; // Модуль постов.
import { CategoriesModule } from './categories/categories.module'; // Модуль категорий.
import { ServeStaticModule } from '@nestjs/serve-static'; // Модуль для раздачи статических файлов. https://docs.nestjs.com/recipes/serve-static
import { join } from 'path'; // Функция для объединения путей. https://nodejs.org/api/path.html#pathjoinpaths

@Module({ // Декоратор модуля.
  imports: [ // Массив импортируемых модулей.
    ConfigModule.forRoot({ isGlobal: true }), // Делает ConfigModule глобальным. https://docs.nestjs.com/techniques/configuration#global-module
    PrismaModule, // Импортирует Prisma для доступа к базе.
    AuthModule, // Импортирует Auth для аутентификации.
    UsersModule, // Импортирует Users для CRUD пользователей.
    PostsModule, // Импортирует Posts для CRUD постов.
    CategoriesModule, // Импортирует Categories для CRUD категорий.
    ServeStaticModule.forRoot({ // Настраивает раздачу статических файлов из uploads. https://docs.nestjs.com/recipes/serve-static
      rootPath: join(__dirname, '..', 'uploads'), // Путь к директории uploads.
      serveRoot: '/uploads', // Префикс URL для доступа (например, /uploads/image.jpg).
    }),
  ],
})
export class AppModule {} // Экспортирует класс корневого модуля.
```

#### src/prisma/prisma.module.ts
```typescript
// Модуль предоставляет PrismaService.
// Не глобальный, как указано в требованиях.
// https://www.prisma.io/docs/concepts/components/prisma-client для PrismaClient.

import { Module } from '@nestjs/common'; // Декоратор модуля.
import { PrismaService } from './prisma.service'; // Импортирует сервис.

@Module({ // Декоратор модуля.
  providers: [PrismaService], // Предоставляет PrismaService.
  exports: [PrismaService], // Экспортирует для других модулей. https://docs.nestjs.com/modules#shared-modules
})
export class PrismaModule {} // Экспортирует класс модуля.
```

#### src/prisma/prisma.service.ts
```typescript
// Сервис управляет жизненным циклом PrismaClient.
// Расширяет PrismaClient для управления подключением.
// https://www.prisma.io/docs/concepts/components/prisma-client

import { Injectable, OnModuleDestroy, OnModuleInit } from '@nestjs/common'; // Декораторы для хуков жизненного цикла. https://docs.nestjs.com/fundamentals/lifecycle-events
import { PrismaClient } from '@prisma/client'; // Импортирует PrismaClient.

/**
 * PrismaService управляет подключением PrismaClient.
 * Расширяет PrismaClient и обрабатывает хуки жизненного цикла.
 * @see https://www.prisma.io/docs/concepts/components/prisma-client
 */
@Injectable() // Декоратор для инъекции.
export class PrismaService extends PrismaClient implements OnModuleInit, OnModuleDestroy { // Расширяет и реализует интерфейсы.
  async onModuleInit() { // Хук инициализации модуля.
    await this.$connect(); // Подключается к базе данных. https://www.prisma.io/docs/reference/api-reference/prisma-client-reference#connect
  }

  async onModuleDestroy() { // Хук уничтожения модуля.
    await this.$disconnect(); // Отключается от базы данных. https://www.prisma.io/docs/reference/api-reference/prisma-client-reference#disconnect
  }
}
```

#### src/auth/auth.module.ts
```typescript
// Модуль аутентификации.
// Регистрирует JWT асинхронно.
// https://docs.nestjs.com/security/authentication

import { Module } from '@nestjs/common'; // Декоратор модуля.
import { AuthController } from './auth.controller'; // Импортирует контроллер.
import { AuthService } from './auth.service'; // Импортирует сервис.
import { PrismaModule } from '../prisma/prisma.module'; // Импортирует Prisma.
import { JwtModule } from '@nestjs/jwt'; // Модуль JWT. https://docs.nestjs.com/security/authentication#jwt-module
import { ConfigModule, ConfigService } from '@nestjs/config'; // Конфигурация для асинхронной регистрации.
import { PassportModule } from '@nestjs/passport'; // Passport для стратегий. https://docs.nestjs.com/security/authentication#passport
import { JwtStrategy } from './strategies/jwt.strategy'; // Стратегия access.
import { JwtRefreshStrategy } from './strategies/jwt-refresh.strategy'; // Стратегия refresh.

@Module({ // Декоратор модуля.
  imports: [ // Импортируемые модули.
    PrismaModule, // Для доступа к базе.
    PassportModule, // Для стратегий passport.
    JwtModule.registerAsync({ // Асинхронная регистрация JWT. https://docs.nestjs.com/security/authentication#async-options
      imports: [ConfigModule], // Импортирует ConfigModule.
      useFactory: async (configService: ConfigService) => ({ // Функция-фабрика.
        secret: configService.get('JWT_ACCESS_SECRET'), // Секрет из env.
        signOptions: { expiresIn: configService.get('JWT_ACCESS_EXPIRES_IN') }, // Время жизни из env.
      }),
      inject: [ConfigService], // Инъекция ConfigService.
    }),
  ],
  controllers: [AuthController], // Регистрирует контроллер.
  providers: [AuthService, JwtStrategy, JwtRefreshStrategy], // Провайдеры.
  exports: [AuthService], // Экспортирует сервис.
})
export class AuthModule {} // Экспортирует модуль.
```

#### src/auth/auth.controller.ts
```typescript
// Контроллер аутентификации для маршрутов.
// Тонкий слой: делегирует сервису.
// https://docs.nestjs.com/controllers

import { Body, Controller, Post, Req, Res, UseGuards } from '@nestjs/common'; // Декораторы и типы. https://docs.nestjs.com/controllers#request-object
import { AuthService } from './auth.service'; // Импортирует сервис.
import { RegisterDto } from './dto/register.dto'; // Импортирует DTO.
import { LoginDto } from './dto/login.dto'; // Импортирует DTO.
import { Request, Response } from 'express'; // Типы Request и Response из Express. https://expressjs.com/en/api.html#req, https://expressjs.com/en/api.html#res
import { AuthGuard } from '@nestjs/passport'; // Гарда для passport. https://docs.nestjs.com/guards

/**
 * AuthController обрабатывает маршруты аутентификации.
 * Вся логика делегируется AuthService.
 * @see https://docs.nestjs.com/controllers
 */
@Controller('auth') // Декоратор контроллера с путём.
export class AuthController { // Класс контроллера.
  constructor(private readonly authService: AuthService) {} // Инъекция сервиса.

  @Post('register') // POST маршрут для регистрации.
  async register(@Body() dto: RegisterDto, @Res() res: Response) { // Обрабатывает тело и ответ.
    return this.authService.register(dto, res); // Делегирует сервису.
  }

  @Post('login') // POST маршрут для входа.
  async login(@Body() dto: LoginDto, @Res() res: Response) { // Обрабатывает тело и ответ.
    return this.authService.login(dto, res); // Делегирует сервису.
  }

  @Post('refresh') // POST маршрут для обновления токена.
  @UseGuards(AuthGuard('jwt-refresh')) // Использует гарду refresh для валидации.
  async refresh(@Req() req: Request, @Res() res: Response) { // Обрабатывает запрос (с req.user) и ответ.
    return this.authService.refresh(req.user, res); // Делегирует сервису с user из гарды.
  }

  @Post('logout') // POST маршрут для выхода.
  @UseGuards(AuthGuard('jwt')) // Использует JWT гарду для валидации.
  async logout(@Req() req: Request, @Res() res: Response) { // Обрабатывает запрос и ответ.
    return this.authService.logout(req.user.id, res); // Делегирует сервису с user.id.
  }
}
```

#### src/auth/auth.service.ts
```typescript
// Сервис аутентификации для логики.
// Обрабатывает токены, куки, хеширование.
// https://docs.nestjs.com/security/authentication

import { BadRequestException, Injectable, UnauthorizedException } from '@nestjs/common'; // Исключения. https://docs.nestjs.com/exception-filters#built-in-exceptions
import { PrismaService } from '../prisma/prisma.service'; // Сервис базы данных.
import { RegisterDto } from './dto/register.dto'; // DTO регистрации.
import { LoginDto } from './dto/login.dto'; // DTO входа.
import * as argon2 from 'argon2'; // Библиотека хеширования. https://www.npmjs.com/package/argon2
import { JwtService } from '@nestjs/jwt'; // Сервис JWT. https://docs.nestjs.com/security/authentication#jwt-functionality
import { ConfigService } from '@nestjs/config'; // Сервис конфигурации.
import { Response } from 'express'; // Тип ответа Express. https://expressjs.com/en/api.html#res

/**
 * AuthService обрабатывает логику аутентификации, генерацию токенов и управление куками.
 * Refresh токены хешируются и хранятся в базе для безопасности.
 * @see https://docs.nestjs.com/security/authentication
 * @see https://www.npmjs.com/package/argon2 для хеширования паролей
 * @see https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#security для HttpOnly cookies
 */
@Injectable() // Декоратор для инъекции.
export class AuthService { // Класс сервиса.
  constructor( // Конструктор с инъекциями.
    private prisma: PrismaService, // Prisma для базы.
    private jwt: JwtService, // JWT для токенов.
    private config: ConfigService, // Конфигурация для env.
  ) {}

  async register(dto: RegisterDto, res: Response) { // Метод регистрации.
    const existingUser = await this.prisma.user.findUnique({ where: { email: dto.email } }); // Проверяет наличие пользователя.
    if (existingUser) throw new BadRequestException('Email уже существует'); // Выбрасывает ошибку, если email занят.

    const hashedPassword = await argon2.hash(dto.password); // Хеширует пароль.
    const user = await this.prisma.user.create({ // Создаёт пользователя.
      data: { email: dto.email, password: hashedPassword }, // Данные пользователя.
    });

    return this.generateTokensAndSetCookies(user.id, user.role, res); // Генерирует токены и устанавливает куки.
  }

  async login(dto: LoginDto, res: Response) { // Метод входа.
    const user = await this.prisma.user.findUnique({ where: { email: dto.email } }); // Находит пользователя.
    if (!user || !(await argon2.verify(user.password, dto.password))) { // Проверяет пароль.
      throw new UnauthorizedException('Неверные учетные данные'); // Выбрасывает ошибку, если неверно.
    }

    return this.generateTokensAndSetCookies(user.id, user.role, res); // Генерирует токены.
  }

  async refresh(user: any, res: Response) { // Метод обновления токена (user из гарды).
    return this.generateTokensAndSetCookies(user.id, user.role, res); // Генерирует новые токены и устанавливает куки.
  }

  async logout(userId: number, res: Response) { // Метод выхода (userId из гарды).
    await this.prisma.user.update({ // Очищает refresh токен в базе.
      where: { id: userId },
      data: { refreshToken: null },
    });
    res.clearCookie('refreshToken'); // Очищает куки. https://expressjs.com/en/api.html#res.clearCookie
    return { message: 'Выход выполнен' }; // Возвращает сообщение.
  }

  private async generateTokensAndSetCookies(userId: number, role: string, res: Response) { // Приватный метод для генерации токенов и установки кук.
    const accessToken = this.jwt.sign({ sub: userId, role }); // Подписывает access токен. https://github.com/auth0/node-jsonwebtoken#jwtsignpayload-secretorprivatekey-options-callback
    const refreshToken = this.jwt.sign({ sub: userId }, { // Подписывает refresh токен.
      secret: this.config.get('JWT_REFRESH_SECRET'), // Секрет из env.
      expiresIn: this.config.get('JWT_REFRESH_EXPIRES_IN') // Время жизни.
    });

    const hashedRefresh = await argon2.hash(refreshToken); // Хеширует refresh токен для базы.
    await this.prisma.user.update({ where: { id: userId }, data: { refreshToken: hashedRefresh } }); // Обновляет базу.

    res.cookie('refreshToken', refreshToken, { // Устанавливает HttpOnly куки для refresh токена.
      httpOnly: true, // Флаг безопасности, предотвращает доступ JS. https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#security
      secure: this.config.get('NODE_ENV') === 'production', // Безопасность в продакшене.
      sameSite: 'strict' // Политика SameSite. https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Set-Cookie/SameSite
    });

    return { accessToken }; // Возвращает access токен в теле ответа.
  }
}
```

#### src/auth/dto/register.dto.ts
```typescript
// DTO регистрации с валидацией.
// https://docs.nestjs.com/pipes#class-validator

import { IsEmail, IsString, MinLength } from 'class-validator'; // Валидаторы. https://github.com/typestack/class-validator

export class RegisterDto { // Класс DTO.
  @IsEmail() // Валидация email.
  email: string; // Поле email.

  @IsString() // Валидация строки.
  @MinLength(6) // Минимальная длина.
  password: string; // Поле пароля.
}
```

#### src/auth/dto/login.dto.ts
```typescript
// DTO входа с валидацией.

import { IsEmail, IsString } from 'class-validator'; // Валидаторы.

export class LoginDto { // Класс DTO.
  @IsEmail() // Валидация email.
  email: string; // Поле email.

  @IsString() // Валидация строки.
  password: string; // Поле пароля.
}
```

#### src/auth/strategies/jwt.strategy.ts
```typescript
// Стратегия JWT для access токена.
// https://docs.nestjs.com/security/authentication#implementing-passport-jwt

import { Injectable } from '@nestjs/common'; // Декоратор инъекции.
import { ConfigService } from '@nestjs/config'; // Сервис конфигурации.
import { PassportStrategy } from '@nestjs/passport'; // Базовая стратегия.
import { ExtractJwt, Strategy } from 'passport-jwt'; // JWT passport. https://www.npmjs.com/package/passport-jwt
import { PrismaService } from '../../prisma/prisma.service'; // Сервис базы данных.

/**
 * JwtStrategy для валидации access токена.
 * @see https://docs.nestjs.com/security/authentication#jwt-functionality
 */
@Injectable() // Декоратор инъекции.
export class JwtStrategy extends PassportStrategy(Strategy, 'jwt') { // Расширяет стратегию.
  constructor(config: ConfigService, private prisma: PrismaService) { // Конструктор.
    super({ // Вызов родительского конструктора с настройками.
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(), // Извлекает токен из заголовка Bearer.
      secretOrKey: config.get('JWT_ACCESS_SECRET'), // Секрет из env.
    });
  }

  async validate(payload: { sub: number; role: string }) { // Метод валидации.
    const user = await this.prisma.user.findUnique({ where: { id: payload.sub } }); // Находит пользователя.
    if (!user) return null; // Возвращает null, если пользователь не найден.
    return { id: user.id, role: user.role }; // Возвращает данные пользователя для req.user.
  }
}
```

#### src/auth/strategies/jwt-refresh.strategy.ts
```typescript
// Стратегия refresh из куки.
// https://docs.nestjs.com/security/authentication#refresh-tokens

import { Injectable } from '@nestjs/common'; // Декоратор.
import { ConfigService } from '@nestjs/config'; // Конфигурация.
import { PassportStrategy } from '@nestjs/passport'; // Базовая.
import { ExtractJwt, Strategy } from 'passport-jwt'; // JWT.
import { Request } from 'express'; // Тип запроса.
import { PrismaService } from '../../prisma/prisma.service'; // База.
import * as argon2 from 'argon2'; // Хеширование.

/**
 * JwtRefreshStrategy для валидации refresh токена из куки.
 * Сравнивает хешированный токен с базой.
 * @see https://docs.nestjs.com/security/authentication#refresh-tokens
 */
@Injectable() // Декоратор.
export class JwtRefreshStrategy extends PassportStrategy(Strategy, 'jwt-refresh') { // Расширяет.
  constructor(config: ConfigService, private prisma: PrismaService) { // Конструктор.
    super({ // Настройки.
      jwtFromRequest: ExtractJwt.fromExtractors([(req: Request) => req.cookies?.refreshToken]), // Извлекает из куки.
      secretOrKey: config.get('JWT_REFRESH_SECRET'), // Секрет.
      passReqToCallback: true, // Передаёт запрос в validate.
    });
  }

  async validate(req: Request, payload: { sub: number }) { // Валидация.
    const user = await this.prisma.user.findUnique({ where: { id: payload.sub } }); // Находит пользователя.
    if (!user || !user.refreshToken) return null; // Проверяет наличие токена.

    const refreshToken = req.cookies?.refreshToken; // Получает токен из куки.
    if (!(await argon2.verify(user.refreshToken, refreshToken))) return null; // Проверяет хеш.

    return { id: user.id, role: user.role }; // Возвращает пользователя для req.user.
  }
}
```

#### src/common/decorators/roles.decorator.ts
```typescript
// Декоратор ролей для метаданных.
// https://docs.nestjs.com/custom-decorators

import { SetMetadata } from '@nestjs/common'; // Установщик метаданных. https://docs.nestjs.com/custom-decorators#setmetadata

export const Roles = (...roles: string[]) => SetMetadata('roles', roles); // Функция декоратора. Устанавливает метаданные 'roles'.
```

#### src/auth/guards/roles.guard.ts
```typescript
// Гарда ролей для RBAC.
// https://docs.nestjs.com/guards

import { CanActivate, ExecutionContext, Injectable } from '@nestjs/common'; // Интерфейсы гарды. https://docs.nestjs.com/guards#canactivate-interface
import { Reflector } from '@nestjs/core'; // Reflector для метаданных. https://docs.nestjs.com/fundamentals/execution-context#reflection-and-metadata
import { Observable } from 'rxjs'; // Тип Observable.

/**
 * RolesGuard для RBAC на основе JWT payload.
 * @see https://docs.nestjs.com/guards
 */
@Injectable() // Декоратор.
export class RolesGuard implements CanActivate { // Реализует гарду.
  constructor(private reflector: Reflector) {} // Инъекция reflector.

  canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> { // Метод CanActivate.
    const requiredRoles = this.reflector.getAllAndOverride<string[]>('roles', [ // Получает метаданные.
      context.getHandler(), // Из обработчика.
      context.getClass(), // Из класса.
    ]);
    if (!requiredRoles) return true; // Разрешает, если роли не требуются.

    const { user } = context.switchToHttp().getRequest(); // Получает пользователя из запроса.
    return requiredRoles.some((role) => user?.role === role); // Проверяет совпадение роли.
  }
}
```

#### src/common/uploads.ts
```typescript
// Конфигурация Multer для загрузки файлов.
// Централизована для всех контроллеров.
// https://docs.nestjs.com/techniques/file-upload

import { diskStorage } from 'multer'; // Движок хранения. https://www.npmjs.com/package/multer#diskstorage
import { extname } from 'path'; // Утилиты пути. https://nodejs.org/api/path.html#path_extname_path
import { v4 as uuid } from 'uuid'; // Генератор UUID. https://www.npmjs.com/package/uuid

/**
 * Конфигурация Multer для загрузки файлов.
 * Локальное дисковое хранилище; расширьте для облака (например, S3).
 * @see https://docs.nestjs.com/techniques/file-upload
 * @see https://www.npmjs.com/package/multer
 */
export const uploadOptions = { // Экспортирует объект опций.
  storage: diskStorage({ // Конфигурация дискового хранения.
    destination: './uploads', // Директория назначения.
    filename: (req, file, cb) => { // Генератор имени файла.
      const filename = `${uuid()}${extname(file.originalname)}`; // Уникальное имя с расширением.
      cb(null, filename); // Коллбэк с именем.
    },
  }),
  fileFilter: (req, file, cb) => { // Фильтр файлов.
    if (!file.mimetype.match(/\/(jpg|jpeg|png|gif)$/)) { // Проверяет типы изображений.
      cb(new Error('Разрешены только изображения'), false); // Отклоняет не-изображения.
    } else {
      cb(null, true); // Принимает.
    }
  },
  limits: { fileSize: 5 * 1024 * 1024 }, // Лимит 5MB. https://www.npmjs.com/package/multer#limits
};
```

#### src/posts/posts.module.ts
```typescript
// Модуль постов.
// https://docs.nestjs.com/modules

import { Module } from '@nestjs/common'; // Декоратор модуля.
import { PostsController } from './posts.controller'; // Контроллер.
import { PostsService } from './posts.service'; // Сервис.
import { PrismaModule } from '../prisma/prisma.module'; // Prisma.

@Module({ // Декоратор.
  imports: [PrismaModule], // Импорты.
  controllers: [PostsController], // Контроллеры.
  providers: [PostsService], // Провайдеры.
})
export class PostsModule {} // Класс модуля.
```

#### src/posts/posts.controller.ts
```typescript
// Контроллер постов для CRUD и избранного.
// Тонкий слой: делегирует сервису.
// https://docs.nestjs.com/controllers

import { Body, Controller, Delete, Get, Param, Post, Put, Query, Req, UseGuards, UseInterceptors, UploadedFile } from '@nestjs/common'; // Декораторы и типы. https://docs.nestjs.com/controllers#request-object
import { PostsService } from './posts.service'; // Импортирует сервис.
import { CreatePostDto } from './dto/create-post.dto'; // Импортирует DTO.
import { UpdatePostDto } from './dto/update-post.dto'; // Импортирует DTO.
import { AuthGuard } from '@nestjs/passport'; // Гарда для passport. https://docs.nestjs.com/guards
import { RolesGuard } from '../auth/guards/roles.guard'; // Гарда ролей.
import { Roles } from '../common/decorators/roles.decorator'; // Декоратор ролей.
import { FileInterceptor } from '@nestjs/platform-express'; // Интерсептор для файлов. https://docs.nestjs.com/techniques/file-upload
import { uploadOptions } from '../common/uploads'; // Конфигурация multer.
import { Request } from 'express'; // Тип Request из Express. https://expressjs.com/en/api.html#req

/**
 * PostsController обрабатывает маршруты постов, включая CRUD и избранное.
 * Вся логика делегируется PostsService.
 * @see https://docs.nestjs.com/controllers
 */
@Controller('posts') // Декоратор контроллера с путём.
export class PostsController { // Класс контроллера.
  constructor(private readonly postsService: PostsService) {} // Инъекция сервиса.

  @Get() // GET маршрут для списка постов.
  async findAll(@Query('category') category?: string) { // Обрабатывает query param для фильтра по категории.
    return this.postsService.findAll(category); // Делегирует сервису.
  }

  @Get(':slug') // GET маршрут для одного поста.
  async findOne(@Param('slug') slug: string) { // Обрабатывает param slug.
    return this.postsService.findOne(slug); // Делегирует сервису.
  }

  @Post() // POST маршрут для создания поста.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Использует гарды для авторизации и ролей.
  @Roles('admin') // Только для админа.
  @UseInterceptors(FileInterceptor('image', uploadOptions)) // Интерсептор для загрузки изображения.
  async create(@Body() dto: CreatePostDto, @UploadedFile() file: Express.Multer.File) { // Обрабатывает тело и файл.
    return this.postsService.create(dto, file); // Делегирует сервису.
  }

  @Put(':id') // PUT маршрут для обновления поста.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды.
  @Roles('admin') // Админ.
  async update(@Param('id') id: string, @Body() dto: UpdatePostDto) { // Обрабатывает param и тело.
    return this.postsService.update(parseInt(id), dto); // Делегирует сервису (id как number).
  }

  @Delete(':id') // DELETE маршрут для удаления поста.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды.
  @Roles('admin') // Админ.
  async delete(@Param('id') id: string) { // Обрабатывает param.
    return this.postsService.delete(parseInt(id)); // Делегирует сервису.
  }

  @Post(':id/favorite') // POST маршрут для добавления в избранное.
  @UseGuards(AuthGuard('jwt')) // Только для авторизованных.
  async addToFavorite(@Param('id') id: string, @Req() req: Request) { // Обрабатывает param и req для user.id.
    return this.postsService.addToFavorite(parseInt(id), req.user.id); // Делегирует сервису с userId.
  }

  @Delete(':id/favorite') // DELETE маршрут для удаления из избранного.
  @UseGuards(AuthGuard('jwt')) // Только для авторизованных.
  async removeFromFavorite(@Param('id') id: string, @Req() req: Request) { // Обрабатывает param и req.
    return this.postsService.removeFromFavorite(parseInt(id), req.user.id); // Делегирует сервису.
  }
}
```

#### src/posts/posts.service.ts
```typescript
// Сервис постов для логики.
// Обрабатывает Prisma вызовы, включая фильтр и избранное.
// https://docs.nestjs.com/providers#services

import { Injectable, NotFoundException } from '@nestjs/common'; // Декоратор и исключения. https://docs.nestjs.com/providers#services
import { PrismaService } from '../prisma/prisma.service'; // Сервис базы данных.
import { CreatePostDto } from './dto/create-post.dto'; // DTO создания.
import { UpdatePostDto } from './dto/update-post.dto'; // DTO обновления.

/**
 * PostsService обрабатывает логику постов, включая CRUD, фильтр по категориям и избранное.
 * @see https://docs.nestjs.com/providers#services
 */
@Injectable() // Декоратор для инъекции.
export class PostsService { // Класс сервиса.
  constructor(private prisma: PrismaService) {} // Инъекция prisma.

  async findAll(category?: string) { // Метод для списка постов с фильтром по категории.
    const where = category ? { category: { name: category } } : {}; // Условие для фильтра.
    return this.prisma.post.findMany({ // Запрос с включением связей.
      where,
      include: { category: true, author: { select: { email: true } } },
    });
  }

  async findOne(slug: string) { // Метод для одного поста.
    const post = await this.prisma.post.findUnique({ // Запрос с включением связей.
      where: { slug },
      include: { category: true, author: { select: { email: true } } },
    });
    if (!post) throw new NotFoundException('Пост не найден'); // Выбрасывает ошибку, если не найден.
    return post; // Возвращает пост.
  }

  async create(dto: CreatePostDto, file?: Express.Multer.File) { // Метод создания поста.
    if (file) dto['imagePath'] = `/uploads/${file.filename}`; // Устанавливает путь изображения с префиксом для статического обслуживания.
    return this.prisma.post.create({ data: dto }); // Создаёт пост в базе.
  }

  async update(id: number, dto: UpdatePostDto) { // Метод обновления поста.
    return this.prisma.post.update({ where: { id }, data: dto }); // Обновляет пост.
  }

  async delete(id: number) { // Метод удаления поста.
    return this.prisma.post.delete({ where: { id } }); // Удаляет пост.
  }

  async addToFavorite(postId: number, userId: number) { // Метод добавления в избранное.
    return this.prisma.user.update({ // Обновляет связь многие-ко-многим.
      where: { id: userId },
      data: { favorites: { connect: { id: postId } } },
      include: { favorites: true }, // Включает избранное в ответ.
    });
  }

  async removeFromFavorite(postId: number, userId: number) { // Метод удаления из избранного.
    return this.prisma.user.update({ // Обновляет связь.
      where: { id: userId },
      data: { favorites: { disconnect: { id: postId } } },
      include: { favorites: true }, // Включает избранное.
    });
  }
}
```

#### src/posts/dto/create-post.dto.ts
```typescript
// DTO создания поста с валидацией.
// https://docs.nestjs.com/pipes#class-validator

import { IsInt, IsString } from 'class-validator'; // Валидаторы. https://github.com/typestack/class-validator

export class CreatePostDto { // Класс DTO.
  @IsString() // Валидация строки.
  title: string; // Заголовок.

  @IsString() // Строка.
  slug: string; // Slug.

  @IsString() // Строка.
  content: string; // Содержимое.

  @IsInt() // Целое число.
  authorId: number; // ID автора.

  @IsInt() // Целое.
  categoryId: number; // ID категории.
}
```

#### src/posts/dto/update-post.dto.ts
```typescript
// DTO обновления поста как частичный.
// https://docs.nestjs.com/openapi/mapped-types#partial

import { PartialType } from '@nestjs/mapped-types'; // Утилита для частичных типов. https://docs.nestjs.com/openapi/mapped-types#partial
import { CreatePostDto } from './create-post.dto'; // Базовый DTO.

export class UpdatePostDto extends PartialType(CreatePostDto) {} // Расширяет как частичный тип.
```

#### src/users/users.module.ts
```typescript
// Модуль пользователей.
// https://docs.nestjs.com/modules

import { Module } from '@nestjs/common'; // Декоратор модуля.
import { UsersController } from './users.controller'; // Контроллер.
import { UsersService } from './users.service'; // Сервис.
import { PrismaModule } from '../prisma/prisma.module'; // Prisma.

@Module({ // Декоратор.
  imports: [PrismaModule], // Импорты.
  controllers: [UsersController], // Контроллеры.
  providers: [UsersService], // Провайдеры.
})
export class UsersModule {} // Класс модуля.
```

#### src/users/users.controller.ts
```typescript
// Контроллер пользователей для CRUD.
// Тонкий слой: делегирует сервису.
// https://docs.nestjs.com/controllers

import { Body, Controller, Delete, Get, Param, Post, Put, UseGuards } from '@nestjs/common'; // Декораторы и типы. https://docs.nestjs.com/controllers#request-object
import { UsersService } from './users.service'; // Импортирует сервис.
import { CreateUserDto } from './dto/create-user.dto'; // DTO создания.
import { UpdateUserDto } from './dto/update-user.dto'; // DTO обновления.
import { AuthGuard } from '@nestjs/passport'; // Гарда. https://docs.nestjs.com/guards
import { RolesGuard } from '../auth/guards/roles.guard'; // Гарда ролей.
import { Roles } from '../common/decorators/roles.decorator'; // Декоратор.

/**
 * UsersController обрабатывает маршруты пользователей (CRUD).
 * Вся логика делегируется UsersService.
 * @see https://docs.nestjs.com/controllers
 */
@Controller('users') // Декоратор с путём.
export class UsersController { // Класс.
  constructor(private readonly usersService: UsersService) {} // Инъекция сервиса.

  @Get() // GET для списка пользователей.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды для авторизации и ролей.
  @Roles('admin') // Только админ.
  async findAll() { // Метод.
    return this.usersService.findAll(); // Делегирует сервису.
  }

  @Get(':id') // GET для одного пользователя.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды.
  @Roles('admin') // Админ.
  async findOne(@Param('id') id: string) { // Обрабатывает param.
    return this.usersService.findOne(parseInt(id)); // Делегирует.
  }

  @Post() // POST для создания пользователя.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды.
  @Roles('admin') // Админ.
  async create(@Body() dto: CreateUserDto) { // Обрабатывает тело.
    return this.usersService.create(dto); // Делегирует (хеширование в сервисе).
  }

  @Put(':id') // PUT для обновления.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды.
  @Roles('admin') // Админ.
  async update(@Param('id') id: string, @Body() dto: UpdateUserDto) { // Param и тело.
    return this.usersService.update(parseInt(id), dto); // Делегирует.
  }

  @Delete(':id') // DELETE для удаления.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды.
  @Roles('admin') // Админ.
  async delete(@Param('id') id: string) { // Param.
    return this.usersService.delete(parseInt(id)); // Делегирует.
  }
}
```

#### src/users/users.service.ts
```typescript
// Сервис пользователей для логики.
// Обрабатывает Prisma вызовы и хеширование.
// https://docs.nestjs.com/providers#services

import { Injectable, NotFoundException } from '@nestjs/common'; // Декоратор и исключения.
import { PrismaService } from '../prisma/prisma.service'; // База.
import { CreateUserDto } from './dto/create-user.dto'; // DTO.
import { UpdateUserDto } from './dto/update-user.dto'; // DTO.
import * as argon2 from 'argon2'; // Хеширование. https://www.npmjs.com/package/argon2

/**
 * UsersService обрабатывает логику пользователей, включая CRUD и хеширование паролей.
 * @see https://docs.nestjs.com/providers#services
 */
@Injectable() // Декоратор.
export class UsersService { // Класс.
  constructor(private prisma: PrismaService) {} // Инъекция.

  async findAll() { // Метод для списка пользователей.
    return this.prisma.user.findMany({ select: { id: true, email: true, role: true, createdAt: true } }); // Запрос без пароля.
  }

  async findOne(id: number) { // Метод для одного пользователя.
    const user = await this.prisma.user.findUnique({ // Запрос.
      where: { id },
      select: { id: true, email: true, role: true, createdAt: true },
    });
    if (!user) throw new NotFoundException('Пользователь не найден'); // Ошибка, если не найден.
    return user; // Возвращает.
  }

  async create(dto: CreateUserDto) { // Метод создания.
    const hashedPassword = await argon2.hash(dto.password); // Хеширует пароль.
    return this.prisma.user.create({ data: { ...dto, password: hashedPassword } }); // Создаёт.
  }

  async update(id: number, dto: UpdateUserDto) { // Метод обновления.
    if (dto.password) dto.password = await argon2.hash(dto.password); // Хеширует, если пароль обновляется.
    return this.prisma.user.update({ where: { id }, data: dto }); // Обновляет.
  }

  async delete(id: number) { // Метод удаления.
    return this.prisma.user.delete({ where: { id } }); // Удаляет.
  }
}
```

#### src/users/dto/create-user.dto.ts
```typescript
// DTO создания пользователя с валидацией.

import { IsEmail, IsString, MinLength, IsOptional } from 'class-validator'; // Валидаторы. https://github.com/typestack/class-validator

export class CreateUserDto { // Класс.
  @IsEmail() // Валидация email.
  email: string; // Email.

  @IsString() // Строка.
  @MinLength(6) // Мин длина.
  password: string; // Пароль.

  @IsOptional() // Опционально.
  @IsString() // Строка.
  role?: string; // Роль (default 'user').
}
```

#### src/users/dto/update-user.dto.ts
```typescript
// DTO обновления пользователя как частичный.

import { PartialType } from '@nestjs/mapped-types'; // Утилита. https://docs.nestjs.com/openapi/mapped-types#partial
import { CreateUserDto } from './create-user.dto'; // Базовый.

export class UpdateUserDto extends PartialType(CreateUserDto) {} // Частичный тип.
```

#### src/categories/categories.module.ts
```typescript
// Модуль категорий.
// https://docs.nestjs.com/modules

import { Module } from '@nestjs/common'; // Декоратор.
import { CategoriesController } from './categories.controller'; // Контроллер.
import { CategoriesService } from './categories.service'; // Сервис.
import { PrismaModule } from '../prisma/prisma.module'; // Prisma.

@Module({ // Декоратор.
  imports: [PrismaModule], // Импорты.
  controllers: [CategoriesController], // Контроллеры.
  providers: [CategoriesService], // Провайдеры.
})
export class CategoriesModule {} // Класс модуля.
```

#### src/categories/categories.controller.ts
```typescript
// Контроллер категорий для CRUD.
// Тонкий слой: делегирует сервису.
// https://docs.nestjs.com/controllers

import { Body, Controller, Delete, Get, Param, Post, Put, UseGuards } from '@nestjs/common'; // Декораторы. https://docs.nestjs.com/controllers#request-object
import { CategoriesService } from './categories.service'; // Сервис.
import { CreateCategoryDto } from './dto/create-category.dto'; // DTO.
import { UpdateCategoryDto } from './dto/update-category.dto'; // DTO.
import { AuthGuard } from '@nestjs/passport'; // Гарда.
import { RolesGuard } from '../auth/guards/roles.guard'; // Гарда ролей.
import { Roles } from '../common/decorators/roles.decorator'; // Декоратор.

/**
 * CategoriesController обрабатывает маршруты категорий (CRUD).
 * Вся логика делегируется CategoriesService.
 * @see https://docs.nestjs.com/controllers
 */
@Controller('categories') // Путь.
export class CategoriesController { // Класс.
  constructor(private readonly categoriesService: CategoriesService) {} // Инъекция.

  @Get() // GET для списка категорий.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды.
  @Roles('admin') // Админ.
  async findAll() { // Метод.
    return this.categoriesService.findAll(); // Делегирует.
  }

  @Get(':id') // GET для одной категории.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды.
  @Roles('admin') // Админ.
  async findOne(@Param('id') id: string) { // Param.
    return this.categoriesService.findOne(parseInt(id)); // Делегирует.
  }

  @Post() // POST для создания.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды.
  @Roles('admin') // Админ.
  async create(@Body() dto: CreateCategoryDto) { // Тело.
    return this.categoriesService.create(dto); // Делегирует.
  }

  @Put(':id') // PUT для обновления.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды.
  @Roles('admin') // Админ.
  async update(@Param('id') id: string, @Body() dto: UpdateCategoryDto) { // Param и тело.
    return this.categoriesService.update(parseInt(id), dto); // Делегирует.
  }

  @Delete(':id') // DELETE для удаления.
  @UseGuards(AuthGuard('jwt'), RolesGuard) // Гарды.
  @Roles('admin') // Админ.
  async delete(@Param('id') id: string) { // Param.
    return this.categoriesService.delete(parseInt(id)); // Делегирует.
  }
}
```

#### src/categories/categories.service.ts
```typescript
// Сервис категорий для логики.
// Обрабатывает Prisma вызовы.
// https://docs.nestjs.com/providers#services

import { Injectable, NotFoundException } from '@nestjs/common'; // Декоратор и исключения.
import { PrismaService } from '../prisma/prisma.service'; // База.
import { CreateCategoryDto } from './dto/create-category.dto'; // DTO.
import { UpdateCategoryDto } from './dto/update-category.dto'; // DTO.

/**
 * CategoriesService обрабатывает логику категорий, включая CRUD.
 * @see https://docs.nestjs.com/providers#services
 */
@Injectable() // Декоратор.
export class CategoriesService { // Класс.
  constructor(private prisma: PrismaService) {} // Инъекция.

  async findAll() { // Метод для списка категорий.
    return this.prisma.category.findMany(); // Запрос.
  }

  async findOne(id: number) { // Метод для одной категории.
    const category = await this.prisma.category.findUnique({ where: { id } }); // Запрос.
    if (!category) throw new NotFoundException('Категория не найдена'); // Ошибка.
    return category; // Возвращает.
  }

  async create(dto: CreateCategoryDto) { // Метод создания.
    return this.prisma.category.create({ data: dto }); // Создаёт.
  }

  async update(id: number, dto: UpdateCategoryDto) { // Метод обновления.
    return this.prisma.category.update({ where: { id }, data: dto }); // Обновляет.
  }

  async delete(id: number) { // Метод удаления.
    return this.prisma.category.delete({ where: { id } }); // Удаляет.
  }
}
```

#### src/categories/dto/create-category.dto.ts
```typescript
// DTO создания категории с валидацией.

import { IsString } from 'class-validator'; // Валидаторы.

export class CreateCategoryDto { // Класс.
  @IsString() // Валидация строки.
  name: string; // Имя категории.
}
```

#### src/categories/dto/update-category.dto.ts
```typescript
// DTO обновления категории как частичный.

import { PartialType } from '@nestjs/mapped-types'; // Утилита. https://docs.nestjs.com/openapi/mapped-types#partial
import { CreateCategoryDto } from './create-category.dto'; // Базовый.

export class UpdateCategoryDto extends PartialType(CreateCategoryDto) {} // Частичный тип.
```

#### docs/auth-flow.md
```
// Документация потока аутентификации.
// Объясняет обработку токенов.

# Поток аутентификации

1. Регистрация/Вход: Генерируются accessToken (JWT, короткоживущий) и refreshToken (долгоживущий). // Токены для сессии.
   - accessToken возвращается в теле ответа. // Для API вызовов.
   - refreshToken устанавливается в HttpOnly куки (безопасно, недоступно для JS). // Безопасность.
   - Хеш refreshToken хранится в базе для валидации. // Предотвращает повторное использование.

2. Защищённые запросы: Отправляется accessToken в заголовке Authorization. // Токен Bearer.

3. При 401 (истёкший access): Клиент вызывает /auth/refresh с куки (refreshToken). // Эндпоинт обновления.
   - Сервер проверяет refreshToken с хешем в базе. // Используется argon2.verify.
   - Если валиден, выдаётся новый accessToken (и новый refreshToken, обновляя хеш в базе). // Ротация.

4. Выход: Очищаются куки и удаляется хеш из базы. // Отзыв токена.

Почему хеш refresh хранится в базе? Предотвращает повторное использование при краже; позволяет отзыв. // Лучшая практика безопасности.
См.: https://auth0.com/blog/refresh-token-rotation-and-reuse-detection-in-node-js/ // Ссылка.
```