PHPUnit
=======

* `composer require --dev "phpunit/phpunit=6.2.*"` - install locally via composer (will be added to `require-dev` in `composer.json`)
* `vendor/bin/phpunit` - execute phpunit: will execute all tests it can found or a testsuite if `phpunit.dist.xml` exists (or show command instructions if no tests are found)
* Define test classes, i.e. which extends `PHPUnit\Framework\TestCase`:
  ```php
  use PHPUnit\Framework\TestCase;
  class MyClassTest extends TestCase {

    public function setUp() {
      // PHPunit will run this before each test function
    }

    public function testMyFunction1() {
      $this->assertTrue($shouldBeTrue);
      // ...this is a test
    }

    public function testMyFunction2() {
      $this->assertTrue($shouldBeTrue);
      // ...this is another test
    }

    public function tearDown() {
      // PHPUnit will run this after all test functions
    }
  }
  ```
* many assert functions, [see the docs](https://phpunit.readthedocs.io/en/9.0/assertions.html) 
* `phpunit.dist.xml` (in root project's directory) - is the xml config file to define test-suite: PHPUnit will use this if it exists (defines a test directory, a bootstrap file to initialize the test-environment e.g. `autoload.php` or a file to initialize a test-database, etc)
  ```xml
  <phpunit bootstrap="./vendor/autoload.php">
    <testsuites>
      <testsuite name="AppTest">
        <directory>./test</directory>
      </testsuite>
    </testsuites>
  </phpunit>
  ```
* `PHP_CodeCoverage`, together with `Xdebug` or `phpdbg` can provide code coverage statistics ([read the docs](https://phpunit.readthedocs.io/en/9.0/code-coverage-analysis.html)). The directories to be included must be specified inside `phpunit.xml.dist` after "testsuites" tag (e.g. with the following all `.php` files inside the `src` folder will be included in the code coverage analysis):
  ```xml
  <filter>
    <whitelist processUncoveredFilesFromWhitelist="true">
      <directory suffix=".php">./src</directory>
    </whitelist>
  </filter>
  ```
  * `vendor/bin/phpunit --coverage-text` - will start the code coverage, or else:
  * `phpdbg -qrr vendor/bin/phpunit --coverage-text` - if using `phpdbg
  * A class or method can be excluded by placing `@codeCoverageIgnore` in the comment above it:
    ```php
    /*
    * @codeCoverageIgnore
    */
    ```


