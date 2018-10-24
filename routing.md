# Маршрутизация

- [Введение](#Введение)
- [Правила маршрутизации](#Правила-маршрутизации)
- [Создание правила указывая тип](#Создание-правила-указывая-тип)
- [Дополнительные методы](#Дополнительные-методы)
- [Группы роутов](#Группы-роутов)
- [Привязка модели](#Привязка-модели)
- [Алиасы роутов](#Алиасы-роутов)
- [Функция route()](#Функция-route)

<a name="Введение"></a>
## Введение

Начиная с версии **4.1.0** MyUCP использует систему маршрутизации, она заменяет обычную систему адресов, где часть адреса указывала на директорию и название файла контроллера. Теперь вы можете сами указывать контроллер и метод который должен обработать нужный вам шаг пользователя.
Все маршруты в MyUCP находяться в `app/routers.php`.

<a name="Правила маршрутизации"></a>
## Правила маршрутизации

В новой версии MyUCP было упрощенно создание правил маршрутизации, для этого были созданы новые методы: `any()`, `get()`, `post()`, `name()`, `uses()`.

При помощи метода `any()` вы можете создать правило которое не будет разделено на тип запроса **GET** или **POST**:

```php
Router::any("/", "HomeController@welcome"); // Вызовет метод welcome контроллера HomeController при переходе на главную страницу
```

Так же, начиная с версии **5.5** вы можете указать `callback` функцию которая будет вызвана при переходе на адрес правила. `Callback` функция может быть указан вторым или третьим параметром, но в случае если она указана, то вызов метода контроллера уже не производится.
```php
Router::get("/", function() {
	return view("welcome");
});
```

<a name="Создание правила указывая тип"></a>
## Создание правила указывая тип

Вы можете разделить запросы которые будут приходить на ваш сайт на типы **GET** и **POST** тем самым это упростит работу например при использовании ajax, вам не нужно будет проводить лишние проверки на тип запроса.

Для создания нового правила в файле `routers.php` необходимо с новой строки прописать новый метод при помощи фасада `Router`:

```php
Router::get("/", "HomeController@welcome"); // Вызовет метод welcome контроллера HomeController при обычном GET запросе
Router::post("/", "HomeController@welcomePost");  // Вызовет метод welcomePost контроллера HomeController при обычном POST запросе
```

Как мы можем заметить метод `get()` как и метод `post()` принимают два параметра, первый параметр это сам URL, второй параметр вы можете указать как строку передав контроллер и метод которые вы хотите вызвать при переходе на данный адрес. 

Если указать второй параметр как массив то он должен иметь два значения: <em>as</em> и <em>uses</em>, первый из которых придаст имя маршруту, второй укажет какой метод вы хотите вызвать при переходе на указанный вами адрес в первом параметре.

```php
Router::get("/", ["as" => "home", "uses" => "HomeController@welcome"]);
```

<a name="Дополнительные методы"></a>
## Дополнительные методы

В примере выше было указано как вы можете указать сразу все необходимые параметры для правила маршрутизации, вы так же это можете сделать при помощи методов `name()` и `uses()`.

Метод `name()` укажет имя для выбранного вами правила маршрутизации при помощи которого в дальнейшем вы сможете обращаться к нему:

```php
Router::get("/", "HomeController@welcome")->name("home");
```

Метод `uses()` укажет контроллер и метод (разделенный знаком @) которые вы хотите вызвать при обращении по указанному вами адресу:

```php
Router::get("/")->uses("HomeController@welcome");
```

<a name="Группы роутов"></a>
## Группы роутов

Иногда вам может быть нужно применить фильтры к набору маршрутов. Вместо того, чтобы указывать их для каждого маршрута в отдельности вы можете сгруппировать маршруты:

```php
Router::group(['session' => ['user_id', '!empty']], function()
{
    Router::get('/', function()
    {
        // К этому маршруту будет привязан фильтр session.
    });

    Router::get('user/profile', function()
    {
        // К этому маршруту также будет привязан фильтр session.
    });
});
```

### Поддоменные роуты
Вы так же можете работать с разными доменами или поддоменами. Вы можете как указать сам домен для провреки, так и указать какой то параметр, в итоге он будет доступен в вашем роутере в качестве аргумента для метода или для callback функции:

```php
Router::group(['domain' => '{account:[a-zA-Z]+}.myapp.com'], function()
{

    Router::get('user/{id}', function($account, $id)
    {
        //
    });

});
```

### Префикс пути
Группа роутов может быть зарегистрирована с одним префиксом без его явного указания с помощью ключа `prefix` в параметрах группы.

```php
Router::group(['prefix' => 'admin'], function()
{

    Router::get('user', function()
    {
        //
    });

});
```

<a name="Привязка модели"></a>
## Привязка модели

Для каждого роута вы можете указать модели которые потом будете использовать как обычно, вместо того что объявлять их уже в методе контроллера. Метод привязки похож с функцией-помощником `<a href="/docs/5.5/helpers#model()">model()</a>`

```php
Router::any("user/profile", "UserController@profile")->models("User");
```

<a name="Алиасы роутов"></a>
## Алиасы роутов

В MyUCP реализована система алиасов для роутов, что бы уменьшить количество кода, вы можете указать методом `alias()` еще один URL который будет действителен по данному роуту:

```php
Router::any("user/profile/{id:[0-9]+}", "UserController@profile")->alias("profile/{id:[0-9]+}");
```

<a name="Функция route()"></a>
## Функция route()

Данная функция будет полезна при составлении URL адресов на разные страницы и получение некоторых данных при разработке вашего проекта. Если не передавать аргумент в функцию, то она вернет экземпляр класса `RouteHelper` с параметрами текущего роута.

Функция возвращает объект данных состоящих из:

```php
route()->http_method //Метод роута (GET, POST)
route()->name //Название роута которое вы задали при его создании, если оно задано не было, оно будет сгенерировано автоматически
route()->url //Текущий URL
route()->rule //Правило роута (адрес)
route()->parameters //Массив с параметрами (разобранного правила) по адресу, если такие есть
route()->callback //Функция которая будет вызвана в случае если она была указана при создании роута
route()->type //Тип роута (callback или controller)
route()->controller //Контроллер который будет вызван
route()->method //Метод который будет вызван из контроллера указанного выше
route()->models //Модели которые привязаны к роуту
```

Так же вы можете вызвать метод `redirect()` и совершить переадресацию на указаный роут:

```php
route("profile")->redirect(); //Если у роута нет параметров
route("profile")->redirect(["id" => 1]); //Если у роута указан параметр id: {id:[0-9]+}
```

> Вы так же можете использовать функцию-помощник <a href="/responses#Редиректы">redirect()</a> которая выполняет подобные функции.