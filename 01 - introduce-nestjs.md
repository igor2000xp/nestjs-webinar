# Введение

[NestJS](https://docs.nestjs.com/) — это фреймворк для построения серверных приложений Node.js, 
который использует TypeScript и вдохновлен архитектурными принципами Angular.  
NestJS использует **классы** и **декораторы** typescript.

# 1. Архитектура NestJS

NestJS следует модульной архитектуре, что делает код легко масштабируемым и поддерживаемым.

### Основные компоненты:

- **[Modules (Модули)](https://docs.nestjs.com/modules)**: Служат для организации приложения на функциональные единицы.
- **Controllers (Контроллеры)**: Обрабатывают входящие запросы и возвращают ответы.
- **Services (Сервисы)**: Содержат бизнес-логику и используются контроллерами.
- **Providers (Поставщики)**: Включают сервисы и любые другие классы, которые используются через Dependency Injection (DI).
- **Dependency Injection (DI)**: Механизм, который позволяет автоматически внедрять зависимости в классы, с помощью которого
реализуется Dependency inversion principle (SOLI**D**).

<img src="https://production-it-incubator.s3.eu-central-1.amazonaws.com/file-manager/Image/d745eff5-7fd3-462a-afbe-6287266652b1_DIP_2.png" alt="Описание изображения" width="1000" height="500">

Пример модуля:
```typescript
import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';

@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule {}
