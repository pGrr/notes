PDO
===

Install db-specific extension
-----------------------------

* install the database specific PDO's extension ([see more](https://www.php.net/manual/en/pdo.installation.php))

Open connection
---------------

* a connection can be opened by creating a `PDO` object

```php
// open the connection specifying the connection string,
// user, pwd and connection options
try {
    $db = new PDO(
        'mysql:host=localhost;dbname=testdb', 
        'username', 
        'password',
        [
            PDO::MYSQL_ATTR_INIT_COMMAND => "SET NAMES utf8"
        ]);
} catch (PDOException $e) {
    throw new Exception(sprintf(
        "PDO connection failed: %s\n",
        $e->getMessage()
    ));
}
```

* then CRUD operations can be executed:

Create
------

```php
// Prepare the statement (using ':' markers for each parameter)
$sql = 'INSERT INTO speakers (name, title, company, url, twitter) VALUES (:name, :title, :company, :url, :twitter)';
$sth = $db->prepare($sql);
$data = [
    ':name' => 'Enrico Zimuel',
    ':title' => 'Senior Software Engineer',
    ':company' => 'Zend Technologies',
    ':url' => 'http://www.zimuel.it',
    ':twitter' => '@ezimuel'
];
// Execute the query
if (! $sth->execute($data)) {
    throw new Exception(sprintf(
        "Error PDO exec: %s", implode(',', // array -> string
            $db->errorInfo() // array containing error info
        )
    ));
}
printf("Speaker added successfully!\n");
```

Read
----

* After the query execution, the statement object can be used to fetch the query result ([see more](https://www.php.net/manual/en/pdostatement.fetchall.php)):
  * `fetchAll` - get all the results as a single object (standard object, associative array, non associative, etc)
  * `fetch` - get one row at a time

```php
/*
 * READ
 */
// FETCHALL (all results as a single object)
$sql = 'SELECT * FROM speakers WHERE company=:company';
$sth = $db->prepare($sql);
$data = [ ':company' => 'Zend Technologies' ];
if (! $sth->execute($data)) {
    throw new Exception(sprintf(
        "Error PDO exec: %s", implode(',', $db->errorInfo())
    ));
}
// fetch result as stdClass object
$result = $sth->fetchAll(PDO::FETCH_OBJ);
// Other options are available, e.g.:
// PDO::FETCH_ASSOC to get it as an associative array
// PDO::FETCH_NU to gen a non-associative array
// ...etc

// FETCH (one row at a time)
$sql = 'SELECT * FROM speakers WHERE company=:company';
$sth = $db->prepare($sql);
$data = [ ':company' => 'Zend Technologies' ];
if (! $sth->execute($data)) {
    throw new Exception(sprintf(
        "Error PDO exec: %s", implode(',', $db->errorInfo())
    ));
}
while ($row = $sth->fetch(PDO::FETCH_OBJ)) {
    var_dump($row);
}
```

Update
------

* `$sth->rowCount()` will give the number of rows affected 

```php
$sql = 'UPDATE speakers SET name=:name WHERE id=:id';
$sth = $db->prepare($sql);
$data = [
    ':name' => 'Alberto Zimuel',
    ':id'   => 1
];
if (! $sth->execute($data)) {
    throw new Exception(sprintf(
        "Error PDO exec: %s", implode(',', $db->errorInfo())
    ));
}
printf("Speaker updated successfully!\n");
```

Delete
------

* `$sth->rowCount()` will give the number of rows affected 

```php
$sql = 'DELETE FROM speakers WHERE company=:company';
$sth = $db->prepare($sql);
$data = [ ':company' => 'Zend Technologies' ];
if (! $sth->execute($data)) {
    throw new Exception(sprintf(
        "Error PDO exec: %s", implode(',', $db->errorInfo())
    ));
}
printf("Speaker/s deleted successfully!\n");
```
