# Create 10-lacal-auth-guard-login-practice-theory branch

# login пользователя

## 1. Создание Auth модуля

С помощью CLI сгенерируем auth модуль в котором будет инкапсулирована логика по аутентификации пользователя:

```bash
nest g resource modules/auth --no-spec
```
Ответы на вопросы в терминале:   
1. REST API
2. NO  
*Эта команда сгенерирует папку `modules/auth` и файлы `auth.module`, `auth.controller`, `auth.service`, (и зарегистрирует всё в модуле и сам модуль в корневом `AppModule`, удобно:))*  

## 2. Регистрация (импорт) JWT модуля в AuthModule

### Install Passport and JWT if you haven't already.:
```bash
npm install @nestjs/jwt @nestjs/passport passport passport-jwt passport-local
npm install -D @types/passport-jwt @types/passport-local
```

Зарегистрируем `JwtModule` и `PassportModule` в модуле `AuthModule`. 
Добавим следующий код в массив `imports` модуля `UsersModule`:
```typescript
import { PassportModule } from '@nestjs/passport';
import { JwtModule } from '@nestjs/jwt';
//...

    PassportModule, 
    JwtModule.register({
        secret: 'secret_key', // секретный ключ (должен браться из env)
        signOptions: { expiresIn: '60m' }, // Время жизни токена
    }),
```
### Case with using .env:
```typescript
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { AuthController } from './auth.controller';
import { PassportModule } from '@nestjs/passport';
import { JwtModule } from '@nestjs/jwt';
import { ConfigModule, ConfigService } from '@nestjs/config';
import { ConfigurationType } from '../../core/config/configurationType';
import { UsersModule } from '../users/users.module';

@Module({
  imports: [
    ConfigModule,
    PassportModule,
    UsersModule,
    JwtModule.registerAsync({
      imports: [ConfigModule],
      inject: [ConfigService],
      useFactory: (configService: ConfigService<ConfigurationType>) => {
        const databaseSettings = configService.get('apiSettings', {
          infer: true,
        })!;

        return {
          secret: databaseSettings.JWT_SECRET,
          signOptions: { expiresIn: databaseSettings.JWT_EXPIRE_IN },
        };
      },
    }),
  ],
  controllers: [AuthController],
  providers: [AuthService],
})
export class AuthModule {}


```
**PassportModule**: Модуль `Passport` используется для интеграции с `Passport.js`, который поддерживает различные стратегии аутентификации.

**JwtModule.register**: Мы регистрируем `JwtModule` с секретным ключом и параметрами подписи.
Секретный ключ используется для подписи JWT, а signOptions определяет время жизни токена (expiresIn),
которое указывает, как долго токен будет считаться действительным.

## 3 Метод по проверке введенного пароля при логине

**Создадим метод** `validateUser` в `AuthService`, который в аргументах принимает `email: string`, `password: string` и реализует следующие шаги:
1. Найдем в `usersRepository` пользователя по его email, используя метод `findByEmail`. Да, нам понадобится `userRepository`, поэтому его надо заинжектировать через конструктор.
2. Если юзер не найден, сгенерируем ошибку: `throw new UnauthorizedException('message');`. Nestjs автоматически интерпретирует ее нужный HTTP код (401), а в body ответа положит наше сообщение
3. С помощью библиотеки bcrypt сравним переданный пароль с хешем пароля сущности User: `const isPasswordValid = await bcrypt.compare(password, user.passwordHash);`
4. Если пароль соответствует хешу пароля, вернем из функции `{ userId: user.id }` иначе сгенерируем ошибку (как в пункте 3)
5. Попробуем скомпилировать код и увидим **ошибку!** Что-то наподобие этого: 
`Nest can't resolve dependencies of the AuthService (?, JwtService). Please make sure that the argument UsersRepository at index [0] is available in the AuthModule context.`

```typescript auth.srvice.ts
  async validateUser(name: string, password: string) {
    const user = await this.usersRepository.findByEmail(name);
    if (!user) throw new UnauthorizedException();
    const isPasswordValid = await bcrypt.compare(password, user.passwordHash);
    if (!isPasswordValid) return null;

    return { user: user.name };
}
```


### 3.1 Импорт модуля и массив exports

