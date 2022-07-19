Solidity Path: Beginner to Intermediate Smart Contracts - Crypto Zombies Course

https://cryptozombies.io/en/course

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
- Inheritence: `contract ZombieFeeding is ZombieFactory {}`

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


