---
layout: post
title: "REST API Laravel: Лучшие практики"
author: "retro3f"
categories: technologies
tags: [technologies, guide, rest, api, laravel, php]
date: 2020-11-25T16:40:00
 
image:
  feature: 2020-11-25-rest-api-laravel-luchshie-praktiki/laravel-rest-api-best-practices.jpg
---



В современном приложении API является одной из основных функций. Оно важно не только для создания мобильных и десктопных приложений, но и для веб-приложений.

Фронтенд и бэкенд разработка претерпела значительные изменения с приходом фреймворков типа Vue или React. Все новые приложения хотят быть [SPA](https://ru.wikipedia.org/wiki/%D0%9E%D0%B4%D0%BD%D0%BE%D1%81%D1%82%D1%80%D0%B0%D0%BD%D0%B8%D1%87%D0%BD%D0%BE%D0%B5_%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D0%B5){:target="_blank"}{:rel="noopener noreferrer"} (single page application — Одностраничное приложение). Вот почему реализация API на бэкэнде так важна.

По сути, API — это интерфейс, возвращающий данные в специальном формате, понятном любому приложению, будь то приложение для Android или для веба.

При разработке API вы должны принять во внимание некоторые рекомендации, которым следует другие разработчики. Я изучил множество уроков и отобрал несколько лучших практик, которым сам следую при написании приложениях на Laravel.

### Стандарты оформления кода

Это относится не только к разработке REST API, но и ко всем другим видам приложений. Каждый раз, когда я начинаю писать какое-либо приложение, то снова читаю стандарты оформления кода, несмотря на то, что я читал их раньше. Это заставляет меня следовать им. Некоторые из них связаны с разработкой REST API в Laravel.

### Используйте файл routes/api.php для маршрутов API
Laravel по умолчанию имеет отдельный файл ```routes/api.php```, отличающийся от обычного файла ```routes/web.php```. Мы должны хранить наши API маршруты в нём. Он имеет встроенный мидлвар (который можно увидеть в ```app/Http/Kernel```.php в переменной ```$middlewareGroups``` под ```api```) и префикс ```api```, так что все определенные маршруты будут доступны по ```/api```.

### Имена маршрутов с префиксом api
Мне нравится задавать групповой маршрут для всего API, чтобы я мог получить доступ к маршрутам по их имени с префиксом api.

```php
Route::get('/users', "API\V1\UserController@index")->name("users");
```

URL этого маршрута может быть получен через ```route("users")```, но возможен конфликт с ```web.php```.

```php
Route::group(['as' => 'api.'], function () {
  Route::get('/users', "API\V1\UserController@index")->name("users");
});
```

А сейчас он будет доступен через ```route('api.users')```.

Если вы используете имена маршрутов, если вы пишете тесты, то, при необходимости сменить URL — вам не придется менять его везде, а только в маршруте и без изменении его имени.

### Используйте множественное число для описания ресурсов
Когда вы делает ресурсный маршрут, то используйте множественное число.

```php 
Route::resource("/users", "API\V1\UserController")->name("users");
// mysite.com/api/users
```

### Управление версиями API
Помните, что API будет использоваться другими программами. Мы периодически обновляем код на сервере и это может нарушить работу клиентских приложений. Здесь пригодится API версионирование. Если ваш API не общедоступный, то вы можете оставить всё как есть. Приведенный выше URL при использовании версий будет выглядеть так:
``` mysite.com/api/v1/users ```

### Используйте Passport вместо JWT для аутентификации API
Мне лично нравится использовать [Passport](https://laravel.com/docs/passport){:target="_blank"}{:rel="noopener noreferrer"} для аутентификации. Это надежно и поддерживается разработчиками Laravel.

### Используйте преобразователь

Я всегда использую преобразователь, чтобы получить данные в одном формате. Хотя у Laravel есть собственный класс преобразователей, но лично мне нравится [league/fractal](https://github.com/thephpleague/fractal){:target="_blank"}{:rel="noopener noreferrer"}

### Коды ответа и обработка ошибок
Коды состояния HTTP упростят обратную связь с пользователями вашего API. Также вы должны вернуть серверу сообщение с ответом. Для обработки ошибок, преобразования и сохранения данных я написал контроллер, который отвечает за обработку ошибок.

### Лимитирование количества запросов за определенный промежуток времени с одного IP-адреса
Если API открыт для всего интернета, то это явная цель для спамеров, ботов и злоумышленников. Вы можете ограничить количество запросов в секунду, например, до 5. Если их больше, то это скорей всего автоматизированная программа, злоупотребляющая вашим API.

```php
namespace App\Http\Controllers\Api\V1;
use League\Fractal\Manager;
use League\Fractal\Resource\Item;
use App\Http\Controllers\Controller;
use League\Fractal\Resource\Collection;
use League\Fractal\Pagination\IlluminatePaginatorAdapter;
class ApiController extends Controller
{
    /**
     * @var int $statusCode
     */
    protected $statusCode = 200;
    const CODE_WRONG_ARGS = 'GEN-FUBARGS';
    const CODE_NOT_FOUND = 'GEN-LIKETHEWIND';
    const CODE_INTERNAL_ERROR = 'GEN-AAAGGH';
    const CODE_UNAUTHORIZED = 'GEN-MAYBGTFO';
    const CODE_FORBIDDEN = 'GEN-GTFO';
    const CODE_INVALID_MIME_TYPE = 'GEN-UMWUT';
    /**
     * @var Manager $fractal
     */
    protected $fractal;
    public function __construct()
    {
        $this->fractal = new Manager;
        if (isset($_GET['include'])) {
            $this->fractal->parseIncludes($_GET['include']);
        }
    }
    /**
     * Get the status code.
     *
     * @return int $statusCode
     */
    public function getStatusCode()
    {
        return $this->statusCode;
    }
    /**
     * Set the status code.
     *
     * @param $statusCode
     * @return $this
     */
    public function setStatusCode($statusCode)
    {
        $this->statusCode = $statusCode;
        return $this;
    }
    /**
     * Repond a no content response.
     * 
     * @return response
     */
    public function noContent()
    {
        return response()->json(null, 204);
    }
    /**
     * Respond the item data.
     *
     * @param $item
     * @param $callback
     * @return mixed
     */
    public function respondWithItem($item, $callback, $message = 'Successfully')
    {
        $resource = new Item($item, $callback);
        $data = $this->fractal->createData($resource)->toArray();
        $data['message'] = $message;
        return $this->respondWithArray($data);
    }
    /**
     * Respond the collection data.
     *
     * @param $collection
     * @param $callback
     * @return mixed
     */
    public function respondWithCollection($collection, $callback, $message = 'Successfully')
    {
        $resource = new Collection($collection, $callback);
        $data = $this->fractal->createData($resource)->toArray();
        $data['message'] = $message;
        return $this->respondWithArray($data);
    }
    /**
     *  Respond the collection data with pagination.
     *
     * @param $paginator
     * @param $callback
     * @return mixed
     */
    public function respondWithPaginator($paginator, $callback, $message = 'Successfully')
    {
        $resource = new Collection($paginator->getCollection(), $callback);
        $resource->setPaginator(new IlluminatePaginatorAdapter($paginator));
        $data = $this->fractal->createData($resource)->toArray();
        $data['message'] = $message;
        return $this->respondWithArray($data);
    }
    /**
     * Respond the data.
     *
     * @param array $array
     * @param array $headers
     * @return mixed
     */
    public function respondWithArray(array $array, array $headers = [])
    {
        return response()->json($array, $this->statusCode, $headers);
    }
    /**
     * Respond the message.
     * 
     * @param  string $message
     * @return json
     */
    public function respondWithMessage ($message) {
        return $this->setStatusCode(200)
            ->respondWithArray([
                    'message' => $message,
                ]);
    }
    /**
     * Respond the error message.
     * 
     * @param  string $message
     * @param  string $errorCode
     * @return json
     */
    protected function respondWithError($message, $errorCode, $errors = [])
    {
        if ($this->statusCode === 200) {
            trigger_error(
                "You better have a really good reason for erroring on a 200...",
                E_USER_WARNING
            );
        }
        return $this->respondWithArray([
            'errors'  => $errors,
            'code'    => $errorCode,
            'message' => $message,
        ]);
    }
    /**
     * Respond the error of 'Forbidden'
     * 
     * @param  string $message
     * @return json
     */
    public function errorForbidden($message = 'Forbidden', $errors = [])
    {
        return $this->setStatusCode(500)
                    ->respondWithError($message, self::CODE_FORBIDDEN, $errors);
    }
    /**
     * Respond the error of 'Internal Error'.
     * 
     * @param  string $message
     * @return json
     */
    public function errorInternalError($message = 'Internal Error', $errors = [])
    {
        return $this->setStatusCode(500)
                    ->respondWithError($message, self::CODE_INTERNAL_ERROR, $errors);
    }
    /**
     * Respond the error of 'Resource Not Found'
     * 
     * @param  string $message
     * @return json
     */
    public function errorNotFound($message = 'Resource Not Found', $errors = [])
    {
        return $this->setStatusCode(404)
                    ->respondWithError($message, self::CODE_NOT_FOUND, $errors);
    }
    /**
     * Respond the error of 'Unauthorized'.
     * 
     * @param  string $message
     * @return json
     */
    public function errorUnauthorized($message = 'Unauthorized', $errors = [])
    {
        return $this->setStatusCode(401)
                    ->respondWithError($message, self::CODE_UNAUTHORIZED, $errors);
    }
    /**
     * Respond the error of 'Wrong Arguments'.
     * 
     * @param  string $message
     * @return json
     */
    public function errorWrongArgs($message = 'Wrong Arguments', $errors = [])
    {
        return $this->setStatusCode(400)
                    ->respondWithError($message, self::CODE_WRONG_ARGS, $errors);
    }
}
```
### Безопасность
Самая важная часть — вы должны позаботиться о безопасности. Приложение Laravel легко защитить, но если вы не сделаете это правильно, то вас могут взломать. Вы можете использовать Laravel Passport — он входит в экосистему Laravel и поддерживает аутентификацию через App ID — App Secret.


