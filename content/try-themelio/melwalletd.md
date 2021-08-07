---
title: "melwalletd reference"
weight: 3
draft: false
# search related keywords
keywords: [""]
---

As with other UTXO-based blockchains like Bitcoin that lacks an "account" abstraction, Themelio requires somewhat involved logic for _wallets_ --- software that manages on-chain assets and provides an interface roughly similar to a bank account.

In Themelio, the "canonical" wallet software is **melwalletd**, a headless program that internally manages wallets and exposes a local REST API for operations on the wallets. Although you can use it directly as a Themelio wallet, melwalletd is intended more as a backend to wallet apps, such as **melwallet-cli**, a CLI wallet that this reference will briefly touch.

---

## Starting melwalletd

After installing melwalletd, start it by giving it a directory in which wallets are stored:

```shell
$ melwalletd --wallet-dir ~/.themelio-wallets
May 18 16:20:43.583  INFO melwalletd: opened wallet directory: []
```

If the directory doesn't exist it will be created. By default, melwalletd will start listening on `localhost:11773`.

---

## Managing wallets

### Creating a wallet

**Endpoint**
`PUT /wallets/[wallet name]`

**Body fields**

- `testnet`: whether or not to make a testnet wallet
- `pwd`: optional field; password with which to encrypt the private key. **Warning**: if not given, private key will be stored in cleartext!

**Response**

- Nothing

**Example**

```shell
$ curl -s 'localhost:11773/wallets/alice' -X PUT --data '{"testnet": true}'
```

### Listing all wallets

**Endpoint**
`GET /wallets`

**Response**

- Hashtable mapping strings to **wallet summaries** with fields:
  - `total_micromel`: total µMEL balance of wallet
  - `network`: 1 for testnet, 255 for mainnet
  - `address`: address-encoded covenant hash
  - `locked`: boolean saying whether or not the wallet is locked.

**Example**

```shell
$ curl -s localhost:11773/wallets | jq
{
  "alice": {
    "total_micromel": 0,
    "network": 1,
    "address": "t607gqktd3njqewnjcvzxv2m4ta6epbcv1sdjkp0qkmztaq3wxn350",
    "locked": true
  },
  "labooyah": {
    "total_micromel": 0,
    "network": 255,
    "address": "t1jhtj4ex1n069xr8w6mbkgrt25jgzw0pam1a25redg9ykpsykbq70",
    "locked": true
  },
  "testnet": {
    "total_micromel": 17322999920,
    "network": 1,
    "address": "t6zf5m662ge2hwax4hcs5kzqmr1a5214fa9sj2rbtassw04n6jffr0",
    "locked": true
  }
}
```

### Dumping a wallet

**Endpoint**
`GET /wallets/[wallet name]`

**Response**

- `summary`: a **wallet summary**
  - `total_micromel`: total µMEL balance of wallet
  - `network`: 1 for testnet, 255 for mainnet
  - `address`: address-encoded covenant hash
  - `locked`: whether or not the wallet is locked
- `full`: a **full wallet object** with fields
  - `unspent_coins`: array of unspent coins, each of which is an array with two elements: a CoinID and a CoinDataHeight.
  - `spent_coins`: previous members of `unspent_coins` that are now spent
  - `tx_in_progress`: hashtable mapping the hash of all in-progress transactions to the actual body of the transaction
  - `tx_confirmed`: hashtable mapping the hash of all confirmed transaction to a two-element array containing both the transaction and the height at which it was confirmed
  - `my_covenant`: the wallet's MelVM covenant, in hexadecimal
  - `network`: 1 for testnet, 255 for mainnet

**Example**

