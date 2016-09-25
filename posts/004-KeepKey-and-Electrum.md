# KeepKey and Electrum

To store the fabulous wealth that I expect to receive from my blog, I decided to procure a Bitcoin hardware wallet.

While Bitcoin is fast becoming the official currency of the Internet, accepted wherever nerds are sold, the only way to work with it is through a computer intermediary. And since we all know general-purpose computers are treacherous little smartasses who will happily wire all your hard-earned money to nerds in non-extradition countries at the slightest buffer overrun, it makes sense to try and limit what they can get up to without your permission.

Hence the Bitcoin hardware wallet: a piece of digital hardware with a small attack surface that holds your Bitcoin private keys, and which only signs transactions after checking with you first. When using a hardware wallet, you don't have to trust the computer you are using to manipulate your Bitcoins, because the hardware wallet will let you check its work.

## KeepKey

There are a few different hardware wallets on the market right now. Excluding some of the dodgier offerings from Ledger (which don't have screens, which makes them sort of useless, because the whole point is to let you securely check transaction details before signing), there are three real options:

* The [**Trezor**](https://app.purse.io/product/B00R6LSAZI), MSRP **$99**, which is the original hardware wallet, and which has the most momentum behind it.
* The [**Ledger Nano S**](https://app.purse.io/product/B01J66NF46), MSRP **$69.00**, which is the cheapest proper hardware wallet I've found.
* The [**KeepKey**](https://app.purse.io/product/B0143M2A5S), MSRP **$99.00**, which is a modified clone of the open-source Trezor, with less development effort behind it, and which, as you can tell from the title of the post, is the one I went with.

The KeepKey may seem like an odd choice, being neither the cheapest nor, on paper, the best. The KeepKey definitely seems to have the smallest software effort behind it: it doesn't even work as a U2F universal second authentication factor, which the Trezor and the Ledger Nano S both do.

I picked the KeepKey mostly because of its large screen. The Trezor is a tiny thing with a screen, from pictures I've seen, that you could easily cover with the pad of your thumb. The KeepKey screen is much bigger: it's a bit bigger, physically than an Android notification, and can display about as much information. Since you interact with a hardware wallet mostly by reading things off the screen, a big screen is important. Moreover, the big screen on the KeepKey lets it be slightly more secure than the Trezor, in that it can display a substitution cipher to use when entering a recovery seed, so that no information about the seed is leaked to the computer you are using to do the recovery. The downside of the big screen on the KeepKey was that it used to sell for about $239, but since it's now priced at $99, I figured I might as well buy one.

The other deciding factor for the KeepKey was its long cable. The Trezor cable is super short, yo you have to hold the device near your USB port while you use it, while the KeepKey cable is normal phone-cable length.

I unfortunately suspect that the steep discount that the KeepKey units are selling for could mean that the company behind the device is having financial troubles. If that is the case, I could soon be among the world's first owners of unsupported Bitcoin hardware wallets, which is why, as you'll see later, I opted to set up my wallet with [Electrum](#setup-with-electrum) instead of the default KeepKey software.

Also worth noting is that the KeepKey ships from the USA, while the Trezor comes from the Czech Republic and the Ledger Nano S from France. So if you have any state adversaries who can afford their own [shiny security stickers](/images/004/sticker.jpg), you may want to consider where you want your wallet to ship from.

## What's In The Box

![The items in the KeepKey box](/images/004/box.jpg)

The KeepKey comes with:

* One (1) KeepKey unit (in the middle)
* One (1) Quick Start Guide (left)
* One (1) Recovery Sentence card (top left)
* One (1) pouch (top right) to put your Recovery Sentence card in
* One (1) woven micro-USB cable (right)
* One (1) warranty card (bottom), referencing an unspecified Warranty Period, and going to great lengths to disclaim any liability for stolen Bitcoins that you may or may not have tried to protect with the device you bought to stop your Bitcoins getting stolen.

We'll see how that last item holds up in court.

Having it in my hands, there are a couple of weird aspects to the hardware design. The first is that the device only has **one button** (for "confirm"), unlike the Trezor and Ledger Nano S that have two (for "confirm" and "cancel"). If your KeepKey ends up asking you to sign a transaction that you don't like, all you can do is unplug it. The other problem is that the **microUSB port is upside-down**, at least relative to how it is on my phone. I think they did it so that they could mount the connector on the back side of the board or something, but it's weird.

## Setup with Electrum

I decided to set up my KeepKey wallet with Electrum, because I like Electrum and I would feel silly using a combination of [both a Chrome Extension and a Chrome App](https://www.keepkey.com/support/get-started/) to manage my money.

I used Electrum 2.6.4, but you probably want the latest version:

```
sudo pip install electrum --upgrade
```

Setup in Electrum was basically a matter of hitting **File -> New/Restore** in the Electrum 2.6.4 menu and following the wizard, but when I first tried it I got this:

![A list index out of range error.](/images/004/error.png)

[It turns out](https://github.com/spesmilo/electrum/issues/1843) you need not only Electrum, but also the [KeepKey Python module](https://github.com/keepkey/python-keepkey) and maybe some udev rules installed. Borrowing from the [official KeepKey documentation](https://support.keepkey.com/support/solutions/articles/6000090262-electrum-keepkey-integration-ubuntu-linux-):

```
# You might need all this stuff to build the module
sudo apt-get install python-dev python-setuptools cython libusb-1.0-0-dev libudev-dev
sudo pip install python-keepkey

# These are some udev rules to make /dev/keepkey0
sudo wget https://raw.githubusercontent.com/keepkey/chrome-proxy/master/51-KeepKey.rules
sudo mv 51-KeepKey.rules /lib/udev/rules.d/

# Instead of rebooting like KeepKey tells you to, just bang on udev with every command I could find on StackOverflow
sudo service udev restart
sudo udevadm control --reload-rules
sudo udevadm trigger

# You also probably want to unplug and replug your KeepKey
```

After that, I restarted Electrum and the setup process went swimmingly:

![KeepKey setup wizard, showing name, seed word length, PIN, and passphrase options.](/images/004/setup.png)

You can select from a 12-, 18-, or 24-word Recovery Sentence, which you can use to restore your wallet onto a new KeepKey (or just into Electrum itself on a trusted machine), should you ever lose the device. The Electrum default is 24 words, which is of course more secure than 12, but as far as I can tell 12 is still considered "secure", as well as being easier to memorize and short enough to fit on the provided card.

You also have two security options: PIN and passphrase. A PIN just locks the KeepKey: it won't sign any transactions or reveal any receive addresses without the PIN. Every time you want to unlock the device, you enter the PIN on a scrambled keypad, the layout of which is displayed on the screen of the device. This keeps your computer from capturing your PIN, although he computer will know if any two digits are the same.

Setting a passphrase just sets a little flag on the KeepKey to tell it that you have a passphrase. When you set a passphrase, the passphrase is combined with the seed to generate the wallet, so **any passphrase will work** and **every passphrase generates a completely different wallet**. In theory you're supposed to be able to use this to keep multiple wallets, and even secret deniable wallets, on one device, but with Electrum, which remembers receiving addresses and transactions on your computer, I think it will get confused if you try and use multiple KeepKey passphrases in the same Electrum wallet file. So you'd need a wallet file for your secret deniable wallet, which sort of defeats the point.

If you set a passphrase, you don't necessarily have to worry so much about the secrecy of your Recovery Sentence, because both the passphrase *and* the Recovery Sentence are needed to access your coins.

## Security Guarantees

Once you have your KeepKey set up in Electrum, it gives you two new operations with security guarantees. One of them is to verify an outgoing transaction. When you go to send Bitcoin from Electrum, there will be an extra step where you approve the transaction by holding down the button on the KeepKey (or reject it by unplugging the device):

![The KeepKey asking if you want to sign a transaction.](/images/004/outgoing.jpg)

This makes sure you don't send your Bitcoins to an address other than the one you want to send them to.

However, there's a second, equally important new step that you need to follow whenever you want to receive Bitcoins: you need to verify the *receiving* address on the KeepKey, to make sure that you actually own it. After all, Electrum could be lying to you, and handing you an address that it says is yours, but which really belongs to a team of elite Bitcoin-stealing hackers who have compromised your PC. If you tell people to send you Bitcoins at that address, the hackers will get them instead of you. So, before you send any request for Bitcoins, right-click on the receiving address in Electrum and hit **Show on KeepKey**. The address will pop up on the screen with a little QR code:

![The KeepKey telling you you own a receiving address.](/images/004/incoming.jpg)

This is how you verify that the address is actually yours; the KeepKey will only display addresses like this when it knows the private key to the address.

Please note that the KeepKey *can't* guarantee that an address you want to send Bitcoins to actually belongs to the person or organization you think it belongs to. Just because an address shows up on a GDAX page in your browser doesn't *necessarily* mean that coins you send to it will got to GDAX: a stealthy secret agent may have [hacked your screen](https://www.schneier.com/blog/archives/2016/08/hacking_your_co.html) to display the FBI's Bitcoin address instead of GDAX's, in order to steal your Bitcoin fortune as part of an elaborate plan to recruit you to spy on your employer for a foreign power. So don't blow all the cool security you got from your expansive hardware wallet by sending all your coins to just any address your PC gives you. Unless you read the address off of *their* hardware wallet, or got it via another similarly-secure means, it might not really be their address.

As a fun extra, you can also use your KeepKey and Electrum to sign messages. Right-click on a receiving address and pick **Sign/verify message**:

![The KeepKey signing a message.](/images/004/messages.jpg)

Note that the KeepKey doesn't display the address you are going to use to sign the message, so keep in mind that you could be signing the message with a different address than you told Electrum to use, if your computer is compromised. It also doesn't look like Electrum uses the KeepKey to verify messages, so Electrum could be lying and telling you a message you received is properly signed when it isn't, or visa versa.

## Fun Computer Science Corner: Key Derivation Paths

If you go in and look at how the KeepKey actually works, it's basically BIP32 ([hierarchical deterministic wallets](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki)) in a box. It holds some entropy (in the form of the Recovery Sentence seed words), and you can feed in some additional entropy in the form of the passphrase. It then uses the two to derive a master private key. From this master key, it can derive a sequence of child keys (numbered 0, 1, 2, ...), and from each of the children, it can repeat the process and derive more children (themselves numbered 0, 1, 2, ... under their parent), and so on, for a whole tree of potential private keys all derived from the seed words and the passphrase. Normally, someone looking at the keys can tell which keys are children of which other keys, but the child derivation can be "hardened" to produce a *different* sequence of children (numbered 0', 1', 2' ...), for which it is cryptographically hard to go back and work out which children came from which parents. This helps with privacy by making it not so obvious that all the Bitcoin addresses you used were derived from the same wallet seed.

The KeepKey needs to know what sequence of key derivations do do when asked for a Bitcoin address, so your computer sends it a "path", written something like `m/0'/1'/3/5`, telling it what private key to use to sign a transaction, or what public key to spit back. The path given here means:

* Start with the master key ("m"). Note that this part is often omitted from the path.
* Derive child #0, using hardened derivation (`/0'`), so it can't be linked back to the master key.
* Derive child #1 from that key, again using hardened derivation (`/1'`), so it can't be linked back to its parent.
* Derive child #3 from *that* key (`/3`), using ordinary non-hardened derivation (meaning it could potentially be linked back to its parent).
* Finally, derive child #5 from the key we just derived in the last step (`/5`), again using normal non-hardened derivation.

If you installed `python-keepkey` like I told you to, you have a `keepkeyctl` command that you can use to derive keys yourself. Make sure Electrum is closed, and do something like this:

```
keepkeyctl get_address -n "44'/0'/0'/0/0" -c Bitcoin -d
```

After entering your PIN on the numpad on your keyboard (or by complicated mental translation to the number keys if you don't have a numpad), and putting in your passphrase (twice!) you should be greeted with your first Bitcoin receiving address that shows up in Electrum (possibly under "Used", if you already sent coins to it).

The path starts with a `44'` and has the structure it does because it's a [BIP44 path](https://github.com/bitcoin/bips/blob/master/bip-0044.mediawiki). BIP44 basically specifies where in hierarchical deterministic wallet key tree space a wallet ought to derive its various keys. Basically it looks like this:

```
m / 44' / coin_type' / account' / change / address_index
```

For Bitcoin, `coin_type` is 0. The `account` parameter lets you maintain several disconnected accounts based on the same master key. You set `change` to 1 for change addresses and 0 for normal addresses, and `address_index` produces multiple addresses of whatever type within an account, which you can use for different things. We've just asked the KeepKey for the 0th non-change address in the 0th BIP44-compliant Bitcoin account, which is what Electrum also asked for, so we both got the same result. If you use your KeepKey with another BIP44-compliant wallet, like the KeepKey Chrome app/extension, or Multibit, that wallet will generate the same set of keys and addresses as Electrum does, and all your coins will be detected (by enumerating keys at different levels until there are a certain number of addresses in a row with no transaction history).

Interestingly, Electrum [doesn't use the BIP44 structure internally](https://github.com/spesmilo/electrum/issues/1604); Native Electrum wallets are BIP32 hierarchical deterministic wallets, but they use a [simpler derivation path](https://bitcoin.stackexchange.com/questions/36955/what-bip32-derivation-path-does-electrum-use) of just:

```
m / change / address_index
```

Note that none of the derivations are hardened, which is potentially bad for privacy. Here's a useful Excel spreadsheet of [all the different BIP32 paths that different wallet programs use](https://onedrive.live.com/view.aspx?resid=584F122BA17116EE!313&app=Excel).


It seems that when adding Trezor and KeepKey support (which are basically the same code), the Electrum developers decided to use the BIP44 paths that other wallet programs using those devices were using, presumably so people wouldn't plug their hardware wallets into Electrum and wonder why their coins didn't show up. (I spent an embarrassing amount of time trying different key derivation paths in `keepkeyctl` trying to replicate the Electrum addresses before I figured this out.)

## Conclusions

Overall, I am satisfied with my new toy. I enjoy using cryptographic bazookas to swat security flies, and I now no longer have to worry that someone will break into my computer and siphon off any Bitcoins I may or may not happen to own. Plus, the KeepKey looks super cool sitting on my desk with its little KeepKey logo screensaver.

To fill my KeepKey with money, use [1Hep1csebDHPCiS5yqwQPDGeDRZbuQyUmX](bitcoin:1Hep1csebDHPCiS5yqwQPDGeDRZbuQyUmX?amount=0.01). [Here it is on my KeepKey](/images/004/incoming.jpg), although you don't necessarily know that that image hasn't been replaced by a cunning Bitcoin thief. Remember: never give more money to bloggers than you can afford to lose.


