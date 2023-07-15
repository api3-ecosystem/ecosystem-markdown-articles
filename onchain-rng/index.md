---
title: Create a Random Generated Number on-chain using API3 Tools for Free!
subtitle: An Intro tutorial on how to send requests for random numbers and utilize them on Remix
author: Billy Jitsu
date_published: 2022-08-31
category: Technology
image: "https://cdn.hashnode.com/res/hashnode/image/upload/v1661875762692/d3pSBAo-g.jpg?w=1600&h=840&fit=crop&crop=entropy&auto=compress,format&format=png"
---

In this tutorial, we will be generating a random number utilizing the QRNG library. QRNG is a partnership project of API3 and the Australian National University (ANU) Quantum Optics Group. It is free to use for developers. You just have to pay for gas.

By the end of this tutorial, you'll be able to create a real random number based on quantum mechanics.


## The Tech Stack

-   Smart Contract Language: Solidity
-   Smart Contract Development Environment: Remix
-   Smart Contract Deploy: Injected Connector - Metamask

## Prerequisites

There are no prerequisites; you will do the tutorial on Remix so you can quickly jump in and start experimenting with the code and random numbers

Why do we even need random numbers on-chain?

The blockchain is deterministic by nature, so miners would be able to know what the outcome would be before mining a transaction and can manipulate the outcome of a transaction. Imagine if you knew the results of flipping a coin each time it happened. By adding a bit of randomness, it adds a chance for all users of your dapp to have a fair chance at winning a prize.

## How Does it Work

The process happens in two transactions. The contract calls for a request from the API on the first transaction. Then on the 2nd transaction on another sponsored contract, the random number is generated and returned to the original contract.

This is different from the prevalent `block.difficulty + block.timestamp` technique, which scammers can easily game. With our technique, returning the generated number generation takes some time.

Below is the final code so that you can test it on Remix.

## Final Code

We pulled from the API3 example base with a few modifications for ease. You can copy and paste most of the code since the setup of the call requests and the returns are the same for nearly every contract. You can implement custom logic in specific functions.

```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
import "@api3/airnode-protocol/contracts/rrp/requesters/RrpRequesterV0.sol";

contract QrngExample is RrpRequesterV0 {
    event RequestedUint256(bytes32 indexed requestId);
    event ReceivedUint256(bytes32 indexed requestId, uint256 response);

    //set the owner of the contract (can use OZ library too)
    address public owner;
    // These can be set using setRequestParameters())
    address public airnode;
    bytes32 public endpointIdUint256;
    address public sponsorWallet;

    uint256 public randomNumberReturn;

    mapping(bytes32 => bool) public expectingRequestWithIdToBeFulfilled;

    constructor(address _airnodeRrp) RrpRequesterV0(_airnodeRrp) {
        owner = msg.sender;
    }

    // Set parameters used by airnodeRrp.makeFullRequest(...)
    // See makeRequestUint256()
    function setRequestParameters(
        address _airnode,
        bytes32 _endpointIdUint256,
        address _sponsorWallet
    ) external {
        // Must limit to owner or somebody can change parameters
        require(msg.sender == owner, "Sender not owner");
        airnode = _airnode;
        endpointIdUint256 = _endpointIdUint256;
        sponsorWallet = _sponsorWallet;
    }

    // Calls the AirnodeRrp contract with a request
    // airnodeRrp.makeFullRequest() returns a requestId to hold onto.
    function makeRequestUint256() external {
        bytes32 requestId = airnodeRrp.makeFullRequest(
            airnode,
            endpointIdUint256,
            address(this),
            sponsorWallet,
            address(this),
            this.fulfillUint256.selector,
            ""
        );
        // Store the requestId
        expectingRequestWithIdToBeFulfilled[requestId] = true;
        emit RequestedUint256(requestId);
    }

    // AirnodeRrp will call back with a response
    function fulfillUint256(bytes32 requestId, bytes calldata data)
        external
        onlyAirnodeRrp
    {
        // Verify the requestId exists
        require(
            expectingRequestWithIdToBeFulfilled[requestId],
            "Request ID not known"
        );
        expectingRequestWithIdToBeFulfilled[requestId] = false;
        uint256 qrngUint256 = abi.decode(data, (uint256));
        // Do what you want with `qrngUint256` here...

        randomNumberReturn = qrngUint256 % 25;
        emit ReceivedUint256(requestId, qrngUint256);
    }

    function getRandom() public view returns (uint256) {
        return randomNumberReturn;
    }
}

```

