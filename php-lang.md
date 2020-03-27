Best Practices (PHP-FIG PSRs)
=============================

See [php-fig](https://www.php-fig.org/psr/psr-1/).

Opening tags
============

* `<?php ... ?>`
* `<?= ... ?>` (implicit echo) 

Comments
========

* `//`, `#` - single-line
* `/* ... */` - multi-line

Data types
==========

* dynamic typing
* `boolean, integer, double, string, array, object, resource, NULL, NAN, INF`

Variables
=========

* case-sensitive
* no declaration needed
* temporary: memory is freed at the end of the script-file execution
* `$myVariable`, `$_myVariable`
* `gettype($myVar)` - returns the current type
* `vardump($myVar)` - prints variable's content in the console (useful for quick debugging)

Constants
=========

* `define('MY_CONSTANT', 'myValue');` 
* `echo MY_CONSTANT`
* `echo constant('MY_CONSTANT`)
* The language provides some magic-constants (double-underscored, e.g. `__FILE__`)  

Operators
=========

* All c-like operators: `+`, `-`, `*`, `/`, `%`, `++`, `--`, `+=`, etc 
* `**` - Power
* `.` - String concat
* `==`, `!=`, `<>`, `<`,`>`,`<=`, `>=` - compare content
* `===`, `!==` - compare content and type (no type inference, to be preferred). See [types comparison](https://www.php.net/manual/en/types.comparisons.php)
* `$a <=> $b` - (php7) spaceship-operator: 
  * 1 if `$a>$b`
  * 0 if `$a==$b`
  * -1 if `$a<$b`
* `$a ?? $b ?? $c` - (php7) null-coalesce: returns the first non-null value or null if all values are null. Useful for assigning default values, e.g.:
  * `$myVar = $arg ?? $default`;
* `(expr1) ? (expr2) : (expr3)` - ternary operator
* `&&`, `and`, `||`, `or`, `!`, `xor`

Integers
========

* Int size may be 32b or 64b depending on the operating system used: 
  * Constants: `PHP_INT_MAX`, `PHP_INT_MIN`, `PHP_INT_SIZE`
* Formats (let N be the number)
  * `N` - decimal
  * `0bN` - binary
  * `0N` - octal
  * `0xN` - hex

Double
======

* `1.234` - 1.234
* `1.2e3` == `1.2E3` == `1.2*10**3` - 1200
* Math functions: `sqrt`, `sin`, `cos`, `log`, `round`, `floor`, `ceil`, etc

Strings
=======

* A strings is a `array` of characters where keys are char's indexes
  * `$myString[1]` - second character
  * `$myString[-1]` (php7) - last character
* `'mytext'` - pure text
* `"$myVar \n"` - interpreted string: variables, expressions and escape sequences are evaluated
  * `\` - deactivates evaluation of the following symbol, e.g. `"\$notExpanded"`
* `.` - String concat
* API functions: 
  * `strlen`,`strpos`, `substr`, `strtoupper`, `strtolower`, ...etc
  * `printf` - c-like
  * `sprintf` - c-like

Heredocs and Nowdocs
====================

```php
$myHereDoc = <<<MYIDENTIFIER
My multiline text
which is evaluated: 
my variable is $variable.
MYIDENTIFIER;

$myNowDoc = <<<'MYIDENTIFIER'
My multiline text
which is NOT evaluated.
MYIDENTIFIER;

// from php 7.3
// the ending identifier can define the base indentation:
$myHereDoc = <<<MYIDENTIFIER
    My multiline text
    which is evaluated: 
    my variable is $variable.
    MYIDENTIFIER;
```

Arrays
======

* Arrays are actually ordered maps (keys can be integers or strings, values can be any type).
  * `unset($myArray[1])` removes the second element, but doesn't move the other ones nor change their index, there just won't be an index "1" in `$myArray` anymore (and there will be a "hole" in the sequence instead)
  * Arrays are iterable with: `current`, `key`, `prev`, `reset`, `end` 

```php
// Declaration:
$myIndexedArray = array('One', 'Two', 'Three'); // indexed
$myIndexedArray = ['One', 'Two', 'Three']; // equals the above
$myAssociativeArray = array( // associative
    'One' => 1,
    'Two' => '2',
);
$myAssociativeArray = [ // equals the above
    'One' => 1,  
    'Two' => '2', // value can be any type
];
define('MY_CONSTANT_ARRAY', ['One', 'Two']); // constants (php7)
// Accessing elements:
echo $myArray[0]; // One
// Add an element:
$myArray[3] = 'Four'; // specifying the index
$myArray[] = 'Five'; // append at the end
// Remove an element
unset[$myArray[1]]; // 0=>'One', 2=>'Three`, 3=>'Four', 5=>'Five'
count($myArray); // array size
// list: convenient syntax to assign array values to variables
// e.g. the following assigns the first to $a, the third to $b
list($a, ,$b) = $myArray; 
[$a, ,$b] = $myArray; // (php7) equals the above
```

Flow-control
============

* All c-like: `if-elseif-else`, `switch`, `while`, `do-while`, etc
* `foreach (array_expression as $value)`
* `foreach (array_expression as $key => $value)`
* `break` - interrupt cycle
* `continue` - go to next iteration

Functions
=========

* `function foo(string $arg1, int $arg2) {...} : string`
* argument typing is optional, may be applied to arguments and return values
* `declare(strict_types=1);` - enables type-checking (default is 0: coercive)
* `$arg=default_values` - default values (last arguments)
* `...$args` - variadic (varargs) (last argument)
* `?$arg`, `return ?$value` - nullable: can be a value or null
* By default arguments are passed by-value
* `&$arg` - pass argument by reference, e.g.: `foo(&$var)`
* `function &foo() {...}` - return value by reference

Anonymous functions = Closures
------------------------------

```php
$average = function(array $values) {
    return array_sum($values / count($values));
}
echo $average([1, 2, 3, 4, 5, 6]);
// Anonymous functions can use external variables with "use"
$foo = function(array $options) use ($average) {
    return $average($options);
}
echo $foo([1, 2, 3, 4, 5, 6]);
```

Date
====

```php
date('D'); // day
date('Y'); // year
```
