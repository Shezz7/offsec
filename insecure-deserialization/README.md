# Insecure Deserialization

![serialize_deserialize](https://github.com/Shezz7/offsec/blob/master/insecure-deserialization/resources/serialize_deserialize.png)

**Serialization** is the process of turning some object into a data format that can be restored later. Objects are often serialized in order to save them to storage or to send as part of communications.

**Deserialization** is the reverse of serialization. It takes structured data from some format and rebuilds it into an object.

Some popular formats for serializing data include JSON and XML.

## Problem

![serialize_deserialize](https://github.com/Shezz7/offsec/blob/master/insecure-deserialization/resources/malicious.png)

Problems arise when an attacker has the ability to control and manipulate the serialized object and as a result cause malicious effects in the program. For example if an application takes in a serialized object as input from a user to determine the privileges of the user who is logged in, an attacker could modify the serialized object and authenticate as someone they are not. Unsafe deserialization could also lead to code execution in certain cases when an attacker embeds code into the serialized objects.

Insecure deserialization vulnerabilities occur when applications deserialize objects without proper sanitization. When operating on untrusted data, the features of thse native deserialization mechanisms can be leveraged by a malicious actor. The attack against insecure deserialization have a wide gamut from denial-of-service to RCE.

Many programming languages offer a native capability for serializing objects. These native formats usually offer more features than JSON or XML. Some of these capabilities include the customizability of the serialization process for example, magic functions run during serialization or deserialization. Almost every serialization framework or library for example in [python](https://docs.python.org/3/library/pickle.html) and [php](https://www.php.net/manual/en/function.unserialize.php), will strongly recommend deserializing data coming from a safe location. For programming language specific explanation of deserialization vulnerabilities:

- [PHP](https://github.com/Shezz7/offsec/blob/master/insecure-deserialization/php-deserialization.md)
- [Python](https://github.com/Shezz7/offsec/blob/master/insecure-deserialization/python-deserialization.md)

## Exploiting Insecure Deserialization

There are several ways to exploit insecure deserialization. It really depends on the way the application is implemented and whether or not user input is unserialized in one way or another. Broadly, we can catergorize some exploitation techniques as follows:

### Modifying object attributes

An attacker can tamper the data and as long as they maintain the valid format of the serialized object, the deserialization process will create an object on the server-side with the modified attributs.  eg. If an attacker if able to manipulate and send the below mentioned byte stream by changing the boolean value of ```isAdmin``` and say the server uses this cookie to check admin access, the attacker could gain privileged access

```O:4:"User":2:{s:8:"username";s:3:"bob";s:7:"isAdmin";b:0;}```

This is not a very common scenario but merely a demonstration of how this vulnerability may be abused.

## Mitigation

The only safe architectural pattern is to not accept serialized objects from untrusted sources or to use serialization mediums that only permit primitive data types. Mitigation actions specific to different programming languages will be covered in their respective sections. There are some language agnostic methods for deserializing safely. These include:

- Avoiding native serialization and deserialization formats. By switching to a pure data format such as JSON or XML, the chances of custom deserialization logic used maliciously is reduced

- Signing the emessages as part of the serialization process if it is known by the application which messages will need to be processed before deserialization. The application could then refuse to deserialize any message which doesn't have an authenticated signature

- Logging deserialization exceptions and failures, such as when the incoming type is unexpected

- Monitoring and alerting if a user deserializes constantly

## References

- <https://owasp.org/www-project-top-ten/2017/A8_2017-Insecure_Deserialization>
- <https://cheatsheetseries.owasp.org/cheatsheets/Deserialization_Cheat_Sheet.html>
- <https://portswigger.net/web-security/deserialization/exploiting>
- <https://dan.lousqui.fr/explaining-and-exploiting-deserialization-vulnerability-with-python-en.html>
