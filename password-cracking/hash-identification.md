# Identifying Hash Types

## Background

Hashing is a one way process, the only way to attack it is to use a list containing possible passwords. Each password from this list is hashed and compared to the original hash. There are a large number of hash types used for different purposes such as verifying file integrity, PBKDF2 to hash passwords etc. Popular hashing algorithms include md5, sha1, bcyrpt etc. Some hashes are universal and used for a variety of purposes and some hashes are used only in certain applications (eg. MySQL) and protocols.

Hashes may sometimes be stored in certain formats such as hash:salt or \$id\$salt\$hash. The id parameter is used to identify the type of algorithm used for hashing. eg:

- `$1$:md5`
- `$5$:sha256`
- `$6$:sha512`

Some hash examples:

- apache: `$apr1$71850310$gh9m4xcAn3MGxogwXztb`
- wordpress: `$P$984478476IagS59wHZvyQMArzfx58u`

This short writeup will shed some light on how you can identify hash types so you can crack them in an appropriate manner.

## Manually guessing hash types

Based on the following criteria, you can make some educated guesses about the type of the hash that you want to crack:

1. **Length of the hash**: The number of characters in different hash types varies typically. For example, an md5 hash is 128 bits (32 characters * 4 bits) long compared to a sha1 hash which is 160 bits long.
2. **Origin of the hash**: Applications utilize certain (possibly) known hash types. For example, if you acquired the hash from a MySQL application, it will probably use a standardized hashing algorithm. Similarly, Windows uses NTLM for hashing its passwords and Linux uses SHA-256, SHA-512, Blowfish etc. Hashcat provides an excellent [mapping](https://hashcat.net/wiki/doku.php?id=example_hashes) of hash structures to applications/algorithms.

## Hash identification tools

Disclaimer: You cannot always be sure about what the hash type is because the plaintext may have undergone additional hashing or encryption before being hashed.

### hashid

[hashid](https://github.com/psypanda/hashID) is a python tool that is used to identify hashes. It can be used to guess from around 220 different hash types. Hashes supported by hashid are shown [here](https://github.com/psypanda/hashID/blob/master/doc/HASHINFO.xlsx).

**Basic Usage:**

```bash
$ ./hashid.py -m "098f6bcd4621d373cade4e832627b4f6" # -m shows the hashcat mode
Analyzing '098f6bcd4621d373cade4e832627b4f6'
[+] MD2 
[+] MD5 [Hashcat Mode: 0]
[+] MD4 [Hashcat Mode: 900]
[+] Double MD5 [Hashcat Mode: 2600]
[+] LM [Hashcat Mode: 3000]
```

### hash-identifier

[hash-identifier](https://github.com/Miserlou/Hash-Identifier) is also a python tool to identify hashes. It gives you both the most possible hash types and the least possible ones. The caveat is that it does not display the hashcat mode.

**Basic Usage:**

```bash
$ python Hash_ID.py 
HASH: 098f6bcd4621d373cade4e832627b4f6

Possible Hashes:
[+]  MD5
[+]  Domain Cached Credentials - MD4(MD4(($pass)).(strtolower($username)))

Least Possible Hashes:
[+]  RAdmin v2.x
[+]  NTLM
[+]  MD4

```

### TunnelsUP

[TunnelsUP](https://www.tunnelsup.com/hash-analyzer/) is a website where you can analyzes hash types online. Apart from guessing the hash type it also provides you with example hash inputs of related hashes that you can refer to.

***

**Note:** While creating hashes using md5sum and echo, be sure to use `echo -n`. The `-n` in echo terminates newlines. For example:

```bash
$ echo "test" | md5sum
d8e8fca2dc0f896fd7cb4cb0031ba249  -
$ echo -n "test" | md5sum
098f6bcd4621d373cade4e832627b4f6  -
```
