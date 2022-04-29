# JS Шаблоны 4. Шаблоны проектирования

## Содержание

- Singleton
- Factory
- Iterator
- Strategy
- Facade
- Proxy
- Mediator
- Observer

Существуют паттерны ООП, описанные командой разработчиков, известных как "Банда Четырех" (Gang of Four, GoF). Эти шаблоны актуальны для таких ЯП как C#, C++, Java и прочих строго типизированных языков. Когда же мы говорим о JS, то не все шаблоны из других ЯП могут быть применимы для него. Дело в том, что такие шаблоны решают задачи, связанные с ограниченностью строго типизированных ЯП. Но т.к. JS сам по себе язык с динамической типизацией и в нем нет таких понятий как класс (что-то похожее на класс можно организовать с помощью конструктора), наследование (есть прототипизация) и т.п., то использование шаблонов из строго типизитрованных языков в JS становится просто бессмысленным. Следующие 8 паттернов действительно есть смысл применять в JS.

## Singleton

Суть шаблона - обеспечить в приложении наличие только 1 экземпляра определенного класса. Но так как в JS нет классов, то новый объект уже является единственным в приложении. Организовать Singleton можно 2-мя способами: либо использовать литерал объекта (что уже по определению является синглтоном), либо использовать конструктор (который является очень отдаленным примером использования классов в JS).

### Singleton с помощью литерала объекта

```javascript
var obj1 = {
  name: 'test1'
}

var obj2 = {
  name: 'test1'
}

// Сравним адреса ссылок наших объектов
// Строгое сравнение без приведения типов
console.log( obj1 === obj2 ); // false
// Сравнение с приведением типов
console.log( obj1 == obj2 ); // false
```

https://jsfiddle.net/fo0ddn6v/

### Singleton с помощью конструктора
Рассмотрим, как с помощью конструктора можно организовать код, который будет работать по шаблону Singleton.

```javascript
function Sun() {
  // Проверяем наличие экземпляра созданного ранее.
	// Если св-во instance является объектом,
	// это означает, что конструктор ранее запускался.
	// Поэтому надо просто вернуть существующий экземпляр.
	if ( typeof Sun.instance === 'object' ) {
	  return Sun.instance;
	}
	
	// Если Sun.instance === 'undefined',
	// то определяем логику самого конструктора,
	// т.е. инициализируем его свойствами
	// и т.о. создаем новый экземпляр
	this.color = 'yellow';
	this.isBig = true;
	
	// В св-во instance сохраняем ссылку на контекст,
	// т.е. сохраняем созданный экземпляр для повторного использования
	Sun.instance = this;
	
	// Неявный возврат экземпляра
	// return this;
}

var sun1 = new Sun(); // Получаем ссылку на новый экземпляр
// Sun.instance = null;
var sun2 = new Sun(); // Получаем ссылку на существующий экземпляр
console.log(sun1 === sun2); // true - ссылки одинаковые
```

Но у этого шаблона есть недостаток: instance является открытым св-вом конструктора и если мы, например, после первичной инициализации экземпляра вызовем `Sun.instance = null;`, то нарушим работу шаблона. Необходимо инкапсулировать instance.

## Factory

Назначение Фабрики - создавать объекты (по аналогии с фабриками из реальной жизни, которые производят мебель, одежду, машины и прочие продукты). Фабрика - это объект, порождающий новые сущности. 

С помощью этого паттерна можно решить следующие проблемы:
- Выполнение повторяющихся операций при создании объектов
- Создание объектов, тип которых неизвестен на этапе компиляции приложения (важно для языков со статической типизацией)

