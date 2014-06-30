TeleportClient
==============

 * requirejs:
```
bower install teleport-client --save
```

 * browserify:
```
npm install teleport-client --save
```

[TeleportServer](https://github.com/MyNodeComponents/TeleportServer)

<h5>Это RPC клиент, умеет:</h5>
 * Подлучать от сервера список телепортируемых объектов, имена их методов и событий.
 * Генирировать на основе полученного списка соответствующие объекты и методы.
 * Выбрасывать события серверных объектов из сгенирированных.

<h5>Ограничения:</h5>
 * Работает только с объектами.
 * Работает только с асинхронными методоми телепортируемых объектов, принимающими неограниченное количество аргументов и callback.
 * Выбрасываемые телепортированными объектами события могут содержать неограниченное количество аргументов.
 * Все аргументы передоваемые на сервер и результаты возвращаемые на клиента проходят через JSON.stringify -> JSON.parse.

<h5>Кил фича:</h5>
Если соединение с сервером кратковременно оборвется, то:
 * Клиент получит все выброшенные телепортированными объектами события за время отсутствия соединения.
 * Если клиентом был вызван некоторый метод до обрыва соединения, 
 	<br>то после переподключения он получит результат этого вызова.
 * Если клиент вызовет метод телепортированного объекта во время отсутствия соединения, 
 	<br>то он будет вызван когда соединение восстановится.

Клиент умеет отличать три типа обрыва соединения:
 * Проблемы интернет соединения.
 * Перезапуск сервера без изменения свойств телепортирумых объектов.
 * Перезапуск сервера с изменением свойств телепортируемых объектов.

<h5>requirejs и browserify совместимый:</h5>
Если подклюна библиотека requirejs или использован browserify, то TeleportClient будет сформирован как модуль,
иначе добавлен в глобальную область видимости.

<h5>Удовлетворения зависимостей:</h5>
 * browserify style:
```js
var TeleportClient = require('teleport-client');

var teleportClient = new TeleportClient({
		serverAddress: "ws://localhost:8000",
		autoReconnect: 3000
});
...
...
```
 * requirejs style:
```js
requirejs.config({
	baseUrl: 'bower_components/',
	paths: {
		TeleportClient: 'teleport-client/TeleportClient',
		util: 'my-helpers/util',
		EventEmitter: 'my-helpers/EventEmitter'
	}
});


requirejs(['TeleportClient'], function(TeleportClient) {
	var teleportClient = new TeleportClient({
		serverAddress: "ws://localhost:8000",
		autoReconnect: 3000
	})
});
```
 * classic style:
```html
	...
	<script type="text/javascript" src="https://rawgit.com/nskazki/web-Helpers/master/util.js"></script>
	<script type="text/javascript" src="https://rawgit.com/nskazki/web-Helpers/master/EventEmitter.js"></script>
	<script type="text/javascript" src="https://rawgit.com/nskazki/web-TeleportClient/master/TeleportClient.js"></script>
</head>
<body>
	<script type="text/javascript">
		var teleportClient = new TeleportClient({
			serverAddress: "ws://localhost:8000",
			autoReconnect: 3000
		});
	</script>
	...
```

<h5>Example:</h5>
```js
var teleportClient = new TeleportClient({
	serverAddress: "ws://localhost:8000",
	autoReconnect: 3000
})
	.on('warn', console.warn.bind(console))
	.on('error', console.error.bind(console))
	.on('ready', function(objectsProps) {
			console.log(objectsProps);

			teleportClient.objects.ipBox
				.getIps(someCallback)
				.on('newIps', someHandler);
	}).init();
```

<h5>Параметры принимаемые конструктором:</h5>
 * `serverAddress` - адрес и порт TeleportServer.
 	<br>default: `ws://localhost:8000`
 * `autoReconnect` - время задержки перед переподключением к серверу, после разрыва соединения.
 	<br>Разрешенные значения:
 	* если `false` - перезапуск произведен не будет.
 	* если число - то это время задержки в миллесекундах.
 	* default: `3000`

<h5>Публичные методы:</h5>
 * `init` - метод инициирующий объект.
 * `destroy` - метод прекращающий работу объекта.

<h5>Info events:</h5>
Эти события выбрасываются с одним аргументом, объектом, cодержащим:
 * поле `desc`, раскрывающим суть события. 
 * дополнительные поля раскрывающие внутреннее состояние TeleportClient.

События:
 * `debug` - логированние клиент-серверного обмена сообщениями.
 * `info` - логированние важных событий.
 	<br>В частности подключение к серверу и получении свойств телепортируемых оьъектов.
 * `warn` - логированние проблем не влияющих на дальнейшую работы программы. 
 	<br>Например получение неожиданного сообщения от сервера, или возврат результата выполнения команды без зарегестрированного каллбека.
 * `error` - логированние получение которые делают программу неработоспособной. 
 	<br>В частности ошибка соединения с сервером.

<h5>State events:</h5>
Эти события отражают текущее состояние TeleportClient.
<br>Выбрасываются без аргументов, если не указанно иное.

 * `ready` - признак успешного соединения с сервром и регистрации всех телепортируемых объектов.
 	<br>Выбрасывается с одним аргументом, содержащим свойства телепортированных объектов.
 * `destroyed` - признак успешного разрушения объекта вызовом метода `destroy`, 
 	<br>содинение с сервером разорванно, все каллбеки ожидающие результат вызваны с ошибкой.
 * `close` - признак разрыва соединения с сервером.
 	<br>Возникает если:
 	* Вызван метод `destroy` и при этом соединение с сервером еще было активно.
 	* Если соединение с сервером разорванно и при этом `autoReconnect` = `false`.
 * `reconnecting` - признак разрыва соединения, через время указанное в поле `autoReconnect` будет предпринята попытка переподключения.
 * `reconnected` - признак успешного переподключения к серверу.
 * `reconnectedToOldServer` - признак успешного переподключения к серверу востановления интернет соединения.
 	<br>Выбрасывается вместе с `reconnected`. 
 * `reconnectedToNewServer` - признак успешного переподключения к перезапущенному серверу. 
 	<br>Или если истекло время ожидания сервером переподключения этого клиента, и поэтому клиент заново зарегистрировался на сервере.
 	<br>Ожидающие результат выполнения каллбеки будут вызванны с ошибкой, команды ожидавшие переподключения так же отправленны не будут, так как это новый экземпляр сервера.
 	<br>Выбрасывается вмете с `reconnected`.
 * `serverObjectsChanged` - признак того, что сервер был перезапущенн и при этом на нем изменились свойства телепортированных объектов.
	<br>Рекомендую поймав это событие обновлять страницу, потому что обычно изменение серверного кода влечет изменение клиентского.
	<br>Выбрасывается вместе с `reconnectedToNewServer`.
