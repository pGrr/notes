# PHP Security

* See [OWASP](https://www.owasp.org)

# Injection - Never trust the user 

* Injection is when someone injects code in your app, which then is executed
  * XSS - Cross Site Scripting
    * malicious code is injected into the output of our application (usually javascript, using `<script>` tags), which then is executed as if it were part of our app: 
      * cookie and seission-hijacking, 
      * external HTTP requests holding sensible data, 
      * phishing website redirections, 
      * malware installation into the browser, 
      * DOM manipulation (e.g. clickjacking), 
      * ...etc 
    * To prevent this:
      * Validate input
      * Sanitize input
      * escape output
      * CSP - Content Security Policy
  * SQL Injection
    * input with malicious SQL code (e.g. `\';DROP TABLE USERS;--`) that modify, destroy or steal our data
    * To prevent this 
      * prepared statements (i.e. filtering and sanitizing SQL input)
      * minimum permissions
  * Directory traversal
    * When a resource or a url is submitted by the user, paths with `.` and `..` or `\0` characters may be used to navigate to other folders and request other files (e.g. other php files or even all the way to `../../../../etc/passwd`)
    * Use basenames
  * ...etc

## Filter (Sanitize & Validate) input

* `filter_input ($type , $variable_name, $filter)`
  * `$type` - one of: `INPUT_GET`, `INPUT_POST`, `INPUT_COOKIE`, `INPUT_SERVER`, or `INPUT_ENV`.
  * `$variable_name` - name of the variable to get (e.g. same name as in `$_POST[$name]`, or `$_GET[$name]`, etc)
  * `$filter` - the filter to apply, e.g. `FILTER_VALIDATE_EMAIL`, `FILTER_SANITIZE_STRING` etc ([see available filters](https://www.php.net/manual/en/filter.filters.php))
  * e.g.
    * `$username = filter_input(INPUT_POST, 'username', FILTER_VALIDATE_EMAIL);` - false if not valid email
    * `$password = filter_input(INPUT_POST, 'password', FILTER_SANITIZE_STRING);` - returns the string stripped from html tags
    * ...etc
* As a rule of thumb, prefer `filter_input` when available, else use `filter_var` (the former use global array values as received by the script, whereas the latter use the current value, which may be changed)

## Escape output

* To prevent XSS (Cross-Site-Scripting), where a user inject `<script>` tags containing malicious javascript into our apps (that will execute whenever we print that content together with our app's output (e.g. the html page)), we should always escape output which was provided by a user:
  * `$comment = strip_tags($comment, '<br>');` - strips all html tags from `$comment`, only allow `<br>` (html comments and php code are always stripped, cannot be permitted)
  * `$new = htmlspecialchars("<a href='test'>Test</a>", ENT_QUOTES);` - converts html tags to entities (thus preventing their interpretation)
    * or `htmlentities()` which converts ALL html tags to entities (instead of `htmlspecialchars`, which just converts a subset: the most dangerous ones)
  * External libraries: e.g. [HTML purifier](http://htmlpurifier.org/), [zend escaper](https://github.com/zendframework/zend-escaper), etc 

## Safe database access: prepared statements and minimum DB permissions

* Prepared statements prevent SQL injection as they filter and sanitize not valid characters
* Roles of user database should be kept to a minimum (this way even if they inject code, they won't be allowed to do much damage)
  * e.g. creating a read-only user for all the actions that don't require writing (e.g. the public area)
  * e.g. creating a read-and-write user for only who needs to write (e.g. the admin dashboard area)
  * e.g. never grant drop table permission to anyone (except the main admin)

## Directory traversal: always use `basename`

* to prevent directory traversal with `.` and `..`, whenever a url is processed or a resource is requested:
  * the string should be filtered and validated as usual, and
  * one of the following method should be used :
    * `$page = basename($_GET['page']);` - returns the basename of the resource (thus preventing `.` and `..` to be used)
    * `ini_set ('open_basedir', __DIR__ . "/pages/");` (at the beginning of the script) - limits the availability of resources to the specified directory

## CSP - Content Security Policy

* CSP - Content Security Policy - are modern http headers that specify what resources can be loaded and belonging to what domains (thus blocking any cross-site request)
  * e.g. `Content-Security-Policy: script-src 'self' https://www.google-analytics.com`
  * External libraries exists: e.g. [csp-builder](https://github.com/paragonie/csp-builder)
  * See [reference](https://content-security-policy.com/)

# Sensible data protection

* To encrypt sensible data, several techniques are used in PHP
  * OpenSSL
  * Password hashing (and salting)
    * Several hash functions can be used to encrypt data, e.g. passwords

## Password hashing and salting

* A hash function $H$ takes a variable-length string $x$ and outputs a fixed-length string that is:
  * not invertible (given $H(x)$, finding $x$ is computationally difficult, thus can't be done easily)
  * doesn't have "collisions", which is: finding another value $y: H(y) = H(x)$ is computationally difficult, thus can't be done easily)
* A good hash function for security purposes should also be parametrizable as of memory and time needed for the computation (high memory usage and computational time, and eventually randomized, will prevent brute force attacks)
* Salting: a pseudo-random string is concatenated to the original string before hashing, and it is included in the output for the verification (e.g. concatenating it or saving it separately), this makes hashing more difficult to predict (e.g. will prevent the efficacy of using lists of hashes of passwords commonly used) 
* Common hash-functions are:
  * `MD5`, `SHA-1` - NOT TO BE USED: are fast (which is not good), not parametrizable and they have showed to have collisions, so they are not considered secure anymore!
  * `bcrypt` - the default used by PHP, the easiest to implement and also very good for security
  * `scrypt` - better parametrization than `bcrypt` - more difficult to implement
  * `Argon2` - better parametrization than `scrypt` - the best as of quality, more difficult to implement
* The PHP "standard" way (as of today, `bcrypt`: very good for security and also easy to use)
  * `$hash = password_hash($password, PASSWORD_DEFAULT);` - compute the hash and takes care automatically of the salting (concatenates it into the hashed string and also to the result)
    * `PASSWORD_DEFAULT` - as of today it uses bcrypt, but might be different depending on PHP version
      * `PASSWORD_BCRYPT` - can be specified to force bcrypt in every version
  * `$isValid = password_verify($password, $hash);` - verify a password using its hash (automatically takes care of the salting)
* Using `argon2` (PHP>7.2) - the most secure. Easy to implement as `bcrypt`, salting is taken care of automatically
  * `password_hash($password, PASSWORD_ARGON2I);` - we just have to specify we want to use it here
  * `$isValid = password_verify($password, $hash);`
* Using `scrypt` - better than `bcrypt`, but more difficult to implement
  * `pecl install scrypt` - the extension must be installed 
  * `extension=scrypt.so` in `php.ini` to enable it
  * `scrypt($input, $salt, $cpuCost, $ramCost, $parallelFactor, $length);`
  * We must take care of everithing (not done automatically as above)
    * salting (e.g. generating a pseudo-random value, concatenating it to the string to be hashed and to the computed hashed in a way that we will be able to extract it (or else saving it separately))
    * extracting the salt, using it to compute the same hash for the given user password in order to verify it