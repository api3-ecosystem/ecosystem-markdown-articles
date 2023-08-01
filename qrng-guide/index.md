---
title: A Guide to Using ANU’s Quantum Random Number Generator in Your Smart Contracts
subtitle: Create your contracts with truly random numbers
author: Vansh Wassan
date_published: 2023-04-15
category: Technology
image: "https://miro.medium.com/v2/resize:fit:700/0*rjYmsGDM5hrddc2f"
---

_Photo by [FLY:D](https://unsplash.com/@flyd2069?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)_

Random number generation (RNG) has always been one of the biggest problems when working with smart contracts. A deterministic virtual machine is incapable of generating ‘true’ randomness. Due to this, RNG needs to be provided as an oracle service.

To fulfill the need for randomness in smart contracts, decentralized pseudorandom RNG has been a common way. One of the most used methods is Chainlink’s VRF or verifiable random function, which provides cryptographically provable random numbers on-chain. It generates a random number off-chain with cryptographic proof that’s used to verify the result.

> However, this configuration suffers from the same issues as any other third-party oracle network. Setting up an oracle node that can provide PRNG exposes potential attack vectors like [Sybil attacks](https://en.wikipedia.org/wiki/Sybil_attack), but also lacks source transparency and decentralization. For example, one needs to trust the governing entity to select the network participants, which means decentralized PRNG is only as secure and decentralized as the governing entity.

## Quantum Random Number Generation

[QRNG](https://docs.api3.org/explore/qrng/) generates randomness via quantum phenomena. It uses a ‘true’ source of entropy using unique properties of quantum physics to generate true randomness.

There are different methods of implementing QRNG with varying levels of practicality, yet the common point is that the resulting numbers will be truly random because the outcome of a quantum event is theoretically uncertain with well-defined characteristics. Therefore, QRNG is the gold standard for random number generation.

## Australian National University’s QRNG Airnode

![](https://miro.medium.com/v2/resize:fit:700/0*sm4Eo5c6GcDTwkIU)

_Dr Aaron Tranter of [ANU](https://quantumnumbers.anu.edu.au/)_

As we already discussed, providing RNG through a third-party oracle network opens space for attack vectors. But first-party oracles ([Airnodes](https://docs.api3.org/airnode/v0.8/)) that are directly operated by the [QRNG API](https://docs.api3.org/reference/qrng/providers.html) Providers optimally counter the Sybil attack risk.

[API3 QRNG](https://docs.api3.org/explore/qrng/) is a public utility offered through the Australian National University (ANU). It is powered by an Airnode hosted by ANU Quantum Random Numbers, meaning that it is a first-party service. [Australian National University](https://www.anu.edu.au/)’s Quantum Optics Division is one of the worlds leading research institutions in the field. The division also operates a REST API, [Quantum Random Numbers API](https://quantumnumbers.anu.edu.au/), to serve QRNG in Web2.

[Read more about how ANU Generates random numbers in real time by measuring the quantum fluctuations of the vacuum](https://quantumnumbers.anu.edu.au/)

It is served as a public good and is free of charge (apart from the gas costs), and it provides ‘true’ quantum randomness via an easy-to-use solution when requiring RNG on-chain.

## How Airnode and API3 QRNG works

![](https://miro.medium.com/v2/resize:fit:700/0*vDXkbKS5MSHUbFL1.png)

[https://docs.api3.org/reference/qrng/](https://docs.api3.org/reference/qrng/)

To begin, we need to deploy and sponsor the `QrngRequester` with a matching sponsor wallet. The `QrngRequester` will be the primary contract that retrieves the random number.

The `QrngRequester` submits a request for a random number to `AirnodeRrpV0`. Airnode gathers the request from the `AirnodeRrpV0` protocol contract, retrieves the random number off-chain, and sends it back to `AirnodeRrpV0`. Once received, it performs a callback to the requester with the random number.

You can read more about how API3 QRNG uses the request-response protocol [here](https://docs.api3.org/reference/qrng/).

## Coding `QrngRequester.sol`

### Getting started

Make sure you have the following installed:

-   node.js
-   yarn/npm

Also, make sure you’ve already cloned and installed the [Airnode Monorepo](https://github.com/api3dao/airnode). If you haven’t, clone the Airnode Monorepo with this command:

```shell
$ git clone https://github.com/api3dao/airnode.git .
```
To install the dependencies, do the following:
```shell
$ yarn run bootstrap
```
To build all the packages, use this command:
```shell
$ yarn run build
```
## Compiling the Contract

To compile the QrngRequester contract, we are going to use [Remix IDE](https://remix.ethereum.org/). It’s an online IDE that allows the development, deploying, and administering of smart contracts for EVM-compatible blockchains.

https://gist.github.com/vanshwassan/0b50c7b36b1e7ebed85549754578ed79#file-qrngrequester-sol

The `QrngRequester` will have three main functions: `setRequestParameters()`, `makeRequestUint256()`, and `fulfillUint256()`.

1.  The `setRequestParameters()` takes in `airnode`, `endpointIdUint256`, `sponsorWallet` and sets these parameters.
2.  The `makeRequestUint256()` function calls the `airnodeRrp.makeFullRequest()` function of the [`AirnodeRrpV0.sol`](https://docs.api3.org/reference/airnode/latest/concepts/#airnoderrpv0-sol) protocol contract which adds the request to its storage and returns a `requestId`.
3.  The targeted off-chain ANU Airnode gathers the request and performs a callback to the requester with the random number.

## Request Parameters

The `makeRequestUint256()` function expects the following parameters to make a valid request.

-   `airnode` (address) and `endpointIdUint256`specify the endpoint. Get these from [here](https://docs.api3.org/reference/qrng/providers.html).
-   [`sponsorWallet`](https://docs.api3.org/reference/airnode/latest/concepts/sponsor.html#sponsorwallet) specifies which wallet will be used to fulfill the request.

## Response Parameters

The callback to the `QrngRequester` contains two parameters:

-   `requestId`: First acquired when making the request and passed here as a reference to identify the request for which the response is intended.
-   `data`: In case of a successful response, this is the requested data that has been encoded and contains a [timestamp](https://docs.api3.org/ois/v1.1/reserved-parameters.html#timestamp-encoded-to-uint256-on-chain) in addition to other response data. Decode it using the function `decode()` from the `abi` object to get your random number.

Head to Remix IDE, make a contract, and paste it in the [QrngRequester](https://gist.github.com/vanshwassan/0b50c7b36b1e7ebed85549754578ed79#file-qrngrequester-sol) code.

![](https://miro.medium.com/v2/resize:fit:700/1*xqIXsrezF-oEibrUXUCHBA.png)

Now, hit compile on the right side of the dashboard and compile the smart contract.

![](https://miro.medium.com/v2/resize:fit:343/1*NC2OnVs_WbJyQ1x92IWNWg.png)

## Deploying the Contract

We are going to deploy our `QrngRequester` to Goerli. Make sure you have enough testnet ETH in your wallet to deploy the contract and fund the `sponsorWallet` later. You can get some testnet Goerli [here](https://goerlifaucet.com/).

Head to deploy, run Transactions, and select the “Injected Provider — MetaMask” option under Environment. Connect your MetaMask. Make sure you’re on Goerli.

![](https://miro.medium.com/v2/resize:fit:370/1*egGwfeoxm80f14aLRGPIgg.png)

The `_rrpAddress`is the main `airnodeRrpAddress`. The RRP contracts have already been deployed on-chain. You can check for your specific chain [here](https://docs.api3.org/reference/qrng/chains.html).

Once the `_rrpAddress` is populated, click on “Deploy.” Confirm the transaction on your MetaMask and wait for it to deploy the Requester contract.

![](https://miro.medium.com/v2/resize:fit:347/1*tBUez1T4eti5tznbrtg6Ew.png)

Make sure you’re on the Goerli Testnet

## Calling the Contract

When your `QrngRequester` gets deployed, head to Deploy, run transactions, and click on the dropdown for your Requester under Deployed Contracts.

Now select the `setRequestParameters` dropdown to set all the parameters.

![](https://miro.medium.com/v2/resize:fit:373/1*rbdNJXXxCsbS0YIEKhTppw.png)

Add the following to the corresponding fields for the function.

-   `_airnode`: The airnode address of the desired QRNG service provider. See its value from the list of [QRNG providers](https://docs.api3.org/reference/qrng/providers.html).
-   `_endpointIdUint256`: The Airnode endpoint ID will return a single random number.
-   `_sponsorWallet`: A wallet derived from the requester contract address, the Airnode address, and the Airnode xpub. The wallet is used to pay gas costs to acquire a random number. A sponsor wallet must be derived using the command [derive-sponsor-wallet-address](https://docs.api3.org/reference/airnode/latest/packages/admin-cli.html#derive-sponsor-wallet-address) from the Admin CLI. Use the value of the sponsor wallet address that the command outputs.

After you’ve set up the Airnode CLI, installed and built all the dependencies and packages, run the following command to derive your `_sponsorWallet`:

### Linux

```shell
npx @api3/airnode-admin derive-sponsor-wallet-address \  
  --airnode-xpub xpub6CUGRUo... \  
  --airnode-address 0xe1...dF05s \  
  --sponsor-address 0xF4...dDyu9
```
### Windows

```shell
npx @api3/airnode-admin derive-sponsor-wallet-address ^  
  --airnode-xpub xpub6CUGRUo... ^  
  --airnode-address 0xe1...dF05s ^  
  --sponsor-address 0xF4...dDyu9
```

ANU’s `airnode-address`and `airnode-xpub` can be found [here](https://docs.api3.org/reference/qrng/providers.html).

![](https://miro.medium.com/v2/resize:fit:612/1*xYnxZk6fRmb5iNFtu5qGhg.png)

Fund the `sponsorWallet` with some test ETH. Click on transact button and confirm the transaction to set the parameters.

![](https://miro.medium.com/v2/resize:fit:374/1*Tycn4l0Yqv5LWSmAcdjDrg.png)

To make the request, click on the `makeRequestUint256` button to call the function and make a full request.

![](https://miro.medium.com/v2/resize:fit:379/1*G6p3_QX0wYCvjRAMO_5G2Q.png)

Now you can head over to [](https://rinkeby.etherscan.io/) [https://goerli.etherscan.io/](https://goerli.etherscan.io/) and check your `sponsorWallet` for any new transactions.

You might need to wait for a while as the Airnode calls the `fulfill()` function in `AirnodeRrpV0.sol` that will in turn call back the requester contract at `fulfillAddress` using function `fulfillFunctionId` to deliver `data`(the random number).

![](https://miro.medium.com/v2/resize:fit:700/1*NMjxVn2BUXZeX3-KWGCS9A.png)

Here, we can see the latest `Fulfill` transaction.

Now go back to Remix and click on `latestRequest` button to check the response.

![](https://miro.medium.com/v2/resize:fit:370/1*BcTgivXIHUHUxj6cmkQObQ.png)

If the callback has been successfully completed, the `randomNumber` will be present. The value of `waitingFulfillment` will be `false`.

If you want to learn more about it, check out the [QRNG Example Project](https://github.com/api3dao/qrng-example).

Read more about [API3 QRNG](https://docs.api3.org/guides/qrng/).

Want to Connect?

**Check out [API3’s Discord Server](https://discord.com/invite/qnRrcfnm5W) and drop your queries there!**
