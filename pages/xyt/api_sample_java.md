---
title: Java REST API sample
keywords: sample, java, api
summary: "This sample demonstrates how to directly access REST API from Java."
sidebar: xyt_sidebar
permalink: api_sample_java.html
folder: xyt
---

## Overview

This sample demonstrates how to directly access REST API from Java.

You can also access data using [convenience library](lib_java.html).

## Sample Code

This code snippet demonstrates how to access service **endpoint** with specified **request** and return the **response** content.

```java
public static InputStream sendRequest(final String endpoint, final Message request, final HttpRequestFactory requestFactory) throws IOException {
    final ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
    outputStream.write(PROTOCOL_MARKER);
    request.writeDelimitedTo(outputStream);
    final HttpContent httpContent = new ByteArrayContent(MediaType.OCTET_STREAM.toString(), outputStream.toByteArray());
    final HttpRequest httpRequest = requestFactory.buildPostRequest(new GenericUrl(serviceUrl + endpoint), httpContent);
    final HttpResponse httpResponse = httpRequest.execute();

    final InputStream dataStream = httpResponse.getContent();
    if (dataStream.read() != PROTOCOL_MARKER) {
        throw new IOException("Malformed message header.");
    }

    return dataStream;
}
```

Note that the `access_token` is set during the creation of the `HttpRequestFactory`:

```java
final HttpRequestFactory requestFactory = HTTP_TRANSPORT.createRequestFactory(request -> {
            final HttpHeaders headers = new HttpHeaders();
            headers.setAuthorization("Bearer " + accessToken);
            request.setHeaders(headers);
            request.setConnectTimeout(60000);
            request.setReadTimeout(60000);
        });
```


{% include links.html %}
