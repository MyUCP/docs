# Конструктор запросов

- [Введение](#Введение)
- [Получение результатов](#Получение-результатов)
    - [Получение всех записей таблицы](#Получение-всех-записей-таблицы)
    - [Получение одной строки/столбца из таблицы](#Получение-одной-строки/столбца-из-таблицы)
    - [Агрегатные функции](#Агрегатные-функции)
    - [Выборка (SELECT)](#выборка)
        - [Указание столбцов для выборки](#Указание-столбцов-для-выборки)
        - [Сырые выражения](#Сырые-выражения)
        - [Объединения (JOIN)](#Объединения-JOIN)
            - [Объединение INNER JOIN](#Объединение-INNER-JOIN)
            - [Объединение LEFT JOIN](#Объединение-LEFT-JOIN)
    - [Условия WHERE](#Условия-WHERE)
        - [Простые условия WHERE](#Простые-условия-WHERE)
            - [Условия ИЛИ](#Условия-ИЛИ)
            - [Дополнительные условия WHERE](#Дополнительные-условия-WHERE)
                - [В интервале](#В-интервале)
                - [Вне интервала](#Вне-интервала)
                - [Фильтрация по совпадению с массивом значений](#Фильтрация-по-совпадению-с-массивом-значений)
                - [Поиск неустановленных значений (NULL)](#Поиск-неустановленных-значений-NULL)
    - [Упорядочивание, группировка, предел и смещение](#Упорядочивание-группировка-предел-и-смещение)
        - [Сортировка](#Сортировка)
		- [Лимит](#Лимит)
- [Вставка (INSERT)](#Вставка)
- [Обновление (UPDATE)](#Обновление)
- [Удаление (DELETE)](#Удаление)


<a name="Введение"></a>
## Введение
Конструктор запросов предоставляет удобный, выразительный интерфейс для создания и выполнения запросов к базе данных. Он может использоваться для выполнения большинства типов операций.

Каждый запрос представлен как отдельный объект класса `Query`, для создания запроса стоит использовать методы конструктора: `query($table = null)`, `table($table)`, `insert($table, array $data)`. 

<a name="Получение-результатов"></a>
## Получение результатов


<a name="Получение-всех-записей-таблицы"></a>
### Получение всех записей таблицы

Для начала создания запроса используйте метод `table()` фасада **Builder**. Метод `table()` возвращает экземпляр конструктора запросов для данной таблицы, позволяя вам «прицепить» к запросу дополнительные условия и в итоге получить результат. В данном примере давайте просто получим `get()` все записи из таблицы:

```php <?php

class HomeController extends Controller
{
    public function index()
    {
        $users = Builder::table('users')->get();
        
        return view('user/index', compact('users'));
    }
}
```

Результат запроса будет возвращен в виде коллекции `DBCollection`. Вы можете получить значение каждого столбца::

```php
foreach ($users as $user) {
    echo $user['name'];
}
```

<a name="Получение-одной-строки/столбца-из-таблицы"></a>
### Получение одной строки/столбца из таблицы

Если вам необходимо получить только одну строку из таблицы БД, используйте метод `first()`.

```php
$user = Builder::table('users')->where('name', 'John')->first();
```
```php
echo $user['name'];
```

Если вам не нужна вся строка, вы можете извлечь одно значение из записи методом `value()`. Этот метод вернёт значение конкретного столбца:

```php
$email = Builder::table('users')->where('name', 'John')->value('email');
```

<a name="Агрегатные-функции"></a>
### Агрегатные функции

Конструктор запросов содержит множество агрегатных методов, таких как `count`, `max`, `min`, `avg` и `sum`. Вы можете вызывать их после создания своего запроса:

```php
$users = Builder::table('users')->count();
$price = Builder::table('orders')->max('price');
```

Разумеется, вы можете комбинировать эти методы с другими условиями для создания вашего запроса:

```php 
$price = Builder::table('orders')
                ->where('finalized', 1)
                ->avg('price');
```
<a name="выборка"></a>
## Выборка (SELECT)

<a name="Указание-столбцов-для-выборки"></a>
### Указание столбцов для выборки

Само собой, не всегда вам необходимо выбрать все столбцы из таблицы БД. Используя метод `select()` вы можете указать необходимые столбцы для запроса:

```php 
$users = Builder::table('users')->select('name', 'email as user_email')->get();
```

Если у вас уже есть экземпляр конструктора запросов и вы хотите добавить столбец к существующему набору для выборки, используйте метод addSelect():

```php 
$query = Builder::table('users')->select('name');
$users = $query->addSelect('age')->get();
```

<a name="Сырые-выражения"></a>
### Сырые выражения

Иногда вам может понадобиться использовать уже готовое SQL-выражение в вашем запросе. Такие выражения вставляются в запрос напрямую в виде строк, поэтому будьте внимательны и не допускайте возможностей для SQL-инъекций! Для создания сырого выражения используйте метод `DB::raw()`:

```php 
$users = Builder::table('users')
                ->select(DB::raw('count(*) as user_count, status'))
                ->where('status', '<>', 1)
                ->groupBy('status')
                ->get();
```

<a name="Объединения-JOIN"></a>
## Объединения (JOIN)

<a name="Объединение-INNER-JOIN"></a>
### Объединение INNER JOIN

Конструктор запросов может быть использован для объединения данных из нескольких таблиц через JOIN. Для выполнения обычного SQL-объединения «inner join», используйте метод `join()` на экземпляре конструктора запросов. Первый аргумент метода `join()` — имя таблицы, к которой необходимо присоединить другие, а остальные аргументы указывают условия для присоединения столбцов. Как видите, вы можете объединять несколько таблиц одним запросом:

```php 
$users = Builder::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();
```

<a name="Объединение-LEFT-JOIN"></a>
### Объединение LEFT JOIN

Для выполнения объединения «left join» вместо «inner join», используйте метод `leftJoin()`. Этот метод имеет ту же сигнатуру, что и метод `join()`:

```php 
$users = Builder::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();
```

> Так же доступны методы `rightJoin()` и `crossJoin()`

<a name="Условия-WHERE"></a>
## Условия WHERE

<a name="Простые-условия-WHERE"></a>
### Простые условия WHERE

Для добавления в запрос условий where используйте метод `where()` на экземпляре конструктора запросов. Самый простой вызов `where()` требует три аргумента. Первый — имя столбца. Второй — оператор (любой из поддерживаемых базой данных). Третий — значение для сравнения со столбцом.

Например, вот запрос, проверяющий равенство значения столбца «votes» и 100:

```php 
$users = Builder::table('users')->where('votes', '=', 100)->get();
```

Для удобства, если вам необходимо просто проверить равенство значения столбца и данного значения, вы можете передать значение сразу вторым аргументом метода `where()`:

```php
$users = Builder::table('users')->where('votes', 100)->get();
```

Разумеется, вы можете использовать различные другие операторы при написании условия where:

```php 
$users = Builder::table('users')
                ->where('votes', '>=', 100)
                ->get();

$users = Builder::table('users')
                ->where('votes', '<>', 100)
                ->get();

$users = Builder::table('users')
                ->where('name', 'like', 'T%')
                ->get();
```

<a name="Условия-ИЛИ"></a>
### Условия ИЛИ

Вы можете сцепить вместе условия where, а также условия or в запросе. Метод `orWhere()` принимает те же аргументы, что и метод `where()`:

```php 
$users = Builder::table('users')
                ->where('votes', '>', 100)
                ->orWhere('name', 'John')
                ->get();
```

<a name="Дополнительные-условия-WHERE"></a>
### Дополнительные условия WHERE

<a name="В-интервале"></a>
#### В интервале

Метод `whereBetween()` проверяет, что значения столбца находится в указанном интервал:

```php 
$users = Builder::table('users')
                ->whereBetween('votes', [1, 100])->get();
```

<a name="Вне-интервала"></a>
#### Вне интервала

Метод `whereNotBetween()` проверяет, что значения столбца находится вне указанного интервала:

```php 
$users = Builder::table('users')
                ->whereNotBetween('votes', [1, 100])
                ->get();
```

<a name="Фильтрация-по-совпадению-с-массивом-значений"></a>
#### Фильтрация по совпадению с массивом значений

Метод `whereIn()` проверяет, что значения столбца содержатся в данном массиве:

```php 
$users = Builder::table('users')
                ->whereIn('id', [1, 2, 3])
                ->get();
```

Метод `whereNotIn()` проверяет, что значения столбца не содержатся в данном массиве:

```php 
$users = Builder::table('users')
                ->whereNotIn('id', [1, 2, 3])
                ->get();
```

<a name="Поиск-неустановленных-значений-NULL"></a>
#### Поиск неустановленных значений (NULL)

Метод `whereNull()` проверяет, что значения столбца равны **NULL**:

```php 
$users = Builder::table('users')
                ->whereNull('updated_at')
                ->get();
```

Метод `whereNotNull()` проверяет, что значения столбца не равны **NULL**:

```php 
$users = Builder::table('users')
                ->whereNotNull('updated_at')
                ->get();
```

<a name="Упорядочивание-группировка-предел-и-смещение"></a>
## Упорядочивание, группировка, предел и смещение

<a name="Сортировка"></a>
### Сортировка

Метод `orderBy($column)` позволяет вам отсортировать результат запроса по заданному столбцу. Первый аргумент метода `orderBy()` — столбец для сортировки по нему, а второй — задаёт направление сортировки и может быть либо asc, либо desc:

```php 
$users = Builder::table('users')
                ->orderBy('name', 'desc')
                ->get();
```

По умолчанию вторым аргументом указанно значение `asc`.

Вы так же можете использовать метод `orderByDesc($column)`, для задавания сортировки по столбцу направления, или метод `latest($column)`.

Для задавания направления `asc` без указания аргумента вы можете использовать метод `oldest($column)`

```php
Builder::table('news')
        ->latest('created_at')
        ->get();
```

<a name="Лимит"></a>
### Лимит

Для ограничения числа возвращаемых результатов из запроса или для пропуска заданного числа результатов в запросе (OFFSET) используется метод `limit()`:

```php 
$users = Builder::table('users')->limit(5, 10)->get();
```

Если вы укажете только первый параметр то будет выбрано точное количество записей, или вы можете использовать метод `take($limit)`.

Для указания количества записей которые необходимо пропустить (OFFSET) вы можете использовать метод `skip($offset)` или метод `offset($offset)`.

```php
$query->limit(5, 10);
$query->take(10)->skip(5);
$query->limit(10)->offset(5);
```

<a name="Вставка"></a>
## Вставка (INSERT)

Конструктор запросов предоставляет метод `insert()` для вставки записей в таблицу БД. Метод `insert()` принимает массив имён столбцов и значения для вставки:

```php 
Builder::table('users')->insert(
  ['email' => 'john@example.com', 'votes' => 0]
);
```

Вы можете вставить в таблицу сразу несколько записей одним вызовом `insert()`, передав ему массив массивов, каждый из которых — строка для вставки в таблицу:

```php 
Builder::table('users')->insert([
  ['email' => 'taylor@example.com', 'votes' => 0],
  ['email' => 'dayle@example.com', 'votes' => 0]
]);
```

> Так же существует идентичный метод `create()`. Вы можете его использовать при работе с моделями, потому что там метод `insert()` не доступен 

<a name="Обновление"></a>
## Обновление (UPDATE)

Разумеется, кроме вставки записей в БД конструктор запросов может и изменять существующие строки с помощью метода `update(array $data)`. Вы так же можете подготовить данные для изменения при помощи метода `set(array $data)`.

```php 
$query = Builder::table('users');
//
$query->set(['votes' => 1]);
//
$query->update();
```

Вы так же можете использовать условия `where` для выполнения изменений:

```php
$query->where('user_id', 10);
```

<a name="Increment"></a>
Increment

Если вы хотите добавить значений к вашему полю, вы можете использовать метод `increment($column, $amount = 1)`.

Если вы укажете просто название поля то его значение будет увеличено на 1, или вторым аргументом вы можете сами указать необходимое значение.

```php 
Builder::table('users')->increment(`votes`);
Builder::table('users')->increment(`votes`, 5);
```

<a name="Decrement"></a>
Decrement

Если вы хотите отнять значений к вашему полю, вы можете использовать метод `decrement($column, $amount = 1)`.

Если вы укажете просто название поля то его значение будет уменьшено на 1, или вторым аргументом вы можете сами указать необходимое значение.

```php 
Builder::table('users')->decrement(`votes`);
Builder::table('users')->decrement(`votes`, 5);
```

<a name="Удаление"></a>
## Удаление (DELETE)

Конструктор запросов предоставляет метод `delete()` для удаления записей из таблиц:

```php 
Builder::table('users')->delete();
```

Вы можете ограничить оператор `delete()`, добавив условие `where()` перед его вызовом:

```php 
Builder::table('users')->where('votes', '<', 100)->delete();
```

Если вы хотите очистить таблицу (усечение), удалив все строки и обнулив счётчик ID, используйте метод `truncate()`:

```php 
Builder::table('users')->truncate();
```

> Усечение таблицы аналогично удалению всех её записей, а также сбросом счётчика autoincrement-полей.
