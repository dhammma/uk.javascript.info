
# Базовий синтаксис класу

```quote author="Вікіпедія"
В об’єктно-орієнтованому програмуванні, *клас* -- це спеціальна конструкція, яка використовується для групування пов’язаних змінних та функцій. При цьому, згідно з термінологією ООП, глобальні змінні класу (члени-змінні) називаються полями даних (також властивостями або атрибутами), а члени-функції називають методами класу.
```

На практиці ми часто повинні створювати багато об’єктів одного й того ж виду, наприклад користувачів, або товари чи що завгодно.

Як ми вже знаємо з розділу <info:constructor-new>, `new function` може допомогти з цим.

Але в сучасному JavaScript існує більш просунута конструкція "клас", яка вводить нові чудові функції, які корисні для об’єктно-орієнтованого програмування.

## Синтаксис "class"

Основний синтаксис:
```js
class MyClass {
  // методи класу
  constructor() { ... }
  method1() { ... }
  method2() { ... }
  method3() { ... }
  ...
}
```

Потім використовуйте `new MyClass()`, щоб створити новий об’єкт з усіма перерахованими методами.

Метод `constructor()` викликається автоматично за допомогою `new`, в ньому ми можемо ініціалізувати об’єкт.

Наприклад:

```js run
class User {

  constructor(name) {
    this.name = name;
  }

  sayHi() {
    alert(this.name);
  }

}

// Використання:
let user = new User("Іван");
user.sayHi();
```

Коли `new User("John")` викликається:
1. Створюється новий об’єкт.
2. `constructor` запускається з даними аргументом і присвоює його `this.name`.

...Потім ми можемо викликати методи об’єкту, такі як `user.sayHi()`.


```warn header="Кома між методами класу не потрібна"
Часта помилка для розробників-початківців полягає в тому, щоб поставити кому між методами класу, що призведе до помилки синтаксису.

Позначення тут не слід плутати з літералами об’єктів. У межах класу не треба ставити кому.
```

## Що таке клас?

Отже, що саме -- `class`? Він не є цілком новим ступенем мови програмування, як можна подумати.

Розкриймо будь-яку магію і подивимося, що дійсно таке клас. Це допоможе в розумінні багатьох складних аспектів.

У JavaScript клас є своєрідною функцією.

Погляньте на це:

```js run
class User {
  constructor(name) { this.name = name; }
  sayHi() { alert(this.name); }
}

// доказ: User -- це функція
*!*
alert(typeof User); // function
*/!*
```

Що конструкція `class User {...}` дійсно робить:

1. Створює функцію, що називається `User` та стає результатом оголошення класу. Код функції береться з методу `constructor` (приймається порожнім, якщо ми не написали такий метод).
2. Зберігає методи класу, такі як `sayHi`, `User.prototype`.

Після того, як `new User` створився, коли ми викликаємо його метод, він береться з прототипу, як описано в розділі <info:function-prototype>. Таким чином, об’єкт має доступ до методів класу.

Ми можемо проілюструвати результат оголошення `class User` наступним чином:

![](class-user.svg)

Ось код, щоб проаналізувати це:

```js run
class User {
  constructor(name) { this.name = name; }
  sayHi() { alert(this.name); }
}

// клас -- це функція
alert(typeof User); // function

// ...або, точніше, метод конструктора
alert(User === User.prototype.constructor); // true

// Методи знаходяться в User.prototype, наприклад:
alert(User.prototype.sayHi); // код sayHi методу

// у прототипі існує рівно два методи
alert(Object.getOwnPropertyNames(User.prototype)); // constructor, sayHi
```

## Не просто синтаксичний цукор

Іноді люди кажуть, що `class` -- це "синтаксичний цукор" (синтаксис, який призначений для того, щоб зробити речі легше для читання, але не вводить нічого нового), тому що фактично ми могли б оголосити те саме без використання ключового слова `class`:

```js run
// переписування класу User в чистих функціях

// 1. Створити функцію-конструктор
function User(name) {
  this.name = name;
}
// прототип функції має властивість "constructor" за замовчуванням,
// тому нам не потрібно її створювати

// 2. Додати метод до прототипу
User.prototype.sayHi = function() {
  alert(this.name);
};

// Використання:
let user = new User("Іван");
user.sayHi();
```

Результат цього оголошення дуже схожий. Отже, дійсно існують причини, чому `class` можна вважати синтаксичним цукром для того, щоб визначити конструктор разом із методами прототипу.

Проте, існують важливі відмінності.

1. По-перше, функція, що створена за допомогою `class`, позначена спеціальною внутрішньою властивістю `[[IsClassConstructor]]: true`. Так що це не зовсім те саме, що створити її вручну.

    Рушій перевіряє цю властивість у різних місцях. Наприклад, на відміну від звичайної функції, її треба викликати з `new`:

    ```js run
    class User {
      constructor() {}
    }

    alert(typeof User); // function
    User(); // Помилка: Конструктор класу User неможливо викликати без 'new'
    ```

    Також, представлення конструктора класу у вигляді рядка у більшості рушіїв JavaScript починається з "class..."

    ```js run
    class User {
      constructor() {}
    }

    alert(User); // class User { ... }
    ```
    Є й інші відмінності, ми побачимо їх найближчим часом.

2. Методи класу неперелічувані.
    Оголошення класу встановлює прапор `enumerable` у `false` для всіх методів в `"prototype"`.

    Це добре тому, що коли ми перебираємо об’єкт за допомогою `for..in`, ми зазвичай не хочемо мати справу з методами класу.

3. Клас завжди `use strict`.
    Весь код всередині конструкції класу автоматично знаходиться в суворому режимі.

