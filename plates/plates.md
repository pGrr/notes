# Template-engines HTML

* Separating php code (=logic) from html (=presentation) is advised, e.g. by:
  *  writing html code with php-tags just to echo variables `<?= $variable ?>`
  *  writing a html code with markers, such as `%s` and then using `printf`
  *  Using template-engine libraries, such as `Plates`, `Blade`(laravel), `Twig` (simfony), `Dwoo`, `Mustache`, `Smarty`, `Zend-View`, ecc
* Also, html code should be escaped in order to avoid possible execution of malicious code (template-engines automatically take care of that)

# Plates

* [See docs](http://platesphp.com/)
* Developed by [thephpleauge (the league of extraordinary packages)](https://thephpleague.com/)
* It's a __non-compiled__ template-engine, i.e. a template system, not a template-language: manages markers using plain php variables and does not need a meta-language to be compiled and translated (contrary to most of the widely used template-engines such as Blade, Twig, Smarty, etc)

# Install

* `composer require league/plates`
* Be sure to also include your Composer autoload file in your project:
  * `require 'vendor/autoload.php';`

# Engine

* stores the environment configuration, functions and extensions
* decouples your templates from the file system and other dependencies

```php
/**
 * Create the Plates' Engine instance
 */
$templates = new League\Plates\Engine();
// set default template folder
// (if a template is missing Plates will look here 
// for a template with the same name)
$templates = new League\Plates\Engine('/path/to/templates');
// set file extension (e.g. '.tpl')
$templates = new League\Plates\Engine('/path/to/templates', 'tpl');
```

# Folders

```php
/**
 * Add folders to the engine, with a given namespace
 */
$templates->addFolder('admin', '/path/to/admin/templates'); // admin namespace
$templates->addFolder('emails', '/path/to/email/templates'); // emails namespace
// You can enable fallback to default theme (for missing templates), passing 'true'
$templates = new \League\Plates\Engine('/path/to/default/theme');
$templates->addFolder('theme1', '/path/to/theme/1', true);
```

# Templates

## Render

```php
/**
 * Render a template
 */
$templates->render('home.php'); // file
$this->templates->render('emails::welcome', [ // namespace::template
    'title' => 'Welcome',
    'name' => 'John',
    'message' => 'Welcome to Plates engine!',
]);
$templates->render('partials/header'); // sub-directory of namespace
$template = $templates->make('emails::welcome', [ // make, then render
    'name' => 'Jonathan'
]);
$template->data(['name' => 'Jonathan']); // assign data in any moment
$template->render();
echo $template; // using template's toString()
```

## Passing data (arguments)

* Variables can be assigned as an associative array in `render`, `make`, `insert`, `fetch`, or in any moment with `..->data()`
* Inside the template, the keys of the array can be used as normal php variables

```php
/**
 * Passing data to templates (arguments)
 */
$this->templates->render('emails::welcome', [ // render
    'name' => 'Johnatan',
]);
$template = $templates->make('emails::welcome', [ // make
    'name' => 'Jonathan'
]);
$template->data(['name' => 'Jonathan']); // or in any moment

/**
 * Use the data inside the template
 */
<p>Hello <?=$this->e($name)?></p>

/**
 * Pre-assign data to templates
 */
$templates->addData(['name' => 'Jonathan'], 'emails::welcome'); // to one template
$templates->addData(['name' => 'Jonathan'], ['login', 'template']); // to many templates
$templates->addData(['name' => 'Jonathan']); // to all templates
```

## Escaping

```php
// shorthand for php's htmlspecialchars()
<h1>Hello, <?=$this->escape($name) // or ?></h1>
<h1>Hello, <?=$this->e($name)?></h1>

// It's VERY important to always double quote HTML attributes 
// that contain escaped variables
<!-- BAD -->
<img src="portrait.jpg" alt='<?=$this->e($name)?>'>
<!-- BAD -->
<img src="portrait.jpg" alt=<?=$this->e($name)?>>
<!-- Good -->
<img src="portrait.jpg" alt="<?=$this->e($name)?>">
```

## Template utils functions

```php
/**
 * Check if a template exists
 */
if ($templates->exists('articles::beginners_guide')) { // from engine
    // It exists!
}
if ($template->exists()) { // from template
    // It exists!
}

/**
 * Get a template path
 */
$path = $templates->path('articles::beginners_guide'); // from engine
$path = $template->path(); // from template
```

# Layouts and Nesting

## Nesting (=include)

```php
<?php // insert (automatically echo) ?>
<?php $this->insert('partials/header', ['name' => 'Jonathan']) ?>
<?php //or ?>
<?php $this->insert('partials::header', ['name' => 'Jonathan']) ?>
<?php // fetch (needs to be echoed) ?>
<?=$this->fetch('partials/header')?>
```

## Layouts

* `$this->layout('template')` - (inside a template) - the current template will use the specified one as it's skeleton (layout), e.g. you can define a site wide layout with header, footer, and so on
  * the layout template contains a `<?=$this->section('content')?>`, this is where the content template will be rendered

```php
// syntax:
<?php $this->layout('template') // in "child" template ?>
<?php $this->layout('shared::template') ?>
<?php $this->layout('template', ['title' => 'User Profile']) ?>
<?=$this->section('content') // in layout template?>
```

Here is an example with a main site layout, a blog layout and a blog article template: 

```php
// template.php (site layout)
?>
<html>
    <head>
        <title><?= $this->e($title) ?></title>
    </head>
    <body>
        <?=$this->section('content')?>
    </body>
</html>

// blog.php  (blog-layout)
?>
<?php $this->layout('template') ?>
<h1>The Blog</h1>
<section>
    <article>
        <?=$this->section('content')?>
    </article>
    <aside>
        <?=$this->insert('blog/sidebar')?>
    </aside>
</section>

// blog-article.php (blog article template)
?>
<?php $this->layout('blog', ['title' => $article->title]) ?>
<h2><?=$this->e($article->title)?></h2>
<article>
    <?=$this->e($article->content)?>
</article>
```

## Sections

* `<?=$this->section('content')?>` is special because it will output all of the child template (see above)
* But you can declare custom sections and re-use them in the same templates or in their layouts:

```php
//profile.php ("child" template)
<?php $this->layout('template', ['title' => 'User Profile']) ?>

<?php $this->start('page') // declare page template ?>
    <h1>Welcome!</h1>
    <p>Hello <?=$this->e($name)?></p>
<?php $this->stop() ?>

<?php $this->start('sidebar') // declare sidebar template ?>
    <ul>
        <li><a href="/link">Example Link</a></li>
        <li><a href="/link">Example Link</a></li>
        <li><a href="/link">Example Link</a></li>
        <li><a href="/link">Example Link</a></li>
        <li><a href="/link">Example Link</a></li>
    </ul>
<?php $this->stop() ?>

// template.php (layout template)
<html>
    <head>
        <title><?=$this->e($title)?></title>
    </head>
    <body>
        <img src="logo.png">
        <div id="page">
            <?=$this->section('page') // use page template?>
        </div>
        <div id="sidebar">
            <?php if ($this->section('sidebar')): // check if defined ?>
                <?=$this->section('sidebar') // use sidebar template?>
            <?php else: ?>
                <?=$this->fetch('default-sidebar')?>
            <?php endif ?>
        </div>
    </body>
</html>
```

* In the example above, instead of using if-else for sidebar section, you could set the default fallback template directly:

```php 
<?=$this->section('sidebar', $this->fetch('default-sidebar') ?>
```

# File extensions

* Default file extension is `.php`, but any extension can be set

```php
/**
 * Set a file extension (eventually), e.g. '.tpl'
 */
$templates = new League\Plates\Engine('/path/to/templates', 'tpl'); // or:
$templates->setFileExtension('tpl');
```

# Functions

```php
/**
 * You can register and re-use one-off functions
 * or group of functions (see Extensions)
 * @see http://platesphp.com/v3/templates/functions/
 * @see http://platesphp.com/v3/engine/extensions/
 */
$templates = new \League\Plates\Engine('/path/to/templates');
$templates->registerFunction('uppercase', function ($string) {
    return strtoupper($string);
});
<h1> Hello <?=$this->e($this->uppercase($name))?> </h1>
// also php built-in functions can be used
<p>Welcome <?=$this->batch($name, 'strip_tags|strtoupper|escape')?></p>
<p>Welcome <?=$this->e($name, 'strip_tags|strtoupper')?></p>
<?=$this->batch('Jonathan', 'escape|strtolower|strtoupper')?>
```

# Additional functionalities (Extensions)

```php
/*
 * Load any additional extensions (advanced)
 * Built in Extensions are available, or you can add your own
 */
$templates->loadExtension(new League\Plates\Extension\Asset('/path/to/public'));
```

# Best practices

* Always use HTML with inline PHP. Never use blocks of PHP.
* Always escape potentially dangerous variables prior to outputting using the built-in escape functions. More on escaping here.
* Always use the short echo syntax (<?=) when outputting variables. For all other inline PHP code, use full the <?php tag. Do not use short tags.
* Always use the alternative syntax for control structures, which are designed to make templates more legible.
* Never use PHP curly brackets.
* Only ever have one statement in each PHP tag.
* Avoid using semicolons. They are not needed when there is only one statement per PHP tag.
* Never use the use operator. Templates should not be interacting with classes in this way.
* Never use the for, while or switch control structures. Instead use if and foreach.
* Avoid variable assignment.

Syntax example:

```php
<?php $this->layout('template', ['title' => 'User Profile']) ?>

<h1>Welcome!</h1>
<p>Hello <?=$this->e($name)?></p>

<h2>Friends</h2>
<ul>
    <?php foreach($friends as $friend): ?>
        <li>
            <a href="/profile/<?=$this->e($friend->id)?>">
                <?=$this->e($friend->name)?>
            </a>
        </li>
    <?php endforeach ?>
</ul>

<?php if ($invitations): ?>
    <h2>Invitations</h2>
    <p>You have some friend invites!</p>
<?php endif ?>
```