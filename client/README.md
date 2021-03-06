# Ralphi Client
Client for ralphi pure Node.js rate limiting server

[![npm version](https://img.shields.io/npm/v/ralphi-client.svg)](https://www.npmjs.com/package/ralphi-client)
[![Build Status](https://travis-ci.org/yonjah/ralphi.svg?branch=master)](https://travis-ci.org/yonjah/ralphi)
[![codecov](https://codecov.io/gh/yonjah/ralphi/branch/master/graph/badge.svg)](https://codecov.io/gh/yonjah/ralphi)
[![Known Vulnerabilities](https://snyk.io/test/npm/ralphi-client/badge.svg)](https://snyk.io/test/npm/ralphi-client)
[![License](https://img.shields.io/npm/l/ralphi.svg?maxAge=2592000?style=plastic)](https://github.com/yonjah/ralphi/blob/master/LICENSE)

`Ralphi` is a simple rate limiting server intended to prevent bruteforce attacks on logins and other sensitive assets.

For more info about Ralphi other components see [ralphi](https://ralphi.js.org/)

## Client Installation

```bash
$ npm install -s ralphi-client
```

## Usage
<!-- eslint-disable strict,no-unused-vars -->

```js
const host = 'localhost';
const port = 8910;
const RalphiClient = require('ralphi-client');
const client = new RalphiClient({port, host});

async function handler (req, res) { //in your handler code
    const limit = await client.take('login', req.ip);
    if (limit.conformant) {
        //allow access
        return `Request was done. You have ${limit.remaining} more requests until ${new Date(limit.ttl * 1000)}`;
    } else {
        //reject access
        throw new Error(`You have made too many requests. You can send ${limit.size} requests after ${new Date(limit.ttl * 1000)}`);
    }
}
```

## Config options
- _host String (localhost)_ - host/ip  where Ralphi server is running on
- _port Integer (8910)_ - port Ralphi server listens on
- _timeout Integer (5000)_ - timeout for requests in miliseconds
- _keepAlive Boolean (false)_ - if true http keep alive will be enabled when connecting to ralphi server (should have performance benefit)

## API
Ralphi Client is using a Promise API all methods return Promises
- _take(bucket String, key String)`_ - remove one request token for key associated with bucket.
    Returns an object containing the following properties -
    - _conformant Boolean_ - true if record had remaining request tokens and can be accepted. false if request did not have any tokens left and should not be accepted.
    - _remaining Integer_  - the amount of requests that can still be made in the current time frame
    - _ttl Integer_ - UNIX timestamp indicating when more requests will be available
    - _size Integer_ - Total number of request tokens available in each time frame
- _give(bucket String, key String)`_ - adds one request token for key associated with bucket.
    Returns an object containing the following properties -
    - _conformant Boolean_ - true if after increase record has remaining request tokens. false if request still does not have any tokens left.
    - _remaining Integer_  - the amount of requests that can still be made in the current time frame
    - _ttl Integer_ - UNIX timestamp indicating when more requests will be available
    - _size Integer_ - Total number of request tokens available in each time frame
    **Please** note that `give` is meant to be used to give back a token that wasn't suppose to be taken (like if login was successful). even after giving a token you can't be sure next request will be conformant cause other requests may have overdrawn the bucket if you want to manually force bucket to have more tokens you should use `reset`
- _query(bucket String, key String)`_ - similar to take but will only query rate limit stats without removing any tokens.
    Returns an object containing the following properties -
    - _conformant Boolean_ - true if record had remaining request tokens and can be accepted. false if request did not have any tokens left and should not be accepted.
    - _remaining Integer_  - the amount of requests that can still be made in the current time frame
    - _ttl Integer_ - UNIX timestamp indicating when more requests will be available
    - _size Integer_ - Total number of request tokens available in each time frame
- _reset(bucket String, key String)_ - remove a single record from the bucket.
    Removing a record means next request with the same key will get a fresh record with maximum available requests. 
    returns `true` if record existed and `false` if it wasn't
- _clean()_ - Remove all expired records from the bucket.