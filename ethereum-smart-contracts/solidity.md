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



## Getting to Work

```
// SPDX-License-Identifier: MIT

// Declare the versions of the Solidity compiler
pragma solidity >= 0.7.0 < 0.9.0;

// Used to convert uint to string
import "@openzepplin/contracts/utils/Strings.sol";

contract TestContract{
    bool public canVote = true;
    int private myAge = 47;
    uint internal favNum = 3;
    string myName = "Hacker";

    //

    constructor() {}

    // function funcName(parameterList) scope returns() {statements}
    function getSum(uint _num1, uint _num2) public pure returns(uint){
        uint _mySum = _num1 + _num2;
        return _mySum;
    }

}
```

```
// SPDX-License-Identifier: MIT
// Each source file begins by identifying its license
// No license : UNLICENSED
// GNU : GPL-2.0-only
// Apache or MIT : Apache-2.0 OR MIT

// A Blockchain is essentially a database that stores transactions. 
// When we add to the blockchain that transaction is timestamped.
// We can only add to the blockchain and not delete, but we can
// update sort of. When you update however we do so using new 
// hashes or addresses. 

// Each block contains data, the hash for the block and the hash for
// the previous block. The hash for the previous block is the chain
// part of a blockchain. 

// A Hash is unique and is used to identify a block and its content

// Blockchains are secured from tampering by slowing down the creation 
// of new blocks. Also the blockchain is shared by many entities and
// if a new block is added it is verified by everyone that shares
// that blockchain.

// Ethereum Virtual Machine : Runtime environment for smart contracts
// Smart contracts are isolated and have limited access to other contracts
// When we make a smart contract and deploy it it gets an account
// The address of a contract is determined when the contract is created

/* If you click Compile and then Deploy we see information about the contract 
on the blockchain */

// You can add multiple plugins on the left

// Declare the versions of the Solidity compiler that work
// with our code
pragma solidity >= 0.7.0 < 0.9.0;

// Used to convert uint to string
import "@openzeppelin/contracts/utils/Strings.sol";

contract TestContract{

    // ----- VARIABLES ----- 
    // Begin with a letter or underscore and can also contain numbers

    // State : Values are permanently stored in contract storage and is available
    // to all functions in the contract
    // Local : Values available only in the function in which they are defined
    // Global : Provide info about the blockchain and are built into Solidity
    // A Private variable can only be called within the contract

    // ----- DATA TYPES -----
    // Booleans are true or false
    // State variable that is accessible outside of the contract because
    // of public
    bool public canVote = true; 

    // Integers store signed and unsigned whole numbers
    // Private so can only be accessed in the contract
    int private myAge = 47;

    // Unsigned Integers (You Ints) start as uint8 and increase to uint256
    // in increments of 8
    // You also have uint8, uint16, uint32, ...
    // An internal variable can only be accessed in the contract
    // or by contracts that inherit from this one
    uint internal favNum = 3;

    // If you want to use numbers bigger than 256 bits, or floating
    // points you have to emulate them. There are libraries that
    // do but they lack standardized number formats
    // Because Solidity concerns itself with finance values must
    // be exact and since floats aren't exact we can't use them
    // as we do with other languages
    
    // Strings 
    string myName = "Derek";

    // Used to initialize contract variables (More on them later) 
    constructor(){}

    // ----- CASTING UINT & BYTES -----
    // We want to reduce our computational expenses by using
    // the correctly sized uint

    // A uint is 256 bit unsigned integer by default
    // uint 256 : Max size 1.15792089 x 10^77
    // uint8 : 2^8 - 1 : 255
    // uint16 : 2^16 - 1 : 65,535
    // uint32 : 2^32 - 1 : 4,294,967,295

    // We convert by putting the value to convert inside parentheses
    // with the type to convert to before
    uint toBig = 250;
    uint8 justRight = uint8(toBig);

    // ----- ETHER UNITS -----
    // Ether is the currency of the Ethereum blockchain
    // Wei is the smallest denomination of Ether
    // 1 Ether = 10^18 Wei (Way)
    // 1 Ether = 1,000,000,000 Gwei
    // 1 Ether = 1,000 Finney
    // We need this many zeroes because $1 == .00032 Ethereum

    // ----- FUNCTIONS ----- 
    // function funcName(parameterList) scope returns() {statements}
    
    // Create a function and define parameters it receives
    // We are stating to store the arguments in memory

    // Arguments are either passed by VALUE : Value changes in function
    // don't effect the value outside of it
    // Or, by REFERENCE : changes in function effect value outside

    // Function variables should start with an underscore to 
    // differentiate them from global variables

    // ----- SCOPE -----
    // 1. Public functions are excessible by other contracts
    // 2. Private functions are only accessible to code that is in the contract
    // Private function names begin with _
    // 3. External functions can't be called by the contract, but can be
    // called outside of the contract
    // 4. Internal functions are only accessible within the contract or by
    // other contracts that inherit from this contract

    // A VIEW function works with data and allows us to view the
    // results of the function (Won't modify the state)
    // A PURE function doesn't allow reading or modifying state

    // This function returns a uint
    // Since it doesn't return a state value but instead a 
    // calculation it is marked as pure
    function getSum(uint _num1, uint _num2) public pure returns (uint) {
        // Variables created in a function are only available in
        // the function : Local Variables
        uint _mySum = _num1 + _num2;
        return _mySum;
    }

    function getResult() public pure returns(uint){
        uint a = 10;
        uint b = 5;
        uint result = a + b;
        return result;
    }

    uint specialVal = 10;

    // External means it can only be called from outside the contract
    // but not from within (We can change state variables)
    function changeSV(uint _val) external {
        specialVal = _val;
    }

    // View needed to return the value but restrict changing it
    function getSV() external view returns(uint){
        return specialVal;
    }

    // ----- MATH OPERATORS -----
    // You can return multiple values
    function doMath(int _num1, int _num2) public pure returns(int, int, int,
    int, int, int){
        // If we want to require something to be true to continue
        // executing the function use require

        // ----- ERROR HANDLING -----
        // assert : If condition is false revert state changes (Uses up gas) 
        // require : If condition is false provides option to return message (Refunds Gas)
        // revert : Only sends back a message if condition in if statement calls it

        require(_num2 != 0, "2nd Number can't be Zero");

        // Assert will cancel execution also if the condition
        // isn't true
        assert(_num2 > 0);

        if(_num2 < 0){
            revert("2nd Number Must be Greater than 0");
        }

        // Assignment Operators : += -= *= /= %=
        // Increment : ++ Decrement : --
        int _add = _num1 + _num2;
        int _sub = _num1 - _num2;
        int _mult = _num1 * _num2;
        int _div = _num1 / _num2; // Integer division
        int _mod = _num1 % _num2;
        int _sqr = _num1 ** 2;
        return (_add, _sub, _mult, _div, _mod, _sqr);
    }

    // ----- GENERATE RANDOM NUMBER -----
    
    function getRandNum(uint _max) public view returns(uint){
        // Generate a pseudo-random hexadecimal using the hash function
        // keccak256 (K Chak) which takes an input and converts it into a random
        // 256 bit hexadecimal number
        // Perform packed encoding of the data before using it to generate 
        // the random value
        uint rand = uint(keccak256(abi.encodePacked(block.timestamp)));

        // Takes result and returns values up to _max
        return rand % _max;
    }

    // ----- STRINGS -----
    // Printable ASCIII characters between single or double quotes
    // 
    string str1 = "Hello";

    // There are escape characters
    // \n \\ \' \" \r \t \xNN (Hex Escape) \uNNNN (Unicode)
    
    // Memory states that you want to temporarily store this string
    // data (Memory is deleted after each function execution)
    // Storage stores data between function calls
    function combineString(string memory _str1, string memory _str2) public pure returns (string memory){
        // Concat the 2 strings passed
        return string(abi.encodePacked(_str1, " ", _str2));
    }

    // Working with bytes saves computations versus working with strings
    // This function returns the number of characters in the string
    function numChars(string memory _str1) public pure returns(uint){
        // Convert string to bytes
        bytes memory _byte1 = bytes(_str1);
        return _byte1.length;
    }

    // ----- CONDITIONALS -----
    // Comparison Operators : == != > < >= <=
    // Logical Operators : && || !
    uint age = 8;
    
    // We are stating to store the arguments in memory
    function whatSchool() public view returns (string memory){
        if (age < 5) {
            return "Stay Home";
        } else if (age >= 5 && age <= 6){
            return "Go to Kindergarten";
        } else if (age >= 6 && age <= 17){
            uint _grade = age - 5;

            // Convert uint into a string
            string memory _gradeStr = Strings.toString(_grade);

            // Concat strings
            return string(abi.encodePacked("Grade ",_gradeStr));
        } else {
            return "Go to College";
        }
    }

    // ----- ARRAYS -----
    // Create an array that holds a dynamic number of values
    uint[] arr1;

    // Create fixed size array
    uint[10] arr2;

    // This creates an array of uints 5 in length
    uint [] public numList = [1,2,3,4,5];

    // Add 1, 2, 3, 4 to array
    function addToArray(uint num) public {
        // Add to end of array
        arr1.push(num);
    }

    // Array is now 1, 2, 3
    function removeFromArray() public {
        // Remove the last element
        arr1.pop();
    }

    // Length should be 3
    function getLength() public view returns (uint){
        // Get array length
        return arr1.length;
    }

    // If we pass 2 : 1, 2, 0
    function setIndexToZero(uint _index) public {
        // Sets value at specific index to 0
        // When you delete a value in an array the length 
        // stays the same
        delete arr1[_index];
    }

    // ----- FOR LOOP -----
    // Initialize starting index for loop
    // Define the loop length
    // How index value changes after each loop

    // If we pass 1 : 1, 0
    function removeIndex(uint _index) public {
        // Move the values up to replace the value to replace
        for(uint i = _index; i < arr1.length-1; i++){
            arr1[i] = arr1[i+1];
        }
        // Remove the last element
        arr1.pop();
    }

    // Get values in array
    function getArrayVals() public view returns (uint[] memory){
        return arr1;
    }

    // ----- FOR LOOP -----
    
    function sumNums() public view returns (uint){
        uint _sum = 0;
        
        for(uint i = 1; i <= numList.length; i++){
            _sum += numList[i];
        }
        return _sum;
    }

    // ----- WHILE LOOP -----
    function sumNums2() public view returns (uint){
        uint _i = 0;
        uint _sum = 0;
        while (_i < numList.length) {
            _sum += numList[_i];
            _i++;
        }
        return _sum;
    }

    // ----- STRUCTS -----
    // Create a data type that contains multiple variables

    struct Customer {
        string name;
        string custAddress;
        uint age;
    }

    Customer[] public customers;

    function addCust(string memory n, string memory ca, uint a) public {
        customers.push(Customer(n, ca, a));
    }

    function getCust(uint _index) public view returns (string memory n, string memory ca, uint a){
        Customer storage cust = customers[_index];
        return (cust.name, cust.custAddress, cust.age);
    }

    // ----- MAPPING -----
    // Allows you to create key / value pairs
    // The key can be a string, uint, or bool
    // Value can be anything

    mapping(string => string) public myMap;

    // Assigns key / value pair
    // If I add "Superman", "Clark Kent"
    function addSuper(string memory _secret, string memory _name) public {
        myMap[_secret] = _name;
    }

    // Returns value assigned to key
    // If sent "Superman" this returns "Clark Kent"
    function getName(string memory _secret) public view returns(string memory){
        return myMap[_secret];
    }

    function deleteName(string memory _secret) public {
        delete myMap[_secret];
    }

    // ----- STRUCTS & MAPPING -----
    mapping(uint => Customer) customer;

    // Map customer data to a index
    function addCust2(uint custID, string memory n, string memory ca, uint a) public {
        customer[custID] = Customer(n, ca, a);
    }

    // Retrieve customer data using an index
    function getCust2(uint _index) public view returns (string memory n, string memory ca, uint a)
    {    
        return (customer[_index].name, customer[_index].custAddress, customer[_index].age);
    }

    // ----- NESTED MAPPING -----
    // Maps inside maps
    // If you wanted a customer list for multiple businesses
    // The businesses would have a unique uint
    mapping(address => mapping(uint => Customer)) public myCusts;

    // Map customer data to different address
    function addMyCusts(uint custID, string memory n, string memory ca, uint a) public {
        // msg.sender is a global variable that is the address that is
        // calling the contract
        myCusts[msg.sender][custID] = Customer(n, ca, a);
    }

    // ----- DATE & TIME -----
    // Solidity has time units with the lowest unit at 1 second
    function timeUnits() public pure {
        // If any of these aren't true the function throws
        // an error
        assert(1 seconds == 1);
        assert(1 minutes == 60 seconds);
        assert(1 hours == 60 minutes);
        assert(1 days == 24 hours);
        assert(1 weeks == 7 days);
    }

    // ----- ENUMS -----
    // Enums are variables that can only have a limited number of values
    enum shirtSize{SMALL, MEDIUM, LARGE}

    // Create variable of type shirtSize
    shirtSize custSize;

    // Set a default size and mark it as constant
    shirtSize constant defaultSize = shirtSize.MEDIUM;

    function pickShirtSmall() public {
        custSize = shirtSize.SMALL;
    }

    function pickShirtMedium() public {
        custSize = shirtSize.MEDIUM;
    }

    // Get current size as a uint
    function getShirtSize() public view returns(shirtSize) {
        return custSize;
    }
}

// ----- SPECIAL VARIABLES -----

// Contracts are like objects of date and functions
// to manipulate that data
contract MyLedger {
    // Create a map of addresses and balances
    mapping(address => uint) public balances;

    // Change the balance for the address
    function changeBalance(uint newBal) public {
        // msg.sender is the sender of the message
        balances[msg.sender] = newBal;
    }

    // Get current balance for address
    function getBalance() public view returns (uint){
        return balances[msg.sender];
    }
}

// ----- FUNCTION MODIFIER -----

// Create a contract that will be inherited that contains a function modifier
// that blocks execution of certain functions unless the entity executing 
// those functions is the owner
contract Owner {

    // Holds the address for the owner who deployed this contract
    address owner;

    // Set the owner as that entity that created the contract
    constructor() public {
        owner = msg.sender;
    }

    // Create function modifier that will block anyone from changing
    // prices except for the owner
    modifier onlyOwner {
        // Requires this condition to be true or an error is thrown
        require(msg.sender == owner);

        // If the caller is the owner then continue executing the 
        // function that uses this function modifier
        _;
    }
}

// By inheriting from Owner we can restrict access to change prices 
contract Purchase is Owner {
    // Mapping that links addresses for purchasers
    mapping (address => bool) purchasers;

    uint price;

    constructor(uint _price) {
        price = _price;
    }

    // You can call this function along with some ether because of
    // payable
    function purchase() public payable {
        purchasers[msg.sender] = true;
    }

    // Only the owner can change this price
    function setPrice(uint _price) public onlyOwner {
        price = _price;
    }
}

// ----- WHAT IS GAS -----
// It is a fee charged for executing a contract
// on the Ethereum Virtual Machine 
// The cost depends on supply and demand. (Price is in Gwei)
// Gas Limit : Max amount of gas your willing to spend for a transaction

// ----- FALLBACK FUNCTIONS -----
// An anonymous function that doesn't take input nor provide output
// It executes if no other function matches the function identifier
// or if no data was provided with a function call
// It also executes when the contract receives Ether without data
// as long as it is marked Payable

contract FallbackTest {
    // Maps address to its balance
    mapping (address => uint) balance;

    // Event that logs gas
    event Log(uint gas);

    // Function shouldn't do much because it will fail if it uses 
    // to much gas. This keeps the Ether and emits a log
    fallback () external payable {
        emit Log(gasleft());
    }

    function getBalance() public view returns(uint) {
        return address(this).balance;
    }
}

// This contract sends Ether to FallbackTest contract
// 1. Deploy FallbackTest & TransferToFallback
// 2. Click copy next to FallvackTest to copy its address
// 3. Paste address into transferFallback
// 4. Change value to 2 Ether
// 5. Click transferFallback
// 6. Click getBalance to see the Ether was transferred
// 7. callFallback works the same
contract TransferToFallback {

    // Sends Ether with transfar method
    function transferFallback(address payable _target) public payable {
        _target.transfer(msg.value);
    }

    // Sends Ether with call method
    function callFallback(address payable _target) public payable {
        (bool sent,) = _target.call{value:msg.value}('');
        require(sent, 'FAILURE: Not Sent');
    }
}
```
