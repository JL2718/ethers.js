-----

Documentation: [html](https://docs.ethers.io/)

-----

Signers
=======

Signer
------

#### *signer* . **connect**( provider ) => *[Signer](/v5/api/signer/#Signer)*

Sub-classes **must** implement this, however they may simply throw an error if changing providers is not supported.


#### *signer* . **getAddress**( ) => *Promise< string< [Address](/v5/api/utils/address/#address) > >*

Returns a Promise that resolves to the account address.

This is a Promise so that a **Signer** can be designed around an asynchronous source, such as hardware wallets.

Sub-classes **must** implement this.


#### *Signer* . **isSigner**( object ) => *boolean*

Returns true if and only if *object* is a **Signer**.


### Blockchain Methods

#### *signer* . **getBalance**( [ blockTag = "latest" ] ) => *Promise< [BigNumber](/v5/api/utils/bignumber/) >*

Returns the balance of this wallet at *blockTag*.


#### *signer* . **getChainId**( ) => *Promise< number >*

Returns the chain ID this wallet is connected to.


#### *signer* . **getGasPrice**( ) => *Promise< [BigNumber](/v5/api/utils/bignumber/) >*

Returns the current gas price.


#### *signer* . **getTransactionCount**( [ blockTag = "latest" ] ) => *Promise< number >*

Returns the number of transactions this account has ever sent. This is the value required to be included in transactions as the `nonce`.


#### *signer* . **call**( transactionRequest ) => *Promise< string< [DataHexString](/v5/api/utils/bytes/#DataHexString) > >*

Returns the result of calling using the *transactionRequest*, with this account address being used as the `from` field.


#### *signer* . **estimateGas**( transactionRequest ) => *Promise< [BigNumber](/v5/api/utils/bignumber/) >*

Returns the result of estimating the cost to send the *transactionRequest*, with this account address being used as the `from` field.


#### *signer* . **resolveName**( ensName ) => *Promise< string< [Address](/v5/api/utils/address/#address) > >*

Returns the address associated with the *ensName*.


### Signing

#### *signer* . **signMessage**( message ) => *Promise< string< [RawSignature](/v5/api/utils/bytes/#signature-raw) > >*

This returns a Promise which resolves to the [Raw Signature](/v5/api/utils/bytes/#signature-raw) of *message*.

A signed message is prefixd with `"\x19Ethereum signed message:\n"` and the length of the message, using the [hashMessage](/v5/api/utils/hashing/#utils-hashMessage) method, so that it is [EIP-191](https://eips.ethereum.org/EIPS/eip-191) compliant. If recovering the address in Solidity, this prefix will be required to create a matching hash.

Sub-classes **must** implement this, however they may throw if signing a message is not supported, such as in a Contract-based Wallet or Meta-Transaction-based Wallet.


#### Note

If *message* is a string, it is **treated as a string** and converted to its representation in UTF8 bytes.

**If and only if** a message is a [Bytes](/v5/api/utils/bytes/#Bytes) will it be treated as binary data.

For example, the string `"0x1234"` is 6 characters long (and in this case 6 bytes long). This is **not** equivalent to the array `[ 0x12, 0x34 ]`, which is 2 bytes long.

A common case is to sign a hash. In this case, if the hash is a string, it **must** be converted to an array first, using the [arrayify](/v5/api/utils/bytes/#utils-arrayify) utility function.





#### *signer* . **signTransaction**( transactionRequest ) => *Promise< string< [DataHexString](/v5/api/utils/bytes/#DataHexString) > >*

Returns a Promise which resolves to the signed transaction of the *transactionRequest*. This method does not populate any missing fields.

Sub-classes **must** implement this, however they may throw if signing a transaction is not supported, which is common for security reasons in many clients.


#### *signer* . **sendTransaction**( transactionRequest ) => *Promise< [TransactionResponse](/v5/api/providers/types/#providers-TransactionResponse) >*

This method populates the transactionRequest with missing fields, using [populateTransaction](/v5/api/signer/#Signer-populateTransaction) and returns a Promise which resolves to the transaction.

Sub-classes **must** implement this, however they may throw if sending a transaction is not supported, such as the [VoidSigner](/v5/api/signer/#VoidSigner) or if the Wallet is offline and not connected to a [Provider](/v5/api/providers/provider/).


#### *signer* . **_signTypedData**( domain , types , value ) => *Promise< string< [RawSignature](/v5/api/utils/bytes/#signature-raw) > >*

Signs the typed data *value* with *types* data structure for *domain* using the [EIP-712](https://eips.ethereum.org/EIPS/eip-712) specification.


#### Experimental feature (this method name will change)

This is still an experimental feature. If using it, please specify the **exact** version of ethers you are using (e.g. spcify `"5.0.18"`, **not** `"^5.0.18"`) as the method name will be renamed from `_signTypedData` to `signTypedData` once it has been used in the field a bit.


```javascript
//_hide: signer = new Wallet("0x1234567890123456789012345678901234567890123456789012345678901234");

// All properties on a domain are optional
const domain = {
    name: 'Ether Mail',
    version: '1',
    chainId: 1,
    verifyingContract: '0xCcCCccccCCCCcCCCCCCcCcCccCcCCCcCcccccccC'
};

// The named list of all type definitions
const types = {
    Person: [
        { name: 'name', type: 'string' },
        { name: 'wallet', type: 'address' }
    ],
    Mail: [
        { name: 'from', type: 'Person' },
        { name: 'to', type: 'Person' },
        { name: 'contents', type: 'string' }
    ]
};

// The data to sign
const value = {
    from: {
        name: 'Cow',
        wallet: '0xCD2a3d9F938E13CD947Ec05AbC7FE734Df8DD826'
    },
    to: {
        name: 'Bob',
        wallet: '0xbBbBBBBbbBBBbbbBbbBbbbbBBbBbbbbBbBbbBBbB'
    },
    contents: 'Hello, Bob!'
};

//_result:
signature = await signer._signTypedData(domain, types, value);
//_log:
```

### Sub-Classes

#### *signer* . **checkTransaction**( transactionRequest ) => *[TransactionRequest](/v5/api/providers/types/#providers-TransactionRequest)*

This is generally not required to be overridden, but may be needed to provide custom behaviour in sub-classes.

This should return a **copy** of the *transactionRequest*, with any properties needed by `call`, `estimateGas` and `populateTransaction` (which is used by sendTransaction). It should also throw an error if any unknown key is specified.

The default implementation checks only if valid [TransactionRequest](/v5/api/providers/types/#providers-TransactionRequest) properties exist and adds `from` to the transaction if it does not exist.

If there is a `from` field it **must** be verified to be equal to the Signer's address.


#### *signer* . **populateTransaction**( transactionRequest ) => *Promise< [TransactionRequest](/v5/api/providers/types/#providers-TransactionRequest) >*

This is generally not required to be overridden, but may be needed to provide custom behaviour in sub-classes.

This should return a **copy** of *transactionRequest*, follow the same procedure as `checkTransaction` and fill in any properties required for sending a transaction. The result should have all promises resolved; if needed the [resolveProperties](/v5/api/utils/properties/#utils-resolveproperties) utility function can be used for this.

The default implementation calls `checkTransaction` and resolves to if it is an ENS name, adds `gasPrice`, `nonce`, `gasLimit` and `chainId` based on the related operations on Signer.


Wallet
------

#### **new ***ethers* . **Wallet**( privateKey [ , provider ] )

Create a new Wallet instance for *privateKey* and optionally connected to the *provider*.


#### *ethers* . *Wallet* . **createRandom**( [ options = {} ] ) => *[Wallet](/v5/api/signer/#Wallet)*

Returns a new Wallet with a random private key, generated from cryptographically secure entropy sources. If the current environment does not have a secure entropy source, an error is thrown.

Wallets created using this method will have a mnemonic.


#### *ethers* . *Wallet* . **fromEncryptedJson**( json , password [ , progress ] ) => *Promise< [Wallet](/v5/api/signer/#Wallet) >*

Create an instance from an encrypted JSON wallet.

If *progress* is provided it will be called during decryption with a value between 0 and 1 indicating the progress towards completion.


#### *ethers* . *Wallet* . **fromEncryptedJsonSync**( json , password ) => *[Wallet](/v5/api/signer/#Wallet)*

Create an instance from an encrypted JSON wallet.

This operation will operate synchronously which will lock up the user interface, possibly for a non-trivial duration. Most applications should use the asynchronous `fromEncryptedJson` instead.


#### *ethers* . *Wallet* . **fromMnemonic**( mnemonic [ , path , [ wordlist ] ] ) => *[Wallet](/v5/api/signer/#Wallet)*

Create an instance from a mnemonic phrase.

If path is not specified, the Ethereum default path is used (i.e. `m/44'/60'/0'/0/0`).

If wordlist is not specified, the English Wordlist is used.


### Properties

#### *wallet* . **address** => *string< [Address](/v5/api/utils/address/#address) >*

The address for the account this Wallet represents.


#### *wallet* . **provider** => *[Provider](/v5/api/providers/provider/)*

The provider this wallet is connected to, which will be used for any [Blockchain Methods](/v5/api/signer/#Signer--blockchain-methods) methods. This can be null.


#### Note

A **Wallet** instance is immutable, so if you wish to change the Provider, you may use the [connect](/v5/api/signer/#Signer-connect) method to create a new instance connected to the desired provider.


#### *wallet* . **publicKey** => *string< [DataHexString](/v5/api/utils/bytes/#DataHexString)< 65 > >*

The uncompressed public key for this Wallet represents.


### Methods

#### *wallet* . **encrypt**( password , [ options = {} , [ progress ] ] ) => *Promise< string >*

Encrypt the wallet using *password* returning a Promise which resolves to a JSON wallet.

If *progress* is provided it will be called during decryption with a value between 0 and 1 indicating the progress towards completion.


```javascript
// Create a wallet instance from a mnemonic...
mnemonic = "announce room limb pattern dry unit scale effort smooth jazz weasel alcohol"
walletMnemonic = Wallet.fromMnemonic(mnemonic)

// ...or from a private key
walletPrivateKey = new Wallet(walletMnemonic.privateKey)

//_result:
walletMnemonic.address === walletPrivateKey.address
//_log:

// The address as a Promise per the Signer API
//_result:
await walletMnemonic.getAddress()
//_log:

// A Wallet address is also available synchronously
//_result:
walletMnemonic.address
//_log:

// The internal cryptographic components
//_result:
walletMnemonic.privateKey
//_log:
//_result:
walletMnemonic.publicKey
//_log:

// The wallet mnemonic
//_result:
walletMnemonic.mnemonic
//_log:

// Note: A wallet created with a private key does not
//       have a mnemonic (the derivation prevents it)
//_result:
walletPrivateKey.mnemonic
//_log:

// Signing a message
//_result:
await walletMnemonic.signMessage("Hello World")
//_log:

tx = {
  to: "0x8ba1f109551bD432803012645Ac136ddd64DBA72",
  value: utils.parseEther("1.0")
}

// Signing a transaction
//_result:
await walletMnemonic.signTransaction(tx)
//_log:

// The connect method returns a new instance of the
// Wallet connected to a provider
//_result:
wallet = walletMnemonic.connect(provider)
//_null:

// Querying the network
//_result:
await wallet.getBalance();
//_log:
//_result:
await wallet.getTransactionCount();
//_log:

// Sending ether
//_hide: wallet = localSigner; /* prevent insufficient funds error from blowing up the docs */
//_result:
await wallet.sendTransaction(tx)
//_log:
```

VoidSigner
----------

#### **new ***ethers* . **VoidSigner**( address [ , provider ] ) => *[VoidSigner](/v5/api/signer/#VoidSigner)*

Create a new instance of a **VoidSigner** for *address*.


#### *voidSigner* . **address** => *string< [Address](/v5/api/utils/address/#address) >*

The address of this **VoidSigner**.


```javascript
address = "0x8ba1f109551bD432803012645Ac136ddd64DBA72"
signer = new ethers.VoidSigner(address, provider)

// The DAI token contract
abi = [
  "function balanceOf(address) view returns (uint)",
  "function transfer(address, uint) returns (bool)"
]
contract = new ethers.Contract("dai.tokens.ethers.eth", abi, signer)

// Get the number of tokens for this account
//_result:
tokens = await contract.balanceOf(signer.getAddress())
//_log:

//
// Pre-flight (check for revert) on DAI from the signer
//
// Note: We do not have the private key at this point, this
//       simply allows us to check what would happen if we
//       did. This can be useful to check before prompting
//       a request in the UI
//

// This will pass since the token balance is available
//_result:
await contract.callStatic.transfer("donations.ethers.eth", tokens)
//_log:

// This will fail since it is greater than the token balance
//_throws:
await contract.callStatic.transfer("donations.ethers.eth", tokens.add(1))
//_log:
```

ExternallyOwnedAccount
----------------------

#### *eoa* . **address** => *string< [Address](/v5/api/utils/address/#address) >*

The [Address](/v5/api/utils/address/#address) of this EOA.


#### *eoa* . **privateKey** => *string< [DataHexString](/v5/api/utils/bytes/#DataHexString)< 32 > >*

The privateKey of this EOA


#### *eoa* . **mnemonic** => *[Mnemonic](/v5/api/utils/hdnode/#Mnemonic)*

*Optional*. The account HD mnemonic, if it has one and can be determined. Some sources do not encode the mnemonic, such as HD extended keys.


