---
title: "Introduction to Solidity"
description: "some common introductory concepts of solidity"
tags:
  - "blockchain"
  - "solidity"
date: 2018-05-10T14:22:37+03:00
categories:
  - "code"
slug: "introduction-to-solidity"
draft: false
---

> A Blockchain is a distributed data store with append only operations, secured cryptographically. While this is just one of the definitions, you could think of it as a database, hosted in multiple nodes, which allows writing of data and viewing or querying, but never modifications. This type of structure is often referred to as _append-only_.

NOTE: A more detailed explanation for all the parts of a Blockchain can be found in the [Solidity Documentation](http://solidity.readthedocs.io/en/latest/introduction-to-smart-contracts.html#blockchain-basics)

Blockchain being a distributed data store can be used to solve some common problems we have with existing non-distributed systems. The existing solutions offered by non-distributed systems are not wrong, but they could be better.

A common example is three partners want to do business. Once they conclude their business and transactions are made, the records will be recorded in three different ledgers, one for each business. There's a couple of duplication going on here, but it makes sense since all these partners may not want any other partner to see their ledgers as it may contain sensitive data. The solution a distributed ledger provides is a digital way for all these partners to only have one ledger, which they can trust has the necessary integrity and security measures to ensure their records are safe.

There are so many blockchain platforms, and [Ethereum](https://www.ethereum.org/) is one of the well known ones. Code that is written and pushed to these blockchain platforms to ensure the above described behavior is possible are called Smart Contracts.

[Solidity](http://solidity.readthedocs.io/en/v0.4.23/) is the language used to write Smart Contracts for some common blockchain platforms, including [Ethereum](https://www.ethereum.org/), [Ethereum Classic](https://ethereumclassic.github.io/), [Qtum](https://qtum.org/en/), [Rootstock](https://www.rsk.co/), [Ubiq](https://ubiqsmart.com/). In this context, however, we will focus mainly on the Ethereum Blockchain.

An example of a use case for Solidity and Ethereum is crowdfunding. Since ether represents value in the Ethereum ecosystem, a contract can be written where users contribute ether, and the percentage they contribute towards a course is stored securely and publicly. This is what most Initial Coin Offerings do.

> An [Initial Coin Offering (ICO)](https://www.investopedia.com/terms/i/initial-coin-offering-ico.asp) is a means of raising funds for a new coin(often refers to as tokens). In a nutshell, you send a mature cryptocurrency with value, you get a new token without any value with the hope that it may gain value, or just to support a course. But the token represents the value/share you hold for this new project.

## What we'll cover

We will look at some of the commonly used language specifications and describe them using examples. If you have experience in any programming language, you will find most things familiar, with very subtle differences from those languages.

We'll cover language specification areas such as:-

* contracts
* static types
* functions
* patterns among others.

# Language Specification

Solidity as a programming language has it's own rules and specification on how it should be written for it to compile. It was inspired by other programming languages such as C++, Python and JavaScript, but it also has it's own uniqueness. The aim here is to see how to write valid Solidity Code, before we can start thinking of actual applications. Solidity files have a _.sol_ extension.

## Language Structure

Solidity is a Contract-Oriented programming language built specifically to work in the Ethereum Virtual Machine. To explain this further, the Ethereum Blockchain Network can be thought of as a network of computers, often called nodes, and within this network there's an actual virtual machine that enables secure execution of contract code. Hence the name Ethereum Virtual Machine.

Solidity being a Contract Oriented programming language means the highest language construct is a Contract. Here's an example.

#### example.sol

{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract MyToken {

    /_ Declare Variables Here _/
    uint data;

    /* Constructor initializes the Token */
    constructor() public { }

    /* Functions */
    function transfer() public {}

}
{{< / highlight >}}

Every solidity program must start with a pragma solidity to instruct the solidity compiler on which version should be used during the compilation. In the image 1.1 above, the version is 0.4.20 and above.

> The caret symbol "^" is used with respect to semantic versioning, which in this case means use version 0.4.20 up to just before 0.5.0. Learn more about semantic versioning https://semver.org/

Everything is then enclosed in a Contract called myToken.

In Object Oriented Programming, a class is usually a collection of member variable and methods, where the variables represent state, while the methods are used to execute code that usually modify the state. The same concept applies to Solidity.

A Contract is mainly collection of state variables and methods. In addition to these, _function modifiers_, _events_, _enums_ and _struct types_ can also be found inside a contract. We'll look at them in detail in the contract section. In the above example the state variable is _uint data_, and the methods are _constructor_ and _transfer_.

> A constructor is usually called when an instance is created and is therefore used to instantiate most of the state variables in a contract

### Structure of a Contract

A Contract is the highest language construct in Solidity. Every piece of code that is written in Solidity is enclosed within a Contract. We'll cover the following.

* State Variables
* Functions
* Function Modifiers
* Events

#### 1. State Variables

They usually consist of a variable name and a data type, and they permanently store values in the contract.

{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract MyToken {

    address public owner; // an address type
    bool public tokenExpired; // a boolean type

}
{{< / highlight >}}

Above, **_owner_** is a variable of type address, and **_tokenExpired_** is another variable of type boolean _(true or false)_. They both have the key word _public_ which is an access modifier and specifies the accessibility of the variables outside the contract.

#### 2. Functions

Functions are used execute code within the contract. Most functions usually modify state variables, and others usually help by doing calculations. They take the general form of functions, taking in parameters and returning a given value or nothing at all. They must have a visibility modifier, either private or public.
{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract MyToken {

    function giveTokens(address to, uint amount) public {
        // code to give tokens goes here
    }

}
{{< / highlight >}}

Functions can take any number of parameters, which in turn can be used to as variables for whatever code execution that needs to be done.

#### 3. Function Modifiers

Function modifiers change the behavior of functions. They are used to put in conditions that will determine how a function is executed. A common use case is having a condition which determines whether a function executes or exits immediately.

{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract MyToken {

    address public owner;

    modifier onlyOwner() { // Modifier
        require(msg.sender == owner);
        _;
    }

    function mint() public onlyOwner { // Modifier usage
        //
    }

}
{{< / highlight >}}

We see a modifier called onlyOwner which makes sure that whoever is executing the code is the owner of the contract.

> _msg_ is a built-in object that is automatically constructed during a transaction in Ethereum. msg.sender here refers to the address of the person/user executing the contract function.

#### 4. Events

Events are specific code that records when particular actions occur inside the contract. You can think of them as logs. Most event code is usually called when there is a state modification, or when a particular behavior fails.
{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract MyToken {

    Event MintOccurred(address from, uint amount); // Event

    function mint(uint amount) public {
        // mint code goes here
        emit MintOccurred(msg.sender, amount);
    }

}
{{< / highlight >}}

Above is an event called MintOccurred which is called every time the mint function is executed successfully.

In addition to the above mentioned parts of a contract, there are **_enums_** and **_struct types_**, which are both ways to create custom or user defined types.

An example of a Contract with all the above-mentioned code is shown below

##### myToken.sol

{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract MyToken {

    struct TokenHolder {
        bytes32 name;
    }

    address public owner;

    event MintOccurred(address from, uint amount); // Event

    constructor() public {
        owner = msg.sender;
    }

    modifier onlyOwner() {
        require(msg.sender == owner);
        _;
    }

    function mint(uint amount) public onlyOwner { // Modifier usage
        // mint code goes here
        emit MintOccurred(msg.sender, amount); // Emit Event
    }

    function transfer(address to, uint amount) public {
        // transfer code goes here
    }

}

{{< / highlight >}}

### Data Types

Since Solidity is a statically typed language, the type of each declared state variable needs to be specified at the time it's declared. If not specified, this value can be deduced from the value provided.

Following are some of the common data types that are available in Solidity

<table>
  <thead>
    <tr>
      <th width="25%">Data Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>booleans</td><td>represents two values: true and false</td>
    </tr>
     <tr>
      <td>integers</td><td>represents whole numbers, with signs and bits. e.g, unsigned integers with 8 bits are represented as uint8. Based on this logic, we have int, uint, int8, uint8, int256, unit 256 ..</td>
    </tr>
     <tr>
      <td>fixed point numbers </td><td>Represent numbers with decimal points. fixed/ufixed. </td>
    </tr>
     <tr>
      <td>string literals</td><td>usually represented within single or double quotes </td>
    </tr>
     <tr>
      <td>address</td><td>represents a 20 byte Ethereum address. e.g 0x49ee5c10079d11a6e599d7e0543a3b9a77b124dd </td>
    </tr>
    <tr>
      <td>enums</td><td>used to create custom types</td>
    </tr>
     <tr>
      <td>function types </td><td>A function can be assigned to a variable, making the variable's type a function.</td>
    </tr>
     <tr>
      <td>arrays </td><td>usually consists of a collection of given elements of the same type</td>
    </tr>
     <tr>
      <td>structs</td><td>a way to represent custom types that have their own properties. You can think of them as objects</td>
    </tr>
     <tr>
      <td>mappings</td><td>represents key value pairs </td>
    </tr>
     <tr>
      <td>Fixed byte-size array </td><td>Represents a collection of bytes. E.g bytes1, bytes2... bytes32.  </td>
    </tr>
  </tbody>
</table>

### Control Structures

Solidity has the following control structures: _if_, _else_, _while_, _do_, _for_, _break_, _continue_, _return_ and the _ternary operator_. Here's an example.

#### if else statements

These are used to check that a certain condition is satisfied before a section of the code is executed. Image 1.7 below shows an if statement that checks whether the balance of a given address is greater than 0, and adds a given amount to the balance, and if not, ends the execution of the function with a return keyword.
{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract MyToken {

    // Declare variables here
    mapping(address => uint) public balances;

    // addOne function
    function addAmount(address to, uint amount) public {
        if (balances[to] > 0) {
            balances[to] = balances[to] + amount;
        } else {
          return;
        }
    }

}
{{< / highlight >}}

#### while, do, for, break

Below shows examples of loop structures. Below are three variations of the same loop, all adding 1 ten times to the balance of a given address.
{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract MyToken {
    /_ Declare Variables Here _/
    mapping(address => uint) public balances;

    /* addOne using a while loop */
    function addOneWhile(address to) public {
        uint i = 0;
        while (i < 10) {
            balances[to] = balances[to] + 1;
            i++;
        }
    }

    /* add one using a Do Loop*/
    function addOneDo(address to) public {
        uint i = 0;
        do {
            balances[to] = balances[to] + 1;
            i++;
        } while (i < 10);
    }

    /* add one using a for loop */
    function addOneFor(address to) public {
        for (uint i = 0; i < 10; i++) {
            balances[to] = balances[to] + 1;
        }
    }

}

{{< / highlight >}}

The code above is strictly to show how loops are created, but this example could easily be written by assigning 10 directly to the address.

> A do-while loop ensures a loop runs at least once, unlike the for and while loops.

We'll end the crash course with an example/scenario.

### Example

We're going to write a Smart Contract that covers most of the topics we have covered above. We will run it in the next article.

#### Scenario

A car exchange contract has the following requirements.

* Cars have the following properties: make, year,
* Users can add their own cars (hint: users can have more than one car)
* Owners of a car can transfer ownership of the car to another user
* We should be able to get a car of a user with one function, which will take in an address and a key.

Try writing the above Smart Contract. The solution can be found in [this gist](https://gist.github.com/gangachris/2337e3f526f58f1d33a9e7b074a42ad1). Leave a comment in the gist for questions.
