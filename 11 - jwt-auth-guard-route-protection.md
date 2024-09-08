# Create 11-jwt-auth-guard-route-protection branch
# Защита маршрута для создания книги при помощи JWT auth guard

## 1. JWT стратегия
 - Создадим файл `jwt.strategy.ts` в директории `core/guards`
 - Согласно [документации](https://docs.nestjs.com/recipes/passport#implementing-passport-jwt), создадим класс JwtStrategy
 - Метод `validate` принимает в аргументах объект (payload), который passportjs автоматически извлекает из accessToken'а
   (мы его "засунули" в токен, когда пользователь логинился). Вернем из метода объект `{ userId: payload.userId }` - 
nestjs поместит этот объект в `request.user` (по аналогии с локальной стратегией)

```typescript
import { ExtractJwt, Strategy } from 'passport-jwt';
import { PassportStrategy } from '@nestjs/passport';
import { Injectable } from '@nestjs/common';
import * as process from 'process';
// import { jwtConstants } from './constants';

export interface PayloadJWTInterface {
  userId: string;
  email: string;
}

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor() {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      // secretOrKey: jwtConstants.secret,
      secretOrKey: process.env.JWT_SECRET,
    });
  }

  async validate(payload: PayloadJWTInterface) {
    return { userId: payload.userId, email: payload.email };
  }
}

```

 - Зарегистрируем нашу стратегию в провайдерах `booksModule`, 
чтобы можно было использовать guard на основе этой стратегии в контроллере `booksController`

```typescript
  providers: [BooksService, BooksRepository, JwtStrategy],
```

 - Создадим guard `core/guards/jwt-auth-guard.ts`, как в документации, и "повесим" его на роут
`POST /users` [Implementing Passport JWT](https://docs.nestjs.com/recipes/passport#implementing-passport-jwt)
```typescript
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}
```
```typescript
  @Post()
@UseGuards(JwtAuthGuard)
async createBook(@Body() bookDto: CreateBookDto) {
//...
}
```

**Теперь создавать книги может только авторизованный пользователь**
