# Laravel 7.x Directory Structure 

* Laravel imposes almost no restrictions on where any given class is located - as long as Composer can autoload the class
* Nonetheless it proposes an application structure intended to provide a great starting point for both large and small applications

## Lacking of a "model" folder

* the lack of a `model` directory is intentional. 
* We find the word "models" ambiguous since it means many different things to many different people. Some developers refer to an application's "model" as the totality of all of its business logic, while others refer to "models" as classes that interact with a relational database.
* For this reason, we choose to place Eloquent models in the app directory by default, and allow the developer to place them somewhere else if they choose.

# Root directory structure

* `app`
  * the core code of your application: almost all of the classes in your application will be in this directory
* `bootstrap`
  * contains `app.php` which bootstraps the framework
  * houses a `cache` directory which contains framework generated files for performance optimization such as the route and services cache files
* `config`
  * contains all of your application's configuration files
  * It's a great idea to read through all of these files and familiarize yourself with all of the options available to you
* `database`
  * contains your database migrations, model factories, and seeds
  * If you wish, you may also use this directory to hold an SQLite database
* `public`
  * should be configured to be the only folder exposed to the outside world by the webserver
  * contains `index.php`
    * it is the front-controller, i.e. the entry point for all requests entering your application
    * it and configures autoloading
  * contains all your static assets such as `images`, `JavaScript`, and `CSS`
* `resources`
  * Should contain:
    * views
    * raw, un-compiled assets such as LESS, SASS, or JavaScript
    * language files