# Routing

## Routing folder and default files

* All Laravel routes are defined in your route files, which are located in the __`routes` directory__. These files are automatically loaded by the framework. 
* `routes/web.php`
  * defines routes that are for your web interface
  * they are assigned the __web middleware group__, which provides features like __session state and CSRF protection__. 
* `routes/api.php` 
  * defines routes for apis
  * they are __stateless__ and are assigned the __api middleware group__.
  * within this group, __the `/api` URI prefix is automatically applied__. You may modify the prefix and other route group options by modifying your `RouteServiceProvider` class.

## Routing API

### Closure or controller

```php
Route::get('/', function() {...}); // closure
Route::get('/', 'WelcomeController@index'); // controller@method
```

### Http-method specific

```php
// a method for each http verb
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);

// multiple http verbs
Route::match(['get', 'post'], '/', function () {});

// any http verb
Route::any('/', function () {});
```

### Shortcuts

```php
// redirect (shortcut, no controller)
Route::redirect('/here', '/there'); // 302
Route::permanentRedirect('/here', '/there'); // 301
Route::redirect('/here', '/there', 301); // custom

// view (shortcut, no controller)
Route::view('/welcome', 'welcome');
Route::view('/welcome', 'welcome', [
  'name' => 'Taylor'
]);
```

### Parameters

* `{varName}` - only letters, numbers and `_` (NOT `-`)
* Parameters are injected into the callback based on their order in the url
* `/` - obviously cannot be part of the parameter: but you can change that using a `where` condition
```php
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    // use $postId and $commentId
});

// allow 'search' to contain every character, even "/"
Route::get('search/{search}', function ($search) {...})->where('search', '.*');
```

### Optional parameters

* `?` after the parameter name makes it optional, but make sure to give the route's corresponding variable a default value

```php
Route::get('user/{name?}', function ($name = null) {
    // use $name
});
```

### Regular expression constraints

```php
Route::get('user/{name}', function ($name) {
    // $name must be one or more letters
})->where('name', '[A-Za-z]+');

Route::get('user/{id}', function ($id) {
    // $id must be one or more digits
})->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
    // multiple constraints can be passed
    // as associative array
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

### Global constraints

```php
// RouteServiceProvider
public function boot()
{
    Route::pattern('id', '[0-9]+');
    parent::boot();
}

// routes/web.php
Route::get('user/{id}', function ($id) {
    // Only executed if {id} is numeric...
});
```

### Naming routes

* You can give a name to a route in order to allow the convenient generation of URLs or redirects for specific routes
* Names must be unique
* Name convention is `articles.show` (similar approach as we use for views)
  * accessible with `routes('articles.show', $article->id)`
    * instead of `$article->id` you can pass `$article` and laravel will infer the key name (similar as in route-model binding)
    * best practice is also to insert in the associated model a method to get the path to that unique instance, e.g. `public function path(){ return route(articles.show, $this); }`

```php
// without parameters:
Route::get('user/profile', function () { ... })->name('profile'); // or:
Route::get('user/profile', 'UserProfileController@show')->name('profile');

// Generating URLs...
$url = route('profile');

// Generating Redirects...
return redirect()->route('profile');

// with parameters:
Route::get('user/{id}/profile', function ($id) {})->name('profile');
// additional parameters will be passed as query string:
$url = route('profile', ['id' => 1, 'photos' => 'yes']);
// /user/1/profile?photos=yes

// check the name of a route
if ($request->route()->named('profile')) {...}
```

## CSRF Protection

* Any HTML forms pointing to POST, PUT, PATCH, or DELETE routes that are defined in the web routes file __should include a CSRF token field__. Otherwise, the request will be rejected. See more in the [CSRF documentation](https://laravel.com/docs/7.x/csrf)

```html
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```