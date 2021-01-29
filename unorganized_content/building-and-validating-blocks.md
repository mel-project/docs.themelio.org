# Building and validating blocks

## Why this document?

One rather particular detail of Themelio blocks is that **transactions within a block are unordered but may depend on each other**. It's therefore not very obvious how to validate blocks.

This page documents an elegant way of both validating and incrementally building Themelio blocks. It's not technically part of the specification as there are other ways of validating blocks that give equivalent results, but it's still very helpful to have a reference guide to the way `go-themelio` validates blocks.

## Validation

There are three steps to validate a block.

**Creating effects**. We create the _effects_ of all the **non-coinbase** transactions first, without validating anything. We loop across all transactions in the block, adding their UTXOs to a _proposed new state tree_. No checking is done at all except for fees.

**Validating transactions**. We then do a _parallel_ loop over the transactions, marking any UTXOs we spend into a concurrent map and aborting if we attempt to add into the map something we already have added or is not in the proposed new state tree. During this step, every transaction is validated according to their kind-specific rules \(stuff like money balancing, fees, etc\).

**Seal the block**. We validate the coinbase transaction and build a new data structure to represent the block.

### Why does this even work?

Why does it even work to create effects before validating their causes? This is possible because each transaction refers to its "causes" by hash. So we can't have self-referential transactions or anything like that.



