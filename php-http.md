# HTTP Requests

## Global variables

A bunch of global variables are used by the web server to communicate HTTP request data with PHP execution environment via the SAPI module

## Message body (php://input)

* `file_get_contents("php://input");` - returns the body of the HTTP request (returns php stdin, which in a HTTP request is the body)

## $_SERVER

* `$_SERVER` - ([See docs](https://www.php.net/manual/en/reserved.variables.server.php)) - associative array containing server and execution environment information: headers, version, request URL, HTTP method used, remote IP, port used, ecc. E' un array associativo contenente le seguenti chiavi (e molte altre): `$_SERVER[...]`:
  * `REQUEST_METHOD`,`REQUEST_URI`, ...
  * `SERVER_PROTOCOL`, ...
  * `HTTP_ACCEPT`, `HTTP_ACCEPT_ENCODING`, ...
  * `HTTP_CONNECTION`, `HTTP_HOST`, `HTTP_USER_AGENT` ...

## $_GET

* `$_GET` - associative array containing paramaters passed with GET
* If the HTTP request has a query string, such as `<url>?<key1>=<val1>&<key2>=<val2>&...` then `$_GET` will be an associative array holding those key-value pairs. 
    * e.g. `$_GET['key2']` contains `<val2>` and so on.

### Query string URI-encoding

* Query strings, such as `<url>?<key1>=<val1>&<key2>=<val2>&...` must be encoded with [RFC 3986 - URI Syntax](https://tools.ietf.org/html/rfc3986): 
  * a part from numbers and letters, each other character must be written in `%<hex-code>` format
  * e.g. `+` = spaces (`%20`)
* `urlencode()` - encodes a string with URI formatting as specified by the rfc standard

## $_POST

* `$_POST` - associative array containing parameters passed with POST
* As in GET, we can access the values by their keys: `$_POST[<key>]` will contain its value
  * E.g. in a html form:
    ```html
    <form method="POST" action="login.php">
        <p>
            <label for="email">Email</label>
            <input type="text" id="email" name="email">
        </p>
    </form>
    ```
  * When sent `login.php` will process the request: inside that file, we can use `$_POST['email']` to get the value sent

## $_FILES

* `$_FILES` - infos about files sent via a POST form
* HTTP allows sending `multipart/form-data`, a specific [MIME](https://en.wikipedia.org/wiki/MIME) format to send files through html forms
  * `Content-Type: multipart/form-data;boundary=<pseudo-random-delimiter>`
* E.g. when the following form is submitted
  ```html
  <form method="POST" action="upload.php" enctype="multipart/form-data">
    <p>
      <label for="fileUpload">File</label>
      <input type="file" name="fileUpload" id ="fileUpload">
    </p>
    <input type="submit" value="Send">
  </form>
  ```
* The http request send will look like:
  ```http
  POST /upload.php HTTP/1.1
  Content-Type: multipart/form-data;boundary=----grTeHAivvJ5oLOz4
  ...
  ----grTeHAivvJ5oLOz4
  content-disposition: form-data;name="fileUpload";filename="photo.jpg"
  <binary-content-here>
  ----grTeHAivvJ5oLOz4
  ```
  * Inside the delimiter (a pseudo-random string) will be placed some info (content disposition, form input field name, filename) followed by the file's binary content, followed by the delimiter at its end
* PHP can manage sent files via `$_FILES` global array, which would look like this:
  ```php
  var_dump($_FILES)
  array(1) {
    ["fileUpload"]=>
      array(5) {
        ["name"]=> string(18) "photo.jpg"
        ["type"]=> string(10) "image/jpeg"
        ["tmp_name"]=> string(14) "/tmp/phpQfjXke"
        ["error"]=> int(0)
        ["size"]=> int(167295)
      }
  }
  ```
  * the key of `$_FILES` array are the file names
  * each file (value) is an array of infos, e.g. filename, type, temp_name, error, size, ...
  * `tmp_name` - is a temporary pseudo-random name given automatically by PHP in order to not substitute an existing file with the same name
  * `error` - might contain an error code, to be checked! 
    * `__UPLOAD_ERR_OK__` = 0 - must be checked in order to determine if the file was sent successfully (other constants available, [see docs](https://www.php.net/manual/en/features.file-upload.errors.php))
* `move_uploaded_file(<filename>, <path>)` can be used to move the uploaded file, e.g.:
  ```php
  $uploadsDir = __DIR__ . '/uploads';
  foreach ($_FILES as $name => $file) {
      if (UPLOAD_ERR_OK === $file['error']) {
          $fileName = basename($file["name"]);
          move_uploaded_file(
            $file["tmp_name"], 
            "$uploadsDir/$fileName" . uniqueid());
      }
  }
  ```
  * `basename` is used for preventing file-system traversal attacks
  * `uniqueid` is used to append a pseudo-random string to the filename, in order to avoid overwriting an existing file with the same name

## $_COOKIE

* `$_COOKIE` - cookies info global array
* Cookies are `key=value` pairs 
  * Sent from the server to the client at first connection
  * Saved in the client storage (by the browser), if not disabled
  * Inserted by the browser as an HTTP header in every subsequent HTTP request the client will make to that server 
  * The server, receiving the cookie, is able to identify the user    
* The server can set a cookie using specific http headers (multiple ones in a single http request are allowed)
  ```http
  Set-Cookie: <cookie-name>=<cookie-value>
  Set-Cookie: <cookie-name>=<cookie-value>; Expires=<date>
  Set-Cookie: <cookie-name>=<cookie-value>; Max-Age=<nonzerodigit>
  Set-Cookie: <cookie-name>=<cookie-value>; Domain=<domain-value>
  Set-Cookie: <cookie-name>=<cookie-value>; Path=<path-value>
  Set-Cookie: <cookie-name>=<cookie-value>; Secure
  Set-Cookie: <cookie-name>=<cookie-value>; HttpOnly
  ```
  * `Expires` (unix timestamp) and `Max-Age` (int seconds) 
    * are used to set a persistent cookie
    * If not present, the cookie will be deleted at browser restart
    * If both are present, `Max-Age` will be used
  * `Domain` 
    * is used to specify allowed hosts to receive the cookie
    * If unspecified, it defaults to the host of the current document location, excluding subdomains
    * If specified, then subdomains are always included
  * `Path` 
    * is used to specify a URL path that must exist in the requested URL in order to send the Cookie
    *  `%x2F` = `/` character is considered a directory separator, and subdirectories will match as well. For example, if `Path=/docs` is set, these paths will match: `/docs`, `/docs/Web/`, `/docs/Web/HTTP`, ecc
  * `HttpOnly` 
    * is used to disable Javascript-API (e.g. `Document.cookie`, `XMLHttpRequest`, `Request`, etc) for that cookie (this way it can't be modified by client-side scripts, thus avoiding XSS risks)
  * `Secure` 
    * is used to make HTTPS and SSL compulsory in order to send that cookie
* `setcookie (<name>, [<value>, <expire>, <path>, <domain>, ecc])` - is used to set a cookie with the given parameters (only `<name>` is mandatory)
  * e.g. `setcookie("User-id", "b74686973a", time() + 3600*24*30, "example.com"); `
    * sets a cookie `User-id=b74686973a`, valid for one month from now, for `example.com` domain (and subdomains)
* When a client receives an HTTP response containing one or more `Set-Cookie` headers, if cookies are not disabled by the user, memorizes the associated informations in a file (text file, SQLlite database, etc: depends on the browser)
* When sending subsequent HTTP requests, it finds the cookies associated with the url, checks if they are valid using the cookie parameters, and if valid it attaches the cookies to the HTTP request as a HTTP header:
  * `Cookie: <cookie-name>=<cookie-value>`
* `$_COOKIE[<cookie-name>]` - can be used in PHP code to retrieve the cookie-value associated with the given cookie-name in the current HTTP request, e.g.:
  ```php
  if (isset($_COOKIE["User-Id"])) {
    // User already registered
  }
  ```
* Secutiry aside:
  * Cookies are memorized by the client so they can be modified by the user
  * For security reason they shouldn't contain sensible information 
  * In most cases, they just should be used by the server to identify the user (usually by pseudo-random strings) in order to retrieve personal informations stored in the server (NOT in the client!)

# HTTP response

* To create a HTTP response, PHP can just write to stdout
  * Response code is 200 (success) by default, if not specified otherwise, e.g. with `header()`
  * Other http fields, if not specified otherwise, are automatically generated by PHP, e.g.: 
    * `Connection: close`
    * `Content-type: <inferred-content-type>`
    * `Date: <timestamp>`
    * `Host: <our-host>`
    * `X-Powered-By: <info-about-php-and-so>`
* `header(string $string, [, bool $replace=true [, int $http_code]])` - modify the default header
  * `$string` - is the string to be added to the header, e.g.
    * e.g. `header('Content-Length: ' . strlen($html))`
  * `$replace` - true (default) if the header will substitute an existing value, false otherwise
  * `$http_code` - 200 by default, but we can specify another HTTP-code


# Authentication

* Authentication ([See docs](https://www.php.net/manual/en/features.http-auth.php))
  * HTTP Basic
  * Digest

## HTTP Basic Access Authentication

* The client sends an HTTP request to the server
* The server sends the HTTP header `WWW-Authenticate: Basic realm="<Realm>"`
* The client is prompted for username and password
* The client sends an HTTP request with the header `Authorization: Basic <auth-string>`
  * where `<auth-string>` is computed with the following algorithm
    * username and password are concatenated, separated by a colon: `<username>:<password>`
    * the resulting string is encoded in octal
    * the resulting octal code is encoded in Base64
    * the resulting string is `<auth-string>`
* PHP automatically decodes `<auth-string>` and make username and password directly accessible:
  * `$_SERVER['PHP_AUTH_USER']` - username
  * `$_SERVER['PHP_AUTH_PW']` - password
* Security concerns: 
  * The message is not encrypted by default! HTTPS is necessary!

Basic example:

```php
$users = ['admin' => '12345678', 'guest' => 'guest'];
$username = $_SERVER['PHP_AUTH_USER'] ?? false;
  // check if the username and password are valid
if (false === $username) {
  require_auth();
  exit;
}
if (!isset($users[$username]) || $users[$username] !== $_SERVER['PHP_AUTH_PW']) {
  require_auth();
  exit;
}
echo '<h1>User authenticated!</h1>';
// Send a http-basic auth request
function require_auth(): void
{
  header('WWW-Authenticate: Basic realm="My Realm"');
  header($_SERVER["SERVER_PROTOCOL"] . ' 401 Unauthorized');
  echo '<p>This page requires authentication!</p>';
}
```

## HTTP Digest Access Authentication

* Similar to HTTP-Basic authentication, but with a more complex and secure algorithm to encode username and password
* Basic version:
  * `auth_string = MD5(HA1:nonce:HA2)` - where:
    * `HA1 = MD5(<username>:<realm>:<password>)` 
    * `HA2 = MD5(<method>:<digestURI>)`
    * `nonce` - is a pseudo-random value
* Extended version: 
  * a `qop` = quality of protection parameter is included, i.e. additional parameters are concatenated (e.g. the computed hash code MD5 of the message body, etc)
* PHP doesn't automatically extracts the username and password, we must extract username and then check the password by computing the same digest:
  * `$data = $_SERVER['PHP_AUTH_DIGEST']` 
  * `$data['username']` - check the username
  * We compute the valid digest using the same digest algorithm and the password we have for that user on the server
  * if the user digest is equal to the computed valid digest, password is correct
* Security concerns:
  * MD5 has been de-classified as not-so secure recently, so HTTP-Digest is not considered enough secure, HTTPS MUST be used!

```php
$realm = 'Restricted area';

//user => password
$users = array('admin' => 'mypass', 'guest' => 'guest');

if (empty($_SERVER['PHP_AUTH_DIGEST'])) {
    header('HTTP/1.1 401 Unauthorized');
    header('WWW-Authenticate: Digest realm="'.$realm.
           '",qop="auth",nonce="'.uniqid().'",opaque="'.md5($realm).'"');

    die('Text to send if user hits Cancel button');
}
// analyze the PHP_AUTH_DIGEST variable
if (!($data = http_digest_parse($_SERVER['PHP_AUTH_DIGEST'])) ||
    !isset($users[$data['username']]))
    die('Wrong Credentials!');
// generate the valid response
$A1 = md5($data['username'] . ':' . $realm . ':' . $users[$data['username']]);
$A2 = md5($_SERVER['REQUEST_METHOD'].':'.$data['uri']);
$valid_response = md5($A1.':'.$data['nonce'].':'.$data['nc'].':'.$data['cnonce'].':'.$data['qop'].':'.$A2);
// check the response
if ($data['response'] != $valid_response)
    die('Wrong Credentials!');
// ok, valid username & password
echo 'You are logged in as: ' . $data['username'];
// function to parse the http auth header
function http_digest_parse($txt)
{
    // protect against missing data
    $needed_parts = array('nonce'=>1, 'nc'=>1, 'cnonce'=>1, 'qop'=>1, 'username'=>1, 'uri'=>1, 'response'=>1);
    $data = array();
    $keys = implode('|', array_keys($needed_parts));
    preg_match_all('@(' . $keys . ')=(?:([\'"])([^\2]+?)\2|([^\s,]+))@', $txt, $matches, PREG_SET_ORDER);

    foreach ($matches as $m) {
        $data[$m[1]] = $m[3] ? $m[3] : $m[4];
        unset($needed_parts[$m[1]]);
    }
    return $needed_parts ? false : $data;
}
```

# Authorization

* Authorization = control access to resources
  * Oauth2
  * RBAC - Role-Based Access Control

## OAuth2

* [See docs](https://www.php.net/manual/en/features.http-auth.php)
* A framework to manage authorizations, now a standard, that enables:
  * a third-party application
  * to access a HTTP service in a limited way
  * on behalf of the owner of the resource
  * via an approval request between the user and the HTTP service
  * or directly between the third-party application and the HTTP service
* OAuth2 defines:
  * Resource owner = user
  * Resource server = API server
  * Authorization server = can be the API server itself or not
  * Client = third-party application

...