## Function by Function

```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
import "@api3/airnode-protocol/contracts/rrp/requesters/RrpRequesterV0.sol";

```

We are importing the API3 library that helps us bring in the functions that make our calls for random numbers much simpler. Bringing in `makeFullRequest` call in the makeRequestUint256() function.

```solidity
    event RequestedUint256(bytes32 indexed requestId);
    event ReceivedUint256(bytes32 indexed requestId, uint256 response)

    //set the owner of the contract (can use OZ library too)
    address public owner;
    // These can be set using setRequestParameters()
    address public airnode;
    bytes32 public endpointIdUint256;
    address public sponsorWallet;

    uint256 public randomNumberReturn;

    mapping(bytes32 => bool) public expectingRequestWithIdToBeFulfilled;

```

Here we are declaring our variables:

-   Events are for broadcasting requests and receives from the contract
-   `airnode`, `endpointIdUint256`, and `sponsorWallet` are all variables you will set to set up the request parameters required by the API3 node.
-   Owner is a declaration of ownership to limit access to certain functions. We could easily use OpenZepplin's `Ownable.sol` library as well.
-   randomNumberReturn is a variable I put to easy display the value of the current number
-   Mapping is a requirement of the API3 request. For the node to return a random number, someone must have sent a request for it. This will check if that is true or not.

```solidity
constructor(address _airnodeRrp) RrpRequesterV0(_airnodeRrp) {
    owner = msg.sender;
}

```

Inside the constructor itself, we declare that the individual who deploys the contract is the owner of this contract (i.e., `msg.sender`).

The constructor only takes in one argument, the Airnode contract address. Depending on the chain you are deploying to, there may be a difference in address per chain.