Последний пункт для JS бессмысленнен, поскольку JS - динамический язык программирования и вместо этапа компиляции в нем есть этап интерпретации. Однако для строго типизированных языков (C#, C++, Java) это проблема и решается она с помощью наследования и полиморфизма. В JS нет ни первого ни второго в силу его природы. Поэтому применение шаблона Фабрика в JS используется скорее для того, чтобы собрать воедино ту логику, с помощью которой мы создаем новые объекты.

```javascript
// Родительский конструктор, выступает в роли
// абстрактного базового типа данных
function Control(){};

// Метод родителя
Control.prototype.render = function(type){
  document.write( 'Control: ' + this.name + '<br> ' + this.control + '<br><br>' );
}

// Фабричный метод
Control.create() = function(type){
  var ctor = type,
	    newControl;
			
	// Проверка наличия конструктора для указанного типа объекта
	if ( typeof Control[ctor] !== 'function' ) {
	  // Выбрасываем исключение, если конструктор не найден
		throw {
		  name: Error,
			message: '' + ctor + ''
		}
	}
	
	// На этом этапе существование конструктора проверено
	// Устанавливаем для конструктора в качестве прототипа объект Control
	// Выполняем данную операцию 1 раз
	if ( typeof Control[ctor].prorotype.render !== 'function' ) {
	  Control[ctor].prorotype = new Control();
	}
	
	// Создаем экземпляр указанного типа
	newControl = new Control[ctor]()
	
	return NewControl;
}

// Специализированные конструкторы (продукты нашей фабрики)
// Выступают в роли производных типов данных
Control.Button = function(){
  this.name = 'Button';
	this.control = '<input type="button" value="testButton" />';
}

Control.TextBox = function(){
  this.name = 'textBox';
	this.control = '<input type="text" />';
}

Control.RadioButton = function(){
  this.name = 'RadioButton';
	this.control = '<input type="radio" /> RadioButton';
}
```

Использованеи Фабрики:
```javascript
var btn = Control.create('Button');
var txt = Control.create('TextBox');
var rbtn = Control.create('RadioButton');

btn.render();
txt.render();
rbtn.render();
```

## Iterator

Реализацию шаблона Итератор можно найти практически во всех языках программирования. Например, в C# даже есть отдельное пространство имен - system collections - место, где расположены коллекции. Коллекции по сути являются реализацией шаблона Итератор. Шаблон Итератор нужен для того, чтобы скрыть реализацию объекта, который содержит в себе какую-то совокупность данных (в виде определенной структуры). Иными словами, нам нужно обеспечить доступ к этим данным, не раскрыв способ их хранения. 

```javascript
var collection = (function(){
  // Закрытые данные
  var current = -1, // Указатель на элемент в структуре данных
	    data = [1,2,3,4,5,6,7,8,9,0], // Скрытая структура данных
			count = data.length;
	
	// Открытые данные
	return {
	  // Метод для премещения по структуре данных
		moveNext: function(){
		  if ( current == count - 1 ) {
			  return false;
			} else {
			  current++;
				return true;
			}
		}
		
		// Метод для получения текущего элемента
		getCurrent: fucntion(){
		  return data[current]
		}
		
		// Метод для сброса указателя на начало коллекции
		reset: function() {
		  current = -1;
		}
	}
})();
```

Использование шаблона Итератор:
```javascript
while ( collection.moveNext() ) {
  var temp = collection.getCurrent();
	document.write( '<p>' + temp + '</p>' );
}
```

## Strategy

Задача шаблона Стратегия - предоставить изменение алгоритма поведения, но при этом не менять интерфейс объекта.

```javascript
var validator = {
  types: {}, // Здесь будут храниться стратегии валидации
  messages: [], // Сюда будут записывать сообщения об ошибках, если таковые будут
  config: {}, // Конфиг для указания какие стратегии в каких ситуациях использовать

  // Используется всего один метод, внутри которого и спрятаны стратегии
	// проверяет корректность значений в объекте data в соответствии с настройками указанными в свойстве config
  // возвращает true при наличии ошибок, false - если свойства объекта заполнены правильно.
  validate: function(data) {
    var i,
      msg,
      type,
      invalid,
      checker;

    this.messages = [];

    for (i in data) {
      if (data.hasOwnProperty(i)) {
        type = this.config[i]; // получаем тип проверки для свойства
        checker = this.types[type]; // получаем объект выполняющий проверку

        if (!type) {
          continue;
        }
        if (!checker) {
          throw {
            name: "ValidatorError",
            messgae: "Не найден валидатор " + type
          }
        }

        invalid = checker.validate(data[i]);
        if (invalid) {
          msg = "Не правильное значение для " + i + ", " + checker.message;
          this.messages.push(msg);
        }
      }
    }

    return this.hasErrors();
  },

  hasErrors: function() {
    return this.messages.length !== 0;
  }
};

// объект выполняет проверку наличия значения в свойстве.
validator.types.required = {
  validate: function(value) {
    return value === "";
  },
  message: "Обязательное значение"
};

// объект проверяет значение на соответствие целочисленному типу
validator.types.number = {
  validate: function(value) {
    return !/\d+/.test(value);
  },
  message: "Значение должно быть числом"
};

// проверяет формат email адреса
validator.types.email = {
  validate: function(value) {
    return !/^\w+@[a-zA-Z_]+?\.[a-zA-Z]{2,3}$/i.test(value);
  },
  message: "Значение должно быть email адресом"
};
```

Использование шаблона Стратегия
```javascript
// Правильно заполненный объект
var data1 = {
  firstName: "Ivan",
  lastName: "Ivaonv",
  age: 25,
  email: "ivanov@example.com"
};

// Неправильно заполненный объект
// Надо придумать стратегии валидации
var data2 = {
  firstName: "Ivan",
  lastName: "",
  age: "qwe",
  email: "example"
};

// настройки объекта для проверки
validator.config = {
  firstName: "required", // Стратегия required
  lastName: "required",
  age: "number", // Стратегия number
  email: "email" // Стратегия email
};

var result = validator.validate(data1);
console.log(result); // false - ошибок не найдено

// проверка и вывод ошибок
if (validator.validate(data1)) {
  console.dir(validator.messages);
}

result = validator.validate(data2);
console.log(result); // true - есть ошибки

// проверка и вывод ошибок
if (validator.validate(data2)) {
  console.dir(validator.messages);
}
```

## Facade

Задача шаблона Фасад - предоставить альтернативный интерфейс для объекта. Может быть применен при упрощении интерфейса определенного объекта или для того, чтобы скрыть различия использования определенных функций в разных браузерах. Это довольно распространенный шаблон, который часто используется в библиотеках. Например, в ранних редакциях jQuery метод on() является фасадом для методов `addEventListener()` и `attachEvent()`, чтобы обеспечить поддержку событий в IE6-8, которые понимают только последний метод.

```javascript
// Фасад, который скрывает от пользователя вызов двух других системных методов.
var Events = {
  stop: function(e) {
    e.preventDefault(); // прекратить распространение события по дереву DOM
    e.stopPropagation(); // отменить действие предусмотренное по умолчанию.
  }
}
```

## Proxy

Прокси - это объект, который контролирует работу другого объекта. Мы можем работать либо с реальным объектом, лиьбо с прокси, который перенаправляет вызовы на реальный объект. Для чего может быть использован прокси. Например, у вас есть объект с большим функционалом. Но вам необходимо ограничить права доступа для разных пользователей и разным пользователям предоставлять только часть фнкционала. В этом случае прокси-объект будет кстати. Другой пример: реальный объект взаимодействует с сервером, а прокси используется для кеширования. 

Рассмотрим пример с кешированием.
```javascript
// Реальный объект
var http = {
  makeRequest: function(id, callback) {
    // Имитация запроса на сервер
    setTimeout(function() {
      callback("Данные от сервера " + new Date().getTime());
    }, 3000);
  }
}

// Прокси объект (реализован по шаблону Модуль)
var proxy = (function() {

  // кэш (закрытая часть)
  var cache = {};
  
	// Открытая часть
  return {
    makeRequest: function(id, callback) {
      if (cache[id]) {
        callback(cache[id]);
      } else {
        http.makeRequest(id, function(data) {
          cache[id] = data;
          callback(data);
        });
      }
    }
  }
})();
```

Интерфейс нашего приложения
```html
<input id="forRequest" type="text" /><span id="loader" style="display:none;">Загрузка...</span>
<br />
<input id="requestBtn1" type="button" value="Запрос" />
<input id="requestBtn2" type="button" value="Запрос (с использование прокси-объекта)" />
<br />
<p id="output"></p>
```

Добавляем обработчики
```javascript
function get(id) {
  return document.getElementById(id);
}

function callback(data) {
  loader.style.display = "none";
  output.innerHTML += data + "<br />";
}

get("requestBtn1").addEventListener("click", function() {
  get("loader").style.display = "inline";
  var id = get("forRequest").value;
  // использование напрямую объекта http
  http.makeRequest(id, callback);
});

get("requestBtn2").addEventListener("click", function() {
  get("loader").style.display = "inline";
  var id = get("forRequest").value;
  // использование proxy который обращается к объекту http
  proxy.makeRequest(id, callback);
});
```

## Mediator

Медитатор (Посредник) позволяет уменьшить кол-во связей между объектами. Предположим, нам нужно создать чат-комнату. Каждый пользователь чата - новый объект, который должен знать о существовании других пользователей-объектов. Таких объектов может быть очень много и организовать одновременную связь между всеми может быть ресурсоемкой задачей. Чтобы уменьшить кол-во связей и легко добавлять/убирать пользователей можно использовать шаблон Медиатор, где объекты связаны не непорсдественно друг с дургом, а они связаны с единым медиатором.

Таким образом, Посредник или Медиатор - паттерн проектирования обеспечивающий взаимодействие множества объектов, формируя при этом слабую связность и избавляя объекты от необходимости явно ссылаться друг на друга.

В данном примере показано использование медиатора на примере игры с двумя игроками и доской результатов. Игроки на клавиатуре нажимают клавиши 1 или 0 а медиатор, определяя нажатия, обновляет доску результатов.

```javascript
var mediator = {
  // объекты которые объединяет медиатор
  players: {},

  // метод для инициализации всех объектов
  setup: function() {
		// Теперь медиатор знает о Player
    this.players.player1 = new Player("Player 1");
    this.players.player2 = new Player("Player 2");
  },

  // обновление интерфейса, если кто-то из игроков сделал ход.
  updateMediator: function() {
    var score = {
      Player1: this.players.player1.points,
      Player2: this.players.player2.points
    };
    scoreboard.update(score);
  },

  // обработчик действия пользователя
  keypress: function(e) {
    e = e || window.event;

    if (e.keyCode === 49) { // 1
      mediator.players.player2.updatePlayer();
      return;
    }
    if (e.keyCode === 48) { // 0
      mediator.players.player1.updatePlayer();
      return;
    }
  }
}

var scoreboard = {
  // HTML элемент, который должен обновляться.
  element: null,

  // обновляет счет на экране
  update: function(score) {
    var i, msg = "";
    for (i in score) {
      if (score.hasOwnProperty(i)) {
        msg += "<p>" + i + " = " + score[i] + "</p>";
      }
    }
    this.element.innerHTML = msg;
  }
}

// Игрок
function Player(name) {
  this.name = name;
  this.points = 0;
}

// Метод для обновление счета игрока
Player.prototype.updatePlayer = function() {
  this.points++;
  mediator.updateMediator(); // Теперь Player знает о медиаторе
}

mediator.setup();
scoreboard.element = document.getElementById("scoreboard")
window.onkeypress = mediator.keypress;
```

## Observer

Шаблон Observer (или "Издатель-подписчик") - наиболее часто встречающийся шаблон, где есть пользовательский интерфейс. С помощью шаблона Observer можно создать механизм у объекта, который позволит получать сообщения от других объектов об изменениях их состояния, тем самым наблюдая за ними. В JavaScript этот шаблон еще называют "Механизм собственных событий". Например, возьмем кнопку: **кнопка** - это *издатель*, потому что, когда мы нажимаем на кнопку, происходит событие. На это событие должен отреагировать **обработчик**, который является *подписчиком*.

Задача Издателя - определить хранилище для функций и определить интерфейс, в который можно было бы добавить/убрать эти функции. Задача Подписчика - определить момент, когда взять из этого хранилища функцию и запустить ее.

Функция makePublisher(obj) просто перегоняет все свойств из эталонного Издателя (объекта publisher) в тот, объект, который мы подали на вход функции (передали в качестве аргумента).

```javascript
// объект-издатель - содержит методы для создания подписчиков и оповещения их о изменениях.
var publisher = {

  // коллекция подписчиков
  // все подписчики хранятся в виде массива функций.
  subscribers: {
    // событие defaultEvent на которое пока нет подписчиков
    defaultEvent: []
  },

  // метод для добавления подписчиков fn - функция обработчик, event - имя события, на которое вешается обработчик.
  subscribe: function(fn, event) {

    // если имя события не было указано - рассматриваем вызов метода как подписку на событие по умолчанию.
    event = event || 'defaultEvent';

    // если в коллекции подписчиков еще нет подписчиков на данное событие то добавить свойство заполнив его пустым массивом.
    if (typeof this.subscribers[event] === "undefined") {
      this.subscribers[event] = [];
    }
    // добавление подписчика на событие
    this.subscribers[event].push(fn);
  },

  // метод инициирует событие указанное в первом параметре
  publish: function(args, type) {
    this.visitSubscribers('publish', args, type);
  },

  // метод удаляет указанную функцию подписчика
  unsubscribe: function(fn, type) {
    this.visitSubscribers('unsubscribe', fn, type);
  },

  // вспомогательный метод для работы с подписчиками
  visitSubscribers: function(action, arg, event) {
    var eventType = event || 'defaultEvent',
      subscribers = this.subscribers[eventType],
      i,
      max = subscribers.length;

    for (i = 0; i < max; i++) {
      if (action == 'publish') {
        // запуск событияе - вызов функций-обработчиков всех подписчиков
        subscribers[i](arg);
      } else {
        // удаление определенного подписчика из массива функций-обработчиков подписчиков
        if (subscribers[i] === arg) {
          subscribers.splice(i, 1);
        }
      }
    }
  }
}

// метод для преобразования любого объекта в издателя
function makePublisher(obj) {
  var i;
  for (i in publisher) {
    if (publisher.hasOwnProperty(i) && typeof publisher[i] === "function") {
      obj[i] = publisher[i];
    }
  }

  obj.subscribers = {
    any: []
  };
}

// объект который будет преобразован в издателя.
var button = {
  click: function() {
    // запуск события click c аргументами 123
    this.publish('123', 'click');
  },

  doubleClick: function() {
    // запуск события doubleClick c аргументами abc
    this.publish('abc', 'doubleClick');
  }
}

// превращаем button в издателя
makePublisher(button);

// объект с функциями-обработчиками 
var handlerObject = {
  handler1: function(e) {
    document.write("handler 1 " + e + " <br />");
  },

  handler2: function(e) {
    document.write("handler 2 " + e + " <br />");
  }
}

button.subscribe(handlerObject.handler1, "click");
button.subscribe(handlerObject.handler2, "doubleClick");

button.click();
button.click();
button.click();
button.doubleClick();

button.unsubscribe(handlerObject.handler1, "click");
button.click();
```
