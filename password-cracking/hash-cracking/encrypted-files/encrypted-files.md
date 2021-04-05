# Password Protected Files

Sometimes we may want to crack the hashes of files that are password protected such as PDF files, SSH private key passwords, Word/Excel files etc. In order to do this we would first have to:
1. Extract the hash from the password protected file
2. Use a cracking tool such as Hashcat to crack the hash

## Extracting hashes from password protected files

There are several tools to extract hashes from files in a format that is digestible by Hashcat. The most popular suite of tools is called JohnTheRipper which consists of a variety of [tools](https://github.com/openwall/john/tree/bleeding-jumbo/src) written in C that can be used to extract hashes. There are also python versions of these tools that are easy to work with.

1. MS Office files

We can use the [office2john.py](https://raw.githubusercontent.com/magnumripper/JohnTheRipper/bleeding-jumbo/run/office2john.py) tool to extract hashes from Microsoft Office documents.

**Syntax:**

```console
python office2john.py testword.docx
```

2. Zip files

We can use the [zip2john.c](https://github.com/openwall/john/blob/bleeding-jumbo/src/zip2john.c) tool to extract hashes from zip files.

**Syntax:**

```console
zip2john testzip.zip
```

3. PDF files

We can use the [pdf2john.py](https://raw.githubusercontent.com/truongkma/ctf-tools/master/John/run/pdf2john.py) tool to extract hashes from PDF documents.

**Syntax:**

```console
python pdf2john.py testpdf.pdf
```

