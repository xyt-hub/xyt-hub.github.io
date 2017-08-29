---
title: API overview
keywords: api, overview
summary: "This is the official documentation for xyt's Java package."
sidebar: xyt_sidebar
permalink: api_overview.html
folder: xyt
---

This API allows for robust integration of xyt API and any third party application.

## Authentication

Access to service API requires that each request contains in the HTTP header an authorization token.

The authorization token is obtained as described in [OAuth 2.0 specification](http://tools.ietf.org/html/rfc6749#section-4.3) of
Resource Owner Password Credentials Grant.

The token service is accessible via the url: `AUTH_SERVICE_URL/oauth/token`.
Authentication requires that following parameters are set:

* **user:password**: xyt:SERVICE_SECRET
* **POST data**:
    * grant_type=password
    * username=USERNAME
    * password=PASSWORD


### Testing the authentication

The authentication service can be accessed with the `curl` application:

```shell
curl http://AUTH_SERVICE_URL/oauth/token -u xyt:SECRET -d grant_type=password -d username=USERNAME -d password=PASSWORD
```

A successful authorization results in the JSON response:

```json
{
  "access_token" : "TOKEN",
  "token_type" : "bearer",
  "expires_in" : 43199,
  "scope" : "read"
}
```

Use the obtained `access_token` to request the protected endpoint, e.g.:

```shell
curl http://AUTH_SERVICE_URL/user -H "Authorization: Bearer TOKEN"
```

## Key concepts

The following terms are used in the documentation and the API contract file:

 * **Endpoint** - the REST URL corresponding to a functionality in the service.
 * **Request** - the data object with service request parameters POSTed as a content of the HTTP request.
 * **Response** - the data object representing response of the service.
 * **API contract** - the protocol buffer specification of the service API. Contains detailed description of supported: **endpoints**, **requests**
   and **responses**.


## Data protocol

The service uses binary data protocol based on [Google Protocol Buffer](https://developers.google.com/protocol-buffers/) version 3.0.

The Protocol Buffer wire format is not self-delimiting, so data parsers cannot determine where a message ends on their own. The service
solves this by prefixing each **request** and **response** with the size of the message written as a `varint32`.

The binary message starts with a single byte: protocol marker (`\xFA`), followed by: size of the serialized message and binary representation of the
Protocol Buffer message.

```
 +------+---------------------------+------------------------------------+
 | \xFA | Message Length (varint32) | Serialized Protocol Buffer message |
 +------+---------------------------+------------------------------------+
```

### Data protocol variants

Protocol Buffer message length can also be represented as fixed length int32, serialized in big endian order. To use this variant, different protocol
marker has to be used in a header: `\xFF`.

```
 +------+--------------------------+------------------------------------+
 | \xFF | Message Length (4 bytes) | Serialized Protocol Buffer message |
 +------+--------------------------+------------------------------------+
```


### Protocol Buffers

Protocol Buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data available for a variety of
languages. Each protocol buffer message is a small logical record of information, containing a series of name-value pairs.

The API for the service is specified on the API contract file. The Protocol Buffer compiler has to be used on the API contract file to generate
data access classes. These provide simple accessors for each field as well as methods to serialize/parse the whole structure to/from raw bytes.

Please refer to official documentation for language specific instructions:

 * [Introduction to Google Protocol Buffers](https://developers.google.com/protocol-buffers/)
 * [GitHub page for Google Protocol Buffers](https://github.com/google/protobuf)
 * [Google Protocol Buffers: Java reference](https://developers.google.com/protocol-buffers/docs/reference/java-generated#message)
 * [Google Protocol Buffers: Python reference](https://developers.google.com/protocol-buffers/docs/reference/python-generated#message)
 * [Google Protocol Buffers: C# reference](https://developers.google.com/protocol-buffers/docs/reference/csharp-generated)


## Accessing the API

The service API requires the valid `access_token` to be present in the HTTP header of each request.

### Sample Code for accessing API

 * [Java](api_sample_java.html)
 * [C#](api_sample_csharp.html)
 * [Python](api_sample_python.html)

{% include links.html %}
