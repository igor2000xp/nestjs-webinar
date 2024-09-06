# Контроллеры в NestJS и реализация CRUD операций
# Create 07-books-controller branch

## 1. Что такое [контроллеры в NestJS](https://docs.nestjs.com/controllers)?

Контроллеры (Controllers) в NestJS отвечают за обработку входящих HTTP-запросов, связывая их с соответствующими методами, которые реализуют бизнес-логику. Контроллеры принимают запросы, делегируют задачи соответствующим сервисам и возвращают ответы клиенту. Они определяют маршруты и методы, которые могут быть вызваны клиентами, например, через REST API.

Каждый метод контроллера обычно ассоциирован с определенным HTTP-методом (GET, POST, PUT, DELETE и т.д.) и маршрутом, который определяет, как запросы должны быть обработаны.

### 1.1 Основные функции контроллеров:
- **Обработка HTTP-запросов**: Контроллеры принимают запросы от клиентов, таких как веб-приложения или мобильные приложения.
- **Маршрутизация**: Контроллеры связывают запросы с соответствующими методами, которые должны их обрабатывать.
- **Делегирование бизнес-логики**: Контроллеры сами по себе не содержат бизнес-логику, а делегируют её сервисам.
- **Формирование ответов**: Контроллеры возвращают ответы клиентам после обработки запросов.

### Пример базового контроллера:

```typescript
import { Controller, Get } from '@nestjs/common';

@Controller('books')
export class BooksController {
    @Get()
    getAll() {
        return 'This action returns all books';
    }
    @Get('my')
    getMyBooks() {
        return 'This action delete all my books';
    }
}
```

В этом примере:
 - **@Controller('books')**: Декоратор, который помечает класс как контроллер и определяет базовый маршрут для всех методов внутри этого контроллера.
 - **@Get()**: Декоратор, который определяет метод как обработчик GET-запросов для маршрута `/books`.
 - **@Get('my')**: Декоратор, который определяет метод как обработчик GET-запросов для маршрута `/books/my`.

## 2. Реализация CRUD операций в контроллере
   Давайте расширим наш контроллер и добавим в него методы для выполнения операций CRUD (Create, Read, Update, Delete) с книгами.

### 2.1 Структура контроллера
Наш контроллер будет иметь следующие методы:

#### GET /books - Получить список всех книг.
#### GET /books/:id - Получить конкретную книгу по её ID.
#### POST /books - Создать новую книгу
#### PUT /books/:id - Обновить информацию о книге.
#### DELETE /books/:id - Удалить книгу.

### 2.2 Добавление методов CRUD в контроллер
Теперь добавим методы в наш контроллер:

```typescript
import {
    Body,
    Controller,
    Delete,
    Get,
    Param,
    Post,
    Put,
} from '@nestjs/common';
import { BooksService } from './books.service';

@Controller('books')
export class BooksController {
    constructor(private readonly booksService: BooksService) {}

    // Получить список всех книг
    @Get()
    async getAllBooks() {
        // необходимо вызвать соответствующий метод сервиса и вернуть результат
        //const result = await this.booksService.someMethod();
        //return result
    }

    // Получить книгу по ID
    @Get(':id')
    async getBookById(@Param('id') id: number) {
        // необходимо вызвать соответствующий метод сервиса и вернуть результат
        //const result = await this.booksService.someMethod();
        //return result
    }

    // Создать новую книгу
    @Post()
    async createBook(@Body() bookDto: any) {
        // необходимо вызвать соответствующий метод сервиса и вернуть результат
        //const result = await this.booksService.someMethod();
    }

    // Обновить информацию о книге
    @Put(':id')
    async updateBook(@Param('id') id: number, @Body() bookDto: any) {
        // необходимо вызвать соответствующий метод сервиса и вернуть результат
        //const result = await this.booksService.someMethod();
    }

    // Удалить книгу
    @Delete(':id')
    async deleteBook(@Param('id') id: number) {
        // необходимо вызвать соответствующий метод сервиса и вернуть результат
        //const result = await this.booksService.someMethod();
    }
}
```

### 2.4 Описание методов:
 - **getAllBooks()**: Этот метод обрабатывает GET-запрос на маршрут `/books` и возвращает список всех книг.

 -  **getBookById(@Param('id') id: number)**: Этот метод обрабатывает GET-запрос на маршрут `/books/:id` и возвращает конкретную книгу по её ID.  

 *Используется декоратор **@Param()** для получения значения ID из маршрута.  
Если в маршруте указано **":param"**, то это значит маршрут содержит динамический (**URI**) параметр под псевдонимом **param**.
 Например, '/books/1234', 1234 - это параметр (id книги), который в коде метода контроллера, в нашем случае (`@Param('id') id: number`) будет доступен в переменной **id***

 - **createBook(@Body() book: Book)**: Этот метод обрабатывает POST-запрос на маршрут `/books` и создает новую книгу. 
