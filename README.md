# JSONQL
Manage JSON databases with the SQL language.

## Usage
In this small guide, we will work with a database named "jsonql", and a table named "test". This is the table's content:
```json
{
  "prototype": ["id", "name", "age", "male"],
  "properties":
  {
    "last_insert_id": 4
  },
  "datas":
  [
    {"id":1, "name": "John Doe", "age": 25, "male": true},
    {"id":2, "name": "Stephan Doe", "age": 18, "male": false},
    {"id":3, "name": "Ellie Doe", "age": 23, "male": false},
    {"id":4, "name": "Isaac Doe", "age": 10, "male": true}
  ]
}
```

### Instanciate the class
```php
<?php
    
  require ('JSONQL.php');
  
  try {
    $jsonql = new JSONQL('./jsonql');
  }
  catch (JSONQLException $e) {
    echo $e->getMessage();
  }

?>
```
When you instanciate jSonQL, you have to give the path to the directory which represent the JSON database. In this folder, all JSON files are tables.

### Give a query
You can give direct or prepared SQL queries, like your doing with PDO:
```php
// Direct queries
$query   = $jsonql->query('your_query');
$results = $query->fetch(); // For SELECT queries

// Prepared queries
$prepare = $jsonql->prepare('your_prepared_query');
$prepare->bindValue(':key1', $val1, JSONQL::PARAM_STR);
$prepare->bindValue(':key2', $val2, JSONQL::PARAM_INT);
$prepare->bindValue(':key3', $val3, JSONQL::PARAM_BOOL);
$prepare->execute();
$results = $prepare->fetch(); // For SELECT queries
```
jSonQL constants `PARAM_STR`, `PARAM_INT`, and `PARAM_BOOL` are used to convert values respectively in `string`, `int` and `boolean`.

### Get query results
You can easily get the result of a SELECT query with a `foreach` loop.
```php
$query   = $jsonql->query('your_select_query');
$results = $query->fetch();

foreach ($results as $result) {
  // ...
}
```

### What is supported ?
* At this time, jSonQL doesn't support SQL functions (MIN, MAX, DECIMAL, NOW, UNIX_TIMESTAMP, etc...)
* jSonQL doesn't understand JOIN in SELECT queries
* Supported SQL operators are <, <=, =, >=, >, <>.

### Examples

#### SELECT
```sql
SELECT field1, field2, field3 [, fieldN]
  FROM nom_table
  [WHERE field1 = val1 AND field2 = val2 AND field3 = val3[ AND fieldN = valN]]
  [ORDER BY fieldX ASC|DESC]
  [LIMIT min, max]
```
```php
$query = $jsonql->query('SELECT * FROM test');
$results = $query->fetch();
/*
$results = array(
  array("id" => 1, "name" => "John Doe", "age" => 25, "male" => true),
  array("id" => 2, "name" => "Stephan Doe", "age" => 18, "male" => false),
  array("id" => 3, "name" => "Ellie Doe", "age" => 23, "male" => false),
  array("id" => 4, "name" => "Isaac Doe", "age" => 10, "male" => true)
);
*/

$query = $jsonql->prepare('SELECT name FROM test WHERE male = :true');
$query->bindValue(':true', true, JSONQL::PARAM_BOOL);
$query->execute();
$results = $query->fetch();
/*
$results = array(
  array("name" => "John Doe"),
  array("name" => "Isaac Doe")
);
*/

$query = $jsonql->query('SELECT age FROM test WHERE id = LAST_INSERT_ID');
$results = $query->fetch();
/*
$results = array(
  array("age" => 10)
);
*/

$query = $jsonql->query('SELECT * FROM test ORDER BY age DESC LIMIT 0, 2');
$results = $query->fetch();
/*
$results = array(
  array("id" => 1, "name" => "John Doe", "age" => 25, "male" => true),
  array("id" => 3, "name" => "Ellie Doe", "age" => 23, "male" => false),
);
*/

// etc...
```

#### INSERT
```sql
INSERT INTO table_name[(field1, field2, field3[, fieldN])]
  VALUE[S] (val1, val2, val3[, valN])
    [, (val1, val2, val3[, valN])]
```
```php
$insert = $jsonql->prepare('INSERT INTO test(name, age, male) VALUE (:nom, :age, :male)');
$insert->bindValue(':nom', "Sylvie Doe", JSONQL::PARAM_STR);
$insert->bindValue(':age', 2, JSONQL::PARAM_INT);
$insert->bindValue(':male', false, JSONQL::PARAM_BOOL);

if ($insert->execute()) {
    echo "Success !";
}
else {
    echo "Error...";
}
```

#### REPLACE
```sql
REPALCE INTO table_name[(field1, field2, field3[, fieldN])]
  VALUE[S] (val1, val2, val3[, valN])
    [, (val1, val2, val3[, valN])]
```

#### DELETE
```sql
DELETE FROM table_name
  [WHERE field1 = val1 AND field2 = val2 AND field3 = val3[ AND fieldN = valN]]
```

#### UPDATE
```sql
UPDATE table_name
  SET field1 = val1, field2 = val2, field3 = val3[, fieldN = valN]
  [WHERE field1 = val1 AND field2 = val2 AND field3 = val3[ AND fieldN = valN]]
```

#### TRUNCATE
```sql
TRUNCATE table_name
```

## Manage JSON databases

### Create database
To create a new JSON database, you have to use the method `JSONQL::create_database()`:
```php
$jsondb->create_database('/path/to/jsondb');
```
The new database will be a folder.

### Create table
To create a new table, you have to first give the path to the database with the constructor or by using the method `JSONQL::setDatabase()`. After, you can use the method `JSONQL::create_table()`:
```php
$jsonql->create_table('table_name', array('field', 'field2', 'field3', 'etc...'));
```
The new table will be a JSON file named **table_name.json** and will content:
```json
{
  "prototype": ["id", "champ1", "champ2", "champ3", "etc..."],
  "properties":
  {
    "last_insert_id": 0
  },
  "datas": []
}
```
The "id" field will be automatically created if you don't.

### Delete database
```php
$jsonql->delete_database('/path/to/jsondb');
```

### Delete table
```php
$jsonql->delete_table('table_name');
```

## Authors
* **Axel Nana**: <ax.lnana@outlook.com> - [https://tutorialcenters.tk](https://tutorialcenters.tk)

## Copyright
(c) 2016 Centers Technologies - Licenced under MIT.