```shell
$ curl -s localhost:11773/wallets/alice | jq
{
  "summary": {
    "total_micromel": 0,
    "network": 1,
    "address": "t607gqktd3njqewnjcvzxv2m4ta6epbcv1sdjkp0qkmztaq3wxn350",
    "locked": true
  },
  "full": {
    "unspent_coins": [],
    "spent_coins": [],
    "tx_in_progress": {
      "86588da7863b39152105e4f78c04e07a5d3f3ebf61d799f95293372dabdb06a1": {
        "kind": 255,
        "inputs": [],
        "outputs": [
          {
            "covhash": "t607gqktd3njqewnjcvzxv2m4ta6epbcv1sdjkp0qkmztaq3wxn350",
            "value": 1001000000,
            "denom": "6d",
            "additional_data": ""
          }
        ],
        "fee": 1000000,
        "scripts": [],
        "data": "3516f96885d6e49f3447dd9971cfc9f92e17198394ee402e3ae00f84c6d7fbe6",
        "sigs": []
      }
    },
    "tx_confirmed": {},
    "my_covenant": "420009f100000000000000000000000000000000000000000000000000000000000000064200005050f020e8fc9285b385abec4e955b79e3ed5fb58bf9dd6b60c86b5f2a4238e1194b476e420001320020",
    "network": 1
  }
}
```

---

## Using a single wallet

### Unlocking a wallet

**Endpoint**
`POST /wallets/[name]/unlock`

**Body fields**

- `password`: password

**Response**

None

### Locking a wallet

**Endpoint**
`POST /wallets/[name]/lock`

**Body fields**

None

**Response**

None

### Sending a faucet transaction

**Note**: obviously, this only works with _testnet_ wallets!

**Endpoint**
`POST /wallets/[name]/send-faucet`

**Body fields**

None. This verb sends a faucet transaction that adds a fixed sum of 1000 MEL to the wallet.

**Response**

Quoted hexadecimal transaction hash of the transaction being sent.

**Example**

```shell
$ curl -s localhost:11773/wallets/alice/send-faucet -X POST
"86588da7863b39152105e4f78c04e07a5d3f3ebf61d799f95293372dabdb06a1"
```

### Listing all transactions

**Endpoint**
`GET /wallets/[name]/transactions/`

**Response**

A JSON object with fields:

- `tx_in_progress`: Hashtable mapping transaction hash to a raw transaction object in JSON format.
- `tx_confirmed`: Hashtable mapping transaction hash to tuple pair (transaction object, confirmed height).

**Example**

