# express-protocol-handler [![build status](https://badgen.net/travis/vladimyr/express-protocol-handler)](https://travis-ci.com/vladimyr/express-protocol-handler/) [![npm package version](https://badgen.net/npm/v/express-protocol-handler)](https://npm.im/express-protocol-handler) [![github license](https://badgen.net/github/license/vladimyr/express-protocol-handler)](https://github.com/vladimyr/express-protocol-handler/blob/master/LICENSE) [![js semistandard style](https://badgen.net/badge/code%20style/semistandard/pink)](https://github.com/Flet/semistandard)

> Resolve custom protocols using Express middleware.

## Instalation

    $ npm install express-protocol-handler

## Usage

```js
const app = require('express')();
const protocolHandler = require('express-protocol-handler')();

const port = 3000;
protocolHandler.protocol('s3://', (url, res) => res.redirect('https://example.com'));

app.get('/resolve', protocolHandler.middleware());
app.listen(port, () => console.log('listening on port: %i!', port));
```

    GET /resolve?url=s3://test HTTP/1.1
    Accept: */*
    Accept-Encoding: gzip, deflate
    Connection: keep-alive
    Host: localhost:3000
    User-Agent: HTTPie/0.9.9


    HTTP/1.1 302 Found
    Connection: keep-alive
    Content-Length: 41
    Content-Type: text/plain; charset=utf-8
    Date: Thu, 20 Dec 2018 13:25:26 GMT
    Location: https://example.com
    Vary: Accept
    X-Powered-By: Express

    Found. Redirecting to https://example.com 

## API

<!-- Generated by documentation.js. Update this documentation by updating the source code. -->

#### Table of Contents

-   [ProtocolHandler](#protocolhandler)
    -   [Parameters](#parameters)
    -   [protocol](#protocol)
        -   [Parameters](#parameters-1)
        -   [Examples](#examples)
    -   [protocols](#protocols)
        -   [Properties](#properties)
        -   [Examples](#examples-1)
    -   [middleware](#middleware)
        -   [Examples](#examples-2)
-   [module.exports](#moduleexports)
    -   [Parameters](#parameters-2)
    -   [Examples](#examples-3)
-   [ProtocolHandlerOptions](#protocolhandleroptions)
    -   [Properties](#properties-1)
-   [ProtocolCallback](#protocolcallback)
    -   [Parameters](#parameters-3)
    -   [Examples](#examples-4)
-   [IRequest](#irequest)
-   [IResponse](#iresponse)

### ProtocolHandler

Create protocol handler

#### Parameters

-   `param` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** name of query param containing target url (optional, default `'url'`)
-   `options` **[ProtocolHandlerOptions](#protocolhandleroptions)** protocol handler options (optional, default `{}`)
    -   `options.blacklist`   (optional, default `[]`)

#### protocol

Register protocol handler

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

Returns **[ProtocolHandler](#protocolhandler)** instance to allow chaining

#### protocols

##### Properties

-   `protocols` **[Set](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Set)&lt;[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)>** registered protocols

##### Examples

```javascript
// check if protocol is registered
const handler = new ProtocolHandler();
handler.register('s3://', resolve);
console.log(handler.protocols.has('s3:'));
//=> true
```

#### middleware

Returns [Express](https://expressjs.com) middleware

##### Examples

```javascript
// create handler
const handler = new ProtocolHandler();
handler.protocol('s3://', resolve);
// attach to express app
app.use(handler.middleware());
```

Returns **function ([IRequest](#irequest), [IResponse](#iresponse))** Express middleware

### module.exports

Create new ProtocolHandler instance

#### Parameters

-   `param` **[String](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/String)** name of query param containing target url (optional, default `'url'`)
-   `options` **[ProtocolHandlerOptions](#protocolhandleroptions)** protocol handler options (optional, default `{}`)

#### Examples

```javascript
const handler = require('express-protocol-handler')('query');
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
-   `res` **[IRequest](#irequest)** server response object

#### Examples

```javascript
// Resolve gdrive urls
const { fetchInfo } = require('gdrive-file-info');

async function resolve(url, res) {
  const itemId = new URL(url).pathname;
  const fileInfo = await fetchInfo(itemId);
  res.redirect(fileInfo.downloadUrl);
}
```

### IRequest

-   **See: <https://expressjs.com/en/4x/api.html#req>**

Express server request

### IResponse

-   **See: <https://expressjs.com/en/4x/api.html#res>**

Express server response
