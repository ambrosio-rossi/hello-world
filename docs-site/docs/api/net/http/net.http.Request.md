---
title: $.net.http.Request
---

$.net.http.Request
===

`$.net.http.Request` class to be used with HTTP client. It extends `$.web.WebRequest`.

## Overview

- Definition: [https://github.com/SAP/xsk/issues/20](https://github.com/SAP/xsk/issues/20)
- Module: [http/http.js](https://github.com/SAP/xsk/tree/main/modules/api/api-xsjs/src/main/resources/META-INF/dirigible/xsk/http/http.js)
- Status: `alpha`

## Sample Usage

=== "request-sample.xsjs"

    ```javascript
    let http = $.net.http;

    let dest = http.readDestination("Demo", "service");
    // create client
    let client = new http.Client();
    // create Request class with the HTTP method constants as a first argument and the path of the resource as a second
    let request = new http.Request(http.GET, "/"); // new Request(METHOD, PATH)
    // the PATH will be prefixed by destination's pathPrefix, e.g. "/search?" on the request

    // Use the Request class to send an http request with the net.http.Client class
    client.request(request, dest);
    ```

## Properties

| Name            | Description                                                                                                                                                                                                                                                                                             | Type                 |
|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------|
| **body**        | The body of the request. The value is undefined when there is no body. Only available on $.request.                                                                                                                                                                                                     | _`$.web.Body`_       |
| **contentType** | The content type of the entity.                                                                                                                                                                                                                                                                         | _`string`_           |
| **cookies**     | The cookies associated with the entity.                                                                                                                                                                                                                                                                 | _`$.web.TupelList`_  |
| **entities**    | The sub-entities of the entity.                                                                                                                                                                                                                                                                         | _`$.web.EntityList`_ |
| **headers**     | The headers of the entity.                                                                                                                                                                                                                                                                              | _`$.web.TupelList`_  |
| **language**    | Readonly. Language of the request in IETF (BCP 47) format. This property contains the language that is used for the request. Application code should rely on this property only. The value is a string in the format specified by the IETF (BCP 47) standard. Inherited From: $.web.WebRequest#language | _`string`_           |
| **method**      | The HTTP method of the incoming HTTP request. Inherited From: $.web.WebRequest#method                                                                                                                                                                                                                   | _`$.net.http`_       |
| **parameters**  | The parameters of the entity.                                                                                                                                                                                                                                                                           | _`$.web.TupelList`_  |
| **path**        | Readonly. The URL path specified in the request.                                                                                                                                                                                                                                                        | _`string`_           |
| **queryPath**   | The URL query path specified in the request. Inherited From: $.web.WebRequest#queryPath                                                                                                                                                                                                                 | _`string`_           |

## Functions


| Function                           | Description                                                                                                                                                                        | Returns  |
|------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|
| **setBody(body, index(optional))** | Sets the body of the entity; the method supports all elemental JavaScript types and ArrayBuffers as a body and numbers as an index. Throws an error if the parameters are invalid. | _`void`_ |
