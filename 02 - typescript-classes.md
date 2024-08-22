# Классы в typescript

---

  Класс в TypeScript - это шаблон для создания объектов с предопределенными свойствами и методами.
  Классы поддерживают **наследование**, **инкапсуляцию** и **полиморфизм**, что делает их основой объектно-ориентированного программирования

В TypeScript классы используются для определения типов данных и создания экземпляров объектов с конкретными характеристиками и поведением

![cat_class](https://production-it-incubator.s3.eu-central-1.amazonaws.com/file-manager/Image/50a37756-5224-487d-8480-bd7b9fec4125_class_cat.jpeg)

## 1. Объявление класса
Синтаксис объявления:
```js filename="class declaration example"
class ClassName {
    constructor(parameters) {
        // инициализация свойств
    }
    method1() {
        // тело метода
    }
}
```

Пример простого класса:
```js filename="class declaration example"
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}
```

### 2. Конструкторы

  **Конструктор** - это специальный метод класса, вызываемый при создании нового экземпляра объекта.
  Он используется для инициализации свойств объекта.


Синтаксис объявления:
```js filename="class constructor example"
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
}
```

## 3. Свойства и методы

  **Свойства класса** - это переменные, определенные внутри класса, используемые для хранения информации или состояния объекта.

**Методы класса** - это функции, определенные внутри класса, предназначенные для выполнения действий с объектами или их свойствами


Пример:
```js filename="class props and methods example"
class Animal {
    name: string;

    constructor(name: string) {
        this.name = name;
    }

    move(distanceInMeters: number = 0) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

#### 3.1 Короткий синтаксис инициализации свойств в конструкторе
В TypeScript инициализация свойств класса может быть упрощена с использованием короткого синтаксиса в конструкторе. 
Это позволяет автоматически создавать и инициализировать свойства, 
передавая параметры в конструктор без явного объявления их в теле класса.

Пример:
```js filename="class props and methods example"
class Animal {
  constructor(private name: string) {}

move(distanceInMeters: number = 0) {
  console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

## 4. Наследование

  Наследование позволяет создавать новые классы на основе существующих, перенимая их свойства и методы.
  Это упрощает повторное использование кода.


Пример наследования простого класса:
```js filename="class extends example"
class Snake extends Animal {

    constructor(name: string) {
      super(name); //вызов конструктора родителя
    }

    move(distanceInMeters = 5) {
        console.log("Slithering...");
        super.move(distanceInMeters); //вызов метода move() родителя
    }
}

const snake = new Snake('cobra');
snake.move(10); //Slithering... cobra moved 10m.
```
![extends_simple](https://production-it-incubator.s3.eu-central-1.amazonaws.com/file-manager/Image/51bf373d-88be-4aa6-85ea-5a20b9c4880b_extends_simple.png)


### 4.1 Наследование, сложный пример (в коде так лучше не делать :)):
![extends_reach](https://production-it-incubator.s3.eu-central-1.amazonaws.com/file-manager/Image/c95519f2-d1e6-45c2-8ca7-c19e70c748ae_09_02.jpg)

## 5. Модификаторы доступа

  TypeScript поддерживает модификаторы доступа для управления доступом к членам класса:
   - **public** - Публичный, доступен всем
   - **private** - Приватный, доступен только внутри класса
   - **protected** - Защищённый, доступен в классе и подклассах
   - **readonly** - Только для чтения, его значение не может быть изменено после инициализации

<img src="https://production-it-incubator.s3.eu-central-1.amazonaws.com/file-manager/Image/0632d030-17c8-4508-a450-2dc3cedf61e4_access_modifier.png" alt="Описание изображения" width="500" height="200">



