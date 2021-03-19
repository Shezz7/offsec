# Identifying hash types

## Background

Hashing is a one way process, the only way to attack it is to use a list containing possible passwords. Each password from this list is hashed and compared to the original hash. There are a large number of hash types used for different purposes such as verifying file integrity, PBKDF2 to hash passwords etc. Popular hashing algorithms include md5, sha1, bcyrpt etc. Some hashes are universal and used for a variety of purposes and some hashes are used only in certain applications (eg. MySQL) and protocols.

Hashes may sometimes be stored in certain formats such as hash:salt or \$id\$salt\$hash. The id parameter is used to identify the type of algorithm used for hashing. eg:

- \$1\$:md5
- \$5\$:sha256
- \$6\$:sha512

Some hash examples:
- apache: \$apr1\$71850310\$gh9m4xcAn3MGxogwX/ztb
- wordpress: \$P\$984478476IagS59wHZvyQMArzfx58u

This short writeup will shed some light on how you can identify hash types so you can crack them in an appropriate manner.

## Manually guessing hash types

Based on the following criteria, you can make some educated guesses about the type of the hash that you want to crack:

1. **Length of the hash**: The number of characters in different hash types varies typically. For example, an md5 hash is 128 bits long compared to a sha1 hash which is 160 bits long.
2. **Origin of the hash**: Applications utilize certain (possibly) known hash types. For example, if you acquired the hash from a MySQL application, it will probably use a standardized hashing algorithm. Similarly, Windows uses NTLM for hashing its passwords and Linux uses SHA-256, SHA-512, Blowfish etc.

## Hash identification tools
