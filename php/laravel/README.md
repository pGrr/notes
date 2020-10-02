# ARTISAN (CLI tool)

* Artisan is Laravel's cli tool for managing a laravel application and to generate various scaffold code
* `artisan` or `php artisan` - see commands
* `artisan <COMMAND> -h` - see help for a command

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
    * e.g. `web`, `api`, `auth` are middlegroups defined by default by laravel
* middleware classes are in `app/Http/Middleware` directory
* `artisan make:middleware CheckAge` will create a middleware class with scaffold code: in `handle` function you can use `$request` object and `$next` closure (which refers to the subsequent layers of the http request).
* you can register a middleware in `app/Http/Kernel.php`:
    * globally by listing your middleware class, e.g.`\App\Http\Middleware\CheckAge::class`, in the `$middleware` property: this way the middleware will be applied to all http requests
    * for a single route (or to many) by registering a `'name' => \App\Http\Middleware\CheckAge::class` in the `$routeMiddleware` property and then adding the desired middleware to each route definition in the route class, e.g. `Route::get('/', function () { })->middleware('first', 'second');` or `Route::get('admin/profile', function () { })->middleware(CheckAge::class);`
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

# VIEWS

* Views contain the HTML served by your application and separate your controller / application logic from your presentation logic.
* Views are stored in the `resources/views` directory.
* `return view('greeting', [ 'name' => 'James' ]);` - will render `resources/views/greeting.blade.php` passing it the name parameter.
    * `return view('admin.profile', $data);` will render `resources/views/admin/profile.blade.php`
    * `view` global function equals `View::make` (see Facades - global helper functions differences)
* `View::exists('emails.customer')` checks if the view exists
* within a service provider's `boot` method you can place additional views configurations (Remember, if you create a new service provider to contain your view composer registrations, you will need to add the service provider to the providers array in the `config/app.php` configuration file)
    * You can make data available to all views by calling `Views::share($data);`  
