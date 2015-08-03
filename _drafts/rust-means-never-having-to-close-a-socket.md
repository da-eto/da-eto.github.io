---
layout: post
title: Rust — это значит, что никогда не нужно закрывать сокет
---

> Вольный перевод статьи Yehuda Katz «[Rust Means Never Having to Close a
> Socket](http://blog.skylight.io/rust-means-never-having-to-close-a-socket/)»

One of the coolest features of Rust is how it automatically manages resources
for you, while still guaranteeing both safety (no segfaults) and high
performance.

Одна из самых клёвых фич Rust это то, как он автоматически управляет ресурсами
для вас, гарантируя при этом и безопасность (отсутствие сегфолтов), и высокую
производительность.


Because Rust is a different kind of programming language, it might be difficult
to understand what I mean, so let me be perfectly clear:

Поскольку Rust — язык программирования особого рода, то может быть сложно
понять, что же я имею ввиду, так что позвольте выразиться предельно ясно:


- In Rust, as in garbage collected languages, you never explicitly free memory
- In Rust, unlike in garbage collected languages, you never [^1] explicitly
  close or release resources like files, sockets and locks
- Rust achieves both of these features without runtime costs (garbage
  collection or reference counting), and without sacrificing safety.

- В Rust, как и в языках со сборкой мусора, вы никогда в явном виде не
  освобождаете память
- В Rust, в отличие от языков со сборкой мусора, вы никогда в явном виде не
  закрываете и не освобождаете ресурсы, такие, как файлы, сокеты и локи
- Rust достигает обеих этих фич без траты ресурсов в рантайме (сборка мусора
  или подсчёт сылок) и не жертвуя безопасностью.


If you’ve ever leaked a socket or a file, or used an abstraction that leaked
these resources, you know how big of a deal this is.

Если вы когда-нибудь наблюдали утечки памяти при работе с сокетами или файлами
или использовали какую-нибудь абстракцию для работы с этими ресурсами, которая
протекала, то вы знаете, насколько это важно.


You have probably come to expect protection against “use after free” memory
bugs, while at the same time not thinking twice about similar bugs that can
arise when you explicitly close sockets. I’m here to tell you that there’s a
better way.

Вы, возможно, ожидаете увидеть какую-то защиту против багов с памятью
наподобие “использования после освобождения”, ну и чтобы два раза не ходить,
здесь же может быть про похожие баги, появляющиеся, когда вы в явном виде
закрываете сокеты. Я здесь чтобы сказать вам, что есть лучший путь.


If you’ve worked in a language with a garbage collector, you should pay close
attention to the resource management aspect of this article. If you’ve worked
in a low-level language, like C or C++, you will likely find the safety aspects
most interesting.

Если вы работали с языком со сборкой мусора, то вам стоит внимательно
рассмотреть аспект менеджмента ресурсов, освещённый в этой статье. Если вы
работали с низкоуровневым языком, таким, как C или C++, вы, возможно,
найдёте эти аспекты безопасности более интересными.


> Many parts of Rust’s system were first explored in other programming
> languages. What makes Rust interesting is putting them all together, and
> making strict guarantees across the entire language. In practice, these
> language-wide guarantees dramatically improve the utility of the ideas.

> Многие части системы Rust были сначала изучены в других языках
> программирования. Что делает Rust интересным, так это то, как он собрал
> их все вместе, обеспечивая строгие гарантии во всём языке. На практике,
> эти повсеместные в языке гарантии значительно увеличивают полезность идей.


## The Ownership System

## Система владения


The way that this works is through Rust’s “ownership” system. Whenever you
create a new object, it is “owned” by the scope that created it.

Вот как работает система “владения” в Rust. Каждый раз, когда вы создаёте
новый объект, им “владеет” область видимости, которая его создала.


Let’s take a look at an example of a function that copies an input file into a
tempfile, processes it, and then copies the output into an output file.

Давайте посмотрим пример функции, которая копирует входной файл во временный
файл, обрабатывает его и записывает результат в выходной файл.


```rust
fn process(from: &Path, to: &Path) -> IoResult<()> {
    // creates a new tempdir with the specified suffix
    let tempdir = try!(TempDir::new("skylight"));

    // open the input file
    let mut from_file = try!(File::open(from));

    // create a temporary file inside the tempdir
    let mut tempfile =
        try!(File::create(&tempdir.path().join("tmp1")));

    // copy the input file into the tempfile
    try!(io::util::copy(&mut from_file, &mut tempfile));

    // use an external program to process the tmpfile in place

    // after processing, copy the tempfile into the output file
    let mut out = try!(File::create(to));

    io::util::copy(&mut tempfile, &mut out)
}
```

```rust
fn process(from: &Path, to: &Path) -> IoResult<()> {
    // создаём новую директорию с указанным суффиксом
    let tempdir = try!(TempDir::new("skylight"));

    // открываем входной файл
    let mut from_file = try!(File::open(from));

    // создаём временный файл во временной директории
    let mut tempfile =
        try!(File::create(&tempdir.path().join("tmp1")));

    // копируем входной файл во временный
    try!(io::util::copy(&mut from_file, &mut tempfile));

    // используем некоторую внешнюю программу для обработки временного файла на месте

    // после обработки копируем временный файл в выходной файл
    let mut out = try!(File::create(to));

    io::util::copy(&mut tempfile, &mut out)
}
```


In this example, the scope of the `process` function is the initial owner of
the `TempDir` created on the first line. In this example, the `process`
function never gave up ownership, so when the function finishes, it will
automatically be dropped, which will delete the `Tempfile`.

В этом примере область видимости функции `process` это изначальный владелец
`TempDir`, созданного в первой строке. В этом примере функция `process`
никогда не передаёт владение, так что когда она завершает работу, это будет
автоматически сброшено, что удалит `Tempfile`.


This is an example of _automatic resource management_. The `TempDir` object is
not just a piece of memory – it represents a managed resource. As soon as the
program stops using the resource, its cleanup logic gets invoked.

Это пример __автоматического управления ресурсами__. Объект `TempDir` не просто
кусок памяти — он представляет собой управляемый ресурс. Как только программа
перестаёт использовать ресурс, вызывается его логика очистки.


This is true about virtually __all__ resources in Rust. Just as automatic
memory management relieves us from the need to free memory, automatic resource
management relieves us from the need to close resources.

Это правда фактически обо __всех__ ресурсах в Rust. Также, как автоматическое
управление памятью освобождает нас от необходимости освобождать память,
автоматическое управление ресурсами освобождает нас от необходимости закрывать
ресурсы.


> Aside: This is called “RAII” (Resource Acquisition Is Initialization) in C++,
> which is a contender for the most confusingly named, yet useful concept in
> programming.

> Отклонение: это называется “RAII” (Resource Acquisition Is Initialization,
> Получение ресурса есть инициализация) в C++, что является кандидатом на
> наиболее запутанное имя, но при этом полезном концепте программирования.


It’s interesting to me that the most successful technique for relieving
programmers of manual memory management makes it very difficult to successfully
and efficiently relieve programmers of manual resource management. In
high-level languages, we never `free` memory, but we routinely `close` sockets
and files, and `release` locks.

Для меня интересно то, что наиболее успешная техника для избавления
программистов от ручного управления памятью делает его очень сложным
успешно и эффективно избавлять программистов от ручного управления ресурсами.
В высокоуровневых языках мы никогда не освобождаем память, но мы рутинно
закрываем сокеты и файлы и освобождаем блокировки.


In practice, leaking these resources is shockingly common in languages with
garbage collectors, so I really enjoy the fact that forgetting to close sockets
is as much a non-issue in Rust as forgetting to free memory. And in Rust, you
are protected from “use-after-release” bugs involving resources just as you are
protected from “use-after-free” bugs involving memory.

На практике, утечки в работе с этими ресурсами потрясающе общие в языках со
сборкой мусора, так что я действительно радуюсь факту, что забывание закрыть
сокеты настолько же не проблема в Rust, как и забывание освобождать память.
И в Rust вы защищены от багов “использования после освобождения” с участием
ресурсов, как вы и защищены от багов “использования после освобождения” с
участием памяти.


It sounds like magic, so you probably have a few questions about how it
actually works.

Это звучит как магия, так что вы, возможно, имеете несколько вопросов о том,
как это на самом деле работает.


First of all, this system relies on the fact that there can be only one owner
at a time. How can I be sure that I didn’t make a mistake and reference the
`TempDir` from multiple places? The answer is that the ownership system is not
advisory. In Rust, the scope that creates an object owns it. It can then
transfer ownership to another scope, or retain ownership until it is finished
executing. When a scope finishes executing, Rust destroys any objects it owns.

В первую очередь, эта система полагается на факте того, что может быть только
один владелец в каждый момент времени. Как я могу быть уверен что я не
сделал ошибку и не сослался на `TempDir` из нескольких мест? Ответ в том, что
система владения — это не советник. В Rust область видимости, которая создала
объект, владеет им. После этого она может передать владение другой области
видимости или удерживать владение пока не закончит исполняться. Когда
область видимости заканчивает исполнение, Rust уничтожает любые объекты ею
созданные.


Because only one scope owns an object at a time, you can tell just by looking
at it which objects will be destroyed when it’s done executing.

Поскольку только одна область видимости владеет объектов в один момент времени,
вы можете сказать просто смотря на неё, какие объекты будут уничтожены, когда
она закончит исполнение.


```rust
struct Person {  
    first: String,
    last: String
}

fn hello() {  
    let yehuda = Person {
        first: "Yehuda".to_string(),
        last: "Katz".to_string()
    };

    // `yehuda` is transferred to `name_size`, so it cannot be
    // used anymore in this function, and it will not be destroyed
    // when this function returns. It is up to `name_size`,
    // or possibly a future owner, to destroy it.
    let size = name_size(yehuda);

    let tom = Person {
        first: "Tom".to_string(),
        last: "Dale".to_string()
    };

    // `tom` wasn't transferred, so it will be
    // destroyed when this function returns.
}

fn name_size(person: Person) -> uint {  
    let Person { first, last } = person;
    first.len() + last.len()

    // this function owns Person, so the Person is destroyed when `name_size` returns
}
```

```rust
struct Person {
    first: String,
    last: String
}

fn hello() {
    let yehuda = Person {
        first: "Yehuda".to_string(),
        last: "Katz".to_string()
    };

    // `yehuda` перенесено в `name_size`, так что она больше не может
    // использоваться в этой функции и не будет уничтожена
    // когда функция завершится. Это дело `name_size`,
    // или возможно будущего владельца уничтожить его
    let size = name_size(yehuda);

    let tom = Person {
        first: "Tom".to_string(),
        last: "Dale".to_string()
    };

    // `tom` не был перемещен, так что он будет
    // уничтожен, когда функция завершится
}

fn name_size(person: Person) -> uint {  
    let Person { first, last } = person;
    first.len() + last.len()

    // эта функция владее Person, так что Person будет уничтоже, когда `name_size` завершится
}
```


Just by looking at each of the two functions, you can see that `yehuda` was
transferred to `name_size` and `tom` was not. By looking at `name_size`, you
can see that it still owns its `person` argument when it returns. Just by
looking at the functions, you can determine exactly which objects (if any) will
be destroyed when they finish executing.

Просто смотря на каждую из этих двух функций вы можете увидеть, что
`yehuda` был перемещён в `name_size` и `tom` не был. Смотря на `name_size` вы
можете увидеть, что он всё ещё владеет его аргументом `person` когда он
завершается. Просто смотря на функции вы можете определить какие объекты
(если такие есть) будут уничтожены когда они завершат исполнение.


But how does that explain the tempfile example? If you look at the third line
of code in the `process` function, you can see that `tempdir.path()` is calling
a method on the `Tempdir`. Doesn't this mean that I created a second reference,
and therefore have two owners? Or does it mean that we transferred ownership
into the `path` function, which will destroy the directory immediately upon
return? Clearly both of those answers will not do.

Но как это объясняет пример со временным файлом? Если вы посмотрите на 
третью строчку кода в функции `process`, вы можете увидеть, что `tempdir.path()`
это вызов метода в `Tempdir`. Разве это не значит, что я создал вторую ссылку
и таким образом имею двух владельцев? Или это значит что мы передали владение
в функцию `path`, которая удалит директорию сразу после завершения?
Ясно, что оба этих ответа не подходят.


## Borrowing and Lending

## Заимствование и одалживание


To understand what's happening here, we need to look at the signature of the
`path` method:

Чтобы понять, что здесь происходит, нам нужно посмотреть на сигнатуру
метода `path`:

```rust
fn path(&self) -> &Path
```

The way to read this is:

Способ это прочитать:


> The path method __borrows__ self and returns a __borrowed__ Path.

> Метод path __заимствует__ self и возвращает __заимствованный__ Path.


A function that _borrows_ an object does not take ownership of it, and will not
destroy it when it returns. It can only use the object during the time that the
function is executing–it cannot, for example, spawn a thread and try to use the
object inside the thread. To put it another way, a borrowed object must not
outlive the scope of the function that borrowed it.

Функция, которая __заимствует__ объект не получает право владения им и не будет
уничтожать его, когда завершится. Она только можно использовать объект во время
своего исполнения — она не может, к примеру, заспаунить поток и попытаться
использовать объект внутри этого потока. Если по-другому, заимствованный
объект не должен покидать область видимости функции, которая его заимствовала.


This means that the Rust compiler can look at every function call and know at
compile time whether the code is trying to take over ownership. Once ownership
of an object is transferred, the original owner is banned from accessing it.

Это значит, что компилятор Rust может посмотреть в любой вызов функции и знать
во время компиляции всё, что код пытается сделать с владением. Как только
владение объектом перемещается, изначальному владельцу запрещен доступ к нему.


```rust
struct Person {
    first: String,
    last: String,
    age: uint
}

fn hello() {  
    let person = Person {
        first: "Yehuda".to_string(),
        last: "Katz".to_string(),
        age: 32
    };

    let thirties = is_thirties(person);
    println!("{}, thirties: {}", person, thirties);
}

// This function tries to take ownership of `Person`; it does not
// ask to borrow it by taking &Person
fn is_thirties(person: Person) {  
    person.age >= 30 && person.age < 40
}
```

```rust
struct Person {
    first: String,
    last: String,
    age: uint
}

fn hello() {
    let person = Person {
        first: "Yehuda".to_string(),
        last: "Katz".to_string(),
        age: 32
    };

    let thirties = is_thirties(person);
    println!("{}, thirties: {}", person, thirties);
}

// Это функцию пробует завладеть `Person`; она не
// спрашивает взаймы, используя &Person
fn is_thirties(person: Person) {
    person.age >= 30 && person.age < 40
}
```

If I try to compile this program, I will get the following error (slightly
abridged):

Если я попробую скомпилировать эту программу, то я получу следующее
сообщение об ошибке (слегка сокращённое):


```
move.rs:16:34: 16:40 error: use of moved value: `person`  
move.rs:16     println!("{}, thirties: {}", person, thirties);  
                                            ^~~~~~

move.rs:15:32: 15:38 note: `person` moved here  
move.rs:15     let thirties = is_thirties(person);  
                                          ^~~~~~
```


What this means is that the scope of the `hello` function was the initial owner
of the `Person`, but when it called `is_thirties`, it transferred ownership to
the scope of the `is_thirties` function. As the new owner, when `is_thirties`
returns, it frees the memory occupied by the `Person`.

Это значит то. что область видимости функции `hello` была изначальным владельцем
`Person`, но когда был вызван `is_thirties`, то владение переместилось в
область видимости функции `is_thirties`. Как новый владелец, когда `is_thirties`
завершается, она очищает память, занятую `Person`.


Instead you would want to write this program using borrowing and lending:

Вместо этого вы можете написать эту программу, используя заимствование и одалживание:


```rust
fn hello() {  
    let person = Person {
        first: "Yehuda".to_string(),
        last: "Katz".to_string(),
        age: 32
    };

    // lend the person -- don't transfer ownership
    let thirties = is_thirties(&person);

    // now this scope still owns the person
    println!("{}, thirties: {}", person, thirties);
}

fn is_thirties(person: &Person) {  
    person.age >= 30 && person.age < 40
}
```

```rust
fn hello() {
    let person = Person {
        first: "Yehuda".to_string(),
        last: "Katz".to_string(),
        age: 32
    };

    // одалживаем переменную person, но не передаём владение
    let thirties = is_thirties(&person);

    // сейчас эта область видимости всё ещё владеет person
    println!("{}, thirties: {}", person, thirties);
}

fn is_thirties(person: &Person) {  
    person.age >= 30 && person.age < 40
}
```


__Fundamentally, what this means is that verified ownership is part of the
interface of your functions__. Rust people sometimes refer to this as "the
borrow checker", but the implications are profound.

__Фундаментально, это значит, что верифицированное право владения это часть
интерфейса ваших функций__. Люди Rust иногда называют это “проверятелем
владения”, но следствия глубокие.


In practice, the reason this works so well is that most of the time, functions
that take values are "borrowing" them. They take a value, do some work with the
value, and return. Holding on to the value for longer, for example by using
threads, is both uncommon and an appropriate time to think a little bit about
what's happening.

На практике, причина, по которой это работает так хорошо, это то что чаще всего
функции, получающие значения, “заимствуют” их. Они берут значение, проводят
какую-то работу с ним, и возвращают. Держать значение дольше, например,
используя потоки, и не обычно, и хороший момент, чтобы подумать немного больше
о том, что происходит.


The starting point when writing new functions is to borrow parameters, not try
to take ownership. After a little while of programming with Rust, this imposes
no cognitive cost; it's simply the default. If the compiler complains (and that
happens less and less as you internalize the rules and they become second
nature), it means you're doing something potentially dangerous, and that's when
you need to think about it.

