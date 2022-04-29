# Web Components 1. Введение

Веб-компоненты - набор технологий, задача которых позволить нам создавать повторно используемые HTML элементы. 

В HTML5 есть ряд элементов, которые являются нечто большим, чем просто теги. Например, `<audio>` - это целый компонент, который в браузере превращается в набор логики и контролов (play/pause/stop/progressbar). Идея веб-компонентов - дать возможность создавать свои теги, которые в браузере будут превращаться в кусок DOM со своей логикой и структурой. В разметке пишем просто тег, в барузере отрисовывается целый компонент.

## Предпосылки возникновения веб-компонентов

### Слабая семантика (div soup)

Изначально HTML задумывался как способ форматирования текста. В современных-веб приложениях HTML используется для построения сложных интерфейсов и компонентов, для которых не предусмотрены стандартные теги. Есть, конечно, возможности HTML5 в виде audio, video, canvas, но этого явно недостаточно. Веб-компоненты задумывались, как способ решить проблему слабой семантики и супа из div-элементов на страницах современных веб-приложений.

### Конфликты стилей

- Необходимо указывать максимально специфические CSS селекторы
- Использовать `!important`
- Нет гарантии, что другие стили не будут конфликтовать (особенно актуально, когда над проектом работает несколько разработчиков)

### Отсутствие пакетов (no bundling)

Нет инструмента, который позволит все зависимости подключить одним вызовом. Например, чтобы подключить полный бутстрап, нужно подключить стили с ядром и стили с темой, в сриптах jquery и бустрап-скрипты.

### Отсутствие шаблонов

Представим новостную ленту, где выводится 10 новостей. Каждая новость оформлена одинаково: у нее есть картинка, заголовок, короткий тест и дата публикации. Весь этот контент мы получаем аяксом с сервера в виде json, парсим этот json и как-то вставляем в разметку. В разметке гораздо удобнее пользоваться 1 шаблоном для новости, который будет копироваться 10 раз, но с разным контентом. На данный момент в современной веб-разработке существуют следующие способы реализации шаблонов:

- Сторонние шаблонизаторы: помещение html шаблона в `<script type="text/html">` (используется в шаблонизаторах типа mustache)
- Сохранение HTML в hidden элементах и клонирование по необходимости
- iframe

Отсутствие стандартов для реализации шаблонов реашется веб-компонентами.

### Отсутствие стандартов

Нет единых стандартов для создания компонентов с использованием HTML, CSS и JS. Есть решения в виде Bootsrap, Angular и т.п., но они реализуют свои механизмы и решают лишь часть проблем.

## Web Components

Веб-компоненты на самом деле - это набор технологий (также как и AJAX, например):

- HTML templates - возможность работы с HTML шаблонами через тег `<template>`
- Shadow DOM - позволяет инкапуслировать DOM дерево и избавляться от конфликтов стилей
- Custom elements - создание пользовательских HTML тегов
- HTML Imports - подключение HTML документа в другой HTML документ

