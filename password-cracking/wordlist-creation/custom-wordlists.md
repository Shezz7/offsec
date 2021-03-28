# Creating custom wordlists

Sometimes using common wordlists such as rockyou.txt might not be enough and we may want to take a more targeted approach. In this situation we may want to generate our own custom wordlists to generate our hashes on. There are various open source tools that enable us to do so.

## Hashcat wordlists

Hashcat provides a few ways to create custom wordlists. Some of these include:

### Hashcat utils

[Hashcat-utils](https://github.com/hashcat/hashcat-utils) is a repo that contains a lot of tools for password cracking. Hashcat [maskprocessor](https://github.com/hashcat/maskprocessor) is used to generate custom wordlists based on masks.

### Cracked passoword history

Hashcat stores previously cracked passwords in a file called the potfile. It stores it in the format ```hash:password```. It can be used to generate new wordlists. Additionally, masks can be applied on them to make more complex wordlists.

## Crunch

Crunch can create custom wordlists based on arguments such as length, character sets or certain patterns.

**Syntax:**

```console
crunch <min_length> <max_length> <charset> -t <pattern> -o <outfile>
```

The "-t" option specifies patterns for generated passwords. The pattern may contain

- "@," representing lower case characters
- "," (comma) will insert upper case characters
- "%" will insert numbers
- "^" will insert symbols

### Generate crunch wordlists

**Syntax:**

```console
$crunch 5 10 -o wordlist
```

The command above creates a wordlist consisting of words with a length of 5 to 10 characters, using the default character set.

For example, SSN numbers contain "XXXX" is the employee ID containing letters, and "YYYY" is the year. We can use crunch to create a list of such passwords.

### Generate crunch using patterns

Let's assume that in a company user passwords are of the form "SecurityYYYYXXXX," where "XXXX" is the employee ID containing letters, and "YYYY" is the year. We can use crunch to create a list of such passwords.

**Syntax:**

```console
$crunch 13 13 -t SECURITY201%@@@@ -o wordlist
```

The pattern mentioned above will create a worldlist that has "SECURITY" in the beginning followed by the year 2010-2019 and 4 letters.

## KWPROCESSOR

[Kwprocessor](https://github.com/hashcat/kwprocessor) creates wordlists with consecutive characters on the keyboard. It is common for users to create passwords such as "qwerty". Kwprocessor builds wordlists based on such patterns.

The help page shows the different ways to use kwprocessor:

**Syntax:**

```console
$ ./kwp -h
Advanced keyboard-walk generator with configureable basechars, keymap and routes

Usage: ./kwp [options]... basechars-file keymap-file routes-file

 Options Short / Long        | Type | Description                                                 | Default
=============================+======+=============================================================+=========
  -V, --version              |      | Print version                                               |
  -h, --help                 |      | Print help                                                  |
  -o, --output-file          | FILE | Output-file                                                 |
  -b, --keyboard-basic       | BOOL | Include characters reachable without holding shift or altgr | 1
  -s, --keyboard-shift       | BOOL | Include characters reachable by holding shift               | 0
  -a, --keyboard-altgr       | BOOL | Include characters reachable by holding altgr (non-english) | 0
  -z, --keyboard-all         |      | Shortcut to enable all --keyboard-* modifier                |
  -1, --keywalk-south-west   | BOOL | Include routes heading diagonale south-west                 | 0
  -2, --keywalk-south        | BOOL | Include routes heading straight south                       | 1
  -3, --keywalk-south-east   | BOOL | Include routes heading diagonale south-east                 | 0
  -4, --keywalk-west         | BOOL | Include routes heading straight west                        | 1
  -5, --keywalk-repeat       | BOOL | Include routes repeating character                          | 0
  -6, --keywalk-east         | BOOL | Include routes heading straight east                        | 1
  -7, --keywalk-north-west   | BOOL | Include routes heading diagonale north-west                 | 0
  -8, --keywalk-north        | BOOL | Include routes heading straight north                       | 1
  -9, --keywalk-north-east   | BOOL | Include routes heading diagonale north-east                 | 0
  -0, --keywalk-all          |      | Shortcut to enable all --keywalk-* directions               |
  -n, --keywalk-distance-min | NUM  | Minimum allowed distance between keys                       | 1
  -x, --keywalk-distance-max | NUM  | Maximum allowed distance between keys                       | 1
```

The basechars-file is the file where the route is suppose to start with. The pattern is based on the geographical direction to move in eg. --keywalk-south moves south from the characters in the basechars-file. The final option specifies the route to take. A route defines how passwords will be created, starting from the base characters. For example, the route 323 can denote the path 3 \* EAST + 2 \* SOUTH + 3 \* WEST from the base character. If the base character is considered to be "E" then the password generated by the route would be "ERTGVCXZ".

## CUPP

[Common User Password Profiler (CUPP)](https://github.com/Mebus/cupp) is a python script that is OSINT and social engineering focused. CUPP takes things like pet names, DOBs, phone numbers etc. as input and creates wordlists from them. It is mostly targeted against social media platforms and the like.

**Syntax:**

```console
$ ./cupp.py -h
usage: cupp.py [-h] [-i | -w FILENAME | -l | -a | -v] [-q]

Common User Passwords Profiler

optional arguments:
  -h, --help         show this help message and exit
  -i, --interactive  Interactive questions for user password profiling
  -w FILENAME        Use this option to improve existing dictionary, or WyD.pl
                     output to make some pwnsauce
  -l                 Download huge wordlists from repository
  -a                 Parse default usernames and passwords directly from
                     Alecto DB. Project Alecto uses purified databases of
                     Phenoelit and CIRT which were merged and enhanced
  -v, --version      Show the version of this program.
  -q, --quiet        Quiet mode (don't print banner)
```

Running it with -i prompts cupp for information on the target

```console
➤ ./cupp.py -i
 ___________
   cupp.py!                 # Common
      \                     # User
       \   ,__,             # Passwords
        \  (oo)____         # Profiler
           (__)    )\
              ||--|| *      [ Muris Kurgas | j0rgan@remote-exploit.org ]
                            [ Mebus | https://github.com/Mebus/]


[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: john
> Surname: doe
> Nickname: joe
> Birthdate (DDMMYYYY): 11111984


> Partners) name: mae
> Partners) nickname: doe
> Partners) birthdate (DDMMYYYY): 11111987


> Child's name:
> Child's nickname:
> Child's birthdate (DDMMYYYY):


> Pet's name: click
> Company name: wonderland


> Do you want to add some key words about the victim? Y/[N]: y
> Please enter the words, separated by comma. [i.e. hacker,juice,black], spaces will be removed: waffles,mars,shoes
> Do you want to add special chars at the end of words? Y/[N]: y
> Do you want to add some random numbers at the end of words? Y/[N]:y
> Leet mode? (i.e. leet = 1337) Y/[N]: y

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to john.txt, counting 56392 words.
> Hyperspeed Print? (Y/n) : y
```

This give you a complex wordlist with combinations between the keywords supplied.

## CeWL

[CeWl](https://github.com/digininja/CeWL) is also used to create custom wordlists. It is used primarily to spider through websites and build wordlists based on the contents of the website.

**Syntax:**

```console
$cewl -d <spider_depth> -m <min_length> -w <wordlist_output> <website_url> -e(optional:get_emails)
```
