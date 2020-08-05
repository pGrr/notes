# Sessions

[See docs](https://www.php.net/manual/en/book.session.php)

Sessions are a mechanism with which PHP preserves certain data across subsequent accesses. It identifies a user, memorizes his informations on the server side and maintains him logged in for a certain amount of time.

* Session start
  * `session_start()` at the beginning of the php script
    * Or setting `session.autostart = 1` in `php.ini` (this comes with some limitations, [see docs](https://www.php.net/manual/en/intro.session.php))
      * Since PHP>=7.0 options are available, e.g. `cookie_lifetime`, etc ([see docs](https://www.php.net/manual/en/function.session-start.php))
  * On receiving a user HTTP request, if a session is started, the PHP engine:
    * finds (or create) the unique session ID token for that user
      * checks if the http request contains a session ID cookie token
      * If not it generates one using a hash function (MD5, SHA1, etc) using source ID, current time and a pseudo-random number generator
      * The default name of the cookie is `PHPSESSID`
    * It locks for writing (thus making other concurrent access requests to that file blocking) a file associated with the unique session id token (if it doesn't exist, it creates a new one)
    * it populates `$_SESSION` global array with the file content
    * Any data written into `$_SESSION` global array will be written into the opened file, until session is active (until then the file is still locked for writing)
    * it returns the unique session ID token to the user
      * either with a cookie, or 
      * by propagating in the url a `SID` parameter, which will be managed by the browser
* On subsequent requests, the session is still considered active, the user's browser (or equivalent) presents the session identifier, again usually by way of a cookie, for inspection
  * The PHP engine will recognize that session id token and open the file corresponding to that user, thus retrieving previously saved information into `$_SESSION` global array 
* The default session handling can be modified, e.g. to create a session handling using a database in order to share session data between many servers, by 
  * creating a class extending `SessionHandlerInterface` 
  * calling `session_set_save_handler($handler, true);` - where `$handler` is the object of the created class
  * Then, calling `session_start()` and using `$_SESSION` as usual will use your custom functions instead of the default ones
  * Sharing session data between different servers is required for all medium-big php apps, as they'll have a load balancer and will be served by different reduntant servers, there are basically two way to implement this: using a shared filesystem (not efficient, less secure) or using a database (more efficient, more secure), which is advised. In this case `SessionHandlerInterface` can be implemented by accessing and saving into the database, thus using `$_SESSION` will write into the db instead of into server's files.

Basic functionning example (for security best-practices, see next sections):

```php
session_start();
$loggedIn = $_SESSION['isLoggedIn'] ?? FALSE;
if (isset($_POST['login'])) {
    if ($_POST['username'] == // username lookup
        && $_POST['password'] == // password lookup) {
        $loggedIn = TRUE;
        $_SESSION['isLoggedIn'] = TRUE;
    }
}
```

# Session security

* There are tremendous security concerns when the session identifier is the sole means of identifying a returning website visitor. E.g.:
  * If an attacker were to obtain the session identifier, for example, by means of a successfully executed Cross-site scripting (XSS) attack, all he/she would need to do would be to set the value of the `PHPSESSID` cookie to the illegally obtained one, and they are now viewed by your application as a valid user
  * Session data can be hijacked: it is strongly recommended to never store any personal information in `$_SESSION`. This would most critically include credit card numbers, government issued ids, and passwords; but would also extend into less assuming data like names, emails, phone numbers, etc which would allow a hacker to impersonate/compromise a legitimate user. 
    * As a general rule, only save worthless/non-personal values, such as numerical identifiers, in `$_SESSION` or other session data.
* Several techniques should be used to improve the overall security of the website, as the one explained below

Narrow the window of time during which the PHPSESSID is valid
-------------------------------------------------------------

* use `session_regenerate_id()`
  * It's a very simple command which:
    * generates a new session identifier
    * invalidates the old one
    * maintains session data intact
    * has a minimal impact on performance. 
  * It can only be executed after the session has started:
    ```php
    session_start();
    session_regenerate_id();
    ```

Properly terminate sessions
---------------------------

  * Ensure that web visitors have a logout option. It is important, however, to not only destroy the session using `session_ destroy()`, but also to unset `$_SESSION` data and to expire the session cookie:
    ```php
    session_unset();
    session_destroy();
    setcookie('PHPSESSID', 0, time() - 3600);
    ```

Use finger-print of the user as secondary verification
------------------------------------------------------

* Another easy technique that can be used to prevent session hijacking is to develop a finger-print or thumb-print of the website visitor. 
* One way to implement this technique is to collect information unique to the website visitor over and above the session identifier. Such information includes 
  * the user agent (that is, the browser)
  * languages accepted
  * remote IP address
* You can derive a simple hash from this information, and store the hash on the server in a separate file. 
* The next time the user visits the website, if you have determined they are logged in based on session information, you can then perform a secondary verification by matching finger-prints:
    ```php
    $remotePrint = md5(
        $_SERVER['REMOTE_ADDR']
        . $_SERVER['HTTP_USER_AGENT']
        . $_SERVER['HTTP_ACCEPT_LANGUAGE']);
    $printsMatch = file_exists(THUMB_PRINT_DIR . $remotePrint);
    if ($loggedIn && !$printsMatch) {
        $info = 'SESSION INVALID!!!';
        error_log('Session Invalid: ' . date('Y-m-d H:i:s'), 0);
        // take appropriate action
    }
    ```
  * Aside-note: In the above example we are using md5() as it's a fast hashing algorithm and is well suited for internal usage. It is not recommended to use md5() for any external use as it is subject to brute-force attacks.
 



