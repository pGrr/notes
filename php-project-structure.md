
Project file structure
======================

This is an example of a well-structured project:

* `config` - configuration files, e.g. database credentials, log files paths, etc
* `data` - data-related informations, such as a SQLite database, SQL files, database structure files, 
  * `cache` - cache files
* `src` - else named `app`, contains source files (with their sub-directory structure)
* `public` - contains everything that is exposed to the web-server: keeping all public files in a single directory makes it easy to manage security and efficiency 
  * `css`
  * `img`
  * `js`
  * `index.php` - this is the front controller of the app (front-controller is an architectural pattern in which there is a signle point of input for the whole app: in a web app, all HTTP requests will be received by this file and processed)
* `test` - automated tests
* `README.md` - presentation, installation and usage tutorial (markdown)
* `LICENSE.md` - license (markdown)
* `phpunit.xml.dist` - phpunit test-environment config file
* `.gitignore` - should contain composer's `vendor` directory, `data/cache` and all other temporary or re-generable files or directories (all what should not be tracked by git)
* `composer.json` - composer's configuration file

Choosing your libraries
======================= 

Consider:

* popularity (e.g. analyze number of forks, stars, contributors, commits, issues, pull-requests and last-update on github)
* doumentation
* community
* support
* functionalities
* license

License
------- 

* `LICENSE` - file in the root directory of the project
* `GPL` - you can use, modify and sell but you must distribute all your source code under the GPL license
* `MIT` - you can use, modify and sell but you must include the copyright and the library's license

