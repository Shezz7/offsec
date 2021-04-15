# Serialization in php

When a PHP object is stored or transferred over the network, you need to use ```serialize()``` to pack it up.

```serialize(): \<PHP object\> => plain string representation of the object```

When the data needs to be used, ```unserialize()``` is used to unpack and get the underlying object.

```unserialize(): \<string containing object data\> => original PHP object```

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
