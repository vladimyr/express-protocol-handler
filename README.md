# custom-protocol-handler
[![build status](https://badgen.net/travis/vladimyr/custom-protocol-handler)](https://travis-ci.com/vladimyr/custom-protocol-handler/) [![install size](https://badgen.net/packagephobia/install/custom-protocol-handler)](https://packagephobia.now.sh/result?p=custom-protocol-handler) [![npm package version](https://badgen.net/npm/v/custom-protocol-handler)](https://npm.im/custom-protocol-handler) [![github license](https://badgen.net/github/license/vladimyr/custom-protocol-handler)](https://github.com/vladimyr/custom-protocol-handler/blob/master/LICENSE) [![js semistandard style](https://badgen.net/badge/code%20style/semistandard/pink)](https://github.com/Flet/semistandard)

> Resolve custom protocols using registered handlers.

## Instalation

    $ npm install custom-protocol-handler

## Usage

This module can be used either in standalone mode or as [Express](https://expressjs.com) middleware.

```js
const protocolHandler = require('custom-protocol-handler')();
protocolHandler.protocol('s3://', url => 'https://example.com');

// Standalone usage
protocolHandler.resolve('s3://test').then(url => console.log(url));
//=> https://example.com

// Using as Express middleware
const port = 3000;
const app = require('express')();
app.get('/resolve', protocolHandler.middleware());
app.listen(port, () => console.log('listening on port: %i!', port));
```

<details>
  <summary>Click to open HTTP log</summary>

    $ ./example.sh

    # resolve registered protocol: `s3:`

    GET /resolve?url=s3://test HTTP/1.1
    Accept: */*
    Accept-Encoding: gzip, deflate
    Connection: keep-alive
    Host: localhost:3000
    User-Agent: HTTPie/1.0.2



    HTTP/1.1 302 Found
    Connection: keep-alive
    Content-Length: 41
    Content-Type: text/plain; charset=utf-8
    Date: Sat, 12 Jan 2019 16:55:26 GMT
    Location: https://example.com
    Vary: Accept
    X-Powered-By: Express

    Found. Redirecting to https://example.com

    # resolve standard protocol: `https:`

    GET /resolve?url=https://google.com HTTP/1.1
    Accept: */*
    Accept-Encoding: gzip, deflate
    Connection: keep-alive
    Host: localhost:3000
    User-Agent: HTTPie/1.0.2



    HTTP/1.1 302 Found
    Connection: keep-alive
    Content-Length: 40
    Content-Type: text/plain; charset=utf-8
    Date: Sat, 12 Jan 2019 16:55:26 GMT
    Location: https://google.com
    Vary: Accept
    X-Powered-By: Express

    Found. Redirecting to https://google.com

    # resolve unknown protocol: `gdrive:`

    GET /resolve?url=gdrive://test HTTP/1.1
    Accept: */*
    Accept-Encoding: gzip, deflate
    Connection: keep-alive
    Host: localhost:3000
    User-Agent: HTTPie/1.0.2



    HTTP/1.1 400 Bad Request
    Connection: keep-alive
    Content-Length: 83
    Content-Type: application/json; charset=utf-8
    Date: Sat, 12 Jan 2019 16:55:27 GMT
    ETag: W/"53-Z2BGf/llR30GzNCkJLqNslE8IJ4"
    X-Powered-By: Express

    {
        "error": {
            "code": 1,
            "message": "Unknown protocol: `gdrive:`",
            "name": "ProtocolError"
        }
    }
</details>

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [ProtocolError](#protocolerror)
    -   [Parameters](#parameters)
-   [ProtocolError.code](#protocolerrorcode)
    -   [Properties](#properties)
-   [ProtocolHandler](#protocolhandler)
    -   [Parameters](#parameters-1)
    -   [protocol](#protocol)
        -   [Parameters](#parameters-2)
        -   [Examples](#examples)
    -   [protocols](#protocols)
        -   [Properties](#properties-1)
        -   [Examples](#examples-1)
    -   [resolve](#resolve)
        -   [Parameters](#parameters-3)
        -   [Examples](#examples-2)
    -   [middleware](#middleware)
        -   [Parameters](#parameters-4)
        -   [Examples](#examples-3)
-   [module.exports](#moduleexports)
    -   [Parameters](#parameters-5)
    -   [Examples](#examples-4)
-   [ProtocolHandlerOptions](#protocolhandleroptions)
    -   [Properties](#properties-2)
-   [ProtocolCallback](#protocolcallback)
    -   [Parameters](#parameters-6)
    -   [Examples](#examples-5)
-   [ProtocolErrorCallback](#protocolerrorcallback)
    -   [Parameters](#parameters-7)
    -   [Examples](#examples-6)

### ProtocolError

**Extends Error**

Custom error indicating invalid, unknown or blacklisted protocol

#### Parameters

-   `code` **[ProtocolError.code](#protocolerrorcode)** Error code
-   `message` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** Error message

### ProtocolError.code

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

#### Properties

-   `ERR_PROTOCOL_INVALID` **[Number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** 
-   `ERR_PROTOCOL_UNKNOWN` **[Number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** 
-   `ERR_PROTOCOL_BLACKLISTED` **[Number](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Number)** 

### ProtocolHandler

Create protocol handler

#### Parameters

-   `options` **[ProtocolHandlerOptions](#protocolhandleroptions)** protocol handler options (optional, default `{}`)
    -   `options.blacklist`   (optional, default `[]`)

#### protocol

Registers protocol handler

##### Parameters

-   `scheme` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** protocol scheme
-   `handler` **[ProtocolCallback](#protocolcallback)** protocol handler

##### Examples

```javascript
// register multiple handlers
const handler = new ProtocolHandler();
handler
  .protocol('s3://', resolve)
  .protocol('gdrive://', resolve);
```

-   Throws **[ProtocolError](#protocolerror)** throws if protocol scheme is invalid or blacklisted

Returns **[ProtocolHandler](#protocolhandler)** instance to allow chaining

#### protocols

##### Properties

-   `protocols` **[Set](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Set)&lt;[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)>** registered protocols

##### Examples

```javascript
// check if protocol is registered
const handler = new ProtocolHandler();
handler.protocol('s3://', resolve);
console.log(handler.protocols.has('s3:'));
//=> true
```

#### resolve

Asynchronously resolves url with registered protocol handler

##### Parameters

-   `url` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** target url

##### Examples

```javascript
// create handler
const handler = new ProtocolHandler();
handler.protocol('s3://', url => 'https://example.com');
// resolve url
handler.resolve('s3://test').then(url => console.log(url));
//=> https://example.com
handler.resolve('file:///local/file.txt').then(url => console.log(url));
//=> file:///local/file.txt
handler.resolve('dummy://unknown/protocol');
//=> throws ProtocolError
```

-   Throws **[ProtocolError](#protocolerror)** throws if url contains invalid or unknown protocol

Returns **[Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)&lt;[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)>** resolved url, redirect location

#### middleware

Returns [Express](https://expressjs.com) middleware

##### Parameters

-   `param` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** name of query param containing target url (optional, default `'url'`)
-   `cb` **[ProtocolErrorCallback](#protocolerrorcallback)?** custom error handling callback

##### Examples

```javascript
// create handler
const handler = new ProtocolHandler();
handler.protocol('s3://', resolve);
// attach to express app
app.use(handler.middleware());
```

### module.exports

Create new ProtocolHandler instance

#### Parameters

-   `options` **[ProtocolHandlerOptions](#protocolhandleroptions)** protocol handler options (optional, default `{}`)

#### Examples

```javascript
const handler = require('custom-protocol-handler')();
```

Returns **[ProtocolHandler](#protocolhandler)** instance

### ProtocolHandlerOptions

Type: [Object](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object)

#### Properties

-   `blacklist` **[Array](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array)&lt;[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)>?** array of blacklisted schemes

### ProtocolCallback

Resolver function for specific protocol

Type: [Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)

#### Parameters

-   `url` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** target url

#### Examples

```javascript
// Resolve gdrive urls
const { fetchInfo } = require('gdrive-file-info');

async function resolve(url) {
  const itemId = new URL(url).pathname;
  const fileInfo = await fetchInfo(itemId);
  return fileInfo.downloadUrl;
}
```

Returns **([String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String) \| [Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)&lt;[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)>)** resolved url _redirect location_

### ProtocolErrorCallback

Custom error calback for Express middleware

Type: [Function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/function)

#### Parameters

-   `err` **[ProtocolError](#protocolerror)** protocol error
-   `url` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** target url

#### Examples

```javascript
const handler = new ProtocolHandler();
handler.protocol('s3://', resolve);
// Redirect ONLY registered protocols
app.use(handler.middleware('url', (err, url, res) => {
  if (!err) res.redirect(url);
  return res.sendStatus(400);
}));
```
