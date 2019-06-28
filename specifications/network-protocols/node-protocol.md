# Client protocol

## Overview

In this document we describe every client-facing method offered by a Themelio node, using `autonode.themelio.org` \(the DNS bootstrapping node\) as an example endpoint.

## Consensus methods

These methods are used to securely obtain the latest block hash without trusting the node, which is necessary for bootstrapping all subsequent trust. 

{% api-method method="get" host="http://autonode.themelio.org" path="/consensus" %}
{% api-method-summary %}
Latest consensus
{% endapi-method-summary %}

{% api-method-description %}
Returns the latest consensus. **TBD**
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="" type="string" required=false %}

{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```javascript
{
    "Height": 12345,
    "BlockHash": "0333ab52d8fd66ec8d4846370161119ff4b450d3a3ba5863a6b9eb18f3e9c6fd",
    "Signatures": [...]
}
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

## Block methods

Information about blocks are served under `/blocks/<height>`.

**Note**: Information about blocks is generally limited to the last 100 blocks, unless in the case of archive nodes.

{% api-method method="get" host="http://autonode.themelio.org" path="/blocks/<height>/tx/<txhash>" %}
{% api-method-summary %}
Individual transactions
{% endapi-method-summary %}

{% api-method-description %}
Returns a particular transaction within a block.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="height" type="integer" required=true %}
Block height
{% endapi-method-parameter %}

{% api-method-parameter name="txhash" type="string" required=true %}
Hexadecimal transaction hash
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text

```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=404 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text
Transaction not found
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="http://autonode.themelio.org" path="/blocks/<height>/header" %}
{% api-method-summary %}
Headers
{% endapi-method-summary %}

{% api-method-description %}
Returns the header of a block at a certain height.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="height" type="integer" required=true %}
Block height
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text
0000000000000000000000000000000000000000000000000000000000000000444f02ba27e2cb72cb3ab39e390e6f46d22ebea980a0dab6264d9350f8aafb1f444f02ba27e2cb72cb3ab39e390e6f46d22ebea980a0dab6264d9350f8aafb1f444f02ba27e2cb72cb3ab39e390e6f46d22ebea980a0dab6264d9350f8aafb1f00000000000000000000000000000000
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=404 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text
Nonexistent block height
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="http://autonode.themelio.org" path="/blocks/<height>/fullblock" %}
{% api-method-summary %}
Dumping entire block
{% endapi-method-summary %}

{% api-method-description %}
Returns all the transactions in a block, in order, concatenated together. This is guaranteed to work only for recent blocks!
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="height" type="integer" required=true %}
Block height
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text
...
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=404 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="http://autonode.themelio.org" path="/blocks/<height>/mbpt/<type>/<key>" %}
{% api-method-summary %}
Digging into MBPTs
{% endapi-method-summary %}

{% api-method-description %}
Returns an MBPT branch, starting from the root to the leaf, as an RLP array of nodes.Unlike the other methods, this one is guaranteed to work only for the **latest** block!
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="height" type="integer" required=true %}
Block height
{% endapi-method-parameter %}

{% api-method-parameter name="type" type="string" required=true %}
One of "history", "state", or "transactions"
{% endapi-method-parameter %}

{% api-method-parameter name="key" type="string" required=true %}
Hexadecimal key into the MBPT
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text
...
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=404 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text
Block too old!
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

## Methods on transactions

Methods on transactions are generally under `/transactions`. 

{% api-method method="post" host="http://autonode.themelio.org" path="/transactions/submit" %}
{% api-method-summary %}
Submitting a transaction
{% endapi-method-summary %}

{% api-method-description %}
Submits a transaction to be included as soon as possible.
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-body-parameters %}
{% api-method-parameter name="" type="object" required=true %}
Raw transaction data
{% endapi-method-parameter %}
{% endapi-method-body-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text
Transaction accepted
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=400 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text
Malformed transaction
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=402 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text
Fee too low
```
{% endapi-method-response-example %}

{% api-method-response-example httpCode=503 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text
Blockchain congested
```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

{% api-method method="get" host="http://autonode.themelio.org" path="/transactions/querycoins?height=...&conshash=..." %}
{% api-method-summary %}
Query coins
{% endapi-method-summary %}

{% api-method-description %}
Queries for coins associated with a particular constraint. Returns an RLP-encoded array of \[TxInput, TxOutputAndHeight\].
{% endapi-method-description %}

{% api-method-spec %}
{% api-method-request %}
{% api-method-path-parameters %}
{% api-method-parameter name="conshash" type="string" required=true %}
hex-encoded hash of the constraint
{% endapi-method-parameter %}

{% api-method-parameter name="height" type="number" required=true %}
block height whose state we're querying
{% endapi-method-parameter %}
{% endapi-method-path-parameters %}
{% endapi-method-request %}

{% api-method-response %}
{% api-method-response-example httpCode=200 %}
{% api-method-response-example-description %}

{% endapi-method-response-example-description %}

```text

```
{% endapi-method-response-example %}
{% endapi-method-response %}
{% endapi-method-spec %}
{% endapi-method %}

