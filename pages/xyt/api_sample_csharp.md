---
title: C# REST API sample
keywords: sample, c#, csharp, api
summary: "This sample demonstrates how to directly access REST API from C#."
sidebar: xyt_sidebar
permalink: api_sample_csharp.html
folder: xyt
---

## C# Overview

This sample demonstrates how to directly access REST API from C#.

You can also access data using [convenience library](lib_csharp.html).

## Sample Code

This code snippet demonstrates how to access service **endpoint** with specified **request** and return the **response** content.

```cs
public static Stream SendRequest(string endpoint, Google.Protobuf.IMessage message, HttpClient httpClient, string authToken)
{
    httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", token);

    using (var buffer = new MemoryStream())
    {
        buffer.WriteByte(PROTOCOL_HEADER);
        message.WriteDelimitedTo(buffer);

        var content = new ByteArrayContent(buffer.ToArray());
        var response = httpClient.PostAsync(endpoint, content).Result;

        response.EnsureSuccessStatusCode();

        var result = response.Content.ReadAsStreamAsync().Result;

        if (result.ReadByte() != PROTOCOL_HEADER)
        {
            throw new IOException("Malformed message header");
        }

        return result;
    }
}
```

{% include links.html %}
