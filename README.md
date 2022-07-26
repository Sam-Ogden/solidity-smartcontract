Solidity Path: Beginner to Intermediate Smart Contracts - Crypto Zombies Course

https://cryptozombies.io/en/course
- A `wei` is the smallest sub-unit of Ether — there are 10^18 wei in one ether.
- Define a contract `contract ZombieFactory {}`
- Functions are `internal`, `exteral`, `private` or `public`, public by default
  - private: only called within this contract (not externally, or by inheriting contracts)
  - public: called by anyone
  - internal: same as private but contracts that inherit can also access it
  - external: Only called outside of the contract
- Private func and args prefixed with `_`
- `msg.sender` to get calling users address, type is `address`
- Hash function: `keccak256` with `abi.encodePacked`
- Maps: `mapping (address => uint) ownerZombieCount;`
- `require` is a function that ensures some condition is met, if it fails then the function exits
- Inheritence: `contract ZombieFeeding is ZombieFactory, ERC721 {}`
- Commenting your code and the natspec standard

## Storage
In Solidity, there are two locations you can store variables — in `storage` (on the blockchain) and in `memory` (temporary, and erased between external calls to the contract)

**State variables** (variables declared outside of functions) are **by default storage and written permanently to the blockchain**

Most of the time you don't need to use these keywords because Solidity handles them by default: variables declared inside functions are memory and will disappear when the function call ends.

However, there are times when you do need to use these keywords, namely when dealing with structs and arrays within functions

```
contract SandwichFactory {
  struct Sandwich {
    string name;
    string status;
  }

  Sandwich[] sandwiches;

  function eatSandwich(uint _index) public {
    // Sandwich mySandwich = sandwiches[_index];

    // ^ Seems pretty straightforward, but solidity will give you a warning
    // telling you that you should explicitly declare `storage` or `memory` here.

    // So instead, you should declare with the `storage` keyword, like:
    Sandwich storage mySandwich = sandwiches[_index];
    // ...in which case `mySandwich` is a pointer to `sandwiches[_index]`
    // in storage, and...
    mySandwich.status = "Eaten!";
    // ...this will permanently change `sandwiches[_index]` on the blockchain.

    // If you just want a copy, you can use `memory`:
    Sandwich memory anotherSandwich = sandwiches[_index + 1];
    // ...in which case `anotherSandwich` will simply be a copy of the 
    // data in memory, and...
    anotherSandwich.status = "Eaten!";
    // ...will just modify the temporary variable and have no effect 
    // on `sandwiches[_index + 1]`. But you can do this:
    sandwiches[_index + 1] = anotherSandwich;
    // ...if you want to copy the changes back into blockchain storage.
  }
}
```

## Interacting with other contracts
- Define an interface (still using contract keyword) with the required functions from the contract you want to interact with (e.g. KittyInterface)

### Using another contract
contract NumberInterface {
  function getNum(address _myAddress) public view returns (uint);
}

address NumberInterfaceAddress = 0xab38... 
// ^ The address of the FavoriteNumber contract on Ethereum
NumberInterface numberContract = NumberInterface(NumberInterfaceAddress);

We can either get this directly from GitHub, or from NPM packages. The framework you're using (like Truffle, Brownie, Remix, Hardhat) will determine whether or not to use GitHub or NPM packages.

## Lesson 3
- after you deploy a contract to Ethereum, it’s immutable, which means that it can never be modified or updated again.
- Extenal dependencies like the Cryptokitty address might change over time as changes are made and new contracts deployed, could use a setKittyContractAddress to update out contract without needed to deploy a new version
- We can make contracts Ownable — meaning the owner can have special privileges.
- Contracts have a `constructor` with same name as contract, these get called 1 time when it is first created
- Function modifiers
```
  modifier onlyOwner() {
    require(isOwner());
    _;
  }

  function transferOwnership(address newOwner) public onlyOwner {...}

```

## Gas
Users have to pay every time they execute a function on your DApp using a currency called gas. Amount of gas depends on amount of resources required to execute it.

### Saving Gas
Normally there's no benefit to using these sub-types because Solidity reserves 256 bits of storage regardless of the uint size. For example, using uint8 instead of uint (uint256) won't save you any gas.

