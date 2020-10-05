# LARAVEL 6.X

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

## Front-end

* Laravel provides basic frontend setup and scaffolding (which you can use or you can set up your own), using `npm`, `bootstrap`, `vue` (or `react`) and `webpack` (see Laravel Mix)
* `composer require laravel/ui:^1.0 --dev` provides bootstrap and vue scaffolding (you can choose react as well or use your own set of libraries). Once `laravel/ui` is installed you can generate frontend scaffolding with `artisan ui bootstrap`, `artisan ui vue`, `artisan ui react`, `artisan ui bootstrap --auth`, `artisan ui vue --auth`, etc
    * you can create your own laravel ui commands by registering them in a service provider (see docs)
* `npm install` in root project directory will install frontend dependencies, then `npm run dev` (or `npm run watch`) will process the instructions in `webpack.mix.js` file. By default it will compile `resources/sass/app.scss` into `public/css` and `resources/js/app.js` into `public/js`producing one single css file and one single js file to be loaded in each page (but this can be customized as you wish)
* it also creates by default a `resources/js/components` where vue or react components may be placed (then they have to be registered inside `resources/js/app.js`
* for more complex front end workflow configurations (e.g. browsersync, etc), [see laravel mix docs](https://laravel.com/docs/6.x/mix)

# DATABASE 

## DB Configuration

* Laravel supports MySql, PostGre, Sqlite and SQL Server.
* Check database connection parameters in `.env` and `config/database.php`

```bash
# Sqlite setup
touch database/database.sqlite
# ...then, in .env file:
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database.sqlite
DB_FOREIGN_KEYS=true # to enable foreign keys
```

## DB API

* Once you have configured your database connection, you may:
    * get the default connection with `DB::connection()` or a specific one with `DB::connection('foo')`, or access the underlying pdo with `$pdo = DB::connection()->getPdo();`
    * run queries using the DB facade, which provides methods for each type of query: select, update, insert, delete, and statement, e.g. `$email = DB::table('users')->where('name', 'John')->value('email');` (see examples below).The select method will always return an array of results. Each result within the array will be a PHP `stdClass` object, allowing to access column values as properties
    * use the query builder api to make queries ([see docs](https://laravel.com/docs/6.x/queries)), e.g. `$email = DB::table('users')->where('name', 'John')->value('email');`
    * receive each SQL query, binding or timing for logging or debugging purposes by registering a `listen` method in a service provider (see docs) 
    * use the `transaction` method on the DB facade to run a set of operations within a database transaction. If an exception is thrown within the transaction Closure, the transaction will automatically be rolled back. If the Closure executes successfully, the transaction will automatically be committed (see docs). Also, you can manually control a transaction with `DB::beginTransaction()`, `DB::rollback()`, `DB::commit()` (this api is the same used in the query builder and in Eloquent ORM).
    * paginate results, e.g. with `DB::table('users')->paginate(15);` (see docs). Also, default ui scaffolding for pagination is available (if you use bootstrap).

```php
$users = DB::select('select * from users where active = ?', [1]);
$results = DB::select('select * from users where id = :id', ['id' => 1]);
DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
$affected = DB::update('update users set votes = 100 where name = ?', ['John']);
$deleted = DB::delete('delete from users');
DB::statement('drop table users');
$email = DB::table('users')->where('name', 'John')->value('email');
// ...etc (see query builder docs)
```

## Migrations

* Migrations are the laravel API to manage the database schema (creating, updating). In addition to what you can accomplish with plain sql, migrations provide version control of the database schema modifications, directly in the source code.
* Each Migration class is a table schema modification associated with a timestamp: when you run `artisan migrate` Laravel will use the timestamp to determine which and in what ordermigrations are be applied (without loosing any data and being able to track schema modification changes alike as in version control)

```bash
artisan make:migration create_users_table # creates a migration using users as table parameter
```

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

# MODELS (Eloquent ORM)

* Laravel uses Eloquent (active-record ORM)
    * Each Model class abstracts a database table and acts as a query builder (i.e. has crud methods for the associated table to be used instead of SQL)
* If you already have a database table, you just have to create the corresponding model with the correct name (considering laravel naming conventions): creating a migration is optional (but can be done).
    * `artisan make:model NAME` creates a model class in `app` folder.
    * Eloquent will assume the `Flight` model stores records in the `flights` table, while an `AirTrafficController` model would store records in an `air_traffic_controllers` table.
    * You can specify a different table name by setting a property in the model class: `protected $table = 'my_flights'`; 
* If you need to create a new table for the model using eloquent you must create the associated migration too and define your table schema there.    
    * `artisan make:model NAME --migration` creates an associated migration together with the model 
    * `artisan make:migration create_users_table` creates a migration using users as table parameter

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

## Relationships

### Define relationships

* to define a relationships of a model, you declare inside that model a public method which returns `$this->hasOne('App\MyRelatedModel')`, or one or the other relationships methods
    * `hasOne` and `belongsTo` (inverse, in the related model)
    * `hasMany` and `belongsTo` (inverse, in the related model)
    * `belongsToMany` and `belongsToMany` - this will create an intermediate table which by default is named `modela_modelb` (where models are placed in alphabetical order)
    * `hasOneThrough` and `hasManyThrough` define a one way relationships through an intermediate table
	* you can access the intermediate table with the `pivot` property, e.g. `App\User::find(1)->roles->pivot->created_at` (pivot represents the intemediate table and can be used as any other model. If you wish, you can assign a different name than 'pivot', see example below)
* Laravel supports polymorphic relationship which allows the target model to belong to more than one type of model using a single association.
	* One-to-one polymorphic: For example, a blog Post and a User may share a polymorphic relation to an Image model. Using a one-to-one polymorphic relation allows you to have a single list of unique images that are used for both blog posts and user accounts. In this case you can `return $this->morphTo();` in the `imageble` method of `Image` and `return $this->morphOne('App\Image', 'imageable');` in the `image` method of `Post` and `User`. Then you can `$image = $post->image;`, `$image = $user->image;` and `$imageable = $image->imageable;`, where `$imageable` will be either a `Post` or `User` instance. 
	* One-to-many polymorphic: For example, imagine users of your application can "comment" on both posts and videos. Using polymorphic relationships, you may use a single comments table for both of these scenarios. You can `return $this->morphTo();` in the `commentable` method of `Comment`, and `return $this->morphMany('App\Comment', 'commentable');` in the `comments` method of both `Post` and `Video`. Then you can `foreach ($post->comments as $comment) {  }`, `foreach ($post->comments as $comment) {  }` and `$commentable = $comment->commentable;`, where `$commentable` can be either a `Post` or `Video` instance.
	* Many-to-many relationships: For example, a blog Post and Video model could share a polymorphic relation to a Tag model. Using a many-to-many polymorphic relation allows you to have a single list of unique tags that are shared across blog posts and videos. You can `return $this->morphToMany('App\Tag', 'taggable');` in the `tags` method of both `Post` and `Video` and then in `Tag` you `return $this->morphedByMany('App\Post', 'taggable');` from the `posts` method and `return $this->morphedByMany('App\Video', 'taggable');` from the `videos` method. Then you can foreach `($post->tags as $tag) {  }`, `($video->tags as $tag) {  }`, `foreach ($tag->videos as $video) { }` and `foreach ($tag->posts as $post) { }`
	* Custom polymorphic types: you may wish to decouple your database from your application's internal structure. In that case, you may define a "morph map" to instruct Eloquent to use a custom name for each model instead of the class name

```php
class User extends Model
{
    public function phone()
    {
        return $this->hasOne('App\Phone'); // assumes a user_id foreign key and local primary key
    }
}

// Variants:

// specify the foreign key, if different
return $this->hasOne('App\Phone', 'foreign_key'); 
// specify the local key, if it's not the primary key
return $this->hasOne('App\Phone', 'foreign_key', 'local_key'); 

// belongsToMany syntax is different:
return $this->belongsToMany('App\Role', 'role_user'); // custom table name
return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id'); // table name, foreign key, local key
return $this->belongsToMany('App\Role')->withPivot('column1', 'column2'); // extra columns
return $this->belongsToMany('App\Role')->withTimestamps(); // add timestamps
return $this->belongsToMany('App\Podcast')->as('subscription')->withTimestamps(); // customize pivot name
User::with('podcasts')->first()->subscription->created_at;
return $this->belongsToMany('App\Role')->wherePivot('approved', 1); // filter basing on pivot (intermediate table) columns
return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);
return $this->belongsToMany('App\Role')->wherePivotNotIn('priority', [1, 2]);
return $this->belongsToMany('App\User')->using('App\RoleUser'); // use custom model (with 'extends Pivot') as intermediate table

// hasOneThrough and hasManyThrough syntax is different:
return $this->hasOneThrough(
    'App\History', // final model
    'App\User', // intermediate model
    'supplier_id', // Foreign key on users table... (optional)
    'user_id', // Foreign key on history table... (optional)
    'id', // Local key on suppliers table... (optional)
    'id' // Local key on users table... (optional)
);
return $this->hasManyThrough(
    'App\Post', // final model
    'App\User', // intermediate model
    'country_id', // Foreign key on users table... (optional)
    'user_id', // Foreign key on posts table... (optional)
    'id', // Local key on countries table... (optional)
    'id' // Local key on users table... (optional)
);

// belongsTo, hasOne, hasOneThrough, and morphOne relationships 
// allow you to define a default model that will be returned if the given relationship is null
return $this->belongsTo('App\User')->withDefault();
return $this->belongsTo('App\User')->withDefault([ 'name' => 'Guest Author', ]);
return $this->belongsTo('App\User')->withDefault(function ($user, $post) { $user->name = 'Guest Author'; });
```

### Query relationships

* Eloquent relations are methods. But Laravel implements a mechanism called "Dinamyc-properties" which allow you to access relationship methods as if they were properties defined on the model (see docs for more info)
    * when you access relations as methods, you get an instance obtain an instance of the relationship without actually executing the relationship queries, and since that instance serves as a query builder, you can use all [query builder methods](https://laravel.com/docs/6.x/queries) to chain constraints to the query before the actual sql execution.
    * when you access relations as properties you don't get a relationship instance: you get an object that will contain the final result of the query. So you can't use them as query builder. By default the data is "lazy loaded", which means the query is executed when you first access the property.
        * You can use eager loading to pre-load relationships you know will be accessed after loading the model. Eager loading provides a significant reduction in SQL queries that must be executed to load a model's relations.

```php
// accessing relationships as methods, you can chain query contstrains
$user->posts()->where('active', 1)->get(); // select * from posts where user_id = ? and active = 1
$user->posts()->where('active', 1)
        ->orWhere('votes', '>=', 100)
        ->get(); // select * from posts where user_id = ? and active = 1 or votes >= 100
$user->posts()->where(function (Builder $query) {
        return $query->where('active', 1)
                     ->orWhere('votes', '>=', 100);
    })->get(); // select * from posts where user_id = ? and (active = 1 or votes >= 100)

// or you can access the query result as a property (lazy loading by default, will be loaded on first usage)
$user = App\User::find(1);
foreach ($user->posts as $post) { }

// To query for relationship existance: has/doesntHave, whereHas/whereDoesntHave, whereHasMorph/whereDoesntHaveMorph
// Retrieve all posts that have at least one comment...
$posts = App\Post::has('comments')->get();
// Retrieve all posts that have three or more comments...
$posts = App\Post::has('comments', '>=', 3)->get();
// Retrieve posts that have at least one comment with votes...
$posts = App\Post::has('comments.votes')->get();
// Retrieve posts with at least one comment containing words like foo%...
$posts = App\Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'foo%');
})->get();
// Retrieve posts with at least ten comments containing words like foo%...
$posts = App\Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'foo%');
}, '>=', 10)->get();

// to count the number of results from a relationship without actually loading them:
$posts = App\Post::withCount('comments')->get();
foreach ($posts as $post) {
    echo $post->comments_count;
}
$posts = App\Post::withCount(['votes', 'comments' => function (Builder $query) {
    $query->where('content', 'like', 'foo%');
}])->get();
echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
$posts = App\Post::withCount([
    'comments',
    'comments as pending_comments_count' => function (Builder $query) {
        $query->where('approved', false);
    },
])->get();
echo $posts[0]->comments_count;
echo $posts[0]->pending_comments_count;
$posts = App\Post::select(['title', 'body'])->withCount('comments')->get();
echo $posts[0]->title;
echo $posts[0]->body;
echo $posts[0]->comments_count;
$book = App\Book::first();
$book->loadCount('genres');
$book->loadCount(['reviews' => function ($query) {
    $query->where('rating', 5);
}])
```

#### Eager loading

* When accessing Eloquent relationships as properties, the relationship data is "lazy loaded". This means the relationship data is not actually loaded until you first access the property. However, Eloquent can "eager load" relationships at the time you query the parent model.

```php
// Eager loading is advised when you know in advance you will make use of a model relationship,
// to avoid the "N+1 query problem", i.e.:
$books = App\Book::all();
foreach ($books as $book) {
    echo $book->author->name; // this will execute a query for each book
}
// whereas with eager loading only 2 queries will be executed:
$books = App\Book::with('author')->get(); // select * from books
foreach ($books as $book) {
    echo $book->author->name; // select * from authors where id in (1, 2, 3, 4, 5, ...)
}

// egear loading for multiple relationships:
$books = App\Book::with(['author', 'publisher'])->get();
// nested eager loading:
$books = App\Book::with('author.contacts')->get();
// eager load specific columns:
$books = App\Book::with('author:id,name')->get();
// nested morphTo eager loading:
$activities = ActivityFeed::query()
    ->with(['parentable' => function (MorphTo $morphTo) {
        $morphTo->morphWith([
            Event::class => ['calendar'],
            Photo::class => ['tags'],
            Post::class => ['author'],
        ]);
    }])->get();
// costraining eager loads:
$users = App\User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%first%');
}])->get();
$users = App\User::with(['posts' => function ($query) {
    $query->orderBy('created_at', 'desc');
}])->get();

// lazy eager loading:
$books = App\Book::all();
if ($someCondition) {
    $books->load('author', 'publisher');
}
$author->load(['books' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
// load only if not already loaded:
$book->loadMissing('author');

// nested eager loading in morphTo
$activities = ActivityFeed::with('parentable')
    ->get()
    ->loadMorph('parentable', [
        Event::class => ['calendar'],
        Photo::class => ['tags'],
        Post::class => ['author'],
    ]);
```

### Insert relationship instances

```php
// Instead of manually setting the post_id attribute on the Comment, you can use save or saveMany:
$comment = new App\Comment(['message' => 'A new comment.']);
$post = App\Post::find(1);
$post->comments()->save($comment); // will automatically add the appropriate post_id
$post = App\Post::find(1);
$post->comments()->saveMany([
    new App\Comment(['message' => 'A new comment.']),
    new App\Comment(['message' => 'Another comment.']),
]);
// create/createMany are the same as save/saveMany, but accept a php array instead of a model instance
$post = App\Post::find(1);
$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
// If you would like to 'save' your model and all of its associated relationships, you may use "push"
$post = App\Post::find(1);
$post->comments[0]->message = 'Message';
$post->comments[0]->author->name = 'Author Name';
$post->push();
$post = App\Post::find(1);
$post->comments()->createMany([
    [
        'message' => 'A new comment.',
    ],
    [
        'message' => 'Another new comment.',
    ],
]);
// You may also use the findOrNew, firstOrNew, firstOrCreate and updateOrCreate methods to create and update models on relationships.

// in belongsTo relationships
// associate/dissociate will set/unset the foreign key on the child model
$account = App\Account::find(10);
$user->account()->associate($account);
$user->save();
$user->account()->dissociate();
$user->save();
App\User::find(1)->roles()->save($role, ['expires' => $expires]); // additional intermediate table values in many-to-many relationships

// in many-to-many relationships
// attach/detach will insert/remove an instance in the intermediate table, without affecting the two end models:
$user = App\User::find(1);
$user->roles()->attach($roleId);
$user->roles()->attach($roleId, ['expires' => $expires]);
$user->roles()->detach($roleId); // Detach a single role from the user...
$user->roles()->detach(); // Detach all roles from the user...
$user->roles()->detach([1, 2, 3]);
$user->roles()->attach([
    1 => ['expires' => $expires],
    2 => ['expires' => $expires],
]);
// with sync, all ids that are not in the given array will be removed from the intermediate table
$user->roles()->sync([1, 2, 3]);
$user->roles()->sync([1 => ['expires' => true], 2, 3]); // add additional intermediate table values
// if you don't want to detach the missing ids, use syncWithoutDetaching
$user->roles()->syncWithoutDetaching([1, 2, 3]);
// toggle will attach the id if missing from the intermediate table, and remove it if present
$user->roles()->toggle([1, 2, 3]);
// updateExistingPivot will update a value in the intermediate table 
$user->roles()->updateExistingPivot($roleId, $attributes); // pivot record foreign key, attributes to update

// When a model belongsTo or belongsToMany another model, such as a Comment which belongs to a Post,
// it is sometimes helpful to update the parent's timestamp when the child model is updated
// For example, when a Comment model is updated, you may want to automatically "touch" the updated_at timestamp of the owning Post
// you can set the Comment to do that automatically by setting its touches property:
protected $touches = ['post'];
```

## Accessors and mutators

* Essentially accessors and mutators are like getters and setters that apply a transformation on an attribute when it is inserted or retrieved from the database or compute an attribute using other attributes.
    * e.g. you can apply encryption when you store a value and decrypt it when your retrieve it, or format a date
    * you can return values obtained by combining other values (e.g. full name from first name and last name), etc. If you would like these computed values to be added to the array / JSON representations of your model, [you will need to append them](https://laravel.com/docs/6.x/eloquent-serialization#appending-values-to-json).
* you just have to create a `getFooAttribute` or `setFooAttribute` (e.g. `getFirstNameAttribute` for a `first_name` attribute) which takes a `$value`, does something with it and then returns the computed value. Then, when you will access that as a model property, you will get (or set) the computed value.
* Additional configurations and utilities are available for [dates](https://laravel.com/docs/6.x/eloquent-mutators#date-mutators), [attribute casting](https://laravel.com/docs/6.x/eloquent-mutators#attribute-casting) (e.g. to always convert a value to a boolean, even if it's stored as integer in the database), and [array/json casting](https://laravel.com/docs/6.x/eloquent-mutators#array-and-json-casting)

```php
class User extends Model
{
    // Accessor
    public function getFirstNameAttribute($value)
    {
        return ucfirst($value);
    }

    // Mutator
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = strtolower($value);
    }

    // Derived accessor
    public function getFullNameAttribute()
    {
        // (if you want it to be added to array / json model representation you will need to append them)
        return "{$this->first_name} {$this->last_name}";
    }
}

// usage:
$user = App\User::find(1);
$firstName = $user->first_name;
$user->first_name = 'Sally';
```

## API Resources

* When building an API, you may need a transformation layer that sits between your Eloquent models and the JSON responses that are actually returned to your application's users. Laravel's resource classes allow you to expressively and easily transform your models and model collections into JSON.
* To generate a resource class, you may use the `make:resource` Artisan command. By default, resources will be placed in `app/Http/Resources` 
* Resource collections apply the same concepts on collections of model, giving also the possibility to add other meta information to the response. To create a resource collection, you should use the `--collection` flag when creating the resource. Or, including the word `Collection` in the resource name
* [See docs](https://laravel.com/docs/6.x/eloquent-resources) for more (e.g. preserving collection keys, customizing the underlying resouce class, nested resources, pagination, data wrapping, conditional attributes, conditional relationships, adding meta-data, return the json resource while modifying the http response header, etc)
* Laravel provides a handy 'Resource' code scaffolding for REST-API CRUD operations. 
    * `php artisan make:controller PhotoController --resource` - will generate a controller with all CRUD methods stubbed, `Route::resource('photos', 'PhotoController');` creates multiple routes to handle actions on the resource. The generated controller will already have methods stubbed for each of these actions
    * you can even bind a model to the resource 
* [see more](https://laravel.com/docs/6.x/controllers#resource-controllers)

```bash
# Resource
php artisan make:resource User
# Resource collection
php artisan make:resource Users --collection
php artisan make:resource UserCollection
```

```php
// Resource
namespace App\Http\Resources;
use Illuminate\Http\Resources\Json\JsonResource;
class User extends JsonResource
{
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}

// usage:
use App\Http\Resources\User as UserResource;
use App\User;
Route::get('/user', function () {
    return new UserResource(User::find(1));
});
Route::get('/user', function () {
    return UserResource::collection(User::all()); // to add meta-data, use a collection class
});

// Resource collection
namespace App\Http\Resources;
use Illuminate\Http\Resources\Json\ResourceCollection;
class UserCollection extends ResourceCollection
{
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}

// Usage:
use App\Http\Resources\UserCollection;
use App\User;
Route::get('/users', function () {
    return new UserCollection(User::all());
});
```

## Model to array / json serialization

* `toArray`/`toJson` converts a model (or an eloquent collection) and recursively all its relationships to a php array or to JSON
    * `attributesToArray`/`attributesToJson` does the same but including only the model's attribute (no relationships)
* `protected $hidden = ['password'];` in a model will exclude 'password' attribute from the serializations
* `protected $visible = ['first_name', 'last_name'];` will make 'first_name' and 'last_name' the only attributes serialized
* `return $user->makeVisible('attribute')->toArray();` and `return $user->makeHidden('attribute')->toArray();` will modify the visibility at runtime
* [see docs](https://laravel.com/docs/6.x/eloquent-serialization) for more info (e.g. appending additional values at compile/run-time, date serialization, etc)

```php
// array serialization
$user = App\User::with('roles')->first();
return $user->toArray();
return $user->attributesToArray();
$users = App\User::all();
return $users->toArray();

// json serialization
$user = App\User::find(1);
return $user->toJson();
return $user->toJson(JSON_PRETTY_PRINT);
// casting a model or collection to string converts to json by default
return (string) $user; 
Route::get('users', function () {
    return App\User::all(); // will return json
});
```

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



