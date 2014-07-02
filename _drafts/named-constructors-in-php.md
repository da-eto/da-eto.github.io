---
layout: post
title: "Именованные конструкторы в PHP"
---

{% include tldr.html text="Не ограничивайте себя единственным конструктором в PHP. Используйте статические фабричные методы." %}

{% include tldr.html text="Don't limit yourself by PHP's single constructor. Use static factory methods." %}


PHP позволяет использовать только один конструктор в объявлении класса.
Это часто раздражает.
Вероятно, у нас никогда не будет правильной перегрузки конструктора в PHP, но как минимум мы можем порадоваться
некоторым преимуществам этого.
Давайте возьмём простой объект-значение для времени: Time. Какой способ его создания будет наилучшим?

PHP allows only a single constructor per class. 
That's rather annoying. 
We'll probably never have proper constructor overloading in PHP, but we can at least enjoy some of the benefits. 
Let's take a simple Time value object. Which is the best way of instantiating it?

```php
<?php
$time = new Time("11:45");
$time = new Time(11, 45);
```
Единственный корректный ответ на этот вопрос звучит так: «По ситуации».
Оба варианта корректны с точки зрения домена. Возможна и поддержка обоих вариантов:

The only correct answer is "it depends". Both are correct from the point of view of the domain. Supporting both is an option:
 

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
И что произойдёт, если нам нужно добавить больше способов создания Time?

This is terribly ugly. It makes using the Time class rather confusing. And what happens if we need to add more ways to instantiate Time?

```php
<?php
$minutesSinceMidnight = 705;
$time = new Time($minutesSinceMidnight);
```

Или если мы хотим поддерживать числовые строки, равно как и целые числа?

Or if we want to support numeric strings as well as integers?

```php
<?php
$time = new Time("11", "45");
```

(Замечание: в боевом коде я бы сделал в своём классе Time намного больше защит от дурака.)

(Note: in production code, I would make my Time class a lot more idiot-proof.)

## Рефакторинг к именованным конструкторам

## Refactor to named constructors

Давайте добавить несколько публичных статических методов создания Time.
Это позволит нам избавиться от условных операторов (что всегда хорошо!)

Let's add some public static methods to instantiate Time. This will allow us to get rid of the conditionals (which is always a good thing!).
 
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

Every method now satisfies the Single Responsibility Principle. 
The public interface is clear and understandable interface, and the implementations are straightforward. Are we done?

Ну, кое-что меня беспокоит: конструктор `__construct($hours, $minutes)` выглядит довольно отстойно.
Он раскрывает внутреннюю кухню объекта-значения Time, и мы не можем поменять интерфейс, поскольку он публичный.
Представьте, что по какой-то причине мы хотим, чтобы Time хранил строковое представление, а не отдельные значения.

Well, something is bothering me: `__construct($hours, $minutes)` kinda sucks: it exposes the internals 
of the Time value object, and we can't change the interface because it is public. 
Imagine that for some reason, we want Time to store the string representation and not the individual values.
 
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

Это уродливо: мы проходим через все проблемы с разделением строки только для того, чтобы вновь собрать её в конструкторе.

This is ugly: we go through all the trouble of splitting up the string, only to rebuild it in the constructor.

Разве нам всё ещё нужен конструктор, когда у нас уже есть именованные конструкторы? Конечно, нет!
Они — просто деталь реализации, которую мы хотим инкапсулировать за выразительными интерфейсами. Так что сделаем его приватным:

Do we even need a constructor now that we have named constructors? Of course not! 
They are just an implementation detail, that we want to encapsulate behind meaningful interfaces. So we make it private: 

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
Например, иногда вы захотите, чтобы каждый именованный конструктор присваивал значения свойствам без того, чтобы
прокидывать их через конструктор:

Now that the constructor is no longer public, we can choose to refactor all the internals of Time as much as we want. 
For example, sometimes you'll want every named constructor to assign properties without passing them through a constructor:
 
```php
<?php
final class Time
{
    private $hours, $minutes;

    // We don't remove the empty cnostructor because it still needs to be private
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

## Ubiquitous Language

Наш код начал красиво очищаться и наш класс Time теперь имеет несколько очень удобных способов создания.
Поскольку это происходит с улучшением архитектуры, другие, ранее скрытые недостатки дизайна становятся видимыми.
Посмотрите на интерфейс Time:

Our code begins to clean up nicely, and our Time class now has some very useful ways of being instantiated. 
As it happens with better design, other, previously hidden design flaws, start to become visible. Look at the interface for Time:
 
```php
<?php
$time = Time::fromValues($hours, $minutes);
$time = Time::fromString($time);
$time = Time::fromMinutesSinceMidnight($minutesSinceMidnight);
```

Ничего не замечаете? Мы смешиваем не меньше трёх языков:

Notice anything? We're mixing no less than three languages:

- `fromString` — это деталь реализации в PHP;
- `fromValues` — это некое общее программистское понятие;
- и `fromMinutesSinceMidnight` — это часть языка домена.

- `fromString` is a PHP implementation detail;
- `fromValues` is a sort of generic programming term;
- and `fromMinutesSinceMidnight` is part of the domain language.

Будучи языковым гиком и поклонником Домен-ориентированной архитектуры (Domain-Driven Design),
я не могу дать этому пройти.
Так как Time — это часть нашего домена, то мой предпочитаемый стиль — это найти вдохновение в Общем языке.

Being a language geek and Domain-Driven Design aficionado, I can't let this pass. 
As Time is part of our domain, my preferred style is to find inspiration in the Ubiquitous Language. 

- `fromString` => `fromTime` 
- `fromValues` => `fromHoursAndMinutes`

(Если вы волнуетесь о дополнительных символах, которые теперь нужно печатать, возьмите редактор с контекстным автодополнением).

(If you worry about the extra characters you'd need to type, get an editor with contextual code completion.)

Это смещает фокус на домен, давая вам некоторые замечательные возможности:

This focus on the domain, gives you some great options:

```php
<?php
$customer = new Customer($name); 
// Мы не говорим "создай заказчика" или "инстанциируем заказчика" в обычной жизни.
// Лучше:
$customer = Customer::fromRegistration($name);
$customer = Customer::fromImport($name);
```

```php
<?php
$customer = new Customer($name); 
// We can't "new a customer" or "instantiate a customer" in real life.
// Better:
$customer = Customer::fromRegistration($name);
$customer = Customer::fromImport($name);
```

Ладно, это не всегда лучше.
В случае Time я могу остановиться на toString, так как может быть, что на этом уровне детализации нашего кода
мы хотим предоставить программисту больше, чем просто домен.
Я даже могу предоставить оба варианта.
Но как минимум, спасибо именованным конструктором, у нас **есть** варианты.

Granted, that's not always better. 
In the case of Time, I might stick to toString, because maybe at this level of detail in our code, 
we want to serve the programmer more than the domain. 
I might even provide both options. 
But at least, thanks to named constructors, we now *have* options.  

## Читать далее:

## Read more

- [When to Use Static Methods](/2014/06/when-to-use-static-methods-in-php/)
- [Accessing private properties from other instances](/2011/03/accessing-private-properties-from-other-instances/)
- [Casting Value Objects to String](/2013/02/casting-value-objects/)
- [Final Classes: Open for Extension, Closed for Inheritance](/2014/05/final-classes-in-php/)