Он делегирует задачу сервису `BooksService`, который выполняет бизнес-логику.  

*Декоратор **@Body()** используется для получения данных книги из тела запроса.*

 -  **updateBook(@Param('id') id: number, @Body() book: Book)**:
 - Этот метод обрабатывает PUT-запрос на маршрут `/books/:id` и обновляет информацию о книге с указанным ID. 
 - Значение ID передается через параметр маршрута, а данные для обновления — через тело запроса.

 -  **deleteBook(@Param('id') id: number)**: Этот метод обрабатывает DELETE-запрос на маршрут `/books/:id` и удаляет книгу с указанным ID.

## 3. Реализация методов в сервисе

### 3.1 Создание методов по получению и созданию книг в сервисе
Теперь давайте реализуем методы по получению и созданию книг в сервисе (слой **BLL**)

```typescript
import { Injectable } from '@nestjs/common';
import { BooksRepository } from './books.repository';
import { Book } from './book.entity';

@Injectable()
export class BooksService {
 constructor(private readonly booksRepository: BooksRepository) {}

 // Get a list of books
 async getAllBooks(): Promise<Book[]> {
  return this.booksRepository.findAll();
 }

 // Get book by ID
 async getBookById(id: number): Promise<Book> {
  return this.booksRepository.findOne(id);
 }

 // Create a new book
 async createBook(dto: any): Promise<void> {
  const book = new Book();
  book.title = dto.title;
  book.ageRestrict = dto.ageRestrict;
  book.author = dto.author;

  await this.booksRepository.save(book);
 }
}
```

Обратим внимание на метод `createBook`:
 - сначала мы создаем новый экземпляр сущности (entity) `book` с помощью конструктора  
*Примечание: Поскольку мы используем сторонние библиотеки (typeORM, NestJS), инициализация свойств экземпляра класса Book происходит вне конструктора, чтобы обеспечить совместимость с механизмами, используемыми TypeORM*
 - затем присваиваем нужные значения свойствам книги.
- задействуем соответствующие методы в контроллере.

### 4. Создание и получение книг через front-end 

Давйте откроем front-end приложение, создадим несколько книг через форму создания, затем запросим список книг и перейдем на страничку конкретной книги (запросим книгу по id).  

В консоли разработчика браузера, во вкладке network мы увидим запросы и ответы, которые отправляет и получает front-end приложение. Ура ;)

# Install the OpenAPI - Swagger

```bash
npm install --save @nestjs/swagger and cookie-parser
 npm install cookie-parser

```


## Change main.ts
```TypeScript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ConfigService } from '@nestjs/config';
import { ConfigurationType } from './core/config/configurationType';
import { HttpStatus, ValidationPipe } from '@nestjs/common';
import cookieParser from 'cookie-parser';
import { DocumentBuilder, SwaggerModule } from '@nestjs/swagger';

async function bootstrap() {
 const app = await NestFactory.create(AppModule);

 //подключение глобального валидационного pipe https://docs.nestjs.com/techniques/validation
 app
         .useGlobalPipes(
                 new ValidationPipe({
                  forbidNonWhitelisted: true,
                  whitelist: true,
                  transform: true,
                 }),
         )
         .use(cookieParser());

 //разрешены запросы с любых доменов
 app.enableCors({
  origin: '*', // Разрешает запросы с любых доменов
  methods: 'GET,HEAD,PUT,PATCH,POST,DELETE', // Разрешенные методы
  credentials: true, // Включает передачу cookies
  allowedHeaders: ['Authorization', 'Content-Type'],
  maxAge: 86400,
  preflightContinue: false,
  optionsSuccessStatus: HttpStatus.NO_CONTENT,
 });

 const swaggerConfig = new DocumentBuilder()
         .setTitle('Description of BD for Books and Auth')
         .setDescription('Docs for REST API')
         .setVersion('1.0.0')
         .addTag('Created by Igor')
         .addBearerAuth({
          description: `[just text field] Please enter token in following format: Bearer <JWT>`,
          name: 'Authorization',
          bearerFormat: 'Bearer',
          scheme: 'Bearer',
          type: 'http',
          in: 'Header',
         })
         .addGlobalParameters({
          in: 'header',
          allowEmptyValue: true,
          required: false,
          name: 'Content-Language',
          schema: {
           enum: ['ru', 'en'],
          },
         })
         .build();
 const document = SwaggerModule.createDocument(app, swaggerConfig);
 SwaggerModule.setup('docs', app, document, {
  useGlobalPrefix: true,
  swaggerOptions: { defaultModelsExpandDepth: -1 },
 });

 //получение конфиг сервиса https://docs.nestjs.com/techniques/configuration#using-in-the-maints
 const configService = app.get(ConfigService<ConfigurationType>);
 const port = configService.get('apiSettings.PORT', { infer: true })!;

 await app.listen(port);
}
bootstrap();

```