[Chains | Documentation](https://docs.api3.org/qrng/reference/chains.html)

> Note each chain has different minimum confirmation requirements (how long it takes to receive a number once requested)

![Minimum confirmations - final.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1661875970829/Ibcsik-uE.jpg?auto=compress,format&format=webp)

For this demo we will be using Goerli (Chain ID 5) so we will be using this address on deployment: `0xa0AD79D995DdeeB18a14eAef56A549A04e3Aa1Bd`

```solidity
// Set parameters used by airnodeRrp.makeFullRequest(...)
    function setRequestParameters(
                address _airnode,
        bytes32 _endpointIdUint256,
        address _sponsorWallet
    ) external {
        // Must limit to owner or somebody can change parameters
        require(msg.sender == owner, "Sender not owner");
        airnode = _airnode;
        endpointIdUint256 = _endpointIdUint256;
        sponsorWallet = _sponsorWallet;
    }

```

You use the `setRequestParameters` function when you deploy this contract on the chain of your choice (for this example, Goerli).

We will be declaring our specific airnode, endpoint, and sponsor wallet.

We get the Airnode and `EndpointIdUint256` address from this portion of the API3 docs:

[API Providers | Documentation](https://docs.api3.org/qrng/reference/providers.html)

At the time of writing this tutorial, there is one global Airnode contract to call has the following address: `0x9d3C147cA16DB954873A498e0af5852AB39139f2`

You will notice in the documents that there are two types of `endpointIdUint256` to return a number and `endpointIdUint256Array` to return an array of numbers.

For this demo, we only need one number, so we are using `endpointIdUint256`.

`0xfb6d017bb87991b7495f563db3c8cf59ff87b09781947bb1e417006ad7f55a78`

We will get to the sponsor wallet once we have deployed the contract. We need to generate the sponsor wallet using the deployed contract address, Airnode address, and the xpub address you saw in the docs. Do not worry about this now; we will come back to this later.

```solidity
// Calls the AirnodeRrp contract with a request
// airnodeRrp.makeFullRequest() returns a requestId to hold onto.
    function makeRequestUint256() external {
        bytes32 requestId = airnodeRrp.makeFullRequest(
            airnode,
            endpointIdUint256,
            address(this),
            sponsorWallet,
            address(this),
            this.fulfillUint256.selector,
            ""
        );
        // Store the requestId
        expectingRequestWithIdToBeFulfilled[requestId] = true;
        emit RequestedUint256(requestId);
    }

```

The function `makeRequestUint256` actually makes the call to the sponsored wallet to generate the random number. You see, it takes in `airnode`, `endpointIdUint256`, our deployed contract address, and the `sponsorWallet` we specified in the previous function. This then calls `fulfillUint256.selector()` and stores the request to be True with an ID attached.

```
mapping(bytes32 => bool) public expectingRequestWithIdToBeFulfilled;

```

> Remember, we need a mapping of requests to make sure the node actually has a request to return a random number.

```solidity
// AirnodeRrp will call back with a response
    function fulfillUint256(bytes32 requestId, bytes calldata data)
        external
        onlyAirnodeRrp
    {
        // Verify the requestId exists
        require(
            expectingRequestWithIdToBeFulfilled[requestId],
            "Request ID not known"
        );
        expectingRequestWithIdToBeFulfilled[requestId] = false;
        uint256 qrngUint256 = abi.decode(data, (uint256));
        // Do what you want with `qrngUint256` here...
    // Your custom code here

        randomNumberReturn = qrngUint256 % 25;
        emit ReceivedUint256(requestId, qrngUint256);
    }

```

The `fulfillUint256` function is only called by the API3 contracts when it returns a value from the request made by `makeRequestUint256()` function. Once it calls, it makes sure that there was a request sent with the require and then sets its to false if it was true. The returned random number gets decoded to a local variable and set to `qrngUint256`. From here, we can use this random number we have received in our custom logic. For simplicity, I'm setting the random number with modular 25 (So it will be a number between 0 and 25) and setting it to the public variable `randomNumberReturn` so I can read it later.

> Up to this point, every contract you use to call for a random number using API3 will have the same setup. It isn't until you receive your random number do you add in your custom code to do what you want to do with your code.

```solidity
function getRandom() public view returns (uint256) {
    return randomNumberReturn;
}

```

This function will return the current random number received from the call request. Since the `randomNumberReturn` variable is public, you don't need it, but if you were trying to keep it private, this is an easy way to view it.

> This portion of the code is not required for the random number generator but makes it easier to view the number if the variable was declared hidden/private for your contract.

We have now covered the complete contract and are ready for deployment. Just to review:

-   created a function that sets up a sponsored wallet to generate a random number for us.
-   Made a function that will request a random number, created another function to receive the random number in which we can create custom code to do something with it
-   Finally, we have a function that returns the number we received with our modifications (0-25)

## Remix

Remix is a coding tool created by the Ethereum foundation that lets you jump in and code instantly without having to set up a coding environment. I use this tool to test out quick ideas and snippets of code on the fly.

## Deploying The Contract

Head over to [https://remix.ethereum.org/](https://remix.ethereum.org/), and it will set you up with a default workspace. It will have three examples of code that you can clear out. Go ahead and delete those and make a `Number.sol` file.

![Clear remix.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876095680/9gznUKSyG.gif?auto=format,compress&gif-q=60&format=webm)

I am reposting the final code so you can copy it into Remix. If you still do not feel comfortable with the code just yet, I suggest you write the code line by line and try to understand each of them alone.

```solidity
//SPDX-License-Identifier: MIT
pragma solidity 0.8.9;
import "@api3/airnode-protocol/contracts/rrp/requesters/RrpRequesterV0.sol";

contract QrngExample is RrpRequesterV0 {
    event RequestedUint256(bytes32 indexed requestId);
    event ReceivedUint256(bytes32 indexed requestId, uint256 response);

    //set the owner of the contract (can use OZ library too)
    address public owner;
    // These can be set using setRequestParameters())
    address public airnode;
    bytes32 public endpointIdUint256;
    address public sponsorWallet;

    uint256 public randomNumberReturn;

    mapping(bytes32 => bool) public expectingRequestWithIdToBeFulfilled;

    constructor(address _airnodeRrp) RrpRequesterV0(_airnodeRrp) {
        owner = msg.sender;
    }

    // Set parameters used by airnodeRrp.makeFullRequest(...)
    // See makeRequestUint256()
    function setRequestParameters(
        address _airnode,
        bytes32 _endpointIdUint256,
        address _sponsorWallet
    ) external {
        // Must limit to owner or somebody can change parameters
        require(msg.sender == owner, "Sender not owner");
        airnode = _airnode;
        endpointIdUint256 = _endpointIdUint256;
        sponsorWallet = _sponsorWallet;
    }

    // Calls the AirnodeRrp contract with a request
    // airnodeRrp.makeFullRequest() returns a requestId to hold onto.
    function makeRequestUint256() external {
        bytes32 requestId = airnodeRrp.makeFullRequest(
            airnode,
            endpointIdUint256,
            address(this),
            sponsorWallet,
            address(this),
            this.fulfillUint256.selector,
            ""
        );
        // Store the requestId
        expectingRequestWithIdToBeFulfilled[requestId] = true;
        emit RequestedUint256(requestId);
    }

    // AirnodeRrp will call back with a response
    function fulfillUint256(bytes32 requestId, bytes calldata data)
        external
        onlyAirnodeRrp
    {
        // Verify the requestId exists
        require(
            expectingRequestWithIdToBeFulfilled[requestId],
            "Request ID not known"
        );
        expectingRequestWithIdToBeFulfilled[requestId] = false;
        uint256 qrngUint256 = abi.decode(data, (uint256));
        // Do what you want with `qrngUint256` here...

        randomNumberReturn = qrngUint256 % 25;
        emit ReceivedUint256(requestId, qrngUint256);
    }

    function getRandom() public view returns (uint256) {
        return randomNumberReturn;
    }
}

```

Once we have finalized the code on Remix, we will want to compile our contract. The third icon on the right is our compiler options. Once we select it, we want to ensure our compiler matches our version of solidity we are using for our project (0.8.9). Once our settings are correct, we can click on "Compile Number.sol" (the blue button). There is also an option for "Auto compile," so Remix will compile any changes we make to the code automictically if you choose.

![compile.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876229887/eUZ7m5ob7.gif?auto=format,compress&gif-q=60&format=webm)

Now we are ready for deployment. The Ethereum Logo icon just underneath the compile button is what we will be using to set up our deployment.

On the top menu label "Environment," we want to click on the dropdown menu "Remix VM (London) and choose "Injected Provider - Metamask." This uses our wallet and the network we are connected to deploy.

We deployed on Goerli, so make sure you have Goerli selected and gas (GoerliETH).

> If you do not have any gas, you can get some here:

[https://goerlifaucet.com/](https://goerlifaucet.com/) or mine it here [https://goerli-faucet.pk910.de/](https://goerli-faucet.pk910.de/)

Under "Contract," make sure to click on the dropdown and select your contract, which in this tutorial is "QrngExample - contracts/Number.sol"

Once selected, deploying will require an argument which is the address for the Airnode of the chain we are using.

> Reminder: Goerli AirNodeRrpV0 is `0xa0AD79D995DdeeB18a14eAef56A549A04e3Aa1Bd`

[https://docs.api3.org/qrng/reference/chains.html](https://docs.api3.org/qrng/reference/chains.html)

![Deploy.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876292604/aFaDbJ9dY.gif?auto=format,compress&gif-q=60&format=webm)

Congrats, we have deployed our contract to Goerli testnet. Looking at the lower left of your remix interface, you will see "Deployed Contracts" with a Copy icon next to the name. This is the address of our deployed contract.

![show contract deployment.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876338076/6ERIMpp1H.gif?auto=format,compress&gif-q=60&format=webm)

> This tutorial’s deployed address on Goerli: `0x1C2608f21d15adf0dEb9Cd95007Df94560367134`

Looking at the function list, remember we have to set our Request Parameters:

```solidity
function setRequestParameters(
        address _airnode,
        bytes32 _endpointIdUint256,
        address _sponsorWallet
    ) external {
        // Must limit to owner or somebody can change parameters
        require(msg.sender == owner, "Sender not owner");
        airnode = _airnode;
        endpointIdUint256 = _endpointIdUint256;
        sponsorWallet = _sponsorWallet;
    }

```

> Reminder from [https://docs.api3.org/qrng/reference/providers.html](https://docs.api3.org/qrng/reference/providers.html)
> 
> Airnode: `0x9d3C147cA16DB954873A498e0af5852AB39139f2`
> 
> endpointIdUint256: `0xfb6d017bb87991b7495f563db3c8cf59ff87b09781947bb1e417006ad7f55a78`
> 
> xpub: `xpub6DXSDTZBd4aPVXnv6Q3SmnGUweFv6j24SK77W4qrSFuhGgi666awUiXakjXruUSCDQhhctVG7AQt67gMdaRAsDnDXv23bBRKsMWvRzo6kbf`

The tricky part is creating our sponsor wallet.

[Admin CLI | Documentation](https://docs.api3.org/airnode/v0.7/reference/packages/admin-cli.html#derive-sponsor-wallet-address)

![derive sponsor.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876387983/jn0mvT2nc.JPG?auto=compress,format&format=webp)

For our purposes, it would look this:

```shell
$ npx @api3/airnode-admin derive-sponsor-wallet-address \
--airnode-xpub xpub6DXSDTZBd4aPVXnv6Q3SmnGUweFv6j24SK77W4qrSFuhGgi666awUiXakjXruUSCDQhhctVG7AQt67gMdaRAsDnDXv23bBRKsMWvRzo6kbf \
--airnode-address 0x9d3C147cA16DB954873A498e0af5852AB39139f2 \
--sponsor-address <ADDRESS_OF_YOUR_DEPLOYED CONTRACT>

```

For this tutorial, the sponsor-address is: `0x1C2608f21d15adf0dEb9Cd95007Df94560367134`

You will have to open up the terminal (I recommend Powershell for Windows users) and enter the information provided (with your deployed contract information).

![generate sponsor wallet.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876524949/eD4gJeUM4.gif?auto=format,compress&gif-q=60&format=webm)

Once we run it, we receive our sponsor's address.

Sponsor wallet address: `0x0B4d2b2589c1891a12F1a02f194EED93CfA54fB4`

Now we can complete the setRequestParameters setup. We have the airnode, endpoint, and sponsor wallet.

> Airnode: `0x9d3C147cA16DB954873A498e0af5852AB39139f2`
> 
> endpointIdUint256: `0xfb6d017bb87991b7495f563db3c8cf59ff87b09781947bb1e417006ad7f55a78`
> 
> Sponsor wallet: `0x0B4d2b2589c1891a12F1a02f194EED93CfA54fB4`

![setRequest.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876549409/Sy15GnMjM.JPG?auto=compress,format&format=webp)

![generate sponsor wallet - transaction.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876559842/bmJvl3X7G.gif?auto=format,compress&gif-q=60&format=webm)

Our contract now has set parameters for doing transactions for our requests. Set this to **require only the owner** to do this function because we don't want anybody to change the parameters against our wishes.


## Very Important

One last thing we need to do before we start requesting random numbers from our contract. We need **to fund our contract with some gas money**. Wait! You said generating random numbers was free!! Yes, it is, but it costs gas to generate the numbers and requests, so we pass that responsibility to the sponsor wallet. The gas you spend is dependent on the chain you deploy. If it is on mainnet or testnets, it will take ETH. If it is on Gnosis Chain, it will take xDai as gas, etc. Anyone can fund the sponsor wallet by sending gas to the contract. The gas required to create the random number depends on the gas costs at the time of the call. If gas is 5 wei, it will use less gas than if it is 50 wei at the time.

Let's fund our Sponsor Wallet.

[https://goerli.etherscan.io/address/0x0B4d2b2589c1891a12F1a02f194EED93CfA54fB4](https://goerli.etherscan.io/address/0x0B4d2b2589c1891a12F1a02f194EED93CfA54fB4)

![empty sponsor.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876626452/FY5xGSMz1.JPG?auto=compress,format&format=webp)

Currently sitting empty with no gas, we would get no return response if we tried to make a request. Nothing would happen.

Send a small amount of eth to get started (sending to the contract address) :

![send eth.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876643649/_fDKHOVWs.JPG?auto=compress,format&format=webp)

![funded sponsor.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876655591/hcSjAEMST.JPG?auto=compress,format&format=webp)

Now the contract is funded and ready to make random number requests!

We have to call `makeRequestUint256()` and let the contract do the rest. Remember that generating a valid random number requires two transactions. One to make the request, another for the sponsor wallet to generate and return the number to the contract. We will start with are `getRandom` function with 0 and see what number we will get (remember it was modified to come back with 0-25)

![request random number.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876681680/zzhYyspqV.gif?auto=format,compress&gif-q=60&format=webm)

We request the number, but after the transaction completes, you see we still have a value of 0. We must wait for the sponsored wallet to generate and return the number.

![returned number.gif](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876700841/khwTmDeSj.gif?auto=format,compress&gif-q=60&format=webm)

After a few moments, we get our number returned, and now we have a new Quantum-generated random number. `22`

So what happened behind the scenes? Let's look at the contracts.

Our deployed contract: [https://goerli.etherscan.io/address/0x1c2608f21d15adf0deb9cd95007df94560367134](https://goerli.etherscan.io/address/0x1c2608f21d15adf0deb9cd95007df94560367134)

![make request hash.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876720538/1mNqHp6Fd.JPG?auto=compress,format&format=webp)

We called "Make RequestUint256" at block `7484990`

At our sponsor wallet contract:

[https://goerli.etherscan.io/address/0x0B4d2b2589c1891a12F1a02f194EED93CfA54fB4](https://goerli.etherscan.io/address/0x0B4d2b2589c1891a12F1a02f194EED93CfA54fB4)

![fulfill hash.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876736046/29Yv7SsEW.JPG?auto=compress,format&format=webp)

The sponsor contract fulfills our request at block `7484992`

**Two transaction blocks later** we receive our random number back at our contract, and we see that some gas that we funded the contract is used (Originally 0.01 Ether)

![new balance after.JPG](https://cdn.hashnode.com/res/hashnode/image/upload/v1661876754803/f6zILvzK5.JPG?auto=compress,format&format=webp)

## Congratulations!

You now have gone through the process of generating a random number for FREE that can support multiple chains across the ecosystem! Now you can use that random number for many cases—generated stats for a gaming NFT character or a lottery-type game. Get creative, and we look forward to seeing what you ship. If you want to learn more about what API3 is doing, please check out their site and documents at: [API3](https://api3.org/)