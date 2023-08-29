# Web3Auth x Solana x React Demo Application

## Quick Start

```bash
git clone https://github.com/shahbaz17/web3auth-solana-react-demo.git
cd web3auth-solana-react-demo
npm install
npm start
```

## Guide
For this guide here, we will be talking through the Web3Auth Plug and Play Modal SDK and using the OpenLogin Provider alongside it to enable Social Authentication through Firebase.

### Steps to Configure web3Auth dashboard
Setup your Web3Auth Dashboard

Create a Project from the Plug and Play Section of the [Web3Auth Developer Dashboard](https://dashboard.web3auth.io/).

Plug n Play Project Creation on Web3Auth Dashboard

![image](https://github.com/solDarkLord/web3Auth-Solana/assets/143541066/feab72b9-057d-4e62-8cd8-9bc31f2b7a13)

Enter your desired Project name

Select the Web3Auth Network as testnet. We recommend creating a project in tesnet network during development. And while moving to a production environment, make sure to convert your project to mainnet, aqua, or cyan network, otherwise you'll end up losing users and keys.

Select the blockchain(s) you'll be building this project on. For interoperability with Torus Wallet, you've an option of allowing the user's private key to be used in other applications using Torus Wallet. We currently have this option across EVM, Solana and Casper blockchains.

Finally, once you create the project, you've the option to whitelist your URLs for the project. Please whitelist the domains where your project will be hosted.

![image](https://web3auth.io/docs/assets/images/plugnplay-whitelist-797c7d41d9e8a12a469bf093236b4eb9.png)

Plug n Play Project - Whitelist

Replcae the clientId variable in App.tsx file inside src directory at line 14 with the client ID generated in the dashboard

## Important Methods Implemented 

#### Web3Auth init

```Js
const init = async () => {
  try {
    const web3auth = new Web3Auth({
      clientId,
      chainConfig: {
        chainNamespace: CHAIN_NAMESPACES.SOLANA,
        chainId: '0x3', // Please use 0x1 for Mainnet, 0x2 for Testnet, 0x3 for Devnet
        rpcTarget: 'https://api.devnet.solana.com',
        displayName: 'Solana Devnet',
        blockExplorer: 'https://explorer.solana.com/?cluster=devnet',
        ticker: 'SOL',
        tickerName: 'Solana Token',
      },
    });

    await web3auth.initModal();
    setWeb3auth(web3auth);

    if (web3auth.provider) {
      setProvider(web3auth.provider);
    }
  } catch (error) {
    console.error(error);
  }
};
```

#### Get User Info : To print Authenticated user info

```Js
 const getUserInfo = async () => {
    if (!web3auth) {
      uiConsole("web3auth not initialized yet");
      return;
    }
    const user = await web3auth.getUserInfo();
    uiConsole(user);
  };
```
#### Get Account : Get Public key of account generated

```Js
const getAccounts = async () => {
  if (!provider) {
    uiConsole('provider not initialized yet');
    return;
  }
  const solanaWallet = new SolanaWallet(provider);
  // Get user's Solana public address
  const accounts = await solanaWallet.requestAccounts();
  uiConsole(accounts);
};
```

#### Get Balance : Get Balance of account generated

```Js
const getBalance = async () => {
  if (!provider) {
    uiConsole('provider not initialized yet');
    return;
  }
  const solanaWallet = new SolanaWallet(provider);
  // Get user's Solana public address
  const accounts = await solanaWallet.requestAccounts();

  const connectionConfig: any = await solanaWallet.request({
    method: 'solana_provider_config',
    params: [],
  });

  const connection = new Connection(connectionConfig.rpcTarget);

  // Fetch the balance for the specified public key
  const balance = await connection.getBalance(new PublicKey(accounts[0]));
  uiConsole(balance / 1000000000);
};
```
#### Get Private Key : Get Private Key of account generated

```Js
const fetchPrivateKey = async () => {
  if (!web3auth) {
    uiConsole('web3auth not initialized yet');
    return;
  }
  const privateKey = await web3auth?.provider?.request({
    method: 'solanaPrivateKey',
  });
  uiConsole(privateKey);
};
```
#### Sign transaction : Sign a transaction using the private key of the account generated

```Js
const signaTransaction = async () => {
  if (!provider) {
    uiConsole('provider not initialized yet');
    return;
  }
  const solanaWallet = new SolanaWallet(provider);
  const connectionConfig: any = await solanaWallet.request({
    method: 'solana_provider_config',
    params: [],
  });

  const connection = new Connection(connectionConfig.rpcTarget);

  const accounts = await solanaWallet.requestAccounts();
  const block = await connection.getLatestBlockhash('finalized');
  const TransactionInstruction = SystemProgram.transfer({
    fromPubkey: new PublicKey(accounts[0]),
    toPubkey: new PublicKey(accounts[0]),
    lamports: 0.01 * LAMPORTS_PER_SOL,
  });
  const transaction = new Transaction({
    blockhash: block.blockhash,
    lastValidBlockHeight: block.lastValidBlockHeight,
    feePayer: new PublicKey(accounts[0]),
  }).add(TransactionInstruction);

  const signedTx = await solanaWallet.signTransaction(transaction);
  uiConsole(signedTx?.signature?.toString('hex'));
};
```
#### Sign and send a transaction : Sign a transaction using the private key of the account generated and send it to the network

```Js
const signandsendTransaction = async () => {
  if (!provider) {
    uiConsole('provider not initialized yet');
    return;
  }
  const solanaWallet = new SolanaWallet(provider);

  const connectionConfig: any = await solanaWallet.request({
    method: 'solana_provider_config',
    params: [],
  });

  const connection = new Connection(connectionConfig.rpcTarget);

  const accounts = await solanaWallet.requestAccounts();
  const block = await connection.getLatestBlockhash('finalized');
  const TransactionInstruction = SystemProgram.transfer({
    fromPubkey: new PublicKey(accounts[0]),
    toPubkey: new PublicKey('8Q3KAP8nV9FAMmtpm3QqoECbafebscsSQ3m6HHgK386v'),
    lamports: 0.01 * LAMPORTS_PER_SOL,
  });
  const transaction = new Transaction({
    blockhash: block.blockhash,
    lastValidBlockHeight: block.lastValidBlockHeight,
    feePayer: new PublicKey(accounts[0]),
  }).add(TransactionInstruction);
  const { signature } = await solanaWallet.signAndSendTransaction(
    transaction,
  );
  uiConsole(signature);
};
 ```



### Base Library 

#### Web3Auth Library 

1. @web3auth/modal : This is the main Core package that contains the Web3Auth SDK.
2. @web3auth/openlogin-adapter : For using Custom Authentication, we need to use the OpenLogin Adapter, where we can initialise the authentication details.
3. @web3auth/base : Since we are using Typescript, we need the @web3auth/base package to provide types of the different variables we'll be using throughout the app building process. This reduces errors to a very large extent.
4. @web3auth/solana-provider : Web3 Auth solana provider

####  Solana Web3 Library 

1. @solana/web3.js : Web3 library to interact with solana blockchain

### Pollyfills 

If you are starting from scratch, to set up this project locally, you will need to create a base Web application, where you can install the required dependencies. However, while working with Web3, there are a few base libraries, which need additional configuration. This is because certain packages are not available in the browser environment, and we need to polyfill them manually. 

This issue is due to the fact that core packages like eccrypto have certain dependencies which are not present within the build environment. Webpack 5 especially introduced a lot of issues since it does not include many of these packages by default. The go-to method for rectifying these issues has been to simply add the missing modules directly into the package, and edit the bundler configuration to take advantage of that.

1. Install react-app-rewired into your application
2. Install Buffer and Process into your application

There might be a possibility that you might need to polyfill more libraries, in case you're using any other blockchain library alongside Web3Auth that requires them. Generally the libraries like crypto-browserify, stream-browserify, browserify-zlib, assert, stream-http, https-browserify, os-browserify, url are the ones that might be required. crypto-browserify and stream-browserify being the most common polyfills.

Create config-overrides.js in the root of your project folder with the content:

```Js
const webpack = require("webpack");

module.exports = function override(config) {
  const fallback = config.resolve.fallback || {};
  Object.assign(fallback, {
    crypto: false, // require.resolve("crypto-browserify") can be polyfilled here if needed
    stream: false, // require.resolve("stream-browserify") can be polyfilled here if needed
    assert: false, // require.resolve("assert") can be polyfilled here if needed
    http: false, // require.resolve("stream-http") can be polyfilled here if needed
    https: false, // require.resolve("https-browserify") can be polyfilled here if needed
    os: false, // require.resolve("os-browserify") can be polyfilled here if needed
    url: false, // require.resolve("url") can be polyfilled here if needed
    zlib: false, // require.resolve("browserify-zlib") can be polyfilled here if needed
  });
  config.resolve.fallback = fallback;
  config.plugins = (config.plugins || []).concat([
    new webpack.ProvidePlugin({
      process: "process/browser",
      Buffer: ["buffer", "Buffer"],
    }),
  ]);
  config.ignoreWarnings = [/Failed to parse source map/];
  config.module.rules.push({
    test: /\.(js|mjs|jsx)$/,
    enforce: "pre",
    loader: require.resolve("source-map-loader"),
    resolve: {
      fullySpecified: false,
    },
  });
  return config;
};
```
Within package.json change the scripts field for start, build and test. Instead of react-scripts replace it with react-app-rewired

```Json
"scripts": {
    "start": "react-app-rewired start",
    "build": "react-app-rewired build",
    "test": "react-app-rewired test",
    "eject": "react-scripts eject"
},
```



