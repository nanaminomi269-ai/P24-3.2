---
title: "Практическая работаv №8: Объектно-ориентированное программирование в Dart"
discipline: "Разработка мобильных приложений"
tags:
  [
    dart,
    oop,
    flutter,
    constructors,
    inheritance,
    mixins,
    abstract-classes,
    sealed-classes,
  ]
---

# Практическая работа: Объектно-ориентированное программирование в Dart (Flutter Architecture)

> [!NOTE]
> **Цель работы:** Глубоко освоить объектно-ориентированную модель языка Dart. Изучить проектирование классов, генеративные, именованные, фабричные и константные конструкторы, списки инициализации, механизм неявных интерфейсов, одиночное наследование, концепцию миксинов (`mixin`), а также современные модификаторы классов (`sealed`, `base`, `interface`) для построения масштабируемой архитектуры приложений на Flutter.

---

## 📋 Содержание

1. [Архитектура ООП в Dart: всё есть объект](#1-архитектура-ооп-в-dart-всё-есть-объект)
2. [Тема 1. Классы, инкапсуляция, методы и геттеры/сеттеры](#тема-1-классы-инкапсуляция-методы-и-геттерысеттеры)
3. [Тема 2. Все виды конструкторов: Named, Initializer List, Factory, Const](#тема-2-все-виды-конструкторов-named-initializer-list-factory-const)
4. [Тема 3. Наследование (extends), полиморфизм и super](#тема-3-наследование-extends-полиморфизм-и-super)
5. [Тема 4. Абстрактные классы, неявные интерфейсы (implements) и Sealed-классы](#тема-4-абстрактные-классы-неявные-интерфейсы-implements-и-sealed-классы)
6. [Тема 5. Миксины (Mixins): композиция поведения через with и on](#тема-5-миксины-mixins-композиция-поведения-через-with-и-on)
7. [50 Смешанных заданий по ООП (Mobile Architecture Focus)](#50-смешанных-заданий-по-ооп)

---

## 1. Архитектура ООП в Dart: всё есть объект

В Dart любая сущность является объектом — экземпляром класса, наследующегося от корневого типа `Object?` (или `Object` для non-nullable значений). Даже числа, функции и `null` являются полноправными объектами.

```
                            ┌────────────────┐
                            │    Object?     │
                            └───────┬────────┘
                                    │
                                    ▼
                            ┌────────────────┐
                            │     Object     │
                            └───────┬────────┘
            ┌───────────────────────┼───────────────────────┐
            ▼                       ▼                       ▼
    ┌──────────────┐        ┌──────────────┐        ┌──────────────┐
    │ Widget (UI)  │        │  Repository  │        │ num, String  │
    └──────────────┘        └──────────────┘        └──────────────┘
```

### Ключевые особенности ООП в Dart

- **Отсутствие ключевых слов public/private/protected:** Видимость ограничивается библиотекой (файлом). Префикс подчеркивания `_` делает класс, поле или метод приватным для текущего файла.
- **Единое наследование классов:** Класс может наследоваться только от одного суперкласса (`extends`).
- **Множественные неявные интерфейсы:** Любой класс неявно определяет интерфейс, содержащий все его публичные поля и методы (`implements`).
- **Миксины (`with`):** Механизм повторного использования кода в нескольких несвязанных иерархиях без проблем множественного наследования (Diamond Problem).

---

## Тема 1. Классы, инкапсуляция, методы и геттеры/сеттеры

### Структура класса

Класс объединяет переменные состояния (поля) и функции, оперирующие этими данными (методы).

### Пример кода

```dart
class BankAccount {
  final String id;
  String owner;

  // Приватное поле (доступно только внутри данного .dart файла)
  double _balance = 0.0;

  BankAccount({required this.id, required this.owner});

  // Геттер для безопасного чтения баланса
  double get balance => _balance;

  // Сеттер с бизнес-валидацией
  set balance(double value) {
    if (value < 0) {
      throw ArgumentError("Баланс счета не может быть отрицательным");
    }
    _balance = value;
  }

  // Бизнес-метод с возвратом логического статуса
  bool withdraw(double amount) {
    if (amount > 0 && _balance >= amount) {
      _balance -= amount;
      return true;
    }
    return false;
  }

  void deposit(double amount) {
    if (amount > 0) {
      _balance += amount;
    }
  }

  // Переопределение метода строкового представления
  @override
  String toString() => "Счет #$id ($owner): ${_balance.toStringAsFixed(2)} ₽";
}

void main() {
  var account = BankAccount(id: "ACC-40817", owner: "Иван Иванов");
  account.deposit(5000);
  account.withdraw(1200);
  print(account); // Вызывает toString()
}
```

### Задания для закрепления

1. **Инкапсулированный термостат:** Создайте класс `Thermostat` с приватным полем `_temperatureCelsius`. Реализуйте геттер температуры в градусах Фаренгейта и сеттер с ограничением диапазона от -50 до +100 °C.
2. **Счетчик шагов с защитой от скручивания:** Разработайте класс `Pedometer`, где текущие шаги можно только увеличивать через метод `addSteps(int count)`, а уменьшать нельзя.
3. **Модель книги с расчетом времени чтения:** Класс `Book` содержит поля названия и числа страниц. Добавьте вычисляемый геттер `estimatedReadingHours`, считающий время из расчета 1 страница в 2 минуты.
4. **Валидатор пароля в сеттере:** Класс `UserCredentials` с приватным паролем `_passwordHash`. Сеттер принимает сырую строку, проверяет ее длину (\(\ge 8\) символов) и сохраняет псевдо-хэш.
5. **Статический счетчик экземпляров:** Добавьте в класс `DeviceSession` статическое поле `static int activeSessionsCount`, которое увеличивается при создании каждого нового объекта.

class Thermostat {
  double _temperature = 20;
  double get fahrenheit => _temperature * 9 / 5 + 32;
  double get celsius => _temperature;
  set celsius(double value) {
    if (value >= -50 && value <= 100) _temperature = value;
  }
}

class Pedometer {
  int _steps = 0;
  int get steps => _steps;
  void addSteps(int count) {
    if (count > 0) _steps += count;
  }
}

class Book {
  String title;
  int pages;
  Book(this.title, this.pages);
  double get estimatedReadingHours => pages * 2 / 60;
}

class UserCredentials {
  String _passwordHash = '';
  set password(String value) {
    if (value.length >= 8) _passwordHash = 'HASH_${value.hashCode}';
  }
  String get passwordHash => _passwordHash;
}

class DeviceSession {
  static int activeSessionsCount = 0;
  DeviceSession() { activeSessionsCount++; }
  void close() {
    if (activeSessionsCount > 0) activeSessionsCount--;
  }
}

void main() {}

---

## Тема 2. Все виды конструкторов: Named, Initializer List, Factory, Const

Dart предоставляет одну из самых выразительных систем конструкторов среди современных языков программирования.

### Сравнительная таблица конструкторов

| Тип конструктора         | Синтаксис                              | Назначение                                   | Flutter-контекст                          |
| :----------------------- | :------------------------------------- | :------------------------------------------- | :---------------------------------------- |
| **Генеративный**         | `User(this.name, this.age);`           | Быстрое связывание параметров с полями       | Инициализация State-объектов              |
| **Именованный**          | `User.anonymous() : ...;`              | Несколько способов создания объекта          | `EdgeInsets.all()`, `Color.fromARGB()`    |
| **Список инициализации** | `User(...) : _id = uuid(), super(...)` | Вычисление полей до тела конструктора        | Валидация `assert`, инициализация `final` |
| **Константный**          | `const User(this.name);`               | Создание объектов во время компиляции        | Неизменяемые виджеты (`const Text()`)     |
| **Фабричный**            | `factory User.fromJson(...)`           | Возврат кэшированного экземпляра или подтипа | Парсинг REST API, синглтоны               |

### Пример кода

```dart
class UserModel {
  final int id;
  final String name;
  final String role;
  final DateTime createdAt;

  // 1. Основной генеративный конструктор с именованными аргументами
  const UserModel({
    required this.id,
    required this.name,
    this.role = "User",
    required this.createdAt,
  });

  // 2. Именованный конструктор: Гостевой пользователь
  UserModel.guest()
      : id = 0,
        name = "Гость",
        role = "Guest",
        createdAt = DateTime.now(); // Список инициализации

  // 3. Фабричный конструктор: Парсинг JSON ответа сервера
  factory UserModel.fromJson(Map<String, dynamic> json) {
    if (!json.containsKey("id") || !json.containsKey("name")) {
      throw FormatException("Некорректный JSON пользователя");
    }
    return UserModel(
      id: json["id"] as int,
      name: json["name"] as String,
      role: json["role"] as String? ?? "User",
      createdAt: DateTime.parse(json["createdAt"] as String),
    );
  }

  // 4. Фабричный конструктор с кэшированием (Паттерн Singleton/Multiton)
  static final Map<int, UserModel> _cache = {};
  factory UserModel.cached(int id, String name) {
    return _cache.putIfAbsent(
      id,
      () => UserModel(id: id, name: name, createdAt: DateTime.now()),
    );
  }
}
```

### Задания для закрепления

1. **Именованные конструкторы цвета:** Создайте класс `AppColor` с конструкторами `AppColor.fromHex(String hex)` и `AppColor.fromRGBO(int r, int g, int b, double o)`.
2. **Константная точка на плоскости:** Создайте класс `const Point2D(this.x, this.y)`. Создайте два идентичных константных объекта и убедитесь, что `identical(p1, p2) == true`.
3. **Список инициализации с assert:** Создайте класс `Rectangle` с полями ширины и высоты. В списке инициализации добавьте проверку `assert(width > 0 && height > 0)`.
4. **Фабричный синглтон сетевого клиента:** Реализуйте класс `ApiClient` с фабричным конструктором `factory ApiClient()`, гарантированно возвращающим один и тот же приватный экземпляр.
5. **Редирект конструктора (Redirecting constructor):** Напишите класс `Circle(this.radius)` и именованный конструктор `Circle.unit() : this(1.0);`, перенаправляющий вызов на основной.


class AppColor {
  final int r, g, b;
  final double opacity;
  const AppColor(this.r, this.g, this.b, [this.opacity = 1]);

  factory AppColor.fromHex(String hex) {
    final c = int.parse(hex.replaceAll('#', ''), radix: 16);
    return AppColor((c >> 16) & 255, (c >> 8) & 255, c & 255);
  }

  factory AppColor.fromRGBO(int r, int g, int b, double o) =>
      AppColor(r, g, b, o);
}

class Point2D {
  final double x, y;
  const Point2D(this.x, this.y);
}

class Rectangle {
  final double width, height;
  Rectangle(this.width, this.height)
      : assert(width > 0 && height > 0);
  double get area => width * height;
}

class ApiClient {
  static final ApiClient _instance = ApiClient._();
  factory ApiClient() => _instance;
  ApiClient._();
}

class Circle {
  final double radius;
  Circle(this.radius);
  Circle.unit() : this(1);
  double get area => 3.14159 * radius * radius;
}

void main() {}

---

## Тема 3. Наследование (extends), полиморфизм и super

Наследование позволяет создавать иерархии типов, переиспользуя логику базовых классов.

```
                     ┌────────────────────────┐
                     │   UiWidget (Базовый)   │
                     ├────────────────────────┤
                     │ + id: String           │
                     │ + render(): void       │
                     └───────────┬────────────┘
                                 │ extends
         ┌───────────────────────┴───────────────────────┐
         ▼                                               ▼
┌──────────────────┐                           ┌──────────────────┐
│   TextWidget     │                           │   ButtonWidget   │
├──────────────────┤                           ├──────────────────┤
│ + text: String   │                           │ + onClick: Void  │
│ + render(): void │                           │ + render(): void │
└──────────────────┘                           └──────────────────┘
```

### Пример кода

```dart
// Базовый класс компонента интерфейса
abstract class BaseScreen {
  final String routeName;
  final bool requiresAuth;

  BaseScreen({required this.routeName, this.requiresAuth = false});

  void onInit() {
    print("[$routeName] Инициализация базовых ресурсов экрана");
  }

  void render();
}

// Дочерний класс экрана настроек
class SettingsScreen extends BaseScreen {
  final bool isDarkModeEnabled;

  SettingsScreen({required this.isDarkModeEnabled})
      : super(routeName: "/settings", requiresAuth: true); // Вызов конструктора предка

  @override
  void onInit() {
    super.onInit(); // Вызов базовой логики
    print("[SettingsScreen] Загрузка локальных предпочтений темы");
  }

  @override
  void render() {
    print("Отрисовка экрана настроек (Темная тема: $isDarkModeEnabled)");
  }
}
```

### Задания для закрепления

1. **Иерархия банковских карт:** Создайте базовый класс `Card` и наследников `DebitCard` (без овердрафта) и `CreditCard` (с лимитом овердрафта).
2. **Полиморфный список виджетов:** Создайте список `List<BaseScreen>` и в цикле вызовите метод `render()` для каждого элемента независимо от конкретного подтипа.
3. **Использование super-параметров (Dart 2.17+):** Перепишите конструктор наследника с использованием компактного синтаксиса `SubClass({required super.id, super.title});`.
4. **Ковариантный параметр метода (`covariant`):** Покажите пример сужения типа аргумента в переопределенном методе подкласса с помощью ключевого слова `covariant`.
5. **Предотвращение переопределения через `@nonVirtual`:** Создайте метод критической проверки лицензии в базовом классе и запретите его изменение в наследниках.

import 'package:meta/meta.dart';

class Card {
  double balance;
  Card(this.balance);
}

class DebitCard extends Card {
  DebitCard(super.balance);
  bool pay(double amount) {
    if (amount > 0 && amount <= balance) {
      balance -= amount;
      return true;
    }
    return false;
  }
}

class CreditCard extends Card {
  final double limit;
  CreditCard(super.balance, this.limit);
  bool pay(double amount) {
    if (amount > 0 && amount <= balance + limit) {
      balance -= amount;
      return true;
    }
    return false;
  }
}

abstract class BaseScreen {
  void render();
}

class HomeScreen extends BaseScreen {
  @override
  void render() {}
}

class SettingsScreen extends BaseScreen {
  @override
  void render() {}
}

class BaseClass {
  final int id;
  final String title;
  BaseClass({required this.id, this.title = ''});
}

class SubClass extends BaseClass {
  SubClass({required super.id, super.title});
}

class Animal {}
class Cat extends Animal {}

class AnimalHandler {
  void accept(Animal animal) {}
}

class CatHandler extends AnimalHandler {
  @override
  void accept(covariant Cat animal) {}
}

class License {
  @nonVirtual
  bool checkLicense() => true;
}

class AppLicense extends License {}

void main() {}


---

## Тема 4. Абстрактные классы, неявные интерфейсы (implements) и Sealed-классы

### Разница между `extends` и `implements`

| Критерий         | `extends` (Наследование)   | `implements` (Реализация интерфейса)                |
| :--------------- | :------------------------- | :-------------------------------------------------- |
| **Количество**   | Строго один класс          | Множество классов через запятую                     |
| **Тела методов** | Наследуются по умолчанию   | **Не наследуются!** Обязательно переопределить всё  |
| **Поля**         | Наследуются поля и геттеры | Поля воспринимаются как контракты геттеров/сеттеров |

### Sealed-классы (Изолированные иерархии в Dart 3+)

Модификатор `sealed` гарантирует, что все возможные наследники класса объявлены **в том же самом файле**. Это позволяет компилятору проверять исчерпываемость выражений `switch` (Exhaustiveness checking) без необходимости ветки `default`.

### Пример кода (Архитектура состояния экрана / MVI / BLoC)

```dart
// Изолированное состояние экрана каталога товаров
sealed class CatalogState {}

class CatalogLoadingState extends CatalogState {}

class CatalogDataState extends CatalogState {
  final List<String> products;
  CatalogDataState(this.products);
}

class CatalogErrorState extends CatalogState {
  final String errorMessage;
  CatalogErrorState(this.errorMessage);
}

// Рендерер UI с исчерпывающим сопоставлением с шаблонами
String renderStateUi(CatalogState state) {
  // Компилятор гарантирует, что рассмотрены ВСЕ возможные состояния!
  return switch (state) {
    CatalogLoadingState() => "Показ спиннера загрузки...",
    CatalogDataState(:final products) => "Каталог: найдено ${products.length} товаров",
    CatalogErrorState(:final errorMessage) => "Ошибка: $errorMessage (Показать кнопку повтора)",
  };
}
```

### Задания для закрепления

1. **Контракт локального кэша:** Объявите интерфейс `StorageService` с методами `save(String key, String val)`, `read(String key)`, `delete(String key)`. Реализуйте класс `MemoryStorageService` через `implements`.
2. **Множественная реализация интерфейсов:** Создайте класс `SmartTv`, реализующий сразу два несвязанных интерфейса: `Displayable` и `NetworkConnectable`.
3. **Sealed-иерархия авторизации:** Создайте `sealed class AuthState` с наследниками `Unauthenticated`, `Authenticating`, `Authenticated(User user)` и `AuthFailed(String reason)`.
4. **Модификатор `base class`:** Создайте класс `base class Engine`, гарантирующий, что сторонние библиотеки смогут только наследовать его (`extends`), но не смогут реализовать его как чистый интерфейс (`implements`).
5. **Модификатор `interface class`:** Создайте чистый интерфейсный контракт, исключающий прямое наследование реализации вне родной библиотеки.

abstract class StorageService {
  void save(String key, String val);
  String? read(String key);
  void delete(String key);
}

class MemoryStorageService implements StorageService {
  final Map<String, String> data = {};
  @override
  void save(String key, String val) => data[key] = val;
  @override
  String? read(String key) => data[key];
  @override
  void delete(String key) => data.remove(key);
}

abstract class Displayable { void display(); }
abstract class NetworkConnectable { void connect(); }

class SmartTv implements Displayable, NetworkConnectable {
  @override
  void display() {}
  @override
  void connect() {}
}

class User {}

sealed class AuthState {}
class Unauthenticated extends AuthState {}
class Authenticating extends AuthState {}
class Authenticated extends AuthState {
  final User user;
  Authenticated(this.user);
}
class AuthFailed extends AuthState {
  final String reason;
  AuthFailed(this.reason);
}

base class Engine {
  void start() {}
}
class Car extends Engine {}

interface class PaymentService {
  void pay() {}
}
class MyPayment implements PaymentService {
  @override
  void pay() {}
}

void main() {}



---

## Тема 5. Миксины (Mixins): композиция поведения через with и on

Миксин (`mixin`) — это блок кода, содержащий поля и методы, который можно подмешать в класс без использования классического наследования.

### Синтаксис и правила

- Объявляется ключевым словом `mixin Name { ... }`.
- Подмешивается к классу оператором `with Name`.
- Миксины не могут объявлять генеративные конструкторы (их нельзя создать через `new` или `Name()`).
- Ограничение применимости: `mixin Name on TargetClass` означает, что подмешать данный миксин можно **только** в класс, являющийся наследником `TargetClass`.

### Пример кода

```dart
// 1. Простой миксин логирования жизненного цикла
mixin LoggingMixin {
  void logAction(String action) {
    print("[LOG - ${DateTime.now().toIso8601String()}] Действие: $action");
  }
}

// 2. Миксин валидации с ограничением на класс формы
abstract class FormElement {
  String get tag;
}

mixin ValidationMixin on FormElement {
  bool validateNotNull(dynamic value) {
    if (value == null) {
      print("Элемент $tag не прошел проверку: поле обязательно!");
      return false;
    }
    return true;
  }
}

// 3. Класс поля ввода, объединяющий базовый класс и оба миксина
class EmailInput extends FormElement with LoggingMixin, ValidationMixin {
  @override
  final String tag = "EmailField";

  void onUserSubmitted(String? email) {
    logAction("Пользователь нажал Submit");
    if (validateNotNull(email)) {
      logAction("Email валиден: $email");
    }
  }
}
```

### Задания для закрепления

1. **Миксин сериализации в JSON:** Создайте миксин `JsonPrintable`, добавляющий любому классу метод `printAsJson()`.
2. **Ограничение миксина на ChangeNotifier:** Создайте миксин `AutoDisposer on ChangeNotifier`, добавляющий флаг `bool isDisposed` и проверяющий его в методах.
3. **Аналитический трекер кликов:** Напишите миксин `AnalyticsTracker` с методом `trackEvent(String eventName)` и примените его к классу кнопки `CustomButton`.
4. **Порядок линеаризации миксинов:** Создайте класс с двумя миксинами, реализующими одинаковый метод `execute()`. Покажите на практике, метод какого миксина выполнится первым (принцип правого приоритета).
5. **Миксин кэширования вычислений:** Создайте миксин `MemoizationMixin`, сохраняющий результаты тяжелых математических функций в словарь по строковому ключу аргументов.

import 'package:flutter/foundation.dart';

mixin JsonPrintable {
  String printAsJson() => toString();
}
class Product with JsonPrintable {}

mixin AutoDisposer on ChangeNotifier {
  bool isDisposed = false;
  @override
  void dispose() {
    isDisposed = true;
    super.dispose();
  }
}

mixin AnalyticsTracker {
  void trackEvent(String eventName) {}
}
class CustomButton with AnalyticsTracker {}

mixin First {
  String execute() => 'First';
}
mixin Second {
  String execute() => 'Second';
}
class Example with First, Second {}

mixin MemoizationMixin {
  final Map<String, double> _cache = {};
  double calculate(String key, double Function() function) =>
      _cache.putIfAbsent(key, function);
}

void main() {}



---

## 50 Смешанных заданий по ООП

Задания моделируют архитектурные сценарии коммерческих Flutter-приложений (State Management, Networking, UI Layouts, Analytics, Security).

### Блок 1: Модели данных, DTO и конструкторы

1. **Неизменяемая карточка товара (Immutable Product):** Создайте константную модель товара с обязательными полями, расчетом цены со скидкой и валидацией неотрицательной стоимости в списке инициализации.
2. **Парсер профиля пользователя с безопасными дефолтами:** Реализуйте конструктор `User.fromJson(Map<String, dynamic> json)` с обработкой отсутствующих полей и защитой от `TypeError`.
3. **Multiton локалей приложения:** Спроектируйте класс `AppLocale` с фабричным конструктором, возвращающим единственный экземпляр для каждого языка: `AppLocale("ru")`, `AppLocale("en")`.
4. **Модель геоточки с парсингом строки:** Класс `GeoPoint` с именованным конструктором `GeoPoint.fromString("55.751244, 37.618423")`.
5. **Конструктор визитки из сущности БД:** Напишите фабричный конструктор `ContactCard.fromDatabaseRecord(Map<String, Object?> row)`.
6. **Клонирование с изменением полей (copyWith):** Добавьте метод `UserModel copyWith({String? name, String? avatarUrl})` для неизменяемой модели.
7. **Конструктор диапазона дат:** Класс `DateRange(this.start, this.end)` со списком инициализации, выбрасывающим `AssertionError`, если дата окончания раньше даты начала.
8. **DTO банковской транзакции:** Класс `TransactionDto` с именованными конструкторами для пополнения (`TransactionDto.deposit`), списания (`TransactionDto.withdrawal`) и перевода (`TransactionDto.transfer`).
9. **Парсер конфигурации окружения (Environment):** Фабричный конструктор `EnvConfig.fromEnvironment()`, читающий флаги через `const String.fromEnvironment()`.
10. **Кастомный оператор эквивалентности:** Переопределите оператор `==` и метод `hashCode` для класса `CartItem` на основе совпадения полей `productId` и `selectedSize`.

class Product {
  final String name;
  final double price, discount;
  const Product(this.name, this.price, this.discount) : assert(price >= 0);
  double get discountedPrice => price * (1 - discount / 100);
}

class User {
  final String name;
  final int age;
  User(this.name, this.age);
  factory User.fromJson(Map<String, dynamic> json) => User(
    json['name'] is String ? json['name'] as String : '',
    json['age'] is int ? json['age'] as int : 0,
  );
}

class AppLocale {
  static final Map<String, AppLocale> _cache = {};
  final String language;
  factory AppLocale(String language) =>
      _cache.putIfAbsent(language, () => AppLocale._(language));
  AppLocale._(this.language);
}

class GeoPoint {
  final double latitude, longitude;
  GeoPoint(this.latitude, this.longitude);
  GeoPoint.fromString(String value)
      : latitude = double.parse(value.split(',')[0].trim()),
        longitude = double.parse(value.split(',')[1].trim());
}

class ContactCard {
  final String name, phone;
  ContactCard(this.name, this.phone);
  factory ContactCard.fromDatabaseRecord(Map<String, Object?> row) =>
      ContactCard(row['name'] as String, row['phone'] as String);
}

class UserModel {
  final String name, avatarUrl;
  UserModel(this.name, this.avatarUrl);
  UserModel copyWith({String? name, String? avatarUrl}) =>
      UserModel(name ?? this.name, avatarUrl ?? this.avatarUrl);
}

class DateRange {
  final DateTime start, end;
  DateRange(this.start, this.end) : assert(!end.isBefore(start));
}

class TransactionDto {
  final String type;
  final double amount;
  final String? recipient;
  TransactionDto.deposit(this.amount) : type = 'deposit', recipient = null;
  TransactionDto.withdrawal(this.amount) : type = 'withdrawal', recipient = null;
  TransactionDto.transfer(this.amount, this.recipient) : type = 'transfer';
}

class EnvConfig {
  final String apiUrl;
  final bool debug;
  EnvConfig.fromEnvironment()
      : apiUrl = const String.fromEnvironment('API_URL'),
        debug = const bool.fromEnvironment('DEBUG');
}

class CartItem {
  final int productId;
  final String selectedSize;
  CartItem(this.productId, this.selectedSize);
  @override
  bool operator ==(Object other) =>
      other is CartItem &&
      productId == other.productId &&
      selectedSize == other.selectedSize;
  @override
  int get hashCode => Object.hash(productId, selectedSize);
}

void main() {}


### Блок 2: Наследование и иерархия UI-компонентов

11. **Иерархия системных уведомлений:** Базовый класс `InAppNotification` (таймаут, заголовок) и наследники `SuccessNotification`, `WarningNotification`, `CriticalAlertNotification` с разным цветом и иконкой.
12. **Компоненты диалоговых окон:** Базовый класс `BaseDialog` с шаблоном отрисовки шапки и подвала, и наследник `ConfirmActionDialog` с кнопками «Подтвердить» и «Отмена».
13. **Иерархия медиаплееров:** Базовый класс `MediaPlayer` (методы `play()`, `pause()`, `seek()`) и специализированные наследники `AudioPlayer` и `HlsVideoPlayer`.
14. **Модели сенсоров смартфона:** Базовый класс `HardwareSensor` и подклассы `AccelerometerSensor` (оси X, Y, Z) и `BarometerSensor` (давление в гПа).
15. **Система тарифов подписки:** Базовый класс `SubscriptionPlan` и наследники `MonthlyPlan`, `AnnualDiscountPlan` (скидка 20%) и `FamilyPlan` (множитель цены за участников).
16. **Слои графического редактора:** Базовый класс `CanvasLayer` (прозрачность, видимость, z-index) и подклассы `VectorLayer`, `RasterBitmapLayer`, `TextLayer`.
17. **Иерархия платежных шлюзов:** Базовый класс `PaymentGateway` с защищенным методом логирования и наследники `SbpGateway` (Система быстрых платежей) и `BankCardGateway`.
18. **Экраны онбординга:** Базовый экран обучающего тура с кнопкой пропуска и конкретные шаги с уникальными иллюстрациями.
19. **Кастомная кнопка с состояниями:** Базовый класс `WidgetButton` и подкласс `AsyncLoadingButton`, отображающий индикатор загрузки при активном асинхронном действии.
20. **Иерархия логгеров приложения:** Базовый класс `AppLogger` и подклассы `ConsoleLogger` (форматированный вывод в терминал) и `FileCrashLogger` (запись на диск).

class InAppNotification {
  final int timeout;
  final String title;
  InAppNotification(this.title, this.timeout);
}
class SuccessNotification extends InAppNotification {
  SuccessNotification(super.title, super.timeout);
  String get color => 'green';
  String get icon => 'check';
}
class WarningNotification extends InAppNotification {
  WarningNotification(super.title, super.timeout);
  String get color => 'yellow';
  String get icon => 'warning';
}
class CriticalAlertNotification extends InAppNotification {
  CriticalAlertNotification(super.title, super.timeout);
  String get color => 'red';
  String get icon => 'error';
}

abstract class BaseDialog {
  void header();
  void footer();
}
class ConfirmActionDialog extends BaseDialog {
  @override
  void header() {}
  @override
  void footer() {}
  void confirm() {}
  void cancel() {}
}

abstract class MediaPlayer {
  void play();
  void pause();
  void seek(int position);
}
class AudioPlayer extends MediaPlayer {
  @override
  void play() {}
  @override
  void pause() {}
  @override
  void seek(int position) {}
}
class HlsVideoPlayer extends AudioPlayer {}

class HardwareSensor {}
class AccelerometerSensor extends HardwareSensor {
  final double x, y, z;
  AccelerometerSensor(this.x, this.y, this.z);
}
class BarometerSensor extends HardwareSensor {
  final double pressure;
  BarometerSensor(this.pressure);
}

class SubscriptionPlan {
  final double price;
  SubscriptionPlan(this.price);
}
class MonthlyPlan extends SubscriptionPlan {
  MonthlyPlan(super.price);
}
class AnnualDiscountPlan extends SubscriptionPlan {
  AnnualDiscountPlan(double price) : super(price * 12 * 0.8);
}
class FamilyPlan extends SubscriptionPlan {
  FamilyPlan(double price, int members) : super(price * members);
}

class CanvasLayer {
  double opacity;
  bool visible;
  int zIndex;
  CanvasLayer(this.opacity, this.visible, this.zIndex);
}
class VectorLayer extends CanvasLayer {
  VectorLayer(super.opacity, super.visible, super.zIndex);
}
class RasterBitmapLayer extends CanvasLayer {
  RasterBitmapLayer(super.opacity, super.visible, super.zIndex);
}
class TextLayer extends CanvasLayer {
  TextLayer(super.opacity, super.visible, super.zIndex);
}

abstract class PaymentGateway {
  void log(String message) {}
  void pay(double amount);
}
class SbpGateway extends PaymentGateway {
  @override
  void pay(double amount) {}
}
class BankCardGateway extends PaymentGateway {
  @override
  void pay(double amount) {}
}

class OnboardingScreen {
  final String illustration;
  OnboardingScreen(this.illustration);
  void skip() {}
}
class WelcomeScreen extends OnboardingScreen {
  WelcomeScreen() : super('welcome.png');
}
class FeaturesScreen extends OnboardingScreen {
  FeaturesScreen() : super('features.png');
}

class WidgetButton {
  void onPressed() {}
}
class AsyncLoadingButton extends WidgetButton {
  bool isLoading = false;
  Future<void> run(Future<void> Function() action) async {
    isLoading = true;
    try { await action(); } finally { isLoading = false; }
  }
}

abstract class AppLogger {
  void log(String message);
}
class ConsoleLogger extends AppLogger {
  @override
  void log(String message) {}
}
class FileCrashLogger extends AppLogger {
  @override
  void log(String message) {}
}

void main() {}



### Блок 3: Абстрактные контракты, интерфейсы и репозитории

21. **Репозиторий каталога товаров (Repository Pattern):** Объявите интерфейс `ProductRepository` с методами `Future<List<Product>> getAll()`, `Future<Product> getById(int id)` и создайте две реализации: `NetworkProductRepository` и `MockProductRepository`.
22. **Интерфейс биометрической аутентификации:** Спроектируйте интерфейс `BiometricAuthService` с методами `canAuthenticate()` и `authenticate()`. Напишите реализацию-заглушку для тестов.
23. **Шифрование пользовательских данных:** Интерфейс `CryptoService` с методами `encrypt(String plainText)` и `decrypt(String cipherText)`. Реализуйте класс `AesCryptoService`.
24. **Контракт аналитического провайдера:** Интерфейс `AnalyticsProvider` с методом `logEvent(String name, Map<String, Object> params)`. Реализуйте множественную отправку через класс `CompositeAnalyticsProvider`.
25. **Интерфейс геолокатора:** Спроектируйте контракт получения координат устройства с поддержкой стрима обновлений позиции `Stream<GeoPoint> get positionStream`.
26. **Служба локальных уведомлений:** Интерфейс `NotificationSender` с методами отправки мгновенных и отложенных системных пушей.
27. **Контракт синхронизатора баз данных:** Абстрактный класс `DataSyncEngine` с абстрактными методами `fetchRemoteDelta()` и `applyLocalChanges()`.
28. **Провайдер сетевого статуса:** Интерфейс `ConnectivityService` с геттером `bool isOnline` и стримом событий потери интернета.
29. **Хранилище ключ-значение (KeyValueStorage):** Спроектируйте интерфейс с обобщенными методами `T? read<T>(String key)` и `Future<void> write<T>(String key, T value)`.
30. **Интерфейс экспортера отчетов:** Контракт `ReportExporter` с реализациями `PdfReportExporter` и `ExcelReportExporter`.

class Product {
  final int id;
  Product(this.id);
}

abstract class ProductRepository {
  Future<List<Product>> getAll();
  Future<Product> getById(int id);
}
class NetworkProductRepository implements ProductRepository {
  @override
  Future<List<Product>> getAll() async => [];
  @override
  Future<Product> getById(int id) async => Product(id);
}
class MockProductRepository implements ProductRepository {
  final List<Product> products;
  MockProductRepository(this.products);
  @override
  Future<List<Product>> getAll() async => products;
  @override
  Future<Product> getById(int id) async =>
      products.firstWhere((p) => p.id == id);
}

abstract class BiometricAuthService {
  Future<bool> canAuthenticate();
  Future<bool> authenticate();
}
class FakeBiometricAuth implements BiometricAuthService {
  @override
  Future<bool> canAuthenticate() async => true;
  @override
  Future<bool> authenticate() async => true;
}

abstract class CryptoService {
  String encrypt(String plainText);
  String decrypt(String cipherText);
}
class AesCryptoService implements CryptoService {
  @override
  String encrypt(String plainText) => plainText;
  @override
  String decrypt(String cipherText) => cipherText;
}

abstract class AnalyticsProvider {
  void logEvent(String name, Map<String, Object> params);
}
class CompositeAnalyticsProvider implements AnalyticsProvider {
  final List<AnalyticsProvider> providers;
  CompositeAnalyticsProvider(this.providers);
  @override
  void logEvent(String name, Map<String, Object> params) {
    for (final provider in providers) {
      provider.logEvent(name, params);
    }
  }
}

class GeoPoint {
  final double lat, lon;
  GeoPoint(this.lat, this.lon);
}
abstract class GeolocatorService {
  Future<GeoPoint> getPosition();
  Stream<GeoPoint> get positionStream;
}

abstract class NotificationSender {
  void send(String message);
  void schedule(String message, DateTime date);
}
class LocalNotificationSender implements NotificationSender {
  @override
  void send(String message) {}
  @override
  void schedule(String message, DateTime date) {}
}

abstract class DataSyncEngine {
  Future<void> fetchRemoteDelta();
  Future<void> applyLocalChanges();
}

abstract class ConnectivityService {
  bool get isOnline;
  Stream<void> get connectionLost;
}

abstract class KeyValueStorage {
  T? read<T>(String key);
  Future<void> write<T>(String key, T value);
}

abstract class ReportExporter {
  void export(String data);
}
class PdfReportExporter implements ReportExporter {
  @override
  void export(String data) {}
}
class ExcelReportExporter implements ReportExporter {
  @override
  void export(String data) {}
}

void main() {}




### Блок 4: Миксины и повторное использование функционала

31. **Миксин дебаунса событий (Anti-Spam Click):** Миксин `DebounceMixin`, предотвращающий повторный вызов одного и того же действия чаще чем раз в 500 мс.
32. **Миксин восстановления состояния экрана (RestorationMixin):** Миксин, автоматически сохраняющий текстовые поля ввода при сворачивании приложения.
33. **Миксин проверки сетевого соединения:** Миксин `NetworkAwareMixin`, внедряющий в презентер экрана проверку доступности интернета перед выполнением любого запроса.
34. **Миксин форматирования цен и валют:** Миксин `CurrencyFormatterMixin`, предоставляющий готовый метод преобразования чисел в красивый денежный формат.
35. **Миксин таймера автообновления:** Миксин `PeriodicRefreshMixin`, запускающий периодический опрос сервера каждые 30 секунд при активном экране.
36. **Миксин валидации паспортных данных:** Миксин `PassportValidationMixin` с методами строгой проверки серии, номера и кода подразделения.
37. **Миксин анимации затухания (FadeAnimationController):** Подмешиваемый контроллер управления прозрачностью элемента от 0.0 до 1.0.
38. **Миксин переключения темы:** Миксин, предоставляющий экрану быстрый доступ к текущей цветовой палитре (темная/светлая).
39. **Миксин глубоких ссылок (DeepLinkHandlerMixin):** Миксин, парсящий входящие URI-схемы приложения и открывающий целевые экраны.
40. **Миксин аудита действий администратора:** Подмешиваемый модуль, отправляющий критические действия модератора на выделенный сервер безопасности.

import 'dart:async';

mixin DebounceMixin {
  final Map<String, DateTime> _times = {};
  bool canRun(String key) {
    final now = DateTime.now();
    final last = _times[key];
    if (last != null && now.difference(last).inMilliseconds < 500) {
      return false;
    }
    _times[key] = now;
    return true;
  }
}

mixin RestorationMixin {
  final Map<String, String> saved = {};
  void save(String key, String value) => saved[key] = value;
  String? restore(String key) => saved[key];
}

mixin NetworkAwareMixin {
  bool isOnline = true;
  Future<void> request(Future<void> Function() action) async {
    if (isOnline) await action();
  }
}

mixin CurrencyFormatterMixin {
  String formatCurrency(double value) => '${value.toStringAsFixed(2)} ₽';
}

mixin PeriodicRefreshMixin {
  Timer? _timer;
  void startRefresh(void Function() refresh) {
    _timer?.cancel();
    _timer = Timer.periodic(const Duration(seconds: 30), (_) => refresh());
  }
  void stopRefresh() {
    _timer?.cancel();
    _timer = null;
  }
}

mixin PassportValidationMixin {
  bool validSeries(String value) => RegExp(r'^\\d{4}$').hasMatch(value);
  bool validNumber(String value) => RegExp(r'^\\d{6}$').hasMatch(value);
  bool validCode(String value) => RegExp(r'^\\d{3}-\\d{3}$').hasMatch(value);
}

mixin FadeAnimationController {
  double opacity = 1;
  void setOpacity(double value) => opacity = value.clamp(0.0, 1.0);
}

mixin ThemeMixin {
  bool isDark = false;
  String get background => isDark ? 'black' : 'white';
}

mixin DeepLinkHandlerMixin {
  String? handleLink(String link) => Uri.tryParse(link)?.path;
}

mixin AdminAuditMixin {
  void audit(String action, void Function(String) send) => send(action);
}

void main() {}



### Блок 5: Современные модификаторы классов и архитектурные стейты

41. **Sealed-состояние корзины интернет-магазина:** Создайте `sealed class CartState` с состояниями: `CartEmpty`, `CartLoading`, `CartLoaded(List<Item> items, double total)`, `CartCheckoutError(String error)`. Напишите функцию обработки состояний через исчерпывающий `switch`.
42. **Sealed-иерархия сетевого ответа (Result Pattern):** Реализуйте `sealed class Result<T>` с подтипами `Success<T>(final T data)` и `Failure<T>(final Exception exception)`.
43. **Архитектура UI-событий экрана логина:** Создайте `sealed class LoginEvent` с событиями `LoginEmailChanged(String email)`, `LoginPasswordChanged(String pass)`, `LoginSubmitted`.
44. **Модификатор `final class` для безопасности криптоключа:** Объявите класс `final class EncryptionKey`, запретив любое его расширение или реализацию вне текущей библиотеки.
45. **Модификатор `interface class` для публичного API SDK:** Разработайте интерфейсный класс платежного терминала, запретив прямое наследование логики клиентами библиотеки.
46. **Исчерпывающий роутинг экрана через Sealed Route:** Спроектируйте `sealed class AppRoute` для типобезопасной навигации во Flutter.
47. **Паттерн «Команда» (Command Pattern):** Создайте абстрактный класс `Command` с методом `execute()` и `undo()` для реализации функции отмены действий в редакторе текста.
48. **Паттерн «Строитель» (Builder):** Реализуйте `NotificationBuilder` с цепочечными вызовами `.setTitle().setBody().setIcon().build()`.
49. **Паттерн «Наблюдатель» (Observer / EventBus):** Создайте легковесную шину событий с классами событий и методами подписки/отписки.
50. **Комплексный архитектурный модуль заказа такси (Production Case):**
    - Спроектируйте `sealed class TaxiOrderState` (поиск машины, машина назначена, поездка, завершено, отменено).
    - Создайте класс заказа `TaxiOrder` с именованными фабричными конструкторами и списком инициализации.
    - Реализуйте миксины `LocationTrackingMixin` и `FareCalculationMixin`.
    - Напишите контроллер управления поездкой с исчерпывающим сопоставлением с шаблонами.

class Item {}

sealed class CartState {}
class CartEmpty extends CartState {}
class CartLoading extends CartState {}
class CartLoaded extends CartState {
  final List<Item> items;
  final double total;
  CartLoaded(this.items, this.total);
}
class CartCheckoutError extends CartState {
  final String error;
  CartCheckoutError(this.error);
}
String handleCart(CartState state) => switch (state) {
  CartEmpty() => 'empty',
  CartLoading() => 'loading',
  CartLoaded() => 'loaded',
  CartCheckoutError() => 'error',
};

sealed class Result<T> {}
class Success<T> extends Result<T> {
  final T data;
  Success(this.data);
}
class Failure<T> extends Result<T> {
  final Exception exception;
  Failure(this.exception);
}

sealed class LoginEvent {}
class LoginEmailChanged extends LoginEvent {
  final String email;
  LoginEmailChanged(this.email);
}
class LoginPasswordChanged extends LoginEvent {
  final String pass;
  LoginPasswordChanged(this.pass);
}
class LoginSubmitted extends LoginEvent {}

final class EncryptionKey {
  final String value;
  EncryptionKey(this.value);
}

interface class PaymentTerminal {
  void pay(double amount) {}
}
class MyTerminal implements PaymentTerminal {
  @override
  void pay(double amount) {}
}

sealed class AppRoute {}
class HomeRoute extends AppRoute {}
class ProfileRoute extends AppRoute {}
class SettingsRoute extends AppRoute {}
String routeName(AppRoute route) => switch (route) {
  HomeRoute() => '/home',
  ProfileRoute() => '/profile',
  SettingsRoute() => '/settings',
};

abstract class Command {
  void execute();
  void undo();
}
class TextCommand extends Command {
  String text = '';
  @override
  void execute() => text = 'New text';
  @override
  void undo() => text = '';
}

class NotificationBuilder {
  String title = '', body = '', icon = '';
  NotificationBuilder setTitle(String value) { title = value; return this; }
  NotificationBuilder setBody(String value) { body = value; return this; }
  NotificationBuilder setIcon(String value) { icon = value; return this; }
  Map<String, String> build() =>
      {'title': title, 'body': body, 'icon': icon};
}

class EventBus {
  final Map<int, void Function(Object)> _listeners = {};
  int _id = 0;
  int subscribe<T>(void Function(T) callback) {
    final id = ++_id;
    _listeners[id] = (event) {
      if (event is T) callback(event);
    };
    return id;
  }
  void unsubscribe(int id) => _listeners.remove(id);
  void publish(Object event) {
    for (final listener in _listeners.values.toList()) {
      listener(event);
    }
  }
}

class Location {
  final double lat, lon;
  Location(this.lat, this.lon);
}
sealed class TaxiOrderState {}
class Searching extends TaxiOrderState {}
class Assigned extends TaxiOrderState {}
class InTrip extends TaxiOrderState {}
class Completed extends TaxiOrderState {}
class Cancelled extends TaxiOrderState {}

mixin LocationTrackingMixin {
  Location? location;
}
mixin FareCalculationMixin {
  double calculateFare(double distance) => distance * 10;
}
class TaxiOrder with LocationTrackingMixin, FareCalculationMixin {
  final String id;
  final TaxiOrderState state;
  TaxiOrder(this.id, this.state);
  factory TaxiOrder.searching(String id) => TaxiOrder(id, Searching());
  factory TaxiOrder.assigned(String id) => TaxiOrder(id, Assigned());
  factory TaxiOrder.inTrip(String id) => TaxiOrder(id, InTrip());
  factory TaxiOrder.completed(String id) => TaxiOrder(id, Completed());
  factory TaxiOrder.cancelled(String id) => TaxiOrder(id, Cancelled());
}
String orderStatus(TaxiOrderState state) => switch (state) {
  Searching() => 'searching',
  Assigned() => 'assigned',
  InTrip() => 'in trip',
  Completed() => 'completed',
  Cancelled() => 'cancelled',
};

void main() {}


---
