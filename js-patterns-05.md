# JS Шаблоны 5. Работа с DOM

## Содержание

- Шаблоны и антишаблоны работы с DOM
- Сценарии, работающие продолжительное время
- Развертывание готовых сценариев

Рассмотрим, как правильно замерять производительность веб-приложений и какие ошибки часто допускают новички при работе с DOM. 

## Шаблоны и антишаблоны работы с DOM

### Проверка возможностей браузера

**Антишаблон** - использование имени браузера, чтобы проверить его возможности.

```javascript
if (navigator.userAgent.indexOf("MSIE") !== -1) {
	document.attachEvent("onclick", function() {
		console.log("clicked");
	});
}
```

Лучше всегда завязываться на проверку конкретных методов, а не браузеров, которые их поддерживают. В данном случае лучше проверять поддерживет ли браузер метод attachEvent, потому что последние версии IE поддерживают addEventListener. Следующий код будет предпочтительней.

```javascript
if (document.attachEvent) {
	document.attachEvent("onclick", function() {
		console.log("clicked");
	});
}
```

Однако, вариант выше может вернуть null или false, что означает, что такой метод существует, но не имеет значения. Тогда более корректный код будет таким:

```javascript
if (typeof document.attachEvent !== "undefined") {
	document.attachEvent("onclick", function() {
		console.log("clicked");
	});
}
```

### Частые обращения к DOM

**Антишаблон**

```javascript
for (var i = 0; i < 1000; i++) {
	// обращаться к DOM на каждой итерации цикла не оптимально.
	document.getElementById("test").innerHTML += i + ", ";
}
```
В примере выше на каждой итерации цикла происходит поиск элемента #test и добавление в innerHTML какой-то информации. И таких поисков и добавлений - 1000. Это может стать причиной "тормозов" страницы.

При работе с циклами, всегда нужно проверять, работаем ли мы в теле цикла с DOM элементами. Если да, то всегда нужно эту работу оптимизировать.

Предпочтительный вариант. Мы кешируем все итерации в переменную и вне цикла единоразово добавляем данные на страницу. Хоть итераций в цикле и 1000, в DOM происодит всего 1 изменение.
```javascript
 var i, msg = "";
 for (i = 0; i < 1000; i++) {
 	msg += i + ", ";
 }
 document.getElementById("test").innerHTML += msg;
```

Можно замерить скорость выполнения кода встроенной функцией браузера console.time().

```javascript
// АНТИШАБЛОН
console.time("first");
for (var i = 0; i < 1000; i++) {
	document.getElementById("test").innerHTML += i + ", ";
}
console.timeEnd("first");

// Предпочтительней 
console.time("second");
var i, msg = "";
for (i = 0; i < 1000; i++) {
	msg += i + ", ";
}
document.getElementById("test").innerHTML += msg;
console.timeEnd("second");
```

### Работа со стилями

Рассмотрим на примере этого фрагмента разметки:
```html
<div id="test">Hello world</div>
```

**Антишаблон**

```javascript
document.getElementById("test").style.padding = "10px";
document.getElementById("test").style.border = "1px solid black";
```

В этом шаблоне есть ряд недостатков: двойное обращение к элементу и двойная перерисовка элемента через style. Предпочтительней получить ссылку на объект и далее работать с этой ссылкой:

```javascript
var e = document.getElementById("test");
e.style.padding = "10px";
e.style.border = "1px solid black";
```

Еще больший прирост производительности даст использование CSS классов:

```javascript
var e = document.getElementById("test");
e.className = "test";
```

Можно также замерить время выполнения:

```javascript
// АНТИШАБЛОН
console.time("1");
for (var i = 0; i < 1000; i++) {
	document.getElementById("test").style.padding = "10px";
	document.getElementById("test").style.border = "1px solid black";
}
console.timeEnd("1");

// Лучше
console.time("2");
for (var i = 0; i < 1000; i++) {
	var e = document.getElementById("test");
	e.style.padding = "10px";
	e.style.border = "1px solid black";
}
console.timeEnd("2");

// Еще лучше (главное не забыть добавить таблицу стилей)
console.time("3");
for (var i = 0; i < 1000; i++) {
	var e = document.getElementById("test");
	e.className = "test";
}
console.timeEnd("3");
```

Вывод: при работе со стилями в JS лучше использовать CSS классы, а не инлайн стили.

