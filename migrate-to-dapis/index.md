---
title: Developers guide on switching your oracle Migrating to dAPIs
author: Ashar Shahid
date_published: 2023-04-10
category: Technology
image: "https://miro.medium.com/v2/resize:fit:700/1*ZQ4_lMwR_TwdYxe614uJFw.png"
---

Oracles provide a secure and reliable way for smart contracts to access real-time market data, which is essential for various DeFi applications.

API3 has developed an oracle node that is operated by the API Provider, removing the intermediary node layer, or middleman. This change in oracle architecture creates a scalable and transparent solution that enables first-party oracles to be aggregated according to user requirements. DeFi protocols looking to utilize first-party oracles do so by integrating dAPIs to their smart contracts.

Within developers in mind, dAPIs have been designed with a simple integration process that abstracts away the technical implementation of accessing data feeds. The API3 Market provides the ability to manage these data feeds while giving users access to a range of first-party data feeds. In the future, additional data feed services such as capturing [Oracle Extracted Value](https://medium.com/api3/oracle-extractable-value-oev-13c1b6d53c5b) (OEV) will be accessible once a dAPI has been integrated.

This tutorial will demonstrate how easy it is for developers to switch from Chainlink data feeds to API3’s first-party oracles.

## Choosing the Contract

To demonstrate how easy it is to port over from Chainlink to API3, we are going to port over the DeFi options contract from this C[hainlink tutorial](https://blog.chain.link/defi-call-option-exchange-in-solidity/).

We don’t need to go over the entire contract because we will only be modifying a small section of the contract to port it over but in summary, the contract enables users to create, buy, exercise and cancel options within the contract using Ethereum (ETH) and Chainlink (LINK) tokens. It makes use of Chainlink’s aggregator interface for obtaining price feeds while exercising options. For more of an in-depth explanation you can check out the [full guide](https://blog.chain.link/defi-call-option-exchange-in-solidity/).

Now lets look at the code we want to modify

https://gist.github.com/Ashar2shahid/fe7004c8abe8277acc404c693f59c17e#file-chainlinkoptions-sol

The contract initializes the AggregatorV3Interfaces via the constructor for both the Eth and Link datafeeds, these can later be used to fetch the price. The two functions `getEthPrice()` and `getLinkPrice()`use the AggregatorV3Interface to fetch the price and return it. The `options` struct defines how options are stored when they are created, along with the `ethOpts` and `linkOpts` array to store the created options.

## Porting it Over

Porting over the contract to use API3’s dAPIs in this options contract can be done in 3 easy steps:

1.  Fetching the proxy address of the “ETH/USD” and “LINK/USD” dAPIs from the [API3 Market](https://market.api3.org/dapis/goerli/ETH-USD)
2.  Import the proxy interface from API3 repo and set the proxy addresses in the constructor
3.  Replace the `getEthPrice()` and the `getLinkPrice()`logic with a `.read()` function call to the proxy interface

and that's it. You do not need to hold any special type of token to be able to read from the oracle, it completely free to read.

Over 100+ dAPIs are currently available on the api3 market and work on a self-funded basis i.e you can top up the gas wallets to start reading from the oracle. If the dAPI is already funded you just need to copy the proxy address as seen below:

![](https://miro.medium.com/v2/resize:fit:700/1*t-fqBL2NonPCs-NVufSIcw.png)

ETH/USD Feed on the market

Here’s the same DeFi options contract updated to use API3 dAPIs

https://gist.github.com/Ashar2shahid/004ebfa339e081941524516c1392928a#file-api3options-sol

As you can see we only needed to call `.read()` on the `IProxy(ethProxy)` interface to start reading from the datafeed. We ended up with lesser lines of code and a much simpler reading interface. You can try running the contract yourself on [REMIX](https://remix.ethereum.org/#url=https://gist.githubusercontent.com/Ashar2shahid/6da449b873d1b45e1e27ee01645c54a1/raw/775bfd74af280902d934e5c69a44d69e1ea49c30/API3Options.sol&lang=en&optimize=false&runs=200&evmVersion=null&version=soljson-v0.8.17+commit.8df45f5f.js). (Note: The remix version only allows you to open and close options in ETH for simplicity).

## Why use dAPIs ?

dAPIs have been designed to abstract away the technical implementation of data feeds. Once a dAPI has been imported to oracle contracts, the API3 DAO can redirect the dAPI mapping upon user requests. This means that data feeds can be upgraded from self-funded to managed dAPIs, or directed to read alternate reference data with zero technical implementation.

Any update to data feeds, or a lack thereof, can create opportunities for OEV, such as arbitrage and liquidations. During each of these interactions value is leaking from the dApp users to both searchers and validators. Once a dAPI has been integrated, DeFi protocols will be able to capture Oracle Extractable Value without any further technical implementation.

Additionally, switching from Chainlink to API3 data feeds means no major alterations to smart contracts. This mitigates the need for audits while keeping battle-tested code intact.
