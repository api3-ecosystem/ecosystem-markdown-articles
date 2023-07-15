---
title: Making an on-chain Payment and Minting an NFT Receipt with Permissionless Price Oracles
subtitle: dApp to accept ERC20 token payments and mint an NFT receipt with the USD value to the user.
author: Vansh Wassan
date_published: 2023-05-03
category: Technology
image: "https://miro.medium.com/v2/resize:fit:700/0*xseXMMG6UZFTj0yc"
---

Photo by [Shubham Dhage](https://unsplash.com/@theshubhamdhage?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)

## Introduction

In my [previous guide](https://vanshwassan.medium.com/using-dapis-on-zksync-era-testnet-30f12efdd95f) I explained how to request the latest price data on-chain. In this guide, I’m going to explain how to code a smart contract that accepts any ERC20 token as payment, and mints the payment receipt as an NFT that includes how much you paid to the contract in USD value at the time of payment. For this tutorial, I’m going to use a mock ERC20 token (Mock WETH) and set it to the ETH/USD dAPI for calculating the USD value for each payment.

[_To know more about dAPIs, check out this previous guide_](https://medium.com/@vanshwassan/using-dapis-on-zksync-era-testnet-30f12efdd95f)

## API3 Market: Activating self-funded dAPIs

Developers can use the [API3 Market](https://market.api3.org/dapis) to search for dAPIs, obtain dAPI proxy contract addresses, and monitor dAPIs. To access self-funded dAPIs, you can head to the [API3 Market](https://market.api3.org) to fund and access a data feed so that a smart contract can utilize the value in a dApp.

![](https://miro.medium.com/v2/resize:fit:700/1*u-NU_DB3Dqo18IUV3OZaAQ.png)

[https://market.api3.org/dapis](https://market.api3.org/dapis)

### Funding a feed

To use the self-funded dAPIs, the user is required to fund the respective dAPI sponsor wallet with gas to activate it. The amount of gas supplied will determine how long your dAPI will be available for use. If the dAPI runs out of gas, it can be funded again to keep it available for use.

If the feed is not already funded, users need to provide chain native collateral to the sponsor address of the required feed. This process happens through the API3 Market UI, where there is functionality to understand the estimated gas remaining and the time that the feed will be active.

![](https://miro.medium.com/v2/resize:fit:456/1*SKfmZpACV7OvZN056nuhog.png)

### Deploying the proxy contract

After ensuring the dAPIs are funded, you can deploy the proxy contract using the Market to access the dAPI.

-   For ETH/USD on Polygon Mumbai — [ETH/USD dAPI](https://market.api3.org/dapis/polygon-testnet/ETH-USD)
-   For MATIC/USD on Polygon Mumbai — [MATIC/USD dAPI](https://market.api3.org/dapis/polygon-testnet/MATIC-USD)

If the proxy contract for a dAPI is already deployed, you can just start using it.

![](https://miro.medium.com/v2/resize:fit:451/1*iYWp5VFUMa4RRs8SI2HtLQ.png)

## Getting Started

Before starting, make sure you have the following installed:

-   _nodejs_
-   _yarn_
-   _git_

We assume Javascript & Solidity basics are understood when reading this.

To get started, clone this repo:

```shell
$ git clone https://github.com/vanshwassan/dAPI-payments.git
```

To install the dependencies,

```shell
$ yarn
```

## Coding the Contracts

There are 3 main contracts that we’ve used in this project.

-   _Payments.sol_
-   _WETH.sol_
-   _WMATIC.sol_

The Payments contract has many functions. Using the contract, any user can:

-   Send any ERC20 (Mock WETH, Mock WMATIC) payments to the contract and mint a Payment Receipt Token(PRT) NFT with the USD value of the assets at the time of payment.
-   Check the USD value of the assets using your NFT’s `tokenId`.

Similarly, the Owner (the address that deployed the contract) can:

-   Set the dAPI Proxies for the ERC20 tokens using the [API3 Market](https://market.api3.org/dapis).
-   Withdraw the total funds sent to the contract.

_Payments.sol_ has all the functions required for collecting the ERC20 funds and calculating its USD value.

`setDapiProxy()` will be used to map and set the dAPI Proxy for the ERC20 tokens which the user will be required to pay. You can see a list of available dAPIs [here](http://market.api3.org).

```solidity
function setDapiProxy(address token, address _proxy)  
        public  
        onlyOwner  
    {  
        tokenProxyMapping[token] = _proxy;  
    }
```

`getTokenPrice()` function calls the dAPI and takes in the token address mapped to its dAPI Proxy obtained from the API3 Market and returns the latest price of the requested asset.

```solidity
function getTokenPrice(address token) public view returns (uint256 tokenPriceUint256) {  
        address proxy = tokenProxyMapping[token];  
        int224 value;  
        uint256 timestamp;  
        (value, timestamp) = IProxy(proxy).read();  
        uint224 tokenPriceUint224 = uint224(value);  
        tokenPriceUint256 = tokenPriceUint224;  
    }
```

The `Payment()` function takes in the address of the token and the total amount that the user wants to pay. It then calls the `getTokenPrice()` function to get the latest price and mints an NFT Receipt with a `tokenId` and the total USD value the user paid.

```solidity
function Payment(address token, uint256 _tokenAmount) public returns(uint256) {  
        require(_tokenAmount > 0, "amount cannot be 0");  
        require(tokenIsAllowed(token), "token not allowed");  
        if (tokenIsAllowed(token)) {  
            uint256 tokenId = _tokenIdCounter.current();  
            _tokenIdCounter.increment();  
            uint256 decimals = ERC20(token).decimals();  
            uint256 tokenPriceUint256 = getTokenPrice(token);  
            uint256 _usdValue = (tokenPriceUint256 * _tokenAmount)/10**decimals;  
            ERC20(token).transferFrom(msg.sender, address(this), _tokenAmount);  
            _safeMint(msg.sender, tokenId);  
            uint256 receipt = makeReceipt(tokenId, _usdValue);  
            return receipt;  
    }  
    }
```

The `checkReceipt()` function is a view function that returns the total USD value of the ERC20 asset paid by the user by taking in the `tokenId` minted with the NFT Receipt at the time of payment.

```solidity
function checkReceipt(uint256 tokenId) public view returns (uint256) {  
        return (TokenIDtoPrice[tokenId]);  
    }
```

The `ownerWithdrawFunds()` function that the owner of the contract can call to withdraw all the funds from the contract to his wallet.

```solidity
function ownerWithdrawFunds() public onlyOwner {  
        for (uint256 i = 0; i < allowedTokens.length; i++) {  
            ERC20(allowedTokens[i]).transfer(msg.sender, ERC20(allowedTokens[i]).balanceOf(address(this)));  
        }  
    }
```

Here’s the full contract:

https://gist.github.com/vanshwassan/f74cab07cf188f40e3e56558a15d258c#file-payments-sol

You can find the repository with contract code [here](https://github.com/vanshwassan/dAPI-payments).

## Deploying the Contracts

Before moving forward, enter your mnemonic in _credentials.json_ to deploy and interact with the contracts.

```shell
$ cp credentials.example.json credentials.json
```

Now go to _credentials.json_ and enter your credentials. As we’re testing this on Mumbai Testnet, you can just add your mnemonic under Mumbai. Make sure you fund your wallet with testnet MATIC before moving forward. You can request [testnet MATIC here](https://mumbaifaucet.com/)

Now compile the contracts:
```shell
$ yarn compile
```
To deploy the contracts and set the dAPI Proxies:
```shell
$ yarn deploy
```
This will deploy the **Mock WETH** and **Mock WMATIC** token contracts alongside the **Payments Contract**. It will also set the dAPI proxy contract addresses mapped to their respective ERC20 tokens. Head on to `deploy/[3_deploy_Payments.js](https://github.com/vanshwassan/dAPI-payments/blob/master/deploy/3_deploy_Payments.js)` If you want to make changes to the dAPI Proxy/use another ERC20 token.

## Using the dApp

Import the Mock WETH contract address in your account’s Metamask. This token is just a mock token, a simple implementation of an openzeppelin ERC20 contract. As it’s a testnet, **I’m going to call it WETH and map its value to the ETH/USD dAPI.**

This token will be used to make payments to the contract.

Similarly, for the **Mock WMATIC token, the value can be mapped to the MATIC/USD dAPI**.

To use the scripts I made to interact with the contracts, head back to the [Project](https://github.com/vanshwassan/dAPI-payments) and use the following commands.

-   Use the below command to send a payment to the contract. This will **send 2.5 Mock WETH and 2.5 Mock WMATIC** to the contract and will mint the **Payments Receipt Tokens(PRT)** to the user’s address as a digital proof of payment with the USD value of the token(s) at the time of payment. You can edit the amounts in `scripts/[sendPayments.js](https://github.com/vanshwassan/dAPI-payments/blob/master/scripts/sendPayment.js)`

```shell
$ yarn sendPayment
```

-   Use the below command to check your Payments Receipt Token(PRT). It takes in the `tokenId` and returns the USD value of how much the user paid. Edit the `tokenId` value in `scripts/[checkReceipt.js](https://github.com/vanshwassan/dAPI-payments/blob/master/scripts/checkReceipt.js)`

```shell
$ yarn checkReceipt
```

-   Use the below command to check the USD value of all the funds that the contract holds.

```shell
$ yarn checkContractBalance
```

-   Use the below command to withdraw all the funds stored in Contract. Only the owner of the contract can call this.

```shell
$ yarn withdraw
```

## Conclusion

In this tutorial, you learned how to code a simple, usable dApp and how we can utilize dAPIs within a smart contract.

Check out the repository for this guide [here](https://github.com/vanshwassan/dAPI-payments).

Read more about dAPIs [here](https://docs.api3.org/guides/dapis/subscribing-self-funded-dapis/).

Developers can get started with dAPIs now through the API3 Market. If you have any questions, you can join the [API3 Discord](https://discord.com/invite/qnRrcfnm5W) server.