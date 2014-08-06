---
layout: post
title: Расширение Chrome шаг за шагом
---

{% include tldr.html text="Как легко писать расширения для Chrome с использованием современных инструментов разработки" %}

## Введение, мысли для развития темы

Небольшой disclamer: я не являюсь frontend-разработчиком, ничего не знаю о том, как писать расширения для Chrome.
Но с помощью этой статьи я хочу доказать себе и всем читателям, что разрабатывать расширения для браузера можно легко
и непринуждённо. Мы проводим в браузере кучу времени, а если делаем сайты — то вообще почти всё время.

План действий: посмотрим, что есть в наше время для разработки, сделаем простое приложение. Приступим!


## Подготовка почвы под ногами

Что у нас тут есть… Grunt, Gulp, NPM, куча всяких страшных слов. Ужас какой-то. Видимо, для этого бывают какие-то
системы сборки, готовые шаблоны, программисты же ленивые твари. О! Yeoman. По описанию — то, что нужно.

Ставим NPM, если его ещё вдруг нет. Это такой пакетный менеджер для NodeJS. С его помощью ставим Yeoman:

```bash
$ sudo npm install -g yo
```

Пара красивых цветных строчек в терминале и у нас есть Yeoman.

Смотрим, какие рецепты нам подходят. О, чудо! Есть прямо готовый для Chrome extension. Заходим, смотрим, ставим:
https://github.com/yeoman/generator-chrome-extension

```bash
$ mkdir bookmark-counter-extension
$ cd bookmark-counter-extension
$ sudo npm install -g generator-chrome-extension
$ yo chrome-extension
```

Скрипт задаст пару вопросов, ничего страшного. Отвечайте так, как посчитаете нужным.

```bash
[?] What would you like to call this extension? bookmark counter extension
[?] How would you like to describe this extension? Bookmark counter extension
```

А вот дальше будет вопрос каверзнее. Непонятно, что на него отвечать:

```bash
[?] Would you like to use UI Action? 
‣ No 
  Browser 
  Page 
```

Ищем в Google, приходим вот на эту страницу:
https://developer.chrome.com/extensions/browserAction

Похоже, это то, что нужно. Выбираем Browser.

Далее вопрос, какие фичи мы будем юзать. Наверно, пока никакие.

Какие разрешения? Видимо, Bookmarks.

Получилось как-то так:

```bash
 ~/web/bookmark-counter-extension  yo chrome-extension
[?] What would you like to call this extension? bookmark counter extension
[?] How would you like to describe this extension? Bookmark counter extension
[?] Would you like to use UI Action? Browser
[?] Would you like more UI Features? 
[?] Would you like to use permissions? Bookmarks
   create Gruntfile.js
   create package.json
   create .gitignore
   create .gitattributes
   create .bowerrc
   create bower.json
   create .jshintrc
   create .editorconfig
   create app/manifest.json
   create app/popup.html
   create app/scripts/popup.js
   create app/images/icon-19.png
   create app/images/icon-38.png
   create app/scripts/background.js
   create app/scripts/chromereload.js
   create app/styles/main.css
   create app/_locales/en/messages.json
   create app/images/icon-16.png
   create app/images/icon-128.png
   invoke   mocha


I'm all done. Running bower install & npm install for you to install the required dependencies. If this fails, try running the command yourself.


   create     test/bower.json
   create     test/.bowerrc
   create     test/spec/test.js
   create     test/index.html


-----------------------------------------
Update available: 1.2.1 (current: 1.1.2)
Run npm update -g yo to update
-----------------------------------------
```

Собственно, нам создали базовые файлы для расширений. Принимать вечные решения о структуре директорий и именованию файлов больше не нужно, ура!
Можно просто кодить в своё удовольствие.

Что лежит в папочке:

