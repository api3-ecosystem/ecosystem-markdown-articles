---
title: How to Call Any API From a Solidity Smart Contract, Part I
subtitle: Deploying an Airnode to get off-chain data in your Smart Contract
author: Vansh Wassan
date_published: 2022-09-07
category: Technology
image: "https://images.unsplash.com/photo-1655635643532-fa9ba2648cbe?ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D&auto=format&fit=crop&w=1632&q=80"
---
​
_Photo by [DeepMind](https://unsplash.com/@deepmind?utm_source=medium&utm_medium=referral) on [Unsplash](https://unsplash.com/?utm_source=medium&utm_medium=referral)_

As we all know that it is not possible by a Smart Contract to directly access external APIs outside of a blockchain. Interacting with off-chain data while working with Smart Contracts is a real problem for a lot of dApps.

> The Ethereum blockchain was designed to be entirely deterministic while the Internet was not. Calling an API directly sounds easy but on a blockchain, it would require all the nodes to call the same endpoint at the same time and expect to get the same data in order to come to a consensus. Plus, the core concept of a blockchain is its security, that is derived from a decentralized network of independent validators that purposefully limit their connection to the outside world.

However, with [API3](https://api3.org/), you can have first-party oracles that are directly operated by the API Providers, called [Airnodes](https://docs.api3.org/explore/airnode/what-is-airnode.html) that provides data to any on-chain dApp. As a result, you can easily make any REST API accessible to a Smart Contract.

## Say Hello to ChainAPI

[ChainAPI](https://bit.ly/Va-Part1-Polygon-ChainAPI) is a platform that enables you to integrate and deploy the open-source Airnode with its step-by-step integration and deployment tools.

To get started, go to [ChainAPI](https://bit.ly/Va-Part1-Polygon-ChainAPI) and log in by connecting your MetaMask.

![](https://miro.medium.com/v2/resize:fit:700/0*135nPlhwqamY3-zY)

You will be prompted to confirm and sign the transaction through your MetaMask extension.

Make sure you’re using a new MetaMask Wallet with a fresh mnemonic. Your mnemonic will later be used to deploy the Airnode. You need to keep it extremely safe as this will serve as the “private key” of your deployed Airnode.

_Each time you return to ChainAPI you will connect again, using MetaMask, to identify yourself by signing a message for the same account._

![](https://miro.medium.com/v2/resize:fit:700/1*-v8l867Nje8Fgb9wgcIHzw.png)

Complete the signup process and name your workspace.

_Workspaces provides you with a way to invite other users to help or collaborate with integrations and deployments. This makes it easy to manage your Airnodes as a team or to outsource the process while still maintaining control over your integrations and deployments._

![](https://miro.medium.com/v2/resize:fit:700/0*D9FbnKKEeM3U82vb)

_To change the name of your workspace in the future, click on name on the top-left of the dashboard_

Within ChainAPI you will be able to create and manage your integrations or Airnode deployments by navigating to the “Integrations” or “Deploy” dashboards on the left hand navigation panel.

![](https://miro.medium.com/v2/resize:fit:700/0*xEsNLFmPTUO7ULPQ)

## Integrating your Airnode

For this tutorial, I am going to use [dxFeed’s public REST API](https://bit.ly/3TwKnQI) endpoints to retrieve stock data.

To get started select the “Integrate API” option in the top right hand of the dashboard.

Enter the details about the API you want to Integrate.

![](https://miro.medium.com/v2/resize:fit:700/0*UUoWo8dgZX4UdjWj)

You need to enter the base URL of your API along with all the endpoints that you want to integrate. If your API requires any security scheme (API Key, Basic HTTP Auth) you have the option to add that too.

As it’s a public API, it doesn’t have any security schemes.

![](https://miro.medium.com/v2/resize:fit:597/0*GrYQa8tGB8PisfDM)

You can now start by adding all your endpoints.

Here, the dxFeed REST API has one `GET` endpoint `/events.json` with some query parameters. You can add all the parameters that your API requires.

The parameters that are required by dxFeed’s API are:

-   `events` — Query Parameter — Takes in the market event. It will be user defined.
-   `symbols` — Query Parameter — Takes in the stock ticker symbol. It will be user defined.

For more information about dxFeed’s API and how it works, [click here](https://tools.dxfeed.com/webservice/rest-demo.jsp) to test out their API.

![](https://miro.medium.com/v2/resize:fit:700/1*PnjLKrxFdcerKRlIzPFY_g.png)

Now you need to add all the parameters and define where they go (query/header/path/cookie). You can also decide if you want their values fixed or not.

Here, the dxFeed REST API has one GET endpoint `/events.json` with some query parameters. You can add all the parameters that your API requires.

[Reserved parameters](https://docs.api3.org/reference/ois/latest/reserved-parameters.html) define what part of the response is to be picked and encoded before fulfillment. It can be defined by the requester but we can also hardcode it in the Airnode configuration.

![](https://miro.medium.com/v2/resize:fit:700/0*ax-C3u0OzPQSgOs8)

You can also add [pre and post-processing snippets](https://docs.api3.org/reference/ois/latest/processing.html) for your Airnode. Although we won’t be using this feature with this integration.

-   [Pre-processing](https://docs.api3.org/reference/ois/latest/specification.html#_5-9-preprocessingspecifications) snippets are executed before making the request to the Airnode.
-   [Post-processing](https://docs.api3.org/reference/ois/latest/specification.html#_5-10-postprocessingspecifications) snippets are executed after receiving the response from the Airnode.

After adding all the required endpoints, you can now press finish and get ready to deploy your Airnode.

![](https://miro.medium.com/v2/resize:fit:700/0*9AF-rwD_ud2kAf75)

## Deploying your Airnode

To deploy the Airnode, go to the deploy section on the menu. Name your deployment and select the integration that you want to use with it.

![](https://miro.medium.com/v2/resize:fit:700/0*ZgK_WRkE_XQRtgGJ)

Select your Cloud Provider where you want your Airnode to be deployed.

![](https://miro.medium.com/v2/resize:fit:700/1*Sr-ngqDgK2wxN9B_JaLoxQ.png)

Now select the Chains for your deployment. You can also select multiple networks and providers if you want it on multiple Chains.

Here, we are going to have our Airnode on the Polygon Mumbai Testnet.

![](https://miro.medium.com/v2/resize:fit:700/1*BFdzGEys6RIw6AZAp2JYqQ.png)

[Authorizer](https://docs.api3.org/reference/airnode/latest/concepts/authorizers.html) contracts allow you to specify which smart contracts can make requests to your Airnode’s endpoints. For this tutorial, we are just going to set it as Public.

When an Airnode receives a request, it can use on-chain authorizer contracts to verify if a response is warranted. This allows the Airnode to implement a wide variety of policies and to authorize requester contract access to its underlying API.

-   Public Authorizers will allow any smart contract to make requests to your Airnode.
-   Restricted Authorizers will only allow smart contract addresses that have been granted access to make requests to your Airnode.

[Click here to learn more about Authorizations.](https://docs.api3.org/reference/airnode/latest/concepts/authorizers.html)

Review your configuration for one final time. If everything seems correct, click on next.

![](https://miro.medium.com/v2/resize:fit:700/0*YYS4QaLfdWI8RuyJ)

Download all the Airnode configuration files and extract them.

![](https://miro.medium.com/v2/resize:fit:667/0*GDIIGLavatCbh33I)

This is what your Airnode config directory should look like:

![](https://miro.medium.com/v2/resize:fit:562/0*SqysW0CmYa_8azbB)

`config`  contains `config.json` and `secrets.env`.

-   The `config.json` file is used during the deployment/redeployment of an Airnode to configure its behavior and to provide mappings of API operations.
-   The `secrets.env` file holds values for config.json that must be kept secret.

The output directory will have the `receipt.json` that will be generated after you successfully deploy the Airnode.

The `aws.env` file holds AWS credentials for deployments targeted to AWS.

_As we are using AWS as our cloud provider, we need to add our AWS IAM Access Keys with the Administrator Access policy. You can refer to this_ [_video_](https://www.youtube.com/watch?v=KngM5bfpttA) _if you are not sure how to obtain them._

The `README.md` contains all the steps to deploy the Airnode provided in a markdown format.

Open `aws.env` and add your `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` that you just created.

![](https://miro.medium.com/v2/resize:fit:615/1*4KWNLu2t-JEvJugQcTb4Mw.png)

Open `config/secrets.env`  and add your wallet mnemonic. Make sure you keep it extremely safe as this will serve as the “private key” of your deployed Airnode. From the mnemonic phrase, Airnode is able to assign wallet addresses to both the Airnode instance and its users.

![](https://miro.medium.com/v2/resize:fit:700/1*StjSe291ojNheWeotTBfPg.png)

You also need to add your Blockchain Provider URL. Here, we are going to use Alchemy for a free Polygon Mumbai Testnet Provider URL. You can use any blockchain provider that supports your network.

![](https://miro.medium.com/v2/resize:fit:700/1*LwibimEzWAdfwAWz5lUY5A.png)

You can also set-up your [_HttpGateway_](https://docs.api3.org/reference/airnode/latest/understand/http-gateways.html)  credentials. It’s an optional service that allows authenticated users to make HTTP requests to your deployed Airnode instance for testing. ChainAPI has already generated these keys for you but you can change them if you want.

![](https://miro.medium.com/v2/resize:fit:700/1*wJNFvs_UmCfd1vfL1wpL3A.png)

One final step before deploying your Airnode is to set [Authorizers](https://docs.api3.org/reference/airnode/latest/concepts/authorizers.html) in the _config.json_ file.

_When an Airnode receives a request, it can use on-chain authorizer contracts to verify if a response is warranted. This allows the Airnode to implement a wide variety of policies and to authorize requester contract access to its underlying API._

For the scope of this tutorial, we can set the authorizer array empty in _config.json_ so that any requester contract can access the Airnode.

![](https://miro.medium.com/v2/resize:fit:553/1*a9kG6ZyESgSdbqw5uyZaJg.png)

Now you’re ready to deploy your Airnode. Make sure you have [Docker](https://www.docker.com/) installed on your system.

Copy and paste the commands below to your terminal at the root directory of your deployment package.

Windows
```shell
docker run -it --rm ^  
      --env-file aws.env ^  
      -v "%cd%/config:/app/config" ^  
      -v "%cd%/output:/app/output" ^  
      api3/airnode-deployer:0.7.3 deploy
```

OSX

```shell
docker run -it --rm \  
      --env-file aws.env \  
      -e USER_ID=$(id -u) -e GROUP_ID=$(id -g) \  
      -v "$(pwd)/config:/app/config" \  
      -v "$(pwd)/output:/app/output" \  
      api3/airnode-deployer:0.7.3 deploy
```

Linux
```shell
docker run -it --rm \  
      --env-file aws.env \  
      -e USER_ID=$(id -u) -e GROUP_ID=$(id -g) \  
      -v "$(pwd)/config:/app/config" \  
      -v "$(pwd)/output:/app/output" \  
      api3/airnode-deployer:0.7.3 deploy
```

![](https://miro.medium.com/v2/resize:fit:484/0*KZ8A_iw2ws-5Duig)

Your Airnode should now be deployed. You can check its status in the deployment section.

Check out the GitHub Repo for this guide [here](https://github.com/vanshwassan/AirnodeTutorials/).

![](https://miro.medium.com/v2/resize:fit:700/0*KtPyklodLfrFv-IW)

In the [part 2](/articles/call-airnode-2), we will show you how to code a Requester contract to call and read the data from the Airnode.
