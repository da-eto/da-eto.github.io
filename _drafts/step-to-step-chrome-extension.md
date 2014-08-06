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