Начальная точка при написании функций это заимствование параметров, не пытаясь
получить владение. После небольшого времени программирования на Rust это не
требует умственных усилий, это просто действие по умолчанию. Если компилятор
жалуется (и это случается реже и реже по мере того, как вы усваиваете правила
и они становятся второй природой), то это значит, то вы делаете что-то
потенциально опасное, и это тогда, когда вам нужно подумать об этом.


## Returning a Borrowed Field from a Borrowed Object

## Возврат заимствованного поля из заимствованного объекта


Earlier on, we looked at this signature:

Ранее мы смотрели на эту сигнатуру:


```rust
fn path(&self) -> &Path  
```


This may have been bothering you. I said before that when a function borrows an
object, it must only use the value while the function is executing, and not
later. Doesn't returning a piece of that object self-evidently violate the
rule?

Это может беспокоить вас. Я сказал ранее, что когда функция заимствует объект,
то она может использовать значение только во время того, когда функция
исполняется, и не позже. Разве возврат части этого объекта самоочевидно
не нарушает правило?


The reason this works is that the caller of `path` obviously has the right to
use the `Tempfile`, since it lent it as an argument. In this case, the Rust
compiler will guarantee that the returned `Path` doesn't outlive the `Tempfile`
that contains it.

Причина, по которой это работает, это то, что вызывающий `path` очевидно
имеет правда использовать `Tempfile`, поскольку он одолжил его как аргумент.
В этом случае компилятор Rust будет гарантировать, что возвращаемый `Path`
не переживёт `Tempfile`, который его содержит.


