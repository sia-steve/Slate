---
title: API Reference

language_tabs: # must be one of https://git.io/vQNgJ
  - shell
  - ruby
  - python
  - javascript

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>
  - <a href='https://github.com/lord/slate'>Documentation Powered by Slate</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Sia Storage Platform API! Sia uses semantic versioning and is backwards compatible to version v1.0.0.

API calls return either JSON or no content. Success is indicated by 2xx HTTP status codes, while errors are indicated by 4xx and 5xx HTTP status codes. If an endpoint does not specify its expected status code refer to #standard-responses.

There may be functional API calls which are not documented. These are not guaranteed to be supported beyond the current release, and should not be used in production.

**Notes:**

- Requests must set their User-Agent string to contain the substring "Sia-Agent".
- By default, siad listens on "localhost:9980". This can be changed using the --api-addr flag when running siad.
- Do not bind or expose the API to a non-loopback address unless you are aware of the possible dangers.

Example GET curl call:

`curl -A "Sia-Agent" "localhost:9980/wallet/transactions?startheight=1&endheight=250"`

Example POST curl call:

`curl -A "Sia-Agent" --data "amount=123&destination=abcd" "localhost:9980/wallet/siacoins"`

# Standard Responses

## Success
The standard response indicating the request was successfully processed is HTTP status code `204 No Content`. If the request was successfully processed and the server responded with JSON the HTTP status code is `200 OK`. Specific endpoints may specify other 2xx status codes on success.

## Error
The standard error response indicating the request failed for any reason, is a 4xx or 5xx HTTP status code with an error JSON object describing the error.

`{
    "message": String

    // There may be additional fields depending on the specific error.
}`

# Authentication

API authentication can be enabled with the `--authenticate-api` siad flag. Authentication is HTTP Basic Authentication as described in [RFC 2617](https://tools.ietf.org/html/rfc2617), however, the username is the empty string. The flag does not enforce authentication on all API endpoints. Only endpoints that expose sensitive information or modify state require authentication.

For example, if the API password is "foobar" the request header should include

`Authorization: Basic OmZvb2Jhcg==`

# Units

Unless otherwise specified, all parameters should be specified in their smallest possible unit. For example, size should always be specified in bytes and Siacoins should be specified in hastings. JSON values returned by the API will also use the smallest possible unit, unless otherwise specified.

If a numbers is returned as a string in JSON, it should be treated as an arbitrary-precision number (bignum), and it should be parsed with your language's corresponding bignum library. Currency values are the most common example where this is necessary.

# Daemon

For examples and detailed descriptions of request and response parameters, refer to Daemon.md.

## /daemon/constants [GET]

returns the set of constants in use.

JSON Response (with comments)

`{
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
}`

## /daemon/stop [GET]

cleanly shuts down the daemon. This may take a few seconds.

Response
standard success or error response. See standard responses.

## /daemon/version [GET]

returns the version of the Sia daemon currently running.

JSON Response (with comments)

`
{
"version": "1.3.2"
}
`

# Consensus

For examples and detailed descriptions of request and response parameters, refer to Consensus.md.

## /consensus [GET]

returns information about the consensus set, such as the current block height.

### JSON Response (with comments)

`
{
  "synced":       true,
  "height":       62248,
  "currentblock": "00000000000008a84884ba827bdc868a17ba9c14011de33ff763bd95779a9cf1",
  "target":       [0,0,0,0,0,0,11,48,125,79,116,89,136,74,42,27,5,14,10,31,23,53,226,238,202,219,5,204,38,32,59,165],
  "difficulty":   "1234"
}
`

## /consensus/blocks [GET]

Returns the block for a given id or height.

### Query String Parameters

One of the following parameters can be specified.

`// BlockID of the requested block.
id 

// BlockHeight of the requested block.
height`

### Response

The JSON formatted block or a standard error response.

```
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

