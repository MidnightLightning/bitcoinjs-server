## Proposed key/value schema

    block~[blockHash]~prevHash => [blockHash]
    block~[blockHash]~merkleRoot => [hash]
    block~[blockHash]~timestamp => [INT]
    block~[blockHash]~target => [INT]
    block~[blockHash]~nonce => [INT]
    block~[blockHash]~version => [INT]
    block~[blockHash]~height => [INT]
    block~[blockHash]~txs => [txHash],[txHash],[txHash] ...
    
    txn~[txHash]~block => [blockHash]
    txn~[txHash]~version => [INT]
    txn~[txHash]~in => [count(in) INT]
    txn~[txHash]~out => [count(out) INT]
    txn~[txHash]~lockTime => [INT]
    
    txn~[txHash]~in~[index]~spends => [txHash]~[outIndex]
    txn~[txHash]~in~[index]~signature => [BIN string]
    txn~[txHash]~in~[index]~sequence => [INT]
    
    txn~[txHash]~out~[index]~value => [INT]
    txn~[txHash]~out~[index]~script => [BIN]
    txn~[txHash]~out~[index]~address => [base58 string]
    txn~[txHash]~out~[index]~spent => [txHash]~[inIndex]
    
    address~[base58 string]~first => [txHash]~[outIndex]
    address~[base58 string]~pubKey => [BIN string]
    address~[base58 string]~utxo~[txHash]~[outIndex] => [value INT]
    
    height~[chainHeight] => [blockHash],[blockHash],[blockHash] ...
    
    meta~chainHeight => [INT]

* `block~[blockHash]~target` is a relabeling of the common field `bits` to something that makes more sense.
* `block~[blockHash]~height` saves the height of the block for easy access. Even if a reorganization happens, the overall height of a block shouldn't change.
* `txn~[txHash]~out~[index]~address` for a transaction with a standard script, converts the address the funds are being sent to to a standard, base58 representation.
* `txn~[txHash]~out~[index]~spent` is not set initially; if that key does not exist for a given `[txHash]~[outIndex]`, it is an unspent output.
* `address~[base58 string]~first` sets the first time that address appears in the blockchain.
* `address~[base58 string]~pubKey` is not set initially; if that address spends any output, it reveals what its public key is, and then this key can be set.
* `address~[base58 string]~utxo~[txHash]~[outIndex]` keeps track of the current balance of a given address. When a transaction is added, a `utxo` key is set for each output, and for each input, the corresponding `utxo` key is deleted. Grabbing the range of keys starting with `address~[base58 string]~utxo~` you can find the value of the address by adding up the data stored in each key.
* `height~[chainHeight]` stores an array of blocks that are at that height. The first block in the list is on the "main" fork, and subsequent blocks are on secondary forks.

To properly sort by key, we can't just use integers in base-10 notation, since the numbers 1-20 would sort like:

    height~1
    height~10
    height~11
    height~12
    height~13
    height~14
    height~15
    height~16
    height~17
    height~18
    height~19
    height~2
    height~20
    height~3
    height~4
    height~5
    height~6
    height~7
    height~8
    height~9

So, we have to zero-pad on the front out to a given size. To reduce the number of characters used, the number can be expressed as hex, which still sorts alphabetically, since ASCII letters are after ASCII numbers. But Base-64 encoding wouldn't work well, since it encodes low numbers as capital letters, which come after the numbers in ASCII order. For the most compact use of space, without resorting to full binary strings which are sometimes hard to print, use the ASCII characters from 0x21 to 0x7e (94 characters) into a "base-94" representation. This would require big integer division to calculate, the same as Bitcoin's base-58 encoding. Therefore, `"!` (10) would be 94 (decimal), `"!!` (100) would be 8,836 (decimal), `"!!!` (1000) would be 830,584 (decimal), which is more than 3 times the current blockchain size.

For representing block and transaction hashes, those are SHA256 hashes, so are 256 bits, so could be up to 2^256 in size. 94^39 just barely doesn't reach that, so 40 characters would be needed to represent a 256-bit number (compared to 64 hex characters).

## Too Many Open Files
One of the issues with the existing LevelDB implementation is some users experience hard faults from the filesystem for opening too many files at once. To solve that, if the key schema were designed such that commonly-accessed attributes were next to each other, then only one file would have to be opened to retrieve them. In that case, it might be better to have the transactions with their blocks?

So instead of `txn~[txHash]~blah`, it could be `block~[blockHash]~txn~[txHash]~blah`. But each of those hashes would have to be at least 40 characters long, so that added 40 bytes to each transaction key. And you need to know the block the transaction is a part of, in order to look it up (so you'd also need `txn~[txHash] => block~[blockHash]`).

Instead, you could do `txn~[txHash] => block~[blockHash]~txn~[index]`, and then `block~[blockHash]~txn~[index]~blah`. That replaces the `txHash` with an index value in the block, which is much shorter. Then if you know the hash of a transaction and want to look it up, look up `txn~[txHash]` to find out which block it's part of, and then fetch the data.

Either way makes transaction lookups take two different lookups if you don't know the block/index the transaction is a part of, which is a worse solution if that's the primary way transactions are found.


* [DailyJS](http://dailyjs.com/2013/05/03/leveldb-and-node-2/) post including namespacing advice.
