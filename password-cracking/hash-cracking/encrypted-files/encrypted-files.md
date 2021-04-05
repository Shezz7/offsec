# Encrypted Files

Sometimes we may want to crack the hashes of files that are encrypted such as PDF files, SSH private key passwords, Word/Excel files etc. In order to do this we would first have to:
1. Extract the hash from the encrypted file
2. Use a cracking tool such as Hashcat to crack the hash

## Extracting hashes from encrypted files

There are several tools to extract hashes from files in a format that is digestible by Hashcat. The most popular suite of tools is called JohnTheRipper which consists of a variety of [tools](https://github.com/openwall/john/tree/bleeding-jumbo/src) written in C that can be used to extract hashes. There are also python versions of these tools that are easy to work with.