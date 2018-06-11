---
title: "A Look into Remix IDE"
description: "Getting familiar with the Online Solidity IDE - Remix"
tags:
  - "blockchain"
  - "solidity"
  - "remix"
categories:
  - "code"
slug: "a-look-into-remix-ide"
date: 2018-06-11T16:48:42+03:00
draft: false
---

We've looked at some common things in the Solidity Language Specification in the [previous article](https://gangachris.com/posts/introduction-to-solidity/). We, however, have did not run any code, so we have no idea whether any of the code we've written works.

We could install the developer tools for writing Solidity into our workstations, but that would require a lot of time initially. When you want to quickly start writing and testing code, the easiest way to do it is using an online IDE.

Remix is an online solidity IDE accessible through the URL https://remix.ethereum.org.

![remix ide](/remix-ide-01.png)

Visiting the URL https://remix.ethereum.org gives you the view as shown in screenshot above. The Remix IDE has the following features.

  - Writing and Testing Smart Contracts in Solidity
  - Linting Solidity Code
  - Debugging Smart Contracts
  - Debug Committed transactions

We will look at all these features individually, then inspect the [Cars Smart Contract](https://gist.github.com/gangachris/2337e3f526f58f1d33a9e7b074a42ad1) we provided as a solution in the [preceding article](https://gangachris.com/posts/introduction-to-solidity/).

## 1. Writing and Testing Smart Contracts in Solidity
Once you open the remix IDE, you will be presented with the following.

  - File Explorer on the left pane.
  - Code Editor on the middle pane.
  - Solidity Tools on the right pane, divided into different tabs.

The example provided has two directories, browser and config. We are going to delete the file in the browser directory `ballot.sol` and put in our own simple contract, let's call it `myToken.sol`, with the following:

{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract MyToken {
    // total token supply
    uint totalSupply;

    /* constructor: initialize the initial supply ¯\_(ツ)_/¯ */
    constructor(uint total) public {
        totalSupply = total;
    }

    /* mint additional tokens */
    function mint(uint n) public {
        totalSupply = totalSupply + n;
    }

    /* get the current total tokens*/
    function get() public constant returns (uint) {
        return totalSupply;
    }
}
{{< / highlight >}}

In this smart contract, we have a state variable `totalSupply`, which represents the total tokens. We then have a constructor which will instantiate the total number of tokens. There's a function `mint` which represents the idea of adding more tokens, the a getter function used to return the value of total supply.

> A getter function is usually used to retrieve a particular state variable, instead of accessing it directly.

By now, your Remix IDE should looks something similar to this.

![](/remix-ide-02.png)

If you click on the annotated compile button, you should see a green alert section written ***MyToken***, meaning the compilation was successful.

To see the smart contract in action, go to the ***Run*** tab in the right pane. You should see something similar to this.

![](/remix-run-tab.png)

We've prefilled some of the items, but we'll explain them in detail below.

  - ***Environment*** – This represents an instance of a running Ethereum network. The three available options are *JavaScript VM*, *Injected Web3*,* Web 3 provider*. We'll use the JavaScript VM, which builds a virtual network for quickly testing out our contract. It's also significantly faster than all other options.
  - ***Account*** – Here you will find the accounts available on the environment you choose. They are all Ethereum addresses. Ours should look something similar to the image below with each account holding 100 ether

  ![](/ether-addresses.png)

  - Gas limit – We will most likely leave this as is. Whenever an Ethereum transaction occurs, there is some amount of ether consumed. This is done to regulate the processing of smart contracts. The ether consumed during these transactions is called Gas.

> The purpose of gas is to limit the amount of work that is needed to execute the transaction and to pay for this execution. This controls how Smart Contract developer write their code, because it means heavy execution will cost them.

Below all these, you will see a section with a drop down, which represents the name of the contract we wrote. In this case, this is `MyToken`.

![](/create-token.png)

Below the contract name, you will see an input field with a Create button. This corresponds to the constructor that we wrote in our Smart Contract:-

{{< highlight solidity >}}
pragma solidity ^0.4.20;
contract MyToken {
    ----

    /* constructor: initialize the initial supply ¯\_(ツ)_/¯ */
    constructor(uint total) public {
        totalSupply = total;
    }

    ----
}
{{< / highlight >}}

Our constructor took in one argument of type `uint`, and this is what is required.

Put in the value 200 in the input field and click on `Create`.

The second annotation with `mint` and `get` represents the public functions that we have in our contract.

If you click on `Get`, you will get the value 200, as shown below.

![](/token-get.png)

To test the mint function, we can input value 300 into the text field next to the mint button, and then press on `mint`.

Clicking on `get` again returns a value of `200 + 300`  which is `500`.

![](/mint-token.png)

Notice also that every action is logged on the terminal below the editor.

![](/remix-terminal.png)

## 2. Linting Solidity Code
Linting code refers to applying rules to a given piece of code to ensure it follows best practices and doesn't have easily fixable syntax or compile time errors. Remix IDE helps us with this while writing solidity code by giving us clear warnings.

Change the Solidity Code we wrote in the Remix IDE to this:-
{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract MyToken {
    // total token supply
    uint totalSupply

    /* constructor: initialize the initial supply ¯\_(ツ)_/¯ */
    constructor(uint total) {
        totalSupply = total;
    }

    /* mint additional tokens */
    function mint(uint n) public {
        totalSupply = totalSupply + n;
    }

    /* get the current total tokens*/
    function get() public constant returns (uint) {
        return totalSupply;
    }
}
{{< / highlight >}}
The code has some syntax errors. Can you figure them out?

Pressing compile in the `Compile` Tab, results in the following screen.

![](/remix-syntax-error.png)

On further inspection, we see an error message expected token SemiColon got Function...

![](/remix-error.png)

We immediately can tell what the error is and fix it.

## 3. Debugging Smart Contracts
Debugging refers to the process of finding out the root cause of bugs in a system. If you expect a given output from a program, and get a different output, you will investigate what's wrong with your code. This process is called debugging.

Let's change the logic of our Smart Contract a bit so that the mint function has a bug. Instead of adding to the existing total supply, it replaces the existing total supply.
{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract MyToken {
    ---

    /* mint additional tokens */
    function mint(uint n) public {
        totalSupply = n;
    }

    ----
}
{{< / highlight >}}

We can then compile this and test it the way we did in the first section.

  1. Press `Start Compile` in the `compile` tab.
  2. Go to the `Run tab`, and ensure `JavaScript VM` is selected in the environment.
  3. Put in the value 200 in the text field next to `Create` Button.
  4. Click on `Create`.

Put in a value of 300 in the `mint` section and click on `mint`

At this point, if you click on the get function at the bottom, you should get a value of 300, as shown below

![](/remix-debug-01.png)

You'll notice that we have two contracts now deployed, the original one we did in part 1 – Writing and Testing Smart Contracts, and the second one, which we are using.

When you clicked on `Get`, you will see that the value is now `300` instead of the expected `500 (200 + 300)`.

Here's where debugging comes into play.

On the terminal below the Code editor, you will see a log of the action that was called when the mint function was called. It should be the second last one. You'll notice it's title corresponds to the mint function:- `transaction to MyToken.mint …`

![](/remix-debug-02.png)

Clicking on the details will show you the parameters that went into making that transaction.

![](/remix-debug-03.png)

To explain some of the parameters:
  1. ***From*** – this refers to the Ethereum address running the transaction. You will notice it is similar to the selected account in the Account section of the run tab.
  2. ***To*** – refers to the address of the Smart Contract that we deployed.
  3. ***Transaction*** cost and Execution cost – this is the gas consumed when the transaction was processed. We covered the importance of gas earlier.
  4. ***Hash*** – every transaction in the Ethereum network usually has a unique hash that can be used to retrieve the transaction.
  5. ***Input*** – an encoded representation of the input parameters passed. See decoded input in the preceding image.

Next click on debug, so that we can find out what's wrong with the code. The right pane will open the debugger tab, and select the code that executed in the code editor.

![](/remix-debug-04.png)

We've annotated the important sections. The most important sections for this article are:

  - ***Transaction*** – This will enable us step through the lines of code that resulted in a given state.
  - ***Solidity Locals*** – This refers to the state variables of the current function being executed. It currently has the value 300 since we passed that value to the mint function previously.
  - ***Solidity State*** – This refers to the global solidity contract state and will list the variables and values. It currently has totalSupply: 200, which was the initial value we instantiated the MyToken Contract with.


Next, press the Step Over button

![](/remix-step-over.png)

> While debugging code, there are usually four important buttons, *Step Over*, *Step Into*, *Step Out* of and *Step*. These usually enable a debugger to go navigate code the way we want, and inspect the variables to see what's happening at a particular point in execution. If you hover over the buttons, you will see the debug buttons, you will see a hover text showing the names.

To run the get the function past the current state. You will see the following results.

![](/remix-debug-05.png)

We see now that the global state has changed to `totalSupply: 300`. Which is equivalent to the value we passed. We can therefore deduce that this is an assignment, and not an increment. This piece of code is fixed by simple changing `totalSupply = n`, back to `totalSupply = totalSupply + n.

This demonstrates the Remix's IDE debugging capabilities.

## Other Remix IDE features
The following are additional Features of the Remix IDE

  - ***Settings*** - this can be found in the right pane in it's own tab and allows customization of the solidity IDE. Examples of customizations that can be done include:
    - Changing compiler version of solidity
    - Changing the theme of the Remix IDE
    - Adding plugins to the Remix IDE

  - Analysis – this can be found in the right pan on it's own tab and allows some statistics to be enabled so that they can be tracked when running transactions.

## Local Installation
We're going to install a local copy of the Remix IDE and test the [Cars contract](https://gist.github.com/gangachris/2337e3f526f58f1d33a9e7b074a42ad1)

Make sure you have Nodejs installed by running node –v in your terminal. Version 6 or higher is recommended.

{{< highlight bash >}}
node -v
v10.4.0
{{< / highlight >}}

Then install remix-ide globally with `npm install -g remix-ide` in your terminal.
{{< highlight bash >}}
npm install -g remix-ide
{{< / highlight >}}

Once the command completes without errors, run the local instance of Remix ide by typing remix-ide in your terminal.
{{< highlight bash >}}
remix-ide

setup notifications for <your-current-directory>
Shared folder : <your-current-directory>
Starting Remix IDE at http://localhost:8080 and sharing <your-current-directory>
Mon Jun 11 2018 18:33:23 GMT+0300 (East Africa Time) Remixd is listening on 127.0.0.1:65520

{{< / highlight >}}

In the image above, you will notice the <your-current-directory> points to the directory in which you ran the command in. We are also informed that the remix IDE is running on localhost:8080. Visit the URL localhost:8080, and you will see a figure similar to the one below, with the following smart contract code.
{{< highlight solidity >}}
pragma solidity ^0.4.20;

contract Cars {
    // we declare a custom type car
    struct Car {
        bytes32 make;
        uint year;
    }

    // we create a map that takes ethereum address and maps them to a Cars array
    mapping(address => Car[]) public carOwners;

    function registerCar(bytes32 _make, uint yr) public {
        // assign new struct
        // assign the car
        carOwners[msg.sender].push(Car({
            make: _make,
            year: yr
        }));
    }

    // change car ownership by providing owner and index
    function changeOwnership(address toOwner, uint index) public returns (bool) {
        // check if sender has cars
        // can be refactored to a function modifier
        if (carOwners[msg.sender].length == 0) {
            return false;
        }

        if (carOwners[msg.sender].length > index+1) {
            return false;
        }

        carOwners[toOwner].push(carOwners[msg.sender][index]);

        // TODO: remove the car from the current owner, otherwise there's a duplicate.
    }

    // get a car by providing owner and index
    function getCarMake(address owner, uint index) public view returns (bytes32 carMake) {
        if (carOwners[owner].length == 0 ){
            return;
        }

        if (carOwners[owner].length > index+1) {
            return;
        }

        carMake = carOwners[owner][index].make;
    }
}
{{< / highlight >}}

Below is the local remix screenshot

![](/remix-local.png)


Above we see that we have pasted the contract we built in the previous article. Inspect it with the tools we discussed in the section Writing and Testing Smart Contracts.

Once you click on Start Compile, and get a green build, you can go to the `run` tab, select `JavaScript VM` as the environment, the click on the `Create` button. You'll notice the Create text field is disabled, and this is because we do not have a constructor in our Smart Contract.

Next we can call all the functions of the Smart Contract. Do the following tasks.

  1. registerCar takes in a `_make` and `yr` of type `bytes32` and `uint` yr. Pass in the following variables
    - An array of bytes representing "toyota" i.e [116,111,121,111,116,97].
    - Year: 2015

Once you click the `registerCar` button, click on debug in the terminal, and inspect the Solidity Locals and Solidity State like we did in the article.

See Ya!!
