# Session

## Session driver config

* The session driver configuration option defines where session data will be stored for each request.
* Session driver can be modified in `.env` - `SESSION=...`
  * That value will be loaded by `config/session.php` (There are listed all available drivers) 
* Laravel's default is `file` session driver, but many other exist: 
  * `file`, `cookie`, `database`, `apc`, `memcached`, `redis`, `dynamodb`, `array` (not persisted)
  * when working with `database` driver, you must create a database table, e.g. using `php artisan session:table`

