---
title: "melwalletd reference"
weight: 1
draft: false
# search related keywords
keywords: [""]
---

As with other UTXO-based blockchains like Bitcoin that lacks an "account" abstraction, Themelio requires somewhat involved logic for _wallets_ --- software that manages on-chain assets and provides an interface roughly similar to a bank account.

In Themelio, the "canonical" wallet software is **melwalletd**, a headless program that internally manages wallets and exposes a local REST API for operations on the wallets. Although you can use it directly as a Themelio wallet, melwalletd is intended more as a backend to GUI wallets, such as **melwallet-cli**, a CLI wallet that this reference will briefly touch.

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

**Response**

- Quoted string representing private key of newly created wallet

**Example**

```shell
$ curl -s 'localhost:11773/wallets/alice' -X PUT --data '{"testnet": true}'
"4239e79eab9b39c49de990363197a64e1a54f0f9a0d12a936e85e69ea7fb05b006425dfe7967003e2a5362e36231730f2faaa6068979afc52784f916466e05b6"
```

### Listing all wallets

**Endpoint**
`GET /wallets`

**Response**

- Hashtable mapping strings to **wallet summaries** with fields:
  - `total_micromel`: total µMEL balance of wallet
  - `network`: 1 for testnet, 255 for mainnet
  - `address`: address-encoded covenant hash

**Example**

```shell
$ curl -s localhost:11773/wallets | jq
{
  "alice": {
    "total_micromel": 2002000000,
    "network": 1,
    "address": "t18784psd022na6g2e3889rmz33453jfbzz4gfmnj4jtd8f5a7t2n0"
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
    "total_micromel": 1001000000,
    "network": 1,
    "address": "t18784psd022na6g2e3889rmz33453jfbzz4gfmnj4jtd8f5a7t2n0"
  },
  "full": {
    "unspent_coins": [
      [
        {
          "txhash": "f8294d0d2c794a889eae5b1fce62e954615f13e0eb79b1fc021364f84660cb25",
          "index": 0
        },
        {
          "coin_data": {
            "covhash": "41d04b65a010aaa3404e1a109c53e3190a393d7ff920fa5644969a879547d0aa",
            "value": 1001000000,
            "denom": "6d",
            "additional_data": ""
          },
          "height": 53947
        }
      ]
    ],
    "spent_coins": [],
    "tx_in_progress": {
      "4950f3af9da569e1a99a7e738026a581d6e96caaf02d94b02efcd645540a2d2f": {
        "kind": 255,
        "inputs": [],
        "outputs": [
          {
            "covhash": "41d04b65a010aaa3404e1a109c53e3190a393d7ff920fa5644969a879547d0aa",
            "value": 1001000000,
            "denom": "6d",
            "additional_data": ""
          }
        ],
        "fee": 1000000,
        "scripts": [],
        "data": "95de2e898daf8df1e9a2f4e74ff0e8d0ae92591cb887fe68f33d9a9d02911e21",
        "sigs": []
      }
    },
    "tx_confirmed": {
      "f8294d0d2c794a889eae5b1fce62e954615f13e0eb79b1fc021364f84660cb25": [
        {
          "kind": 255,
          "inputs": [],
          "outputs": [
            {
              "covhash": "41d04b65a010aaa3404e1a109c53e3190a393d7ff920fa5644969a879547d0aa",
              "value": 1001000000,
              "denom": "6d",
              "additional_data": ""
            }
          ],
          "fee": 1000000,
          "scripts": [],
          "data": "2b03d04c322d4a51c75cdbc94399b205d0460dc2868b178e42b822a0cc15e125",
          "sigs": []
        },
        53947
      ]
    },
    "my_covenant": "420009f100000000000000000000000000000000000000000000000000000000000000064200005050f02006425dfe7967003e2a5362e36231730f2faaa6068979afc52784f916466e05b6420001320020",
    "network": 1
  }
}

```

---

## Using a single wallet

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
"6c5edfe4dec3d173ad4a20a86b8ece6205b872afbb551744916362d519588cef"
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
6c5edfe4dec3d173ad4a20a86b8ece6205b872afbb551744916362d519588cef | jq
{
  "raw": {
    "kind": 255,
    "inputs": [],
    "outputs": [
      {
        "covhash": "41d04b65a010aaa3404e1a109c53e3190a393d7ff920fa5644969a879547d0aa",
        "value": 1001000000,
        "denom": "6d",
        "additional_data": ""
      }
    ],
    "fee": 1000000,
    "scripts": [],
    "data": "9e54625e99642b19453869cbd81c7c75c48dd71ca7e0f029663f013654147cfd",
    "sigs": []
  },
  "confirmed_height": 53977,
  "outputs": [
    {
      "coin_data": {
        "covhash": "41d04b65a010aaa3404e1a109c53e3190a393d7ff920fa5644969a879547d0aa",
        "value": 1001000000,
        "denom": "6d",
        "additional_data": ""
      },
      "is_change": true,
      "coin_id": "6c5edfe4dec3d173ad4a20a86b8ece6205b872afbb551744916362d519588cef-0"
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

- `outputs`: an array of CoinDatas, representing the desired outputs of the transaction. Any change outputs that are added are guaranteed to be added after these outputs.
- `signing_key`: an ed25519 signing key that corresponds to the wallet's covenant.
- `kind`: _optional_ TxKind of the transaction (defaults to Normal)
- `data`: _optional_ additional data of the transaction (defaults to empty)

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
