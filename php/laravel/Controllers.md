
# ROUTING

* route files are located in `routes` directory and are automatically loaded by the framework.
    * `routes/web.php` define web interface routes (with `web` middleware group which provides session state, CSRF, etc).
    * `routes/api.php` are stateless (`api` middleware group, `/api` URI prefix is automatically applied, can be modified in `RouteServiceProvider` class)
* you can define routes resolving using closures or a controller's method and with using parameters (and optionally regex to constrain them)
* you can name a route in order to reference it by name, and you can use groups to apply settings to many routes at once

```php
// Resolve route using closure
Route::get('/', function() {...});

// Resolve route using controller@method
Route::get('/', 'WelcomeController@index'); 

// Parameters
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
Route::get('user/{name?}', function ($name = 'John') {
    return $name;
});
Route::get('user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
Route::get('search/{search}', function ($search) {
    return $search;
})->where('search', '.*'); // allows search parameter to cointain '/'

// Http verbs
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
Route::match(['get', 'post'], '/', function () { });
Route::any('/', function () { });

// Redirect shortcut
Route::redirect('/here', '/there'); // 302
Route::redirect('/here', '/there', 301); // 301
Route::permanentRedirect('/here', '/there'); // 301

// View only shortcut
Route::view('/welcome', 'welcome');
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

// Fallback (404)
Route::fallback(function () {
    // 404 (should be the last route registered)
});

// Route naming
Route::get('user/profile', 'UserProfileController@show')->name('profile');
$url = route('profile'); // Generating URLs...
return redirect()->route('profile'); // Generating Redirects...
$url = route('profile', ['id' => 1, 'photos' => 'yes']);
// Additional parameters will result in a query string: /user/1/profile?photos=yes
if ($request->route()->named('profile')) {} // check the route name from a route middleware 'handle' function

// Get route info
$route = Route::current();
$name = Route::currentRouteName();
$action = Route::currentRouteAction();
```

# MIDDLEWARE

* you can envision middleware as a series of "layers" HTTP requests must pass through before they hit your application. Each layer can examine the request and even reject it entirely.
* middleware classes are in `app/Http/Middleware` directory and Laravel provides several ones by default (e.g. `web`, `api`, `auth`, etc). 
* `artisan make:middleware CheckAge` will create a middleware class with scaffold code: in `handle` function you can use `$request` object and `$next` closure (which refers to the subsequent layers of the http request).

```php
namespace App\Http\Middleware;
use Closure;

class CheckAge
{
    public function handle($request, Closure $next)
    {
        if ($request->age <= 200) {
            return redirect('home');
        }
        return $next($request);
    }
}
```

`app/Http/Kernel.php` is the default class for middleware setup: you can register your middleware classes (e.g. `\App\Http\Middleware\CheckAge::class`) here. 

    * adding it to the `$middleware` array will make the middleware applied to all http requests
    * adding a `'name' => \App\Http\Middleware\CheckAge::class` in the `$routeMiddleware` associative array will register a named middleware which you can apply in route definitions or in controller classes (or methods):

```php
class UserController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');
        $this->middleware('log')->only('index');
        $this->middleware('subscribed')->except('store');
        $this->middleware(function ($request, $next) {
            // ...
            return $next($request);
        });
    }
}
```

    * inside a controller class constructor, for the entire class or for some of its methods with `$this->middleware('auth');` or `$this->middleware('log')->only('index');`, or `$this->middleware('subscribed')->except('store');`, or with a closure: `$this->middleware(function ($request, $next) { ...;  return $next($request); });`
* you can define middleware groups to add them all at once by specifying the group name instead of the single middleware name, by registering the group in the `$middlewareGroups` property 
* you can assing an order to the middlegroup with the `$middlewarePriority` array
* you can add middleware parameters by adding additional arguments to the `handle` function, after the `$next` argument, and then you can pass those parameter during the middleware registration in a route, by separating middleware name and parameters by a comma, e.g. `Route::put('post/{id}', function ($id) {})->middleware('role:editor');`
* the `terminate` method of the middleware class allows you to perform additional tasks after the response is sent to the browser

```bash
artisan make:middleware CheckAge
```

```php
namespace App\Http\Middleware;
use Closure;

class CheckAge
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->age <= 200) {
            return redirect('home');
        }
        return $next($request); // pass the request deeper into the application
    }
}
```

# CONTROLLERS

* Instead of defining all of your request handling logic as Closures in route files, you may wish to organize this behavior using Controller classes. Controllers can group related request handling logic into a single class
* Controllers are in `app/Http/Controllers` 
* You can create a controller with `artisan make:controller UserController` will create a controller: since it extdends from the base controller class you have methods such as `middleware`, `validate`, `dispatch`, etc. Then you associate a route to a controller with `Route::get('user/{id}', 'UserController@show');`. The route parameters will also be passed to the method.
* you can assign middleware in controller's constructor:

```php
public function __construct()
{
    $this->middleware('auth');
    $this->middleware('log')->only('index');
    $this->middleware('subscribed')->except('store');
    $this->middleware(function ($request, $next) {
	// you can use a closure, without defining an entire middleware class
	return $next($request);
    });
}
```

* if you type-hint a controller constructor or method's arguments the Service Container will resolve it automatically (You may also type-hint any Laravel contract. If the container can resolve it, you can type-hint it)

## Laravel "resources"

* Laravel provides a handy 'Resource' code scaffolding for REST-API CRUD operations. 
    * `php artisan make:controller PhotoController --resource` - will generate a controller with all CRUD methods stubbed, `Route::resource('photos', 'PhotoController');` creates multiple routes to handle actions on the resource. The generated controller will already have methods stubbed for each of these actions
    * you can even bind a model to the resource 
* [see more](https://laravel.com/docs/6.x/controllers#resource-controllers)

