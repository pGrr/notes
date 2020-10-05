# TESTS

* PHPUnit is included out of the box and a `phpunit.xml` file is already set up
* Tests inside `tests/Unit` and `tests/Feature` will be run when `phpunit` command is executed 
* If you define your own `setUp` / `tearDown` methods within a test class, be sure to call the respective `parent::setUp()` / `parent::tearDown()` methods on the parent class.
* Laravel provides API for unit testing, feature testing, http/browser/console testing, database testing and for mocking data ([see docs](https://laravel.com/docs/6.x/testing))

```bash
// Create a test in the Feature directory...
php artisan make:test UserTest
// Create a test in the Unit directory...
php artisan make:test UserTest --unit
```

## Database testing

* In `phpunit.xml` check `<server name="DB_CONNECTION" value="mysql"/>` and `<server name="DB_DATABASE" value="homestead"/>` are correct when testing a model or any database query: by default laravel sets a specific sqlite configuration as the default database for testing.
* to refresh the database (clean all the data) just `use RefreshDatabase` trait inside your test class
* to generate test data, you can use seeders and/or factories (see below)
* Laravel provides several database assertions for your PHPUnit feature tests:
    * `$this->assertDatabaseHas($table, array $data);`	Assert that a table in the database contains the given data.
    * `$this->assertDatabaseMissing($table, array $data);`	Assert that a table in the database does not contain the given data.
    * `$this->assertDeleted($table, array $data);`	Assert that the given record has been deleted.
    * `$this->assertSoftDeleted($table, array $data);`	Assert that the given record has been soft deleted.

### Database seeding

* Database seeders are classes, with a method `run`, that fill your database with test data and are located in `database/seeds`
* `database/seeds/DatabaseSeeder` is the main class, defined by Laravel, which `run` method will be executed when executing `artisan db:seed` command. Here you can call, in your preferred order, any additional seeder class `run`, e.g. with `$this->call([ UsersTableSeeder::class, PostsTableSeeder::class, CommentsTableSeeder::class, ]);`
* `php artisan make:seeder UsersTableSeeder` will create a new seeder class
* inside the `run` method of any seeder, you can insert data into the database using query builder methods (i.e. "manually") or using an Eloquent model factory (see below)
* in a test class extending `TestClass` you can call `$this->seed();` to run the main `DatabaseSeeder` or `$this->seed(OrderStatusesTableSeeder::class);` to run a specific seeder

```bash
# Create a seeder class
artisan make:seeder UsersTableSeeder
composer dump-autoload # necessary!

# Execute DatabaseSeeder run method
artisan make db:seed
# Execute a specific seeder class run method
artisan db:seed --class=UsersTableSeeder

## Drop all tables, re-run all migrations and then seed:
php artisan migrate:fresh --seed
```

```php
class DatabaseSeeder extends Seeder
{
    public function run()
    {
        // Use query builder methods ("manual" insertion)
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@gmail.com',
            'password' => Hash::make('password'),
        ]);

        // Use a factory
        factory(App\User::class, 50)->create()->each(function ($user) {
            $user->posts()->save(factory(App\Post::class)->make());
        });

        // Call additional seeders
        $this->call([
            UsersTableSeeder::class,
            PostsTableSeeder::class,
            CommentsTableSeeder::class,
        ]);
    }
}

// Seed usage (in feature tests)
$this->seed(); // Run the DatabaseSeeder...
$this->seed(OrderStatusesTableSeeder::class); // Run a single seeder...
```

### Model Factories

* Model factories are classes that, using the faker library, provide fake instances of their model.
    * You can set the Faker locale by adding a faker_locale option to your `config/app.php` configuration file.
* they are located in `database/factories`, all files in this directory will automatically be loaded by Laravel
* `artisan make:factory PostFactory` will create a new factory class
    * `artisan make:factory PostFactory --model=Post` will pre-fill the generated factory file with the given model

```php
// factory definition:
$factory->define(App\User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
        'remember_token' => Str::random(10),
    ];
});
// extend existing factory (e.g. for a model that has been extended):
$factory->define(App\Admin::class, function (Faker\Generator $faker) {
    return factory(App\User::class)->raw([
        // ...
    ]);
});
// define a factory's state
$factory->state(App\User::class, 'delinquent', [
    'account_status' => 'delinquent',
]);
$factory->state(App\User::class, 'address', function ($faker) {
    return [
        'address' => $faker->address,
    ];
});
// factory event callbacks (e.g. to associate related models)
$factory->afterMaking(App\User::class, function ($user, $faker) {
    // ...
});
$factory->afterCreating(App\User::class, function ($user, $faker) {
    $user->accounts()->save(factory(App\Account::class)->make());
});
$factory->afterMakingState(App\User::class, 'delinquent', function ($user, $faker) {
    // ...
});
// relationships definition
$factory->define(App\Post::class, function ($faker) {
    return [
        'title' => $faker->title,
        'content' => $faker->paragraph,
        // you can define relationships directly in the factory definition
        'user_id' => factory(App\User::class),
        'user_type' => function (array $post) {
            return App\User::find($post['user_id'])->type;
        },
    ];
});

// Factory Usage (in tests or in seed files):
// make will create the instance, but not save it in the database
$user = factory(App\User::class)->make(); // will not persist in db, needs "save"
$users = factory(App\User::class, 3)->make();
$users = factory(App\User::class, 5)->states('delinquent')->make();
$users = factory(App\User::class, 5)->states('premium', 'delinquent')->make();
$user = factory(App\User::class)->make([
    'name' => 'Abigail', // override attribute
]);
// create will create an instance and save it in the database
$user = factory(App\User::class)->create();
$users = factory(App\User::class, 3)->create();
$user = factory(App\User::class)->create([
    'name' => 'Abigail',
]);
// ...you can create relationships directly in the test:
$users = factory(App\User::class, 3)
           ->create()
           ->each(function ($user) {
                $user->posts()->save(factory(App\Post::class)->make());
            });
$user->posts()->createMany(
    factory(App\Post::class, 3)->make()->toArray()
);
```



