---
title: API Reference


toc_footers:
  - <a href='https://cyaptor.com'>Cypator</a>

search: true

code_clipboard: true

meta:
  - name: description
    content: Documentation for the Kittn API
---

# Cypator Introduction
Bla Bla Bal

# FIX Taker API 

## Cypator FIX Introduction

This document details the Financial Information eXchange (FIX) protocol used by the Cyaptor Crypto trading ECN. Cypator uses the FIX protocol to stream prices and handle orders for FX Spot with a counterparty, referred to generically in this document as the “Client”.
To communicate with the ECN via the FIX protocol, there must be IP connectivity between Cyaptor and the Client, and the Client must initiate the connection. The Client must support the FIX 4.4 or FIX 4.2 protocol to communicate properly with the ECN.
The diagram below presents a high-level overview of the FIX-based FX electronic dealing
architecture.
<br />
Cypator  provides two FIX sessions for interaction with Clients. The first session is specifically for price communication and the second is for trading. Clients need to ensure that the appropriate session is used when messages are sent to the ECN.
<br />
This document defines the Capacitor FIX API for sending out market prices, receiving orders  and providing trading execution notifications via the Cypator FIX gateway

* The FIX gateway is accessible with an OpenVPN connection.
* There are two interfaces: FIX Market data and FIX Trading.
* FIX Trading and Fix Market data require a valid SenderCompId , login and password specified in the Logon message.
* The protocol is based on FIX protocol 4.4 (http://www.fixtradingcommunity.org/FIXimate/FIXimate3.0/). Refer to FIX 4.4 documentation if there is no tag information specified.
* FIX 4.2 is also supported, but is more limited and less recommended
* The FIX gateway supports subset of messages and tags listed in this document.
* Price is represented in natural value (e.g. 2500.01 for BTCEUR).

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

## Trading Workflow
The client will send a “New Order Single” message to place an order. Cypator will respond to the order with an Execution report message (35=8) of either Accept or Reject. If accepted, further execution report messages will indicate order fills until it is Done or Canceled.
* The Cypator trading interface supports the two types of orders:
  * Limit
  * Market
* The Cypator trading interface supports the following Time in force (TIF):
  * IOC - Immediate or Cancel
  * FOK - Fill or kill
  * Market - Best attempt at market price


<aside class="notice">
In the case where a Trade Acknowledgment and accompanying Trade Fill or Reject message are
not received within 5 seconds, from the time the trade request was sent, the Client MUST
contact Cyaptor using the following support email address: support@cypator.com. This support inbox is manned 24 hours a day 7 days a week, providing global support at all hours. Contact with Cypator should be made via an automated email alert from the Client’s trading system. However, in the event that the Client cannot support this, we would expect a manual email or a call to our
support desk.
</aside>

<aside class="warning">The Cypator trading interface currently does not support resting orders.</aside>


## Logon

> FIX 4.4 Client -> Cypator

```plaintext 
8=FIX.4.4|9=79|35=A|49=cc11|56=cs1|34=1|52=20221031-07:40:55|98=0|108=20|553=User1|554=123456|10=034|
```

> FIX 4.4 Cypator -> Client

```plaintext 
8=FIX.4.4|9=62|35=A|34=1|49=cs1|52=20221031-07:40:55.074|56=cc11|98=0|108=20|10=039|
```


> FIX 4.2 Client -> Cypator

```plaintext 
8=FIX.4.2|9=58|35=A|49=cc21|56=cs1|34=1|52=20221031-07:41:49|98=0|108=20|10=102|
```

> FIX 4.2 Cypator -> Client

```plaintext 
8=FIX.4.2|9=62|35=A|34=1|49=cs1|52=20221031-07:41:50.005|56=cc21|98=0|108=20|10=028|
```

This message is sent to initiate a FIX session and establishes the communication session, authenticates the connecting client, and initializes the message sequence number.

| Tag | Name            | Mandatory | Description                                                                        | 
|-----|-----------------|-----------|------------------------------------------------------------------------------------|
| 35  | MsgType         | Y         | A                                                                                  |
| 98  | EncryptMethod   | Y         | Y                                                                                  |
| 108 | HeartBtInt      | Y         | Y                                                                                  |
| 141 | ResetSeqNumFlag | N         | Indicated that both parties of the FIX Session should reset their sequence numbers |
| 553 | Username        | N         | Available only in FIX 4.4                                                          |
| 554 | Password        | N         | Available only in FIX 4.4                                                          |


## Heartbeat

This message is sent during periods of application inactivity to ensure connection validity. The receiving party should always respond with a heartbeat message.


| Tag | Name       | Mandatory | Description                                                                | 
|-----|------------|-----------|----------------------------------------------------------------------------|
| 35  | MsgType    | Y         | 0                                                                          |
| 98  | TestReqID  | N         | Required only when the heartbeat is in response to a Test Request Message  |


## Test Request

This message is used to verify connectivity and synchronize sequence numbers.  A test request should be responded to with a heartbeat from recipient

| Tag | Name       | Mandatory | Description                                      | 
|-----|------------|-----------|--------------------------------------------------|
| 35  | MsgType    | Y         | 1                                                |
| 98  | TestReqID  |           | Identifier to be returned in Heartbeat response  |


## Logout

This message signals the normal termination of the trading session. A session terminated without a Logout message will be considered an abnormal condition.

| Tag | Name     | Mandatory | Description | 
|-----|----------|-----------|-------------|
| 35  | MsgType  | Y         | 5           |
| 55  | Text     | N         |             |


## Market Data Request

> FIX 4.4 Client -> Cypator subscribe

```plaintext 

8=FIX.4.4|9=99|35=V|49=cc12|56=cs1|34=8|52=20221031-07:43:44|262=1|263=1|264=0|146=1|55=BTC/USD|267=2|269=0|269=1|10=095|
```

> FIX 4.2 Client -> Cypator subscribe

```plaintext 

8=FIX.4.2|9=99|35=V|49=cc22|56=cs1|34=3|52=20221031-08:34:43|262=1|263=1|264=0|146=1|55=BTC/USD|267=2|269=0|269=1|10=089|
```


> FIX 4.4 Client -> Cypator Unsubscribe

```plaintext 

8=FIX.4.4|9=99|35=V|49=cc12|56=cs1|34=7|52=20221031-08:35:17|262=1|263=2|264=0|146=1|55=BTC/USD|267=2|269=0|269=1|10=097|
```

> FIX 4.4 Client -> Cypator Unsubscribe

```plaintext 

8=FIX.4.2|9=99|35=V|49=cc22|56=cs1|34=5|52=20221031-08:34:55|262=1|263=2|264=0|146=1|55=BTC/USD|267=2|269=0|269=1|10=095|
```

Once the logon process is complete, Market Data Requests can be sent to the ECN. Cypator will respond immediately with either a Market Data Full Refresh (35=W) message or a Market Data Request Reject message (35=Y).
Only a single product can be requested in each request. The client will receive both bid and ask prices  in a single message ).
The ECN also supports layers (also known elsewhere as price bands or tiers). Quotes containing bid prices and quantities for all layers are always streamed in the same message, as is the case for quotes containing ask prices and quantities.

