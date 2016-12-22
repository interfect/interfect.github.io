# How to Use Tokens on the Ethereum Classic Blockchain

The Ethereum Classic blockchain is a cheap place to deploy your own custom cryptocurrency token or other smart contract. However, setting up the software required to manipulate these tokens is not always easy. This guide explains how to do it, step by step. By the end, you should be able to receive new kinds of cryptocurrency tokens, check your balance, and send them to others.

## Step 1: Get Details on your Token

The first step to using a token on Ethereum Classic is getting two pieces of information: the token's contract address (which describes where it is), and the token's "ABI" or "interface" (which describes how to use it). If your token follows the the [ERC-20 standard](https://github.com/ethereum/EIPs/issues/20), which defines a standard interface for tokens, then you may not need the ABI (because it's standard).

For this example, we will be using my "NovakBucks" token, which is deployed at this address:

```
0x73212fb39622414c660a140bf37138165a44d9c0
```

It uses this ABI, which is ERC-20 compliant:

```
[{"constant":true,"inputs":[],"name":"name","outputs":[{"name":"","type":"string"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"}],"name":"approve","outputs":[{"name":"success","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"totalSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_from","type":"address"},{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transferFrom","outputs":[{"name":"success","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"decimals","outputs":[{"name":"","type":"uint8"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"standard","outputs":[{"name":"","type":"string"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"","type":"address"}],"name":"balanceOf","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"constant":true,"inputs":[],"name":"symbol","outputs":[{"name":"","type":"string"}],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_to","type":"address"},{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[],"payable":false,"type":"function"},{"constant":false,"inputs":[{"name":"_spender","type":"address"},{"name":"_value","type":"uint256"},{"name":"_extraData","type":"bytes"}],"name":"approveAndCall","outputs":[{"name":"success","type":"bool"}],"payable":false,"type":"function"},{"constant":true,"inputs":[{"name":"","type":"address"},{"name":"","type":"address"}],"name":"allowance","outputs":[{"name":"","type":"uint256"}],"payable":false,"type":"function"},{"inputs":[],"payable":false,"type":"constructor"},{"payable":false,"type":"fallback"},{"anonymous":false,"inputs":[{"indexed":true,"name":"from","type":"address"},{"indexed":true,"name":"to","type":"address"},{"indexed":false,"name":"value","type":"uint256"}],"name":"Transfer","type":"event"}]
```

## Step 2: Set Up a Wallet

To manipulate tokens, you need wallet software on your computer. At the moment, there is no web wallet solution for Ethereum Classic that can manipulate custom tokens.

The simplest solution is to download the [Ethereum Classic Mist builds](https://github.com/ethereumproject/mist/releases). Download and install the version for your platform.

Note that if you already have Ethereum Forked Mist installed, Classic Mist won't work properly. If you intend to use multiple chains, you might want to use [Parity](https://ethcore.io/parity.html), which can keep multiple chains on disk and switch between them.

## Step 3: Sync the Blockchain

To use Ethereum Classic, you need to download the entire history of transactions (the "blockchain") so you can be up to date with everyone else.

Start up your wallet and wait. If using Ethereum Classic Mist, you may need to close and re-start the wallet after it gets to the "Resolved Client Binaries" step for the first time.

Eventually, you should get to some sort of screen about downloading blocks. On OS X Mist, it looks something like this:

![Ethereum Classic Syncing Window](/images/007/chaindownload.png)

You may get put through a wizard on first startup. Make sure to select the "main network". You are unlikely to have a pre-sale file. When prompted for a password, enter something you can remember; it will be used to encrypt your wallet file on disk. There are also tutorials about how to issue your own tokens and how to raise money through crowdsales.

Syncing the blockchain can take anywhere from half an hour to over a day, depending on your Internet connection and your computer's speed. Having an SSD can help make it faster. While syncing, you are not only downloading block data, but also repeating the execution of all the smart contracts on the Ethereum Classic chain, and making sure that you get the same results from those computations as everyone else. In essence, your computer is verifying that the rest of the world is playing by the rules; it will check for and refuse to accept blocks containing any frauds, thefts, or appropriations of tokens that contravene the rules of deployed smart contracts. (However, frauds, thefts, and appropriations of tokens that *obey* the rules of deployed smart contracts will be accepted quite happily.)

During the sync process, it is OK to close Mist. When you come back later and open it again, the sync should resume where it left off.

## Step 4: Create an Account

Once the blockchain is synced, the main Mist UI will open. (You may need to hit "Launch Application".)

If you didn't set up a first account already, set up an account now by clicking on "Add Account". Accounts hold tokens as well as "ether", a special token which is used to pay transaction fees. Accounts are identified by addresses, like this:

```
0x0e7E2B394beE90A6a4BaD5549469168749290034
```

**Remember your password!** It is used to encrypt your account's key, so if you forget it, there is no way to recover the funds stored in the account!

Also, **back up your keys**, which you can find in a folder by going to **Accounts -> Backup -> Accounts** in the Mist menu. You need **both** the file **and** the password to access the account.

## Step 5: Add the Token to Mist

You need to add your token contract to your Mist, so that you can look at and interact with it. There are many tokens on Ethereum Classic, and Mist needs to be told which tokens you care about.

Click on "Contracts" at the top right of the Mist window:

![Contracts interface](/images/007/contracts.png)

Then scroll all the way down to the bottom, look under "Custom Tokens", and click "Watch Token". Paste in the token's address. and the other details should populate automatically:

![Adding the NovakBucks token](/images/007/watch.png)

Then click "OK". The token should now show up under "Custom Tokens", along with your balance. But clicking on it there doesn't really do anything; it just lets you edit the token information.

In fact, there's not much more you can do with the token until you [actually have some](#step-6-get-money).


### Extra: Add the Token as a Contract

You can use the same contract address, along with the ABI for the token contract, to add the token as a custom contract. This lets you do more advanced, low-level things, like querying the total supply of the token, or the balances of other people's addresses.

To do this, go back to the main "Contracts" page, and hit "Watch Contract" under "Custom Contracts". Then fill in the contract details again, but this time including the ABI.

![Adding the NovakBucks contract](/images/007/watchcontract.png)

Then click "OK". The contract should now show up under "Cussom Contracts", and you can click on it to get into the contract's UI. Make sure to scroll down on the contract's page to get to the actual controls that do things. You can read from the contract on the left, and send commands to the contract manually with the interface on the right:

![Interrogating NovakBucks](/images/007/contractui.png)

If you are working with a token that is *not* ERC-20 compliant, you will need to add it as a contract rather than a token, and you will need to use this generic contract UI to manipulate it.

## Step 6: Get Money

Now that you have your token contract attached to your Mist, you need to get some tokens sent to you.

Copy your account address from Mist, and send that to whoever is selling or distributing the tokens, along with any required payment. They should send the tokens to you, and you should be able to observe the resulting increase in your balance by calling `balanceOf` on your address.

You also probably want to buy some Ether Classic (ETC), and keep it in the same account you use to hold your tokens. ETC is used to pay transaction fees when performing smart contract operations that change global state, such as sending tokens to someone else. Smart contract operations take "gas", and you pay ETC to pay for the gas used by the transactions you initiate. Since having tokens you can't send is kind of useless, you'll want some ETC to pay for sending them to someone else. About a dollar's worth should do it.

Whoever sold you the token might also be able to sell you the ETC to move it, but if not the easiest way to get ETC is to buy it with another cryptocurrency through [Shapeshift](https://shapeshift.io/).

Shapeshift is a service that quickly converts between cryptocurrencies. To buy ETC, go to [http://shapeshift.io/](http://shapeshift.io/), and set the "to" currency to "Ether Classic". The "from" currency should be whatever other cryptocurrency you actually have (for example, Bitcoin):

![Shapeshift BTC to ETC](/images/007/shapeshift.png)

Hit "Continue". On the next screen, you can enter your Ethereum Classic address (copied from Mist again), and your return address for the currency you're selling, in case the money needs to be sent back. Once you start the transaction, Shapeshift should give you an address to send to. A few minutes after you send your currency to sell, Shapeshift should send you your ETC.

At this point, you should make sure to record the transaction for your records. Converting between cryptocurrencies could create a taxable capital gain, or tax-deductible capital loss.

## Step 7: Spend your Ill-Gotten Gains

Now that you have a supply of your chosen token, and some ETC with which to pay transaction fees, you probably want to send some to someone else. If your token is ERC-20 compliant, and you added it as a token in Mist, this should be super easy. Just go to "Wallets" on the upper left, and click on your account. If you have any tokens, your token should show up under the account name, together with your balance, and a "Send" button (which will only appear when you hover over the token entry):

![View your tokens](/images/007/viewtokens.png)

To send someone tokens, you need *their* address. Once you have it, click on the "Send" button, and then fill in the details of the transaction: the destination address and the number of tokens to send:

![Send your tokens](/images/007/sendtokens.png)

Then ignore the "Add Data" option and the fee slider, and scroll all the way to the bottom and hit "Send".

Mist will present you with a window to review the details of the transaction. It's a bit cryptic, and in particular it will have a little arrow pointing to the address of the token contract, and not the address you are sending tokens to. This is because you are really telling the contract to transfer ownership of the tokens to someone else. The *actual* "to" address will show up embedded in the cryptic hex data at the bottom, along with the amount (in hex):

![Confirm your transaction](/images/007/confirmsend.png)

When you are happy with the transaction, including the fee you have to pay in Ether, enter your account password and hit "Send". After a few seconds, your transaction will be added to the Ethereum Classic blockchain, and your token balance should go down. If you [added your token as a contract](#extra-add-the-token-as-a-contract), you can enter the address you sent to under `balanceOf` in the contract UI to check the recipient's new balance.

## Step 8: Profit!

This concludes the tutorial on manipulating tokens on the Ethereum Classic blockchain. You should now be able to receive, hold, and send NovakBucks (or any other ERC-20 compliant token).

Finding and purchasing tokens is left as an exercise to the reader.








