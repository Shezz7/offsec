# Python Deserialization Exploitation

## Serialization in python

In python, the default library used for serialization and deserialization is [pickle](https://docs.python.org/3.6/library/pickle.html).

```pickle.dumps(python_object): => plain string representation of the object```

Here's an example code snippet to demonstrate what serialization with pickle looks like:

```python
import pickle


class User(object):
    username = 'bob'
    role = 'admin'


pickled_data = pickle.dumps(User)

print(type(pickled_data))
print(pickled_data)
```

Running the code we get a serialized string that represents the ```user``` object.

```console
<class 'bytes'>
b'\x80\x04\x95%\x00\x00\x00\x00\x00\x00\x00}\x94(\x8c\x08username\x94\x8c\x03bob\x94\x8c\x04role\x94\x8c\x05admin\x94u.'
```

This has now converted an object of type *dict* to a stream of bytes.

## Deserialization in python

When the data needs to be used, ```pickle.loads()``` is used to unpack and get the underlying object.

```pickle.loads(serialized_bytes): => original python object```

Here's an example to demonstrate deserialization with pickle:

```python
import pickle


class User(object):
    username = 'bob'
    role = 'admin'

pickled_data = pickle.dumps(User)
unpickled_data = pickle.loads(pickled_data)


print(type(unpickled_data))
print(unpickled_data.username)
print(unpickled_data.role)
```

This code snippet serializes the user object of type *dict* to bytes and then deserializes it back to the *dict* object. The output is as follows:

```console
<class 'type'>
bob
admin
```

## Manipulating the behaviour of pickling/unpickling

Pickle provides a way to execute arbitrary commands through the use of its ```__reduce__()``` method. To understand how this is possible, we must first take a look at the working of the ```__reduce__()``` method.

### __reduce__() explained

Pickle has certain restrictions as in not every object can be serialized. Objects such as file handles and others come with certain restrictions. The pickle docs give an overview of [what can be pickled and unpickled](https://docs.python.org/3/library/pickle.html#what-can-be-pickled-and-unpickled).

For the special cases where an object cannot be pickled, pickle allows you to define custom behaviours of the pickling process for your class instances. Let's take an example of what happens when a file descriptor is serialized:

```python
import pickle


class Demo(object):
    def __init__(self, file_path):
        self.some_random_file = open(file_path, 'wb')    # Open a file in write mode


file_tester = Demo('/home/bob/test.txt')
pickle.dumps(file_tester)    # Attempt to serialize an object containing a file descriptor
```

As expected we get the following error:

```console
Traceback (most recent call last):
  File "/Users/shehzaad.saifulla/src/offsec/deserialization/test.py", line 22, in <module>
    pickle.dumps(file_tester)                   # Attempt to serialize an object containing a file descriptor
TypeError: cannot pickle '_io.BufferedWriter' object
```

This is where the ```__reduce__()``` method comes into play. We can define how we want pickle to serialize this object.