### Получение доступа к элементам DOM

Чтобы получить элемент DOM, можно использовать getElementById,  getElementsByName, getElementsByTagName,  getElementsByClassName.
```javascript
var div1 = document.getElementById("test");
div1.innerHTML = "Hello world";
```

В большинстве случаев функции querySelector работают быстрее чем другие функции для получения элементов из DOM.

```javascript
var div2 = document.querySelector("#test");
div2.innerHTML = "Hello world";
```

### Создание элементов DOM

**Антишаблон**

```javascript
var p, t;

p = document.createElement("p");
t = document.createTextNode("First paragraph");
p.appendChild(t);
document.body.appendChild(p); // плохо

p = document.createElement("p");
t = document.createTextNode("Second paragraph");
p.appendChild(t);
document.body.appendChild(p); // плохо
```

Код выше плох тем, что дважды выполняется запрос к DOM дереву и дважды перерисовывается страница. Предпочтительнее все добавления производить не в реальный DOM, а в виртуальный. Для этого подойдет *Doument Fragment*. **Doument Fragment** - фрагмент документа, в который можно добавлять дочерние элементы. При этом дочерние элементы фрагмента не будут отображаться на странице до тех пор, пока фрагмент не станет дочерним элементом существующего элемента DOM.

```javascript
var p, t, fragment;

fragment = document.createDocumentFragment();

p = document.createElement("p");
t = document.createTextNode("First paragraph");
p.appendChild(t);
fragment.appendChild(p); // добавление элемента в фрагмент

p = document.createElement("p");
t = document.createTextNode("Second paragraph");
p.appendChild(t);
fragment.appendChild(p); // добавление элемента в фрагмент

document.body.appendChild(fragment); // добавление фрагмента в тело документа.
// все элементы которые есть в фрагменте станут дочерними элементами документа.
```

## Сценарии, работающие продолжительное время

Браузер всегда отслеживает, сколько времени тратится на выполнение сценария. И если браузер видит, что сценарий работает продолжительное время, он старается его принудительно остановить. JS - это однопоточный язык и при выполнении тяжелых скриптов, время, отведенное на выполнение потока, тратится на выполнение скрипта, а не отрисовку содержимого.

**Антишаблон**

```javascript
function calcFib(x) {
  if (x > 1) {
    return calcFib(x - 1) + calcFib(x - 2);
  } else {
    return x;
  }
}

for (var i = 0; i < 500; i++) {
  document.write(i + " = " + calcFib(i) + "<br />");
}
```

Эта функция рекурсивно вычисляет Чи́сла Фибона́ччи, но т.к. она неоптимизирована, а по условию нужно вычислить 500 чисел, то выполнение в браузере прервется (браузер покажет модальное окно с вопросом подождать или убить процесс). Мы могли бы использовать мемоизацию. Но если мы имеем дело с обработкой изображений на стороне клиента или большими объемами данных, то мемоизация уже не поможет.

Чтобы нашему скрипту выделялось больше ресурсов и процессорного времени, мы можем воспользоваться веб-воркерами. Веб-воркеры позволяют создать отдельный поток в браузере и возложить на этот поток определенную работу. 

```javascript
window.onload = function() {
  document.getElementById("startButton").onclick = function() {
    // Создание нового потока
    var worker = new Worker("fibonachiWorker.js");

    // Создание обработчика для получения сообщений от потока
    worker.addEventListener("message", function(e) {
      document.getElementById("output").innerHTML += e.data + "<br />";
    }, false);

    // запуск потока
    worker.postMessage("");
  }
}
```

fibonachiWorker.js
```javascript
function calcFib(x) {
  if (x > 1) {
    return calcFib(x - 1) + calcFib(x - 2);
  } else {
    return x;
  }
}

addEventListener("message", function() {

  for (var i = 0; i < 500; i++) {
    postMessage(i + " = " + calcFib(i) + "<br />");
  }

}, false);
```

Интерфейс воркера
```html
<button id="startButton">Start</button>
<p id="output"></p>
```

## Развертывание готовых сценариев

- Подключать скрипты перед закрывающим тегом </body>
- Конкатенировать скрипты в 1 файл для уменьшения запросов к серверу
- Минифицировать скрипты для уменьшеня их веса.
- Использовать инструменты профилирования кодаи приложения в целом