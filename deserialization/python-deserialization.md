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
print(pickled_data)
```

Running the code we get a serialized string that represents the ```user``` object:

```b'\x80\x04\x95%\x00\x00\x00\x00\x00\x00\x00}\x94(\x8c\x08username\x94\x8c\x03bob\x94\x8c\x04role\x94\x8c\x05admin\x94u.'```
