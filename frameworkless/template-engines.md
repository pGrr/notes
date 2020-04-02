# Template-engines HTML

* Separating php code (=logic) from html (=presentation) is advised, e.g. by:
  *  writing html code with php-tags just to echo variables `<?= $variable ?>`
  *  writing a html code with markers, such as `%s` and then using `printf`
  *  Using template-engine libraries, such as `Plates`, `Blade`(laravel), `Twig` (simfony), `Dwoo`, `Mustache`, `Smarty`, `Zend-View`, ecc
* Also, html code should be escaped in order to avoid possible execution of malicious code (template-engines automatically take care of that)

## Plates

* [See docs](http://platesphp.com/)
* Developed by [thephpleauge (the league of extraordinary packages)](https://thephpleague.com/)
* It's a __non-compiled__ template-engine, i.e. a template system, not a template-language: manages markers using plain php variables and does not need a meta-language to be compiled and translated (contrary to most of the widely used template-engines such as Blade, Twig, Smarty, etc)
* `composer require league/plates`

Here is an example with a main site layout, a blog layout and a blog article template: 

// template.php (site layout)
```php
?>
<html>
    <head>
        <title><?= $this->e($title) ?></title>
    </head>
    <body>
        <?=$this->section('content')?>
    </body>
</html>
```

* `$this->section('content')` - when another template will use `template.php` as its layout, this line will render that template's code (so `template.php` can define a skeleton with header, footer, etc to be used for all pages)

// blog.php  (blog-layout)
```php
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
```

* `$this->layout('template')` - this means that this template should be rendered inside the `template` layout (exactly where `$this->section('content')` is placed)

// blog-article.php (blog article template)
```php 
?>
<?php $this->layout('blog', ['title' => $article->title]) ?>
<h2><?=$this->e($article->title)?></h2>
<article>
    <?=$this->e($article->content)?>
</article>
```

* `$this->layout('template')` - this means that this template should be rendered inside the `blog` layout (exactly where `$this->section('content')` is placed): the blog layout will also be rendered inside `template` layout, and so on: templates "skeletons" are inherited and a hierarchy of templates can be defined (general to specific sections of html, thus reusing code and avoiding repetition) 
