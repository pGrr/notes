% PHP Persistance

\newpage

File
====

* Entire file - to be used for small files and few users
  * `$nBytes = file_put_contents($path, $text);` - write text in a file
  * `$content = file_get_contents($path);` - read contens of a file
  * both return `false` if failure (to be checked!)
  * there are actually many other options and arguments: [see more 1](https://www.php.net/manual/en/function.file-get-contents.php), [see more 2](https://www.php.net/manual/en/function.file-put-contents.php)
* Sequential access (Portions of file) - to be preferred if big files or many concurrent users
  * `fopen($fileName, 'w+');` - open file - [see more](https://www.php.net/manual/en/function.fopen.php)
    * if failure returns `false`, to be checked!
  * `$nBytes = fwrite($file, $newLine);` - writes a line
    * if failure returns `false`, to be checked!
  * `$buffer = fread($file, $bufferSize);` - read a specified amount of bytes from a file advancing the file pointer and putting them on a buffer - the buffer size should be as big as possible, depending on the use case, in order to minimize the amount of accesses
  * `fseek($file, $nBytes, $relativeTo);` - moves the file pointer of nBytes from the specified point (`SEEK_SET` start, `SEEK_CUR` current position, `SEEK_END` end)
  * `$isFinished = feof($file);` - checks if the file pointer has reached the end of file
  * `fclose($file);` - closes the file (all resources are closed at the end of the script, but closing explicitally is a best practice)
* Memory usage
  * `$nBytes = memory_get_usage($real = false);` - get the amount of allocated memory for this script at the time of calling
  * `$nBytes = memory_get_peak_usage($real = false);` - get the maximum amount of allocated memory for this script during its whole execution
    * best practice: put this as the last line of the script
  * `$real` - `false` (memory actually used by the script), `true` (memory allocated by the OS)
* File lock - preventing race conditions due to concurrent accesses 
  * `$lockSuccess = flock($file, $lockMode)` - locks the file in a specified mode
    * `LOCK_SH` - shared = can be read by any script
    * `LOCK_EX` - exclusive = locked for writing by the current script only
    * `LOCK_UN` - unlock = release a previous block
    * By default this function is blocking until the lock is acquired (e.g. if another lock is active it will wait until it is released and then continue), but this can be customized ([see more](https://www.php.net/manual/en/function.flock.php))
    * A lock works only for other scripts which use `flock`, access by other script not using it is not prevented (e.g. a script not using it may read inconsistent data from a file that is currently being written by another one), so it is a best practice to always use `flock` in case of possible race conditions
    * `fflush()` - should be used before the unlock, in order to actually write instantly the file content (instead of letting the OS decide), otherwise the OS may decide to actually do the writing only at `fclose`

CSV
---

* `fputcsv($file, $array);` - write an array as a csv line into a file
* `$array = fgetcsv($file);` - read a line from a csv file and get it as array. Returns `false` if file is ended.
* `$array = str_getcsv($string);` read a string as a csv line and get it as array
* default separator is comma `,` - but another one can be specified
  * `fputcsv($file, $data, $separator);`
  * `fgetcsv($file, 0, $separator);`
  * `str_getcsv($string, $separator);`
* other options exists ([see docs](https://www.php.net/manual/en/function.fgetcsv.php)) 

JSON
----

* Encoding-decoding
  * `$string = json_encode($array);` - encodes an array into a json string. Return `false` if failure (to be checked!)
    * in case of failure errors can be checked with `json_last_error();` and `json_last_error_msg();`
  * `$array = json_decode($string)` - decodes a json string into a generic object (`stdClass`)
  * `$array = json_decode($string, true);` - decodes a json string into an array
  * other options available (e.g. `PRETTY_PRINT`, `JsonSerializable` interface to specify a custom encoding, etc)

XML
---

* See [SimpleXML](https://www.php.net/manual/en/book.simplexml.php)
  * `simplexml_load_file()`, `simplexml_load_string()`, `attributes()`, `asXML()` or `saveXML()`, `libxml_get_errors()`, `addChild`, `unset()`, etc