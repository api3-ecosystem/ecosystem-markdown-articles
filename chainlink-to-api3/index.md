---
title: Switching from Chainlink VRF to API3 QRNG.
subtitle: Switching from Chainlink VRFs to API3 QRNG to generate NFTs and utilizing true quantum randomness in your mints.
author: Vansh Wassan
date_published: 2022-10-27
category: Technology
image: "https://miro.medium.com/v2/resize:fit:700/0*R8WocxYVyHB7TGe0"
---

_Photo by [Yassine Khalfalli](https://unsplash.com/@yassine_khalfalli?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)_

> I’ve already discussed the problem of generating true randomness in a smart contract in my previous article. This is going to be a more advanced version of that. You can read it [here](https://vanshwassan.medium.com/a-guide-to-using-anus-quantum-random-number-generator-in-your-smart-contracts-21be5bed5aba).

## Introduction

In this tutorial, we’ll be making a simple mint contract and minting an NFT for a random Dungeons and Dragons character giving it attributes/statistics to show its strength, mana, intelligence and experience utilizing API3’s Quantum Random Number Generation (QRNG). I got this idea while I was going through [Patrick Collins’s tutorial](https://blog.chain.link/random-numbers-nft-erc721/) on making the same but instead of using Chainlink’s VRFs to generate and set random attributes, we are going to utilize API3’s QRNG.

To read more about what QRNG is and how it works, click [here](https://docs.api3.org/explore/qrng/).

> [The API3 QRNG Service is free and live on all major EVM Compatible Testnets.](https://docs.api3.org/qrng/reference/chains.html)

The [guide](https://blog.chain.link/random-numbers-nft-erc721/) that Patrick Collins wrote uses Chainlink VRF to generate random attributes for the Character. I’ve already discussed the shortcomings of using a third-party oracle network in my previous article, and I also find it somewhat complicated in setting up LINK token spending limits within your smart contract (It’s not a free service). However, API3 QRNG is completely free and is much easier to integrate. You don’t need to set any spending limits in your smart contracts, instead just fund a simple sponsor wallet that uses the native gas token for the chain to cover all the gas costs.

## Coding the NFT Contract

Let’s start with creating a basic character which has five main attributes:

`strength`, `intelligence`, `mana`, `experience` and `name`. We are going to store all these attributes under a Character struct. Whenever we will make a new Character, these attributes will be set randomly.

We then define a Character array that defines all the characters which exist within the contract.

https://gist.github.com/vanshwassan/ebbd2d94f26f689721473857490e8b66#file-character-sol

We’ve imported the _RrpRequesterV0.sol_ which is the on-chain [RRP protocol contract](https://docs.api3.org/reference/airnode/latest/concepts/) _(_[_AirnodeRrpV0.sol_](https://docs.api3.org/reference/airnode/latest/concepts/#airnoderrpv0-sol)_)_. It is responsible for making the actual full request to the QRNG Airnode. The off-chain Airnode then fetches the event logs after the request has been made and gets the requested random number and performs a callback to the requester.

We’ve also imported the _ERC721.sol_ and _Ownable.sol_ from OpenZeppelin to make and mint an NFT.

The `airnode` is the address for the QRNG Airnode, `endpointIdUint256` specifies the endpoint of the QRNG API path and `sponsorWallet` will be the wallet that will make the actual request. We’ll need to fund this wallet later.

The `expectingRequestWithIdToBeFulfilled` mapping will keep a track of our request and check if it has been fulfilled or not.

The `requestToCharacterName` mapping keeps a track of each request for randomness as it is tied to the Character’s name.

Each character has a unique ID that represents it which is mapped to `requestToTokenId`. `requestToSender` is tied to the address that mints the NFT.

The constructor will accept the `_airnodeRrp` address as an argument.

_ERC721_ takes 2 arguments, the name for the token and the symbol for it. Here, the name is `DungeonsAndDragonsCharacter` and the symbol is `D&D` .

https://gist.github.com/vanshwassan/ea168131267a3071eb6b9339eeb6cfb8#file-character-sol

The _setRequestParameters_ function will set the request parameters.

The _requestNewRandomCharacter_ function will request for randomness to the Airnode. It takes the name of the character as input. It then calls the _makeFullRequest_ function of the [AirnodeRrpV0.sol](https://docs.api3.org/reference/airnode/latest/concepts/#airnoderrpv0-sol) protocol contract which adds the request to its storage and returns a `requestId.`The `requestId` will then be used to set all the mappings in the function.

Once the request has been made to the Airnode, we now wait for it to be fulfilled, which will be done through the _fulfillRandomness_ function.

The targeted off-chain QRNG Airnode gathers the request and performs a callback to the requester (using the _fulfillRandomness_ function) with the random number (`qrngUint256`).

https://gist.github.com/vanshwassan/9dfe33c1dbd0068f1ae704ba7aea9676#file-character-sol

Using the `qrngUint256` , we can now set the attributes for our character and push the character to the array.

Finally, we will mint the token and send it to the address that made the randomness request along with its _Id_.

The _getCharacterStats_ is a view function that will get back the character stats. It takes in `tokenId` .

We can now move forward and deploy and compile the Smart Contract.

## Compiling and Deploying the NFT Contract

Head to [Remix IDE](https://remix.ethereum.org), make a contract and paste in the [API3QRNG](https://gist.github.com/vanshwassan/48baa25995ef3c556ebf208c725f7a86) code. Now hit compile on the right side of the dashboard and compile the Smart Contract.

![](https://miro.medium.com/v2/resize:fit:379/1*yvwsN04zZT8IplHVdcKYAg.png)

We are going to deploy it on the Goerli Testnet. Make sure you have enough testnet ETH in your wallet to deploy the contract and fund the `sponsorWallet` later. You can get some from [here](https://goerlifaucet.com/).

Head to Deploy and run Transactions and select Injected Provider — MetaMask option under Environment. Connect your MetaMask. Make sure you’re on Goerli Testnet.

![](https://miro.medium.com/v2/resize:fit:359/1*qJPTWT4F1Sze8HTxwpepMQ.png)

The `_rrpAddress` is the main _airnodeRrpAddress._ The RRP Contracts have already been deployed on-chain. You can check for your specific chain [here](https://docs.api3.org/qrng/reference/chains.html).

Fill in the `_rrpAddress` and click on Deploy. Confirm the transaction on your MetaMask and wait for it to deploy the requester Contract.

## Calling the Contract

When your _API3QRNG_ gets deployed, head to Deploy & run transactions and click on the dropdown for your requester under Deployed Contracts.

Now select the _setRequestParameters_ dropdown to set all the parameters.

![](https://miro.medium.com/v2/resize:fit:335/1*i-8w300dh7xw0n7TFJCWEw.png)

You can find airnode (QRNG Provider Airnode Address) and endpointIdUint256 [here](https://docs.api3.org/reference/qrng/providers.html).

`sponsortWallet` is your wallet that is derived from the requester contract address, the Airnode address and the Airnode xpub. The wallet is used to pay gas costs to acquire a random number. A sponsor wallet must be derived using the command [derive-sponsor-wallet-address](https://docs.api3.org/reference/airnode/latest/packages/admin-cli.html#derive-sponsor-wallet-address) from the Admin CLI. Use the value of the _sponsor wallet address_ that the command outputs. Make sure you fund this wallet with enough test ETH as it’s gonna make the actual request.

Click on transact button and confirm the transaction to set the parameters.

Now we can move forward and request a random character. To do that, click on _requestNewRandomCharacter_ and give your character a name. Click on transact.

![](https://miro.medium.com/v2/resize:fit:329/1*L0gr-z58yM_twMRafrKUnA.png)

Now you can head over to [https://goerli.etherscan.io](https://goerli.etherscan.io) and check your _sponsorWallet_ for any new transactions.

> You might need to wait for a while as the Airnode calls the `fulfill()` function in AirnodeRrpV0.sol that will in turn call back the requester contract at `fulfillAddress` using function `fulfillFunctionId` to deliver `data`(the random number).

![](https://miro.medium.com/v2/resize:fit:700/1*ef2XWtbxdClyd7_hTuXfNQ.png)

Here, we can see the latest _Fulfill_ transaction. You can click on it and see the D&D NFT has been minted and sent to your address.

![](https://miro.medium.com/v2/resize:fit:700/1*0xkrQlN25d1XdHGnk-MYXA.png)

To check your Character stats, go back to Remix, click on _getCharcterStats_ and input your tokenId (0 in this case)

![](https://miro.medium.com/v2/resize:fit:389/1*SLyqHTJ003aUaTsYmTgP0Q.png)

Here, these are the Character Stats/Attributes for our Character.

So we requested and minted a new Character NFT using Quantum Randomness.

Check the [Github repo](https://github.com/vanshwassan/QRNG-NFT-Mint) for all the code used in this guide.

Read more about [API3 QRNG](https://docs.api3.org/qrng/).

Any questions?

**Check out [**API3’s Discord Server**](https://discord.com/invite/qnRrcfnm5W) and drop your queries there!**
