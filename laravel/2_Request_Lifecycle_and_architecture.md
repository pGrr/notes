# Request Lifecycle

Basically what happens is: 
  
  1) the application instance is created 
  2) the service providers are registered, and 
  3) the request is handed to the bootstrapped application

## 1. Front controller

* `public/index.php`
  * is the front-controller: all requests are directed to this file by your web server (Apache / Nginx) configuration
  * loads the __Composer generated autoloader__ definition
  * then __retrieves an instance of the Laravel application from `bootstrap/app.php`__ script
  * The first action taken by Laravel itself is to __create an instance of the application / service container__.
  * Next, the incoming request is __sent to either the HTTP kernel or the console kernel__, depending on the type of request. These are the app's central locations that all requests flow through

## 2a. HTTP Kernel 

* `app/Http/Kernel.php`
  * extends the `Illuminate\Foundation\Http\Kernel` class
  * __receives all http requests__
  * __defines an array of bootstrappers that will be run before the request is executed__
    * error handling, logging, detect the application environment, etc.
  * __defines a list of HTTP middleware__ that all requests must pass through before being handled by the application
    * reading and writing the HTTP session, determining if the application is in maintenance mode, verifying the CSRF token, etc
  * `handle` method receive a `Request` and return a `Response` (a big black box that represents your entire application)
  * __loads all the Service Providers__ that finds in the `config/app.php` configuration file's `$providers` array. First, the `register()` method will be called on all providers, then, once all providers have been registered, the `boot()` method will be called.
    * Service providers are responsible for bootstrapping all of the framework's various components, such as the database, queue, validation, and routing components.
    * default service providers are stored in the `app/Providers` directory (Having a firm grasp of service providers is very valuable as they are the central part of the framework).
    * By default, the `AppServiceProvider` is fairly empty. This provider is a great place to add your application's own bootstrapping and service container bindings. For large applications, you may wish to create several service providers, each with a more granular type of bootstrapping.
  * __dispatch requests__: once the application has been bootstrapped and all service providers have been registered, the `Request` will be handed off to the router for dispatching to a route or controller (as well as run any route specific middleware).
  
# Service container

* A Service provider registers a service into the container using `$this->app->bind('keyname', $closure)`, where closure is an anonymous function instantiating an object and returning it (ex after reading a config file, etc)
  * you can make the service a singleton by using instead `$this->app->singleton('keyname', $closure)`
* you can access the service from the container using:
  * `app()->resolve('keyname');`
* Automatic resolution
  * when the key is not presend in the containers' `$bindings`, Laravel will try anyway to resolve it (else will throw an exception). The steps are:
    * If the passed argument is a binded key, resolve it with the given closure
    * If the passed argument is a class, it will try to instantiate it. If the class has dependencies (constructor parameters) it will look for those classes and try to instantiate them... and so on (if there are ultimately no config paramenters, etc it will be able to resolve and return the object of MyClass with all dependencies instantiated)
    * 
  * `app()->resolve(MyClass::class)` - will try to instantiate the given class and all its dependencies
  * `app()->make(MyClass::class)` - same as above
  * `app()->make(MyClass::class, $neededParameters)` - same as above but passing the needed parameters
* Custom service providers
  * should be registered in `app/Providers/AppServiceProvider.php` in the `¶egister` function
  * inside any service provider, `app()` can be accessed as `$this->app` 

# Facades

* `view('welcome')` = `View::make('welcome')`
* Facades are a convenient way to access services within the service container as static methods (i.e. Facades are static interfaces):
  * Every Facade extends the parent `Facade` class and has a `getFacadeAccessor` method that returns the key of the binded Facade-related service inside the service container
  * The parent class use the `__callStatic` magic method to invoke the non-existing called static method on the facade-related service provided by the service container
* `view()`, `request()`, etc are __helper functions__ (it's just an alternate syntax, but the working is the same as facades)
* It is preferrable to define all class dependencies in the constructor, in order to make evident all class dependencies, and how much they grow

# Service providers

* inside laravel, every component has a `ServiceProvider` class (you can see that by inspecting the framework's code)
* `ServiceProvider`
  * inside their `register` method (which has `ServiceContainer` as parameter), just call `bind` on the service container, providing the closure to instantiate the service object 
  * may have a `boot` method, which if present is called after all service provider are registered
* So, the service container:
  * first binds all the services, by calling `register` on each `ServiceProvider`
  * then calls `boot` on each `ServiceProvider`, if present
* So `register` is for binding, `boot` is for initialization purposes, e.g. any action that must be performed on that service (this way if the service is dependent on another service or a service action, there won't be problems)