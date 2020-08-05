# Authorization

## Simple, easy way (small apps)

* in `app/Providers/AuthServiceProvider.php`, in the `boot` method, you can register your own authentication functions, through the `Gate::define('my-key', $closure)` method, where the closure takes the required parameters as input and returns true if the user is authorized, false otherwise
* 

```php
public function boot()
{
    $this->registerPolicies();

    Gate::define('update-conversation', function ($user, $conversation) {
        return $conversation->user->is($user);
    });

    Gate::define('edit-settings', function ($user) {
        return $user->isAdmin;
    });

    Gate::define('update-post', function ($user, $post) {
        return $user->id === $post->user_id;
    });

    Gate::define('update-post', 'App\Policies\PostPolicy@update');
}

//usage in views:
@can('update-conversation', $conversation)

@endcan

// usage in controllers:
public function store(Reply $reply)
{
    $this->authorize($reply->conversation);

    // do authorized stuff: will be executed only if the auth succeeded
    // else, 403 - Unauthorized
}

// or (same implementation, different syntax)
public function store(Reply $reply)
{
    if (Gate::denies('update-conversation', $reply->conversation)) {

    } else if (Gate::allows('update-conversation', $reply->conversation)) {

    }
}
```

## Policies

* a policy is a class that encapsulates an authorization policy for a model
* `php artisan make:policy MyPolicy -m MyModel` - creates a policy class inside `app/Policies`, where you can define methods for you usecases, e.g. `update`, `create` (can user update, can create, etc)
* Policies don't need registration in `app/Providers/AuthServiceProvider.php`, they are automatically resolved by Laravel using conventions: a conversation policy is related to a model, so when you pass a model object to the `authorize` functions (see above), they will be resolved to the corresponding policy on the method given as "key"

```php
// in controller
public function store(Reply $reply)
{
    $this->authorize('update', $this->conversation); 
    // since we are passing a Conversation model object,
    // the authorization will be resolved on ConversationPolicy class, 
    // update method
}

// in ControllerPolicy class
public function before(User $user) // there is also an "after"
{
    // this will be executed before any authorization method on this class
    // only if this method doesnt return the called auth method will be called
    if ($user->isAdmin()) {
        return true;
    }
    // never "just return", e.g. "return $user->isAdmin();"
    // or the auth method will never be called (in this case, only return if it's true)
}

// "auth" method
public function update(User $user, Conversation $conversation)
{
    return $conversation->user->is($user);
}

// in the view
@can ('update', $conversation) ... @endcan

// if you want to define a global "before" method, 
// e.g. to grant everything to an admin without repeating code,
// you can define it in AuthServiceProvider boot method:
public function boot()
{
    $this->registerPolicies();
    Gate::before(function (User $user) {
        if ($user->isAdmin() {
            return true; // never "just return" (see above)
        })
    });
}

```

# Roles


