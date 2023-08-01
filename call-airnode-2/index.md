---
title: How to Call Any API From a Solidity Smart Contract, Part II
subtitle: Making and deploying a Requester contract to request data from the Airnode
author: Vansh Wassan
date_published: 2022-09-14
category: Technology
image: "https://miro.medium.com/v2/resize:fit:4800/0*crNHRAgjWh9LoIFX"
---

_Photo by [GuerrillaBuzz Crypto PR](https://unsplash.com/@theshubhamdhage?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)_

This is the second part of the tutorial where we deploy the Requester Contract and request data from the [Airnode](https://docs.api3.org/explore/airnode/what-is-airnode.html). Check out the first part [here](https://betterprogramming.pub/how-to-call-any-api-from-a-solidity-smart-contract-part-i-d1d2f34461b6).

In Part I, we successfully integrated and deployed an Airnode through ChainAPI. Now we will code a simple Requester Contract to call and read data from our Airnode.

Before starting, make sure you set up the [Airnode Monorepo](https://github.com/api3dao/airnode) on your system. Follow through the Readme to install and build all the dependencies and packages to be able to access the Airnode CLI.

Clone the Airnode Monorepo.
```shell
$ git clone [https://github.com/api3dao/airnode.git](https://github.com/api3dao/airnode.git) .
```
To install the dependencies,
```shell
$ yarn run bootstrap
```
To build all the packages,
```shell
$ yarn run build
```
_An Airnode is a first-party oracle that pushes off-chain API data to your on-chain contract. It makes a request to the on-chain_ [_RRP protocol contract_](https://docs.api3.org/reference/airnode/latest/concepts/) _(_[_AirnodeRrpV0.sol_](https://docs.api3.org/reference/airnode/latest/concepts/#airnoderrpv0-sol)_) that adds the request to the event logs. The off-chain Airnode then accesses the event logs, gets the API data and performs a callback to the requester._

![https://docs.api3.org/airnode/v0.8/grp-developers/](https://miro.medium.com/v2/resize:fit:700/0*JDASappLaTk5nA9p.png)

[https://docs.api3.org/airnode/v0.8/grp-developers/](https://docs.api3.org/airnode/v0.8/grp-developers/)

A [_Requester_](https://docs.api3.org/reference/airnode/latest/concepts/requester.html)  is a contract that triggers an Airnode request. To do so, the requester needs to be sponsored and make the request using a matching sponsor wallet.

The Requester then calls the protocol contract, which emits a blockchain event with the request parameters. Airnode listens to the events emitted by the _AirnodeRrpV0_ contract. During the next run cycle, Airnode gets the request parameters from the emitted event.

## Coding Requester.sol

The Requester Contract will have two main functions, `makeRequest()` and `fulfill()`. The `makeRequest()` function will call the `makeFullRequest()` function of the AirnodeRrpV0.sol protocol contract which adds the request to its storage. The targeted off-chain Airnode gathers the request from _AirnodeRrpV0.sol_’s storage and responds using the `fulFill()` function of _AirnodeRrpV0.sol_.

https://gist.github.com/vanshwassan/593ba6f2655eb5289180bdc94ac87911#file-requester-sol

## Request Parameters

The `makeRequest()` function expects the following parameters to make a valid request.

-   `airnode` (address) and `endpointId` specify the endpoint.
-   `sponsor` and `sponsorWallet` (addresses) specify which wallet will be used to fulfill the request.
-   `parameters` specify the API and [Reserved Parameters](https://docs.api3.org/reference/ois/latest/reserved-parameters.html) (see Airnode ABI [specifications](https://docs.api3.org/reference/ois/latest/specification.html) for how these are encoded) We will encode the parameters off-chain using `@airnode-abi` library.

## Response Parameters

The callback to the _Requester_ contains two parameters:

-   `requestId`: First acquired when making the request and passed here as a reference to identify the request for which the response is intended.
-   `data`: In case of a successful response, this is the requested data which has been encoded and contains a timestamp in addition to other response data. Decode it using the function `decode()` from the `abi` object.

## Compiling the Contract

To deploy the Requester Contract, we are going to use [Remix IDE](https://remix.ethereum.org/). It’s an online IDE that allows developing, deploying and administering smart contracts for EVM Compatible Blockchains.

Make a contract and paste in the [Requester.sol](https://github.com/vanshwassan/AirnodeTutorials/blob/master/Contracts/Requester.sol) code.

![](https://miro.medium.com/v2/resize:fit:700/1*iOh6xK45pBF8MX9YvXBt1A.png)

Now hit compile on the right side of the dashboard and compile the Smart Contract.

![](https://miro.medium.com/v2/resize:fit:331/1*A0SBboW_fTyAC45ws65O2Q.png)

Now we are all set to deploy our Requester.

## Deploying the Requester

As we are going to deploy the contract on Polygon Mumbai Testnet, make sure you have enough MATIC in your wallet to deploy the Requester and then fund the `sponsorWallet` later. You can get some from the [Mumbai Faucet](https://mumbaifaucet.com/).

Head to Deploy and run Transactions and select Injected Provider — MetaMask option under Environment. Connect your MetaMask. Make sure you’re on Mumbai Testnet.

![](https://miro.medium.com/v2/resize:fit:275/1*h34LOQkc5WIAt5l_cWbJ-A.png)

The `_rrpAddress`is the main _airnodeRrpAddress._ The RRP Contracts have already been deployed on-chain. You can check for your specific chain [here](https://docs.api3.org/reference/airnode/latest/).

Fill in the `_rrpAddress` and click on Deploy. Confirm the transaction on your MetaMask and wait for it to deploy the Requester Contract.

![](https://miro.medium.com/v2/resize:fit:638/1*WMB8sP2JDG6XF4XrrTFWog.png)

Make sure you’re on the Polygon Mumbai Testnet

## Calling the Requester

When your Contract gets deployed, head to Deploy & run transactions and click on the dropdown for your Requester under Deployed Contracts.

Now select the `makeRequest` dropdown to see all the parameters you need to pass in order to make a full request to the Airnode.

![](https://miro.medium.com/v2/resize:fit:438/1*3uaFU21-HM1n8EZVXKxU4A.png)

Here, you need to pass in your `airnode`(Airnode address), `endpointID`, `sponsor`(The Requester itself),`sponsorWallet`and `parameters` in order to call the `makeRequest()` function.

We can find the `airnode` in the `receipt.json` under the output directory obtained when we deployed our Airnode.

![](https://miro.medium.com/v2/resize:fit:700/1*-k9a8EhOFqTbusum7DxMPA.png)

The `endpointID` can be found under the `config.json` file.

![](https://miro.medium.com/v2/resize:fit:700/1*ha0EzXURgRHKFK74vnLvPA.png)

We need to derive the `sponsorWallet` through the Airnode CLI command that will make the actual call. We also need to fund it with some MATIC to cover the gas cost.

After you’ve setup the Airnode CLI and installed and built all the dependencies and packages, run the following command to derive your `sponsorWallet` :

### Linux:
```shell
$ npx @api3/airnode-admin derive-sponsor-wallet-address \  
  --airnode-xpub xpub6CUGRUo... \  
  --airnode-address 0xe1...dF05s \  
  --sponsor-address 0xF4...dDyu9
```
### Windows:
```shell
$ npx @api3/airnode-admin derive-sponsor-wallet-address ^  
  --airnode-xpub xpub6CUGRUo... ^  
  --airnode-address 0xe1...dF05s ^  
  --sponsor-address 0xF4...dDyu9
```

Your `airnode-address`and `airnode-xpub` (The _Airnode’s_ extended public key) can be found in the same `receipt.json.` The `sponsor-address` will be the address of the Requester contract itself (the one that you just deployed).

![](https://miro.medium.com/v2/resize:fit:700/1*Tl_4usi_XHOZSH65lrdJew.png)

Run the command to obtain your `sponsorWallet`.

![](https://miro.medium.com/v2/resize:fit:700/1*cMWyikM8lIAxGk6YEGtd-g.png)

Fund the `sponsorWallet` with some test MATIC.

The parameters are required to be encoded in `bytes32` before you send it. We are going to use the `@airnode-abi` library for encoding the parameters off-chain and then sending it to the Requester.

You can set it up by cloning [this tutorial’s repository](https://github.com/vanshwassan/AirnodeTutorials).

Run the following command to get your encoded `parameters` :
```shell
$ node .\src\encodeParams.js
```
Now you have all the parameters that you require to run the `makeRequest` function. Populate all the fields and click on Transact.

_Note: The `sponsor` here will be the address of the Requester Contract that you just deployed._

![](https://miro.medium.com/v2/resize:fit:459/1*buQhbSCE-pq9hAibHbIOmA.png)

Click on transact, confirm the transaction on MetaMask and wait for the transaction to complete.

Now you can head over to [https://mumbai.polygonscan.com](https://mumbai.polygonscan.com) and check your `sponsorWallet` for any new transactions.

_You might need to wait for a while as the Airnode calls the `fulfill()` function in AirnodeRrpV0.sol that will in turn call back the requester contract at `fulfillAddress` using function `fulfillFunctionId` to deliver `data`._

![](https://miro.medium.com/v2/resize:fit:700/1*NnNht40SsIYsKEkNW45DvA.png)

Here, we can see the latest _Fulfill_ transaction.

Now go back on Remix and check for `requestId` under logs for the latest transaction.

![](https://miro.medium.com/v2/resize:fit:700/1*Km2hDmwINDqShb1PlnXXug.png)

You can also find your `requestId` under logs in the Polygon Mumbai Block Explorer.

![](https://miro.medium.com/v2/resize:fit:700/1*Ula12KPb0Ykl1NjhhD3z_A.png)

Copy your `requestId` and paste it on under the `fulfilledData` method to decode the response. Click on call and you will see the API response. Here, we requested Tesla’s Stock price.

![](https://miro.medium.com/v2/resize:fit:412/1*dgVH5M1HlJb-AAwfTao9Hw.png)

![](https://miro.medium.com/v2/resize:fit:700/1*AZ3OgEd9QKtVnPql1hifXA.png)

Now you successfully deployed an Airnode and made a Requester Contract to read data from it. You can also refer to [this Repo](https://github.com/vanshwassan/AirnodeTutorials) for all the code that I’ve used for this tutorial.

Thanks for reading.

**Any questions?** Check out [**API3’s Discord Server**](https://discord.com/invite/qnRrcfnm5W) and drop your queries there!
