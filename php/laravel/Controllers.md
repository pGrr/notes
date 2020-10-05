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

// ...using any of the Http verbs
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
Route::match(['get', 'post'], '/', function () { });
Route::any('/', function () { });
```

## Get route info

```php
// Use the Route facade 
$route = Route::current();
$name = Route::currentRouteName();
$action = Route::currentRouteAction();

// $request->route() in a route middleware 'handle' function
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }
    return $next($request);
}
```

## Redirects

```php
Route::redirect('/here', '/there'); // 302
Route::redirect('/here', '/there', 301); // 301
Route::permanentRedirect('/here', '/there'); // 301
```

## View-only routes

```php
// View only shortcut
Route::view('/welcome', 'welcome');
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

## Fallback route (404)

* Typically, unhandled requests will automatically render a "404" page via your application's exception handler. However, since you may define the fallback route within your routes/web.php file, all middleware in the web middleware group will apply to the route. You are free to add additional middleware to this route as needed:

```php
// in web routes file:
Route::fallback(function () {
    // 404 (should be the last route registered)
});
```

## Routes with parameters

```php
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) { // required
    //
});
Route::get('user/{name?}', function ($name = 'John') { // optional
    return $name;
});
```

### Parameters regex

```php
Route::get('user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']); // regex

Route::get('search/{search}', function ($search) {
    return $search;
})->where('search', '.*'); // allows search parameter to cointain '/'

// ...or in RouteServiceProvider:
public function boot()
{
    Route::pattern('id', '[0-9]+'); // globally and automatically applied
    parent::boot();
}
// ...and then:
Route::get('user/{id}', function ($id) {
    // Only executed if {id} is numeric...
});
```

## Route naming

```php
Route::get('user/{id}/profile', function ($id) { })->name('profile');
Route::get('user/profile', 'UserProfileController@show')->name('profile');
$url = route('profile'); // Generating URLs...
return redirect()->route('profile'); // Generating Redirects...
$url = route('profile', ['id' => 1, 'photos' => 'yes']); // passing parameters
// (Additional parameters will result in a query string: /user/1/profile?photos=yes)
```

## Route grouping (middlewares, namespaces, subdomains, etc)

```php
// Middleware
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second Middleware
    });

    Route::get('user/profile', function () {
        // Uses first & second Middleware
    });
});

// Namespaces
Route::namespace('Admin')->group(function () {
    // Controllers Within The "App\Http\Controllers\Admin" Namespace
});

// Subdomains
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});

// Prefix the URI of each route in the group
Route::prefix('admin')->group(function () {
    Route::get('users', function () {
        // Matches The "/admin/users" URL
    });
});

// Prefix the name of each route in the group
Route::name('admin.')->group(function () {
    Route::get('users', function () {
        // Route assigned name "admin.users"...
    })->name('users');
});
```

## Route - model binding

* If you type-hint the route callback (or the controller constructor/method) by specifying an eloquent model class as parameter, Laravel will try to automatically fetch the model instance with an id equal to the related parameter of the URI. If it can find it, it will inject the instance to the route handler (i.e. the callback or the controller class/method), else it will generate a 404 response.

```php
// default route-model binding
Route::get('api/users/{user}', function (App\User $user) {
    return $user->email;
});

// to use a database column other than id when retrieving a given model class, you may override the getRouteKeyName method on the Eloquent model:
public function getRouteKeyName()
{
    return 'slug';
}

// to customize the resolution logic override the resolveRouteBinding method of your eloquent model:
public function resolveRouteBinding($value)
{
    return $this->where('name', $value)->firstOrFail();
}
```

## Routes rate limiting

* Laravel includes a middleware to rate limit access to routes within your application. 
* The `throttle` middleware accepts two parameters that determine the maximum number of requests that can be made in a given number of minutes. 

```php
// An authenticated user may access the route 60 times per minute
Route::middleware('auth:api', 'throttle:60,1')->group(function () {
    Route::get('/user', function () {
        // 
    });
});

// An authenticated user may access the route rate_limit times per minute,
// where rate_limit is a User model's attribute
Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});

// 10 requests per minute for guests and 60 for authenticated users:
Route::middleware('throttle:10|60,1')->group(function () {
    //
});
// 10 requests for guests and rate_limit for authenticated users:
// where rate_limit is a User model's attribute
Route::middleware('auth:api', 'throttle:10|rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});

// Different rate limits for different segments of the API:
Route::middleware('auth:api')->group(function () {
    Route::middleware('throttle:60,1,default')->group(function () {
        Route::get('/servers', function () {
            //
        });
    });
    Route::middleware('throttle:60,1,deletes')->group(function () {
        Route::delete('/servers/{id}', function () {
            //
        });
    });
});
```

# MIDDLEWARE

* you can envision middleware as a series of "layers" HTTP requests must pass through before they hit your application. Each layer can examine the request and even reject it entirely.
* middleware classes are in `app/Http/Middleware` directory and Laravel provides several ones by default (e.g. `web`, `api`, `auth`, etc). 

## Create a middleware

* `artisan make:middleware CheckAge` will create a middleware class with scaffold code: in `handle` function you can use `$request` object and `$next` closure (which refers to the subsequent layers of the http request).

```bash
artisan make:middleware CheckAge
```

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

## Register and apply middlewares

* `app/Http/Kernel.php` is the default class for middleware setup: you can register your middleware classes (e.g. `\App\Http\Middleware\CheckAge::class`) here. 
    * adding it to the `$middleware` array will make the middleware applied to all http requests
    * adding a `'name' => \App\Http\Middleware\CheckAge::class` in the `$routeMiddleware` associative array will register a named middleware which you can apply in route definitions or in controller classes (or methods)
    * adding a `'name' => [ \App\Http\Middleware\MyMiddleware1::class, ... ]` in the `$middlewareGroups` will register a named group of middlewares which you can apply all at once in route definitions or in controller classes (or methods)
    * you can assing an order to the non-global middlewares in the `$middlewarePriority` array

```php
// Register a middleware in a route:
Route::get('profile', 'UserController@show')->middleware('auth');
Route::get('/', function () {
    //
})->middleware('first', 'second'); 
Route::get('admin/profile', function () {
    //
})->middleware(CheckAge::class); // you may also pass the fully qualified class name
Route::get('/', function () {
    //
})->middleware('web'); // web refers a group of middleware
// assign the middlewares to a group of routes
Route::group(['middleware' => ['web']], function () { });
Route::middleware(['web', 'subscribed'])->group(function () { }); 

// Register a middleware in a controller (constructor or method):
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

## Middleware parameters

* you can add middleware parameters by adding additional arguments to the `handle` function, after the `$next` argument, and then you can pass those parameter during the middleware registration in a route:

```php
// you can assign additional parameters after $next
class CheckRole
{
    public function handle($request, Closure $next, $role) 
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }
        return $next($request);
    }
}

// ...then you can pass parameters during middleware registration with 'name:par1:par2:...' syntax
Route::put('post/{id}', function ($id) {
    //
})->middleware('role:editor');
```

## Add additional tasks after middleware (before response is sent to the browser)

* the `terminate` method of the middleware class allows you to perform additional tasks after the response is sent to the browser

```php
class StartSession
{
    public function handle($request, Closure $next)
    {
        return $next($request);
    }

    public function terminate($request, $response)
    {
        // Store the session data...
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

