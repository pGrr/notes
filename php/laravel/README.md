# ARTISAN (CLI tool)

* Artisan is Laravel's cli tool for managing a laravel application and to generate various scaffold code
* `artisan` or `php artisan` - see commands
* `artisan <COMMAND> -h` - see help for a command

# MODELS

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

### Table name 

* Eloquent will assume the `Flight` model stores records in the `flights` table, while an `AirTrafficController` model would store records in an `air_traffic_controllers` table.
* You can specify a table name: `protected $table = 'my_flights'`; 

### Primary key

* Eloquent will assume that each table has an auto-incrementing integer primary key column named `id`. 
* You can 
    * define a custom name: `protected $primaryKey = 'flight_id';`
    * define a custom type: `protected $keyType = 'string';`
    * set it non-auto-incrementing: `public $incrementing = false;`

### Timestamps

* By default, Eloquent creates `created_at` and `updated_at` to exist on your tables
    * You can disable this with `public $timestamps = false;`
    * You can customize the format of the timestamps with `protected $dateFormat = 'U';`
    * You can customize the names of such columns with `const CREATED_AT = 'creation_date';` and `const UPDATED_AT = 'last_update';`

### Database connection

* By default, all Eloquent models will use the default database connection
    * You can specify a different one with `protected $connection = 'connection-name';`

### Attributes

* Each attribute you define will be a column of your db table (data types are automatically infered by Eloquent using php data types)
* You can set default attribute (columns) values:

```php
protected $attributes = [
        'delayed' => false,
        // ...
];
```

# TESTS

* PHPUnit is included out of the box and a `phpunit.xml` file is already set up
    * Check `<server name="DB_CONNECTION" value="mysql"/>` and `<server name="DB_DATABASE" value="homestead"/>` are correct when testing a model or any database query: by default laravel sets a specific sqlite configuration as the default database for testing.
* Tests inside `tests/Unit` and `tests/Feature` will be run when `phpunit` command is executed

