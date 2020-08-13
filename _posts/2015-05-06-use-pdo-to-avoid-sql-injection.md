---
title: Use PDO to Avoid SQL Injection
date: 2015-05-06T11:00:00+00:00
author: ZerUniverse
layout: post
categories: Web
---

Ten years ago, people talked about SQL Injection. Today, people still talked about this. The difference is, SQL Injection used to be very easy. For some old website, you might simply input<!--more--> `' or 1=1 --` in username box and login yourself without password. Nowadays, people at least use some filtering method to escape so called dangerous characters like `'`. However, this is still not enough to protect database from sophisticated attacks. The best way to solve this problem is telling SQL server SQL query and variables separately instead of splicing SQL string (combining things together and pass the entire SQL query to SQL server). If SQL server knows something is variable, it won't incorrectly treat it as part of structure of SQL query. This is exactly how PDO works to avoid SQL Injection.

I recently upgrade all my PHP code to use PDO to connect MySQL. Here I recommend to use the newest PHP version since there're some vulnerabilities in previous PHP version which allows injection through PDO (by using the vulnerability of encoding, I'll discuss later).

Before I go to PDO, let's first look at some examples on stackoverflow about how injection happens even when filter functions are used. I use `mysql_real_escape_string()` as a example of filter function. It's similar if `addslashes()` is used.

```php
$sql="SELECT title FROM article WHERE id = " . mysql_real_escape_string($_GET["id"])
mysql_query($sql);

//The attack could be
$_GET["id"] = '-1 union all select table_name from information_schema.tables';
//mysql_real_escape_string() will not escape anything in this string

//The final SQL query processed by SQLServer is
//SELECT ... WHERE id = -1 union all select table_name from information_schema.tables
//which is totally not the thing we want
```

Even if we use single quote characters, we can't entirely avoid SQL injections.

```php
mysql_query('SET NAMES gbk');
$var = mysql_real_escape_string($_POST['name']);
mysql_query("SELECT * FROM test WHERE name = '$var' LIMIT 1");

//The attack is
$_POST['name']="\xbf\x27 OR 1=1 /*";

//mysql_real_escape_string() can't find any dangerous characters here. But in GBK, what we really post to server is:
$_POST['name']="縗' OR 1=1 /*";
```

Even we find a really secure escape function one day, this is still not good enough. Because we change the original string the user inputs.

PDO is a new database connection abstraction library for PHP5, which supports giving SQLserver query and variables separately. By this way, we don't need to pre-process user inputs at all because we know SQLserver will treat all user inputs as variables. However, for PHP version before 5.3.6, there's a vulnerability that may leads to SQL injection. The reason is these versions use native prepare (prepare the whole SQL query and then pass to SQL server) by default while ascii is always used as character set. If SQL server doesn't use ascii, there might be some vulnerabilities attackers can make use of. If you must use these versions, you should let SQLserver do such preparations instead of letting PHP do them. You can set this option by the following PHP script after you create a new PDO object.

```php
//$db = new PDO(...);
$db->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
```

For compatibility and portability reason, I recommend you always add this in your PHP code. The following script is my version of MySQL connection library.

```php
function sqllink()
{
    $dbhost="localhost";
    $dbname="xxx";
    $dbusr="xxx";
    $dbpwd="xxx";
    $db=NULL;
    $opt = array(PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8',);
    $dsn='mysql:host=' . $dbhost . ';dbname=' . $dbname.';charset=utf8';
    try {
        $db = new PDO($dsn, $dbusr, $dbpwd, $opt);
        $db->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
        $db->setAttribute(PDO::ATTR_ERRMODE,PDO::ERRMODE_EXCEPTION);
    }
    catch (PDOExceptsddttrtion $e) {
        return NULL;
    }
    return $db;
}
function sqlexec($sql,$array,$link)
{
    $stmt = $link->prepare($sql);
    $exeres = $stmt->execute($array);
    if($exeres) return $stmt; else return NULL;

}
function sqlquery($sql,$link)
{
    return $link->query($sql);
}
```

Then I can just simply communicate with MySQL database in my code

```php
require_once("sqllink.php");
//First connect to MySQL DB
$link = sqllink();
if($link == NULL) die("error");

//make query involves no user inputs
$sql = "SELECT COUNT(*) FROM sometable";
$result = sqlquery($sql,$link);
$result = $result->fetch(PDO::FETCH_NUM);
$num = $result[0];

//make query involves user inputs
$sql = "UPDATE sometable SET x= ?, y=?, z=? WHERE a=? AND b=? AND c=?";
//? is placeholder
$res=sqlexec($sql,array($_POST['x'],$_POST['y'],$_POST['z'],$_POST['a'],$_POST['b'],$_POST['c']),$link);
if($res != NULL) echo "success"; else echo "fail";
```

By the way, it's also easy to make a transaction with PDO.

```php
require_once("sqllink.php");
$link = sqllink();
if($link == NULL) die("error");
if(!$link->beginTransaction()) die('error');
try {
       /*DO SOME THING*/
       $link->commit();
       //FINISH
}
catch (Exception $e){
      $link->rollBack();
      //recover the database to beginTransaction
      //change made after beginTransaction will be erased
      /*DO SOME THING*/
}

//rollBack() or commit() is the end of a transaction
```