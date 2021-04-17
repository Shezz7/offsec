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

PHP magic methods are functions that have special properties. All PHP magic methods are defined [here](https://www.php.net/manual/en/language.oop5.magic.php). The magic methods most relevant to deserialization vulnerabilities are ```__wakeup()``` and ```__destruct()```. These methods are executed automatically as soon as unserialize() is called on an object.

Essentially, there are 3 broad steps when working with PHP objects. The object is instantiated, some operations are performed on/with the object and finally the object is destroyed:

#### Instantiate object

The unserialize() method takes a serialized string as input and creates an instance of a class in memory. It then searches for the ```__wakeup()``` function to reconstruct any resources that the object may have. The ```__wakeup()``` method is typically used to reestablish database connections that may have been lost during serialization and to perform other reinitializations.

#### Use object

The object is used by the program and some operations/actions may be performed using it.

#### Destroy object

When no reference to the object exists ```__destruct()``` is called to clean up the object

