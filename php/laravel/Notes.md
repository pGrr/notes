# Utils

* `dd($var)` - dump and die (inspect a variable)
* `ddd($var)` - dump, die, debug
* `abort(404)` - will throw an exception and display a 404 page
* `Request::path()`  - check the current path (the part between domain and query)
* `Request::is('about')` - checks if the current path is 'about'
* `die('hello')` - echoes 'hello' and die
* `dump($something)` - dumps
* `redirect($path)`
* `redirect($path)->with($message)` - will redirect and will set a message into the session, for one request only
* `back()` - redirects to back page
* `compact()`
* `config('services.foo')` - returns the value of `foo` defined in `config/services.php`
* `public_path('index.php')`, `base_path()`, `resource_path()`, ...etc
* `sync()` - on eloquent collections