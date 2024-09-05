# Добавление новой сущности в проект

В этом разделе мы добавим новую сущность в проект по аналогии с сущностью `User`. 
Мы создадим сущность `Book`, опишем её поля, а также с помощью Nest CLI создадим модуль,
контроллер, сервис и репозиторий для работы с книгами.


## 1. Установка [Nest CLI](https://docs.nestjs.com/cli/overview)

Если Nest CLI не установлен на вашем локальном компьютере, 
выполните следующие шаги для его установки:

### 1.1 Установка Nest CLI глобально

Для установки Nest CLI глобально используйте следующую команду:

```bash
npm install -g @nestjs/cli
```

### 1.2 Проверка установки
После установки вы можете проверить, успешно ли установлен Nest CLI, выполнив команду:

```bash
nest --version
```

Если версия Nest CLI отображается, установка прошла успешно, и вы можете продолжить работу.

Эта команда установит Nest CLI глобально,
что позволит вам использовать его для создания модулей, 
контроллеров, сервисов и других компонентов проекта.

## 2. Создание сущности `Book`

Начнем с создания сущности `Book`, которая будет хранить информацию о книгах в нашей базе данных.
В нашем проекте мы используем **code first** подход. То есть описываем данные, которые будет храниться с БД
с помощью классов в коде. Эти классы с помощью ORM, будут превращены в таблицы в БД.

### 2.1 Описание сущности `Book`

Создайте новый файл `books.entity.ts` в директории `src/modules/books/`. 
Опишите сущность следующим образом:

```typescript
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';
import { BaseEntity } from '../../core/config/entity/base.entity';

@Entity('books')
export class Book extends BaseEntity {
    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    title: string;

    @Column()
    author: string;

    @Column()
    ageRestriction: number; //возрастные ограничения на книгу

    @Column({ nullable: true })
    ownerId: number; //id пользователя, который добавил книгу

    @Column({ nullable: true })
    image?: string;
}

```

## 3. Создание модуля, контроллера, сервиса и репозитория для Books
   Теперь мы используем Nest CLI для генерации необходимых файлов и классов для работы с сущностью Books.

### 3.1 Генерация модуля

Для создания модуля Books выполните команду:
``` bash
nest g module modules/books
```

### 3.2 Генерация контроллера
Для создания контроллера Books выполните команду:

```bash
nest g controller modules/books --no-spec
```

### 3.3 Генерация сервиса
Для создания сервиса Books выполните команду:

```bash
nest g service modules/books --no-spec
```

### 3.4 Создание и регистрация собственного репозитория
В нашем проекте используется собственный слой репозитория, 
в который инжектируется TypeOrmRepository. 
Давайте создадим и зарегистрируем репозиторий для сущности Books.

#### 3.4.1 Создание репозитория
Создайте новый файл `books.repository.ts` в директории `src/modules/books/`. 
Опишите репозиторий следующим образом:

```
import { Injectable, NotFoundException } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { Book } from './book.entity';

@Injectable()
export class BooksRepository {
  constructor(
    @InjectRepository(Book)
    private readonly booksORMRepository: Repository<Book>,
  ) {}

  async save(book: Book): Promise<Book> {
    return this.booksORMRepository.save(book);
  }

  async findAll(): Promise<Book[]> {
    return this.booksORMRepository.find();
  }

  async findOneOrNotFoundFail(id: number): Promise<Book> {
    const result = await this.booksORMRepository.findOne({ where: { id } });

    if (!result) {
      throw new NotFoundException('book not found'); //тут код прервется и выдаст ошибку, которую nestjs отправит в ответе
    }

    return result;
  }

  async remove(id: number): Promise<void> {
    await this.booksORMRepository.delete(id);
  }
}
```

#### 3.4.2 Регистрация репозитория
Откройте файл `books.module.ts`, который находится в src/modules/books/, 
и добавьте регистрацию репозитория в модуле:
```
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { BooksService } from './books.service';
import { BooksController } from './books.controller';
import { Book } from './books.entity';
import { BooksRepository } from './books.repository';

@Module({
  imports: [TypeOrmModule.forFeature([Book])],
  controllers: [BooksController],
  providers: [BooksService, BooksRepository],
})
export class BooksModule {}
```

### 3.5 Зачем использовать собственный репозиторий?
Может возникнуть вопрос: зачем создавать собственный репозиторий, 
если можно просто инжектировать TypeOrmRepository напрямую в сервис? 
Ответ заключается в нескольких ключевых преимуществах использования собственного слоя репозиториев:

 - **Инкапсуляция логики доступа к данным**: Собственные репозитории позволяют инкапсулировать логику доступа к данным, что упрощает тестирование и обслуживание кода. Логика работы с базой данных (например, сложные запросы или обработка транзакций) концентрируется в одном месте.

 - **Модульность и переиспользование**: Репозиторий может быть легко переиспользован в других сервисах или модулях, не требуя дублирования кода. Это способствует модульности и улучшает структуру проекта.

 - **Гибкость**: Используя собственные репозитории, вы можете легко изменять или расширять логику работы с данными без необходимости изменения сервисов. Например, вы можете добавить дополнительные методы для работы с сущностью или обрабатывать ошибки специфическим образом.

 - **Тестирование**: Собственные репозитории облегчают тестирование, так как вы можете легко мокировать репозиторий и его методы при написании unit-тестов для сервисов.

## 4. Регистрация модуля Books в AppModule
Чтобы модуль `Books` был доступен в приложении, необходимо зарегистрировать его в `AppModule`.
Если модуль генерировался с помощью nest CLI, то он будет автоматически импортирован в `AppModule`.
Если нет - его нужно добавить вручную в массив `imports`.

Перезапустим приложение и в PGadmin должна появиться новая табличка **`8books`**.


