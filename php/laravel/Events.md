# Events

* when an event happens (e.g. the user pays) you might have to do many actions ("side effects") and the controller method might grow too much. 
  * Pros: you can immediatly see what the controller does
* There are many ways to handle this (e.g. creating a "use case class"
  * e.g. Payment, where you define all the methods
* else create an event and its listeners, then the controller will just fire the event)
  * cons: now you have to search all event listeners in order to understand what the controller does
  * pros: it is more structured, isolated and open for extension: you can reuse listeners for multiple events, you can just add listeners when needed
* `php artisan make:event MyEvent` - create event in `app/Events`
* `php artisan make:listener MyListener` - create listener in `app/Listeners`
  * `handle(MyEvent $e)` method, should be type-hinted with the event
  * `php artisan make:listener MyListener --event=MyEvent` - is a shortcut
* `php artisan event:list` - list all events and listeners registered
* `app/Providers/EventServiceProvider.php` - bootstrap for everything that concerns events in the applications
  * `$listen` is an associative array containing events as keys and listeners as values
  * __You MUST register your events and listeners there__

```php
// usage
MyEvent::dispatch($args);
// or
event(new MyEvent($args));
```

# Auto-discover events setting

By overriding the following method in `EventServiceProvder` Laravel will auto-discover evetns, by:

* scanning Events and Listener folders classes through reflection API
* resolve the dependencies thanks to type-hinting

```php
public function shouldDiscoverEvents()
{
    return true;
}
```