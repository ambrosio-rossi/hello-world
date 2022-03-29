---
title: $.util
---

$.util
===

## Overview

- Definition: [https://github.com/SAP/xsk/issues/16](https://github.com/SAP/xsk/issues/16)
- Module: [util/util.js](https://github.com/SAP/xsk/blob/main/modules/api/api-xsjs/src/main/resources/META-INF/dirigible/xsk/util/util.js)
- Status: `alpha`

## Sample Usage

```javascript
const util = $.util;

let randomID = util.createUuid();

// Uint8Array
let arrayBuffer = [
    84, 104, 105, 115, 32, 105,
    115, 32, 97, 32, 85, 105,
];

let convertedBuff = util.stringify(arrayBuffer);

let result = `randomID is : ${randomID} `;
result += `\nconvertedBuff is: ${arrayBuffer} `;

$.response.setBody(result);
```

## Classes

| Classes                                        | Description                                               |
|------------------------------------------------|-----------------------------------------------------------|
| **[SAXParser](../util.SAXParser/)**                     | Class for parsing XML. It is based on expat. |
| **[Zip](../util.Zip/)** | Class for manipulation of zip archives. |

## Functions

| Members             | Description                                            | Returns     |
|---------------------|--------------------------------------------------------|-------------|
| **createUuid()**    | Returns a unique UUID.                                 | _`string`_  |
| **stringify(data)** | Recieves UintArray and return converted value.         | _`string`_  |