In practice, this means that you can return borrowed contents upstream, and
Rust will take care of keeping track of the original container that they came
from.

На практике это значит, что вы может возвращать заимствованное содержимое
наверх, и Rust будет заботиться об отслеживании оригинального контейнера, из
которого оно пришло.


To illustrate, let's look at an example:

Для иллюстрации давайте посмотрим на пример:


```rust
fn hello() -> &str {  
    let person = Person {
        first: "Yehuda".to_string(),
        last: "Katz".to_string(),
        age: 32
    };

    first_name(&person)
}

fn first_name(person: &Person) -> &str {  
    // as_slice borrows a slice "view" out of a string
    person.first.as_slice()
}
```

```rust
fn hello() -> &str {  
    let person = Person {
        first: "Yehuda".to_string(),
        last: "Katz".to_string(),
        age: 32
    };

    first_name(&person)
}

fn first_name(person: &Person) -> &str {  
    // as_slice заимствует слайсовый "вид" из строки
    person.first.as_slice()
}
```


If you look at this, you can immediately see a problem. The `hello` function is
trying to return a borrowed `&str`, but the hello function owns the original
`Person` that contains the bytes. As soon as `hello` returns, the `Person` no
longer exists, so the borrowed contents (the slice) point at an invalid
location.

Если вы посмотрите на это, вы немедленно увидите проблему. Функция `hello`
пытается вернуть заимствованный `&str`, но функция `hello` владеет оригинальным
`Person`, который содержит байты. Как только `hello` завершается, `Person`
больше не существует, так что заимствованное содержимое (слайс) указывает
на неверный адрес.


