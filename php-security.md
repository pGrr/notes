# PHP Security

* See [OWASP](https://www.owasp.org)


# Injection - Never trust the user 

* Injection is when someone injects code in your app, which then is executed
  * XSS
  * SQL Injection 
    * prepared statements (i.e. filtering SQL)
    * minimum permissions
  * Directory traversal
    * 
  * ...etc

### Filter (Sanitize & Validate) input

* `filter_input ($type , $variable_name, $filter)`
  * `$type` - one of: `INPUT_GET`, `INPUT_POST`, `INPUT_COOKIE`, `INPUT_SERVER`, or `INPUT_ENV`.
  * `$variable_name` - name of the variable to get (e.g. same name as in `$_POST[$name]`, or `$_GET[$name]`, etc)
  * `$filter` - the filter to apply, e.g. `FILTER_VALIDATE_EMAIL`, `FILTER_SANITIZE_STRING` etc ([see available filters](https://www.php.net/manual/en/filter.filters.php))
  * e.g.
    * `$username = filter_input(INPUT_POST, 'username', FILTER_VALIDATE_EMAIL);` - false if not valid email
    * `$password = filter_input(INPUT_POST, 'password', FILTER_SANITIZE_STRING);` - returns the string stripped from html tags
    * ...etc
* As a rule of thumb, prefer `filter_input` when available, else use `filter_var` (the former use global array values as received by the script, whereas the latter use the current value, which may be changed)

### Database access: prepared statements and minimum DB permissions

* Prepared statements prevent SQL injection as they filter and sanitize not valid characters
* Roles of user database should be kept to a minimum (this way even if they inject code, they won't be allowed to do much damage)
  * e.g. creating a read-only user for all the actions that don't require writing (e.g. the public area)
  * e.g. creating a read-and-write user for only who needs to write (e.g. the admin dashboard area)
  * e.g. never grant drop table permission to anyone (except the main admin)

### Directory traversal: always use `basename`

* to prevent directory traversal with `.` and `..`, whenever a url is processed or a resource is requested:
  * the string should be filtered and validated as usual, and
  * one of the following method should be used :
    * `$page = basename($_GET['page']);` - returns the basename of the resource (thus preventing `.` and `..` to be used)
    * `ini_set ('open_basedir', __DIR__ . "/pages/");` (at the beginning of the script) - limits the availability of resources to the specified directory

### XSS: escape output, CSP

* To prevent XSS (Cross-Site-Scripting), where a user inject `<script>` tags containing malicious javascript into our apps (that will execute whenever we print that content together with our app's output (e.g. the html page)), we should always escape output which was provided by a user:
  * `$comment = strip_tags($comment, '<br>');` - strips all html tags from `$comment`, only allow `<br>` (html comments and php code are always stripped, cannot be permitted)
  * `$new = htmlspecialchars("<a href='test'>Test</a>", ENT_QUOTES);` - converts html tags to entities (thus preventing their interpretation)
    * or `htmlentities()` which converts ALL html tags to entities (instead of `htmlspecialchars`, which just converts a subset: the most dangerous ones)
  * External libraries: e.g. [HTML purifier](http://htmlpurifier.org/), [zend escaper](https://github.com/zendframework/zend-escaper), etc 
* CSP - Content Security Policy - are modern http headers that specify what resources can be loaded and belonging to what domains (thus blocking any cross-site request)
  * e.g. `Content-Security-Policy: script-src 'self' https://www.google-analytics.com`
  * External libraries exists: e.g. [csp-builder](https://github.com/paragonie/csp-builder)
  * See [reference](https://content-security-policy.com/)

# Sensible data protection

* To encrypt sensible data, several techniques are used in PHP
  * OpenSSL
  * Password hashing (and salting)
