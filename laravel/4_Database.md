# Database

## DB Configuration

* `/.env` - contains all environments configuration parameters
  * `/config/database.php` - will read configuration parameters from it
* create the database (if not exists)

## DB Create schema

* Various options:
  * Create your schema manually (e.g. via SQL script)
    * then access with `\DB::table('table_name')->...`
      * e.g. `\DB::table('table_name')->where('column_name', $isEqualTo)->firstOrFail()`
  * Using Eloquent ORM (i.e. creating a model for an existing table)
    * create a model with `php artisan make:model MyModel`
    * then you can access with `MyModel::where('varname, $isEqualTo)`
      * e.g. `MyModel::where('column_name', $isEqualTo)->get()`
  * Via Migrations
    * create a migration with `php artisan make:migration create_mytable_table`
    * create the columns in `up()` function
    * `php artisan migrate`, or `php artisan migrate:fresh` (attention!)
    * Now in your model class:
      * you can treat the columns as class fields `$this->myColumn`
      * you can persist and flush the changes to database with `$this->save()`
  * Shortcut:
    * `php artisan make:model -mc` - creates the stub for a model, a controller and a migration
    * `php artisan make:model -a` - creates the stub for a model, a controller, a migration and a factory
* `php artisan tinker` - interactive shell

## Migrations

* You can create foreign keys:

```php
$table->foreign('user_id')
  ->references('id')
  ->on('users')
  ->onDelete('cascade');
```

# Accessing data (eloquent models)

* `\App\Article::all()`, `\App\Article::take(2)->get()`, `\App\Article::paginate(2)`
* `\App\Article::latest()->get()` - order by created_at in descending order
* `\App\Article::latest()->get('published_at')` - order by published_at in descending order

# Create and persist instances

* mass assignment:
  * `\App\Article::create(['name' => $name, ... ])`
    * in order to use this, for security, data passed in create method must also listed in the `$fillable`, which is an array containing the only columns name that can be mass-assigned (else someone would be able to other columns), or vice-versa you can use the `$guarded` to tell only the values which are not fillable (one or the other)
    * IMPORTANT: this will save the created instance into the database! If you want to just create the object you must use the standard syntax `new ...` and then, if you want, `save()` to persist it

# Eloquent relationships

* You declare an eloquent relationship inside the model class, defining a method to get the objects of that relationship, which calls a "relationship-method" and returns its result
* relationships are:
  * `belongsTo` - `hasOne`
  * `hasMany` - `belongsToMany`
  * ...etc

```php 

// one to one

class Project extends Model {

  // ...

  public function author() 
  {
    // if you specify the method name as different
    // from the class name (e.g. author instead of user)
    // you must specify the key column name
    return $this->belongsTo(User::class, 'author_id');
  }

}

// one to many

class User extends Model {

  // ...

  public function projects() 
  {
    return $this->hasMany(Project::class);
    // return the result of an appropriate query, e.g.
    // select * from projects where user_id = $this->id
  }

}

// retrieving

$user->projects; // is an eloquent collection
$user->projects->where('name', $name);
// Note: relationships are defined as methods,
// but you access the query result as a property
```

## Pivot (Join) tables

* you must create a pivot table in the migration
* name convention is `article_tag` (singular, in alphabetical order)

```php
// create_tags_table migration
Schema::create('article_tag', function Blueprint $table {
  $table->bigIncrements('id');
  $table->unsignedBigInteger('article_id');
  $table->unsignedBigInteger('tag_id');
  $table->timestamps();

  $table->unique('article_id', 'tag_id');
  $table->foreign('article_id')
    ->references('id')
    ->on('articles')
    ->onDelete('cascade');
  $table->foreign('tags_id')
    ->references('id')
    ->on('tags')
    ->onDelete('cascade');
});

// Article model
class Article extends Model {
  // ...
  public function tags()
  {
    return $this->belongsToMany(Tag::class)->withTimeStamps();
  }
}

// usage
$article->tags->pluck('name');
$tag->articles;

// attach
$article->tags()->attach($tags);
```

# Factories

* Factories are a way to easily create fake instances for a model, using `Faker` library

```php
// inside the factory class:
$factory->define(App\User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        // you can reference another factory for relationships:
        'profile' => $factory(Profile::class)
        'email_verified_at' => now(),
        'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
        'remember_token' => Str::random(10),
    ];
});

// usage (outside of factory class)
factory(User::class, 5)->create(['title' => 'MyTitle']);
// creates 5 instances with the specified title and
// other fake data
```

# Utils

* `\DB::getQueryLog()`