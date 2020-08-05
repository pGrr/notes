# Controllers

* `php artisan make:controller MyClassName` - create controller
  * `php artisan make:model -c MyClassName` - create model + controller


# Passing data from routes to controller

* You can use a route wildcard as a controller's method parameter

## Route model binding

* or you can "type-hint) the controller method parameter, laravel will infer the associated model and retrieve it for you using the wildcard as the primary key of the model. 
  * Route's wildcard name and controller's method's parameter name must be the same
  * If something goes wrong, it aborts the request resulting in a 404 page (equivalent of `findOrFail`)
* you can specify a different searching parameter by overriding the method `getRouteKeyName` in the associated model. In that method you just return the name of the column to be used as a searching parameter

