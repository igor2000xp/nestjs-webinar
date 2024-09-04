# [Request lifecycle](https://docs.nestjs.com/faq/request-lifecycle#request-lifecycle) и [Валидация входных параметров в nestJS](https://docs.nestjs.com/techniques/validation#auto-validation).  

## 1. Request lifecycle (Жизненный цикл запроса):
В NestJS каждый входящий запрос проходит через несколько этапов. Это включает маршрутизацию, выполнение middleware, guard, interceptor, и pipe.
Цикл начинается с маршрутизации запроса к соответствующему контроллеру, затем запрос может быть модифицирован или отфильтрован на каждом этапе, прежде чем достигнет метода обработчика.

![req_lifecycle](https://production-it-incubator.s3.eu-central-1.amazonaws.com/file-manager/Image/b73e1cc9-eb2a-40fc-9097-f1d847453855_req_lifecycle.png)

## 2. Создание DTO для валидации

Валидация входных данных является важным аспектом разработки приложений.
Она позволяет убедиться, что данные, полученные от клиента, соответствуют ожидаемому формату и типам,
прежде чем они будут использованы для создания или обновления сущности.
В NestJS процесс валидации данных легко организовать с использованием **DTO (Data Transfer Object)** и библиотеки **class-validator**.  

Валидация происходит благодаря глобальному **ValidationPipe**

Первым шагом является создание DTO (Data Transfer Object), который будет использоваться для передачи данных и их валидации 
при создании и обновлении книги.  
Требования для валидации:
 - минимальное возрастное ограничение у книги - 5 лет (детские книги не принимаем ;))
 - максимально возрастное ограничение у книги - 120 лет (значение должно соответствовать здравому смыслу ;))
 - длинна названия книги не должна быть меньше 2х символов

### 2.1 DTO для создания книги

Создайте файл `create-book.dto.ts` в директории `src/modules/books/dto/`:

```typescript
import { IsInt, IsString, Max, Min, MinLength } from 'class-validator';

export class CreateBookDto {
    @IsString()
    @MinLength(2)
    title: string;

    @IsInt()
    @Min(5)
    @Max(120)
    ageRestriction: number;

    @IsString()
    author: string;

    @IsOptional()
    @IsString()
    image?: string;
}

```

### 2.2 DTO для обновления книги
Для обновления книги мы можем использовать схожий DTO, но с учётом того, что не все поля могут быть обязательными. 
Например, пользователь может обновить только одно из свойств книги.

Создайте файл `update-book.dto.ts` в той же директории:
```typescript
import {
    IsInt,
    IsOptional,
    IsString,
    Max,
    Min,
    MinLength,
} from 'class-validator';
import { PartialType } from '@nestjs/mapped-types';
import { CreateBookDto } from './create-book.dto';

export class UpdateBookDto {
    @IsString()
    @IsOptional()
    @MinLength(2)
    title?: string;

    @IsInt()
    @Min(5)
    @Max(120)
    @IsOptional()
    ageRestriction?: number;

    @IsString()
    @IsOptional()
    author?: string;
}

//чтобы избежать дублирования dto можно использовать встроенную утилиту PartialType, которая делает ве поля опциональными
//export class UpdateBookDto extends PartialType(CreateBookDto) {}
```
В DTO для обновления используются декораторы `@IsOptional()`, чтобы сделать поля необязательными.

## 3. Валидация в контроллере
   Теперь давайте обновим наш контроллер для использования DTO и валидации данных при создании и обновлении книг.

Откройте файл `books.controller.ts` и обновите методы `createBook` и `updateBook` (вместо `any` укажите в качестве типов соответствующие DTO):

```typescript
  @Post()
async createBook(@Body() bookDto: CreateBookDto) {
    // необходимо вызвать соответствующий метод сервиса и вернуть результат
    await this.booksService.createBook(bookDto);
}

// Обновить информацию о книге
@Put(':id')
async updateBook(@Param('id') id: number, @Body() bookDto: UpdateBookDto) {
    // необходимо вызвать соответствующий метод сервиса и вернуть результат
    //const result = await this.booksService.someMethod();
}
```

Так же, чтобы nestJS применил валидацию, необходимо в `main.ts` добавить глобальный **validation pipe**
(в нашем приложении уже добавлен изначально).

```typescript filename="main.ts"
  app.useGlobalPipes(new ValidationPipe());
```

Попробуем сделать запрос с фронта с невалидными данными и увидим ошибку, с указанием какие поля и какие значения не верны.
