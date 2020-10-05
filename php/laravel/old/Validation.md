# Validation

* inside the controller you can easily validate using laravel defaults, e.g.:

```php
request()->validate([
    'title' => ['required', 'min:3', 'max:10'],
    'body' => 'required',
    'tags' => 'exists:tags,id'
]);
```

* Laravel provides a global `$errors` variable containing the error data, always available (so we don't have to check if it exists)
* `@if ($errors-has('title')) {{ $errors->first('title') }} @endif`, which would be the same as:
* `@error('title') {{ $errors->first('title') }} @enderror`
* To also provide the old value inserted before the error, you can use `old()'
* `old('title')`