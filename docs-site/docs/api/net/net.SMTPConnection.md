---
title: $.net.SMTPConnection
---

$.net.SMTPConnection
===

`$.net.SMTPConnection` class for sending `$.net.Mail` objects via SMTP connection.

## Overview

- Definition: [https://github.com/SAP/xsk/issues/19](https://github.com/SAP/xsk/issues/19)
- Module: [net/net.js](https://github.com/SAP/xsk/tree/main/modules/api/api-xsjs/src/main/resources/META-INF/dirigible/xsk/net/net.js)
- Status: `alpha`

## Sample Usage

!!! Note
	Requires a running mail server. If `mailConfig` is not set the api defaults to a local mail server.
	For more information please take a look [here](https://blogs.sap.com/2020/02/05/sending-e-mails-with-the-eclipse-dirigible-mail-api/).

```javascript
var net = $.net

// Create email from JS Object.
var mail = new net.Mail({
   sender: {address: "sender@sap.com"},
   to: [{ name: "John Doe", address: "john.doe@sap.com"}, {name: "Jane Doe", address: "jane.doe@sap.com"}],
   cc: [{address: "cc1@sap.com"}, {address: "cc2@sap.com"}],
   bcc: [{ name: "Jonnie Doe", address: "jonnie.doe@sap.com"}],
   subject: "subject",
   subjectEncoding: "UTF-8",
   parts: [ new net.Mail.Part({
       type: net.Mail.Part.TYPE_TEXT,
       text: "The body of the mail.",
       contentType: "text/plain",
       encoding: "UTF-8",
   })]
});

// Set mail server configurations.
let mailConfig = {
    "mail.user": "<your-user>",
    "mail.password": "<your-password>",
    "mail.transport.protocol": "smtps",
    "mail.smtps.host": "<your-mail-provider-host>",
    "mail.smtps.port": "465",
    "mail.smtps.auth": "true"
};

var smtp = new net.SMTPConnection(mailConfig);
let returnValue = smtp.send(mail);

$.response.setBody(JSON.stringify(returnValue));
```
## Constructors

```javascript
new $.net.SMTPConnection(mailConfig)
```

## Parameters

| Parameter Name | Description                                                                                                                                                     | Required     | Type       |
|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|------------|
| **mailConfig** | JS object containing mail server configuration properties. | _`optional`_ | _`object`_ |

## mailConfig Properties

Property     | Description | Type
------------ | ----------- | --------
**mail.user**   | The mailbox user | *string*
**mail.password**   | The mailbox password | *string*
**mail.transport.protocol**   | (optional) The mail transport protocol, default is *smtps* | *string*
**mail.smtps.host**   | The mail SMPTPS host | *string*
**mail.smtps.port**   | The mail SMPTPS port | *number as string*
**mail.smtps.auth**   | Enable/Disable mail SMPTPS authentication | *boolean as string*
**mail.smtp.host**   | The mail SMPTP host | *string*
**mail.smtp.port**   | The mail SMPTP port | *number as string*
**mail.smtp.auth**   | Enable/Disable mail SMPTP authentication | *boolean as string*

Addition mail client options can be found here:
- [SMTP/SMTPS](https://javaee.github.io/javamail/docs/api/com/sun/mail/smtp/package-summary.html)
- [IMAP](https://javaee.github.io/javamail/docs/api/com/sun/mail/imap/package-summary.html)
- [POP3](https://javaee.github.io/javamail/docs/api/com/sun/mail/pop3/package-summary.html)

## Functions


| Function       | Description                                                                            | Returns     |
|----------------|----------------------------------------------------------------------------------------|-------------|
| **close()**    | Mocked. The SMTP Connection is now automatically closed after calling the send method. | _`void`_    |
| **isClosed()** | Mocked. The SMTP Connection is always closed.                                          | _`boolean`_ |
| **send(Mail)** | Accepts and sends the net.Mail class.                                                  | _`void`_    |
