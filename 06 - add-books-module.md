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

## 2.0.0 Описание project template:

### src/core/config/configurationType.ts
```typescript
import process from 'process';

export type ConfigurationType = ReturnType<typeof getSettings>;
const getSettings = () => ({
apiSettings: {
PORT: Number.parseInt(process.env.PORT!),
},
dbSettings: {
DB_NAME: process.env.DB_NAME,
DB_HOST: process.env.DB_HOST,
DB_PORT: Number.parseInt(process.env.DB_PORT!),
DB_TYPE: process.env.DB_TYPE,
USERNAME: process.env.DB_USER,
PASSWORD: process.env.DB_PASSWORD,
},
});

export default getSettings;
```

## 2. Создание сущности `Base` and `Book`
### 2.0.0 Описание сущности `Base`

Начнем с создания сущности `Base`:
Создайте новый файл `base.entity.ts` в директории `core/entity/`.
Опишите сущность следующим образом:

```typescript
import {
  CreateDateColumn,
  PrimaryGeneratedColumn,
  UpdateDateColumn,
} from 'typeorm';

export class BaseEntity {
  @PrimaryGeneratedColumn()
  id: number;

  @CreateDateColumn({ default: new Date() })
  createdAt: Date;

  @UpdateDateColumn({ nullable: true, default: null })
  updatedAt: Date;
}

```

### 2.0.1 Описание сущности `Books`

Начнем с создания сущности `Base`:
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

### 03. Создание модуля, контроллера, сервиса и репозитория для Users
Теперь мы используем Nest CLI для генерации необходимых файлов и классов для работы с сущностью Books.

### 3.1 Генерация модуля
Для создания модуля users выполните команду:

```bash
nest g module modules/users
```

### 3.2 Генерация контроллера
Для создания контроллера Books выполните команду:

```bash
nest g controller modules/users --no-spec
```


### 3.3 Генерация сервиса
Для создания сервиса Books выполните команду:

```bash
nest g service modules/users --no-spec
```

### 0.4 Create src/modules/users/user.entity.ts

```typescript
import { Column, Entity } from 'typeorm';
import { BaseEntity } from '../../core/entity/base.entity';

@Entity('users')
export class User extends BaseEntity {
  @Column()
  name: string;

  @Column({ unique: true })
  email: string;

  @Column()
  age: number;

  @Column()
  passwordHash: string;
}
```
### 0.5 Create src/modules/users/users.repository.ts

```typescript
import { Injectable, NotFoundException } from '@nestjs/common';
import { User } from './user.entity';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';

@Injectable()
export class UsersRepository {
  constructor(
    @InjectRepository(User)
    private usersOrmRepository: Repository<User>,
  ) {}
  async save(user: User): Promise<User> {
    const result = await this.usersOrmRepository.save(user);

    return result;
  }

  findByEmail(email: string) {
    return this.usersOrmRepository.findOneBy({ email });
  }

  async findByIdOrNotFoundFail(id: number) {
    const result = await this.usersOrmRepository.findOneBy({ id });

    if (!result) {
      throw new NotFoundException('user not found');
    }

    return result;
  }

  findAll() {
    return this.usersOrmRepository.find();
  }
}

```
## 0.5 Change src/modules/users/users.module.ts

```typescript
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersRepository } from './users.repository';
import { UsersController } from './users.controller';
import { User } from './user.entity';
import { UsersService } from './users.service';

@Module({
  imports: [TypeOrmModule.forFeature([User])],
  controllers: [UsersController],
  providers: [UsersRepository, UsersService],
})
export class UsersModule {}
```

## 0.6 Change src/modules/users/users.controller.ts

```typescript
import { Body, Controller, Get, Post } from '@nestjs/common';
import { UsersService } from './users.service';
import { RegisterUserDto } from './dto/register-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly userService: UsersService) {}

  @Post('registration')
  registerUser(@Body() dto: RegisterUserDto) {
    return this.userService.registerUser(dto);
  }

  @Get()
  getAlUsers() {
    return this.userService.getAllUsers();
  }
}
```

