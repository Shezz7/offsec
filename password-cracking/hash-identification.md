# Identifying hash types

## Background

There are a large number of hash types used for different purposes. Popular hashing algorithms include md5, sha1, bcyrpt etc. Some hashes are universal and used for a variety of purposes and some hashes are used only in certain applications (eg. MySQL) and protocols. This short writeup will shed some light on how you can identify hash types so you can crack them in an appropriate manner.

## Manually guessing hash types

Based on the following criteria, you can make some educated guesses about the type of the hash that you want to crack:

1. **Length of the hash**: The number of characters in different hash types varies typically. For example, an md5 hash is 128 bits long compared to a sha1 hash which is 160 bits long.
2. **Origin of the hash**: Applications utilize certain (possibly) known hash types. For example, if you acquired the hash from a MySQL application, it will probably use a standardized hashing algorithm. Similarly, Windows uses NTLM for hashing its passwords and Linux uses SHA-256, SHA-512, Blowfish etc.

## Hash identification tools
