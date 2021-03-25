# Creating custom wordlists

Sometimes using common wordlists such as rockyou.txt might not be enough and we may want to take a more targeted approach. In this situation we may want to generate our own custom wordlists to generate our hashes on. There are various open source tools that enable us to do so.

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


## Princeprocessor

## CUPP

[Common User Password Profiler (CUPP)](https://github.com/Mebus/cupp) is a python script that is OSINT and social engineering focused. CUPP takes things like pet names, DOBs, phone numbers etc. as input and creates wordlists from them. It is mostly targeted against social media platforms and the like.

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