If you actually try to compile this program today, you get:

На самом деле, если вы попробует скомпилировать эту программу, то вы получите:


```
move.rs:8:15: 8:19 error: missing lifetime specifier [E0106]  
move.rs:8 fn hello() -> &str {  
                        ^~~~
```


This slightly confusing error message means that we are trying to return
borrowed bytes, but the caller of this function didn't lend us the `Person` we
borrowed the bytes from. Rust is asking us to tell it which "lifetime" is it
attached to, if not the caller's scope.

Это слегка сбивающее с толку сообщение об ошибке значит, что мы пытаемся вернуть
заимствованные байты, но вызыватель этой функции не одалживал `Person`, из которого
мы заимствуем байты. Rust просит нас сказать, какое “время жизни” прикреплено к нему,
если это не область видимости вызывающего.


> Typically, Rust ties the scope of returned values to the scope of a borrowed
> argument. Here, we have no borrowed arguments, so Rust is asking us to be
> more explicit.

> Обычно, Rust связывает область видимости возвращаемого аргумента с областью
> видимости заимствованного аргумента. Здесь же у нас нет заимствованных
> аргументов, так что Rust просит нас быть более определёнными.


In practice, this means that a function can easily return borrowed contents
contained in a borrowed parameter. Otherwise, you will need to find storage for
the value that your caller has access to, or clone the value so the caller has
its own copy that it owns.

