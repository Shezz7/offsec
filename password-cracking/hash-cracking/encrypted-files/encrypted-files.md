# Password Protected Files

Sometimes we may want to crack the hashes of files that are password protected such as PDF files, SSH private key passwords, Word/Excel files etc. In order to do this we would first have to:
1. Extract the hash from the password protected file
2. Use a cracking tool such as Hashcat to crack the hash

## Extracting hashes from password protected files

There are several tools to extract hashes from files in a format that is digestible by Hashcat. The most popular suite of tools is called JohnTheRipper which consists of a variety of [tools](https://github.com/openwall/john/tree/bleeding-jumbo/src) written in C that can be used to extract hashes. There are also python versions of these tools that are easy to work with.

+ MS Office files

We can use the [office2john.py](https://raw.githubusercontent.com/magnumripper/JohnTheRipper/bleeding-jumbo/run/office2john.py) tool to extract hashes from Microsoft Office documents.

**Syntax:**

```console
python office2john.py testword.docx
```

+ Zip files

We can use the [zip2john.c](https://github.com/openwall/john/blob/bleeding-jumbo/src/zip2john.c) tool to extract hashes from zip files.

**Syntax:**

```console
zip2john testzip.zip
```

+ PDF files

We can use the [pdf2john.py](https://raw.githubusercontent.com/truongkma/ctf-tools/master/John/run/pdf2john.py) tool to extract hashes from PDF documents.

**Syntax:**

```console
python pdf2john.py testpdf.pdf
```

## Cracking extracted hashes with Hashcat

Once we have the extracted hashes we can find their respective Hashcat identifiers using `hashid -m` and then utilize the variety of hash cracking techniques supported by hashcat
