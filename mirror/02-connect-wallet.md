The first step for any dApp is setting up a way to connect to a user's wallet. This allows the dApp to authenticate the user without using traditional passwords. More importantly, this will allow the dApp to pull the user's onchain data so we can build user-specific dApp experiences.

{% sidenote title="Box 2.1: A note on crypto wallets" %}
Crypto wallets are the gateway through which users interact with blockchain protocols. The wallet locally stores a user's private keys encrypted by a password and allows the user to sign transactions that prove control of an account's keys in a trustless way. Developers can leverage this technology to authenticate users and build applications where users control their own data in a portable way. For a more in-depth look at wallets, check out the [Solana Wallet](https://learn.figment.io/tutorials/solana-wallet-intro) tutorial.

In this case, we'll be using a JavaScript library called [ethers.js](https://docs.ethers.io/) to interact with a user's browser extension wallet like MetaMask. This will allow us to detect whether the wallet is installed, and access the user account's API for getting its data and leveraging its functionality.

# Implementation 🧩

The first step in performing any blockchain operation is typically instantiating a connection. This can be accomplished with the **Providers** class in ethers.js. A Web 3 provider object includes functionality to read from a protocol.

If you're familiar with React development, you can probably guess that we'll need a way to preserve application state after we connect to the wallet. That way we don't have to make the user re-connect their wallet every time the page is rendered. For this, we'll use React's [Context](https://reactjs.org/docs/context.html).

There are several ways to implement this, but in our case we'll be using a context provider (not to be confused with the Web 3 provider!) we built into our template in `context/web3Context.tsx`. 

If we look at the Web3Provider function, notice we're using a simple implementation with the ethers.js **Web3Provider**:

```typescript
const provider =
    typeof window == 'undefined' || !window.ethereum
      ? null
      : new ethers.providers.Web3Provider(window.ethereum);
```

Focusing on the last line, notice that we're instantiating a provider based on `window.ethereum`. That object comes [courtesy of MetaMask](https://docs.metamask.io/guide/ethereum-provider.html), so we can pass it in as an argument to instantiate our Web 3 provider object. The rest of the code is simply checking whether the MetaMask extension is present, so the dApp still runs even if there isn't one. 

Once we have a Web 3 provider, we can use the provider's `send` method to request the accounts associated with the wallet we want to connect (it's good to learn about how [Hierarchical Deterministic wallets](https://docs.ethers.io/v5/api/utils/hdnode/) work). Then we can instantiate a **Signer** object that will give us access to functions that allow us to change the state of the blockchain.

In our case, we have a `connect` function that first checks whether a provider is present, and then should get our user's public address and set it in the app state using a React hook: `setAddress`. We've included a check for the chainId, to ensure that users are connected to the Mumbai testnet - every blockchain network uses a unique chain identifier, to prevent transactions from one blockchain being replayed on another. Also, if the user doesn't have MetaMask installed it should alert them to install it.

We can complete this function with four simple lines. First, we need to request the accounts available in the MetaMask provider. Then we need to instantiate a signer that will then give us access to methods to get the user's current address and the chainId. 

```typescript
await provider.send('eth_requestAccounts', []);
const signer = provider.getSigner();
const currentAddress = await signer.getAddress();
const chainId = await signer.getChainId();
```

{% sidenote title="Box 2.2: Without centralized servers, what are we connecting to?" %}
Going deep into something like the Ethereum Virtual Machine is beyond the scope of this tutorial, but this question is critical if you want to better understand the backend you're building on. Instead of connecting to a specific server and reading from a database, we're reading from a distributed state machine. It's distributed because there are lots of nodes all over the world holding the same state. It's a state machine because it's in one state at any given time. We interact with it and compete with others using it to update a slice of its state that will get recorded before the next cycle of state updates. _Don't confuse the state of the EVM with the state of a React app, they're similar concepts but not the same thing!_

Putting these four lines together with our `connect` function and understanding that our custom hook is already in use, our users should now be able to connect by clicking on the **Connect Wallet** button at the top right.

![Forget logins, now our users can connect to our app with their wallets!](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/mirror/assets/wallet.gif)

{% label %}
Forget logins, now our users can connect to our app with their wallets!

##### _Listing 2.1: Code for connecting a wallet_
```typescript
const connect = useCallback(async () => {
  if (provider) {
    await provider.send('eth_requestAccounts', []);
    const signer = provider.getSigner();
    const currentAddress = await signer.getAddress();
    const chainId = await signer.getChainId();

    if (chainId != (80001 as number)) {
      alert('Please connect to the Polygon Mumbai testnet in MetaMask!')
    }

    setAddress(currentAddress);
  } else {
    alert('Please install MetaMask at https://metamask.io');
  }
}, [provider]);
```

# Challenge 🏋️

Navigate to `context/web3Context.tsx` in your code editor and follow the steps included as comments to finish writing the `Web3Provider` function. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in **Listing 2.2** below.

##### _Listing 2.2: Instructions for connecting a wallet_
```typescript
const connect = useCallback(async () => {
  if (provider) {
    // Request accounts, get signer, signers address & chainId
    // More information can be found at https://docs.ethers.io/v5/getting-started/#getting-started--connecting

    const currentAddress = '';
    const chainId = 0;

    if (chainId != (80001 as number)) {
      alert('Please connect to the Polygon Mumbai testnet in MetaMask!')
    }

    // Read more about React hooks: https://reactjs.org/docs/hooks-intro.html
    // The setAddress React hook sets the value of address in the app state
    setAddress(currentAddress);
  } else {
    alert('Please install MetaMask at https://metamask.io');
  }
}, [provider]);
```

 Once you've completed the code, you will want to try connecting your MetaMask wallet by clicking on the **Connect Wallet** button in the top right corner of the screen:

 ![Screenshot of connecting wallet](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/mirror/assets/connect.jpg)