На практике это значит, что функция может легко возвращать заимствованное
содержание, содержащееся в заимствованном параметре. Иначе вам нужно будет найти
хранилище для значения, к которому ваш вызыватель имеет доступ, или клонировать
значение так, чтобы вызывающий имел свою копию того, чем он владеет.


## Ergonomics

## Эргономика


At first glance, all of this ownership machinery feels very involved, and looks
like it would likely have a large impact on the ergonomics of using Rust. And
indeed, it does feel that way initially.




But several factors make things far more usable than they appear.

First, a large amount of real-world code fits within the lend/borrow pattern.
As I've written more and more Rust, I have come to realize that programs
written in Ruby follow similar patterns: functions make some objects and pass
them to child functions to do some work, and the child functions return a new
value.

This is, of course, recursive, so it only becomes obvious when the difference
(between getting parameters to use during the function call, and using them for
longer) is formalized as it is in Rust. Only by making this distinction a
function signatures across the board, and checking for mistakes, can we get the
guarantees that Rust provides.

> In contrast, C++ makes the distinction explicit in some (but not all) cases,
> and doesn't check for mistakes. Languages with a GC typically hide the
> distinction between "transferred" and "lent" arguments.

As I said above, this means that Rust programmers quickly learn to treat
borrowing as the default when writing new functions, which alleviates a lot of
the cognitive load of the system.

