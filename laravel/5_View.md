# View

## Front-end

[See templated.co](https://templated.co/)

* laravel-mix is a built-in webpack wrapper
  * see `package.json` and `webpack.mix.js`
  * `npm install`
  * `npm run dev` or `npm run watch`

## Variables and expressions

* `{{ ... }}` - evaluates, automatically escapes and echoes what is inside
* `{{!!  }}` - doesn't escape (careful!) 

## Layout and sections

* `@extends('layout')` - in the "child" template (will load 'layout.blade.php')
* `@section('mysection') ... @endsection` - defines a named section
* place `@yield('mysection')` in the parent template, wherever you want the child to be displayed
* Note: you can define "hook points" in your template, and then add unique content basing on the child template, e.g. 
  * placing `@yield('head')` in the parent head tag
  * defining a `@section('head')` in the child, where we load a script which is unique for that page

## Flow control

* `@foreach ($items as $item) ... @endforeach`
* `@forelse ($items as $item) ... @empty ... @endforelse`
* `@error('fieldName') ...echo $message... @enderror`
