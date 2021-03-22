# Hashcat

[Hashcat](https://github.com/hashcat/hashcat) is an open source password cracking tool with over 300 supported hash algorithms. It provides a number of different ways to perform the cracking (attack modes). The help page of hashcat, along with basic usage and some options is shown below:

```console
$ hashcat -h
hashcat (v6.1.1-120-g15bf8b730) starting...

Usage: hashcat [options]... hash|hashfile|hccapxfile [dictionary|mask|directory]...

- [ Options ] -

 Options Short / Long           | Type | Description                                          | Example
================================+======+======================================================+=======================
 -m, --hash-type                | Num  | Hash-type, see references below                      | -m 1000
 -a, --attack-mode              | Num  | Attack-mode, see references below                    | -a 3
 -V, --version                  |      | Print version                                        |
 -h, --help                     |      | Print help                                           |
     --quiet                    |      | Suppress output                                      |
     --hex-charset              |      | Assume charset is given in hex                       |
     --hex-salt                 |      | Assume salt is given in hex                          |
     --hex-wordlist             |      | Assume words in wordlist are given in hex            |
     --force                    |      | Ignore warnings                                      |
     --status                   |      | Enable automatic update of the status screen         |
     --status-json              |      | Enable JSON format for status ouput                  |
     --status-timer             | Num  | Sets seconds between status screen updates to X      | --status-timer=1
     --stdin-timeout-abort      | Num  | Abort if there is no input from stdin for X seconds  | --stdin-timeout-abort=300
```

Hashcat supports the following attack modes:

| **#** | **Attack mode** |
|------------|-------------
| 0 | Dictionary |
| 1 | Combination |
| 3 | Brute-Force |
| 6 | Hybrid Wordlist + Mask |
| 7 | Hybrid Mask + Wordlist |

## Hashcat examples

Hashcat has the option to show example hashes of the different hash types it supports including the hashcat mode number.

```console
$ hashcat --example-hashes | head -n 21
hashcat (v6.1.1-120-g15bf8b730) starting...

MODE: 0
TYPE: MD5
HASH: 8743b52063cd84097a65d1633f5c74f5
PASS: hashcat

MODE: 10
TYPE: md5($pass.$salt)
HASH: 3d83c8e717ff0e7ecfe187f088d69954:343141
PASS: hashcat

MODE: 11
TYPE: Joomla < 2.5.18
HASH: b78f863f2c67410c41e617f724e22f34:89384528665349271307465505333378
PASS: hashcat

MODE: 12
TYPE: PostgreSQL
HASH: 93a8cf6a7d43e3b5bcd2dc6abb3e02c6:27032153220030464358344758762807
PASS: hashcat

$ hashcat --example-hashes | grep MODE | wc -l
     338
```

## Hashcat performance test

You can test how many hashes per second your hardware is capable of producing (for a specific hash type) by running the following:

```console
$ hashcat -b -m 0
hashcat (v6.1.1-120-g15bf8b730) starting in benchmark mode...

Benchmarking uses hand-optimized kernel code by default.
You can use it in your cracking session by setting the -O option.
Note: Using optimized kernel code limits the maximum supported password length.
To disable the optimized kernel code in benchmark mode, use the -w option.

OpenCL API (OpenCL 1.2 (Dec 21 2020 17:26:36)) - Platform #1 [Apple]
====================================================================
* Device #1: Intel(R) Core(TM) i7-8569U CPU @ 2.80GHz, 16320/16384 MB (4096 MB allocatable), 8MCU
* Device #2: Intel(R) Iris(TM) Plus Graphics 655, skipped

Benchmark relevant options:
===========================
* --optimized-kernel-enable

Hashmode: 0 - MD5

Speed.#1.........:   347.9 MH/s (23.89ms) @ Accel:1024 Loops:1024 Thr:1 Vec:4

Started: Sun Mar 21 22:09:51 2021
Stopped: Sun Mar 21 22:10:00 2021
```

## Hashcat Attack Types

Lets have a deeper look at each of the hashcat attack types.

### Dictionary Attack

It uses a wordlist, generates the hashes of the words in the wordlist and compares them to the hash that needs to be cracked. The best source to lists of leaked usernames, passwords, wordlists (you name it!) is [SecLists](https://github.com/danielmiessler/SecLists).

**Syntax:**

```console
hashcat -a 0 -m <hash_type_id> <hash_to_crack || hash_file> <wordlist>
```

### Combination Attack

It takes two separate wordlists as input and creates combinations from them. For example if wordlist1 has 4 words and wordlist2 has 2 words then hashcat generates hashes for a total of 4 x 2 = 8 words.

**Syntax:**

```console
hashcat -a 1 -m <hash_type_id> <hash_to_crack> <wordlist1> <wordlist2>
```

### Mask Attack

Masks can be used to match words that have a specific pattern. They are the most useful in cases where the length of the password or its structure is known. Masks can be created using various built in characters:

| **Built-in Charsets** | **Description** |
|------------|-------------
| ?l | abcdefghijklmnopqrstuvwxyz |
| ?u | ABCDEFGHIJKLMNOPQRSTUVWXYZ |
| ?d | 0123456789 |
| ?h | 0123456789abcdef |
| ?H | 0123456789ABCDEF |
| ?s | «space»!"#$%&'()*+,-./:;<=>?@[\]^_`{|}~ |
| ?a | ?l?u?d?s |
| ?b | 0x00 - 0xff |

Hashcat also provides the ability to define custom charsets which are described by the notations **-1** to **-4**. For example, to define multiple custom charsets of alphabets and numbers we can use '```-1 abcdfgke -2 3456```'. An example of a masked word for a number that represents a year between 2000-2009 would be ```200?d```.

**Syntax:**

```console
hashcat -a 3 -m <hash_type_id> <hash_to_crack> <custom_charsets> <masked_word>
```

### Hybrid Mode

Hybrid mode is similar to the combination attack. It uitilizes multiple modes to generate wordlists. This mode is useful when you have a good idea about the length and structure of the password. There are two different kinds of attack modes under this mode:

- -6: Words from the wordlist are read and a string is appended to them based on the mask provided
- -7: Words from the wordlist are prepended with words that are generated through a given mask

**Syntax:**

To append, the syntax is as follows:

```console
hashcat -a 6 -m <hash_type_id> <hash_to_crack> <wordlist> <mask_append>
```

For example to append 2 digits and a lowercase letter to a wordlist we do the following:

```console
hashcat -a 6 -m <hash_type_id> <hash_to_crack> <wordlist> '?d?d?l'
```

To prepend, the syntax is as follows:

```console
hashcat -a 7 -m <hash_type_id> <hash_to_crack> <custom_charset> <mask_prepend> <wordlist>
```

For example, to prepend 2 digits and a lowercase letter to a wordlist we do the following:

```console
hashcat -a 7 -m <hash_type_id> <hash_to_crack> <custom_charset> '?d?d?l' <wordlist>
```
