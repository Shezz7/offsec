# Introduction

**Serialization** is the process of turning some object into a data format that can be restored later. Objects are often serialized in order to save them to storage or to send as part of communications.

**Deserialization** is the reverse of serialization. It takes structured data from some format and rebuilds it into an object.

Some popular formats for serializing data include JSON and XML.

# Problem

Many programming languages offer a native capability for serializing objects. These native formats usually offer more features than JSON or XML. Some of these capabilities include the customizability of the serialization process for example, magic functions run during serialization or deserialization.

When operating on untrusted data, the features of thse native deserialization mechanisms can be leverage by a malicious actor. The attack against insecure deserialization have a wide gamut from denial-of-service to RCE.

# Mitigation

Mitigation actions specific to different programming languages will be covered in their respective sections. There are a couple of langague agnostic methods for deserializing safely. These include:

## Using alternative data formats

A major reduction of risk is achieved by avoiding native serialization and deserialization formats. By switching to a pure data format such as JSON or XML, the chances of custom deserialization logic used maliciously is reduced.

## Only deserialize signed data

If it is known by the application which messages will need to be processed before deserialization, those message could be signed as part of the serialization process. The application could then refuse to deserialize any message which doesn't have an authenticated signature