## 0.7.0 Create src/modules/users/dto/register-user.dto.ts

```typescript
export type RegisterUserDto = {
  name: string;
  email: string;
  age: number;
  password: string;
};
```

## 0.7 Change src/modules/users/users.service.ts

```typescript
import { Injectable } from '@nestjs/common';
import { User } from './user.entity';
import { UsersRepository } from './users.repository';
import { RegisterUserDto } from './dto/register-user.dto';
import bcrypt from 'bcrypt';

@Injectable()
export class UsersService {
  constructor(private usersRepository: UsersRepository) {}
  async registerUser(dto: RegisterUserDto): Promise<number> {
    const passwordHash = await bcrypt.hash(dto.password, 10);
    const user = new User();
    user.age = dto.age;
    user.email = dto.email;
    user.name = dto.name;
    user.passwordHash = passwordHash;

    const createdUser = await this.usersRepository.save(user);

    return createdUser.id;
  }

  getAllUsers() {
    return this.usersRepository.findAll();
  }
}
```

## 0.8 Change src/modules/users/users.controller.ts

```typescript
import { Body, Controller, Get, Post } from '@nestjs/common';
import { UsersService } from './users.service';
import { RegisterUserDto } from './dto/register-user.dto';

@Controller('users')
export class UsersController {
  constructor(private readonly userService: UsersService) {}

  @Post('registration')
  registerUser(@Body() dto: RegisterUserDto) {
    return this.userService.registerUser(dto);
  }

  @Get()
  getAlUsers() {
    return this.userService.getAllUsers();
  }
}
```

## 0.9 Change src/main.ts

```typescript
// import { NestFactory } from '@nestjs/core';
// import { AppModule } from './app.module';
//
// async function bootstrap() {
//   const app = await NestFactory.create(AppModule);
//   await app.listen(3000);
// }
// bootstrap();

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { ConfigService } from '@nestjs/config';
import { ConfigurationType } from './core/config/configurationType';
import { ValidationPipe } from '@nestjs/common';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);

  //подключение глобального валидационного pipe https://docs.nestjs.com/techniques/validation
    app.useGlobalPipes(new ValidationPipe({ transform: true }));

  //разрешены запросы с любых доменов
  app.enableCors({
    origin: '*', // Разрешает запросы с любых доменов
    methods: 'GET,HEAD,PUT,PATCH,POST,DELETE', // Разрешенные методы
    credentials: true, // Включает передачу cookies
  });

  //получение конфиг сервиса https://docs.nestjs.com/techniques/configuration#using-in-the-maints
  const configService = app.get(ConfigService<ConfigurationType>);
  const port = configService.get('apiSettings.PORT', { infer: true })!;

  await app.listen(port);
}
bootstrap();
```

## 0.10 Change src/app.module.ts

```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule, ConfigService } from '@nestjs/config';
import configuration, {
  ConfigurationType,
} from './core/config/configurationType';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UsersModule } from './modules/users/users.module';

@Module({
  imports: [
    //подключение и настройка конфиг модуля из пакета @nestjs/config
    //в файле configuration указаны переменные окружения https://docs.nestjs.com/techniques/configuration
    ConfigModule.forRoot({
      load: [configuration],
    }),

    //подключение и настройка базы данных
    TypeOrmModule.forRootAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService<ConfigurationType>) => {
        const databaseSettings = configService.get('dbSettings', {
          infer: true,
        })!;

        return {
          type: 'postgres',
          host: databaseSettings.DB_HOST,
          port: databaseSettings.DB_PORT,
          username: databaseSettings.USERNAME,
          password: databaseSettings.PASSWORD,
          database: databaseSettings.DB_NAME,
          autoLoadEntities: true,
          synchronize: true,
          logger: 'debug',
        };
      },
    }),
    UsersModule,
  ],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
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
```TypeScript
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


