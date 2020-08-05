# Notifications

* Notifications are to be preferred to emails whenever they represent a response to an action. The advantage over emails is that they are not limited to emails, but can be sent in other channels.
* `php artisan make:notification PaymentReceived` - creates a class in `app/Notifications/`
  * with methods `via` to define the channel, `toMail`, etc
  * with methods to construct the message, e.g. `subject`, `line`, `action`, `greeting`, etc

```php
Notification::send(request()->user(), new PaymentReceived());
// or via the user
request()->user()->notify(new PaymentReceived());
// or
$user->notify(new PaymentReceived());
```

# Database notifications

* create a table with `php artisan notifications:table`, `php artisan migrate`
* you can insert `database` into `via` function of the Notification class to enable the database notification

```php
public function via($notifiable)
{
    // will both send an email and 
    // save a database notification
    return ['mail', 'database'];
}
```

# Get notifications

* Any `Notifiable` entity (such as `User`) has helper methods to get notifications:
  * `notifications`, `readNotifications`, `unreadNotifications`, etc

```php
auth()->user()->notifications;
```

```php
@foreach ($notifications as $notification)
    We received a payment of {{$notification->data['amount']}}
@endforeach
```

# Mark as read/unread

* `$notifications` - Eloquent notifications collection
  * `markAsRead`, etc
* `$notification` - single notification
  * `markAsRead`, etc

# SMS

* `composer require laravel/nexmo-notification-channel`
* in `.env` define `NEXMO_KEY` and `NEXMO_SECRET` as provided by the service (register on their website)
* in `config/services.php` you can define the sender telephone number in `nexmo['sms_from']` variable (see laravel docs) 

```php
'nexmo' => [
    'sms_from' => '15556666666',
],
```

* in the notification class, add `nexmo` in the array returned by `via` method

```php
public function toNexmo($notifiable)
{
    return (new NexmoMessage)
                ->content('Your SMS message content');
}
```

* you can set a different number on each request adding `...->from($number)`
* in the model, bind the user phone number to nexmo adding the following method:

```php
public function routeNotificationForNexmo($notification)
{
    return $this->phone_number;
}
```
