# Hash Cracking

Once we identify the type of the hash, we look at techniques to attempt to crack it.

## Hashcat

[https://github.com/hashcat/hashcat](hashcat) is an open source password cracking tool with over 300 supported hash algorithms. It provides a number of different ways to perform the cracking (attack modes). Hashcat supports the following attack modes:

| **#** | **Attack mode** |
|------------|-------------
| 1 | Straight |
| 2 | Combination |
| 3 | Brute-Force |
| 6 | Hybrid Wordlist + Mask |
| 7 | Hybrid Mask + Wordlist |
