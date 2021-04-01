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

To append, the syntax is as follows:

**Syntax:**

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

## Hashcat Rules

Rules help you perform various functions on the input wordlist such as reversing, prefixing/suffixing characters, changing letters to special characters etc. The following table describes some rule functions: (full list [here](https://hashcat.net/wiki/doku.php?id=rule_based_attack#implemented_compatible_functions))

| **Function** | **Description** |
|----------|-------------------------------|
|    l     | Convert all letters to lowercase |
|    u     | Convert all letters to uppercase |
|   c/C    | capitalize / lowercase first letter and invert the rest |
|   t/TN   | Toggle case : whole word / at position N |
| d/q/zN/ZN| Duplicate word / all characters / first character / last character |
|   {/}    | Rotate word left / right |
|  ^X/$X   | Prepend / Append character X |
|    r     | Reverse |

### Sample rule creation

For example if a company has a password policy that ends with the current year eg. 2021, then we use the following rules on a wordlist:

```c so0 si1 s5$ se3 $2 $0 $2 $1```

The first letter of the word is capital and then the rule uses the substitute ffunction ```s``` to substitute the letter with the special characters.

To test combinations in hashcat you can use the following syntax:

**Syntax:**

```console
$echo 'c so0 si1 s5$ se3 $2 $0 $2 $1' > rule.txt
$hashcat -r rule.txt <wordlist> --stdout
```

### Cracking hashes using rules

To crack hashes using rules do the following:

**Syntax:**

```console
$hashcat -a 0 -m <hash_id> <hash_to_crack> <path_to_wordlist> -r <path_to_rules_file>
```

### Hashcat default rules

Hashcat has a lot of prebuilt rulesets that are usually better to try before building your own ruleset:

```console
$ls -l /usr/share/hashcat/rules/

total 2576
-rw-r--r-- 1 root root    933 Jun 19 06:20 best64.rule
-rw-r--r-- 1 root root    633 Jun 19 06:20 combinator.rule
-rw-r--r-- 1 root root 200188 Jun 19 06:20 d3ad0ne.rule
-rw-r--r-- 1 root root 788063 Jun 19 06:20 dive.rule
-rw-r--r-- 1 root root 483425 Jun 19 06:20 generated2.rule
-rw-r--r-- 1 root root  78068 Jun 19 06:20 generated.rule
drwxr-xr-x 1 root root   2804 Jul  9 21:01 hybrid
-rw-r--r-- 1 root root 309439 Jun 19 06:20 Incisive-leetspeak.rule
-rw-r--r-- 1 root root  35280 Jun 19 06:20 InsidePro-HashManager.rule
-rw-r--r-- 1 root root  19478 Jun 19 06:20 InsidePro-PasswordsPro.rule
-rw-r--r-- 1 root root    298 Jun 19 06:20 leetspeak.rule
-rw-r--r-- 1 root root   1280 Jun 19 06:20 oscommerce.rule
-rw-r--r-- 1 root root 301161 Jun 19 06:20 rockyou-30000.rule
-rw-r--r-- 1 root root   1563 Jun 19 06:20 specific.rule
-rw-r--r-- 1 root root  64068 Jun 19 06:20 T0XlC-insert_00-99_1950-2050_toprules_0_F.rule
-rw-r--r-- 1 root root   2027 Jun 19 06:20 T0XlC-insert_space_and_special_0_F.rule
-rw-r--r-- 1 root root  34437 Jun 19 06:20 T0XlC-insert_top_100_passwords_1_G.rule
-rw-r--r-- 1 root root  34813 Jun 19 06:20 T0XlC.rule
-rw-r--r-- 1 root root 104203 Jun 19 06:20 T0XlCv1.rule
-rw-r--r-- 1 root root     45 Jun 19 06:20 toggles1.rule
-rw-r--r-- 1 root root    570 Jun 19 06:20 toggles2.rule
-rw-r--r-- 1 root root   3755 Jun 19 06:20 toggles3.rule
-rw-r--r-- 1 root root  16040 Jun 19 06:20 toggles4.rule
-rw-r--r-- 1 root root  49073 Jun 19 06:20 toggles5.rule
-rw-r--r-- 1 root root  55346 Jun 19 06:20 unix-ninja-leetspeak.rule
```

### Generate random hashcat rules

Hashcat also has the option to create random rules on the fly and apply it on a wordlist. The ```-g``` flag will generate x random rules and apply them to the worldlist selected

**Syntax:**

```console
$hashcat -a 0 -m <hash_id> -g 100 <hash_to_crack> <path_to_wordlist>
```
