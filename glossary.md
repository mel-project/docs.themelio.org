# Glossary

**State elements** constitute the components of the **world state**

* **Blocks** are defined by their constituent **transactions**, optionally with a consensus proof
* **Consensus proofs** are attached to blocks, turning them into **confirmed** blocks.
* The world state contains **history** and **coin state**

Each **transaction** has:

* **Transaction inputs**, each of which spends a **coin** \(mapping in the coin state\) by **unlocking** its covenant
* **Transaction outputs**, each of which has a **value**, a **denomination**, and a MelVM **covenant**
* **Attached data**

Currencies:

* The base currency is "**Mel**" and the unit is "**mel**". This is like "Bitcoin" vs "bitcoin".
  * _We accept Mel, Bitcoin, and other cryptocurrencies._
  * _The burger cost 4 mels_ 
* Decimal SI prefixes can be used.
  * _A burger and a side of fries costs 6 mels and 33 centimels._
  * _The smallest unit of money on Themelio is 1 micromel._
* **Sym** and **syms** is similar.
* Actual usage that emerges will probably be different and will be "correct".

Algorithms:

* The PoS system as a whole is **Synkletos**.
* **Symphonia** is an internal name referring to a particular implementation of non-pipelined HotStuff. It's not really an "official" part of Themelio's definition.
* **Melmint** is the cryptocurrency issuance mechanism. It's not a currency.
  * _In the last week Melmint minted a record 3 megamels onto the Themelio blockchain, half of which were locked up and reissued as ERC-20 tokens._

\_\_



