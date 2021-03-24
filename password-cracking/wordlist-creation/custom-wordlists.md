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
$crunch 15 15 -t SECURITY%@@@@ -o wordlist
```

## KWPROCESSOR

## Princeprocessor

## CUPP

## CeWL