Second, after using Rust for a little while, most people discover that the
borrow checker errors are warning them about real, serious, and subtle
mistakes. After a while, the borrow checker naturally pushes you into
programming patterns that are less subject to these kinds of subtle bugs.

Third, I have personally found that a very clear understanding of the ownership
of my objects significantly improves my ability to reason about my programs. It
is both clarifying in general, and makes it much, much harder to introduce
accidental memory leaks that cost huge amounts of time to track down later.

And finally, there are real ergonomic benefits to automatic resource management
that both prevent resource leaks (when I'm being lazy) and extra boilerplate or
indentation (when I'm being careful).

Outside of C++, very few programmers have experienced a programming environment
where automatic resource management was the norm, and it's very, very easy to
turn on the "blub" parts of our brain and assume it isn't that useful. Rust
changes enough of the traditional tradeoffs in this space that I would urge you
to push back on the little voice in your head that's telling you "if I didn't
need it in \, how important can it be?"

## Reference Counting (and GC)

You may be aware that Rust has reference-counted pointers (and plans to have GC
in the future).

How does that fit into all of this?

In my experience, once you get used to the ownership paradigm, you very rarely
want to reach for Rc pointers. For example, the entire Cargo codebase has no
instances of reference counted pointers, and only a single use of an atomically
reference counted pointer (for sharing a lock between threads in the code that
implements parallel builds).

I think this is because ownership is extremely clarifying, and really improves
local reasoning. If you look at any function that uses normal Rust references,
you can tell, locally, which pieces of memory (and resources) will still be
alive once the function returns, and which will not. For example, if you use a
closure, you can tell immediately whether it outlives the current function, and
if it does, what objects that closure owns.

I also think that the concepts of ownership and lending map onto most
real-world programming patterns very nicely. There are some things you can't
do, but in most cases, slight tweaks to code structure will get things
compiling. In exchange, both memory and resource leaks happen very
infrequently, and code clarity is improved.

If this wasn't the case, I suspect even experienced Rust developers would reach
for `Rc` a lot more often.

All of that said, there are still occasional cases where reference counting, or
even garbage collection, is the right approach. Rust's "smart pointer" system
allows `Rc` pointers to transparently operate within the same ownership and
borrowing system, and destructors get run when the reference count is
decremented down to 0 (with the obvious cost in local reasoning and runtime
performance).

## Facilities in Other Languages

Garbage collected languages often have facilities that can help programmers
deal with the problem of manual resource management. In most modern languages,
you don't call close explicitly, but you do need to opt into language
constructs that tie the resource to a lexical scope and release it when you're
done.

Let's look at a few examples, and then I'll talk about the disadvantages of
these approaches.

In Ruby, you can use a block to indicate that you are going to use the resource
within a given scope. Once the block returns, the resource gets cleaned up.

```ruby
File.open("/etc/passwd") do |file|  
  # use the file
end  
```

In Python, a special `with` language keyword creates a protocol for acquiring a
resource, and then releasing it once the block has completed:

```python
with open("/etc/passwd") as file:  
  # use the file
```

Both the Ruby approach, which uses a general-purpose language construct, and
the Python approach, which creates a new protocol, abstracts away the
resource-specific closing mechanism. The user never has to know what closing
looks like, but they must use a special abstraction to ensure that the closing
occurs.

In Go, a `defer` keyword allows a programmer to provide cleanup logic next to
the original creation of the object that manages the resource:

```go
file, error := os.Open("/etc/passwd")  
if err != nil {  
    return;
}
defer file.Close()

// use the file
```

This has advantages over `try/catch/finally`, because it keeps the cleanup logic next 
to the code that acquired the resource, but it does not abstract away the closing logic.

All of these approaches have a number of problems. Again, I urge you to
disengage the “blub” center of your brain, which will probably be telling you
that these problems “don’t end up mattering in practice”.

* It is impossible to add resource releasing logic after the fact to existing
  constructs, because their clients will have used the normal object creation
  API. This makes it more difficult to abstract resources inside of higher level
  objects, because the resource management leaks out into the public API.
* The block-based approaches (Ruby and Python, but not Go) introduce rightward
  drift. Every time you want to work with a resource, you are forced to create
  a new scope. This is fairly annoying in Ruby (which has very good blocks) and
  Python (which uses a language-level construct), and a serious problem in
  JavaScript, where introducing a new scope prevents you from returning and
  breaking from surrounding loops.
* These approaches (including Go’s defer) require you to use the resource
  within a given lexical scope. This forces awkward (or impossible) styles of
  programming when you want to pass the resource around to multiple functions. In
  effect, it is forcing a scope-based ownership system on languages in which that
  model is not idiomatic for object management.
  * Once you start calling other functions with the resource, it is fairly
    trivial to accidentally create “use-after-free” bugs, where a function
    hangs onto the resource (in a closure, say) and then tries to use it after the
    original function closed the resource.

Automatic resource management in Rust alleviates all of these problems:

* Resource-managing objects can define a destructor, which abstracts away the
  releasing logic. Creating an object in the normal way will cause the
destructor to be invoked at the right time. An object can add a destructor
after the fact, without having to modify client code.
  * Note that destructors are not like finalizers in languages with GCs. They
    run predictably, exactly when the object is no longer being used, and do
not introduce runtime costs beyond the cost of running the destructor.
* Because automatic resource management works the same way as automatic memory
  management, no indentation is required. This eliminates annoyance, and also
preserves the semantics of the surrounding code.
* In Rust, you can pass around the resource the same way you would pass around
  any other kind of object. If you transfer ownership to a new scope, the
resource will be closed when the new scope finishes. Otherwise, the borrowing
system will guarantee that “use-after-free” is impossible, just as it is for
memory.

In short, there are real benefits to using the same system for memory and
resource management.

I won’t claim that the Rust ownership system is as mindless to use as garbage
collection. That said, Rust has done a lot of very smart things to recoup a lot
of the costs and even improve on the ergonomics of garbage collected languages
in some cases, as we have seen.

In exchange, you get an extremely fast language that lets you control memory
directly with an absolute guarantee of safety.

Because of that, it enables a whole generation of users of high-level languages
to write low-level code, and that really excites me. The communities have a lot
to learn from each other.

Do you want to learn more about the performance of your Rails app? Sign up for
a [free 30-day trial](https://www.skylight.io/) of Skylight. No credit card
required.

[^1]: When I say “never”, I mean very, very rarely. In garbage collected
  languages, you sometimes end up directly managing memory, and in Rust, you
occasionally end up directly managing resources. In both cases, the dominant
programming model is that the language manages resources for you, and that’s
what’s important.
