# Localization

* language strings are in `resources/lang`, there should be a subdirectory for each language supported by the application: e.g. `en`, `it`, etc
* each folder can contain multiple language files, containing an associative array of strings, e.g.

```php
// resources/lang/en/messages.php
return [
    'welcome' => 'Welcome to our application'
];
```

## Config

* default locale setting is in `config/app.php` (where you can also set a "fallback locale" `'fallback_locale' => 'en'`), or can be changed at runtime with `App::setLocale($locale)`
* you can get or check the locale with `App:getLocale` and `App::isLocale` facade methods 

## Alternative: translation Strings As Keys

* For applications with heavy translation requirements, defining every string with a "short key" can become quickly confusing when referencing them in your views
* Laravel also provides support for defining translation strings using the "default" translation of the string as the key.
* Translation files that use translation strings as keys are stored as JSON files in the resources/lang directory. For example, if your application has a Spanish translation, you should create a `resources/lang/es.json` file:

```json
{
    "I love programming.": "Me encanta programar."
}
```

## Retrieving Translation Strings

* You may retrieve lines from language files using the `__` helper function. The `__` method accepts the file and key of the translation string as its first argument. For example, let's retrieve the welcome translation string from the `resources/lang/messages.php` language file:

```php
echo __('messages.welcome');
// or
echo __('I love programming.'); // for translation strings as keys

// blade syntax
{{ __('messages.welcome') }}
@lang('messages.welcome') // this doesn't escape output
```

* If the specified translation string does not exist, the `__` function will return the translation string key. So, using the example above, the `__` function would return `messages.welcome`

## Strings placeholders (parameters)

* If you wish, you may define placeholders in your translation strings with a `:` 
* If your placeholder contains all capital letters, or only has its first letter capitalized, the translated value will be capitalized accordingly

```php
// in language file:
'welcome' => 'Welcome, :name'
'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle

// usage:
echo __('messages.welcome', ['name' => 'dayle']);
```

## Pluralization

* By using a "pipe" character, you may distinguish singular and plural forms of a string
* You may even create more complex pluralization rules which specify translation strings for multiple number ranges
* After defining a translation string that has pluralization options, you may use the `trans_choice` function to retrieve the line for a given "count"
* If you would like to display the integer value that was passed to the trans_choice function, you may use the :count placeholder

```php
// in the language file:
'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',
'apples' => 'There is one apple|There are many apples',
// or
'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many'
'apples' => '{0} There are none|{1} There is one|[2,*] There are :count'

// usage:
echo trans_choice('messages.apples', 10);
echo trans_choice('time.minutes_ago', 5, ['value' => 5]);
```

## Overriding vendor packages lang files

* Some packages may ship with their own language files. Instead of changing the package's core files to tweak these lines, you may override them by placing files in the `resources/lang/vendor/{package}/{locale}` directory.
* For example, if you need to override the English translation strings in `messages.php` for a package named `skyrim/hearthfire`, you should place a language file at: `resources/lang/vendor/hearthfire/en/messages.php`
* Within this file, __you should only define the translation strings you wish to override__
* Any translation strings you don't override will still be loaded from the package's original language files.