But there's an exception to this: inside structs where you'll want to use the smallest integer sub-types you can get away with.

> view functions don't cost any gas when they're called externally by a user.
> This is because view functions don't actually change anything on the blockchain – they only read the data. So marking a function with view tells web3.js that it only needs to query your local Ethereum node to run the function, and it doesn't actually have to create a transaction on the blockchain (which would need to be run on every single node, and cost gas).

> Note: If a view function is called internally from another function in the same contract that is not a view function, it will still cost gas. This is because the other function creates a transaction on Ethereum, and will still need to be verified from every node. So view functions are only free when they're called externally.

### Storage is Expensive 
One of the more expensive operations in Solidity is using storage — particularly writes.

This is because every time you write or change a piece of data, it’s written permanently to the blockchain. Forever! Thousands of nodes across the world need to store that data on their hard drives, and this amount of data keeps growing over time as the blockchain grows. So there's a cost to doing that.

In order to keep costs down, you want to avoid writing data to storage except when absolutely necessary. Sometimes this involves seemingly inefficient programming logic — like rebuilding an array in memory every time a function is called instead of simply saving that array in a variable for quick lookups.

In most programming languages, looping over large data sets is expensive. But in Solidity, this is way cheaper than using storage if it's in an external view function, since view functions don't cost your users any gas. (And gas costs your users real money!).

### Memory arrays
You can use the memory keyword with arrays to create a new array inside a function without needing to write anything to storage. Memory arrays must be created with a length argument (in this example, 3). They currently cannot be resized like storage arrays can with array.push()


## Payable Modifier
`payable` functions are a special type of function that can receive Ether

```
  function buySomething() external payable {
    require(msg.value == 0.001 ether);
    transferThing(msg.sender);
  }

  // From web3js
  OnlineStore.buySomething({from: web3.eth.defaultAccount, value: web3.utils.toWei(0.001)})
```

>If a function is not marked payable and you try to send Ether to it as above, the function will reject your transaction.

After you send Ether to a contract, it gets stored in the contract's Ethereum account, and it will be trapped there — unless you add a function to withdraw the Ether from the contract.

```
contract GetPaid is Ownable {
  function withdraw() external onlyOwner {
    address payable _owner = address(uint160(owner()));
    _owner.transfer(address(this).balance); 
  }
}
```
- `address(this).balance` returns balance on the contract

>It is important to note that you cannot transfer Ether to an address unless that address is of type address payable. But the _owner variable is of type uint160, meaning that we must explicitly cast it to address payable.

## Random Numbers
Cant really generate random numbers safely.

The best source of randomness we have in Solidity is the keccak256 hash function

e.g. `uint random = uint(keccak256(abi.encodePacked(now, msg.sender, randNonce))) % 100;`

- This method is vulnerable to attack by a dishonest node

In Ethereum, when you call a function on a contract, you broadcast it to a node or nodes on the network as a transaction. The nodes on the network then collect a bunch of transactions, try to be the first to solve a computationally-intensive mathematical problem as a "Proof of Work", and then publish that group of transactions along with their Proof of Work (PoW) as a block to the rest of the network.

Once a node has solved the PoW, the other nodes stop trying to solve the PoW, verify that the other node's list of transactions are valid, and then accept the block and move on to trying to solve the next block.

Example:
Let's say we had a coin flip contract — heads you double your money, tails you lose everything. Let's say it used the above random function to determine heads or tails. (random >= 50 is heads, random < 50 is tails).

If I were running a node, I could publish a transaction only to my own node and not share it. I could then run the coin flip function to see if I won — and if I lost, choose not to include that transaction in the next block I'm solving. I could keep doing this indefinitely until I finally won the coin flip and solved the next block, and profit.

https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract

# Tokens
A token on Ethereum is basically just a smart contract that follows some common rules

`ERC20` tokens act like currencies and are interchangable

`ERC721` tokens are not interchangeable since each one is assumed to be unique, and are not divisible. You can only trade them in whole units, and each one has a unique ID

