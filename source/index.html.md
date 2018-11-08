---
title: Sia API Documentation

language_tabs: # must be one of https://git.io/vQNgJ
  - go

toc_footers:
  - <a href='https://sia.tech'>The Official Sia Website
  - <a href='https://gitlab.com/NebulousLabs/Sia'>Sia on GitLab</a>

includes:
  - errors

search: true
---

# Introduction

**Welcome to the Sia Storage Platform API!**

Sia uses semantic versioning and is backwards compatible to version v1.0.0.

API calls return either JSON or no content. Success is indicated by 2xx HTTP status codes, while errors are indicated by 4xx and 5xx HTTP status codes. If an endpoint does not specify its expected status code refer to [standard responses](#Standard-Responses).

There may be functional API calls which are not documented. These are not guaranteed to be supported beyond the current release, and should not be used in production.

**Notes:**

- Requests must set their User-Agent string to contain the substring "Sia-Agent".
- By default, siad listens on "localhost:9980". This can be changed using the `--api-addr` flag when running siad.
- **Do not bind or expose the API to a non-loopback address unless you are aware of the possible dangers.**

Example GET curl call:

`curl -A "Sia-Agent" "localhost:9980/wallet/transactions?startheight=1&endheight=250"`

Example POST curl call:

`curl -A "Sia-Agent" --data "amount=123&destination=abcd" "localhost:9980/wallet/siacoins"`

# Standard Responses

## Success
The standard response indicating the request was successfully processed is HTTP status code `204 No Content`. If the request was successfully processed and the server responded with JSON the HTTP status code is `200 OK`. Specific endpoints may specify other 2xx status codes on success.

## Error

```
{
    "message": String

    // There may be additional fields depending on the specific error.
}
```

The standard error response indicating the request failed for any reason, is a 4xx or 5xx HTTP status code with an error JSON object describing the error.

# Authentication

API authentication can be enabled with the `--authenticate-api` siad flag. Authentication is HTTP Basic Authentication as described in [RFC 2617](https://tools.ietf.org/html/rfc2617), however, the username is the empty string. The flag does not enforce authentication on all API endpoints. Only endpoints that expose sensitive information or modify state require authentication.

For example, if the API password is "foobar" the request header should include

`Authorization: Basic OmZvb2Jhcg==`

# Units

Unless otherwise noted, all parameters should be identified in their smallest possible unit. For example, size should always be specified in bytes and Siacoins should always be specified in hastings. JSON values returned by the API will also use the smallest possible unit, unless otherwise noted.

If a number is returned as a string in JSON, it should be treated as an arbitrary-precision number (bignum), and it should be parsed with your language's corresponding bignum library. Currency values are the most common example where this is necessary.

# Daemon

For examples and detailed descriptions of request and response parameters, refer to Daemon.md.

## /daemon/constants [GET]

```go
{
  "blockfrequency":         600,        // seconds per block
  "blocksizelimit":         2000000,    // bytes
  "extremefuturethreshold": 10800,      // seconds
  "futurethreshold":        10800,      // seconds
  "genesistimestamp":       1257894000, // Unix time
  "maturitydelay":          144,        // blocks
  "mediantimestampwindow":  11,         // blocks
  "siafundcount":           "10000",
  "siafundportion":         "39/1000",
  "targetwindow":           1000,       // blocks

  "initialcoinbase": 300000, // Siacoins (see note in Daemon.md)
  "minimumcoinbase": 30000,  // Siacoins (see note in Daemon.md)

  "roottarget": [0,0,0,0,32,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
  "rootdepth":  [255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255],

  "maxtargetadjustmentup":   "5/2",
  "maxtargetadjustmentdown": "2/5",

  "siacoinprecision": "1000000000000000000000000" // hastings per siacoin
}
```

Returns the set of constants in use.

`"blockfrequency":         600,        // seconds per block`
// Target for how frequently new blocks should be mined.

`"blocksizelimit":         2000000,    // bytes`
// Maximum size, in bytes, of a block. Blocks larger than this will be rejected by peers.

`"extremefuturethreshold": 10800,      // seconds`
// Farthest a block's timestamp can be in the future before the block is rejected outright.

`"futurethreshold":        10800,      // seconds`
// How far in the future a block can be without being rejected. A block further into the future will not be accepted immediately, but the daemon will attempt to accept the block as soon as it is valid.

`"genesistimestamp":       1257894000, // Unix time`
// Timestamp of the genesis block.

`"maturitydelay":          144,        // blocks`
// Number of children a block must have before it is considered "mature."

`"mediantimestampwindow":  11,         // blocks`
// Duration of the window used to adjust the difficulty.

`"siafundcount":           "10000",`
// Total number of siafunds.

`"siafundportion":         "39/1000",`
// Fraction of each file contract payout given to siafund holders.

`"targetwindow":           1000,       // blocks`
// Height of the window used to adjust the difficulty.

`"initialcoinbase": 300000, // Siacoins`
// Number of coins given to the miner of the first block. Note that elsewhere in the API currency is typically returned in hastings and as a bignum. This is not the case here.

`"minimumcoinbase": 30000,  // Siacoins`
// Minimum number of coins paid out to the miner of a block (the coinbase decreases with each block). Note that elsewhere in the API currency is typically returned in hastings and as a bignum. This is not the case here.

`"roottarget": [0,0,0,0,32,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],`
// Initial target.

`"rootdepth":  [255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255,255],`
// Initial depth.

`"maxtargetadjustmentup":   "5/2",`
// Largest allowed ratio between the old difficulty and the new difficulty.

`"maxtargetadjustmentdown": "2/5",`
// Smallest allowed ratio between the old difficulty and the new difficulty.

`"siacoinprecision": "1000000000000000000000000" // hastings per siacoin`
// Number of Hastings in one Siacoin.

## /daemon/stop [GET]

cleanly shuts down the daemon. This may take a few seconds.

### Response
standard success or error response. See [standard responses](#Standard-Responses).

## /daemon/version [GET]

```go
{
"version": "1.3.4"
}
```

Returns the version of the Sia daemon currently running. This number is visible to its peers on the network.

# Consensus

## /consensus [GET]

```go
{
  "synced":       true,
  "height":       62248,
  "currentblock": "00000000000008a84884ba827bdc868a17ba9c14011de33ff763bd95779a9cf1",
  "target":       [0,0,0,0,0,0,11,48,125,79,116,89,136,74,42,27,5,14,10,31,23,53,226,238,202,219,5,204,38,32,59,165],
  "difficulty":   "1234"
}
```

Returns information about the consensus set, such as the current block height.

  `"synced": true,`
  // True if the consensus set is synced with the network, e.g. it has downloaded the entire blockchain.

  `"height": 62248,`
  // Number of blocks preceding the current block.

  `"currentblock": "00000000000008a84884ba827bdc868a17ba9c14011de33ff763bd95779a9cf1",`
  // Hash of the current block.

  `"target": [0,0,0,0,0,0,11,48,125,79,116,89,136,74,42,27,5,14,10,31,23,53,226,238,202,219,5,204,38,32,59,165],`
  // An immediate child block of this block must have a hash less than this target for it to be valid.

  `"difficulty": "1234" // arbitrary-precision integer`
  // The difficulty of the current block target.

## /consensus/blocks [GET]

```go
{
    "height": 20032,
    "id": "00000000000033b9eb57fa63a51adeea857e70f6415ebbfe5df2a01f0d0477f4",
    "minerpayouts": [
        {
            "unlockhash": "c199cd180e19ef7597bcf4beecdd4f211e121d085e24432959c42bdf9030e32b9583e1c2727c",
            "value": "279978000000000000000000000000"
        }
    ],
    "nonce": [4,12,219,7,0,0,0,0],
    "parentid": "0000000000009615e8db750eb1226aa5e629bfa7badbfe0b79607ec8b918a44c",
    "timestamp": 1444516982,
    "transactions": [
	{
	    // ...
	}
        {
            "arbitrarydata": [],
            "filecontractrevisions": [],
            "filecontracts": [],
            "minerfees": [],
            "siacoininputs": [
                {
                    "parentid": "24cbeb9df7eb2d81d0025168fc94bd179909d834f49576e65b51feceaf957a64",
                    "unlockconditions": {
                        "publickeys": [
                            {
                                "algorithm": "ed25519",
                                "key": "QET8w7WRbGfcnnpKd1nuQfE3DuNUUq9plyoxwQYDK4U="
                            }
                        ],
                        "signaturesrequired": 1,
                        "timelock": 0
                    }
                }
            ],
            "siacoinoutputs": [
                {
                    "unlockhash": "d54f500f6c1774d518538dbe87114fe6f7e6c76b5bc8373a890b12ce4b8909a336106a4cd6db",
                    "value": "1010000000000000000000000000"
                },
                {
                    "unlockhash": "48a56b19bd0be4f24190640acbd0bed9669ea9c18823da2645ec1ad9652f10b06c5d4210f971",
                    "value": "5780000000000000000000000000"
                }
            ],
            "siafundinputs": [],
            "siafundoutputs": [],
            "storageproofs": [],
            "transactionsignatures": [
                {
                    "coveredfields": {
                        "arbitrarydata": [],
                        "filecontractrevisions": [],
                        "filecontracts": [],
                        "minerfees": [],
                        "siacoininputs": [],
                        "siacoinoutputs": [],
                        "siafundinputs": [],
                        "siafundoutputs": [],
                        "storageproofs": [],
                        "transactionsignatures": [],
                        "wholetransaction": true
                    },
                    "parentid": "24cbeb9df7eb2d81d0025168fc94bd179909d834f49576e65b51feceaf957a64",
                    "publickeyindex": 0,
                    "signature": "pByLGMlvezIZWVZmHQs/ynGETETNbxcOY/kr6uivYgqZqCcKTJ0JkWhcFaKJU+3DEA7JAloLRNZe3PTklD3tCQ==",
                    "timelock": 0
                }
            ]
        },
        {
	    // ...
        }
    ]
}
```

Returns the block for a given id or height.

### Query String Parameters

One of the following parameters can be specified.

// BlockID of the requested block.
`id` 

// BlockHeight of the requested block.
`height`

## /consensus/validate/transactionset [POST]

validates a set of transactions using the current utxo set.

### Request Body Bytes

Since transactions may be large, the transaction set is supplied in the POST body, encoded in JSON format.

### Response

standard success or error response. See [standard responses](#Standard-Responses).

# Gateway

## /gateway [GET]

```go
{
    "netaddress": String,
    "peers":      []{
        "netaddress": String,
        "version":    String,
        "inbound":    Boolean
    }
}

> Example

{
    "netaddress":"333.333.333.333:9981",
    "peers":[
        {
            "netaddress":"222.222.222.222:9981",
            "version":"1.0.0",
            "inbound":false
        },
        {
            "netaddress":"111.111.111.111:9981",
            "version":"0.6.0",
            "inbound":true
        }
    ]
}
```

returns information about the gateway, including the list of connected peers.

`"netaddress": String,` // netaddress is the network address of the gateway as seen by the rest of the network. The address consists of the external IP address and the port Sia is listening on. It represents a `modules.NetAddress`.

`"peers":      []{` // peers is an array of peers the gateway is connected to. It represents an array of `modules.Peer`s.
        
`"netaddress": String,` // netaddress is the address of the peer. It represents a `modules.NetAddress`.
        
`"version": String,` // version is the version number of the peer.

`"inbound": Boolean,` // inbound is true when the peer initiated the connection. This field is exposed as outbound peers are generally trusted more than inbound peers, as inbound peers are easily manipulated by an adversary.

`"local": Boolean` // local is true if the peer's IP address belongs to a local address range such as 192.168.x.x or 127.x.x.x

## /gateway/connect/:*netaddress* [POST]

connects the gateway to a peer. The peer is added to the node list if it is not already present. The node list is the list of all nodes the gateway knows about, but is not necessarily connected to.

### Path Parameters

netaddress is the address of the peer to connect to. It should be a reachable ip address and port number, of the form `IP:port`. IPV6 addresses must be enclosed in square brackets.  
  
Example IPV4 address: 123.456.789.0:123  
Example IPV6 address: [123::456]:789  
`:netaddress`

### Response
standard success or error response. See [standard responses](#Standard-Responses).

## /gateway/disconnect/:*netaddress* [POST]

disconnects the gateway from a peer. The peer remains in the node list. Disconnecting from a peer does not prevent the gateway from automatically connecting to the peer in the future.

### Path Parameters

netaddress is the address of the peer to connect to. It should be a reachable ip address and port number, of the form `IP:port`. IPV6 addresses must be enclosed in square brackets.  
  
Example IPV4 address: 123.456.789.0:123  
Example IPV6 address: [123::456]:789  
`:netaddress`

### Response
standard success or error response. See [standard responses](#Standard-Responses).

# Host

## /host [GET]

```go
{
  "externalsettings": {
    "acceptingcontracts":   true,
    "maxdownloadbatchsize": 17825792, // bytes
    "maxduration":          25920,    // blocks
    "maxrevisebatchsize":   17825792, // bytes
    "netaddress":           "123.456.789.0:9982",
    "remainingstorage":     35000000000, // bytes
    "sectorsize":           4194304,     // bytes
    "totalstorage":         35000000000, // bytes
    "unlockhash":           "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789ab",
    "windowsize":           144, // blocks

    "collateral":    "57870370370",                     // hastings / byte / block
    "maxcollateral": "100000000000000000000000000000",  // hastings

    "contractprice":          "30000000000000000000000000", // hastings
    "downloadbandwidthprice": "250000000000000",            // hastings / byte
    "storageprice":           "231481481481",               // hastings / byte / block
    "uploadbandwidthprice":   "100000000000000",            // hastings / byte

    "revisionnumber": 0,
    "version":        "1.0.0"
  },

  "financialmetrics": {
    "contractcount":                 2,
    "contractcompensation":          "123", // hastings
    "potentialcontractcompensation": "123", // hastings

    "lockedstoragecollateral": "123", // hastings
    "lostrevenue":             "123", // hastings
    "loststoragecollateral":   "123", // hastings
    "potentialstoragerevenue": "123", // hastings
    "riskedstoragecollateral": "123", // hastings
    "storagerevenue":          "123", // hastings
    "transactionfeeexpenses":  "123", // hastings

    "downloadbandwidthrevenue":          "123", // hastings
    "potentialdownloadbandwidthrevenue": "123", // hastings
    "potentialuploadbandwidthrevenue":   "123", // hastings
    "uploadbandwidthrevenue":            "123"  // hastings
  },

  "internalsettings": {
    "acceptingcontracts":   true,
    "maxdownloadbatchsize": 17825792, // bytes
    "maxduration":          25920,    // blocks
    "maxrevisebatchsize":   17825792, // bytes
    "netaddress":           "123.456.789.0:9982",
    "windowsize":           144, // blocks

    "collateral":       "57870370370",                     // hastings / byte / block
    "collateralbudget": "2000000000000000000000000000000", // hastings
    "maxcollateral":    "100000000000000000000000000000",  // hastings

    "mincontractprice":          "30000000000000000000000000", // hastings
    "mindownloadbandwidthprice": "250000000000000",            // hastings / byte
    "minstorageprice":           "231481481481",               // hastings / byte / block
    "minuploadbandwidthprice":   "100000000000000"             // hastings / byte
  },

  "networkmetrics": {
    "downloadcalls":     0,
    "errorcalls":        1,
    "formcontractcalls": 2,
    "renewcalls":        3,
    "revisecalls":       4,
    "settingscalls":     5,
    "unrecognizedcalls": 6
  },

  "connectabilitystatus": "checking",
  "workingstatus":        "checking"
}
```

fetches status information about the host.

`"externalsettings": {`
// The settings that get displayed to untrusted nodes querying the host's status.
  
`"acceptingcontracts": Boolean,`
// Whether or not the host is accepting new contracts.

`"maxdownloadbatchsize": 17825792, // bytes`
// The maximum size of a single download request from a renter. Each download request has multiple round trips of communication that exchange money. Larger batch sizes mean fewer round trips, but more financial risk for the host - the renter can get a free batch when downloading by refusing to provide a signature.

`"maxduration": 25920, // blocks`
// The maximum duration that a host will allow for a file contract. The host commits to keeping files for the full duration under the threat of facing a large penalty for losing or dropping data before the duration is complete. The storage proof window of an incoming file contract must end before the current height + maxduration.

`"maxrevisebatchsize": 17825792, // bytes`
    // The maximum size of a single batch of file contract revisions. The renter can perform DoS attacks on the host by uploading a batch of data then refusing to provide a signature to pay for the data. The host can reduce this exposure by limiting the batch size. Larger batch sizes allow for higher throughput as there is significant communication overhead associated with performing a batch upload.

`"netaddress": "123.456.789.0:9982"`
// The IP address or hostname (including port) that the host should be contacted at.

`"remainingstorage": 35000000000, // bytes`
    // The amount of unused storage capacity on the host in bytes. It should be noted that the host can lie.

`"sectorsize": 4194304, // bytes`
// The smallest amount of data in bytes that can be uploaded or downloaded when performing calls to the host.

`"totalstorage": 35000000000, // bytes`
    // The total amount of storage capacity on the host. It should be noted that the host can lie.

`"unlockhash": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789ab",`
    // The unlock hash is the address at which the host can be paid when forming file contracts.

`"windowsize": 144, // blocks`
// The storage proof window is the number of blocks that the host has to get a storage proof onto the blockchain. The window size is the minimum size of window that the host will accept in a file contract.

`"collateral": "57870370370", // hastings / byte / block`
    // The maximum amount of money that the host will put up as collateral for storage that is contracted by the renter.

`"maxcollateral": "100000000000000000000000000000",  // hastings`
// The maximum amount of collateral that the host will put into a single file contract.

`"contractprice": "30000000000000000000000000", // hastings`
// The price that a renter has to pay to create a contract with the host. The payment is intended to cover transaction fees for the file contract revision and the storage proof that the host will be submitting to the blockchain.

`"downloadbandwidthprice": "250000000000000", // hastings / byte`
// The price that a renter has to pay when downloading data from the host.

`"storageprice": "231481481481", // hastings / byte / block`
// The price that a renter has to pay to store files with the host.

`"uploadbandwidthprice": "100000000000000", // hastings / byte`
// The price that a renter has to pay when uploading data to the host.

`"revisionnumber": 0,`
// The revision number indicates to the renter what iteration of settings the host is currently at. Settings are generally signed. If the renter has multiple conflicting copies of settings from the host, the renter can expect the one with the higher revision number to be more recent.

`"version": "1.0.0"`
// The version of external settings being used. This field helps coordinate updates while preserving compatibility with older nodes.

`"financialmetrics": {`
// The financial status of the host.
  
`"contractcount": 2,`
// Number of open file contracts.

`"contractcompensation": "123", // hastings`
// The amount of money that renters have given to the host to pay for file contracts. The host is required to submit a file contract revision and a storage proof for every file contract that gets created, and the renter pays for the miner fees on these objects.

`"potentialcontractcompensation": "123", // hastings`
// The amount of money that renters have given to the host to pay for file contracts which have not been confirmed yet. The potential compensation becomes compensation after the storage proof is submitted.

`"lockedstoragecollateral": "123", // hastings`
// The amount of storage collateral which the host has tied up in file contracts. The host has to commit collateral to a file contract even if there is no storage, but the locked collateral will be returned even if the host does not submit a storage proof - the collateral is not at risk, it is merely set aside so that it can be put at risk later.

`"lostrevenue": "123", // hastings`
// The amount of revenue, including storage revenue and bandwidth revenue, that has been lost due to failed file contracts and failed storage proofs.

`"loststoragecollateral": "123", // hastings`
// The amount of collateral that was put up to protect data which has been lost due to failed file contracts and missed storage proofs.

`"potentialstoragerevenue": "123", // hastings`
// The amount of revenue that the host stands to earn if all storage proofs are submitted corectly and in time.

`"riskedstoragecollateral": "123", // hastings`
// The amount of money that the host has risked on file contracts. If the host starts missing storage proofs, the host can forfeit up to this many coins. In the event of a missed storage proof, locked storage collateral gets returned, but risked storage collateral does not get returned.

`"storagerevenue": "123", // hastings`
// The amount of money that the host has earned from storing data. This money has been locked down by successful storage proofs.

`"transactionfeeexpenses": "123", // hastings`
// The amount of money that the host has spent on transaction fees when submitting host announcements, file contract revisions, and storage proofs.

`"downloadbandwidthrevenue": "123", // hastings`
// The amount of money that the host has made from renters downloading their files. This money has been locked in by successsful storage proofs.

`"potentialdownloadbandwidthrevenue": "123", // hastings`
// The amount of money that the host stands to make from renters that downloaded their files. The host will only realize this revenue if the host successfully submits storage proofs for the related file contracts.

`"potentialuploadbandwidthrevenue": "123", // hastings`
// The amount of money that the host stands to make from renters that uploaded files. The host will only realize this revenue if the host successfully submits storage proofs for the related file contracts.

`"uploadbandwidthrevenue": "123" // hastings`
// The amount of money that the host has made from renters uploading their files. This money has been locked in by successful storage proofs.

`"internalsettings": {`
// The settings of the host. Most interactions between the user and the host occur by changing the internal settings.

`"acceptingcontracts": true,`
// When set to true, the host will accept new file contracts if the terms are reasonable. When set to false, the host will not accept new file contracts at all.

`"maxdownloadbatchsize": 17825792, // bytes`
// The maximum size of a single download request from a renter. Each download request has multiple round trips of communication that exchange money. Larger batch sizes mean fewer round trips, but more financial risk for the host - the renter can get a free batch when downloading by refusing to provide a signature.

`"maxduration": 25920, // blocks`
// The maximum duration of a file contract that the host will accept. The storage proof window must end before the current height + maxduration.

`"maxrevisebatchsize": 17825792, // bytes`
// The maximum size of a single batch of file contract revisions. The renter can perform DoS attacks on the host by uploading a batch of data then refusing to provide a signature to pay for the data. The host can reduce this exposure by limiting the batch size. Larger batch sizes allow for higher throughput as there is significant communication overhead associated with performing a batch upload.

`"netaddress": "123.456.789.0:9982",`
// The IP address or hostname (including port) that the host should be contacted at. If left blank, the host will automatically figure out its ip address and use that. If given, the host will use the address given.

`"windowsize": 144, // blocks`
// The storage proof window is the number of blocks that the host has to get a storage proof onto the blockchain. The window size is the minimum size of window that the host will accept in a file contract.

`"collateral": "57870370370", // hastings / byte / block`
 // The maximum amount of money that the host will put up as collateral per byte per block of storage that is contracted by the renter.

`"collateralbudget": "2000000000000000000000000000000", // hastings`
    // The total amount of money that the host will allocate to collateral across all file contracts.

`"maxcollateral": "100000000000000000000000000000", // hastings`
    // The maximum amount of collateral that the host will put into a
single file contract.

`"mincontractprice": "30000000000000000000000000", // hastings`
    // The minimum price that the host will demand from a renter when forming a contract. Typically this price is to cover transaction fees on the file contract revision and storage proof, but can also be used if the host has a low amount of collateral. The price is a minimum because the host may automatically adjust the price upwards in times of high demand.

`"mindownloadbandwidthprice": "250000000000000", // hastings / byte`
    // The minimum price that the host will demand from a renter when the renter is downloading data. If the host is saturated, the host may increase the price from the minimum.

`"minstorageprice": "231481481481", // hastings / byte / block`
    // The minimum price that the host will demand when storing data for extended periods of time. If the host is low on space, the price of storage may be set higher than the minimum.

`"minuploadbandwidthprice": "100000000000000" // hastings / byte`
    // The minimum price that the host will demand from a renter when the renter is uploading data. If the host is saturated, the host may increase the price from the minimum.

`"networkmetrics": {`
  // Information about the network, specifically various ways in which renters have contacted the host.

`"downloadcalls": 0,`
    // The number of times that a renter has attempted to download something from the host.

`"errorcalls": 1,`
    // The number of calls that have resulted in errors. A small number of errors are expected, but a large number of errors indicate either buggy software or malicious network activity. Usually buggy software.

`"formcontractcalls": 2,`
    // The number of times that a renter has tried to form a contract with the host.

`"renewcalls": 3,`
    // The number of times that a renter has tried to renew a contract with the host.

`"revisecalls": 4,`
// The number of times that the renter has tried to revise a contract with the host.

`"settingscalls": 5,`
// The number of times that a renter has queried the host for the host's settings. The settings include the price of bandwidth, which is a price that can adjust every few minutes. This value is usually very high compared to the others.

`"unrecognizedcalls": 6`
// The number of times that a renter has attempted to use an unrecognized call. Larger numbers typically indicate buggy software.

`"connectabilitystatus": "checking",`
// connectabilitystatus is one of "checking", "connectable", or "not connectable", and indicates if the host can connect to itself on its configured NetAddress.

`"workingstatus": "checking"`
// workingstatus is one of "checking", "working", or "not working" and indicates if the host is being actively used by renters.

## /host [POST]

> Query String Parameters

```go
acceptingcontracts   // Optional, true / false
maxdownloadbatchsize // Optional, bytes
maxduration          // Optional, blocks
maxrevisebatchsize   // Optional, bytes
netaddress           // Optional
windowsize           // Optional, blocks

collateral       // Optional, hastings / byte / block
collateralbudget // Optional, hastings
maxcollateral    // Optional, hastings

mincontractprice          // Optional, hastings
mindownloadbandwidthprice // Optional, hastings / byte
minstorageprice           // Optional, hastings / byte / block
minuploadbandwidthprice   // Optional, hastings / byte
```

configures hosting parameters. All parameters are optional; unspecified parameters will be left unchanged.

`acceptingcontracts // Optional, true / false`
// When set to true, the host will accept new file contracts if the terms are reasonable. When set to false, the host will not accept new file contracts at all.

`maxdownloadbatchsize // Optional, bytes`
// The maximum size of a single download request from a renter. Each download request has multiple round trips of communication that exchange money. Larger batch sizes mean fewer round trips, but more financial risk for the host - the renter can get a free batch when downloading by refusing to provide a signature.

`maxduration // Optional, blocks`
// The maximum duration of a file contract that the host will accept. The storage proof window must end before the current height + maxduration.

`maxrevisebatchsize // Optional, bytes`
// The maximum size of a single batch of file contract revisions. The renter can perform DoS attacks on the host by uploading a batch of data then refusing to provide a signature to pay for the data. The host can reduce this exposure by limiting the batch size. Larger batch sizes allow for higher throughput as there is significant communication overhead associated with performing a batch upload.

`netaddress // Optional`
// The IP address or hostname (including port) that the host should be contacted at. If left blank, the host will automatically figure out its ip address and use that. If given, the host will use the address given.

`windowsize // Optional, blocks`
// The storage proof window is the number of blocks that the host has to get a storage proof onto the blockchain. The window size is the minimum size of window that the host will accept in a file contract.

`collateral // Optional, hastings / byte / block`
// The maximum amount of money that the host will put up as collateral per byte per block of storage that is contracted by the renter.

`collateralbudget // Optional, hastings`
// The total amount of money that the host will allocate to collateral across all file contracts.

`maxcollateral // Optional, hastings`
// The maximum amount of collateral that the host will put into a single file contract.

`mincontractprice // Optional, hastings`
// The minimum price that the host will demand from a renter when forming a contract. Typically this price is to cover transaction fees on the file contract revision and storage proof, but can also be used if the host has a low amount of collateral. The price is a minimum because the host may automatically adjust the price upwards in times of high demand.

`mindownloadbandwidthprice // Optional, hastings / byte`
// The minimum price that the host will demand from a renter when the renter is downloading data. If the host is saturated, the host may increase the price from the minimum.

`minstorageprice // Optional, hastings / byte / block`
// The minimum price that the host will demand when storing data for extended periods of time. If the host is low on space, the price of storage may be set higher than the minimum.

`minuploadbandwidthprice // Optional, hastings / byte`
// The minimum price that the host will demand from a renter when the renter is uploading data. If the host is saturated, the host may increase the price from the minimum.

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /host/announce [POST]

Announce the host to the network as a source of storage. Generally only needs to be called once.

Note that even after the host has been announced, it will not accept new contracts unless configured to do so. To configure the host to accept contracts, see [/host](## /host [POST]).

### Query String Parameters

`netaddress string // Optional`
// The address to be announced. If no address is provided, the automatically discovered address will be used instead.

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /host/contracts [GET]

```go
{
  "contracts": [
    {
      > Amount in hastings to cover the transaction fees for this storage obligation.
      "contractcost":			"1234",		// hastings
      > Size of the data that is protected by the contract.
      "datasize":			500000,		// bytes
      > Amount that is locked as collateral for this storage obligation.
      "lockedcollateral":		"1234",		// hastings
      > Id of the storageobligation, which is defined by the file contract id of the file contract that governs the storage obligation.
      "obligationid":			"fff48010dcbbd6ba7ffd41bc4b25a3634ee58bbf688d2f06b7d5a0c837304e13",
      > Potential revenue for downloaded data that the host will reveive upon successful completion of the obligation.
      "potentialdownloadrevenue":	"1234",		// hastings
      > Potential revenue for storage of data that the host will reveive upon successful completion of the obligation.
      "potentialstoragerevenue":	"1234",		// hastings
      > Potential revenue for uploaded data that the host will reveive upon successful completion of the obligation.
      "potentialuploadrevenue":		"1234",		// hastings
      > Amount that the host might lose if the submission of the storage proof is not successful.
      "riskedcollateral":		"1234",		// hastings
      > Number of sector roots.
      "sectorrootscount":		2,
      > Amount for transaction fees that the host added to the storage obligation.
      "transactionfeesadded":		"1234",		// hastings
      > Expiration height is the height at which the storage obligation expires.
      "expirationheight":		123456,		// blocks
      > Negotion height is the height at which the storage obligation was negotiated.
      "negotiationheight":		123456,		// blocks
      > The proof deadline is the height by which the storage proof must be submitted.
      "proofdeadline":			123456,		// blocks
      > Status of the storage obligation. There are 4 different statuses:
    // obligationFailed:	the storage obligation failed, potential revenues and risked collateral are lost
    // obligationRejected:	the storage obligation was never started, no revenues gained or lost
    // obligationSucceeded:	the storage obligation was completed, revenues were gained
    // obligationUnresolved: 	the storage obligation has an uninitialized value. When the "proofdeadline" is in the past this might be a stale obligation.
      "obligationstatus":		"obligationFailed",
      > Origin confirmed indicates whether the file contract was seen on the blockchain for this storage obligation.
      "originconfirmed":		true,
      > Proof confirmed indicates whether there was a storage proof seen on the blockchain for this storage obligation.
      "proofconfirmed":			true,
      > The host has constructed a storage proof
      "proofconstructed":		true
      > Revision confirmed indicates whether there was a file contract revision seen on the blockchain for this storage obligation.
      "revisionconfirmed":		false,
      > Revision constructed indicates whether there was a file contract revision constructed for this storage obligation.
      "revisionconstructed":		false,
    }
  ]
}
```

Get contract information from the host database. This call will return all storage obligations on the host. Its up to the caller to filter the contracts based on his needs.

## /host/storage [GET]

### JSON Response (with comments)

```go
{
  "folders": [
    {
      "path":              "/home/foo/bar",
      "capacity":          50000000000,     // bytes
      "capacityremaining": 100000,          // bytes

      "failedreads":      0,
      "failedwrites":     1,
      "successfulreads":  2,
      "successfulwrites": 3
    }
  ]
}
```

gets a list of folders tracked by the host's storage manager.

`"path": "/home/foo/bar",`
// Absolute path to the storage folder on the local filesystem.

`"capacity": 50000000000, // bytes`
// Maximum capacity of the storage folder. The host will not store more than this many bytes in the folder. This capacity is not checked against the drive's remaining capacity. Therefore, you must manually ensure the disk has sufficient capacity for the folder at all times. Otherwise you risk losing renter's data and failing storage proofs.

`"capacityremaining": 100000, // bytes`
// Unused capacity of the storage folder.

`"failedreads":  0, "failedwrites": 1,`
// Number of failed disk read & write operations. A large number of failed reads or writes indicates a problem with the filesystem or drive's hardware.

`"successfulreads":  2, "successfulwrites": 3`
// Number of successful read & write operations.

## /host/storage/folders/add [POST]

> Query String Parameters

```go
path // Required
size // bytes, Required
```

adds a storage folder to the manager. The manager may not check that there is enough space available on-disk to support as much storage as requested

`path // Required`
// Local path on disk to the storage folder to add.

`size // bytes, Required`
// Initial capacity of the storage folder. This value isn't validated so it is possible to set the capacity of the storage folder greater than the capacity of the disk. Do not do this.

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /host/storage/folders/remove [POST]

> Query String Parameters

```go
path  // Required
force // bool, Optional, default is false
```

remove a storage folder from the manager. All sotrage on the folder will be moved to other stoarge folders, meaning that no data will be lost. If the manager is unable to save data, an error will be returned and the operation will be stopped.

`path // Required`
// Local path on disk to the storage folder to removed.

`force // bool, Optional, default is false`
// If `force` is true, the storage folder will be removed even if the data in the storage folder cannot be moved to other storage folders, typically because they don't have sufficient capacity. If `force` is true and the data cannot be moved, data will be lost.

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /host/storage/folders/resize [POST]

> Query String Parameters

```go
path    // Required
newsize // bytes, Required
```

grows or shrinks a storage file in the manager. The manager may not check that there is enough space on-disk to support growing the storasge folder, but should gracefully handle running out of space unexpectedly. When shrinking a storage folder, any data in the folder that neeeds to be moved will be placed into other storage folders, meaning that no data will be lost. If the manager is unable to migrate the data, an error will be returned and the operation will be stopped.

`path // Required`
// Local path on disk to the storage folder to resize.

`newsize // bytes, Required`
// Desired new size of the storage folder. This will be the new capacity of the storage folder.

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /host/storage/sectors/delete/:*merkleroot* [POST]

> Path Parameters

```go
:merkleroot 
```

deletes a sector, meaning that the manager will be unable to upload that sector and be unable to provide a storage proof on that sector. This endpoint is for removing the data entirely, and will remove instances of the sector appearing at all heights. The primary purpose is to comply with legal requests to remove data.

`:merkleroot`
// Merkleroot of the sector to delete.

### Path Parameters (with comments)

`:merkleroot`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /host/estimatescore [GET]

```go
{
	"estimatedscore": "123456786786786786786786786742133",
	"conversionrate": 95
}
```

> Query String Parameters

```go
acceptingcontracts   // Optional, true / false
maxdownloadbatchsize // Optional, bytes
maxduration          // Optional, blocks
maxrevisebatchsize   // Optional, bytes
netaddress           // Optional
windowsize           // Optional, blocks

collateral       // Optional, hastings / byte / block
collateralbudget // Optional, hastings
maxcollateral    // Optional, hastings

mincontractprice          // Optional, hastings
mindownloadbandwidthprice // Optional, hastings / byte
minstorageprice           // Optional, hastings / byte / block
minuploadbandwidthprice   // Optional, hastings / byte
```

returns the estimated HostDB score of the host using its current settings, combined with the provided settings.

`"estimatedscore": "123456786786786786786786786742133",`
// estimatedscore is the estimated HostDB score of the host given the settings passed to estimatescore.
  
`"conversionrate": 95`
// conversionrate is the likelihood given the settings passed to estimatescore that the host will be selected by renters forming contracts.

# Host DB

## /hostdb [GET]

```go
{
    "initialscancomplete": false
}
```

shows some general information about the state of the hostdb.

### Response

either the listed JSON struct or an error response. See [standard responses](#Standard-Responses).

## /hostdb/active [GET]

> Query String Parameters

```go
numhosts // Optional
```

```go
{
  "hosts": [
    {
      "acceptingcontracts":   true,
      "maxdownloadbatchsize": 17825792, // bytes
      "maxduration":          25920,    // blocks
      "maxrevisebatchsize":   17825792, // bytes
      "netaddress":           "123.456.789.2:9982",
      "remainingstorage":     35000000000, // bytes
      "sectorsize":           4194304,     // bytes
      "totalstorage":         35000000000, // bytes
      "unlockhash":           "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789ab",
      "windowsize":           144, // blocks
      "publickey": {
        "algorithm": "ed25519",
        "key":        "RW50cm9weSBpc24ndCB3aGF0IGl0IHVzZWQgdG8gYmU="
      }
      "publickeystring": "ed25519:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
    }
  ]
}
```

lists all of the active hosts known to the renter, sorted by preference.

### Query String Parameters

`numhosts // Optional`
// Number of hosts to return. The actual number of hosts returned may be less if there are insufficient active hosts. Optional, the default is all active hosts.

### JSON Response

`"acceptingcontracts": true`
// true if the host is accepting new contracts.

`"collateral": "20000000000", // hastings / byte / block`
// The maximum amount of money that the host will put up as collateral for storage that is contracted by the renter

`"contractprice": "1000000000000000000000000", // hastings`
// The price that a renter has to pay to create a contract with the host. The payment is intended to cover transaction fees for the file contract revision and the storage proof that the host will be submitting to the blockchain.

`"downloadbandwidthprice": "35000000000000", // hastings / byte`
// The price that a renter has to pay when downloading data from the host.

`"firstseen": 160000, // blocks`
// Firstseen is the last block height at which this host was announced.

`"historicdowntime": 0`
// Total amount of time the host has been offline.

`"historicfailedinteractions": 0`
// Number of historic failed interactions with the host.

`"historicsuccessfulinteractions": 5`
// Number of historic successful interactions with the host.

`"historicuptime": 41634520900246576,`
// Total amount of time the host has been online.

`"ipnets": [
        "1.2.3.0",
        "2.1.3.0"
      ],`  
// List of IP subnet masks used by the host. For IPv4 the /24 and for IPv6 the /54 subnet mask is used. A host can have either one IPv4 or one IPv6 subnet or one of each. E.g. these lists are valid: [ "IPv4" ], [ "IPv6" ] or [ "IPv4", "IPv6" ]. The following lists are invalid: [ "IPv4", "IPv4" ], [ "IPv4", "IPv6", "IPv6" ]. Hosts with an invalid list are ignored.

`"lasthistoricupdate": 174900, // blocks`
// The last time that the interactions within scanhistory have been compressed into the historic ones

`"lastipnetchange": "2015-01-01T08:00:00.000000000+04:00"`
// The last time the list of IP subnet masks was updated. When equal subnet masks are found for different hosts, the host that occupies the subnet mask for a longer time is preferred.

`"maxcollateral": "1000000000000000000000000000", // hastings`
// The maximum amount of collateral that the host will put into a single file contract.

`"maxdownloadbatchsize": 17825792, // bytes`
// Maximum number of bytes that the host will allow to be requested by a single download request.

`"maxduration": 25920, // blocks`
// Maximum duration in blocks that a host will allow for a file contract. The host commits to keeping files for the full duration under the threat of facing a large penalty for losing or dropping data before the duration is complete. The storage proof window of an incoming file contract must end before the current height + maxduration.  

There is a block approximately every 10 minutes. e.g. 1 day = 144 blocks

`"maxrevisebatchsize": 17825792, // bytes`
// Maximum size in bytes of a single batch of file contract revisions. Larger batch sizes allow for higher throughput as there is significant communication overhead associated with performing a batch upload.

`"netaddress": "123.456.789.0:9982"`
// Remote address of the host. It can be an IPv4, IPv6, or hostname, along with the port. IPv6 addresses are enclosed in square brackets.

`"publickey": {`
// Public key used to identify and verify hosts.
        
`"algorithm": "ed25519",`
// Algorithm used for signing and verification. Typically "ed25519".

`"key": "RW50cm9weSBpc24ndCB3aGF0IGl0IHVzZWQgdG8gYmU="`
// Key used to verify signed host messages.

`"publickeystring": "ed25519:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"`
// The string representation of the full public key, used when calling /hostdb/hosts.

`"recentfailedinteractions": 0,`
// Number of recent failed interactions with the host.

`"recentsuccessfulinteractions": 0,`
// Number of recent successful interactions with the host.

`"remainingstorage": 35000000000, // bytes`
// Unused storage capacity the host claims it has.

`"revisionnumber": 12733798,`
// The revision number indicates to the renter what iteration of settings the host is currently at. Settings are generally signed. If the renter has multiple conflicting copies of settings from the host, the renter can expect the one with the higher revision number to be more recent.

`"scanhistory": [
        {
          "success": true,
          "timestamp": "2018-09-23T08:00:00.000000000+04:00"
        },
        {
          "success": true,
          "timestamp": "2018-09-23T06:00:00.000000000+04:00"
        },
        {
          "success": true,
          "timestamp": "2018-09-23T04:00:00.000000000+04:00"`  
// Measurements that have been taken on the host. The most recent measurements are kept in full detail.

`"sectorsize": 4194304, // bytes`
// Smallest amount of data in bytes that can be uploaded or downloaded to or from the host.

`"storageprice": "14000000000", // hastings / byte / block`
// The price that a renter has to pay to store files with the host.

`"totalstorage": 35000000000, // bytes`
// Total amount of storage capacity the host claims it has.

`"unlockhash": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789ab",`
// Address at which the host can be paid when forming file contracts.

`"uploadbandwidthprice": "3000000000000", // hastings / byte`

`"version": "1.3.7",`
// The version of the host.

`"windowsize": 144 // blocks`
// A storage proof window is the number of blocks that the host has to get a storage proof onto the blockchain. The window size is the minimum size of window that the host will accept in a file contract.

## /hostdb/all [GET]

```go
{
  "hosts": [
    {
      "acceptingcontracts":   true,
      "maxdownloadbatchsize": 17825792, // bytes
      "maxduration":          25920,    // blocks
      "maxrevisebatchsize":   17825792, // bytes
      "netaddress":           "123.456.789.2:9982",
      "remainingstorage":     35000000000, // bytes
      "sectorsize":           4194304,     // bytes
      "totalstorage":         35000000000, // bytes
      "unlockhash":           "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789ab",
      "windowsize":           144, // blocks
      "publickey": {
        "algorithm": "ed25519",
        "key":        "RW50cm9weSBpc24ndCB3aGF0IGl0IHVzZWQgdG8gYmU="
      }
      "publickeystring": "ed25519:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
    }
  ]
}
```

lists all of the hosts known to the renter. Hosts are not guaranteed to be in any particular order, and the order may change in subsequent calls.

### JSON Response (with comments)

`"acceptingcontracts": true,`
// true if the host is accepting new contracts.

`"collateral": "20000000000", // hastings / byte / block`
 // The maximum amount of money that the host will put up as collateral for storage that is contracted by the renter.

`"contractprice": "1000000000000000000000000", // hastings`
// The price that a renter has to pay to create a contract with the host. The payment is intended to cover transaction fees for the file contract revision and the storage proof that the host will be submitting to the blockchain.

`"downloadbandwidthprice": "35000000000000", // hastings / byte`
// The price that a renter has to pay when downloading data from the host.

`"firstseen": 160000, // blocks`
// Firstseen is the last block height at which this host was announced.

`"historicdowntime": 0,`
// Total amount of time the host has been offline.

`"historicfailedinteractions": 0,`
// Number of historic failed interactions with the host.

`"historicsuccessfulinteractions": 5,`
// Number of historic successful interactions with the host.

`"historicuptime": 41634520900246576,`
// Total amount of time the host has been online.

`"ipnets": [
        "1.2.3.0",
        "2.1.3.0"
      ],`
// List of IP subnet masks used by the host. For IPv4 the /24 and for IPv6 the /54 subnet mask is used. A host can have either one IPv4 or one IPv6 subnet or one of each. E.g. these lists are valid: [ "IPv4" ], [ "IPv6" ] or [ "IPv4", "IPv6" ]. The following lists are invalid: [ "IPv4", "IPv4" ], [ "IPv4", "IPv6", "IPv6" ]. Hosts with an invalid list are ignored.

`"lasthistoricupdate": 174900, // blocks`
// The last time that the interactions within scanhistory have been compressed into the historic ones

`"lastipnetchange": "2015-01-01T08:00:00.000000000+04:00",`
// The last time the list of IP subnet masks was updated. When equal subnet masks are found for different hosts, the host that occupies the subnet mask for a longer time is preferred.

`"maxcollateral": "1000000000000000000000000000", // hastings`
// The maximum amount of collateral that the host will put into a single file contract.

`"maxdownloadbatchsize": 17825792, // bytes`
// Maximum number of bytes that the host will allow to be requested by a single download request.

`"maxduration": 25920, // blocks`
// Maximum duration in blocks that a host will allow for a file contract. The host commits to keeping files for the full duration under the threat of facing a large penalty for losing or dropping data before the duration is complete. The storage proof window of an incoming file contract must end before the current height + maxduration.  

There is a block approximately every 10 minutes. e.g. 1 day = 144 blocks

`"maxrevisebatchsize": 17825792, // bytes`
// Maximum size in bytes of a single batch of file contract revisions. Larger batch sizes allow for higher throughput as there is significant communication overhead associated with performing a batch upload.

`"netaddress": "123.456.789.0:9982"`
// Remote address of the host. It can be an IPv4, IPv6, or hostname, along with the port. IPv6 addresses are enclosed in square brackets.

`"publickey": {`
// Public key used to identify and verify hosts.
        
`"algorithm": "ed25519",`
// Algorithm used for signing and verification. Typically "ed25519".

`"key": "RW50cm9weSBpc24ndCB3aGF0IGl0IHVzZWQgdG8gYmU="`
// Key used to verify signed host messages.

`"publickeystring": "ed25519:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"`
// The string representation of the full public key, used when calling /hostdb/hosts.

`"recentfailedinteractions": 0,`
// Number of recent failed interactions with the host.

`"recentsuccessfulinteractions": 0,`
// Number of recent successful interactions with the host.

`"remainingstorage": 35000000000, // bytes`
// Unused storage capacity the host claims it has.

`"revisionnumber": 12733798,`
// The revision number indicates to the renter what iteration of settings the host is currently at. Settings are generally signed. If the renter has multiple conflicting copies of settings from the host, the renter can expect the one with the higher revision number to be more recent.

`"scanhistory": [
        {
          "success": true,
          "timestamp": "2018-09-23T08:00:00.000000000+04:00"
        },
        {
          "success": true,
          "timestamp": "2018-09-23T06:00:00.000000000+04:00"
        },
        {
          "success": true,
          "timestamp": "2018-09-23T04:00:00.000000000+04:00"`  
// Measurements that have been taken on the host. The most recent measurements are kept in full detail.

`"sectorsize": 4194304, // bytes`
// Smallest amount of data in bytes that can be uploaded or downloaded to or from the host.

`"storageprice": "14000000000", // hastings / byte / block`
// The price that a renter has to pay to store files with the host.

`"totalstorage": 35000000000, // bytes`
// Total amount of storage capacity the host claims it has.

`"unlockhash": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789ab",`
// Address at which the host can be paid when forming file contracts.

`"uploadbandwidthprice": "3000000000000", // hastings / byte`

`"version": "1.3.7",`
// The version of the host.

`"windowsize": 144 // blocks`
// A storage proof window is the number of blocks that the host has to get a storage proof onto the blockchain. The window size is the minimum size of window that the host will accept in a file contract.

## /hostdb/hosts/:*pubkey* [GET]

> Path Parameters
`:pubkey`

```go
{
  "entry": {
    "acceptingcontracts":   true,
    "maxdownloadbatchsize": 17825792, // bytes
    "maxduration":          25920,    // blocks
    "maxrevisebatchsize":   17825792, // bytes
    "netaddress":           "123.456.789.0:9982",
    "remainingstorage":     35000000000, // bytes
    "sectorsize":           4194304,     // bytes
    "totalstorage":         35000000000, // bytes
    "unlockhash":           "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789ab",
    "windowsize":           144, // blocks
    "publickey": {
      "algorithm": "ed25519",
      "key":       "RW50cm9weSBpc24ndCB3aGF0IGl0IHVzZWQgdG8gYmU="
    }
    "publickeystring": "ed25519:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
  },
  "scorebreakdown": {
    "score": 1,

    "ageadjustment":              0.1234,
    "burnadjustment":             0.1234,
    "collateraladjustment":       23.456,
    "interactionadjustment":      0.1234,
    "priceadjustment":            0.1234,
    "storageremainingadjustment": 0.1234,
    "uptimeadjustment":           0.1234,
    "versionadjustment":          0.1234,
  }
}
```

fetches detailed information about a particular host, including metrics regarding the score of the host within the database. It should be noted that each renter uses different metrics for selecting hosts, and that a good score on in one hostdb does not mean that the host will be successful on the network overall.

### Path Parameters

`:pubkey`
// The public key of the host. Each public key identifies a single host.  

Example Pubkey: ed25519:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef

### JSON Response (with comments)

`"acceptingcontracts": true,`
// true if the host is accepting new contracts.

`"collateral": "20000000000", // hastings / byte / block`
// The maximum amount of money that the host will put up as collateral for storage that is contracted by the renter

`"contractprice": "1000000000000000000000000", // hastings`
// The price that a renter has to pay to create a contract with the host. The payment is intended to cover transaction fees for the file contract revision and the storage proof that the host will be submitting to the blockchain.

`"downloadbandwidthprice": "35000000000000", // hastings / byte`
// The price that a renter has to pay when downloading data from the host.

`"firstseen": 160000, // blocks`
// Firstseen is the last block height at which this host was announced.

`"historicdowntime": 0,`
// Total amount of time the host has been offline.

`"historicfailedinteractions": 0,`
// Number of historic failed interactions with the host.

`"historicsuccessfulinteractions": 5,`
// Number of historic successful interactions with the host.

`"historicuptime": 41634520900246576,`
// Total amount of time the host has been online.

`"ipnets": [
      "1.2.3.0",
      "2.1.3.0"
    ],`
// List of IP subnet masks used by the host. For IPv4 the /24 and for IPv6 the /54 subnet mask is used. A host can have either one IPv4 or one IPv6 subnet or one of each. E.g. these lists are valid: [ "IPv4" ], [ "IPv6" ] or [ "IPv4", "IPv6" ]. The following lists are invalid: [ "IPv4", "IPv4" ], [ "IPv4", "IPv6", "IPv6" ]. Hosts with an invalid list are ignored.

`"lasthistoricupdate": 174900, // blocks`
// The last time that the interactions within scanhistory have been compressed into the historic ones

`"lastipnetchange": "2015-01-01T08:00:00.000000000+04:00",`
// The last time the list of IP subnet masks was updated. When equal subnet masks are found for different hosts, the host that occupies the subnet mask for a longer time is preferred.

`"maxcollateral": "1000000000000000000000000000", // hastings`
// The maximum amount of collateral that the host will put into a single file contract.

`"maxdownloadbatchsize": 17825792, // bytes`
// Maximum number of bytes that the host will allow to be requested by a single download request.

`"maxduration": 25920, // blocks`
// Maximum duration in blocks that a host will allow for a file contract. The host commits to keeping files for the full duration under the threat of facing a large penalty for losing or dropping data before the duration is complete. The storage proof window of an incoming file contract must end before the current height + maxduration.  

There is a block approximately every 10 minutes. e.g. 1 day = 144 blocks

`"maxrevisebatchsize": 17825792, // bytes`
// Maximum size in bytes of a single batch of file contract revisions. Larger batch sizes allow for higher throughput as there is significant communication overhead associated with performing a batch upload.

`"netaddress": "123.456.789.0:9982",`
// Remote address of the host. It can be an IPv4, IPv6, or hostname, along with the port. IPv6 addresses are enclosed in square brackets.

`"publickey": {`
// Public key used to identify and verify hosts.

`"algorithm": "ed25519",`
// Algorithm used for signing and verification. Typically "ed25519".

`"key": "RW50cm9weSBpc24ndCB3aGF0IGl0IHVzZWQgdG8gYmU="`
// Key used to verify signed host messages.

`"publickeystring": "ed25519:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",`
// The string representation of the full public key, used when calling /hostdb/hosts.

`"recentfailedinteractions": 0,`
// Number of recent failed interactions with the host.

`"recentsuccessfulinteractions": 0,`
// Number of recent successful interactions with the host.

`"remainingstorage": 35000000000, // bytes`
// Unused storage capacity the host claims it has.

`"revisionnumber": 12733798,`
// The revision number indicates to the renter what iteration of settings the host is currently at. Settings are generally signed. If the renter has multiple conflicting copies of settings from the host, the renter can expect the one with the higher revision number to be more recent.

`"scanhistory": [
      {
        "success": true,
        "timestamp": "2018-09-23T08:00:00.000000000+04:00"
      },
      {
        "success": true,
        "timestamp": "2018-09-23T06:00:00.000000000+04:00"
      },
      {
        "success": true,
        "timestamp": "2018-09-23T04:00:00.000000000+04:00"
      }
    ],`
// Measurements that have been taken on the host. The most recent measurements are kept in full detail.

`"sectorsize": 4194304, // bytes`
// Smallest amount of data in bytes that can be uploaded or downloaded to or from the host.

`"storageprice": "14000000000", // hastings / byte / block`
// The price that a renter has to pay to store files with the host.

`"totalstorage": 35000000000, // bytes`
// Total amount of storage capacity the host claims it has.

`"unlockhash": "0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789ab",`
// Address at which the host can be paid when forming file contracts.

`"uploadbandwidthprice": "3000000000000", // hastings / byte`
// The price, in hastings, of the upload bandwidth.

`"version": "1.3.7",`
// The version of the host.

`"windowsize": 144 // blocks`
// A storage proof window is the number of blocks that the host has to get a storage proof onto the blockchain. The window size is the minimum size of window that the host will accept in a file contract.

`"scorebreakdown": {`
// A set of scores as determined by the renter. Generally, the host's final score is all of the values multiplied together. Modified renters may have additional criteria that they use to judge a host, or may ignore certin criteia. In general, these fields should only be used as a loose guide for the score of a host, as every renter sees the world differently and uses different metrics to evaluate hosts.
  
`"ageadjustment": 0.1234,`
// The multiplier that gets applied to the host based on how long it has been a host. Older hosts typically have a lower penalty.

`"burnadjustment": 23.456,`
// The multiplier that gets applied to the host based on how much proof-of-burn the host has performed. More burn causes a linear increase in score.

"collateraladjustment": 23.456,
// The multiplier that gets applied to a host based on how much collateral the host is offering. More collateral is typically better, though above a point it can be detrimental.

`"conversionrate": 9.12345,`
// conversionrate is the likelihood that the host will be selected by renters forming contracts.

`"interactionadjustment": 0.1234,`
// The multipler that gets applied to a host based on previous interactions with the host. A high ratio of successful interactions will improve this hosts score, and a high ratio of failed interactions will hurt this hosts score. This adjustment helps account for hosts that are on unstable connections, don't keep their wallets unlocked, ran out of funds, etc.

`"pricesmultiplier": 0.1234,`
// The multiplier that gets applied to a host based on the host's price. Lower prices are almost always better. Below a certain, very low price, there is no advantage.

`"score": 123456,`
// The overall score for the host. Scores are entriely relative, and are consistent only within the current hostdb. Between different machines, different configurations, and different versions the absolute scores for a given host can be off by many orders of magnitude. When displaying to a human, some form of normalization with respect to the other hosts (for example, divide all scores by the median score of the hosts) is recommended.

`"storageremainingadjustment": 0.1234,`
// The multiplier that gets applied to a host based on how much storage is remaining for the host. More storage remaining is better, to a point.

`"uptimeadjustment": 0.1234,`
// The multiplier that gets applied to a host based on the uptime percentage of the host. The penalty increases extremely quickly as uptime drops below 90%.

`"versionadjustment": 0.1234`
// The multiplier that gets applied to a host based on the version of Sia that they are running. Versions get penalties if there are known bugs, scaling limitations, performance limitations, etc. Generally, the most recent version is always the one with the highest score.

## /hostdb/filtermode [POST]

```go
{
  "filtermode": "whitelist", // can be either whitelist, blacklist, or disable
  "hosts": [                 // comma separated pubkeys
    "ed25519:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef",
    "ed25519:1234567890abcdef1234567890abcdef1234567890abcdef1234567890abcdef"
  ]
}
```

lets you enable and disable a filter mode for the hostdb. Currenlty the two modes supported are `blacklist` mode and `whitelist` mode. In `blacklist` mode, any hosts you identify as being on the `blacklist` will not be used to form contracts. In `whitelist` mode, only the hosts identified as being on the `whitelist` will be used to form contracts. In both modes, hosts that you are blacklisted will be filtered from your hostdb. To enable either mode, set `filtermode` to the desired mode and submit a list of host pubkeys as the corresponding `blacklist` or `whitelist`. To disable either list, the `host` field can be left blank (e.g. empty slice) and the `filtermode` should be set to `disable`.  

**NOTE:** Enabling and disabling a filter mode can result in changes with your current contracts with can result in an increase in contract fee spending. For example, if `blacklist` mode is enabled, any hosts that you currently have contracts with that are also on the provide list of `hosts` will have their contracts replaced with non-blacklisted hosts. When `whitelist` mode is enabled, contracts will be replaced until there are only contracts with whitelisted hosts. Even disabling a filter mode can result in a change in contracts if there are better scoring hosts in your hostdb that were previously being filtered out.

`"filtermode": "whitelist",`
// can be either whitelist, blacklist, or disable

`"hosts"`
// comma separated pubkeys

# Miner

## /miner [GET]

returns the status of the miner.

### JSON Response (with comments)

```go
{
  "blocksmined":      9001,
  "cpuhashrate":      1337,
  "cpumining":        false,
  "staleblocksmined": 0,
}
```

returns the status of the miner.

`"blocksmined": 9001,`
// Number of mined blocks. This value is remembered after restarting.

`"cpuhashrate": 1337,`
// How fast the cpu is hashing, in hashes per second.

`"cpumining": false,`
// true if the cpu miner is active.

`"staleblocksmined": 0,`
// Number of mined blocks that are stale, indicating that they are not included in the current longest chain, likely because some other block at the same height had its chain extended first.


## /miner/start [GET]

starts a single threaded CPU miner. Does nothing if the CPU miner is already running.

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /miner/stop [GET]

stops the cpu miner. Does nothing if the cpu miner is not running.

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /miner/header [GET]

provides a block header that is ready to be grinded on for work.

### Byte Response
For efficiency the header for work is returned as a raw byte encoding of the header, rather than encoded to JSON.

Blocks are mined by repeatedly changing the nonce of the header, hashing the header's bytes, and comparing the resulting hash to the target. The block with that nonce is valid if the hash is less than the target. If none of the 2^64 possible nonces result in a header with a hash less than the target, call /miner/header [GET] again to get a new block header with a different merkle root. The above process can then be repeated for the new block header.  

The other fields can generally be ignored. The parent block ID field is the hash of the parent block's header. Modifying this field will result in an orphan block. The timestamp is the time at which the block was mined and is set by the Sia Daemon. Modifying this field can result in invalid block. The merkle root is the merkle root of a merkle tree consisting of the timestamp, the miner outputs (one leaf per payout), and the transactions (one leaf per transaction). Modifying this field will result in an invalid block.

Field | Byte range within response | Byte range within header
-------------- | -------------- | --------------
target | [0-32)
header | [32-112)
parent block ID | [32-64) | [0-32)
nonce | [64-72) | [32-40)
timestamp | [72-80) | [40-48)
merkle root | [80-112) | [48-80)

## /miner/header [POST]

submits a header that has passed the POW.

### Byte Response
For efficiency headers are submitted as raw byte encodings of the header in the body of the request, rather than as a query string parameter or path parameter. The request body should contain only the 80 bytes of the encoded header. The encoding is the same encoding used in `/miner/header [GET]` endpoint.

Blocks are mined by repeatedly changing the nonce of the header, hashing the header's bytes, and comparing the resulting hash to the target. The block with that nonce is valid if the hash is less than the target. If none of the 2^64 possible nonces result in a header with a hash less than the target, call /miner/header [GET] again to get a new block header with a different merkle root. The above process can then be repeated for the new block header.  

The other fields can generally be ignored. The parent block ID field is the hash of the parent block's header. Modifying this field will result in an orphan block. The timestamp is the time at which the block was mined and is set by the Sia Daemon. Modifying this field can result in invalid block. The merkle root is the merkle root of a merkle tree consisting of the timestamp, the miner outputs (one leaf per payout), and the transactions (one leaf per transaction). Modifying this field will result in an invalid block.

Field | Byte range within response | Byte range within header
-------------- | -------------- | --------------
target | [0-32)
header | [32-112)
parent block ID | [32-64) | [0-32)
nonce | [64-72) | [32-40)
timestamp | [72-80) | [40-48)
merkle root | [80-112) | [48-80)

# Renter

## /renter [GET]

```go
{
  "settings": {
    "allowance": {
      "funds":       "1234", // hastings
      "hosts":       24,
      "period":      6048, // blocks
      "renewwindow": 3024  // blocks
    },
    "maxuploadspeed":     1234, // BPS
    "maxdownloadspeed":   1234, // BPS
    "streamcachesize":  4    
  },
  "financialmetrics": {
    "contractfees":     "1234", // hastings
    "contractspending": "1234", // hastings (deprecated, now totalallocated)
    "downloadspending": "5678", // hastings
    "storagespending":  "1234", // hastings
    "totalallocated":   "1234", // hastings
    "uploadspending":   "5678", // hastings
    "unspent":          "1234"  // hastings
  },
  "currentperiod": 200
}
```

returns the current settings along with metrics on the renter's spending.

`"settings": {`
// Settings that control the behavior of the renter.

`"allowance": {`
// Allowance dictates how much the renter is allowed to spend in a given period. Note that funds are spent on both storage and bandwidth.

`"funds": "1234", // hastings`
// Amount of money allocated for contracts. Funds are spent on both storage and bandwidth.

`"hosts":24,`
// Number of hosts that contracts will be formed with.

`"period": 6048, // blocks`
// Duration of contracts formed, in number of blocks.

`"renewwindow": 3024 // blocks`
// If the current blockheight + the renew window >= the height the contract is scheduled to end, the contract is renewed automatically. Is always nonzero.

`"maxuploadspeed":     1234, // bytes per second`
// MaxUploadSpeed by default is unlimited but can be set by the user to manage bandwidth.

`"maxdownloadspeed":   1234, // bytes per second`
// MaxDownloadSpeed by default is unlimited but can be set by the user to manage bandwidth.

`"streamcachesize":  4`
// The StreamCacheSize is the number of data chunks that will be cached during streaming.

`"financialmetrics": {`
// Metrics about how much the Renter has spent on storage, uploads, and downloads.

`"contractfees": "1234", // hastings`
// Amount of money spent on contract fees, transaction fees and siafund fees.

`"contractspending": "1234", // hastings, (deprecated, now totalallocated)`
// How much money, in hastings, the Renter has spent on file contracts, including fees.

`"downloadspending": "5678", // hastings`
// Amount of money spent on downloads.

`"storagespending": "1234", // hastings`
// Amount of money spend on storage.

`"totalallocated": "1234", // hastings`
// Total amount of money that the renter has put into contracts. Includes spent money and also money that will be returned to the renter.

`"uploadspending": "5678", // hastings`
// Amount of money spent on uploads.

`"unspent": "1234" // hastings`
// Amount of money in the allowance that has not been spent.

`"currentperiod": 200`
// Height at which the current allowance period began.

## /renter [POST]

> Query String Parameters
```go
funds // hastings
hosts
period      // block height
renewwindow // block height
```

modify settings that control the renter's behavior.

### Query Response Parameters

`checkforipviolation // true or false`
// Enables or disables the check for hosts using the same ip subnets within the hostdb. It's turned on by default and causes Sia to not form contracts with hosts from the same subnet and if such contracts already exist, it will deactivate the contract which has occupied that subnet for the shorter time.

`funds // hastings`
// Number of hastings allocated for file contracts in the given period.

`hosts`
// Number of hosts that contracts should be formed with. Files cannot be uploaded to more hosts than you have contracts with, and it's generally good to form a few more contracts than you need.

`period // block height`
// Duration of contracts formed. Must be nonzero.

`renewwindow // block height`
// Renew window specifies how many blocks before the expiration of the current contracts the renter will wait before renewing the contracts. A smaller renew window means that Sia must be run more frequently, but also means fewer total transaction fees. Storage spending is not affected by the renew window size.

`maxdownloadspeed`
// Max download speed permitted, speed provide in bytes per second

`maxuploadspeed`
// Max upload speed permitted, speed provide in bytes per second

`streamcachesize`
// Stream cache size specifies how many data chunks will be cached while streaming.  

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /renter/contract/cancel [POST]

> Query String Parameters

```go
// ID of the file contract
id
```
cancels a specific contract of the Renter.

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /renter/contracts [GET]

> Contract Parameters
```go
inactive   // true or false - Optional
expired    // true or false - Optional
```

```go
{
  "activecontracts": [
    {
      "downloadspending": "1234", // hastings
      "endheight": 50000, // block height
      "fees": "1234", // hastings
      "hostpublickey": {
        "algorithm": "ed25519",
        "key": "RW50cm9weSBpc24ndCB3aGF0IGl0IHVzZWQgdG8gYmU="
      },
      "id": "1234567890abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
      "lasttransaction": {},
      "netaddress": "12.34.56.78:9",
      "renterfunds": "1234", // hastings
      "size": 8192, // bytes
      "startheight": 50000, // block height
      "StorageSpending": "1234",
      "storagespending": "1234", // hastings
      "totalcost": "1234", // hastings
      "uploadspending": "1234" // hastings
      "goodforupload": true,
      "goodforrenew": false,
    }
  ],
  "inactivecontracts": [],
  "expiredcontracts": [],
}
```

returns the renter's contracts.  Active contracts are contracts that the Renter is currently using to store, upload, and download data, and are returned by default. Inactive contracts are contracts that are in the current period but are marked as not good for renew, these contracts have the potential to become active again but currently are not storing data.  Expired contracts are contracts not in the current period, where not more data is being stored and excess funds have been released to the renter.

`"downloadspending": "1234", // hastings`
// Amount of contract funds that have been spent on downloads.

`"endheight": 50000, // block height`
// Block height that the file contract ends on.

`"fees": "1234", // hastings`
// Fees paid in order to form the file contract.

`"hostpublickey": {  
 "algorithm": "ed25519",  
 "key": "RW50cm9weSBpc24ndCB3aGF0IGl0IHVzZWQgdG8gYmU="`  
// Public key of the host the contract was formed with.

`"id": "1234567890abcdef0123456789abcdef0123456789abcdef0123456789abcdef",`
// ID of the file contract.

`"lasttransaction": {},`
// A signed transaction containing the most recent contract revision.

`"netaddress": "12.34.56.78:9",`
// Address of the host the file contract was formed with.

`"renterfunds": "1234", // hastings`
// Remaining funds left for the renter to spend on uploads & downloads.

`"size": 8192, // bytes`
// Size of the file contract, which is typically equal to the number of bytes that have been uploaded to the host.

`"startheight": 50000, // block height`
// Block height that the file contract began on.

`"StorageSpending": 0,`
// DEPRECATED: This is the exact same value as StorageSpending, but it has incorrect capitalization. This was fixed in 1.3.2, but this field is kept to preserve backwards compatibility on clients who depend on the incorrect capitalization. This field will be removed in the future, so clients should switch to the StorageSpending field (above) with the correct lowercase name.

`"storagespending": "1234", // hastings`
// Amount of contract funds that have been spent on storage.

`"totalcost": "1234", // hastings`
// Total cost to the wallet of forming the file contract. This includes both the fees and the funds allocated in the contract.

`"uploadspending": "1234" // hastings`
// Amount of contract funds that have been spent on uploads.

`"goodforupload": true,`
// Signals if contract is good for uploading data

`"goodforrenew": false,`
// Signals if contract is good for a renewal

## /renter/downloads [GET]

```go
{
  "downloads": [
    {
      "destination":     "/home/users/alice/bar.txt",
      "destinationtype": "file",
      "length":          8192,
      "offset":          2000,
      "siapath":         "foo/bar.txt",

      "completed":           true,
      "endtime":             "2009-11-10T23:10:00Z", // RFC 3339 time
      "error":               "",
      "received":            8192,
      "starttime":           "2009-11-10T23:00:00Z", // RFC 3339 time
      "totaldatatransfered": 10031
    }
  ]
}
```

lists all files in the download queue.

### JSON Response

`"destination": "/home/users/alice",`
// Local path that the file will be downloaded to.

`"destinationtype": "file",`
// What type of destination was used. Can be "file", indicating a download to disk, can be "buffer", indicating a download to memory, and can be "http stream", indicating that the download was streamed through the http API.

`"length": 8192, // bytes`
// Length of the download. If the download was a partial download, this will indicate the length of the partial download, and not the length of the full file.

`"offset": 0,`
// Offset within the file of the download. For full file downloads, the offset will be '0'. For partial downloads, the offset may be anywhere within the file. offset+length will never exceed the full file size.

`"siapath": "foo/bar.txt",`
// Siapath given to the file when it was uploaded.

`"completed": true,`
// Whether or not the download has completed. Will be false initially, and set to true immediately as the download has been fully written out to the file, to the http stream, or to the in-memory buffer. Completed will also be set to true if there is an error that causes the download to fail.

`"endtime": "2009-11-10T23:00:00Z", // RFC 3339 time`
// Time at which the download completed. Will be zero if the download has not yet completed.

`"error": ""`
// Error encountered while downloading. If there was no error (yet), it will be the empty string.

`"received": 4096, // bytes`
// Number of bytes downloaded thus far. Will only be updated as segments of the file complete fully. This typically has a resolution of tens of megabytes.

`"starttime": "2009-11-10T23:00:00Z", // RFC 3339 time`
// Time at which the download was initiated.

`"totaldatatransfered": 10321,`
// The total amount of data transfered when downloading the file. This will eventually include data transferred during contract + payment negotiation, as well as data from failed piece downloads.

## /renter/downloads/clear [POST]

> Timestamp Parameters
```go
before  // Optional - unix timestamp found in the download history
after   // Optional - unix timestamp found in the download history
```

Clears the download history of the renter for a range of unix time stamps.  Both parameters are optional, if no parameters are provided, the entire download history will be cleared.  To clear a single download, provide the timestamp for the download as both parameters.  Providing only the before parameter will clear all downloads older than the timestamp. Conversely, providing only the after parameter will clear all downloads newer than the timestamp.

### JSON Response

standard success or error response. See [standard responses](#Standard-Responses).

## /renter/files [GET]

```go
{
  "files": [
    {
      "siapath":        "foo/bar.txt",
      "localpath":      "/home/foo/bar.txt",
      "filesize":       8192, // bytes
      "available":      true,
      "renewing":       true,
      "redundancy":     5,
      "bytesuploaded":  209715200, // total bytes uploaded
      "uploadprogress": 100, // percent
      "expiration":     60000
    }
  ]
}
```

lists the status of all files.

### JSON Response

`"siapath": "foo/bar.txt",`
// Path to the file in the renter on the network.

`"localpath": "/home/foo/bar.txt",`
// Path to the local file on disk.

`"filesize": 8192, // bytes`
// Size of the file in bytes.

`"available": true,`
// true if the file is available for download. Files may be available before they are completely uploaded.

`"renewing": true,`
// true if the file's contracts will be automatically renewed by the renter.

`"redundancy": 5,`
// Average redundancy of the file on the network. Redundancy is calculated by dividing the amount of data uploaded in the file's open contracts by the size of the file. Redundancy does not necessarily correspond to availability. Specifically, a redundancy >= 1 does not indicate the file is available as there could be a chunk of the file with 0 redundancy.

`"uploadedbytes": 209715200, // bytes`
// Total number of bytes successfully uploaded via current file contracts. This number includes padding and rendundancy, so a file with a size of 8192 bytes might be padded to 40 MiB and, with a redundancy of 5, encoded to 200 MiB for upload.

`"uploadprogress": 100, // percent`
// Percentage of the file uploaded, including redundancy. Uploading has completed when uploadprogress is 100. Files may be available for download before upload progress is 100.

`"expiration": 60000`
// Block height at which the file ceases availability.

## /renter/file/*siapath* [GET]

```go
{
  "file": {
    "siapath":        "foo/bar.txt",
    "localpath":      "/home/foo/bar.txt",
    "filesize":       8192, // bytes
    "available":      true,
    "renewing":       true,
    "redundancy":     5,
    "bytesuploaded":  209715200, // total bytes uploaded
    "uploadprogress": 100, // percent
    "expiration":     60000
  }
}
```

lists the status of specified file.

### JSON Response

`"siapath": "foo/bar.txt",`
// Path to the file in the renter on the network.

`"localpath": "/home/foo/bar.txt",`
// Path to the local file on disk.

`"filesize": 8192, // bytes`
// Size of the file in bytes.

`"available": true,`
// true if the file is available for download. Files may be available before they are completely uploaded.

`"renewing": true,`
// true if the file's contracts will be automatically renewed by the renter.

`"redundancy": 5,`
// Average redundancy of the file on the network. Redundancy is calculated by dividing the amount of data uploaded in the file's open contracts by the size of the file. Redundancy does not necessarily correspond to availability. Specifically, a redundancy >= 1 does not indicate the file is available as there could be a chunk of the file with 0 redundancy.

`"uploadedbytes": 209715200, // bytes`
// Total number of bytes successfully uploaded via current file contracts. This number includes padding and rendundancy, so a file with a size of 8192 bytes might be padded to 40 MiB and, with a redundancy of 5, encoded to 200 MiB for upload.

`"uploadprogress": 100, // percent`
// Percentage of the file uploaded, including redundancy. Uploading has completed when uploadprogress is 100. Files may be available for download before upload progress is 100.

`"expiration": 60000`
// Block height at which the file ceases availability.

## /renter/prices [GET]

> Query String Parameters
```go
all optional or all required

funds // hastings
hosts
period // block height
renewwindow // block height
```

```go
{
  "downloadterabyte":      "1234", // hastings
  "formcontracts":         "1234", // hastings
  "storageterabytemonth":  "1234", // hastings
  "uploadterabyte":        "1234", // hastings
  "funds":                 "1234", // hastings
  "hosts":                     24,
  "period":                  6048, // blocks
  "renewwindow":             3024  // blocks
}
```

lists the estimated prices of performing various storage and data operations. An allowance can be submitted to provide a more personalized estimate. If no allowance is submitted then the current set allowance will be used, if there is no allowance set then sane defaults will be used. Submitting an allowance is optional, but when submitting an allowance all the components of the allowance are required. The allowance used to create the estimate is returned with the estimate.

### Query String Parameters

all optional or all required

`funds // hastings`
// Number of hastings allocated for file contracts in the given period.

`hosts`
// Number of hosts that contracts should be formed with. Files cannot be uploaded to more hosts than you have contracts with, and it's generally good to form a few more contracts than you need.

`period // block height`
// Duration of contracts formed. Must be nonzero.

renewwindow // block height`
// Renew window specifies how many blocks before the expiration of the current contracts the renter will wait before renewing the contracts. A smaller renew window means that Sia must be run more frequently, but also means fewer total transaction fees. Storage spending is not affected by the renew window size.

### JSON Response

`"downloadterabyte": "1234", // hastings`
// The estimated cost of downloading one terabyte of data from the network.

`"formcontracts": "1234", // hastings`
// The estimated cost of forming a set of contracts on the network. This cost also applies to the estimated cost of renewing the renter's set of contracts.

`"storageterabytemonth": "1234", // hastings`
// The estimated cost of storing one terabyte of data on the network for a month, including accounting for redundancy.

`"uploadterabyte": "1234", // hastings`
// The estimated cost of uploading one terabyte of data to the network, including accounting for redundancy.

`"funds": "1234", // hastings`
// Amount of money allocated for contracts. Funds are spent on both storage and bandwidth.

`"hosts":24,`
// Number of hosts that contracts will be formed with.

`"period": 6048, // blocks`
// Duration of contracts formed, in number of blocks.

`"renewwindow": 3024 // blocks`
// If the current blockheight + the renew window >= the height the contract is scheduled to end, the contract is renewed automatically. Is always nonzero.

## /renter/file/*siapath* [POST]

> Path Parameters
```go
// SiaPath of the file on the network. The path must be non-empty, may not include any path traversal strings ("./", "../"), and may not begin with a forward-slash character.
*siapath
```

> Query String Parameters
```go
// If provided, this parameter changes the tracking path of a file to the  specified path. Useful if moving the file to a different location on disk.
trackingpath
```

endpoint for changing file metadata.

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /renter/delete/*siapath* [POST] START FROM HERE THURSDAY NIGHT






## /renter/prices [GET]

lists the estimated prices of performing various storage and data operations.

### JSON Response (with comments)
`
{
  "downloadterabyte":      "1234", // hastings
  "formcontracts":         "1234", // hastings
  "storageterabytemonth":  "1234", // hastings
  "uploadterabyte":        "1234"  // hastings
}
`

## /renter/delete/*siapath [POST]

deletes a renter file entry. Does not delete any downloads or original files, only the entry in the renter.

### Path Parameters (with comments)

`*siapath`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /renter/download/*siapath [GET]

downloads a file to the local filesystem. The call will block until the file has been downloaded.

### Path Parameters (with comments)

`*siapath`

### Query String Parameters (with comments)

`async
destination
httpresp
length
offset`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /renter/downloadasync/*siapath [GET]

downloads a file to the local filesystem. The call will return immediately.

### Path Parameters (with comments)

`*siapath`

### Query String Parameters (with comments)

`destination`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /renter/rename/*siapath [POST]

renames a file. Does not rename any downloads or source files, only renames the entry in the renter. An error is returned if siapath does not exist or newsiapath already exists.

### Path Parameters (with comments)

`*siapath`

### Query String Parameters (with comments)

`newsiapath`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /renter/stream/*siapath [GET]

downloads a file using http streaming. This call blocks until the data is received. The streaming endpoint also uses caching internally to prevent siad from redownloading the same chunk multiple times when only parts of a file are requested at once. This might lead to a substantial increase in ram usage and therefore it is not recommended to stream multiple files in parallel at the moment. This restriction will be removed together with the caching once partial downloads are supported in the future.

### Path Parameters (with comments)

`*siapath`

### Response

standard success with the requested data in the body or error response. See [standard responses](#Standard-Responses).

## /renter/upload/*siapath [POST]

uploads a file to the network from the local filesystem.

### Path Parameters (with comments)

`*siapath`

### Query String Parameters (with comments)

`datapieces   // int
paritypieces // int
source       // string - a filepath`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

# Transaction Pool

## /tpool/confirmed/:id [GET]

returns whether the requested transaction has been seen on the blockchain. Note, however, that the block containing the transaction may later be invalidated by a reorg.

### JSON Response
`
{
  "confirmed": true
}
`

## /tpool/fee [GET]

returns the minimum and maximum estimated fees expected by the transaction pool.

### JSON Response (with comments)
`
{
  "minimum": "1234", // hastings / byte
  "maximum": "5678"  // hastings / byte
}
`

## /tpool/raw/:id [GET]

returns the ID for the requested transaction and its raw encoded parents and transaction data.

### JSON Response (with comments)
`
{
	// id of the transaction
	"id": "124302d30a219d52f368ecd94bae1bfb922a3e45b6c32dd7fb5891b863808788",

	// raw, base64 encoded transaction data
	"transaction": "AQAAAAAAAADBM1ca/FyURfizmSukoUQ2S0GwXMit1iNSeYgrnhXOPAAAAAAAAAAAAQAAAAAAAABlZDI1NTE5AAAAAAAAAAAAIAAAAAAAAACdfzoaJ1MBY7L0fwm7O+BoQlFkkbcab5YtULa6B9aecgEAAAAAAAAAAQAAAAAAAAAMAAAAAAAAAAM7Ljyf0IA86AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAACgAAAAAAAACe0ZTbGbI4wAAAAAAAAAAAAAABAAAAAAAAAMEzVxr8XJRF+LOZK6ShRDZLQbBcyK3WI1J5iCueFc48AAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEAAAAAAAAAA+z4P1wc98IqKxykTSJxiVT+BVbWezIBnIBO1gRRlLq2x/A+jIc6G7/BA5YNJRbdnqPHrzsZvkCv4TKYd/XzwBA==",
	"parents": "AQAAAAAAAAABAAAAAAAAAJYYmFUdXXfLQ2p6EpF+tcqM9M4Pw5SLSFHdYwjMDFCjAAAAAAAAAAABAAAAAAAAAGVkMjU1MTkAAAAAAAAAAAAgAAAAAAAAAAHONvdzzjHfHBx6psAN8Z1rEVgqKPZ+K6Bsqp3FbrfjAQAAAAAAAAACAAAAAAAAAAwAAAAAAAAAAzvNDjSrme8gwAAA4w8ODnW8DxbOV/JribivvTtjJ4iHVOug0SXJc31BdSINAAAAAAAAAAPGHY4699vggx5AAAC2qBhm5vwPaBsmwAVPho/1Pd8ecce/+BGv4UimnEPzPQAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAQAAAAAAAACWGJhVHV13y0NqehKRfrXKjPTOD8OUi0hR3WMIzAxQowAAAAAAAAAAAAAAAAAAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABAAAAAAAAAABnt64wN1qxym/CfiMgOx5fg/imVIEhY+4IiiM7gwvSx8qtqKniOx50ekrGv8B+gTKDXpmm2iJibWTI9QLZHWAY=",
}
`

## /tpool/raw [POST]

submits a raw transaction to the transaction pool, broadcasting it to the transaction pool's peers.

### Query String Parameters (with comments)

`parents     string // raw base64 encoded transaction parents
transaction string // raw base64 encoded transaction`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

# Wallet

For examples and detailed descriptions of request and response parameters, refer to Wallet.md.

## /wallet [GET]

returns basic information about the wallet, such as whether the wallet is locked or unlocked.

### JSON Response (with comments)
`
{
  "encrypted":  true,
  "unlocked":   true,
  "rescanning": false,

  "confirmedsiacoinbalance":     "123456", // hastings, big int
  "unconfirmedoutgoingsiacoins": "0",      // hastings, big int
  "unconfirmedincomingsiacoins": "789",    // hastings, big int

  "siafundbalance":      "1",    // siafunds, big int
  "siacoinclaimbalance": "9001", // hastings, big int

  "dustthreshold": "1234", // hastings / byte, big int
}
`

## /wallet/033x [POST]

loads a v0.3.3.x wallet into the current wallet, harvesting all of the secret keys. All spendable addresses in the loaded wallet will become spendable from the current wallet.

### Query String Parameters (with comments)

`source
encryptionpassword`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /wallet/address [GET]

gets a new address from the wallet generated by the primary seed. An error will be returned if the wallet is locked.

### JSON Response (with comments)
`
{
  "address": "1234567890abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789ab"
}
`

## /wallet/addresses [GET]

fetches the list of addresses from the wallet. If the wallet has not been created or unlocked, no addresses will be returned. After the wallet is unlocked, this call will continue to return its addresses even after the wallet is locked again.

### JSON Response (with comments)
`
{
  "addresses": [
    "1234567890abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789ab",
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  ]
}
`

## /wallet/backup [GET]

creates a backup of the wallet settings file. Though this can easily be done manually, the settings file is often in an unknown or difficult to find location. The /wallet/backup call can spare users the trouble of needing to find their wallet file.

### Parameters (with comments)

`destination`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /wallet/init [POST]

initializes the wallet. After the wallet has been initialized once, it does not need to be initialized again, and future calls to /wallet/init will return an error. The encryption password is provided by the api call. If the password is blank, then the password will be set to the same as the seed.

### Query String Parameters (with comments)

`encryptionpassword
dictionary // Optional, default is english.
force // Optional, when set to true it will destroy an existing wallet and reinitialize a new one.`

### JSON Response (with comments)
`
{
  "primaryseed": "hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello"
}
`

## /wallet/init/seed [POST]

initializes the wallet using a preexisting seed. After the wallet has been initialized once, it does not need to be initialized again, and future calls to /wallet/init/seed will return an error. The encryption password is provided by the api call. If the password is blank, then the password will be set to the same as the seed. Note that loading a preexisting seed requires scanning the blockchain to determine how many keys have been generated from the seed. For this reason, /wallet/init/seed can only be called if the blockchain is synced.

### Query String Parameters (with comments)

`encryptionpassword
dictionary // Optional, default is english.
seed
force // Optional, when set to true it will destroy an existing wallet and reinitialize a new one.`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /wallet/seed [POST]

gives the wallet a seed to track when looking for incoming transactions. The wallet will be able to spend outputs related to addresses created by the seed. The seed is added as an auxiliary seed, and does not replace the primary seed. Only the primary seed will be used for generating new addresses.

### Query String Parameters (with comments)

`encryptionpassword
dictionary
seed`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /wallet/seeds [GET]

returns the list of seeds in use by the wallet. The primary seed is the only seed that gets used to generate new addresses. This call is unavailable when the wallet is locked.

### Query String Parameters (with comments)

`dictionary`

### JSON Response (with comments)
`
{
  "primaryseed":        "hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello",
  "addressesremaining": 2500,
  "allseeds":           [
    "hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello world hello",
    "foo bar foo bar foo bar foo bar foo bar foo bar foo bar foo bar foo bar foo bar foo bar foo bar foo bar foo bar foo",
  ]
}
`

## /wallet/siacoins [POST]

sends siacoins to an address or set of addresses. The outputs are arbitrarily selected from addresses in the wallet. If 'outputs' is supplied, 'amount' and 'destination' must be empty.

### Query String Parameters (with comments)
`
amount      // hastings
destination // address
outputs     // JSON array of {unlockhash, value} pairs`

### JSON Response (with comments)
`
{
  "transactionids": [
    "1234567890abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  ]
}
`

## /wallet/siafunds [POST]

sends siafunds to an address. The outputs are arbitrarily selected from addresses in the wallet. Any siacoins available in the siafunds being sent (as well as the siacoins available in any siafunds that end up in a refund address) will become available to the wallet as siacoins after 144 confirmations. To access all of the siacoins in the siacoin claim balance, send all of the siafunds to an address in your control (this will give you all the siacoins, while still letting you control the siafunds).

### Query String Parameters (with comments)
`
amount      // siafunds
destination // address`

### JSON Response (with comments)
`
{
  "transactionids": [
    "1234567890abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
    "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb"
  ]
}
`

## /wallet/siagkey [POST]

loads a key into the wallet that was generated by siag. Most siafunds are currently in addresses created by siag.

### Query String Parameters (with comments)
`
encryptionpassword
keyfiles`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /wallet/sweep/seed [POST]

Function: Scan the blockchain for outputs belonging to a seed and send them to an address owned by the wallet.

### Query String Parameters (with comments)
`
dictionary // Optional, default is english.
seed`

### JSON Response (with comments)
`
{
  "coins": "123456", // hastings, big int
  "funds": "1",      // siafunds, big int
}
`

## /wallet/lock [POST]

locks the wallet, wiping all secret keys. After being locked, the keys are encrypted. Queries for the seed, to send siafunds, and related queries become unavailable. Queries concerning transaction history and balance are still available.

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /wallet/transaction/:*id* [GET]

gets the transaction associated with a specific transaction id.

### Path Parameters (with comments)

`:id`

### JSON Response (with comments)
`
{
  "transaction": {
    "transaction": {
      // See types.Transaction in https://github.com/NebulousLabs/Sia/blob/master/types/transactions.go
    },
    "transactionid":         "1234567890abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
    "confirmationheight":    50000,
    "confirmationtimestamp": 1257894000,
    "inputs": [
      {
        "parentid":       "1234567890abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
        "fundtype":       "siacoin input",
        "walletaddress":  false,
        "relatedaddress": "1234567890abcdef0123456789abcdef0123456789abcdef0123456789abcdef0123456789ab",
        "value":          "1234", // hastings or siafunds, depending on fundtype, big int
      }
    ],
    "outputs": [
      {
        "id":             "1234567890abcdef0123456789abcdef0123456789abcdef0123456789abcdef",
        "fundtype":       "siacoin output",
        "maturityheight": 50000,
        "walletaddress":  false,
        "relatedaddress": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
        "value":          "1234", // hastings or siafunds, depending on fundtype, big int
      }
    ]
  }
}
`

## /wallet/transactions [GET]

returns a list of transactions related to the wallet in chronological order.

### Query String Parameters (with comments)
`
startheight // block height
endheight   // block height`

### JSON Response (with comments)
`
{
  "confirmedtransactions": [
    {
      // See the documentation for '/wallet/transaction/:id' for more information.
    }
  ],
  "unconfirmedtransactions": [
    {
      // See the documentation for '/wallet/transaction/:id' for more information.
    }
  ]
}
`

## /wallet/transactions/:addr [GET]

returns all of the transactions related to a specific address.

### Path Parameters (with comments)

`:addr`

### JSON Response (with comments)
`
{
  "transactions": [
    {
      // See the documentation for '/wallet/transaction/:id' for more information.
    }
  ]
}
`

## /wallet/unlock [POST]

unlocks the wallet. The wallet is capable of knowing whether the correct password was provided.

### Query String Parameters (with comments)

`encryptionpassword`

### Response

standard success or error response. See [standard responses](#Standard-Responses).

## /wallet/verify/address/:addr [GET]

takes the address specified by :addr and returns a JSON response indicating if the address is valid.

### JSON Response (with comments)
`
{
	"valid": true
}
`

## /wallet/changepassword [POST]

changes the wallet's encryption key.

### Query String Parameters (with comments)
`
encryptionpassword
newpassword`

### Response

standard success or error response. See [standard responses](#Standard-Responses).



















# Kittens

## Get All Kittens

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
curl "http://example.com/api/kittens"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
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

Parameter | Default | Description
--------- | ------- | -----------
include_cats | false | If set to true, the result will also include cats.
available | true | If set to false, the result will include kittens that have already been adopted.

<aside class="success">
Remember — a happy kitten is an authenticated kitten!
</aside>

## Get a Specific Kitten

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
curl "http://example.com/api/kittens/2"
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
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

Parameter | Description
--------- | -----------
ID | The ID of the kitten to retrieve

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
curl "http://example.com/api/kittens/2"
  -X DELETE
  -H "Authorization: meowmeowmeow"
```

```javascript
const kittn = require('kittn');

let api = kittn.authorize('meowmeowmeow');
let max = api.kittens.delete(2);
```

> The above command returns JSON structured like this:

```json
{
  "id": 2,
  "deleted" : ":("
}
```

This endpoint deletes a specific kitten.

### HTTP Request

`DELETE http://example.com/kittens/<ID>`

### URL Parameters

Parameter | Description
--------- | -----------
ID | The ID of the kitten to delete

