OOP
===

Basics
------

* By convention (best-practice), one class per file.
* `class` - java-like
* `public/protected/private` (default is public) - java-like
* objects are copied by reference (not value as variables)
  * `$b = clone $a` - copies by value (creates an object clone)
* `instanceof` - java-like
* a variable can hold a class: `$class = 'User'; $user = new $class();`

Magic methods
-------------

* `public function __construct($arg1, $arg2, ...$args)` - constructor 
* `public function __destruct($arg1, $arg2, ...$args)` - constructor 
* `public function __toString() : string` - toString
* See other [magic methods](https://www.php.net/manual/en/language.oop5.magic.php)

Instance entities
-----------------

* `$o = new MyObject()` - java-like
* `$this` - java-like
* `$o->oMethod()` - java-like

Static entities
---------------

* `static` - java-like, but accessed by `::` instead of `->` and `self` or `static` or `parent` instead of `$this`
* `self::staticMethod();` - self refers to the class that contains the actual code (might be the parent class of the currently used class)
* `static::staticMethod();` - Late Static Binding - always refers to the currently used class (resolve the problem caused by `self`)
* `MyClass::staticMethod();` - always refers to MyClass (so it is more specific but less reusable in child-classes (which extends the current one), not to be abused)
* `parent::staticMethod();` - refers to the parent class of the current class
* All the above syntaxes are valid for properties too

Inheritance, abstract, final, interface
---------------------------------------

* `extends` - java-like
  * `parent::__construct($arg1, $arg2)` - parent = super 
  * overriding by signature - java-like
* `abstract` - class and methods - java like (but no default implementations)
* `final` - class and methods - java-like
* `interface`, `extends`, `implements` - java-like
  * optional arguments can be added to interfaces' signatures (but not to be abused)

Anonymous classes
-----------------

* `new class implements MyInterface {...}` - anonymous classes - java-like but with different syntax `new  class`

Traits
------

* `trait` - class-like piece of code that represents a functionality to be reused inside many classes
* `use MyTrait` (inside a class) - includes the code of the trait inside the class 

```php
// Basic trait example
trait UserName {
    protected $name = '';

    public function getName() : string {
        return $this->name;
    }

    public function setName(string $name) {
        $this->name = $name;
    }
}
class Developer {
    // UserName code is included here
    use UserName; 
}

// More advanced traits' features
trait A {
    public function hello() {
        return 'Hello';
    }
    public function me() {
        return 'A';
    }
}
trait B {
    public function me() {
		 return 'B';
     }
}
class C {
    use A, B { // multiple traits can be included in a class
        // conflicts must be resolved using insteadof
        B::me insteadof A; 
        // trait visibility can be redefined
        me as protected; 
        // trait functions/properties names can be redefined
        hello as private myPrivateHello; 
    }
}
```

Namespaces
==========

* `namespace Zend\Form\Element\Button;` - declare the namespace to be used in current file
* by convention (best-practice), class Button will be in `Zend/Form/Element` directory

Require (import)
================

* `require($file)` - load file (like java import)
* `require_once($file)` - loads only once

Autoloading
-----------

```php
/*
 * Basic autoloading implementation 
 * (composer use an optimized and more complex version of this)
 * place this in a file autoload.php and require it in every
 * php file you write: you'll get automatic class-loading
 * (spl_autoload_register specifies how to load a class in its
 * first use). 
 */
spl_autoload_register(function($class) {
    require_once __DIR__ . '/' . str_replace('\\', '/', $class) . '.php';
});
```