```bash
 ~/web/bookmark-counter-extension  l
итого 60K
drwxr-xr-x  4 ilya ilya 4,0K авг.   3 22:01 .
drwxr-xr-x 15 ilya ilya 4,0K авг.   3 21:48 ..
drwxr-xr-x  7 ilya ilya 4,0K авг.   3 22:01 app
-rw-r--r--  1 ilya ilya  112 июля  13 16:07 bower.json
-rw-r--r--  1 ilya ilya   44 июля  13 16:07 .bowerrc
-rw-r--r--  1 ilya ilya  415 июля  13 16:07 .editorconfig
-rw-r--r--  1 ilya ilya   11 июля  13 16:07 .gitattributes
-rw-r--r--  1 ilya ilya   91 июля  13 16:07 .gitignore
-rw-r--r--  1 ilya ilya 9,4K авг.   3 22:01 Gruntfile.js
-rw-r--r--  1 ilya ilya  439 июля  13 16:07 .jshintrc
-rw-r--r--  1 ilya ilya  895 авг.   3 22:01 package.json
drwxr-xr-x  3 ilya ilya 4,0K авг.   3 22:01 test
-rw-r--r--  1 ilya ilya   27 авг.   3 22:01 .yo-rc.json
```

Создаём реп, не обращаем внимания на всякие файлы настроек, есть git-овые файлы, пробуем их:

```bash
$ git init
$ git add .gitignore && git commit -m"init"
$ git status
$ l app
```

Пожалуй, добавим и остальные файлы, которые не в .gitignore:
```bash
$ git add bower.json .bowerrc .editorconfig .gitattributes app/images/ app/_locales/ \
app/manifest.json app/popup.html app/scripts/ app/styles/ .jshintrc Gruntfile.js package.json test/ .yo-rc.json
$ gc -m"app skeleton"
```

Ну и сразу запушим в какой-нибудь реп на github, чего уж там.
```bash
 ~/web/bookmark-counter-extension   master  git remote add origin git@github.com:da-eto-ya/bookmark-counter-extension.git
 ~/web/bookmark-counter-extension   master  git push -u origin master
Counting objects: 36, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (26/26), done.
Writing objects: 100% (36/36), 18.07 KiB, done.
Total 36 (delta 0), reused 0 (delta 0)
To git@github.com:da-eto-ya/bookmark-counter-extension.git
 * [new branch]      master -> master
Branch master set up to track remote branch master from origin.
```

Пока хватит, уже начало положено.


## Наелся?

Кажется, если так продолжать, то самого расширения не будет сделано никогда.
Ибо время и желания — штуки слишком текучие.
Поэтому попробуем перевести историю в другое русло.
О том, как сделать MVP, шнягу, покрытую тестами, а потом — и супер-пупер-прожект,
если отсыпят вдохновения. Так что творю MVP, а потом уже статьи и всё остальное.

Да, меня не особо колышет, что этот лично-дневниковый пост будет в анналах github.

Начинаем вот с этого и следуем почти по-тупому:
https://developer.chrome.com/extensions/getstarted

Пара минут на домашние дела, установку свежего EAP PhpStorm, просмотр мельком документации
(читать лень) и имеем файл manifest.json с именем и кратким описанием экстеншена.
Включаем его в Chrome, радуемся.

Иконки нет. Дефолтная убога. Что делать? Ну, добавим иконку.
Находим сходу вот это:
https://developer.chrome.com/extensions/manifest/icons

Заходим сюда через поиск, выбираем первую попавшуюся приличную иконку:
https://www.iconfinder.com/iconsets/flat-ui-icons-24-px

В размере 128 почему-то не хочет использовать. Ок, сделаем несколько размеров тупо через Gimp.
Ах, да. Нужно было прочитать, что дефолтный размер 48. Ну, ок.

Теперь вот на этой странице отображается иконка:
chrome://extensions/

Однако, отдельной кнопочки около адресной строки не появилось. Возвращаемся к первому доку,
узнаём про browser_action. Пробуем поставить 32, что-то размыто.

    "browser_action": {
        "default_icon": "img/icon-16.png"
    }

