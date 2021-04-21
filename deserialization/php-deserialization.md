# PHP Deserialization Exploitation

## Serialization in php

When a PHP object is stored or transferred over the network, you need to use ```serialize()``` to pack it up.

```serialize(): \<PHP object\> => plain string representation of the object```

Here's an example code snippet to demonstrate what PHP serialization looks like:

```php
<?php

class User{
public $username;
public $role;
public $SSN;
}

$user = new User;
$user -> username = 'bob';
$user -> role = 'admin';
$user -> SSN = '199001016666';

print(serialize($user));
?>
```

Running the code we get a serialized string that represents the ```user``` object:

```O:4:"User":3:{s:8:"username";s:3:"bob";s:4:"role";s:5:"admin";s:3:"SSN";s:12:"199001016666";}```

Where O is object and s is string. This is in the format:

```O:LENGTH_OF_OBJ_NAME:"CLASS_NAME":NUMBER_OF_PROPERTIES:{PROPERTIES}```

## Deserialization in php

When the data needs to be used, ```unserialize()``` is used to unpack and get the underlying object.

```unserialize(): \<string containing object data\> => original PHP object```

Here's an example to demonstrate deserialization:

```php
<?php

class User{
public $username;
public $role;
public $SSN;
}

$user = new User;
$user -> username = 'bob';
$user -> role = 'admin';
$user -> SSN = '199001016666';

$serialized = serialize($user);
$unserialized = unserialize($serialized);

var_dump($unserialized);
var_dump($unserialized -> username);
var_dump($unserialized -> role);
var_dump($unserialized -> SSN);
?>
```

This code snippet serializes the ```user``` object, deserializes it and displays its properties one by one. The output is as follows:

```console
object(User)#2 (3) {
  ["username"]=>
  string(3) "bob"
  ["role"]=>
  string(5) "admin"
  ["SSN"]=>
  string(12) "199001016666"
}
string(3) "bob"
string(5) "admin"
string(12) "199001016666"
```

## Exploring the unserialize() method

Let's have a closer look at the unserialize method

### PHP magic methods

PHP magic methods are functions that have special properties. All PHP magic methods are defined [here](https://www.php.net/manual/en/language.oop5.magic.php). The magic methods most relevant to deserialization vulnerabilities is the ```__wakeup()``` method. This method is executed automatically as soon as unserialize() is called on an object.

Essentially, there are 3 broad steps when working with PHP objects. The object is instantiated, some operations are performed on/with the object and finally the object is destroyed:

#### Instantiate object

The unserialize() method takes a serialized string as input and creates an instance of a class in memory. It then searches for the ```__wakeup()``` function to reconstruct any resources that the object may have. The ```__wakeup()``` method is typically used to reestablish database connections that may have been lost during serialization and to perform other reinitializations.

#### Use object

The object is used by the program and some operations/actions may be performed using it.

#### Destroy object

When no reference to the object exists ```__destruct()``` is called to clean up the object

## Exploiting PHP deserialization

When an attacker has control over a serialized string that is passed into the ```unserialize()``` method, they control the properties of the created object. Another thing that is possible is to control the flow of the program by manipulating the values passed into magic methods such as ```__wakeup()```.

This technique is called PHP object injection. PHP object injection can lead to variable manipulation, RCE, SQL injection etc.

### Modifying properties

One possible way of exploiting a PHP object injection vulnerabilities is modifying the properties in a serialized object and passing that as input. eg. if we have the following serialized object:

```O:4:"User":3:{s:8:"username";s:3:"bob";s:4:"role";s:5:"user";}```

We could potentially pass in the string by changing the ```role``` property which defines the privilege of the username "bob" as follows:

```O:4:"User":3:{s:8:"username";s:3:"bob";s:4:"role";s:5:"admin";}```

### Remote code execution

It may be possible to get RCE using PHP object injection through one of the magic methods. For example if we have the following ```__wakeup()``` function:

```php
function __wakeup(){
      if (isset($this->hook)) eval($this->hook);
  }
```

This function executes whatever is inside the ```hook``` property within the object that is undergoing deserialization. If we were able to control the input to the object undergoing deserialization, we could effectively acheive RCE.

## PHP Object injection examples

PHP object injection attacks occur when the attacker controls the input to a variable that will be deserialized. Extending the usage of the wakeup function above, we have the following vulnerable examples:

### Example 1: RCE through cookie injection

```php
class Example
{
  private $hook;

  function __wakeup(){
      if (isset($this->hook)) eval($this->hook);
  }
}

........

$user_input = unserialize($_COOKIE['data']);
```

In the above example, since the input ```data``` cookie is unserialized, an attacker could set the cookie with a serialized string. The string when unserialized will call the wakeup function and the ```hook``` property will get executed. We could inject this property with malicious code and acheive RCE as follows:

```php
class Example
{
    private $hook = "phpinfo();";
}

print urlencode(serialize(new Example));
```

The URL encoding is done since we would pass this string through a URL. The generated string looks as follows:

Before URL encoding:
```O:7:"Example":1:{s:13:"Examplehook";s:10:"phpinfo();";}```

After URL encoding:
```O%3A7%3A%22Example%22%3A1%3A%7Bs%3A13%3A%22%00Example%00hook%22%3Bs%3A10%3A%22phpinfo%28%29%3B%22%3B%7D```

Passing the string above into the ```data``` cookie, the ```phpinfo();``` function will be executed. The flow of the program looks as follows:

1. A serialized ```Example``` object is passed into the program as the ```data``` cookie
2. The program calls ```unserialize()``` on the ```data``` cookie
3. Because the ```data``` cookie is a serialized ```Example``` object, ```unserialize()``` instantiates a new ```Example``` object
4. The magic method ```__wakeup()``` is available so at this point, it is called
5. The ```$hook``` property of the object is checked for and if it is not NULL, it is executed through ```eval()```
