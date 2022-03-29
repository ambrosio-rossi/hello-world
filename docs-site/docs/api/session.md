---
title: $.session
---

$.session
===

`$.session` represents the Session with its fields and methods.

## Overview

- Definition: [https://github.com/SAP/xsk/issues/11](https://github.com/SAP/xsk/issues/11)
- Module: [session/session.js](https://github.com/SAP/xsk/tree/main/modules/api/api-xsjs/src/main/resources/META-INF/dirigible/xsk/session/session.js)
- Status: `alpha`

## Sample Usage

```javascript
var session = $.session;

var username = session.getUsername()
var timeout = session.getTimeout()
var token = session.getSecurityToken()
var authType = session.authType

// Check the language of the session
response.println("Session language: " +session.language)

// Check if a particular user has the "Administrator" role
if (username === "dirigible" && session.hasAppPrivilege("Administrator")) {
    // Check a specific system privilege for that user
    if (session.hasSystemPrivilege("Dirigible")) {
        // Perform some operation with his session's information
        $.response.setBody("Username: " + username + " with session authentication type: " + authType + " token: " + token + " and timeout " + timeout);
    }
} else {
    // Assert that the user is a Developer in all other cases
    try {
        session.assertAppPrivilege("Developer");
        // Check the authentification type
        if (authType === "BASIC") {
            // Use the information from the current session
            $.response.setBody("Username: " + username + " with session authentication type: " + authType + " token: " + token + " and timeout " + timeout);
        }
    } catch(error) {
        //Display the missing role that was being asserted
        $.response.setBody("User does not have the role: " + error.privilege);
    }
}

// After all calls are complete, check the invocation count of the current session
$.response.setBody("Invocation count: " + session.getInvocationCount());
```

## Properties


| Name              | Description                                                                                           | Type                                    |
|-------------------|-------------------------------------------------------------------------------------------------------|-----------------------------------------|
| **authType**      | Authentication method that was used for the current session.                                          | _`string/null`_                         |
| **language**      | Language of the session in IETF (BCP 47) format.                                                      | _`string`_                              |
| **samlAttribute** | Provides the detailed content of the AttributeStatement tag which can be part of a SAML assertion.    |_`Array.<$.Session~SamlAttributeObject>`_|
| **samlUserInfo**  | Provides the materialized content of the AttributeStatement tag which can be part of a SAML assertion.|_`object`_|   

## Functions


| Function                                | Description                                                                  | Returns     |
|-----------------------------------------|------------------------------------------------------------------------------|-------------|
| **assertAppPrivilege(privilegeName)**   | Asserts that the logged-on user has a specified application privilege.       | _`-`_       |
| **assertSystemPrivilege(privilegeName)**| Asserts that the logged-on user has a specified system privilege.            | _`-`_       |
| **getInvocationCount()**                | Returns the number of requests sent to the current session.                  | _`Number`_  |
| **getSecurityToken()**                  | Returns unique session-specific token that could be used for XSRF prevention.| _`string`_  |
| **getTimeout()**                        | The timeout of the XS session in seconds.                                    | _`integer`_ |
| **getUsername()**                       | Returns the username of the logged-on database user.                         | _`string`_  |
| **hasAppPrivilege(privilegeName)**      | Checks whether the logged-on user has a specified application privilege.     | _`boolean`_ |
| **hasSystemPrivilege(privilegeName)**   | Checks whether the logged-on user has a specified system privilege.          | _`boolean`_ |