This message is used to subscribe/unsubscribe to market data rate information.


| Tag         | Name                    | Mandatory | Description                                                                                                                                                                                | 
|-------------|-------------------------|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 35          | MsgType                 | Y         | V                                                                                                                                                                                          |
| 262         | MDReqID                 | Y         | Unique Market Data Request ID.  This will be used in responses by Cypator or by the client to cancel a request. To unsubscribe from market data, the same ID must be sent with tag 263 = 2 |
| 263         | SubscriptionRequestType | Y         | 1 – Snapshot + Updates (Subscribe) <br />  2 – Disable Snapshot + Updates (Unsubscribe)                                                                                                    |
| 264         | MarketDepth             | Y         | 0 - Full Book  <br /> 1 - Top of the Book                                                                                                                                                  |
| 265         | MDUpdateType            | N         | 0 - Full refresh                                                                                                                                                                           | 
| 266         | AggregatedBook          | N         | Y - VWAP book <br /> N - Raw prices may or may not include the liquidity provider names.                                                                                                   |
| 267         | NoMDEntryTypes          | Y         | Number of MDEntryType fields being requested. 2 - bid and offer <br /> Note – please make sure to request in tag 269 both Bid and Offer. Request for a single side will be rejected!!!     |
| -><br />269 | MDEntryType             | Y         | Market Data entries types list: <br /> 0 - Bid <br /> 1 - Offer <br /> Repeated field: 269=0, 269=1                                                                                        |
| 146         | NoRelatedSym            | Y         | 1 ( we allow only a single asset per subscription)                                                                                                                                         |
| -><br />55  | Symbol                  | Y         | Asset - “BTC/USD”                                                                                                                                                                          |
| -><br />64  | FutSettDate             | N         | Value date YYYYMMDD. Currently unused, will be used once forward is supported.                                                                                                             |


## Market Data Request Reject

> FIX 4.4 Cypator -> Client

```plaintext 

8=FIX.4.4|9=85|35=Y|34=106|49=cs1|52=20221031-09:03:57.488|56=cc12|58=Duplicate MDReqID|262=1|281=1|10=094|
```

> FIX 4.2 Cypator -> Client

```plaintext 

8=FIX.4.2|9=84|35=Y|34=84|49=cs1|52=20221031-08:58:35.817|56=cc22|58=Duplicate MDReqID|262=1|281=1|10=050|
```