```
contract ERC721 {
  event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
  event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);

  function balanceOf(address _owner) external view returns (uint256);
  function ownerOf(uint256 _tokenId) external view returns (address);
  function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
  function approve(address _approved, uint256 _tokenId) external payable;
}
```


## Contract security enhancements: Overflows and Underflows
```
uint8 number = 255;
number++; // overflows to 0

uint8 number = 0;
number--; // overflows to 255
```

### OpenZepplin SafeMath library
- A library is a special type of contract in Solidity. One of the things it is useful for is to attach functions to native data types.
```
library SafeMath {
  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
  // ...
}
```

- Even though the function calls for 2 args, the uint we call the function on is automatically passed in as the first argument.
```
contract ZombieFactory is Ownable {

  using SafeMath for uint256;
  ...
}
  // Usage ex.
  uint256 a = 5;
  uint256 b = a.add(3); // 5 + 3 = 8
  uint256 c = a.mul(2); // 5 * 2 = 10
```

### Assert vs Require
assert is similar to require, where it will throw an error if false. The difference between assert and require is that require will refund the user the rest of their gas when a function fails, whereas assert will not. 


# Web3.js
Ethereum network is made up of nodes, with each containing a copy of the blockchain. When you want to call a function on a smart contract, you need to query one of these nodes and tell it:

The address of the smart contract
The function you want to call, and
The variables you want to pass to that function.

Ethereum nodes only speak a language called JSON-RPC

Example:
CryptoZombies.methods.createRandomZombie("Vitalik Nakamoto 🤔")
  .send({ from: "0xb60e8dd61c5d32be8058bb8eb970870f07233155", gas: "3000000" })

Remember, Ethereum is made up of nodes that all share a copy of the same data. Setting a Web3 Provider in Web3.js tells our code which node we should be talking to handle our reads and writes. It's kind of like setting the URL of the remote web server for your API calls in a traditional web app.

You could host your own Ethereum node as a provider.

## Infura
A third-party service that makes your life easier so you don't need to maintain your own Ethereum node in order to provide a DApp for your users

Infura is a service that maintains a set of Ethereum nodes with a caching layer for fast reads, which you can access for free through their API.

`var web3 = new Web3(new Web3.providers.WebsocketProvider("wss://mainnet.infura.io/ws"));`

## Metamask
Metamask is a browser extension for Chrome and Firefox that lets users securely manage their Ethereum accounts and private keys, and use these accounts to interact with websites that are using Web3.js.

Metamask uses Infura's servers under the hood as a web3 provider

Metamask injects their web3 provider into the browser in the global JavaScript object web3. So your app can check to see if web3 exists, and if it does use web3.currentProvider as its provider.

```
window.addEventListener('load', function() {

  // Checking if Web3 has been injected by the browser (Mist/MetaMask)
  if (typeof web3 !== 'undefined') {
    // Use Mist/MetaMask's provider
    web3js = new Web3(web3.currentProvider);
  } else {
    // Handle the case where the user doesn't have web3. Probably
    // show them a message telling them to install Metamask in
    // order to use our app.
  }

  // Now you can start your app & access web3js freely:
  startApp()

})
```

- Metamask allows users to manage multiple accounts
- use `var userAccount = web3.eth.accounts[0]` to get currently active one
- account may change over time, so we need to keep checking the currently active account;
```
var accountInterval = setInterval(function() {
  // Check if account has changed
  if (web3.eth.accounts[0] !== userAccount) {
    userAccount = web3.eth.accounts[0];
    // Call some function to update the UI with the new account
    updateInterface();
  }
}, 100);
```

## Talking to Contracts
- Need: Contract address and contract ABI
- After you deploy your contract, it gets a fixed address on Ethereum where it will live forever.
- `ABI` stands for `Application Binary Interface`. Basically it's a representation of your contracts' methods in JSON format that tells Web3.js how to format function calls in a way your contract will understand. Provided by the compiler before deploying.

## Calling Contract Functions: Call & Send
- `call` is used for `view` and `pure` functions. It only runs on the local node, and *won't create a transaction on the blockchain*. They also dont cost any gas & user wont be required to sign a transaction with metamask
`myContract.methods.myMethod(123).call()`

