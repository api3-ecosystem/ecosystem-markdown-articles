---
title: Building Quantumon — Part 1 — Smart Contract Integration with QRNG
author: Ashar Shahid
date_published: 2022-07-01
category: Technology
image: "https://miro.medium.com/v2/resize:fit:700/1*ZQ4_lMwR_TwdYxe614uJFw.png"
---

When API3 first launched their QRNG service one of the first possibilities that came to my mind was being able to mint Quantum Random NFTs. I decided to build a project that combines AI and Quantum Randomness to build something special.

[Quantumon](http://Quantumon.xyz) or Quantum Monsters are a collection of AI generated monsters that are minted to users with the help of QRNG and DALL-E AI.

![](https://miro.medium.com/v2/resize:fit:630/1*QkNhrgaVmiwNL5ZAEUrqQw.png)

![](https://miro.medium.com/v2/resize:fit:630/1*0i-45FbM3wpX1D4WI0mbQA.png)

![](https://miro.medium.com/v2/resize:fit:630/1*2FoxREJW-ZD3mjfSwnpKnQ.png)

Some of the monster that you can mint

In this article I will be going over the smart contract integration of Quantumon and how you can use ERC-721 standard and QRNG to create your own random NFT minting and distribution contract.

One of the most important aspects of NFTs is their perceived rarity making it very important for the attributes that affect rarity to be randomly generated. If a bad actor exploits the minting contract, they can keep the rare items for themselves making it unfair for everyone else. With QRNG we can do fair distribution of NFTs using [“out of order”](https://www.justinsilver.com/technology/cryptocurrency/nft-mint-random-token-id/) minting. Now that we have that out of the way lets get into the contract

## Imports

We will start of by specifying the license, solidity version ,importing the necessary contracts and then creating the actual contract itself

https://gist.github.com/Ashar2shahid/6667da37b967538c7b646c8b81064859#file-quantumon-sol

_Our contract after importing all the necessary contracts_

**Ownable**- This is [Openzeppelin implementation](https://docs.openzeppelin.com/contracts/2.x/access-control#ownership-and-ownable) of an ownership contract. Contracts that inherit the Ownable contract have access to modifiers that make it possible for certain function to only be invoked by the owner. The owner of the contract is the contract deployer. However the ownership can be transferred by the owner. Openzeppelin contracts are audited and used in production by many protocols making them ideal to use in our code.

**ERC721**- This is [Openzeppelin implementation](https://docs.openzeppelin.com/contracts/3.x/erc721) of a Non Fungible Token(NFT) standard. This is the contract we inherit to have our contract be able to mint and transfer NFTs.

**RrpRequesterV0**- Our contract will inherit this contract to be declared as a Requester that will be communicating with the [Request Response Protocol](https://docs.api3.org/airnode/v0.7/concepts/)(RRP). Using the RRP protocol we will request for a random number from the [ANU](https://qrng.anu.edu.au/contact/how-to-use-qrng-on-web3/) Airnode.

## Constructor and Public Variables

https://gist.github.com/Ashar2shahid/1231c156af629d917ef621d99a17ae30#file-quantumon-sol

_Our Contract after we have defined the constructor and public variables_

```solidity
using Strings for uint256;
```
This statement makes it possible for a uint256 variable to call up functions in the Strings library.

**`ids`:** This is an array used to help in [“out of order”](https://www.justinsilver.com/technology/cryptocurrency/nft-mint-random-token-id/) minting process. This array **does not** keep track of the already minted NFTs.

**`index`:** This variable will be used to keep track of the number of NFTs minted so far.

**`_baseURIextended:`** This variable holds the base URI which will be appended to the [**tokenURI**](https://docs.opensea.io/docs/metadata-standards#implementing-token-uri)  of each minted NFT.

**`airnode`**: The airnode variable holds the address of the [Australian National University’s Airnode](https://docs.api3.org/qrng/providers.html#anu-quantum-random-numbers) which we will be requesting a random number from.

**`endpointIdUint256`**: When sending a request to the airnode we need to specify the endpointId. You can think of the endpointId as the path in a url. For example if the URL is `www.google.com/images` then `www.google.com` would be the airnode and `/images` would be the endpointId.

**`sponsorWallet`**: The fullfilment transaction that returns us the random number will be done via the sponsorWallet of [Australian National University’s Airnode](https://docs.api3.org/qrng/providers.html#anu-quantum-random-numbers). It is called sponsorWallet because it is derived via the sponsor address(since we import RrpRequesterV0 this contract sponsor’s itself so the sponsor address is the address of the contract after it is deployed) and the [ANU Airnode’s extended public key](https://docs.api3.org/qrng/providers.html#anu-quantum-random-numbers).

**`expectingRequestWithIdToBeFulfilled`**: Whenever we make a request to the airnode we are returned a requestId. This mapping maps the requestId to a boolean, setting it true when we make a request and setting it false when the request is fulfilled.

**`requesterToSender`**: This mapping maps the requestId to the address making the request.

The constructor of the contract will be invoked when we deploy the contract. Our constructor will accept a single argument called `_airnodeRrp` . This argument is the address of the Airnode Request Response Protocol of the chain this contract will be deployed to. A list of all the AirnodeRrp addresses can be [found here](https://docs.api3.org/qrng/chains.html) for each chain. This address is passed onto the constructor of [RrpRequesterV0](https://github.com/api3dao/airnode/blob/master/packages/airnode-protocol/contracts/rrp/requesters/RrpRequesterV0.sol) which sets the current deployed contract as a sponsor for itself. You don’t have to worry about sponsorship for now we will cover that in a bit.

We also call the constructor of ERC721 and give it the `tokenName`and `tokenSymbol`as arguments. In our case it is QUANTUMON for both the name and symbol.

We leave the constructor body empty because we don’t need to execute any other code during deployment.

## Setting the Request Parameters

https://gist.github.com/Ashar2shahid/15596ccad2e6252bc88532fe6ba31106#file-quantumon-sol

We create a function called `setRequestParameters` that sets the airnode, endpointIdUint256 and sponsorWallet. This function has the `onlyOwner` modifier which means that only the owner of this contract can call this function. The `setRequestParameters` function should be called immediately after deploying the contract.

## Metadata and TokenURI functions

NFT platforms call the `tokenURI(uint256 tokenId)` function to get the url that points to the metadata of the NFT. This metadata contains different attributes of the NFT like its name, image and other custom attributes depending on which platform you use. Opensea expects the metadata to be of the following format:
```json
{     
    "description": "Friendly OpenSea Creature",  
    "external_url": "https://openseacreatures.io/3",  
    "image": "https://storage.googleapis.com/opensea-prod.appspot.com/puffs/3.png",  
    "name": "Dave Starbelly",  
    "attributes": [ ... ],    
}
```
We will override the default ERC721 `tokenURI()` method to enable custom URI for every token and also add a baseURL that gets prefixed to every `tokenURI`.

https://gist.github.com/Ashar2shahid/ca14fec16f95f1bfb65bdc5e2a49f1c7#file-quantumon-sol

we define 4 functions overriding the _baseURI() and `tokenURI()` function.

**`_baseURI`**: This function is an internal function which means that this can only be called by this contract or contracts that inherit this contract. `_baseURI` is a method of the original ERC721 contract, we are overriding it using the `override` keyword to just return the`_baseURIextended` string.

**`setBaseURI`**: This function is an `onlyOwner` function that can set the `_baseURIextended` string.

**`_setTokenURI`**: This is an internal function that maps the URI to a tokenId .If the tokenId hasn’t been minted it will revert.

**`tokenURI`**: This is a view function (function that don’t change the state of the blockchain) that is overridden to return the URI in the `_tokenURIs` mapping. The code comments explain what it does in more detail.

Note: In solidity `string(abi.encodePacked(StringA,StringB))` returns the concatenation of `stringA` and `stringB` .

## How the NFTs are structured

We have all the necessary variables and functions to request a random number and mint our NFT. In our case each NFT has a `_baseUrlExtended` that is set to

[https://quantumon.s3.amazonaws.com/data/quantum_dex/nft/](https://quantumon.s3.amazonaws.com/data/quantum_dex/nft/)

if we add 0 to the end of that URL we get the metadata of the 0th Quantumon. If we add 1, we get the metadata of the 1st Quantumon. In total there are 9958 Quantumons which means there are 9958 metadata files. We want to be able to randomly choose the number that gets appended to this url, however if a Quantumon is already minted we don’t want to mint it again. To do this we use this algorithm by Justin Silver that takes a random number and gives a completely unique non minted QuantumonId:
```solidity
function _pickRandomUniqueId(uint256 random) private returns (uint256 id) {uint256 len = ids.length - index++;require(len > 0, 'no ids left');uint256 randomIndex = random % len;id = ids[randomIndex] != 0 ? ids[randomIndex] : randomIndex;ids[randomIndex] = uint16(ids[len - 1] == 0 ? len - 1 : ids[len - 1]);ids[len - 1] = 0;}
```

you can head over to his [blog](https://www.justinsilver.com/technology/cryptocurrency/nft-mint-random-token-id/) to understand it in depth.

https://gist.github.com/Ashar2shahid/e7b70712f23dd33b556aa66a6d8788b4#file-quantumon-sol

_The Complete Contract_

## requestQuantumon

To use the algorithm to mint the Quantumons we need to request the random number first. The `requestQuantumon` function creates the request to the airnode using the `airnodeRrp.makeFullRequest()` function. The `airnodeRrp.makeFullRequest()` function expects the following arguments:
```
airnode  
endpointId  
sponsor  
sponsorWallet  
fulfillmentAddress  
fulfillmentFunction  
parameters
```
We have already covered `airnode`and `endpointId`, so let me explain about `sponsor`and `sponsorWallet`.

To understand sponsors and sponsorWallets, we need to first understand the first party oracle architecture. The diagram below will help us understand it a little better.

![](https://miro.medium.com/v2/resize:fit:700/0*ELtyRPpQfvI9p82T.png)

Essentially when we call `makeFullRequest()`an event is emitted and the API’s airnode is listening to this event. Based on the event it makes the call to the actual API itself and fetches the data we requested. In our case it fetches the random number and makes a callback. This callback is again a blockchain transaction. Blockchain transaction costs gas and the API’s airnode doesn’t want to spend money on gas. Hence we “sponsor” our smart contract and send gas tokens to the api cloud provider to make the callback transaction.

But where do we send our funds? How will the API’s airnode know which sponsor does the funds belong to in case of multiple such sponsors?

There is a way to derive a wallet that is owned by the api cloud provider which will be unique for each sponsor.

using the [command](https://docs.api3.org/airnode/v0.7/reference/packages/admin-cli.html#derive-sponsor-wallet-address) below you can derive a unique “sponsorWallet” for a given sponsor, all you need is the airnode address and the airnode’s extended public key.
```shell
npx @api3/airnode-admin derive-sponsor-wallet-address \  
  --airnode-xpub xpub6CUGRUo... \  
  --airnode-address 0xe1e0dd... \  
  --sponsor-address 0x9Ec6C4...
```
![](https://miro.medium.com/v2/resize:fit:700/0*4ON_V1MKK9OS05Iu.png)

[https://docs.api3.org/airnode/v0.7/concepts/sponsor.htm](https://docs.api3.org/airnode/v0.7/concepts/sponsor.html#frontmatter-title)l

By inheriting the `RrpRequesterV0` contract, we declare the contract as its own sponsor. This means that we can substitute the address of the deployed contract as the sponsor address in the above command.

`fulfillmentAddress` is the address the request will be fullfilled to, in our case it will be the address of the contract itself.

`fullfullmentFunction` is the function within the `fulfillmentAddress` that will be called with the requested data. In our case it is the `generateQuantumon` Function

Now that we have all our parameters we can call the `makeFullRequest()` function. The `makeFullRequest()` function returns a `requestId` , which we use to set the `expectingRequestWithIdToBeFulfilled` mapping and also the `requestToSender` mapping.

The `requestQuantumon` function has a `require(msg.value >= 5 ether,"...")` condition, which means the user must pay 5 ether to be able to call this function.

> To have a streamlined UX we can have the user pay for the fulfillment transaction by sending some funds over to the sponsorWallet using `sponsorWallet.call{value: 0.01 ether}("")` . In most cases this amount should be enough to cover the fulfillment transaction, however it depends on the gas price and gas cost of the transaction, so vary it accordingly.

## generateQuantumons

https://gist.github.com/Ashar2shahid/5d982b04d507d5126e935e97bbe687cd#file-generatequantumon

When the airnode makes the fullfillment transaction it will call `generateQuantumon()`with `requestId` and `data` as arguments.

> Note: generateQuantumon() has the `onlyAirnodeRrp`modifier which means that only the `AirnodeRrp`contract can call this function.

we decode the data, using `abi.decode(data, (uint256))` . We decode using uint256 because we know that `data`only contains a uint256 random number. Using this random number we call `_pickRandomUniqueId()` to get a non minted Quantumon Id. We use `index` to keep track of the how many Quantumons have been minted and it also serves as the`tokenId` of the next mint.

We now call the `_mint()` function and supply it `destinationAddress, tokenId`as arguments. The destination address is the address we set in the `requestToSender` mapping for the specified requestId.

We also call the `_setTokenURI()` which maps the Quantumon `id`to the `tokenId`. (Remember: the tokenIds are minted sequentially, but we want the Quantumon ids to be minted “out of order”).

## Closing Thoughts

With that we have covered all but 1 function in the Quantumon contract. I’ll leave it upto you to decipher what it does. I’m sure it won’t take you long.

If you liked this guide, be sure to mint a [Quantumon](http://Quantumon.xyz) as a token of appreciation. Holders of Quantumon NFTs will be eligible for special rewards in future projects I build.