```shell
$ curl -s localhost:11773/wallets/bar/transactions | jq                                                                      ~
{
  "tx_in_progress": {
    "f72416b72ae8a0e00e80f571cb0905bfdb7f1c3bba923b5008cac27e3ae2f99b": {
      "kind": 0,
      "inputs": [
        {
          "txhash": "042816e3c4e5cf5fe140189ca1d2504852fd9721ca44f7e91e21521bc36bc830",
          "index": 1
        }
      ],
      "outputs": [
        {
          "covhash": "t6zf5m662ge2hwax4hcs5kzqmr1a5214fa9sj2rbtassw04n6jffr0",
          "value": 10000,
          "denom": "6d",
          "additional_data": ""
        },
        {
          "covhash": "t24k6kcb5epsbw7c1vx2aq3cpkshk398bbw9gm88qc21d9szcs54tg",
          "value": 1000988954,
          "denom": "6d",
          "additional_data": ""
        }
      ],
      "fee": 23,
      "scripts": [
        "420009f100000000000000000000000000000000000000000000000000000000000000064200005050f02044406b661affad63580f1b8d978cc8e06bb7debf2a95b7dbd70a0707e45dd87e420001320020"
      ],
      "data": "",
      "sigs": [
        "ba4846bb1db91cf767efaf6c965cccfbd0b9778ff215caa61dffe6efccde7c1ef53636fc1ce65d27bbd8d7b48fcd8bfae70c4c1beee3d50df8f8dc96ac320d0e"
      ]
    }
  },
  "tx_confirmed": {
    "042816e3c4e5cf5fe140189ca1d2504852fd9721ca44f7e91e21521bc36bc830": [
      {
        "kind": 0,
        "inputs": [
          {
            "txhash": "6e7d17356e9453836f1539e2e903fdb4c8ad07a6556d0a91cdfab214af476877",
            "index": 0
          }
        ],
        "outputs": [
          {
            "covhash": "t6zf5m662ge2hwax4hcs5kzqmr1a5214fa9sj2rbtassw04n6jffr0",
            "value": 1000,
            "denom": "6d",
            "additional_data": ""
          },
          {
            "covhash": "t24k6kcb5epsbw7c1vx2aq3cpkshk398bbw9gm88qc21d9szcs54tg",
            "value": 1000998977,
            "denom": "6d",
            "additional_data": ""
          }
        ],
        "fee": 23,
        "scripts": [
          "420009f100000000000000000000000000000000000000000000000000000000000000064200005050f02044406b661affad63580f1b8d978cc8e06bb7debf2a95b7dbd70a0707e45dd87e420001320020"
        ],
        "data": "",
        "sigs": [
          "f3de58f94427983b46fcf4466c78f4f70257bbce686bc2a8e7a836b507474ff40e9bb675f12ea5570a612c60c0878966575cf5d19c96fe7006a56b7540dfa108"
        ]
      },
      93622
    ],
    "6e7d17356e9453836f1539e2e903fdb4c8ad07a6556d0a91cdfab214af476877": [
      {
        "kind": 255,
        "inputs": [],
        "outputs": [
          {
            "covhash": "t24k6kcb5epsbw7c1vx2aq3cpkshk398bbw9gm88qc21d9szcs54tg",
            "value": 1001000000,
            "denom": "6d",
            "additional_data": ""
          }
        ],
        "fee": 1000000,
        "scripts": [],
        "data": "873340b25a7b250871e009ec995ce35a80d43214d3816f258c4d4aee20df03c1",
        "sigs": []
      },
      93619
    ]
  }
}
```

### Checking on a transaction

**Endpoint**
`GET /wallets/[name]/transactions/[txhash]`

**Response**

A JSON object with fields:

- `raw`: the actual transaction in JSON format
- `confirmed_height`: `null` if not confirmed, otherwise the height at which the transaction was confirmed.
- `outputs`: an array of JSON objects like:
  - `coin_data`: a CoinData object
  - `is_change`: is this a change output that goes to myself?
  - `coin_id`: a string-represented CoinID (txhash-index)

**Example**

```shell
$ curl -s localhost:11773/wallets/alice/transactions/
86588da7863b39152105e4f78c04e07a5d3f3ebf61d799f95293372dabdb06a1 | jq
{
  "raw": {
    "kind": 255,
    "inputs": [],
    "outputs": [
      {
        "covhash": "t607gqktd3njqewnjcvzxv2m4ta6epbcv1sdjkp0qkmztaq3wxn350",
        "value": 1001000000,
        "denom": "6d",
        "additional_data": ""
      }
    ],
    "fee": 1000000,
    "scripts": [],
    "data": "3516f96885d6e49f3447dd9971cfc9f92e17198394ee402e3ae00f84c6d7fbe6",
    "sigs": []
  },
  "confirmed_height": 93477,
  "outputs": [
    {
      "coin_data": {
        "covhash": "t607gqktd3njqewnjcvzxv2m4ta6epbcv1sdjkp0qkmztaq3wxn350",
        "value": 1001000000,
        "denom": "6d",
        "additional_data": ""
      },
      "is_change": true,
      "coin_id": "86588da7863b39152105e4f78c04e07a5d3f3ebf61d799f95293372dabdb06a1-0"
    }
  ]
}
```

### Claiming a coin

**Note**: This is used to _add a coin_ to the wallet. When receiving payments, by default melwalletd will not scan the blockchain for all the coins that the wallet owns. Instead, when somebody sends you money, they should give you a CoinID that you then add to your wallet to claim the money.

