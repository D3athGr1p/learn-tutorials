One of the key tenets of Web 3 is composability, so you can think of this step as a precursor for a more advanced blog entry marketplace. Maybe you or someone else wants to build a dApp that allows people to buy and sell blog entry ownership.

Because the Web 3 stack is decentralized, it means data no longer lives behind a walled garden where the app owners control access and own the data. In Web 3 users retain not only ownership over the data, but the ability to port their data to other UIs that can access it directly from the blockchain.

![Hooray! No more walled gardens for data](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/mirror/assets/free.jpeg)

{% label %}
Hooray! No more walled gardens for data

# Implementation 🧩

In order to make each entry's NFT transferable, we need to write a function that activates the blue **Transfer NFT** button shown on each entry's page. If we go to the `TransferNFTForm.tsx` component, we see a `handleSubmit` button with some simple instructions to connect the smart contract to a signer, and to call `transferFrom` on the smart contract.

Recall from Step 6 that we can instantiate a signer by calling `getSigner` on `provider`. Recall also that we can connect the `signer` to the `contract` as well, so we can use that code here to connect the smart contract and the signer at the top of `handleSubmit`:

```typescript
const signer = provider.getSigner();
const contractWithSigner = contract.connect(signer);
```

# Transferring an NFT ♻️

Now all that's left is calling the `transferFrom` function in the contract. You might be wondering where that function is since we didn't write it in Step 6 when we finalized the `MirrorClone.sol` contract.

The `transferFrom` function actually comes from the ERC721 standard. We inherited the function from the OpenZeppelin ERC721 contract, which means we can call it anytime even though we didn't explicitly write it in our contract. If you open `node_modules/@openzeppelin/contracts/token/ERC721/ERC721.sol`, on line 152 you can see the function requires a from address, a to address, and a tokenId.

Note that it is preferable in many cases to use the `safeTransferFrom` method because otherwise it is possible to send an NFT to an address that is unaware of the ERC721 protocol and is therefore not guaranteed to be able to handle the NFT properly. This is an advanced topic, so for the purposes of this tutorial `transferFrom` will suffice.

We can complete the `handleSubmit` function by calling `transferFrom` on our `contractWithSigner`:

```typescript
const resp = await contractWithSigner.transferFrom(
  address,
  recipient,
  tokenId,
);
const rec = await resp.wait();

console.log(rec);
```

With that in place, we can now test it by transferring the NFT. You should create another account inside your MetaMask wallet by clicking the big colorful button on the top right and then clicking "Create Account".

Once you do this, you'll have another public address controlled by your private key. Copy that address and paste it into the **Recipient Address** field. Then go back to your original account and click **Transfer NFT**.

After accepting the transaction on MetaMask, your second account will own the entry (once the transaction is confirmed)!

##### _Listing 7.1: Code for transferring an NFT_
```typescript
const handleSubmit = useCallback(
  .
  .
  .
  // Connect smart contract to signer
  const signer = provider.getSigner();
  const contractWithSigner = contract.connect(signer);

  // Call `transferFrom` method of smart contract
  const resp = await contractWithSigner.transferFrom(
    address,
    recipient,
    tokenId,
  );
  const rec = await resp.wait();

  console.log(rec);
  .
  .
  .
);
```

# Challenge 🏋️

Navigate to `components/TransferNFTForm/TransferNFTForm.tsx` in your code editor and follow the steps included as comments to finish writing the `handleSubmit` function for transferring NFTs. We include a description along with a link to the documentation you need to review in order to implement each line. The relevant code block is also included in **Listing 7.2** below.

##### _Listing 7.2: Instructions for transferring an NFT_
```typescript
const handleSubmit = useCallback(
  .
  .
  .
  // Connect smart contract to signer
  // Call `transferFrom` method of smart contract
  .
  .
  .
);
```

Once you have completed the code, you can try transferring an entry NFT by pasting the address of the account you want to transfer the NFT to into the textinput, then clicking on **Transfer NFT** and confirming the transaction in MetaMask:

![Screenshot of transfer](https://raw.githubusercontent.com/figment-networks/learn-tutorials/master/mirror/assets/transfer.jpg)