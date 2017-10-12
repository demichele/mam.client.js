# MAM Client JS Library

This is wrapper library for the WASM/ASM.js output of the [MAM rust repository](https://github.com/iotaledger/MAM). This will enable you to interact and create MAM streams without any further software.

> **Join the Discussion**
>
> If you want to get involved in the community, need help with getting setup, have any issues related with the library or just want to discuss Blockchain, Distributed Ledgers and IoT with other people, feel free to join our Slack. [Slack](http://slack.iota.org/) You can also ask questions on our dedicated forum at: [IOTA Forum](https://forum.iota.org/).

## Installation

### Node.js

```
npm install iotaledger/mam.client.js
```

## Concepts

Flash Channels use a binary tree topology reduce the number of transactions to be attached. Usually each transfer would have to be attached to the tangle, Flash enables realtime streaming of tokens off-tangle.

#### Security Levels

Public 

- Root is equal to address and the side key is ‘999999…’

Private 

- Root is hashed address and the side key is ‘999999…’

Restricted

 - Root is hashed address and a custom side key ‘AJDKHGKAGD…’ (changing or not)



### Transfer

------

#### `prepare()`

This function checks for sufficient funds and return a transfer array that will correctly transfer the right amount of IOTA, in relation to channel stake, into the users settlement address . 

**Input**

```javascript
transfer.prepare(
  flash.settlementAddresses,
  flash.deposit,
  index,
  transfers
)
```

1. **settlementAddresses**: `array` the settlement addresses for each user
2. **deposit**: `array` the amount each user can still spend
3. **index**: `int` the index of the user used as an input
4. **transfers**: `array` the `{value, address}` destination of the output bundle (excluding remainder)

**Return**

1. `Array` - Transfer object 

------

#### `compose()`

This method takes the `transfers` object from the `transfer.prepare` function and takes the flash object then constructs the required bundle for the next state in the channel.

**Input**

```javascript
transfer.compose(
  flash.balance,
  flash.deposit,
  flash.outputs,
  multisig,
  flash.remainderAddress,
  flash.transfers,
  newTansfers,
  close
)
```

1. **balance**: `Int`  The total amount of iotas in the channel
2. **deposit**: `Array` The amount of iotas still available to each user to spend from
3. **outputs**: `String` The accrued outputs through the channel
4. **multisig**: `Object` history the leaf bundles
5. **remainderAddress**: `String` The remainder address of the Flash channel
6. **history**:`Array` Transfer history of the channel
7. **newTransfers**:`Array` transfers the array of outputs for the transfer
8. **close**:`Boolean` whether to use the minimum tree or not

**Return**

`Array` Array of constructed bundle representing the latest state of the Flash channel. These bundles do not have signatures. 

------

#### `sign()`

This takes the constructed bundles from `compose` and then generates an `array` of ordered signatures to be applied to the bundle.

**Input**

```javascript
transfer.sign(
  multisig,
  seed,
  bundles
)
```

1. **multisig**: `Object` history the leaf bundles
2. **seed**: `String` Tryte encoded seed
3. **bundles**: `Array` Array of bundles that require signatures.

**Return**

1. `Array` - An ordered array of signatures to be applied to the transfer bundle array. 

------

#### `appliedSignatures()`

This takes the bundles that have been generated by `compose` and the signatures of **one** user and then applies them to the bundle. Use this function to apply all the signatures to the bundle. 

The signatures **must** be applied in the order the address was generated. Otherwise the signatures will be invalid.

*Note: You use the output of this function to apply the next set of signatures.*

**Input**

```javascript
transfer.appliedSignatures(
  bundles, 
  signatures
)
```

1. **bundles**: `Array` Array of bundles that require signatures.
2. **signatures**: `Array` Ordered set of **one** users signatures to be applied to the proposed bundle.

**Return**

1. `Array` - An ordered array of bundles that have had the user's signatures applied.

------

#### `getDiff()`

This takes the channel history and latest bundles and runs checks to see where the changes are in the transfer.

*Note: this is already called in the `applyTransfers` function*

**Input**

```javascript
transfer.getDiff(
	root, 
  	remainder, 
  	history, 
  	bundles
)
```

1. **root**: `Array` multisig object starting at the root of the tree
2. **remainder**: `Object` multisig object of the remainder address
3. **history**:`Array` Transfer history of the channel
4. **bundles**: `Array` Array of bundles for the proposed transfer 

**Return**

1. `Array` - An array of diffs per address

------

#### `applyTransfers()`

**Input**

```javascript
transfer.applyTransfers(
  flash.root,
  flash.deposit,
  flash.outputs,
  flash.remainderAddress,
  flash.transfers,
  signedBundles
)
```

1. **root**: `Object` Representation of the current state of the Flash tree
2. **deposit**: `Array` The amount of iotas still available to each user to spend from
3. **outputs**: `String` The accrued outputs through the channel
4. **remainderAddress**: `String` The remainder address of the Flash channel
5. **history**:`Array` Transfer history of the channel
6. **signedBundles**:`Array` Signed bundle

**Return**

This function mutates the flash objects that are passed to it. If there is an error in applying the transfers ie. Signatures aren't valid, the function will throw an error.

------

#### `close()`

Used in place of the `prepare` function when closing the channel. This is used to generate the closing transfers to each settlement address. It does this by correctly dividing the remaining channel balance amongst the channel's users.

**Input**

```javascript
transfer.close(
  flash.settlementAddresses, 
  flash.deposit
)
```

1. **settlementAddresses**: `array` the settlement addresses for each user
2. **deposit**: `array` the amount each user can still spend

**Return**

1. `Array` - Closing transfer object 