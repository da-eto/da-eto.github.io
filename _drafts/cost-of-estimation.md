---
layout: post
title: Цена оценки
---

> Перевод статьи Mathias Verraes «[The Cost of Estimation](http://verraes.net/2014/06/cost-of-estimation/)»

**tl;dr** Asking for estimates can cost you more than you think.
When you do estimate, take into account time, complexity, and risk.

**tl;dr** Спрашивать оценки времени может стоить вам больше, чем вы думаете.
Если делаете оценки, берите в расчёт время, сложность и риски.

## Плюс-минус километр

## Ballpark Figures

![Риски](/public/posts/cost-of-estimation/risk.jpg)

<img style="float:left;margin-right: 10px" src="http://verraes.net/img/posts/2014-06-17-cost-of-estimation/78704029_3e0e8cf027_z.jpg" alt="Risk">

Когда кто-нибудь просит вас оценить, сколько времени займёт фича или проект,
вы будете оценивать **минимальное возможное время**, в течение которого вы можете завершить его.
Подумайте об этом. Понаблюдайте за своими мыслями пока оцениваете, и вы будете знать, что это правда.
Даже если запрос был сделан с наилучшими намерениями, неявно предполагается,
что это не «как много времени это займёт, чтобы сделать правильно», а «как быстро вы можете это сделать».

When somebody asks you to estimate how long a feature or a project will take,
you will estimate the **shortest possible time** in which you can complete it.
Think about this. Observe your own thoughts while estimating, and you’ll know it’s true.
Even if the request was done with the best of intentions, the implicit assumption
is not “how long will it take to build it right” but “how fast can you build it”.

Если вы сгорели от этого, то вы перемещаетесь в фазу 2: раздувание оценки.
Вы делаете это втайне, и вы чувствуете себя, словно обманщик, делая это.
По какой-то причине, это **табу считать время, чтобы сделать правильно, в отличие от того, чтобы сделать быстро**.

If you have been burned by this, you move on to phase 2: padding your estimation.
You do it secretly, and you feel like a cheat for doing it.
For some reason, it’s **a taboo to estimate the time to do it right, as opposed to doing it fast**.

В худшем конце спектра, это «поиск стрелочника».
Кто-нибудь врывается в офис и требует «приблизительные оценки».
Да, он всё всё ожидает, что вы закончите задачу в это время,
без каких-либо колебаний скидывая ещё больше задач на ваши колени.
В конце концов, вы становитесь виновны в «расхождении с планом» или «совершении неверных оценок».
Недоверие в команде и отлынивание от работы растут.
Множество параллельных вселенных возникают: реальность,
планы менеджеров и негласный договор, что план в любом случае не верен.

At the worst end of the spectrum, there’s **blame culture**.
Somebody storms into the office, and demands a “guesstimate” or a “ballpark figure”.
Yet they still expect you to complete the task in that time,
without even hesitating to dump more tasks in your lap.
In the end, you are blamed for “not sticking to the plan” or “making wrong estimations”.
Distrust between the team and the rest of business grows.
A set of parallel universes spring into existence: reality;
the plans of the managers; and the public secret that the plan is wrong anyway.

## Тройные оценки

## Ternary Estimations

Некоторые люди пробуют похакать это, используя **тройные оценки**: комбинацию ожидаемой продолжительности,
оптимистической продолжительности и пессимистической продолжительности. Я никогда не видел, чтобы это работало на практике.
Управление программными проектами достаточно сложно само по себе, а использование всё время трёх чисел вместе не делает его проще.
И числа обычно считаются (опять неявно) как `ожидаемое время ± 20%`,
что не то же самое, что правильные оптимистичные и пессимистичные оценки.

Some people try to hack this, using **ternary estimations**: the combination of the expected duration,
the optimistic duration, and the pessimistic duration. I’ve never seen this work in practice.
Managing software projects is hard enough as it is, having to deal with three numbers all the time doesn’t make it easier.
And the numbers are usually calculated (again implicitly) as `expected time +- 20%`,
which is not the same as a proper optimistic or pessimistic estimate.

## Творческое решение проблемы

## Creative Problem Solving

Всё это обычно. Это интуитивно.
Многие люди спрашивают оценки без всякой мысли.
Оценивание чувствуется как эффективный способ контроля проектов.
Мы измеряем время и деньги, поскольку время и деньги проще измерять.
Мы даже дурачим себя верой в то, что время — это деньги и они могут быть легко конвертированы друг в друга.

All of this is normal. It’s intuitive.
Many people ask for estimation without ever giving it any thought.
Estimation feels like an effective way to control projects.
We measure time and money, because time and money are the easiest to measure.
We even fooled ourselves into believing that time _is_ money, and that they can easily be converted back and forth.

Это было сказано много раз ранее, но нужно повторить: разработка ПО — это творческая профессия.
Если вы проектируете ПО правильно, то вы решаете каждую задачу лишь однажды.
Это значит, что **каждая задача — это новая задача**.
Вы никогда не решали её раньше, так что вы в темноте относительно того, какое решение лучше и как много времени оно займёт.

It’s been said many times before, but it bears repeating: software development is a creative profession.
If you do software design right, you solve every problem only once.
That means that **every problem is a new problem**.
You’ve never solved it before, so you are in the dark as to what the best solution is and how long that will take.

И это беспорядок: много возможных решений, все с различными сильными и слабыми сторонами.
Некоторые ясно лучше, но ясно, что более затратные, но чаще всего это различие не очевидно.
И не каждой задаче нужно *лучшее* решение,
нужно решение, которое предпочтительно дешевле, чем общие затраты.
Представьте себе многомерную матрицу с множеством различных решений, разбросанных по ней.

And it’s messy: there are many possible solutions, all with different strengths and weaknesses.
Some are clearly better, but clearly more expensive, but most of the time, that distinction is not obvious.
And not every problem needs the *best* solution;
it needs a solution that is preferably cheaper than the total value.
Imagine a multidimensional matrix, with many possible solutions scattered across.

## Цена оценки

## The Cost of Estimation

Время и креативность — враги. Вы не можете решать задачи быстро.
Вы не можете [думать быстрее](http://amzn.to/1iDPNQY).
Есть чудесные химические процессы, происходящие в вашем мозгу, когда вы думаете,
и вы не можете ускорить их, просто усилив напор.

Time and creativity are enemies. You can’t solve problems faster.
You can’t [think faster](http://amzn.to/1iDPNQY).
There’s some wonderful chemistry going on in your brain when you think,
and you can’t just speed that up by pushing harder.

Это реальная цена оценивания:
когда кто-нибудь смотрит через ваше плечо, спрашивая, как много времени это займёт,
когда три часа утра и вы патчите сервер,
короче, когда вы под давлением, тогда вы **прекращаете решать задачи**.
Ваш мозг бешено ищет быстрейшую вещь, которую вы можете сделать, чтобы проблема исчезла.
Вы не принимаете во внимание полную матрицу решений и последствий, с их стоимостью и бонусами.
Это похоже на страстное желание нездоровой пищи: вы голодны прямо сейчас
и вы обещаете себе, что сядете на диету после для компенсации.
Но, конечно, вы не [управляете техническим долгом, который создаёте](http://verraes.net/2013/07/managed-technical-debt/),
поскольку у вас нет времени даже на это.

This is the real cost of estimation:
When somebody watches over your shoulder, asking how much longer it will take;
when it’s three in the morning and you’re patching a server;
in short when you are under pressure, then you **stop solving problems**.
Your brain is frantically looking for the quickest thing you can do to make the problem go away.
You’re not considering the whole matrix of solutions and consequences, of costs and benefits.
It’s like craving for junk food: you’re hungry right now,
and you promise yourself you’ll diet later to compensate.
But of course, you’re not [managing the technical debt you’re creating](http://verraes.net/2013/07/managed-technical-debt/),
because you don’t have time for that either.

Другими словами, **давление временем — это источник многого нашего legacy-кода**.
Да, мальчик, у нас есть legacy-код в индустрии!
Как следует из моего введения, оценки часто имеют причиной или следуют из давления временем.
Интересно, что будучи сделанными в неуправляемом техническом долге, они обладают привнесённой сложностью:
модели, которые не соответствуют домену, непонятные пользовательские интерфейсы, недостаточное тестирование…
Это, в свою очередь, стоит: сложные системы тяжелее изучить, тяжелее с ними работать, тяжелее менять.
**Когда бизнес зависит от ПО, которое тяжело менять, тогда сам бизнес становится тяжело менять**.
Пока вы поняли это, следующий стартап вырвался вперёд,
быстрее, умнее, более гибкий, и он стёр вашу организацию прямо с лица земли.

In other words, **time pressure is the source of much of our legacy**.
And boy, do we have legacy in industry!
Following from my introduction, estimates often cause or imply time pressure.
The interest that builds up on unmanaged technical debt, is accidental complexity:
models that do not match the domain, incomprehensible user interfaces, lack of tests, …
That, in turn, is costly: a complex system is harder to learn, harder to work with, harder to change.
**When the business depends on software that is hard to change, the business itself becomes hard to change.**
Before you know it, the next startup comes along,
faster, smarter, more agile, and wipes your organization straight from the face of the Earth.

## Почему оценки всегда неправильные

## Why Estimates Are Always Wrong

![Риски](/public/posts/cost-of-estimation/hine-icarus.jpg)

<img style="float:right;margin-left: 10px" src="/img/posts/2014-06-17-cost-of-estimation/1200521270Hine_Icarus_575.jpg" alt="Risk">

Если мы всегда оцениваем кратчайшее возможное время на выполнение задачи,
то мы всякий раз будем ошибаться, мы будем факапить и искать лучшее место:
истинное завершение может быть точно после любого возможного происшествия на дороге.
Немного счастливых случайностей, из-за которых задача завершается быстрее.

If we always estimate the shortest possible time to complete a task,
then whenever we are wrong, we will fail upwards:
the actual completion can only get longer for every possible bump in the road.
There’s little to no happy accidents, causing the task to finish sooner.

Почему так? В разработке ПО решения высоко переиспользуются.
Вы автоматически берёте это в расчёт, когда оцениваете.
«Мы уже делали что-то похожее, мы сможем использовать это».
Но несмотря на то, что разработка первой части занимает больше времени, чем вы думали,
в этот момент вы всё ещё предполагаете, что неудач не будет.
Мы слепы к нашей неспособности оценивать.
Нам, кажется, никогда не научиться тому, что будет всегда ударять:
от траты половины дня на поиск бага, вызванного забытой запятой где-нибудь,
до потери половины месяца в ожидании, что клиент предоставит какую-нибудь критическую часть
информации, про которую он сказал «будет в вашем ящике к концу дня».
И это если _они_ как-то магически знают, как оценивать!

Why is that? In software, solutions are highly reusable.
You automatically take this into account when estimating.
"We've already built something similar, we can reuse that."
But even though building the first part took longer than you thought,
you still assume that this time, there will be no setbacks.
We are blind to our own inability to estimate.
We never seem to learn that there will always be bumps:
from spending half a day looking for a bug, caused by a missing comma somewhere,
to losing half a month waiting for the client to deliver some critical piece
of information that they said "will be in your mailbox by the end of the day".
As if _they_ somehow magically do know how to estimate!

В интересах делания неявного явным,
давайте назовёт это **«риск»: вероятность, с которой основанная на времени оценка может быть неверной**.

In the interest of making the implicit explicit,
let's call this **"risk": the probability by which a time based estimate can be wrong**.

## Лучшие оценки

## Better Estimation

Мы куда-то пришли сейчас.
Есть лучший метод оценки, делая части более видимыми.
Формула в стори-поинтах выглядит так:

We're getting somewhere now.
There is a better way of estimating, by making the ingredients very visible.
The formula for estimating in story points, is:

```
стори-поинты = время × сложность × риск
```

```
story points = time x complexity x risk
```

Как это работает на практике и как проводить оценочные сессии — это тема для будущего поста в блоге.

How this works in practice, and how to run estimation sessions, is the topic for future blog post.