Крім того, синтаксис `class` приносить багато інших особливостей, які ми дослідимо пізніше.

## Class Expression

Так само, як функції, класи можуть бути визначені всередині іншого виразу, передані, повернуті, присвоєні тощо.

Ось приклад class expression:

```js
let User = class {
  sayHi() {
    alert("Привіт");
  }
};
```

Подібно до Named Function Expression, Class Expression може мати назву.

Якщо Class Expression має назву, то її видно лише у класі:

```js run
// "Named Class Expression"
// (немає такого терміну у специфікації, але це схоже на Named Function Expression)
let User = class *!*MyClass*/!* {
  sayHi() {
    alert(MyClass); // назву MyClass видно тільки всередині класу
  }
};

new User().sayHi(); // працює, показує визначення MyClass

alert(MyClass); // помилка, назву MyClass не видно за межами класу
```

Ми можемо навіть зробити класи динамічними "на вимогу", наприклад:

```js run
function makeClass(phrase) {
  // оголошуємо клас і повертаємо його
  return class {
    sayHi() {
      alert(phrase);
    }
  };
}

// Створюємо новий клас
let User = makeClass("Привіт");

new User().sayHi(); // Привіт
```


## Геттери/сеттери

Подібно до літералів об’єктів, класи можуть включати геттери/сеттери, обчислені назви властивостей тощо.

Ось приклад для `user.name`, що реалізований за допомогою `get/set`:

```js run
class User {

  constructor(name) {
    // викликає сеттер
    this.name = name;
  }

*!*
  get name() {
*/!*
    return this._name;
  }

*!*
  set name(value) {
*/!*
    if (value.length < 4) {
      alert("Ім’я занадто коротке.");
      return;
    }
    this._name = value;
  }

}

let user = new User("Іван");
alert(user.name); // Іван

user = new User(""); // Ім’я занадто коротке.
```

Технічно, таке оголошення класу працює шляхом створення геттерів та сеттерів в `User.prototype`.

## Обчислені назви [...]

Ось приклад з обчисленою назвою методу за допомогою використання дужок `[...]`:

```js run
class User {

*!*
  ['say' + 'Hi']() {
*/!*
    alert("Привіт");
  }

}

new User().sayHi();
```

Такі особливості легко запам’ятати, оскільки вони нагадують літерали об’єктів.

## Поля класу

```warn header="Старим браузерам може знадобитися поліфіл"
Поля класу -- це недавнє доповнення до мови.
```

Раніше наші класи мали лише методи.

"Поля класу" -- це синтаксис, який дозволяє додавати будь-які властивості.

Наприклад, додаймо властивість `name` до `class User`:

```js run
class User {
*!*
  name = "Іван";
*/!*

  sayHi() {
    alert(`Привіт, ${this.name}!`);
  }
}

new User().sayHi(); // Привіт, Іван!
```

Отже, ми просто пишемо "<property name> = <value>" в оголошенні, і це все.

Важливою відмінністю полів класу є те, що вони встановлюються на окремих об’єктах, а не в `User.prototype`:

```js run
class User {
*!*
  name = "Іван";
*/!*
}

let user = new User();
alert(user.name); // Іван
alert(User.prototype.name); // undefined
```

Ми також можемо присвоїти значення, використовуючи складніші вирази та виклики функції:

```js run
class User {
*!*
  name = prompt("Ім’я, будь ласка?", "Іван");
*/!*
}

let user = new User();
alert(user.name); // Іван
```


### Створення методів, що пов’язані з полями класу

Як показано в розділі <info:bind>, функції в JavaScript мають динамічний `this`. Він залежить від контексту виклику.

Отже, якщо метод об’єкта передається і викликається в іншому контексті, `this` більше не буде посиланням на цей об’єкт.

Наприклад, цей код покаже `undefined`:

```js run
class Button {
  constructor(value) {
    this.value = value;
  }

  click() {
    alert(this.value);
  }
}

let button = new Button("привіт");

*!*
setTimeout(button.click, 1000); // undefined
*/!*
```

Ця проблема називається "втратою `this`".

Існує два підходи до розв'язання цієї проблеми, як обговорювалося в розділі <info:bind>:

1. Передати функцію-обгортку, наприклад `setTimeout(() => button.click(), 1000)`.
2. Зв’язати метод з об’єктом за допомогою функції `bind`, наприклад у конструкторі.

Поля класу надають інший, досить елегантний синтаксис:

```js run
class Button {
  constructor(value) {
    this.value = value;
  }
*!*
  click = () => {
    alert(this.value);
  }
*/!*
}

let button = new Button("привіт");

setTimeout(button.click, 1000); // привіт
```

Поле класу `click = () => {...}` створюється на основі конкретного об’єкта, існує окрема функція для кожного об’єкта `Button`, з `this` всередині неї, що посилається на цей об’єкт. Ми можемо передати кнопку куди завгодно, а значення `this` завжди буде коректним.

Це особливо корисно в середовищі браузера, для слухачів подій.

## Підсумки

Основний синтаксис класу виглядає так:

```js
class MyClass {
  prop = value; // властивість

  constructor(...) { // конструктор
    // ...
  }

  method(...) {} // метод

  get something(...) {} // геттер метод
  set something(...) {} // сеттер метод

  [Symbol.iterator]() {} // метод з обчисленим ім’ям (символом в цьому випадку)
  // ...
}
```

`MyClass` технічно є функцією (тою, яку ми надаємо як `constructor`), тоді як методи, геттери та сеттери записуються до `MyClass.prototype`.

У наступних розділах ми дізнаємося більше про класи, включаючи наслідування та інші особливості.
