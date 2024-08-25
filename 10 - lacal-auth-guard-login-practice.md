# login пользователя

## 1. Регистрация JWT модуля в users

Первым делом давайте зарегистрируем JwtModule и PassportModule модуль в модуле UsersModule. 
Добавим следующий код в массив `imports` модуля `UsersModule`:
```typescript
PassportModule, 
    JwtModule.register({
        secret: 'your_jwt_secret_key', // секретный ключ (должен браться из env)
        signOptions: { expiresIn: '60m' }, // Время жизни токена
    }),
```

**PassportModule**: Модуль `Passport` используется для интеграции с `Passport.js`, который поддерживает различные стратегии аутентификации.

**JwtModule.register**: Мы регистрируем `JwtModule` с секретным ключом и параметрами подписи.
Секретный ключ используется для подписи JWT, а signOptions определяет время жизни токена (expiresIn),
которое указывает, как долго токен будет считаться действительным.

## 2 Метод по проверке введенного пароля при логине

**Создадим метод** `validateUser` в `UserService`, который в аргументах принимает `email: string`, `password: string` и реализует следующие шаги:
1. Найдем в репозитории юзера по его емейлу, используя метод `findByEmail`
2. Если юзер не найден, сгенерируем ошибку: `throw new UnauthorizedException('message');`
3. С помощью библиотеки bcrypt проверим сравним переданный пароль и хешем пароля сущности user: `const isPasswordValid = await bcrypt.compare(password, user.passwordHash);`
4. Если пароль соответствует хешу пароля, вернем из функции `{ userId: user.id }` иначе сгенерируем ошибку (как в пункте 3)

## 3 Метод login

**Создадим метод** `login` в `UserService`, который в аргументах принимает `userId: string`
и должен вернуть объект:
```typescript
{
    accessToken: string
}
```
1. `accessToken` сгенерируем с помощью `JwtService`, метода `sign`. В качестве payload в метод `sign` отдадим
`{ userId }` - этот объект будет зашит в токен и его можно будет извлечь, для определения пользователя, который "стучится"
на защищенный эндпоинт
2. Чтобы воспользоваться `JwtService` в методе класса `UserService`, `JwtService` нужно
инжектировать в `UserService` через конструктор. NestJs автоматически создаст нужные экземпляры
в момент инициализации приложения:

*Под капотом у неста:*
```typescript
const userRepository = new UserRepository();
const jwtService = new JwtService();
const userService = new UserService(userRepository, jwtService);
```
*Примечание:  
инжектирование JwtService в данном модуле возможно, благодаря тому, что
мы зарегистрировали (добавили в массив imports) JwtModule (пункт 1)*

## 4. Локальная стратегия

В соответствии с [документацией nestjs passport local](https://docs.nestjs.com/recipes/passport#implementing-passport-local) создадим локальную стратегию 
и добавим ее в массив `providers` модуля `UsersModule`:
1. Создадим файл `/src/users/local.strategy.ts`
2. Создадим класс `LocalStrategy`, наследованный от `PassportStrategy(Strategy)`,
а лучше скопируем пример из документации :))
3. Заменим `authService` (из документации) на наш `usersService` в конструкторе класса
4. В super() (родительский конструктор) отдадим настройку `super({ usernameField: 'email' });`,
поскольку в качестве username мы используем поле email
5. Метод `validate` возвращает объект, которы nestjs под капотом запишет в request.user
и мы сможем прочитать его в методе контроллера, чтобы затем добавить JWT.
6. Добавим нашу локальную стратегию в массив `providers` модуля `UsersModule`, чтобы nestjs мог в 
нужный момент внедрить нашу стратегию куда надо :))

## 5. Эндпоинт login и localAuthGuard

1. Создадим маршрут login в контроллере users, который должен вернуть пользователю объект
`{ accessToken }`
2. Используем декоратор `UseGuard` в который передадим `AuthGuard('local)`, в соответствии с 
[документацией](https://docs.nestjs.com/recipes/passport#login-route).
А лучше вынести auth guard в отдельный класс и его использовать
```typescript
@Injectable()
export class LocalAuthGuard extends AuthGuard('local') {}
```

## 6. Блок-схема login flow

<img width="800" height="650" src="https://production-it-incubator.s3.eu-central-1.amazonaws.com/file-manager/Image/506fa1b5-01ab-44cf-bc6b-3f098f79f5c1_passport_auth_guard.png"/>

**Теперь используя frontend можем зарегистрировать пользователя и залогинить его.**


