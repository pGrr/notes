Composer
========  

Install
-------

* [see docs](https://getcomposer.org/doc/00-intro.md#installation-linux-unix-macos)

```sh
#!/bin/bash
# Download, install and verify composer
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === 'e0012edf3e80b6978849f5eff0d4b4e4c79ff1609dd1e613307e16318854d24ae64f26d17af3ef0bf7cfb710ca74755a') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
# Now a composer.phar file should be available locally
# to install globally run:
mv composer.phar /usr/local/bin/composer
```

Choosing and installing libraries
---------------------------------

* Choose a library from: 
  * [packagist.org](https://packagist.org/), by coping its `<package-string>`
  * or from a user-defined url (e.g. from github), custom repositories, url from which to fetch a zip file containing source files, etc. ([see docs](https://getcomposer.org/doc/02-libraries.md#publishing-to-a-vcs))
* Install dependencies locally, either: 
  * modifying `composer.json` directly and then running `composer install`, OR
  * `composer install <package-string>`

File structure
--------------

* `composer.json` - holds all the composer configuration and lists dependencies and their version constraints ([see docs](https://getcomposer.org/doc/articles/versions.md))
* `composer.lock`
  * contains the actual version currently used for this project, which should be the same for all the team (whereas `composer.json` contains a range of acceptable versions). This is crucial for avoiding libraries' conflicts.
  * when `composer install` is run
    * if `composer.lock` is present, it reads from it the versions to be used
    * if `composer.lock` is not present, `composer.json` is used: the latest available compatible with the range specified versions are installed, and `composer.lock` is created accordingly
* `vendor` folder
  * is created by `composer install`
  * contains:
    * `autoload.php` - script that manages the auto-loading of classes, to be included in each of our php file
    * `composer` directory - contains files with installation and autoloading info
    * many `<package-name>` directories - one for each dependency, containing the source code

Version constraints (composer.json)
-----------------------------------

* `X.Y.Z` or `vX.Y.Z` - X major, Y minor, Z patch - see [Semantic Versioning](https://semver.org/)
* `*` - wildcard to specify a range (e.g. `1.2.*` - having `*` in patch version is a best practice: we get security fixes)
* `>`, `<`, `>=`, `<=`, `!=`, `&&`, `||`, etc - to specify ranges, e.g. `>=1.0<1.1||>=1.2` means "from 1.0 onwards, skipping the 1.1"
* `-` (dash) - to specify ranges, e.g. `1.0.0 - 2.1.0` equals `>=1.0.0<=2.1.0`
* `~` (tilde) - all patch versions greater then (e.g. `~1.2.3` equals to `>=1.2.3<1.3.0`)
* `^` (caret) - all minor versions greater then (e.g. `^1.2.3` equals to `>=1.2.3<2.0.0`) - this is most commonly used, as it ensures backward-compatibility
* `-dev`, `-patch`, `-alpha`, `-beta`, `-RC` - other options
* [semver.mwl.be](semver.mwl.be) - You can see the default version constraint which Composer would add to your `composer.json` file for a given package, and then adjust it to your need (with a feedback on what version you actually selects)

Autoload (vendor/autoload.php)
------------------------------

* `require __DIR__ . '/vendor/autoload.php';` (in a php file) - autoloading: makes it possible to use the libraries inside the vendor directory without an explicit `require` for each of them (it uses `spl_autoload_register` php function internally)
* you can add your own code to the autoloader by adding an `autoload` field to composer.json. [See more](https://getcomposer.org/doc/01-basic-usage.md#autoloading)
  * `psr-4` naming and path convention by PHP-FIG is recommended (see [more](https://www.php-fig.org/psr/psr-4/)): java-like: one class per file, filename is the same of the class, path reflects the namespace, etc
  * `composer dump-autoload --optimize` - updates the autoload file `vendor/autoload.php` with current `composer.json`

E.g.:

* the src directory is in your project root, on the same level as vendor directory is
* there is a `src/Foo.php` file containing an `Acme\Foo` class.
* Then the following should be added to `composer.json`:
  ```json
  // composer.json
  {
      "autoload": {
          "psr-4": {"Acme\\": "src/"}
      }
  }
  ```
* Or you can add it directly in a single file (e.g. in a test suite), with:
  ```php
  $loader = require __DIR__ . '/vendor/autoload.php';
  $loader->addPsr4('Acme\\Test\\', __DIR__);
  ```
* `php composer.phar dump-autoload` must be run to update the autoload after each change
