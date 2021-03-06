# Контейнер

- [Вступление](#Вступление)
- [Применение](#Применение)
    - [Метод Make](#Метод-Make)
    - [Алиасы](#Алиасы)
    - [Проверка на существование](#Проверка-на-существование)
- [Получение путей](#Получение-путей)
- [Автоматическое внедрение](#Автоматическое-внедрение)
- [Помощник](#Помощник)

<a name="Вступление"></a>
## Вступление

В MyUCP используеться контейнер для сохранения всех экземпляров классов для дальнейшего их использования.
Большая часть основных классов и модулей доступна уже в контейнер.

Например для получения текущего запроса можно использовать контейнер, если вы его используете в контроллере,
то контроллер уже расширен контейнером, и вы можете обращаться к методам контейнера из контроллера.

```php
<?php
/*
* MyUCP
*/

class HomeController extends Controller
{
	public function welcome()
    {
		$request = $this->make('request');
	}
}
```

Стоить понимать что контейнер создан для хранения единичного экземлпяра класса, вы не сможете добавить новый объект для того же класса,
при попытке это сделать он будет просто заменен.

<a name="Применение"></a>
## Применение

Для использования контейнера вы можете обратиться к функции-помощнику <a href="/docs/5.7/helpers#app">`app()`</a>,
она возвращает экземпляр класса `Application`, который и есть контейнером.

К контейнеру вы можете обращаться так же как к массиву для получения данных, при этом будет использован метод `make()`.

<a name="Метод-Make"></a>
### Метод Make

У контейнера существует метод `make()`, который позволяет обращаться к уже реализованным классам.

К примеру нам нужно получить объект класса `Request`, для этого вы можете просто вызвать метод `make()` и указать
в качестве аргумента класс `Request`:

```php
$request = app()->make(Request::class);
$request->getRoute();
```

Но если вы хотите обратиться к класу экземпляра которого нет в контейнер, он будет создан, или вы можете передать реализацию
вторым аргументом:

```php
app()->make(MyClass::class);
app()->make(MyInterface::class, new MyClass());
```

Если ваш класс принимает какие-то данные в конструктор, вы можете использовать метод `makeWith()`, который создает экземпляр класса
с переданными параметрами:

```php
app()->makeWith(MyClass:class, ['User']);
```

Теперь при создании экземпляра контейнер передаст значение `User` первым аргументом.

<a name="Алиасы"></a>
### Алиасы

В контейнере так же присутсвует система алиасов, если осмотреть код вы можете заметить что в некоторых местах используеться
сокращение класса, например класс `Request` вызываеться как `request`.

Это связано с тем что для стандартных классов фреймворка уже предусмотрены некоторые алиасы и установлены изначально.

Ниже приведен список стандартных алиасов:

```php
"config"                =>  Config::class,
"handleException"       =>  HandleExceptions::class,
"db"                    =>  DB::class,
"session"               =>  Session::class,
"request"               =>  Request::class,
"response"              =>  Response::class,
"csrftoken"             =>  CsrfToken::class,
"load"                  =>  Load::class,
"lang"                  =>  Translator::class,
"view"                  =>  View::class,
"router"                =>  Router::class,
"url"                   =>  UrlGenerator::class,
"dotenv"                =>  \MyUCP\Dotenv\Dotenv::class,
"extension"             =>  \MyUCP\Extension\Extension::class,
```

Для создания своего алиаса вы можете использовать метод `alias()`:

```php
app()->alias('myRequest', Request::class);
```

Теперь обращаясь к `myRequest` вы будете получать экземпляр класса `Request`. Но при этом старый алиас `request` будет так же доступен.

Метод `alias()` работает так же как и метод `make()`, только он принимает на один аргумент больше, первым аргументом идет алиас,
остальные аргументы такие же как и у метода `make()`.

Так же существует метод `aliasWith()`, который работает так же как метод `makeWith()`, но первым аргументом принимает алиас:

```php
app()->alias('my', MyInterface::class, new MyClass());
app()->aliasWith('my', MyClass::class, ['User'];
```

<a name="Проверка-на-существование"></a>
### Проверка на существование

Для того что бы проверить существует ли экземпляр уже в контейнер по указанному названию вы можете использовать метод `has()`,
он так же проверить и существование алиаса:

```php
app()->has('request'); // True
app()->has(Request::class); // True
app()->has(MyClass::class); // False
```

Стоит учесть что данный метод не создает экземпляр класса если тот не создан в контейнере, он возвращает только логическое значение.

<a name="Получение-путей"></a>
## Получение путей

В контейнер так же существуют некоторые методы помощники, которые помогут вам сформировать путь к нужной вам директории.

Если раньше для получения полного пути к нужной директории вам нужно было использовать константы (`APP_DIR`, `VIEWS_DIR`...),
то сейчас вы можете использовать методы для формирования полного пути.

```php
app()->appPath('routes.php');                   // .../app/routes.php
app()->enginePath('protected/Load.php');        // .../engine/protected/Load.php
app()->resourcesPath('lang/ru/example.php');    // .../resources/lang/ru/example.php
app()->viewsPath('welcome.php');                // .../resources/views/welcome.php
app()->assetsPath('image.png');                 // .../assets/image.png
app()->configPath('main.php');                  // .../configs/main.php
```

<a name="Автоматическое-внедрение"></a>
## Автоматическое внедрение

При работе с контроллерами вы получаете возможность в аргументах метода который будет вызван через маршруты,
указать тип класса что бы контейнер смог его создать и передать, или передать уже существующий.

Например при использовании метода вы хотите получить сразу доступ к текущему запросу, вы можете это сделать просто:

```php
<?php
/*
* MyUCP
*/

class HomeController extends Controller
{
	public function welcome(Request $request)
    {
		return $request->getUri();
	}
}
```

<a name="Помощник"></a>
## Помощник

Для простоты работы с контейнером вы можете использовать функцию помощник <a href="/docs/5.7/helpers#app">`app()`</a>,
она используеться для получения экземпляра из контейнера и работает как метод `make()`, но принимает только один аргумент с именем
экземпляра, или это может быть алиас. Но так же если найденный экземпляр не будет найден в контейнере,
он будет создан.

```php
app('request');         // app()->make('request');
app(MyClass::class);    // app()->make(MyClass::class);
```
