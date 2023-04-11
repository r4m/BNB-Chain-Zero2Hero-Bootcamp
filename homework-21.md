### Optimising Storage

Take [this contract](https://gist.github.com/extropyCoder/6e9b5d5497b8ead54590e72382cdca24)
Use the [sol2uml tool](https://github.com/naddison36/sol2uml) to find out how many storage slots it is using.
By re ordering the variables, can you reduce the number of storage slots needed?

Running the command `sol2uml storage Store.sol -c Store` after having installed locally the sol2uml tool, we can see that the contract occupies 61 slots.

Looking at the contract we can re-arrange it in the following way to occupy 59 slots:

```solidity
// SPDX-License-Identifier: UNLICENSED
pragma solidity ^0.8.13;

contract Store {
    struct payments {
        bool valid;
        uint256 amount;
        address sender;
        uint8 paymentType;
        uint256 finalAmount;
        address receiver;
        uint256 initialAmount;
        bool checked;
    }
    uint256 public number;
    uint8 index;
    bool flag1;
    bool flag2;
    bool flag3;
    address admin;
    address admin2;
    payments[8] topPayments;
    mapping(address => uint256) balances;

    constructor() {}

    function setNumber(uint256 newNumber) public {
        number = newNumber;
    }

    function increment() public {
        number++;
    }
}
```

since we can put in slot 8 the `number` int since it occupies exactly 32 bytes. We can then aggregate together in a single slot the booleans `flag1`, `flag2`, `flag3` that take 1 byte each, the `index` int that takes 1 byte too and then we can add an address like the `admin` one such that in slot 1 we just leave 8 bytes unallocated. Then we can put in slot 2 the other address `admin2` and having from slot 3 up to 58 the fixed size array of `topPayments` based on the payments structure. Lastly we can leave in slot 59 the `balances` mapping, that will have a dynamic length.