Забиваем в гугле «chrome extension browser_action icon size».
https://developer.chrome.com/extensions/browserAction

Ага. 19 и 38. Ок. Берём снова наше изображение в 128 и подгоняем размер.
Теперь уже куда ни шло. Потом можно будет поиграть и попробовать выбрать картинку, где
совсем не будет размытия. Но это всё потом.

В принципе, пока нам не нужно от browser_action никакого экшена, висит иконка - и ладно.
Основная работа нашего расширения где-то в бэкграунде, должны быть какие-то события,
связанные с букмарками. Ищем в гугле и находим:
https://developer.chrome.com/extensions/event_pages
https://developer.chrome.com/extensions/bookmarks

Ещё видим пример как раз background-страниц, но там говорят, что нужны event-pages (это моднее).
https://developer.chrome.com/extensions/background_pages

Ок, будем использовать просто как подсказку.

Понимаем, что нужно создать какие-то event-pages и что нужны будут разрешения на букмарки.
Ок, пробуем просто навешаться на добавление букмарка. Будем в этот момент показывать счётчик.
Хранить пока ничего не будем, пусть счётчик сбрасывается каждый раз при запуске.

Chrome что-то не нравится в переменных… Ну, добавим ему либу, чтобы он знал:
https://github.com/borisyankov/DefinitelyTyped

В качестве действия по подписке на события пусть просто плюется алертом… Чтобы проверить идею,
так сказать. Счётчик, видимо, связан с browser_action, придётся чуть позже найти.

Проверили — работает. По ходу, ему пофиг, что добавляется, страница или папка.
Смотрим приходящую в листенер инфу:
https://developer.chrome.com/extensions/bookmarks#type-BookmarkTreeNode

Непонятно.

Так что вместо алерта плюём bookmark в консоль и попробуем подебажить (нечего всё в гугл лезть).

chrome.bookmarks.onCreated.addListener(function (id, bookmark) {
    console.log(bookmark);
});

Чтобы увидеть, что происходит, нужно нажать на странице расширений «Фоновая страница»,
попадём в веб-инспектор расширения.

Попробовали, видим какую-то такую фигню:

Object {dateAdded: 1407358907499, dateGroupModified: 1407358907499, id: "4175", index: 21, parentId: "1"…}
dateAdded: 1407358907499
dateGroupModified: 1407358907499
id: "4175"
index: 21
parentId: "1"
title: "fold"
__proto__: Object
 app.js:4
Object {dateAdded: 1407358934026, id: "4176", index: 22, parentId: "1", title: "Расширения"…}
dateAdded: 1407358934026
id: "4176"
index: 22
parentId: "1"
title: "Расширения"
url: "chrome://extensions/"
__proto__: Object
 app.js:4
Object {dateAdded: 1407358980625, id: "4177", index: 23, parentId: "1", title: "5"…}
dateAdded: 1407358980625
id: "4177"
index: 23
parentId: "1"
title: "5"
url: "http://w/"
__proto__: Object

Получается два кандидата на роль отсеивателя: url (только у страниц) и dateGroupModified (похоже, что только у папок).
Возвращаемся к докам — и правда, url используется только для страниц (логично). Будем делить по нему.
Делаем алерт только на страницы, проверяем. Работает. Это не может не радовать.

Возвращаемся к вопросу «Как же это показать?»
Гугл: «chrome extension development numbers on icon»
Первый же ответ:
http://stackoverflow.com/questions/5759130/google-chrome-extension-numbers-on-the-icon
https://developer.chrome.com/extensions/browserAction#method-setBadgeText

Ну и нам нужно где-то хранить количество иконок. Как ни странно, заведём переменную для этого.
Хром очень щепетливый к типам данных, поэтому напарываемся на то, что текст должны быть истинной строчкой.
Ок, приводим наше число к строке любым методом. Я предпочитаю использовать явный .toString().
Заодно и тайтл поставим. Пусть там будет чуть более подробная информация.