| Tag | Name            | Mandatory | Description                                                                                                                                        | 
|-----|-----------------|-----------|----------------------------------------------------------------------------------------------------------------------------------------------------|
| 35  | MsgType         | Y         | Y                                                                                                                                                  |
| 262 | MDReqID         | Y         | The  Unique ID of the received market data                                                                                                         |
| 281 | MDReqRejReason  | N         | Reason for rejection <br />  0 - Unknown Symbol <br /> 1 - Duplicate MDReqID <br /> 2 - Request not supported <br /> 3 - Insufficient Permissions  |


## Market Data Snapshot Full

> FIX 4.4 Cypator -> Client

```plaintext 

8=FIX.4.4|9=710|35=W|34=8|49=cs1|52=20221031-08:35:11.416|56=cc12|55=BTC/USD|262=1|268=20|269=0|270=202.15|271=0.00000001|269=0|270=202.16|271=0.00000002|269=0|270=202.17|271=0.00000003|269=0|270=202.18|271=0.00000004|269=0|270=202.19|271=0.00000005|269=0|270=202.2|271=0.00000006|269=0|270=202.21|271=0.00000007|269=0|270=202.22|271=0.00000008|269=0|270=202.23|271=0.00000009|269=0|270=202.24|271=0.0000001|269=1|270=202.35|271=0.00000001|269=1|270=202.36|271=0.00000002|269=1|270=202.37|271=0.00000003|269=1|270=202.38|271=0.00000004|269=1|270=202.39|271=0.00000005|269=1|270=202.4|271=0.00000006|269=1|270=202.41|271=0.00000007|269=1|270=202.42|271=0.00000008|269=1|270=202.43|271=0.00000009|269=1|270=202.44|271=0.0000001|10=073|
```

> FIX 4.2 Cypator -> Client

```plaintext 

8=FIX.4.2|9=710|35=W|34=3|49=cs1|52=20221031-08:34:44.164|56=cc22|55=BTC/USD|262=1|268=20|269=0|270=202.15|271=0.00000001|269=0|270=202.16|271=0.00000002|269=0|270=202.17|271=0.00000003|269=0|270=202.18|271=0.00000004|269=0|270=202.19|271=0.00000005|269=0|270=202.2|271=0.00000006|269=0|270=202.21|271=0.00000007|269=0|270=202.22|271=0.00000008|269=0|270=202.23|271=0.00000009|269=0|270=202.24|271=0.0000001|269=1|270=202.35|271=0.00000001|269=1|270=202.36|271=0.00000002|269=1|270=202.37|271=0.00000003|269=1|270=202.38|271=0.00000004|269=1|270=202.39|271=0.00000005|269=1|270=202.4|271=0.00000006|269=1|270=202.41|271=0.00000007|269=1|270=202.42|271=0.00000008|269=1|270=202.43|271=0.00000009|269=1|270=202.44|271=0.0000001|10=072|
```

| Tag         | Name               | Mandatory | Description                                                                | 
|-------------|--------------------|-----------|----------------------------------------------------------------------------|
| 35          | MsgType            | Y         | W                                                                          |
| 262         | MDReqID            | Y         | The  Unique ID of the received market data request                         |
| 55          | Symbol             | Y         | e.g “BTC/USD”                                                              |
| 64          | FutSettDate        | N         | Value date YYYYMMDD. Required for forward. Will be supported in the future |
| 268         | NoMDEntries        | Y         | No. of market data updates in the message                                  |
| -><br />269 | MDEntryType        | Y         | ‘0’ (Bid) <br /> ‘1’ (Offer)                                               |
| -><br />270 | MDEntryPx          | Y         | Price of entry                                                             |
| -><br />271 | MDEntrySize        | Y         | Quantity  of entry                                                         |
| -><br />282 | MDEntryOriginator  | N         | Liquidity provider name. In case of 266=N in Market Data Request Message.  |

In the event Cypator disables an instrument, it will send a message to the Taker side to clear its book for that instrument. This is done in order to prevent a reject or timeout for an order generated after the disable of the instrument.
The clear book message will be the standard 35=W message per instrument with 268=0.
An example of the message:

> FIX 4.4 Cypator -> Client

```plaintext 

8=FIX.4.4|9=100|35=W|34=268|49=cs1|52=20230619-09:45:58.883|56=cc12|55=ETH/USD|262=1|268=0|10=184| 
```

> FIX 4.2 Cypator -> Client

```plaintext 

8=FIX.4.2|9=100|35=W|34=268|49=cs1|52=20230619-09:45:58.883|56=cc12|55=ETH/USD|262=1|268=0|10=184| 
```





## TEEEEEEEEEEST

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

# FIX Maker API

# WebSocket Maker API

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
