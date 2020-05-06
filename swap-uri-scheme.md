# URI Scheme Specificaton

#### Version: 0.1
#### Date published: 5-7-2020



## Purpose

The purpose of this URI scheme is to enable users to easily initiate Signal, Watch, and Pay (SWaP) Protocol transactions by simply clicking links on webpages or scanning QR Codes.



## Specification

### General rules for handling (important!)

Clients MUST NOT act on URIs without getting the user's authorization.
They SHOULD require the user to manually approve each transaction individually, though in some cases they MAY allow the user to automatically make this decision.

### Operating system integration

Graphical SWaP clients SHOULD register themselves as the handler for the "swap:" URI scheme by default, if no other handler is already registered. If there is already a registered handler, they MAY prompt the user to change it once when they first run the client.

### General Format

SWaP URIs follow the general format for URIs as set forth in RFC 3986. The path component consists of a SWaP message transaction hash, and the query component provides additional transaction options.

Elements of the query component must not contain characters outside the valid range.

### ABNF grammar


| Placeholder  | Format                                                       |
| ------------ | ------------------------------------------------------------ |
| slpurn       | = "swap:" txhash [ "?" params ]                              |
| txhash       | = *hex                                                       |
| params       | = param [ "&" params ]                                       |
| param        | = [ d ]                                                      |
| d            | = "d=" *hex                                                  |




### Implementation Requirements

1. Bitcoin Cash is the blockchain associated with the "swap" URI scheme.
2. The character set requirements for the `*hex` format are those of a hexidecimal number expressed as a string
3. The `d` parameter is a string representing the entire OP_RETURN scriptPubKey (including the OP_RETURN opcode itself) for the transaction at `txhash`. Having this data present, particularly in the case of Threshold Crowdfund (Type 3) transactions, allows offline wallets to construct valid Payment messages.

### Examples

This section is non-normative and does not cover all possible syntax.
Please see the ABNF grammar above for the normative syntax.

* **Just the transaction hash of a SWaP Signal transaction:**
  * `swap:b03883ca0b106ea5e7113d6cbe46b9ec37ac6ba437214283de2d9cf2fbdc997f`

* **Transaction hash and OP_RETURN data of a SWaP Signal transaction:**
    * `swap:b03883ca0b106ea5e7113d6cbe46b9ec37ac6ba437214283de2d9cf2fbdc997f?d=6a045357500001010101204de69e374a8ed21cbddd47f2338cc0f479dc58daa2bbe11cd604ca488eca0ddf0453454c4c02025801002090dfb75fef5f07e384df4703b853a2741b8e6f3ef31ef8e5187a17fb107547f801010100`



## Reference Implementations

### Clients
None currently

### Libraries
[swap-bch-js](https://github.com/vinarmani/swap-bch-js)