- `send` *will create a transaction and change data on the blockchain*. You'll need to use send for any functions that aren't view or pure. Requires gas and transaction signed by metamask `myContract.methods.myMethod(123).send()`

- when you declare a variable public, it automatically creates a public "getter" function with the same name

### Send
- sending a transaction requires a from address of who's calling the function (which becomes msg.sender in your Solidity code). We'll want this to be the user of our DApp, so MetaMask will pop up to prompt them to sign the transaction.

- sending a transaction costs gas

There will be a significant delay from when the user sends a transaction and when that transaction actually takes effect on the blockchain. This is because we have to wait for the transaction to be included in a block, and the block time for Ethereum is on average 15 seconds. If there are a lot of pending transactions on Ethereum or if the user sends too low of a gas price, our transaction may have to wait several blocks to get included, and this could take minutes.

Example send:
```
function createRandomZombie(name) {
  // This is going to take a while, so update the UI to let the user know
  // the transaction has been sent
  $("#txStatus").text("Creating new zombie on the blockchain. This may take a while...");
  // Send the tx to our contract:
  return cryptoZombies.methods.createRandomZombie(name)
  .send({ from: userAccount })
  .on("receipt", function(receipt) { // "receipt" fires when transaction was accepted onto chain
    $("#txStatus").text("Successfully created " + name + "!");
    // Transaction was accepted into the blockchain, let's redraw the UI
    getZombiesByOwner(userAccount).then(displayZombies);
  })
  .on("error", function(error) { // error will fire if there's an issue that prevented the transaction from being included in a block, such as the user not sending enough gas.
    // Do something to alert the user their transaction has failed
    $("#txStatus").text(error);
  });
}
```

> Note: You can optionally specify gas and gasPrice when you call send, e.g. .send({ from: userAccount, gas: 3000000 }). If you don't specify this, MetaMask will let the user choose these values.


### Web3 Utils Convert eth to wei
`web3js.utils.toWei("1");`

## Subscribing to events
```
cryptoZombies.events.NewZombie()
.on("data", function(event) { ... })
.on("error", console.error);
```

Note the above would trigger an alert every time ANY zombie was created in our DApp — not just for the current user. What if we only wanted alerts for the current user? We need `indexed`

### Using indexed
In order to filter events and only listen for changes related to the current user, our Solidity contract would have to use the indexed keyword, like we did in the Transfer event of our ERC721 implementation:

`event Transfer(address indexed _from, address indexed _to, uint256 _tokenId);`

because _from and _to are indexed, that means we can filter for them in our event listener in our front end:

```
// Use `filter` to only fire this code when `_to` equals `userAccount`

cryptoZombies.events.Transfer({ filter: { _to: userAccount } })

.on("data", function(event) {
  let data = event.returnValues;
  // The current user just received a zombie!
  // Do something here to update the UI to show it
}).on("error", console.error);
```

### Querying Past events
We can even query past events using getPastEvents, and use the filters fromBlock and toBlock to give Solidity a time range for the event logs ("block" in this case referring to the Ethereum block number):
```
cryptoZombies.getPastEvents("NewZombie", { fromBlock: 0, toBlock: "latest" })
.then(function(events) {
  // `events` is an array of `event` objects that we can iterate, like we did above
  // This code will get us a list of every zombie that was ever created
});
```

Because you can use this method to query the event logs since the beginning of time, this presents an interesting use case: Using events as a cheaper form of storage.
The tradeoff here is that events are not readable from inside the smart contract itself.


# Chainlink: Decentralized Oracles
- Defi app needs to access data external from the outside world, e.g. the price of eth
- We need to get this data from decentralized oracle network (DON) and decentralized data sources.
- Chainlink is a framework for decentralized oracle networks, and is a way to get data in from multiple sources across multiple oracles. This DON aggregates data in a decentralized manner and places it on the blockchain in a smart contract (often referred to as a "price reference feed" or "data feed") for us to read from
- Using Chainlink Data Feeds is a way to cheaply, more accurately, and with more security gather data from the real world in this decentralized context. Since the data is coming from multiple sources, multiple people can partake in the ecosystem and it becomes even cheaper than running even a centralized oracle (https://data.chain.link/)
- Lending protocols like Compound rely on this 