* You can register view composers, i.e. callbacks or class methods that are called when a view is rendered (e.g. to bind default data to a view each time it is rendered)
* Laravel by default use [Blade templates](https://laravel.com/docs/6.x/blade)

## Blade syntax 

* Blade is the simple, yet powerful templating engine provided with Laravel. Unlike other popular PHP templating engines, Blade does not restrict you from using plain PHP code in your views. In fact, all Blade views are compiled into plain PHP code and cached until they are modified, meaning Blade adds essentially zero overhead to your application. 
* Blade view files use the .blade.php file extension and are typically stored in the `resources/views` directory.
* Blade provide sections to compose layouts:
    * `@section('mysection') ... @endsection` defines a section (without showing it)
    * `@section('mysection') ... @show` defines and immediately shows a section
    * `@section('mysection') @parent ... @endsection`  append section to the same-named parent's one
    * `@yield('mysection')` shows a section's content (e.g. in the parent layout)
* ...and components to define reusable view components:
    * `@component('myview', [ 'foo' => $bar ])  ... @endcomponent` renders the specified view, substituting its variable `$slot` with provided content and passing the variables array
* ...and a mechanisms to simply "include" other blade templates:
    * `@include('view.name', ['some' => 'data'])` will include the template and pass the data
    * `@includeIf('view.name', ['some' => 'data'])` only if the view exists
    * `@includeWhen($boolean, 'view.name', ['some' => 'data'])` only when condition is true (or `@includeUnless`)
* `{{ $phpContent }}` shows content (alike to `<?= htmlspecialchars($phpContent) ?>`
    * `{{!! $unescaped !!}}` will not escape with `htmlspecialchars` (e.g. to output raw html)
* `@json($phpArray)` will `json_encode` the provided array
* e.g. to seed vue components or `data-*` variables: `<example-component :some-prop='@json($array)'></example-component>`
* if you use a js framework which uses `{{ var }}`, you should escape them: `@{{ var }}` will not be elaborated by blade (it will just remove `@`), else you can surround a bigger portion of code with `@verbatim ... @endverbatim`
* Control structures are:
    * `@if (...) ... @elseif (...) ... @else ... @endif`
    * `@for (...) ... @endfor`
    * `@foreach (...) ... @endforeach` and `@forelse (...) ... @empty ... @endforelse`
    * `@while (...) ... @endwhile`
    * `@break` or `@break($condition)` to be used inside loops
    * the `$loop` object that contains info about the loop as properties (useful for nested loops, e.g. `$loop->parent`, `$loop->depth` etc)
    * `@unless (...) ... @endunless`
    * `@isset($var) ... @endisset`
    * `@empty($var) ... @endempty`
    * `@auth ... @endauth` and `@guest ... @endguest`
    * `@hasSection('mysection') ... @endif`
    * `@switch($i) @case(1) ... @break @case(2) ... @break ... @default ... @endswitch`
* `{{-- This comment will not be present in the rendered HTML --}}`
* `@php ... @endphp` is the same as `<?php .. ?>` (both valid for blade)
* you may `@push('scripts') <script src="/example.js"></script> @endpush` in multiple places and then `<head>  @stack('scripts') </head>` in the final child view to render the complete stack content (also, you can prepend with `@prepend('scripts') This will be first...  @endprepend`
* The `@inject`<!-- Head Contents --> directive may be used to retrieve a service from the Laravel service container.
* you can extend blade creating custom directives and conditional directives (see docs)

## Forms and CSRF prevention

* Any HTML forms pointing to POST, PUT, or DELETE routes that are defined in the web routes file should include a CSRF token field. Otherwise, the request will be rejected.

```html
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

HTML forms do not support PUT, PATCH or DELETE actions. So, when defining PUT, PATCH or DELETE routes that are called from an HTML form, you will need to add a hidden _method field to the form. The value sent with the _method field will be used as the HTTP request method:

```html
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```
...but in this case you can simply use the `@method` blade directive:

```htm
<form action="/foo/bar" method="POST">
    @method('PUT')
    @csrf
</form>
```

## Front-end assets

* Laravel provides basic frontend setup and scaffolding (which you can use or you can set up your own), using `npm`, `bootstrap`, `vue` (or `react`) and `webpack` (see Laravel Mix)
* `composer require laravel/ui:^1.0 --dev` provides bootstrap and vue scaffolding (you can choose react as well or use your own set of libraries). Once `laravel/ui` is installed you can generate frontend scaffolding with `artisan ui bootstrap`, `artisan ui vue`, `artisan ui react`, `artisan ui bootstrap --auth`, `artisan ui vue --auth`, etc
    * you can create your own laravel ui commands by registering them in a service provider (see docs)
* `npm install` in root project directory will install frontend dependencies, then `npm run dev` will process the instructions in `webpack.mix.js` file. By default it will compile `resources/sass/app.scss` into `public/css` and `resources/js/app.js` into `public/js`producing one single css file and one single js file to be loaded in each page (but this can be customized as you wish)
* it also creates by default a `resources/js/components` where vue or react components may be placed (then they have to be registered inside `resources/js/app.js`


# MODELS (Eloquent ORM)

* Laravel uses Eloquent (active-record ORM)
    * Each Model class abstracts a database table and acts as a query builder (i.e. has crud methods for the associated table to be used instead of SQL)
    * Each Migration class is a table schema modification associated with a timestamp: when you run `artisan migrate` Laravel will use the timestamp to determine which and in what ordermigrations are be applied (without loosing any data and being able to track schema modification changes alike as in version control)
* Check database connection parameters in `.env` and `config/database.php`
* If you already have a database table, you just have to create the corresponding model with the correct name (considering laravel naming conventions): creating a migration is optional (but can be done).
    * `artisan make:model NAME` creates a model class in `app` folder.
    * Eloquent will assume the `Flight` model stores records in the `flights` table, while an `AirTrafficController` model would store records in an `air_traffic_controllers` table.
    * You can specify a different table name by setting a property in the model class: `protected $table = 'my_flights'`; 
* If you need to create a new table for the model using eloquent you must create the associated migration too and define your table schema there.    
    * `artisan make:model NAME --migration` creates an associated migration together with the model 
    * `artisan make:migration create_users_table` creates a migration using users as table parameter

## Define/update the table schema in the migration class

```php
class CreateFlightsTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('flights', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('name');
            $table->string('airline');
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::drop('flights');
    }
}
```

## Model configuration

* Table name 
    * Eloquent will assume the `Flight` model stores records in the `flights` table, while an `AirTrafficController` model would store records in an `air_traffic_controllers` table.
    * You can specify a table name: `protected $table = 'my_flights'`; 
* Primary key
	* Eloquent will assume that each table has an auto-incrementing integer primary key column named `id`. 
	* You can 
	    * define a custom name: `protected $primaryKey = 'flight_id';`
	    * define a custom type: `protected $keyType = 'string';`
	    * set it non-auto-incrementing: `public $incrementing = false;`
* Timestamps
    * By default, Eloquent creates `created_at` and `updated_at` to exist on your tables
        * You can disable this with `public $timestamps = false;`
        * You can customize the format of the timestamps with `protected $dateFormat = 'U';`
        * You can customize the names of such columns with `const CREATED_AT = 'creation_date';` and `const UPDATED_AT = 'last_update';`
* Database connection
    * By default, all Eloquent models will use the default database connection
        * You can specify a different one with `protected $connection = 'connection-name';`
* Default attributes
    * You can set default attribute (columns) values: `protected $attributes = [ 'delayed' => false, // ...  ];` 

## Use model

* When you have a model and it's associated db table (an existing one or created via migrations), you can already use models.
* Models act as query builders: 
    * you can retrieve single instances, e.g. with `find`, `first`, `firstWhere`, `firstOr`, `firstOrFail`, `findOrFail`, and then refer column values and relationship-binded instances as properties
    * you can retrieve collections of instances using the same methods as above but passing an array, or with `all`, or using query-builder methods (`where`, `orderBy`, `take`, etc, and then `get`): the result will be an Eloquent Collection object so you can use on them any [method they provide](https://laravel.com/docs/6.x/eloquent-collections#available-methods) and you can loop through single instances with `foreach` or you can compute aggregate functions such as `count`, `sum`, `max`, etc.
    * you can repeat the retrieval of the query with `fresh` and `refresh` methods on the resulting object.
    * when you work with thousands of results, you can use `chunk` or `cursor` methods to optimize performance and memory usage, using [Lazy collections](https://laravel.com/docs/6.x/collections#lazy-collections).
    * you can use `select`, `addSelect` for subqueries with multiple tables, `orderby` for ordering results, etc

```php
// usage example
$flights = App\Flight::where('active', 1)
               ->orderBy('name', 'desc')
               ->take(10)
               ->get();
foreach ($flights as $flight) {
    echo $flight->name;
    $flight->name = "Modified name";
    $flight->save();
}
$newFlight = new Flight;
$flight->name = "new name";
$flight->save();
```

## Insert / Update / Delete / Compare model instances

* You can create a new model instance
    * with `$instance = new App\MyModel;` then assigning properties values to initialize them
    * with mass assignment, i.e. using `App\MyModel::create(['name' => $value, ...]);` or `$flight->fill(['name' => $value, ...]);
        * in order to use mass assignment, you have to register mass assignable properties in the model class, e.g. with `protected $fillable = ['name', ...];` (or you can make them all mass assignable excluding only some of them with `protected $guarded = ['price', ...];`)
* `save` method will persist changes to an existing instance or insert a newly created one (e.g. with `$instance = new App\MyModel;`)
* you can update an entire eloquent collection using `$collection->update([ 'mycolumn' => $myValue, ... ]);`
* you can check an instance (or one of its properties) has been changed since it was loaded with its instance methods `isDirty`, `isClean` and `wasChanged`.
* you can insert/update basing on a query result with `firstOrCreate`, `firstOrNew` and `updateOrCreate`
* you can delete one or more database rows with `$instance->delete()` or `$collection->delete();`, or `App\MyModel::destroy($id ...);`
* you can replicate an instance with `$instance->replicate()`
* soft deletion is supported but must be activated:
    * inside the model class, you must `use softdeletes;` and you must create the `deleted_at` table column (manually with sql or inside a migration class with `$table->softdeletes();`)
    * now, when you call the delete method on the model, the deleted_at column will be set to the current date and time. and, when querying a model that uses soft deletes, the soft deleted models will automatically be excluded from all query results.
    * you can determine if an instance is soft deleted with `if ($myInstance->trashed())`
    * you can include soft deleted instances in a query with `App\MyModel::withtrashed()->...` or query for just soft-deleted instances with `onlytrashed`
    * you can restore a soft deleted instance (or an entire collection) into an active state with `$instance->restore();`
    * when using soft-deletion, to permanently delete an instance or a collection you must use `forcedelete` instead of `delete`
* you can compare two instances with `$instance1->is($instance2)`

## Model's custom query scopes

* Query scopes are a laravel api to add constraints to queries.
* Global scopes are constraints on all queries of the model (e.g. adding a `where ...` to all sql queries of that model)
    * define a class that implements the `Illuminate\Database\Eloquent\Scope` interface. This interface requires you to implement one method: `apply` in which you can apply query constraints (such as `where`) to the `$builder` argument. Then to assign a global scope to a model, you should override a given model's boot method and use the `addGlobalScope` method
    * ...or else you can add it to the model class using a closure (see docs)
    * then, you can remove the global scope constraint on single queries with `App\MyModel::withoutGlobalScope(MyScope::class)->...` (or `withoutGlobalScope('name')` if it was defined with a closure), or you can remove all global scopes at once from that query with `withoutGlobalScopes()->..`
* Local scopes are re-usable sets of constraints that you can use as a query builder method (i.e. like `where` method): e.g. you can define a `popular` local scope by adding a `scopePopular` method to your model, and then you can use it when querying `App\MyModel::popular()->...` (see docs)
* Dynamic scopes are like local scopes, but with parameters (see docs)

## Model events and model Observers

* Eloquent models fire several events, allowing you to hook into the following points in a model's lifecycle: `retrieved`, `creating`, `created`, `updating`, `updated`, `saving`, `saved`, `deleting`, `deleted`, `restoring`, `restored`
* You can implement your event classes (e.g. `UserDeleted`) and register them in your model class with the `protected $dispatchesEvents = [ 'saved' => UserSaved::class, ... ];` property
* ...or you can create an Observer class for your model, which will contain a method for each event you listen to (thus grouping all listeners in a single class)
    * you can create a `UserObserver` with `php artisan make:observer UserObserver --model=User`
    * ...then you can register the observer by overriding the `boot` method in a service provider: e.g. inside the `AppServiceProviderClass` `boot` method you write `User::observe(UserObserver::class);`

# TESTS

* PHPUnit is included out of the box and a `phpunit.xml` file is already set up
    * Check `<server name="DB_CONNECTION" value="mysql"/>` and `<server name="DB_DATABASE" value="homestead"/>` are correct when testing a model or any database query: by default laravel sets a specific sqlite configuration as the default database for testing.
* Tests inside `tests/Unit` and `tests/Feature` will be run when `phpunit` command is executed

