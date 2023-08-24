---
title: API Reference


toc_footers:
  - <a href='https://cyaptor.com'>Cypator</a>

includes:
  - errors

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Kittn API
---

# Cypator Introduction

Welcome to the Cypator API! You can use our API to access API endpoints.

# Cypator code example

> To authorize, use this code:

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
```

```shell
# With shell, you can just pass the correct header with each request
curl "api_endpoint_here" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require("kittn");

let api = kittn.authorize("meowmeowmeow");
```

> Make sure to replace `meowmeowmeow` with your API key.

Kittn uses API keys to allow access to the API. You can register a new Kittn API key at our [developer portal](http://example.com/developers).

Kittn expects for the API key to be included in all API requests to the server in a header that looks like the following:

`Authorization: meowmeowmeow`

<aside class="notice">
You must replace <code>meowmeowmeow</code> with your personal API key.
</aside>

# FIX Taker API 

## Connectivity

* The following are the ways a client can connection to the API:
  * Internet over VPN
* VPN connectivity is routed to the nearest geographical instance - New York, London or Singapore.
* Access to the platform has to go through SSL encrypted TCP connection over the Internet. The client should provide a certificate signing request (CSR) in the format of .crt/.cer, which will be added to Cypator certification file.
* In case the client does not have a CSR file, please refer to the links below for additional instructions:
  * [CSR information](https://www.globalsign.com/en/blog/what-is-a-certificate-signing-request-csr)  
  * How to generate .crt file using openSSL (see section Create Certificate Authority): https://devopscube.com/create-self-signed-certificates-openssl/
  * How to generate .cer file using keytool: https://docs.oracle.com/cd/E19798-01/821-1841/gjrgy/

## Session Time
* The session will be up the full week with a 3-minute scheduled restart on a weekly basis every Sunday. Exact time for restart will be scheduled with the client.
  * StartDay=Sunday
  * EndDay=Sunday
  * StartTime=07:03:00
  * EndTime=07:00:00

## Messages, Products and FIX Support
* The API supports the following:
  * FIX version 4.4 (recommended) and 4.2
  * Single session or Dual session
  * Product type - Spot trading only
  * Assets - any crypto coin and FIAT currency is supported. Limitations on what Assets are allowed are defined in the application and business agreement
  * For fix 4.2 and 4.4 dual sessions - if we receive a message on the wrong session- it will be rejected. In addition, Fix4.2 doesn’t support all message types.


| Message type                          | Fix Version Supported | Market Session   | Trading Session | 
| ------------------------------------- | --------------------- | ---------------- |-----------------|
| Heartbeat <0>                         | 4.2/4.4               | Y                | Y               |
| Test Request <1>                      | 4.2/4.4               | Y                | Y               |
| Resend Request <2>                    | 4.2/4.4               | Y                | Y               |
| Reject <3>                            | 4.2/4.4               | Y                | Y               |
| Sequence Reset <4>                    | 4.2/4.4               | Y                | Y               |
| Logout <5>                            | 4.2/4.4               | Y                | Y               |
| Logon <A>                             | 4.2/4.4               | Y                | Y               |
| Market Data Request <V>               | 4.2/4.4               | Y                | N               |
| Market Data Request Reject <Y>        | 4.2/4.4               | Y                | N               |
| Market Data-Snapshot/Full Refresh <W> | 4.2/4.4               | Y                | N               |
| New Order Single <D>                  | 4.2/4.4               | N                | Y               |
| Execution Report <8>                  | 4.2/4.4               | N                | Y               |
| Trade Capture Report <AE>             | 4.4                   | N                | Y               |
| Trade Capture Report Request <AD>     | 4.4                   | N                | Y               |
| Trade Capture Report Request Ack <AQ> | 4.4                   | N                | Y               |

## Duplicate check
There is always a possibility of duplicate trade being sent out, for example after a network disconnect. The client is expected to be able to identify duplicate trades and reject them, by using the tag 37 - OrderID

## Header and Trailer 
The following define the FIX messages standard header and trailer

Header

| Tag | Name         | Mandatory    | 
|-----|--------------|--------------|
| 8   | BeginString  | Y            |
| 9   | BodyLength   | Y            |
| 34  | MsgSeqNum    | Y            |
| 35  | MsgType      | Y            |
| 49  | SenderCompID | Y            |
| 50  | SenderSubID  | Y            |
| 52  | SendingTime  | Y            |
| 56  | TargetCompID  | Y            |

Trailer

| Tag | Name      | Mandatory    | 
|-----|-----------|--------------|
| 10  | CheckSum  | Y            |


## Session Messages
The initialization sequence consists of two messages sent between the Client and Cypator during
startup. The purpose of this is simply to initialize the session. If the logon sequence causes resending to take place (as described in the FIX Protocol Specification v. 4.2+) then quotes will not be replayed – instead they will be replaced with a “Sequence Reset – Gap Fill” message.
[Cypator](Cypator.com) currently supports inbound connections only, meaning that Clients are responsible for
logging into the electronic dealing platform


## Logon

> FIX 4.4

```plaintext 
Client -> Cypator
8=FIX.4.4|9=79|35=A|49=cc11|56=cs1|34=1|52=20221031-07:40:55|98=0|108=20|553=User1|554=123456|10=034|

