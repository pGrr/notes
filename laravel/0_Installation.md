# Laravel 7.x

# Server requirements

* The Laravel framework has a few system requirements
  * All of these requirements are satisfied by the Laravel Homestead virtual machine ([see more](https://laravel.com/docs/7.x/homestead))
  * Else, check all requirements are satisfied ([see more](https://laravel.com/docs/7.x#server-requirements))

# Install

* Installing laravel installer globally
  * `composer global require laravel/installer`
    * `composer global about` - will tell you what is the directory that composer used for global installations
  * `laravel new <project-root>` - will create a fresh Laravel installation in the directory you specify,with all of Laravel's dependencies already installed
* Or else simply installing it locally with composer: 
  * `composer create-project --prefer-dist laravel/laravel <project-root>`

# Serve

* `php artisan serve` - will start a development server at `http://localhost:8000` using the PHP built in server (if Homestead or other local environments solutions are not used)
* For homestead-powered installations, see Homestead file

# Configuration

* configure your web server's document / web root to be the `public` directory. The `index.php` in this directory serves as the front controller for all HTTP requests. This must be the only folder that should be made public by the webserver!
* `config` folder holds all configuration files. Each option is documented, so feel free to look through the files and get familiar with the options available to you.
* Directories within the `storage` and the `bootstrap/cache` directories should be writable by your web server or Laravel will not run. If you are using the Homestead virtual machine, these permissions should already be set.
* Set your application key to a random string: 
  * If you installed Laravel via Composer or the Laravel installer, this key has already been set for you by the `php artisan key:generate` command
  * Typically, this string should be 32 characters long. 
  * The key can be set in the `.env` environment file. 
  * If you have not copied the `.env.example` file to a new file named `.env`, you should do that now. 
  * If the application key is not set, your user sessions and other encrypted data will not be secure!
* Pretty URLs must be configured for Apache and Nginx if Homestead is not used ([see more](https://laravel.com/docs/7.x#pretty-urls))
* You are now ready to go. 
* For more configurations:
  * Review the `config/app.php` file and its documentation. It contains several options such as timezone and locale that you may wish to change according to your application. You may also want to configure a few additional components of Laravel, such as: Cache, Database, Session, etc
  * [See docs]() for more, such as: 
    * Environment Configuration: Environment Variable Types, Retrieving Environment Configuration,Determining The Current Environment, Hiding Environment Variables From Debug Pages
    * Accessing Configuration Values
    * Configuration Caching
    * Maintenance Mode
