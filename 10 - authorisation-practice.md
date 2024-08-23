# Авторизация пользователя и защита маршрута создания книги

## 1. Регистрация JWT модуля в users

Первым делом давайте зарегистрируем JWT модуль в модуле users:
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

## 2 Добавим метод по проверке введенного пароля при логине

**Создадим метод** `validateUser` в `UserService`, который в аргументах принимает `email: string`, `password: string` и реализует следующие шаги:
1. Найдем в репозитории юзера по его емейлу, используя метод `findByEmail`
2. Если юзер не найден, сгенерируем ошибку: `throw new UnauthorizedException('message');`
3. С помощью библиотеки bcrypt проверим сравним переданный пароль и хешем пароля сущности user: `const isPasswordValid = await bcrypt.compare(password, user.passwordHash);`
4. Если пароль соответствует хешу пароля, вернем из функции `user.id` иначе сгенерируем ошибку (как в пункте 3)

## 3 Добавим метода login

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








