
# VIEWS

* Views contain the HTML served by your application and separate your controller / application logic from your presentation logic.
* Views are stored in the `resources/views` directory.
* `return view('greeting', [ 'name' => 'James' ]);` - will render `resources/views/greeting.blade.php` passing it the name parameter.
    * `return view('admin.profile', $data);` will render `resources/views/admin/profile.blade.php`
    * `view` global function equals `View::make` (see Facades - global helper functions differences)
* `View::exists('emails.customer')` checks if the view exists
* within a service provider's `boot` method you can place additional views configurations (Remember, if you create a new service provider to contain your view composer registrations, you will need to add the service provider to the providers array in the `config/app.php` configuration file)
    * You can make data available to all views by calling `Views::share($data);`  
* You can register view composers, i.e. callbacks or class methods that are called when a view is rendered (e.g. to bind default data to a view each time it is rendered)
* Laravel by default use [Blade templates](https://laravel.com/docs/6.x/blade)

## Blade syntax 

* Blade is the simple, yet powerful templating engine provided with Laravel. Unlike other popular PHP templating engines, Blade does not restrict you from using plain PHP code in your views. In fact, all Blade views are compiled into plain PHP code and cached until they are modified, meaning Blade adds essentially zero overhead to your application. 
* Blade view files use the .blade.php file extension and are typically stored in the `resources/views` directory.
* Blade provide sections to compose layouts:
    * `@section('mysection') ... @endsection` defines a section (without showing it)
    * `@section('mysection') ... @show` defines and immediately shows a section
    * `@section('mysection') @parent ... @endsection`  append section to the same-named parent's one
    * `@yield('mysection')` shows a section's content (e.g. in the parent layout)
* ...and components to define reusable view components:
    * `@component('myview', [ 'foo' => $bar ])  ... @endcomponent` renders the specified view, substituting its variable `$slot` with provided content and passing the variables array
* ...and a mechanisms to simply "include" other blade templates:
    * `@include('view.name', ['some' => 'data'])` will include the template and pass the data
    * `@includeIf('view.name', ['some' => 'data'])` only if the view exists
    * `@includeWhen($boolean, 'view.name', ['some' => 'data'])` only when condition is true (or `@includeUnless`)
* `{{ $phpContent }}` shows content (alike to `<?= htmlspecialchars($phpContent) ?>`
    * `{{!! $unescaped !!}}` will not escape with `htmlspecialchars` (e.g. to output raw html)
* `@json($phpArray)` will `json_encode` the provided array
* e.g. to seed vue components or `data-*` variables: `<example-component :some-prop='@json($array)'></example-component>`
* if you use a js framework which uses `{{ var }}`, you should escape them: `@{{ var }}` will not be elaborated by blade (it will just remove `@`), else you can surround a bigger portion of code with `@verbatim ... @endverbatim`
* Control structures are:
    * `@if (...) ... @elseif (...) ... @else ... @endif`
    * `@for (...) ... @endfor`
    * `@foreach (...) ... @endforeach` and `@forelse (...) ... @empty ... @endforelse`
    * `@while (...) ... @endwhile`
    * `@break` or `@break($condition)` to be used inside loops
    * the `$loop` object that contains info about the loop as properties (useful for nested loops, e.g. `$loop->parent`, `$loop->depth` etc)
    * `@unless (...) ... @endunless`
    * `@isset($var) ... @endisset`
    * `@empty($var) ... @endempty`
    * `@auth ... @endauth` and `@guest ... @endguest`
    * `@hasSection('mysection') ... @endif`
    * `@switch($i) @case(1) ... @break @case(2) ... @break ... @default ... @endswitch`
* `{{-- This comment will not be present in the rendered HTML --}}`
* `@php ... @endphp` is the same as `<?php .. ?>` (both valid for blade)
* you may `@push('scripts') <script src="/example.js"></script> @endpush` in multiple places and then `<head>  @stack('scripts') </head>` in the final child view to render the complete stack content (also, you can prepend with `@prepend('scripts') This will be first...  @endprepend`
* The `@inject`<!-- Head Contents --> directive may be used to retrieve a service from the Laravel service container.
* you can extend blade creating custom directives and conditional directives (see docs)

## Forms and CSRF prevention

* Any HTML forms pointing to POST, PUT, or DELETE routes that are defined in the web routes file should include a CSRF token field. Otherwise, the request will be rejected.

```html
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

HTML forms do not support PUT, PATCH or DELETE actions. So, when defining PUT, PATCH or DELETE routes that are called from an HTML form, you will need to add a hidden _method field to the form. The value sent with the _method field will be used as the HTTP request method:

```html
<form action="/foo/bar" method="POST">
    <input type="hidden" name="_method" value="PUT">
    <input type="hidden" name="_token" value="{{ csrf_token() }}">
</form>
```
...but in this case you can simply use the `@method` blade directive:

```htm
<form action="/foo/bar" method="POST">
    @method('PUT')
    @csrf
</form>
```

## Front-end

* Laravel provides basic frontend setup and scaffolding (which you can use or you can set up your own), using `npm`, `bootstrap`, `vue` (or `react`) and `webpack` (see Laravel Mix)
* `composer require laravel/ui:^1.0 --dev` provides bootstrap and vue scaffolding (you can choose react as well or use your own set of libraries). Once `laravel/ui` is installed you can generate frontend scaffolding with `artisan ui bootstrap`, `artisan ui vue`, `artisan ui react`, `artisan ui bootstrap --auth`, `artisan ui vue --auth`, etc
    * you can create your own laravel ui commands by registering them in a service provider (see docs)
* `npm install` in root project directory will install frontend dependencies, then `npm run dev` (or `npm run watch`) will process the instructions in `webpack.mix.js` file. By default it will compile `resources/sass/app.scss` into `public/css` and `resources/js/app.js` into `public/js`producing one single css file and one single js file to be loaded in each page (but this can be customized as you wish)
* it also creates by default a `resources/js/components` where vue or react components may be placed (then they have to be registered inside `resources/js/app.js`
* for more complex front end workflow configurations (e.g. browsersync, etc), [see laravel mix docs](https://laravel.com/docs/6.x/mix)