Итак, если мы хотим переиспользовать провайдер (сервис, репозиторий и т.п.) из одного модуля в другом. Мы должны **экспортировать**
провайдер, который хотим переиспользовать, из своего модуля. То есть добавить в массив `exports`!  
Теперь, если мы **импортируем** модуль из которого что-то **экспортируется** в искомый модуль - мы сможем это что-то переиспользовать и искомом модуле.
Смотрим картинку и не паникуем :):

<img width="900" height="520" src="https://production-it-incubator.s3.eu-central-1.amazonaws.com/file-manager/Image/9475b5e6-a804-4885-9dbc-90de24d8dafd_exports_modules.png">

1. Добавим в массив exports модуля `UsersModule` `UsersRepository` 
(мы хотим чтобы этот репозиторий мог быть использован снаружи, теми кто импортирует этот модуль)
```typescript user.module.ts
...

@Module({
    imports: [TypeOrmModule.forFeature([User])],
    controllers: [UsersController],
    providers: [UsersRepository, UsersService],
    exports: [UsersRepository], // !!!!!!!!!!!!!!!!
})

...
```
2. Импортируем `UsersModule` в `AuthModule`. Запустим и увидим что ошибки больше нет - nest разобрался со всеми зависимостями и создал все сервисы в нужном порядке!
```typescript auth.module.ts
...
@Module({
    imports: [
        ConfigModule,
        PassportModule,
        UsersModule,
...
```
3. Теперь модули нашего приложения выглядят так: 

<img width="1000" height="530" src="https://production-it-incubator.s3.eu-central-1.amazonaws.com/file-manager/Image/104d0935-73c3-4e52-a188-07f585e6b416_webirar-modules-step-2.png"/>

## 4 Метод login

**Создадим метод** `login` в `AuthService`, который в аргументах принимает `userId: string`
и должен вернуть объект:
```typescript
{
    accessToken: string
}
```
1. `accessToken` сгенерируем с помощью `JwtService`, метода `sign`. В качестве payload в метод `sign` отдадим
`{ userId }` - этот объект будет зашит в токен и его можно будет извлечь, для определения пользователя, который "стучится"
на защищенный эндпоинт
2. Чтобы воспользоваться `JwtService` в методе класса `AuthService`, `JwtService` нужно
инжектировать в `AuthService` через конструктор. NestJs автоматически создаст нужные экземпляры
в момент инициализации приложения:

```typescript auth.service.ts
  async login(userID: string) {
    const user = await this.usersRepository.findByIdOrNotFoundFail(
        Number(userID),
    );
    if (!user) throw new UnauthorizedException();

    return { accessToken: this.jwt.sign({ userId: user.id }) };
}
```

*Под капотом у неста:*
```typescript
const userRepository = new UserRepository();
const jwtService = new JwtService();
const authService = new AuthService(userRepository, jwtService);
```
*Примечание:  
инжектирование JwtService в данном модуле возможно, благодаря тому, что
мы зарегистрировали (добавили в массив imports) JwtModule (пункт 1), 
то есть этот сервис **экспортируется** из `JWTModule` (где-то внутри библиотеки `@nestjs/jwt`)* 

## 5. Локальная стратегия

В соответствии с [документацией nestjs passport local](https://docs.nestjs.com/recipes/passport#implementing-passport-local) 
создадим локальную стратегию 
и добавим ее в массив `providers` модуля `AuthModule`:
1. Создадим файл `/src/modules/auth/local.strategy.ts`
2. Создадим класс `LocalStrategy`, наследованный от `PassportStrategy(Strategy)`,
а лучше скопируем пример из документации :))
3. В super() (родительский конструктор) отдадим настройку `super({ usernameField: 'email', passwordField: 'password' });`,
поскольку в качестве `username` мы используем поле `email`, `password` - as well `password`.
4. Метод `validate` возвращает объект, которы nestjs под капотом запишет в **request.user**
и мы сможем прочитать его в методе контроллера, чтобы затем добавить в JWT.

```typescript local.strategy.ts
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { PassportStrategy } from '@nestjs/passport';
import { Strategy } from 'passport-local';
import { AuthService } from './auth.service';

@Injectable()
export class LocalStrategy extends PassportStrategy(Strategy) {
  constructor(private authService: AuthService) {
    super({ usernameField: 'email', passwordField: 'password' }); // By default, Passport expects fields named 'username' and 'password'
  }

  async validate(username: string, password: string): Promise<any> {
    const user = await this.authService.validateUser(username, password);
    if (!user) {
      throw new UnauthorizedException();
    }
    return user; // The user object will be attached to req.user
  }
}

```

