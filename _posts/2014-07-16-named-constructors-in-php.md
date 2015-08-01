---
layout: post
title: "Именованные конструкторы в PHP"
---

> Перевод статьи Mathias Verraes «[Named Constructors in PHP](http://verraes.net/2014/06/named-constructors-in-php/)»

{% include tldr.html text="Не ограничивайте себя единственным конструктором в PHP. Используйте статические фабричные методы." %}

PHP позволяет использовать только один конструктор в объявлении класса.
Это часто раздражает.
Вероятно, у нас никогда не будет правильной перегрузки конструктора в PHP, но как минимум мы можем порадоваться
некоторым преимуществам этого (не запутаемся лишний раз в коде, например).
Давайте представим простой объект-значение для времени: Time. Какой способ его создания будет наилучшим?

```php
<?php
$time = new Time("11:45");
$time = new Time(11, 45);
```

Единственный корректный ответ на этот вопрос звучит так: «По ситуации».
Оба варианта корректны с точки зрения логики домена. Возможна и поддержка обоих вариантов:

```php
<?php
final class Time
{
    private $hours, $minutes;
    public function __construct($timeOrHours, $minutes = null)
    {
        if(is_string($timeOrHours) && is_null($minutes)) {
            list($this->hours, $this->minutes) = explode($timeOrHours, ':', 2);
        } else {
            $this->hours = $timeOrHours;
            $this->minutes = $minutes;
        }
    }
}
```

Такой подход ужасно уродлив. Он делает использование класса Time более запутанным.
И что произойдёт, если нам нужно добавить новые способы создания Time?

```php
<?php
$minutesSinceMidnight = 705;
$time = new Time($minutesSinceMidnight);
```

Или если мы хотим поддерживать на входе числовые строки, равно как и целые числа?

```php
<?php
$time = new Time("11", "45");
```

(Замечание: в боевом коде я бы сделал для своего класса Time более мощную защиту от дурака.)

## Переписываем с использованием именованных конструкторов

Давайте добавим несколько публичных статических методов для создания Time.
Это позволит нам избавиться от условных операторов (что всегда хорошо!)

```php
<?php
final class Time
{
    private $hours, $minutes;

    public function __construct($hours, $minutes)
    {
        $this->hours = (int) $hours;
        $this->minutes = (int) $minutes;
    }

    public static function fromString($time)
    {
        list($hours, $minutes) = explode($time, ':', 2);
        return new Time($hours, $minutes);
    }

    public static function fromMinutesSinceMidnight($minutesSinceMidnight)
    {
        $hours = floor($minutesSinceMidnight / 60);
        $minutes = $minutesSinceMidnight % 60;
        return new Time($hours, $minutes);
    }
}
```

Теперь каждый метод отвечает Принципу единственной ответственности (Single Responsibility Principle).
Публичный интерфейс класса стал чистый и понятный, а реализации методов стали проще и прямее. Всё ли мы сделали?

Ну, кое-что меня беспокоит: конструктор `__construct($hours, $minutes)` выглядит довольно отстойно.
Он раскрывает внутреннюю кухню объекта-значения Time, и мы не можем поменять интерфейс, поскольку он публичный.
Представьте, что по какой-то причине мы хотим, чтобы Time хранил строковое представление, а не отдельные значения.

```php
<?php
final class Time
{
    private $time;

    public function __construct($hours, $minutes)
    {
        $this->time = "$hours:$minutes";
    }

    public static function fromString($time)
    {
        list($hours, $minutes) = explode($time, ':', 2);
        return new Time($hours, $minutes);
    }
    // ...
}
```

Это уродливо: мы режем на куски строковое представление времени только для того, чтобы вновь собрать её в конструкторе.

Разве нам всё ещё нужен конструктор, когда у нас уже есть именованные конструкторы? Конечно, нет!
Они — просто деталь реализации, которую мы хотим скрыть за выразительными интерфейсами. Так что сделаем его приватным:

```php
<?php
final class Time
{
    private $hours, $minutes;

    private function __construct($hours, $minutes)
    {
        $this->hours = (int) $hours;
        $this->minutes = (int) $minutes;
    }

    public static function fromValues($hours, $minutes)
    {
        return new Time($hours, $minutes);
    }
    // ...
}
```

Теперь, когда конструктор больше не является публичным, мы можем отрефакторить все внутренности Time так, как захотим.
Например, иногда может оказаться удобным, чтобы каждый именованный конструктор присваивал значения свойствам без того, чтобы
прокидывать их через конструктор:

```php
<?php
final class Time
{
    private $hours, $minutes;

    // Мы не удаляем конструктор, поскольку всё ещё нужно, чтобы он был приватным
    private function __construct(){}

    public static function fromValues($hours, $minutes)
    {
        $time = new Time;
        $time->hours = $hours;
        $time->minutes = $minutes;
        return $time;
    }
    // ...
}
```

## Общий язык

Наш код становится чище и наш класс Time теперь имеет несколько очень удобных способов создания.
Поскольку это происходит вместе с улучшением архитектуры, другие (ранее скрытые) недостатки дизайна
становятся видимыми.
Посмотрите на интерфейс Time:

```php
<?php
$time = Time::fromValues($hours, $minutes);
$time = Time::fromString($time);
$time = Time::fromMinutesSinceMidnight($minutesSinceMidnight);
```

Ничего не замечаете? Мы смешиваем не меньше трёх языков:

- `fromString` — это деталь реализации в PHP;
- `fromValues` — это некое общее программистское понятие;
- и `fromMinutesSinceMidnight` — это часть языка домена.

Будучи языковым гиком и поклонником Домен-ориентированной архитектуры (Domain-Driven Design),
я не могу это так просто оставить.
Так как Time — это часть нашего домена, то мой предпочтительный стиль — найти вдохновение в Общем языке.

- `fromString` => `fromTime`
- `fromValues` => `fromHoursAndMinutes`

(Если вы беспокоитесь о дополнительных символах, которые теперь нужно печатать, возьмите редактор с контекстным автодополнением).

Такой подход смещает фокус на домен, предоставляя вам некоторые замечательные возможности:

```php
<?php
$customer = new Customer($name); 
// Мы не говорим "создай заказчика" или "инстанциируем заказчика" в обычной жизни.
// Лучше так:
$customer = Customer::fromRegistration($name);
$customer = Customer::fromImport($name);
```

Ладно, так писать не всегда лучше.
В случае Time я могу остановиться на toString, так как может оказаться, что на этом уровне детализации нашего кода
мы хотим предоставить программисту больше, чем просто домен.
Я даже могу предоставить оба варианта.
Но по крайней мере, благодаря именованным конструкторам у нас **есть** эти варианты.

### Почитать на тему:

- [When to Use Static Methods](http://verraes.net/2014/06/when-to-use-static-methods-in-php/)
- [Accessing private properties from other instances](http://verraes.net/2011/03/accessing-private-properties-from-other-instances/)
- [Casting Value Objects to String](http://verraes.net/2013/02/casting-value-objects/)
- [Final Classes: Open for Extension, Closed for Inheritance](http://verraes.net/2014/05/final-classes-in-php/)

