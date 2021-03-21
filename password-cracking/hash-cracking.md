# Hash Cracking

Once we identify the type of the hash, we look at techniques to attempt to crack it.

## Hashcat

[Hashcat]([hashcat](https://github.com/hashcat/hashcat)) is an open source password cracking tool with over 300 supported hash algorithms. It provides a number of different ways to perform the cracking (attack modes). The help page of hashcat, along with basic usage and some options is shown below:

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
| 1 | Straight |
| 2 | Combination |
| 3 | Brute-Force |
| 6 | Hybrid Wordlist + Mask |
| 7 | Hybrid Mask + Wordlist |

### Hash examples

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

### Hashcat performance test

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