Cypator -> Client
8=FIX.4.4|9=62|35=A|34=1|49=cs1|52=20221031-07:40:55.074|56=cc11|98=0|108=20|10=039|
```

> FIX 4.2

```plaintext 
Client -> Cypator
8=FIX.4.2|9=58|35=A|49=cc21|56=cs1|34=1|52=20221031-07:41:49|98=0|108=20|10=102|

Cypator -> Client
8=FIX.4.2|9=62|35=A|34=1|49=cs1|52=20221031-07:41:50.005|56=cc21|98=0|108=20|10=028|
```

This message is sent to initiate a FIX session and establishes the communication session, authenticates the connecting client, and initializes the message sequence number.

| Tag | Name            | Mandatory | Description                                                                        | 
|-----|-----------------|-----------|------------------------------------------------------------------------------------|
| 35  | MsgType         | Y         | Y                                                                                  |
| 98  | EncryptMethod   | Y         | Y                                                                                  |
| 108 | HeartBtInt      | Y         | Y                                                                                  |
| 141 | ResetSeqNumFlag | N         | Indicated that both parties of the FIX Session should reset their sequence numbers |
| 553 | Username        | N         | Available only in FIX 4.4                                                          |
| 554 | Password        | N         | Available only in FIX 4.4                                                          |


## Logon


```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get()
```

```shell
curl "http://example.com/api/kittens" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require("kittn");

let api = kittn.authorize("meowmeowmeow");
let kittens = api.kittens.get();
```

> The above command returns JSON structured like this:

```json
[
  {
    "id": 1,
    "name": "Fluffums",
    "breed": "calico",
    "fluffiness": 6,
    "cuteness": 7
  },
  {
    "id": 2,
    "name": "Max",
    "breed": "unknown",
    "fluffiness": 5,
    "cuteness": 10
  }
]
```

This endpoint retrieves all kittens.

### HTTP Request

`GET http://example.com/api/kittens`

### Query Parameters

| Parameter    | Default | Description                                                                      |
| ------------ | ------- | -------------------------------------------------------------------------------- |
| include_cats | false   | If set to true, the result will also include cats.                               |
| available    | true    | If set to false, the result will include kittens that have already been adopted. |

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## FIX Maker API

## WebSocket Maker API

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.get(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.get(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require("kittn");

let api = kittn.authorize("meowmeowmeow");
let max = api.kittens.get(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "name": "Max",
  "breed": "unknown",
  "fluffiness": 5,
  "cuteness": 10
}
```

This endpoint retrieves a specific kitten.

<aside class="warning">Inside HTML code blocks like this one, you can't use Markdown, so use <code>&lt;code&gt;</code> blocks to denote code.</aside>

### HTTP Request

`GET http://example.com/kittens/<ID>`

### URL Parameters

| Parameter | Description                      |
| --------- | -------------------------------- |
| ID        | The ID of the kitten to retrieve |

## Delete a Specific Kitten

```ruby
require 'kittn'

api = Kittn::APIClient.authorize!('meowmeowmeow')
api.kittens.delete(2)
```

```python
import kittn

api = kittn.authorize('meowmeowmeow')
api.kittens.delete(2)
```

```shell
curl "http://example.com/api/kittens/2" \
  -X DELETE \
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require("kittn");

let api = kittn.authorize("meowmeowmeow");
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted": ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

| Parameter | Description                    |
| --------- | ------------------------------ |
| ID        | The ID of the kitten to delete |
