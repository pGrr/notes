ORM - Object Relational Mapping
===============================

__Pros__:

* __Portability__ (Abstracts SQL dialects)
* __Nested data__ (e.g. in many-to-many relationships you don't need to access associative tables, data is automatically nested for you and ready to be accessed)
* __Simplicity__ (e.g. abstracts from SQL so you don't normally need to write SQL queries manually, you don't have to differentiate insert and update queries, etc)

__Cons__:

* __Slow performance__ (compared to raw SQL, as there will be a translation layer)
* __No tuning__ (cannot make queries faster using SQL dialect-specific language constructs, cannot write SQL in an efficient fashion, etc)
* __Limitations on Complex Queries__ (some ORM layers have limitations, so sometimes you will find yourself writing raw SQL anyway)
* __Learning curve for whatever goes beyond the basics__ (each ORM has it's own structure and way of working, so whenever you need to go beyond the basics, e.g. for tuning or for complex queries, you need to learn the inner workings)

Doctrine
--------

* [See docs](https://www.doctrine-project.org/)
* Born by the influence of Java's Hibernate or Ruby's ActiveRecord
* Easy install with composer:
    ```json
    "require": {
        "doctrine/orm": "2.4.*"
    }
    ```
* DQL - Doctrine Query Language - is a SQL-like language that operates on the data model (objects) instead of tables
* Doctrine takes care of the data model (OOP-to-SQL conversion):
  * the data model is created by Doctrine by processing our php classes (as opposed to creating the database in a standard way with SQL)
  * IDs are managed automatically (completely abstracted away by Doctrine: the developer doesn't take care of them, just writes OOP code)
  * We need to instruct Doctrine on how to convert our OOP-code into a relational model. There are 3 ways of doing that:
    * Docblock - annotations in php comments
    * XML 
    * YAML
* `vendor/bin/doctrine` - execute the conversion of the data model (actually creates the database)

### Docblock

* [See docs](https://www.doctrine-project.org/projects/doctrine-orm/en/2.7/reference/basic-mapping.html#basic-mapping)
* `@Entity @Table(name="my_table_name")` - map class into a table
* `@Id @GeneratedValue` - map the property into the table's ID and makes it auto-increment (so IDs will be automatically managed by Doctrine for the developer)
* `@Column(type="...")` - map a property into a table column
  * `@Column(type="...", nullable=true)` - make the column nullable
* `@ManyToMany` - specifies a many to many relationship

```php
/** @Entity @Table(name="talks") */
class Talk
{
    /** @Id @Column(type="integer") @GeneratedValue */
    protected $id;
    /** @Column(type="string") */
    protected $title;
    /** @Column(type="string", nullable=true) */
    protected $abstract;
    /** @Column(type="date", nullable=true) */
    protected $day;
    /** @Column(type="time", nullable=true) */
    protected $start;
    /** @Column(type="time", nullable=true) */
    protected $end;
    //...
}
```

### Creating the database

1. Create the files required for the Doctrine console interface to work:
   1. `/bootstrap.php` file, with all info required by Doctrine's `EntityManager` (e.g. database used, source folder, etc)
        ```php
        // boostrap.php
        use Doctrine\ORM\Tools\Setup;
        use Doctrine\ORM\EntityManager;
        require_once "vendor/autoload.php";
        $isDevMode = true;
        $config = Setup::createAnnotationMetadataConfiguration(
            array(__DIR__."/src"),
            $isDevMode);
        $conn = array(
            'driver' => 'pdo_sqlite',
            'path'   => __DIR__ . '/db.sqlite',
        );
        return EntityManager::create($conn, $config);
        ```
   2. `cli-config.php` file, with the path to your bootstrap file:
        ```php
        // cli-config.php
        use Doctrine\ORM\Tools\Console\ConsoleRunner;
        return ConsoleRunner::createHelperSet(require "bootstrap.php");
        ```
    3. `/vendor/bin/doctrine` - now you can try doctrine's cli interface to see if it works (should display a help message)
    4. `/vendor/bin/doctrine orm:schema-tool:create` - creates the database
    5. `/vendor/bin/doctrine orm:schema-tool:update --force` - updates the database (after code changes)

### Association mapping (join tables)

* [See docs](https://www.doctrine-project.org/projects/doctrine-orm/en/2.7/reference/association-mapping.html#association-mapping)

### Docblock

* `@ManyToMany`, `@OneToMany`, `@ManyToOne`, `@OneToOne`, etc

```php
/**
 * @Entity @Table(name="speakers")
 */
class Speaker
{
    /**
     * @ManyToMany(
     * targetEntity="Talk", 
     * inversedBy="speakers", 
     * cascade={"persist","remove"})
     * @JoinTable(name="speakers_talks")
     */
    private $talks;
    // ...
}
/**
 * @Entity @Table(name="talks")
 */
class Talk
{
    /**
     * @ManyToMany(
     * targetEntity="Speaker", 
     * mappedBy="talks", 
     * cascade={"persist","remove"})
     */
    private $speakers;
    // ...
}
```

### Initialization in constructor

* `Doctrine\Common\Collections\ArrayCollection` - manages the collections of doctrine's objects

```php
/**
 * @Entity @Table(name="speakers")
 */
class Speaker
{
    // ...
    public function __construct() {
        $this->talks = new \Doctrine\Common\Collections\ArrayCollection();
    }
    // ...
}
/**
 * @Entity @Table(name="talks")
 */
class Talk
{
    // ...
    public function __construct() {
        $this->speakers = new \Doctrine\Common\Collections\ArrayCollection();
    }
    // ...
}
```

### Add / Remove methods

* Methods to add / remove objects from doctrine's collections should be added

```php
/**
* @Entity @Table(name="speakers")
*/
class Speaker
{
    // ...
    public function addTalk(Talk $talk)
    {
        if ($this->talks->contains($talk)) {
            return;
        }
        $this->talks->add($talk);
        $talk->addSpeaker($this);
    }
    public function removeTalk(Talk $talk)
    {
        if (! $this->talks->contains($talk)) {
            return;
        }
        $this->talks->removeElement($talk);
        $talk->removeSpeaker($this);
    }
    // ...
}
/**
 * @Entity @Table(name="talks")
 */
class Talk
{
    // ...
    public function addSpeaker(Speaker $speaker)
    {
        if ($this->speakers->contains($speaker)) {
            return;
        }
        $this->speakers->add($speaker);
        $speaker->addTalk($this);
    }

    public function removeSpeaker(Speaker $speaker)
    {
        if (! $this->speakers->contains($speaker)) {
            return;
        }
        $this->speakers->removeElement($speaker);
        $speaker->removeTalk($this);
    }
    // ...
}
```

* Then you can write changes to the database with:
  * `$em = require_once "bootstrap.php"` - create the entity manager
  * Submit
    * `$em->persist($obj)` - makes doctrine prepare the data to be written
    * `$em->remove($obj)` - makes doctrine prepare the removal of the given object from the db
    * `$em->flush()` - actually submits the changes to the database
  * `$obj->getId()` - after the writing, doctrine can show the id of the written object in the db
  * Select
    * `$em->find($entityName, $id)` - finds an object (db row) by its id
    * `$em->getRepository($entityName)->...` - fetch the given entity
      * `...->findBy($associativeArray)` - selects records from the given table using the given parameters
      * `...->findOneBy($associativeArray)` - returns the first matching result

```php
$em = require_once "bootstrap.php"; // EntityManager

$speaker = new Speaker();
$speaker->setName("Enrico Zimuel");
$speaker->setTitle("Senior Software Engineer");
$speaker->setCompany("Zend Technologies");
$speaker->setUrl("http://www.zimuel.it");
$speaker->setTwitter("@ezimuel");

$em->persist($speaker); // prepare
$em->flush(); // write
// object id is now available
printf ("Added Speaker with Id %d\n", $speaker->getId());
// find by id
$speaker = $em->find("Speaker", 1);
printf("Speaker: %s\n", $speaker->getName());
// find by parameters
$company = "Zend Technologies";
$speakers = $em->getRepository("Speaker")
               ->findBy(["company" => $company]);
printf("Speakers working for %s:\n", $company);
foreach ($speakers as $speaker) {
    printf("%s with ID %d\n", $speaker->getName(), $speaker->getId());
}
// Update
$speaker = $em->getRepository("Speaker")->findOneBy(["name" => "Enrico Zimuel"]);
if (! $speaker) {
    printf("The Speaker specified doesn't exist!");
    exit(1);
}
$speaker->SetName("Alberto Zimuel");
$em->persist($speaker);
$em->flush();
printf("Updated Speaker with ID %d\n", $speaker->getId());
// Remove
$id = 1;
$speaker = $em->find("Speaker", $id);
if (! $speaker) {
    printf("No speaker found with ID %d\n", $id);
    exit(1);
}
$em->remove($speaker);
$em->flush();
printf("Removed Speaker with ID %d\n", $id);
```

### Doctrine's DQL - Data Query Language

* Doctrine DQL is a SQL-like language that can work with Doctrines' entities instead of db tables. 

```php
$em->require_once "bootstrap.php";

$query = $em->createQuery(
    "select s from Speaker s where SIZE(s.talks) > 1"); // DQL
// Whereas in standard SQL it would have been:
//  SELECT s.* FROM speakers AS s 
//  JOIN speakers_talks ON (s.id = speaker_id)
//  GROUP BY s.id HAVING COUNT(s.id) > 1
$speakers = $query->getResult();
if (!speakers) {
    printf("No speaker has more than one talk.\n");
}
foreach($speakers as $s) {
    printf("%s has %d talks\n", 
        $s->getName(), 
        count($s->getTalks()));
}
```
 
