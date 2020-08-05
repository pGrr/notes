# Email config

* `.env` contains the parameters, `config/mail.php` loads parameters from there
* multiple drivers are available: you can change the driver to test functionalities (the interface will remain the same), e.g.:
  * `log` - will save it in `storage/logs` folder
  * `smtp`, ...
* there is a global mail address which will be used by default, defined in `config/mail.php` and editable in `.env` (if not present there it can be added)

```php
// via Mail facade
// text mail
Mail::raw('Hello!!', function ($message) {
    $message->to('target@email.it')
        ->subject('hello message');
});
// redirect with a message flashed into the session
// (for one request only)
return redirect('/contact')
        ->with('Email sent!');

// in /contact view
@if (session('message'))
    <p>{{ $session('message') }}<p>
    // since the message was one-request only,
    // refreshing the page it will disappear
@endif
```

# Testing emails with MAILTRAP

* You can use __MAILTRAP__ for testing purposes:
  * `MAIL_DRIVER=smtp`
  * `MAIL_PORT=2525`
  * `HOST=smtp.mailtrap.io`
  * `MAIL_USERNAME=..`, `MAIL_PASSWORD=..` - the one provided by mailtrap

# HTML emails

* `php artisan make:mail`
* creates a mail class inside `app/mail`
* inside the method `build` you can return a blade view (as for a a normal web page)
  * e.g. if you have a `/resources/views/mail/contact-me.blade.php` template, your `build` method will contain `¶eturn $this->view('email.contact-me')`

```php 
// app/mail/ContactMe.php

class ContactMe extends Mailable 
{
  // ...
  public function build() {
    return $this->view('email.contact-me')
              ->subject('My subject');
  }
}


// usage
Mail::to(request('email'))->send(new ContactMe());
```

# Markdown mails

* provide a way to send an html email pre-styled, from a provided markdown-view 
* `markdown` method instead of `view`
* You can have all the boiler plate ready by simply running:
  * `php artisan make:mail --markdown=email.contact-me`

```php 
// app/mail/ContactMe.php

class ContactMe extends Mailable 
{
  // ...
  public function build() {
    return $this->markdown('email.contact-me')
              ->subject('My subject');
  }
}
// usage
Mail::to(request('email'))->send(new ContactMe());

// resources/views/email/contact-me.php

```
```md
@component('mail::message')

# My mail

* the double colons say that the view ('component')
* belongs to the vendor directory, instead of
* the user's view directory

@component('mail::button', ['url' => 'url.com']);
@endcomponent

@endcomponent
```

# Customizing styles

* `php artisan vendor:publish` - lets you publish any publishable vendor assets into your project resources directory in order to tweak them
  * this will publish all publishable assets, but you can specify one of them by using its tag:
    * e.g. `--tag=laravel-mail`
* you can specify your own css file as default in `config/mail.php`, in the `markdown` section, where you can specify the theme name (i.e. the name of the css file) and its path
* you can also switch the theme on the fly why sending the email (check the docs)