**Endpoint**
`PUT /wallets/[name]/coins/[coinid]`

**Response**

- _200 OK_ if the coin exists
- _404 Not Found_ if the coin doesn't exist (or is already spent)

**Example**

```shell
$ curl -s localhost:11773/wallets/alice/coins/1b7446a9a5fdbfbaf817a90f989529acc86507899ed7f1792e3c1b93349c4ad0-1 -X PUT
```

### Preparing a transaction

**Note**: This _prepares_ a transaction to be sent, creating a filled-in, valid transaction, but _without_ changing the wallet state. The user is free to just forget about the response to this API call; if the user actually wants to send the transaction, the `send-tx` call must be used.

**Endpoint**
`POST /wallets/[name]/prepare-tx`

**Body fields**

- `inputs`: _optional_ an array of `CoinID`s, representing inputs that must be spent by this transaction. This is useful for building covenant chains and such.
- `outputs`: an array of `CoinData`s, representing the desired outputs of the transaction. Any change outputs that are added are guaranteed to be added after these outputs.
- `signing_key`: an ed25519 signing key that corresponds to the wallet's covenant.
- `kind`: _optional_ TxKind of the transaction (defaults to Normal)
- `data`: _optional_ additional data of the transaction (defaults to empty)
- `nobalance`: _optional_ vector of `Denom`s on which balancing --- checking that exactly the same number of coins are produce by a transaction as those consumed by it --- should not be done. Normally, this is used to exempt nomDOSC balance from being checked when preparing a DOSC-minting transaction.

**Response**

- JSON-encoded transaction

**Example**

```shell
$ curl -s localhost:11773/wallets/alice/prepare-tx -X POST --data '{
    "outputs": [
        {
            "covhash": "57df1dd5b067f77a177127cbe4d69aa10fe8fcac3b2f9718cb8263d5a6216ab0",
            "value": 1000,
            "denom": "6d",
            "additional_data": ""
        }
    ],
    "signing_key": "4239e79eab9b39c49de990363197a64e1a54f0f9a0d12a936e85e69ea7fb05b006425dfe7967003e2a5362e36231730f2faaa6068979afc52784f916466e05b6"
}'

{
    "kind": 0,
    "inputs": [
        {
            "txhash": "4950f3af9da569e1a99a7e738026a581d6e96caaf02d94b02efcd645540a2d2f",
            "index": 0
        }
    ],
    "outputs": [
        {
            "covhash": "57df1dd5b067f77a177127cbe4d69aa10fe8fcac3b2f9718cb8263d5a6216ab0",
            "value": 1000,
            "denom": "6d",
            "additional_data": ""
        },
        {
            "covhash": "41d04b65a010aaa3404e1a109c53e3190a393d7ff920fa5644969a879547d0aa",
            "value": 1000998977,
            "denom": "6d",
            "additional_data": ""
        }
    ],
    "fee": 23,
    "scripts": [
        "420009f100000000000000000000000000000000000000000000000000000000000000064200005050f02006425dfe7967003e2a5362e36231730f2faaa6068979afc52784f916466e05b6420001320020"
    ],
    "data": "",
    "sigs": [
        "df3cc2795d3c9576b85c4c960501af3e44bbc490c9dc51c8422f18a3a0fa1150c2f745440206748c8205971ba0cff32b87c9797a4d1098ce3bd677db62e8560c"
    ]
}
```

### Sending a transaction

**Note**: This endpoint will reject any transactions that are malformed or don't belong to the wallet.

**Endpoint**
`POST /wallets/[name]/send-tx`

**Body**

- JSON-encoded transaction

**Response**

- Quoted transaction hash

---

## Using melwallet-cli

`melwallet-cli` is a simple, easy-to-use CLI frontend to `melwalletd`.

Our ["my first transaction"]({{< ref my-first-tx.md>}}) guide has a brief introduction to `melwallet-cli`. More details can be seen in the in-program help `melwallet-cli -h`.