5. Добавим нашу локальную стратегию в массив `providers` модуля `AuthModule`, чтобы nestjs мог в 
нужный момент внедрить нашу стратегию куда надо :))

```typescript local.strategy.ts
...
providers: [AuthService, LocalStrategy],
...
```

## 6. Эндпоинт login и localAuthGuard

1. Создадим маршрут login в контроллере auth, который должен вернуть пользователю объект
`{ accessToken }`
2. Используем декоратор `UseGuard` в который передадим `AuthGuard('local)`, в соответствии с 
[документацией](https://docs.nestjs.com/recipes/passport#login-route).
А лучше вынести auth guard (src/modules/auth/local-auth.guard.ts) в отдельный класс и его использовать

```typescript auth.control.ts
...
@UseGuards(AuthGuard('local'))

// OR if you have a user's local Guard as above

import { Controller, Post, Body, UseGuards, Request } from '@nestjs/common';
import { AuthService } from './auth.service';

import {
    ApiBadRequestResponse,
    ApiForbiddenResponse,
    ApiOperation,
    ApiResponse,
    ApiTags,
    ApiUnauthorizedResponse,
} from '@nestjs/swagger';
import { LogoutMessage } from './dto/message-logout.dto';
import { LocalAuthGuard } from './guards/local-auth.guard';
import { LoginUserDto } from '../users/dto/login-user.dto';
import { JwtService } from '@nestjs/jwt';

@ApiTags('Authorization')
@Controller('auth')
export class AuthController {
    constructor(
        private readonly authService: AuthService,
        private readonly jwt: JwtService,
    ) {}

    // @UsePipes(new ValidationPipe())
    @UseGuards(LocalAuthGuard)
    // @UseGuards(AuthGuard('local'))
    @ApiOperation({ summary: 'To login you need the pass and email' })
    @ApiResponse({
        status: 201,
        type: LogoutMessage,
        description: 'Everything is OK',
    })
    @ApiBadRequestResponse({ description: 'Validation error' })
    @ApiUnauthorizedResponse({ description: 'Login or password incorrect' })
    @ApiForbiddenResponse({ description: 'User blocked' })
    // @HttpCode(HttpStatus.OK)
    @Post('login')
    async login(@Request() req: { user: string }, @Body() userDto: LoginUserDto) {
        return {
            accessToken: this.jwt.sign({ userId: req.user, user: userDto.email }),
        };
    }
```

### AuthGuard
```typescript local-auth.guard.ts
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class LocalAuthGuard extends AuthGuard('local') {}

```

### And finally authService

```typescript
import { Injectable, UnauthorizedException } from '@nestjs/common';
import { UsersRepository } from '../users/users.repository';
import * as bcrypt from 'bcrypt';
import { JwtService } from '@nestjs/jwt';

@Injectable()
export class AuthService {
  constructor(
    private readonly usersRepository: UsersRepository,
    private readonly jwt: JwtService,
  ) {}

  // async login(userID: string) {
  //   const user = await this.usersRepository.findByIdOrNotFoundFail(
  //     Number(userID),
  //   );
  //   if (!user) throw new UnauthorizedException();
  //
  //   return { accessToken: this.jwt.sign({ userId: user.id }) };
  // }
  async login(name: string, password: string) {
        const user = await this.usersRepository.findByEmail(name);
        if (!user) throw new UnauthorizedException();
        const isValid = await this.validateUser(name, password);
        if (!isValid) return null;

        return { accessToken: this.jwt.sign({ userId: user.id }) };
  }

  async validateUser(name: string, password: string) {
    const user = await this.usersRepository.findByEmail(name);
    if (!user) throw new UnauthorizedException();
    const isPasswordValid = await bcrypt.compare(password, user.passwordHash);
    if (!isPasswordValid) return null;

    return user.id;
  }
}
```

## 7. Блок-схема login flow

<img width="1000" height="820" src="https://production-it-incubator.s3.eu-central-1.amazonaws.com/file-manager/Image/2705a816-3dd1-4cba-9a7c-8f386f87c3d0_login-flow-with-local-strategy.png"/>

**Теперь используя frontend можем зарегистрировать пользователя и залогинить его.**


