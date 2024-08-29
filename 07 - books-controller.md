# Контроллеры в NestJS и реализация CRUD операций

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

 // Получить список всех книг
 async getAllBooks(): Promise<Book[]> {
  return this.booksRepository.findAll();
 }

 // Получить книгу по ID
 async getBookById(id: number): Promise<Book> {
  return this.booksRepository.findOne(id);
 }

 // Создать новую книгу
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