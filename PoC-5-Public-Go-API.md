The Go implementation has, besides the regular API implementation, a public API. The public API gives you the ability to rapidly develop external, stand-alone applications. The following public objects and methods can be found:

**Please note**: Addresses and hashes are represented by a hex string. Arguments which take a address or a hash also take a hex string instead of raw bytes.

* `GetBlock(hexHash)` Returns the block, or nil if nothing was found.
* `GetKey()` Returns your current seckey.
* `GetStateObject(hexAddress)` Returns the state object, or nil if nothing found.
* `GetStorage(hexAddress, hexStorageAddress)` Returns the value of the storage found at the address.
* `GetTxCount()` Returns the balance of the current key.
* `IsContract(hexAddress)` Returns a boolean indicating whether this is a contract or not.
* `Transact(seckey, to, value, gas, gasPrice, data)` Creates a new transaction signed by the specified `seckey`. Arguments are all strings.
* `Create(secKey, value, gas, gasPrice, init, body)` Creates a new contract signed by the specified `seckey`. Argumens are all strings.

#### PBlock

* `Number` canonical number in the chain.
* `Hash` hash of the block represented by a hex string.

#### PTx

* `Value` value of the transaction.
* `Hash` hash of the transaction.
* `Address` address the transaction was send to.
* `Contract` indicates whether the transaction was a contract creation transaction.

#### PKey

* `Address` the address belonging to this private key.
* `PublicKey`
* `PrivateKey`

#### PReceipt

A receipt is being returned by `Transact` and `Create`.

* `CreatedContract` indicates whether the transaction created a contract or not.
* `Address` if the transaction created a contract, this is the address of the created contract.
* `Hash` hash of the transaction.
* `Sender` the sender of the transaction.

#### PStateObject

A PStateObject can either be a contract or a user's account.

* `GetStorage(hexAddress)` Get the value at the specified address. Return value defaults to "0".
* `Value()`
* `Address()` returns the address of the state object
* `Nonce()` the nonce (or tx count).
* `IsContract()`