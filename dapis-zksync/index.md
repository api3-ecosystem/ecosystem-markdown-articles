---
title: Using dAPIs on zkSync Era Testnet
subtitle: Integrate API3’s first-party data feeds in your smart contracts
author: Vansh Wassan
date_published: 2023-04-15
category: Technology
image: "https://miro.medium.com/v2/resize:fit:720/0*2QHd70zTEIH7pJAx"
---


_Photo by [Sandro Katalina](https://unsplash.com/@sandrokatalina?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)_


[dAPIs](https://docs.api3.org/explore/dapis/what-are-dapis.html) are continuously updated streams of off-chain data, such as the latest cryptocurrency, stock, and commodity prices. As such they play a crucial role in the infrastructure of DeFi.

  

[API3](https://api3.org/) uses first-party oracles to power decentralized APIs (dAPIs). dAPIs are secure, transparent, and cost-efficient data feeds that connect smart contracts directly to first-party data sources.

  

Over 100 forex & crypto data feeds are available to builders on zkSync Era Testnet. These can be [_viewed here_](https://market.api3.org/dapis?chains=zksync-goerli-testnet).

  

## API3 data feeds: dAPIs

  

[Self-funded dAPIs](https://docs.api3.org/guides/dapis/read-self-funded-dapi/) give DeFi builders access to real-time market data through first-party oracles. Users do this by funding a dAPI to be used for oracle transactions, thus activating the feed in a permissionless format.

  

To ensure accuracy and reliability, the data feeds are continuously updated by first-party oracles using signed data, which provides cryptographic proof of their authenticity. As a result, smart contracts can read the on-chain value of any dAPI in real-time.

  

![](https://miro.medium.com/v2/resize:fit:700/0*T-VoTbkWanABxvJQ)

  

In the coming months, multi-source data feeds will be usable that see beacon sets (aggregated first-party feeds) consumable through a dAPI. The API3 DAO configures a dAPI mapping as requested, also enabling OEV to be captured later in the year.

  

## API3 Market

  

The [API3 Market➚](https://market.api3.org) is a user-friendly interface to seamlessly access dAPI services. For self-funded dAPIs, this end-to-end process consists of:

  

- Browsing and selecting your dAPI

- Funding the dAPI

- Deploying a proxy contract to access the data feed

  

![](https://miro.medium.com/v2/resize:fit:700/1*v-PcKvCDkQDcKkptZF8gBQ.png)

  

## Using dAPIs in your Smart Contracts

  

### Before you start

  

Users need to make sure they have:

  

- Metamask connected to zkSync Era Testnet.

- Testnet Goerli Eth on zkSync Era Testnet. _To bridge your Goerli ETH to zkSync Era Testnet,_ [_click here_](https://goerli.bridge.zksync.io/)_._

  

Or you can also refer to this repository to get started.

  

- [https://github.com/api3dao/data-feed-reader-example](https://github.com/api3dao/data-feed-reader-example)

  

## Funding the dAPIs and Deploying a Proxy

  

To integrate self-funded dAPIs in your smart contracts, head to the API3 Market and select the dAPIs you would like to use. Make sure you’re on the same network where you want to use the dAPI.

  

For this tutorial, we’ll use the ETH/USD dAPI on zkSync Era.

  

![](https://miro.medium.com/v2/resize:fit:700/1*nzVL-u77sq9ZmOhEae_ZXQ.png)

  

With self-funded dAPIs, you can fund the dAPI with your funds. The amount of gas you supply will determine how long your dAPI will be available for use. If you run out of gas, you can fund the dAPI again to keep it available for use.

  

The API3 Market has an estimation tool that helps you manage this.

  

If the dAPI is already funded for your required time, you can click on **Get Proxy** and start using it.

  

To fund a dAPI, click on the **Fund Gas** button.

  

![](https://miro.medium.com/v2/resize:fit:596/1*rlCPM_E3RxCv6q40OHGghA.png)

  

Use the gas estimator to select how much gas is needed by the dAPI. Click on **Send ETH** to send the entered amount to the sponsor wallet of the respective dAPI.

  

![](https://miro.medium.com/v2/resize:fit:600/1*q5tK5sPfKCbswkDzhgVmYg.png)

  

Once the transaction is broadcasted and confirmed on the blockchain a transaction confirmation screen will appear.

  

![](https://miro.medium.com/v2/resize:fit:700/1*YIQ6EsHWLeMhwkrz2mvl6Q.png)

  

You can now click on the **Get Proxy** button to deploy the proxy contract to read from the dAPI. If the proxy contract for a dAPI is already deployed, it will show you its address.

  

![](https://miro.medium.com/v2/resize:fit:700/1*LwKRJ2nO-5gYtjWXSdS_bg.png)

  

You now press ‘Get Proxy’ where you read the most recent value of the data feed.

  

![](https://miro.medium.com/v2/resize:fit:700/1*J2U-9qtsHdXrdkN3XifTdQ.png)

  

The dashboard will look similar to the below with the last updated value visible.

  

![](https://miro.medium.com/v2/resize:fit:700/1*BSFtabzPC7xMe896xzlZYQ.png)

  

## Coding and Deploying the Contract

  

Now we will code a simple contract that returns the price of Ethereum using the ETH/USD dAPI.

First, import the required contracts. After importing, we can make the contract and inherit from _Ownabe_.

[https://docs.api3.org/guides/dapis/read-self-funded-dapi/](https://docs.api3.org/guides/dapis/read-self-funded-dapi/)

-  `setProxy()` is used to set the address of the dAPI Proxy Contract.

- The `readDataFeed()` function will call the `read()` function from the `IProxy` interface. This will return the data feed value and timestamp on-chain.


Users can now use the data feed `value` returned in any way they want.

You can refer to this [example project](https://github.com/vanshwassan/DataFeedReader-zkSync) on how to deploy and read the `DataFeedReader` contract on zkSync Era Testnet.

```shell
$ git clone https://github.com/vanshwassan/DataFeedReader-zkSync.git
```

- Install all the packages:

```shell
$ yarn
```
  
- Make a `.env` file similar to `example.env` and add your private key:

```shell
$ echo 'PRIVATE_KEY=' > .env
```

- To compile the `DataFeedReader` contract:

```shell
$ yarn compile
```
  
- To deploy the `DataFeedReader` contract:

```shell
$ yarn deploy
```
 
- To read the data feed value:

```shell
$ yarn readDapi
```

You can also refer to [this Repo](https://github.com/api3dao/data-feed-reader-example) for a more detailed example.

Check out the [API3 Docs](https://docs.api3.org/) for more guides.


## Conclusion

dAPIs have been designed to abstract away the technical implementation of data feeds with a modular design approach made possible by first-party oracles. Once a dAPI has been imported to oracle contracts, the API3 DAO can redirect the dAPI mapping upon user requests to other API3 data feeds or services.

This means once the straight forward steps above have been followed you can utilize first-party oracles. To learn more or get started with dAPIs head to the [API3 Docs](https://docs.api3.org/explore/dapis/what-are-dapis.html).

You can also check [this guide on how to switch your oracle from Chainlink and start using dAPIs.](https://medium.com/@ashar2shahid/developers-guide-on-switching-your-oracle-migrating-to-dapis-eb53e673c793)

[Click here](https://discord.gg/qnRrcfnm5W) to join our Discord if you have any questions.
