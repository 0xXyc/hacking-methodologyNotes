---
description: 10-15-22
---

# Solidity

## Ethereum IDE

{% embed url="https://remix.ethereum.org/#optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.7+commit.e28d00a7.js" %}

## Background

* <mark style="color:yellow;">Solidity</mark> is used to <mark style="color:yellow;">develop smart contracts that run on Ethereum</mark>

Well, what is a smart contract?

* A program that runs on the Ethereum Blockchain
* It can have a balance
* Capable of handling transactions
* Commonly used to ensure payment
* Impose penalties in certain situations

### Pragmas

* While reviewing source code, you will commonly see "<mark style="color:yellow;">`pragma solidity >=0.4.0 <0.7.0`</mark>`;`"
* This is known as a <mark style="color:yellow;">pragma</mark>
* A pragma contains a <mark style="color:yellow;">set of instructions for compilers about how to treat the source code</mark>

### Smart Contracts & Solidity

* A contract in the sense of Solidity is a collection of code and data that resides at a specific address on the Ethereum blockchain
* Blockchain -> Smart Contract -> Address; Hash of Smart Contract

## Let's Write Some Code

* First, you must include a license (comment)

Example:

```
// SPDX-License-Identifier: MIT
```

Stacked License Example:

```
// SPDX-License-Identifier: Apache-2.0 OR MIT
```

Unlicensed Example:

```
// SPDX-License-Identifier: UNLICENSED
```

* Next, we need to declare the compiler version
* A.K.A. the <mark style="color:yellow;">pragma</mark>
* <mark style="color:yellow;">YOU MUST END all of your statements with SEMI-COLONS ;</mark>

Example:

Declare different solidity versions for our compiler

0.7.0 is our minimum and 0.9.0 cannot be surpassed in this example

```
pragma solidity >= 0.7.0 < 0.9.0;
```

Your code should now be looking something like this:

```solidity
// SPDX-License-Identifier: MIT

// Declare the versions of the Solidity compiler
pragma solidity >= 0.7.0 < 0.9.0;

// Used to convert uint to string
import "@openzepplin/contracts/utils/Strings.sol";
```

* uint? Unsigned integer
* Now it is time to start making a contract!

## Making a Contract // Variables

```solidity
contract TestContract{
    bool public canVote = true;
}
```

### Boolean

bool

* Remember, booleans can only be <mark style="color:yellow;">true</mark> or <mark style="color:yellow;">false</mark>

### Integer

int

### Unsigned Integer

uint

### String

string

### Put them all Together

```solidity
contract TestContract{
    bool public canVote = true;
    int private myAge = 47;
    uint internal favNum = 3;
    string myName = "Hacker";
}
```

### Floats

* Floats or Floating Point Numbers are not used in Solidity because it is a financial programming language
* Floats are not very accurate as they represent a range of numbers
* Financial data needs to be very accurate for obvious reasons
* Therefore, you would need to use an outside library if floats needed to be used

## Default Constructor

* What is a constructor
* Used to initialize variables

## Functions

Function format:

<mark style="color:yellow;">`function funcName(parameterList) scope returns() {statements}`</mark>

