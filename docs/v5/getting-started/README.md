-----

Documentation: [html](https://docs.ethers.io/)

-----

Getting Started
===============

Installing
----------

```
/home/ricmoo> npm install --save ethers
```

Importing
---------

### Node.js

```
const { ethers } = require("ethers");
```

```
import { ethers } from "ethers";
```

### Web Browser

```
<script type="module">
    import { ethers } from "https://cdn.ethers.io/lib/ethers-5.2.esm.min.js";
    // Your code here...
</script>
```

```
<script src="https://cdn.ethers.io/lib/ethers-5.2.umd.min.js"
        type="application/javascript"></script>
```

Common Terminology
------------------

Common Terms



Connecting to Ethereum: Metamask
--------------------------------

```
// A Web3Provider wraps a standard Web3 provider, which is
// what Metamask injects as window.ethereum into each page
const provider = new ethers.providers.Web3Provider(window.ethereum)

// The Metamask plugin also allows signing transactions to
// send ether and pay to change state within the blockchain.
// For this, you need the account signer...
const signer = provider.getSigner()
```

Connecting to Ethereum: RPC
---------------------------

```
// If you don't specify a //url//, Ethers connects to the default 
// (i.e. ``http:/\/localhost:8545``)
const provider = new ethers.providers.JsonRpcProvider();

// The provider also allows signing transactions to
// send ether and pay to change state within the blockchain.
// For this, we need the account signer...
const signer = provider.getSigner()
```

### Querying the Blockchain

```javascript
// Look up the current block number
//_result:
await provider.getBlockNumber()
//_log:

// Get the balance of an account (by address or ENS name, if supported by network)
//_result:
balance = await provider.getBalance("ethers.eth")
//_log:

// Often you need to format the output to something more user-friendly,
// such as in ether (instead of wei)
//_result:
ethers.utils.formatEther(balance)
//_log:

// If a user enters a string in an input field, you may need
// to convert it from ether (as a string) to wei (as a BigNumber)
//_result:
ethers.utils.parseEther("1.0")
//_log:
```

### Writing to the Blockchain

```
// Send 1 ether to an ens name.
const tx = signer.sendTransaction({
    to: "ricmoo.firefly.eth",
    value: ethers.utils.parseEther("1.0")
});
```

Contracts
---------

```javascript
// You can also use an ENS name for the contract address
const daiAddress = "dai.tokens.ethers.eth";

// The ERC-20 Contract ABI, which is a common contract interface
// for tokens (this is the Human-Readable ABI format)
const daiAbi = [
  // Some details about the token
  "function name() view returns (string)",
  "function symbol() view returns (string)",

  // Get the account balance
  "function balanceOf(address) view returns (uint)",

  // Send some of your tokens to someone else
  "function transfer(address to, uint amount)",

  // An event triggered whenever anyone transfers to someone else
  "event Transfer(address indexed from, address indexed to, uint amount)"
];

// The Contract object
const daiContract = new ethers.Contract(daiAddress, daiAbi, provider);

//_hide: _page.daiAbi = daiAbi;
//_hide: _page.daiContract = daiContract;
```

### Read-Only Methods

```javascript
//_hide: const daiContract = _page.daiContract;

// Get the ERC-20 token name
//_result:
await daiContract.name()
//_log:

// Get the ERC-20 token symbol (for tickers and UIs)
//_result:
await daiContract.symbol()
//_log:

// Get the balance of an address
balance = await daiContract.balanceOf("ricmoo.firefly.eth")
//_log: balance

// Format the DAI for displaying to the user
//_result:
ethers.utils.formatUnits(balance, 18)
//_log:
```

### State Changing Methods

```
// The DAI Contract is currently connected to the Provider,
// which is read-only. You need to connect to a Signer, so
// that you can pay to send state-changing transactions.
const daiWithSigner = contract.connect(signer);

// Each DAI has 18 decimal places
const dai = ethers.utils.parseUnits("1.0", 18);

// Send 1 DAI to "ricmoo.firefly.eth"
tx = daiWithSigner.transfer("ricmoo.firefly.eth", dai);
```

### Listening to Events

```javascript
//_hide: const daiContract = _page.daiContract;
//_hide: const formatEther = ethers.utils.formatEther;

// Receive an event when ANY transfer occurs
daiContract.on("Transfer", (from, to, amount, event) => {
    console.log(`${ from } sent ${ formatEther(amount) } to ${ to}`);
    // The event object contains the verbatim log data, the
    // EventFragment and functions to fetch the block,
    // transaction and receipt and event functions
});

// A filter for when a specific address receives tokens
myAddress = "0x8ba1f109551bD432803012645Ac136ddd64DBA72";
filter = daiContract.filters.Transfer(null, myAddress)
//_log: filter

// Receive an event when that filter occurs
daiContract.on(filter, (from, to, amount, event) => {
    // The to will always be "address"
    console.log(`I got ${ formatEther(amount) } from ${ from }.`);
});

//_hide: daiContract.removeAllListeners(); /* Don't want to block the docs from compiling... */
```

### Query Historic Events

```javascript
//_hide: const signer = new ethers.VoidSigner("0x8ba1f109551bD432803012645Ac136ddd64DBA72");
//_hide: const daiContract = _page.daiContract;

// Get the address of the Signer
myAddress = await signer.getAddress()
//_log: myAddress

// Filter for all token transfers from me
filterFrom = daiContract.filters.Transfer(myAddress, null);
//_log: filterFrom

// Filter for all token transfers to me
filterTo = daiContract.filters.Transfer(null, myAddress);
//_log: filterTo

// List all transfers sent from me a specific block range
//_result:
await daiContract.queryFilter(filterFrom, 9843470, 9843480)
//_log:

//
// The following have had the results omitted due to the
// number of entries; but they provide some useful examples
//

// List all transfers sent in the last 10,000 blocks
await daiContract.queryFilter(filterFrom, -10000)

// List all transfers ever sent to me
await daiContract.queryFilter(filterTo)
```

Signing Messages
----------------

```javascript
//_hide: const signer = ethers.Wallet.createRandom();

// To sign a simple string, which are used for
// logging into a service, such as CryptoKitties,
// pass the string in.
signature = await signer.signMessage("Hello World");
//_log: signature

//
// A common case is also signing a hash, which is 32
// bytes. It is important to note, that to sign binary
// data it MUST be an Array (or TypedArray)
//

// This string is 66 characters long
message = "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef"

// This array representation is 32 bytes long
messageBytes = ethers.utils.arrayify(message);
//_log: messageBytes

// To sign a hash, you most often want to sign the bytes
signature = await signer.signMessage(messageBytes)
//_log: signature
```

