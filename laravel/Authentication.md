# Authentication

* `php artisan ui vue --auth`
  * preset scaffolding ui for authentication using vue (you can also use `bootstrap` or `¶eact`)
* `middleware('auth')`
  * `$this->middleware('auth')` - inside the controller's method, or
  * in the route: `Route::get('/home', 'HomeController@index')->name('home')->middleware('auth');`
* `@if (Auth::check()) Hello, {{Auth::user()->name}}! @endif` 
  * `{{auth()->user()->name}}` (is the same)
  * `@if (Auth::user()) ... endif` (is the same)
  * `@auth Hello, {{Auth::user()->name}}! @endauth` (is the same)
  * `@guest ... @endguest` (is the opposite of `@auth`)

# Password reset flow

1. Click forgot password
2. Fill out a form with their email address
3. Prepare a unique token and associate it with that user's account
4. Send an email containing a link with the verification url + the unique token + the user identifier (e.g. the email) that confirms the email ownership
5. Link back to the website, confirm the token and set a new password

Auth is achieved using `Http/Controllers/Auth/...` code and a database table `password_resets`, and the views in `resources/views/auth/...`