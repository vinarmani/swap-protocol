# JSON Oracle Standard
### Specification version: 0.1
### Date published: May 27, 2020

## Author
Vin Armani

# Purpose
We propose a unified standard for on-chain oracle messages for Bitcoin Cash. These messages are constructed as JavaScript Object Notation (JSON) strings and are stored on the blockchain using the [Bitcoin Files Protocol](https://github.com/simpleledger/slp-specifications/blob/master/bitcoinfiles.md).

JSON oracle messages can be used, on their own, as sources of immutable, timestamped data or they can be used in conjunction with collaborative transaction protocols such as [Signal, Watch, and Pay (SWaP)](https://github.com/vinarmani/swap-protocol/blob/master/swap-protocol-spec.md#22-oracle-signals).

# Oracle Message Types
This specification describes a general structure for oracle messages as well as defining the particular structure of two types of oracle messages: price and event.

The specification can be extended with further oracle message types.

## Price Type
The “price” oracle message is used to communicate the price of a given asset - such as a currency, commodity, or security - in terms of the exchange rate of that asset for 1 BCH. Because utilizing whole-number integers provide the greatest degree of interoperability among wallets and script contracts, the specification for this type of message has a decimals property which can be used to express precise floating point / decimal numbers through the exclusive use of human-readable integers within the data.

### JSON EXAMPLE (Price)
``` 
{
    "type":"price",
    "data":{
        "asset":"usd",
        "rate":24592,
        "decimals":2,
        "height":636181,
        "messages":[
            {
                "id":"cashscript_priceoracle",
                "message":"15b5091060",
                "signature":"304402201d065ac37b1597f59432359b790735a1436fb5875eb7de6c854ea911ffcf1cd4022009f9c703c692486c279601df94c15feb24eb8558b5efc4181ae7f3effdd9c507"
            }
        ]
    },
    "contracts":{
        "jeton":{
            "threshold":{
                "parties":[
                    {
                        "name":"gt",
                        "weight":1
                    },
                    {
                        "name":"lte",
                        "weight":1
                    }
                ],
                "terms":[
                    {
                        "name":"oraclePubKey",
                        "type":"bytes",
                        "value":"0372aef47f9c18b4b4dd236eb6ad52bc81dc19302e30478385848ed478c9b2445c"
                    },
                    {
                        "name":"threshold",
                        "type":"int"
                    },
                    {
                        "name":"nLockTime",
                        "type":"int"
                    }
                ]
            }
        }
    }
}
```

* `type` - “price”: String defining the type of oracle message as price
* `data` - The metadata for this oracle message
  * `asset` - String representing the asset whose price is being reported
  * `rate` - Integer representing the exchange rate of the specified asset to 1 BCH (ie USD/BCH)
  * `decimals` - Integer representing the number of decimal places back - number of times to divide by 10 - from the final digit of rate should be added to deliver a human-readable value. For example, a price of 235.12 would be represented as rate=23512 and decimals=2
  * `height` - Integer representing the blockheight of the BCH blockchain at the time this price existed. This must, necessarily, be less than or equal to the current BCH blockheight
  * `messages` - Array of objects defining `OP_CHECKDATASIG` messages in various formats
    * `id` - String defining the name of the compiler template used to create the message. Generally a message for a price oracle will consist of concatenated bytes representing blockheight and price
    * `message` - Hexadecimal string representing message to be used when spending the contract using `OP_CHECKDATASIG`
    * `signature` - Hexadecimal string representing signature to be used when spending the contract using `OP_CHECKDATASIG`
* `contracts` - Object defining known contracts which this oracle can support using SWaP Protocol.
  * `compilerId` - String as object property referencing the compilerId field in a SWAP Offer Signal
  * `compilerVersion` - String as object property referencing the compilerVersion field in a SWAP Offer Signal
    * `parties` - Array of objects specifying the parties of the contract
      * `name` - String representing the name used for a given party in the contract compiler code
      * `weight` - Integer representing the relative weight of this party’s contribution to the contract. This can be used to specify desired “odds”. A weight of 2 for one party and 1 for the other party would mean that the former party would need to contribute twice as much to the funded contract (and appended fees) as the latter party in order for the contract to be considered valid in a SWaP-based contract negotiation and execution.
    * `terms` - Array of objects defining the contract terms, passed as arguments to the compiler at the time the script contract is created
      * `name` - String representing the name of the argument as defined in the compiler code
      * `type` - String representing the data type of the data to be passed to the compiler for this argument
      * `value` (optional) - A value that is required to be included in the contract
      
## Event Type

The “event” oracle message type allows for collaborative escrow based on the outcome of external events. Specifically, this oracle message can be used with script contracts that utilize the OP_CHECKDATASIG and OP_CHECKDATASIGVERIFY opcodes.

This message type consists of two chained messages where the first input of the “result” message chain comprising the Bitcoin Files Protocol data is the output “baton” of the initial “event” message.

### JSON EXAMPLE (Event)

```{
    "type":"event",
    "data":{
        "league":"nba",
        "date":"20200222",
        "status":"UNPLAYED",
        "awayTeam":{
            "id":"SAC",
            "name":"Sacramento Kings",
            "logo":"https://global.nba.com/media/img/teams/00/logos/SAC_logo.svg"
        },
        "homeTeam":{
            "id":"LAC",
            "name":"Los Angeles Clippers",
            "logo":"https://global.nba.com/media/img/teams/00/logos/LAC_logo.svg"
        }
    },
    "contracts":{
        "jeton":{
            "escrow":{
                "parties":[
                    {
                        "name":"SAC Wins",
                        "message":"NBA-20200222-SAC-LAC_SAC-WIN",
                        "weight":1
                    },
                    {
                        "name":"LAC Wins",
                        "message":"NBA-20200222-SAC-LAC_LAC-WIN",
                        "weight":1
                    }
                ],
                "terms":[
                    {
                        "name":"refereePubKey",
                        "type":"bytes",
                        "value":"02c3e42dd2a3806f1bc9a9f32c3a97b872ed03ce8a779242b8bf2dba636ce655b0"
                    }
                ]
            }
        }
    }
}
```

* `type` - “event”: String defining the type of oracle message as an event
* `data` - The metadata for this oracle message. There is no required data for the event type in this standard. By convention, the data object will be parsed and displayed by the end-user’s wallet.
* `contracts` - Object defining known contracts which this oracle can support using SWaP Protocol.
  * `compilerId` - String as object property referencing the compilerId field in a SWAP Offer Signal
    * `compilerVersion` - String as object property referencing the compilerVersion field in a SWAP Offer Signal
      * `parties` - Array of objects specifying the parties of the contract
        * `name` - String representing the name used for a given party in the contract compiler code
        * `message` - String representing the message that will be signed in the result Oracle message
        * `weight` - Integer representing the relative weight of this party’s contribution to the contract. This can be used to specify desired “odds”. A weight of 2 for one party and 1 for the other party would mean that the former party would need to contribute twice as much to the funded contract (and appended fees) as the latter party in order for the contract to be considered valid in a SWaP-based contract negotiation and execution.
      * `terms` - Array of objects defining the contract terms, passed as arguments to the compiler at the time the script contract is created
        * `name` - String representing the name of the argument as defined in the compiler code
        * `type` - String representing the data type of the data to be passed to the compiler for this argument
        * `value` (optional) - A value that is required to be included in the contract

### JSON EXAMPLE (Result)

```
{
    "type":"result",
    "data":{
        "finalScore":{
            "awayTeam":112,
            "homeTeam":103
        }
    },
    "contracts":{
        "jeton":{
            "escrow":[
                {
                    "id":"message",
                    "type":"string",
                    "value":"NBA-20200222-SAC-LAC_SAC-WIN"
                },
                {
                    "id":"refereeSig",
                    "type":"bytes",
                    "value":"304402202d435db55d497053e9b9fc7620985c99a8f7ff62d164446731473fddc1661baa02205f986222198947c0897aadc70982203a1ee6021a86aa9b0da57ca93e1a43fc8e"
                }
            ]
        }
    }
}
```

* `data` - The metadata for this oracle message. There is no required data for the event type in this standard. By convention this data provides information about the outcome of the event
* `contracts` - Object defining known contracts which this oracle can support using SWaP Protocol.
  * `compilerId` - String as object property referencing the compilerId field in a SWAP Offer Signal
    * `compilerVersion` - String as object property referencing the compilerVersion field in a SWAP Offer Signal. Identifies an array of objects with data necessary to properly sign contract "payout" inputs
      * `id` - String representing the name of the argument as defined in the compiler code
      * `type` - String representing the data type of the data to be passed to the compiler for this argument
      * `value` - A value that is required to be included in the scriptSig (unlocking script)

