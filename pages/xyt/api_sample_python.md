---
title: Python REST API sample
keywords: sample, python, api
summary: "This sample demonstrates how to directly access REST API from Python."
sidebar: xyt_sidebar
permalink: api_sample_python.html
folder: xyt
---

## Overview

This sample demonstrates how to directly access REST API from Python.

You can also access data using [convenience library](lib_python.html).

## Sample Code

This code snippet demonstrates how to access service **endpoint** with specified **request** and return the **response** content.
Note that the `access_token` is included in the HTTP headers.

```python
def send_request(endpoint, request, token):
    buffer = BytesIO()
    utils.write_delimited_to(request, buffer)

    headers = {'Authorization': 'Bearer ' + token,
               'Content-type': 'application/octet-stream'}
    response = requests.post(endpoint, data = buffer.getvalue(), headers = headers)

    if (response.status_code == 200):
        return response.content
    else:
        try:
            details = response.json()['error_description']
        except:
            details = ''
        raise Exception('Data request: {} failed with HTTP code: {}. {}'.format(request, response.status_code, details))
```

As current version of Google Protocol Buffers Python library doesn't provide means to read/write delimited messages, utility methods have to be
used to interact with the service API:

```python
from google.protobuf.internal import encoder
from google.protobuf.internal import decoder


PROTOCOL_HEADER = 0xFA


def write_delimited_to(message, buffer):
    message_str = message.SerializeToString()
    delimiter = encoder._VarintBytes(len(message_str))
    buffer.write(struct.pack('B', PROTOCOL_HEADER))
    buffer.write(delimiter + message_str)


def read_delimited_from(MessageType, buffer):
    if not buffer[0] == PROTOCOL_HEADER:
        raise Exception('Malformed message header')

    size, pos = decoder._DecodeVarint32(buffer, 1)

    if len(buffer) - pos < size:
        raise Exception('Unexpected end of message')

    message = MessageType()
    message.ParseFromString(buffer[pos:])

    return message
```

{% include links.html %}
