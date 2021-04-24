# Python Deserialization Exploitation

## Serialization in python

In python, the default library used for serialization and deserialization is [pickle](https://docs.python.org/3.6/library/pickle.html).

```pickle.dumps(python_object): => plain string representation of the object```

Here's an example code snippet to demonstrate what serialization with pickle looks like:

```python
import pickle

user = {}
user['username'] = 'bob'
user['role'] = 'admin'
pickled_data = pickle.dumps(user)
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

user = {}
user['username'] = 'bob'
user['role'] = 'admin'
pickled_data = pickle.dumps(user)
unpickled_data = pickle.loads(pickled_data)
print(type(unpickled_data))
print(unpickled_data)
```

This code snippet serializes the user object of type *dict* to bytes and then deserializes it back to the *dict* object. The output is as follows:

```console
<class 'dict'>
{'username': 'bob', 'role': 'admin'}
```
