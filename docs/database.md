# Database

minimint uses a simple key-value store as its database. In theory any such KV store with the following features can
be used:

* insert, update, delete actions
* transactions
* key prefix search

In practice we use [sled](https://docs.rs/sled/) as it is a native rust database and seems sufficiently performant.

## Server DB Layout
The Database is split into different key spaces based on prefixing that can be understood as different tables (each
"table's" content can be retrieved using prefix search). There are three general prefix ranges:

* 0x00-0x0A: consensus
* 0x10-0x1A: mint
* 0x20-0x2A: client (different db, but to be sure)
* 0x30-0x3A: wallet

The following "tables" exist:

| Name                 | Prefix | Key                                                                                             | Value                           |
|----------------------|--------|-------------------------------------------------------------------------------------------------|---------------------------------|
| Consensus Items      | 0x01   | issuance_request_id (8 bytes, 0 if no issuance involved), serialized `ConsensusItem` (variable) | none                            |
| Partial Signature    | 0x02   | issuance_request_id (8 byte), peer_id (8 byte)                                                  | serialized `PartialSigResponse` |
| Transaction Statuses | 0x03   | TransactionId (32 bytes)                                                                        | serialized `TransactionStatus`  |
| Tx Output Outcomes   | 0x04   | TransactionId (32 bytes) + output idx (8 bytes)                                                 | serialized `OutputOutcome`      |
| Used Coins           | 0x10   | coin nonce (unknown bytes, bincode magic currently)                                             | none                            |
| Blocks               | 0x30   | block hash (32 bytes)                                                                           | block height (4 bytes)          |
| Our UTXOs            | 0x31   | OutPoint (32 bytes txid + 4 bytes output)                                                       | data necessary for spending     |
| Last Block           | 0x32   | none                                                                                            | block height (4bytes)           |
| Queued PegOut        | 0x33   | mint tx id (32 bytes)                                                                           | address, amount                 |
| Unsigned transaction | 0x34   | none                                                                                            | PSBT                            |
| Pending transaction  | 0x35   | txid                                                                                            | consensus encoded tx            |

### TODO:
* consensus items per module
* transaction status = CIs of minimint module (add tx data to status)

## Client DB Layout

| Name      | Prefix | Key                                | Value                        |
|-----------|--------|------------------------------------|------------------------------|
| Coins     | 0x20   | amount (8 bytes), nonce (32 bytes) | serialized `SpendableCoin`   |
| Issuances | 0x21   | issuance_id (32 bytes)             | serialized `IssuanceRequest` |
| Peg-Ins   | 0x22   | secret contract key (32 bytes)     | none                         |