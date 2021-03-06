# Transaction Models

* [UTXO](#utxo)
* [Accounts](#account)

## UTXO <a name = "utxo"></a>
Each transaction has at least one input and one output. Each **input** spends the amount paid to a previous output. Each **output** then waits as an Unspent Transact Output (UTXO) until a later input spends it.

In other words, each transaction takes UTXOs generated by the previous transactions, then *spends* them by creating new UTXOs. The *balance* of any address at any time is the sum of all the UTXOs currently spendable by the address. In this model, the global state at any given time is the set of all spendable UTXOs.

In this context, wallets are higher abstractions which locate all UTXOs under control of an address and aggregate them to display an account's *balance*.

UTXOs consist of two parts:
1. the amount transferred to the output
2. the locking script (`scriptPubKey`) that specifies the conditions to be met in order to spend the output

The UTXO set is stored in the chainstate, a LevelDB database that provides persistent key-value storage. 

Types of associated outputs:
* Pay-to-PubKey (P2PK) outputs
* Pay-to-PubKeyHash (P2PKH) outputs
* Pay-to-multisig (P2MS) outputs
* Pay-to-ScriptHash (P2SH) outputs<br>
*After SegWit, the following were added*
* Pay-to-Witness-Public-Key-Hash (P2WPKH)
* Pay-to-Witness-Script-Hash (P2WSH)

A **dust output** is the output of a transaction in which the fee to redeem it is greater than 1/3 of its value. An *unprofitable output* is an output in which the amount holds less value than the fee necessary for the output to be spent, resulting in a net financial loss if an associated transaction is invoked.

Bitcoin's UTXO model grows linearly as you accumulate more inputs from previous transactions, due to the fact that all inputs require their own signature to be spent.

### Pros
**Fast, Cheap Verification of Transaction Validity**
* Since the UTXO set contains all unspent outputs, it stores all the required information to validate a new transaction without having to inspect the full blockchain.
* Transaction inputs always link to existing UTXOs. This ensures transaction can't be replayed in the same chain; in other words, sequence order/dependency is easy to authenticate. It is similarly easy to verify if a UTXO has been spent.

UTXO transactions can be processed in parallel => conducive to **Schnorr Signatures**: signatures can be aggregated across multiple inputs (UTXOs) of a transaction; instead of having to provide multiple signatures for *each* input, a transaction with Schnorr allows for a *single* transaction on ALL inputs.

### Cons
* the UTXO model is stateless => more difficult to build general purpose state channels and other sorts of smart contract applications on top of the UTXO Model

## Account <a name = "account"></a>
> more statefulness

A global state stores a list of accounts with balances, code, and internal storage. A transaction is valid if the sending account has enough balance to pay for it, in which case the sending account is debited and the receiving account is credited with the value. If the receiving account has associated code, the code runs, and internal storage may also be changed accordingly, or the code may even create additional messages to other accounts which lead to further debits and credits.

### Pros
* stateful => enables general purpose smart contracts
    * facilitates state channel constructions

* constant light client reference: light clients can at any point access all data related to an account by scanning down the state tree in a specific direction.

### Cons
* not easily compatible with Schnorr Signatures (signature aggregation is nontrivial in this context)

### References
* [UTXO vs Account](https://medium.com/nervosnetwork/my-comparison-between-the-utxo-and-account-model-821eb46691b2) -- by Nervos Network, October 2018
* [Bitcoin Developer Guide: Transactions](https://bitcoin.org/en/developer-guide#transactions)
* [Thoughts on UTXO by V](https://medium.com/@ConsenSys/thoughts-on-utxo-by-vitalik-buterin-2bb782c67e53)
* [Ethereum Design Rationale: Accounts and Not UTXOs](https://github.com/ethereum/wiki/wiki/Design-Rationale#accounts-and-not-utxos)