Все это нативные технологии браузера, которые на данный момент (январь 2017) полноценно реализованы только в [Chrome](http://caniuse.com/#search=Web%20Components)

Однако, для браузеров, которые не поддерживают Веб-компоненты, можно использовать полифиллы. 

- [https://www.webcomponents.org/polyfills](https://www.webcomponents.org/polyfills)
- `bower install --save webcomponents/webcomponentsjs`
- `npm install --save webcomponents/webcomponentsjs`

Кроме того, есть обертки и надстройки для работы с Веб-компонентами, например, **Polymer**.

### HTML templates

Техники для использования шаблонов сегодня:

1. Использование тега `<script>`
`<script type="text/custom-name"> ... </script>`
Например, handlebars, kendo ui

2. Спрятанный DOM элемент
`<div style="display: none;" id="my-template"> ... </div>`

Для эффективного использования шаблонов необходимо:

- Ничего в шаблоне не должно **выполняться** или **визуализироваться**
- Необходим удобный способ клонирования DOM элементов

Особенности элемента `<template>`

- Не визуализируется до тех пор, пок ане будет клонирован
- Спрятан от селекторов: `document.querySelector('p')` не найдет элементы `<p>`, определенные в элементе `<template>`
- Может быть размещен в любой части страницы

#### Инициализация `<template>`

**HTML**

```html
<template>
  <p> Hello world from template</p>
</template>
```

**JS**

```javascript
// получение шаблона
var template = document.querySelector("template");
// клонирование шаблона, true - глубокое клонирование всего содержимого шаблона
var templateClone = document.importNode(template.content, true);
// добавление содержимого шаблона в DOM дерево
document.body.appendChild(templateClone);
```

#### Шаблонизация с помощью `<template>`

**HTML**

```html
<template>
  <p>Template name <span id="template-name"></span>.</p>
  <p>Has been copied <span id="template-count"></span> times.</p>
</template>
```

**JS**

```javascript
var templateCounter = 0;

// обработчик нажатия по кнопке
document.querySelector("#add-button").addEventListener("click", function() {

  // Создание шаблона
  var template = document.querySelector("template");
  var templateClone = document.importNode(template.content, true);

  // Внедрение данных в шаблон
  templateClone.querySelector("#template-name").textContent = "My Tempalte";
  templateClone.querySelector("#template-count").textContent = ++templateCounter;

  // Размещение шаблона в конце страницы
  document.body.appendChild(templateClone);
})
```

### Custom elements

Пользовательские HTML элементы должны содержать в имени хотя бы один знак дефиса (символ `-`).

Примеры правильных имен:

- `<my-component>`
- `<super-video>`
- `<registration-form>`

Неправильные имена:

- `<mycomponent>`
- `<super_video>`
- `<registrationForm>`

С помощью пользовательских элементов можно расширять существующие: 

`<button is="my-component"></button>`

#### Инициализация пользовательского элемента

```javascript
// Определение прототипа, который базируется на прототипе HTML элемента
var HelloWorldComponentProto = Object.create(HTMLElement.prototype);

// Определение функции обратного вызова, которая сработает после создания элемента
HelloWorldComponentProto.createdCallback = function() {
  this.innerHTML = "<h3>Hello world</h3>";
};

// Регистрация нового элемента.
var HelloWorldComponent = document.registerElement("helloworld-component", {
  prototype: HelloWorldComponentProto
});
```

#### Внедрение экзмепляра элемента в DOM

Вариант 1. Простая HTML разметка
```html
<helloworld-component></helloworld-component>
```

Вариант 2. Использование конструктора
```javascript
var component1 = new HelloWorldComponent();
document.body.appendChild(component1);
```

Вариант 3. Использование системного метода createElement()
```javascript
var component2 = document.createElement("helloworld-component");
document.body.appendChild(component2);
```

Вариант 4. Вставка через innerHTML
```javascript
document.body.innerHTML += "<helloworld-component></helloworld-component>";
```

#### Расширение системных элементов

```javascript
var MessageBlockProto = Object.create(HTMLDivElement.prototype);

MessageBlockProto.createdCallback = function() {
  this.innerHTML = "<a href='http://example.com'>Link</a>";
  this.style.border = "1px solid gray";
  this.style.padding = "15px";
};

var MessageBlock = document.registerElement("message", {
  prototype: MessageBlockProto,
  extends: "div"
});
```

#### Применение расширений

Вариант 1. Простая HTML разметка
```html
<div is="message"></div>
```

Вариант 2. Использование системного метода createElement()
```javascript
var div1 = document.createElement("div", "message");
document.body.appendChild(div1);
```

Вариант 3. Использование конструктора
```javascript
document.body.appendChild(new MessageBlock());
```

Вариант 4. Вставка через innerHTML
```javascript
document.body.innerHTML += "<div is='message'></div>";
```

#### Жизненный цикл пользовательских элементов

- **createdCallback** - экземпляр пользовательского компонента создан
- **attachedCallbak** - экземпляр пользовательского компонента добавлен в DOM
- **detachedCallback** - экземпляр пользовательского компонента удален из DOM
- **attributeChangedCallback** - атрибут пользовательского компонента добавлен, удален или изменен

https://jsfiddle.net/7y4v4myg/1/
