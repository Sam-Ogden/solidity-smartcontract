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