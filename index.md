### Welcome to my Capstone Research Blog.
This is where I will be posting my weekly research related to a post-quantum implementation of Bitcoin. In this blog I will be looking into various topics including post-quantum signatures, effects of quantum computing on Bitcoin, benefits of switching to post-quantum algorithms, how to integrate a backwards compatible post-quantum algorithm into the Bitcoin protocol, and whatever other topics come up.

### Members
Reid Bixler (Researcher)

David Evans (Technical Thesis Advisor)

### Contact
If you have any questions or would like to contact us about my research, feel free to email either me or my technical advisor: {rmb3yz, evans}@virginia.edu

---
## BLISS Bitcoin Scripts
### 11/11/16 - 12/9/16

## Using BLISS to Burn/Lock Bitcoin
As of now, I have come up with two ways to completely lock Bitcoin in a Bitcoin address until BLISS gets integrated.

## Burnt BLISS
The first of which is actually fairly simple, and doesn't even have to rely on BLISS. The security of this method relies on the fact that hashing will not be easily broken with quantum computers. Technically, anybody can create a random address that passes the checks of a valid address, but actually doesn't hash from a ECDSA public key. You could put together a string of 0's, a really interesting vanity address that obviously would be impossible to find (ex. "1YouWontBeAbleToGetThisAddress..."), or hash some made up public key. But for our case, we want to actually be able to use this Bitcoin once the Bitcoin Community (hopefully) switches over to the BLISS protocol. For this we can generate a BLISS private and public key, hash our public key, and use that hash just like any other address in our Locking Script.

The locking script would look as so:

`OP_DUP` `OP_HASH160` `<BlisskeyPubHash>` `OP_EQUALVERIFY` `OP_CHECKSIG`

and our unlocking script would be:

`<BlissSig>` `<BlissPubK>`

Assuming that we only want to transfer our own coin into a locked BLISS address, we simply take that address and send money to it. The only way that this address could ever spend money are one of two ways.

1. The Bitcoin Community alters the `OP_CHECKSIG` opcode to allow BLISS signature checking. The reason why we can't have a `OP_CHECKBLISS` in this case is it would require a fork beforehand, which this method is not relying on. It would be 'fairly' trivial to alter the `OP_CHECKSIG` code to check the length of the signature given to verify if an ECDSA or BLISS signature was used. 

2. ASSUMING that the Public Hash is a 'valid address' (i.e. starting with 1 or 3, indicating P2PKH and P2SH respectively), however unlikely considering a BLISS Public Key Hash would likely not have the same starting bytes as an ECDSA public key hash. Somebody breaks the RIPEMD hash algorithm and produces a hash collision with an ECDSA public key which can be used to validate the transaction.

### Benefits
* This can be done at anytime, even before the Bitcoin Community considers switching to BLISS
* Guaranteed security (assuming no hash-break) against quantum attack
* Usable once BLISS becomes integrated and requires 'minimal' changes in the protocol

### Drawbacks
* HIGH POSSIBILITY OF LOSING YOUR BITCOIN. Obviously this is a huge risk to put all of your faith into the assumption that the Bitcoin Community MAY end up integrating this in this exact way. Even if the Bitcoin Community decides to switch to BLISS, but does it another way, then your Bitcoin would still be lost!
* Requires altering an opcode, likely increasing the chance of security flaws. Adding an additional opcode like `OP_CHECKBLISS` would be great, but does not allow users of Bitcoin to protect their Bitcoin before the fact.
* If the Bitcoin Community chooses not to do this, then you are just adding an additional transaction on top of the blockchain that is totally useless and takes up unnecessary space.

### Considerations
* This is a very drastic measure for 'protecting' your Bitcoin with BLISS, and there are entirely more reasonable methods to protect your Bitcoin, most notably, just putting your Bitcoin into an address that has not been used before AND not spending any of those coins from that address. Assuming that hashing isn't broken, your ECDSA public key does not need to be published to the blockchain until you want to spend coins from an address, therefore protecting your Bitcoin from quantum attack until the Bitcoin Community implements a more reasonable protection. The only reason that you would prefer Burnt BLISS over this is if you think that you REALLY want to guarantee that a quantum attack won't steal your Bitcoin AND you want to leave Bitcoin in this address even after this method became implemented (which is highly unlikely). 


## Scripted Burnt BLISS
While I have done a lot of research on the Bitcoin protocol, I cannot guarantee that I know everything that is necessary to verify that this can be a valid method. The assumption that I am going to be making for this is that a RedeemScript can have a currently Reserved word/opcode without making it an invalid transaction. To explain why this assumption is necessary, we need to look at the Pay to Script Hash (P2SH). P2SH was added late into the development of Bitcoin to allow the recipients to create their own ways to ensure the security of the Bitcoin that they were receiving. The cool thing about P2SH is that the requirements to unlock funds could be a number of signatures, a password, or even the answer to a complicated math problem. In our case, the necessary 'key' will be our BLISS Public Key. However, the main component of our method will be the fact that our script can contain a currently invalid opcode. Technically, as of right now, if a node sees that a transaction is invalid then they will toss it aside and ignore it. However, if the invalid opcode is within a hash, then technically the transaction is 'valid' and becomes truly valid once the opcode becomes valid itself. This method is similar to our Burnt BLISS, but has a very interesting benefit of not requiring the alteration of `OP_CHECKSIG`. 

Let us consider our Redeem Script to be:

`<BlissPubK>` `OP_EQUALVERIFY` `OP_RESERVED1`

Our Locking Script will then be:

`OP_HASH160` `<20-byte hash of Redeem Script>` OP_EQUAL

and our Unlocking Script will be:

`<BlissSig>` `<Redeem Script>`

Assuming once again that `OP_RESERVED1` is technically valid as long as it is hashed, then this Bitcoin will be locked into the script address given by the specific `<BlissPubK>` and cannot be unlocked until `OP_RESERVED1` becomes a valid opcode. This is similar to our previous Burnt BLISS but has the addition of not requiring the replacement of code in `OP_CHECKSIG`, instead we will rely on the Bitcoin Community to put our assumed `OP_CHECKBLISS` into the `OP_RESERVED1`. Once `OP_CHECKBLISS` is implemented, our previously locked address becomes spendable with our BLISS Signature. This is where my knowledge of the blockchain comes into question. I have looked as in depth as I could, but when I try to determine if having previously invalid opcodes in old blocks all I could find was this on the Bitcoin Wiki "you must ensure that adding this new opcode doesn't cause transactions that were once invalid to be valid." I am unsure if this means if a P2SH Redeem Script could or could not have an invalid opcode in it. From what I grasp, it means that an unaccepted and invalid transaction from before cannot now become valid transactions. Technically, this method is a completely valid transaction, since the Redeem Script is never publicly released beforehand.

### Benefits
* Similarly to Burnt BLISS, this can be done at anytime, even before the Bitcoin Community considers switching to BLISS
* Guaranteed security (assuming no hash-break) against quantum attack
* Usable once BLISS becomes integrated
* Doesn't require altering current opcodes

### Drawbacks 
* HIGH POSSIBILITY OF LOSING YOUR BITCOIN. Obviously this is a huge risk to put all of your faith into the assumption that the Bitcoin Community MAY end up integrating this in this exact way. Even if the Bitcoin Community decides to switch to BLISS, but does it another way, then your Bitcoin would still be lost!
* Requires taking up a reserved opcode. Adding an additional opcode like `OP_CHECKBLISS` is extremely useful, but seeing as reserved opcodes are rare, it may be risky to give one up for the BLISS protocol
* If the Bitcoin Community chooses not to do this, then you are just adding an additional transaction on top of the blockchain that is totally useless and takes up unnecessary space.

### Considerations
* Similarly to the other burnt coin, this is a very drastic measure for 'protecting' your Bitcoin with BLISS. Admittedly, compared to Burnt BLISS, this method is 'more likely' to be accepted because there would not be alterations to current opcodes. Depending on if the Bitcoin Community wants more versatility with an opcode by changing `OP_CHECKSIG` or adding a new opcode over a reserved slot with `OP_CHECKBLISS` will determine what path is best.


## How to Slowly Integrate BLISS into Bitcoin
Technically, we will not be able to be able to 100% protect our Bitcoin from quantum attack unless we never spend from our final output address (by this I mean you can never have leftover Bitcoin in an address after a transaction, else you risk having it stolen from quantum attack) or use the previously mentioned burn tactics. Obviously, a lot of people still want to be able to constantly use their Bitcoin throughout, and nobody really would like to take the risk of burning their Bitcoin (or at least a majority of it). At the moment, I have considered a method that allows for the Bitcoin community to slowly integrate BLISS into the protocol, admittedly at the cost of speed and immediate security.

## STEP 1) Message BLISS
A simple way to 'protect' our Bitcoin from quantum attack would be to tell the Bitcoin not to accept transactions from an address without verifying a BLISS key. To do this, we can use the 40 additional bytes that can be tacked onto a transaction with `OP_RETURN`.

Our Locking Script will look like any ordinary P2PKH with `OP_RETURN` on top:

`OP_DUP` `OP_HASH160` `<PubKHash>` `OP_EQUALVERIFY` `OP_CHECKSIG` `OP_RETURN` `<Message>`

Our Unlocking Script will be the normal P2PKH script:

`<Sig>` `<PubK>`

Our `<Message>` will contain something along the lines of "Please don't accept this transaction without verifying my BLISS public key available at <website>". This is obviously unlikely to stop many people from accepting the transaction, but in the worst case if your Bitcoin were stolen from quantum attack, then you could 'prove' that the transaction should be invalidated. However, given Bitcoin's previous stagnancy on reverting transactions, this will likely provide little protection.

## STEP 2) Forked BLISS
At this point, we can wait until the Bitcoin Community decides to do a soft or hard fork with BLISS included. We cannot assume in this case how they will do it, but the previous two possibilities of burning can be used for this new BLISS protection. In this case we have 2 options, replacing a reserved opcode for `OP_CHECKBLISS` or altering `OP_CHECKSIG` to allow for BLISS signatures. Once this is added, we have the opportunity to replace any P2PKH or P2SH with BLISS signatures and BLISS public keys. By waiting until a fork happens, we take away any risk that your Bitcoin ends up getting burnt through assumptions in changes of the protocol.

### Benefits
* Can still spend Bitcoin before forking to BLISS
* Guaranteed security once Step 2 is achieved
* Very low possibility of losing Bitcoin. Cannot lose Bitcoin through burnt addresses and can only lose it assuming a quantum attack happens before Step 2.
* Doesn't require altering current opcodes

### Drawbacks 
* This requires the Bitcoin Community to eventually fork and make the BLISS changes necessary. Until Step 2, your Bitcoin are not guaranteed to be secure with a simple message.
* Taking risk that your address could be the reason why Bitcoin shifts to BLISS. By this, I mean that if and when a quantum computer user decides to attack a Bitcoin address, there will likely be some amount of Bitcoins stolen until the blockchain forks to the BLISS protocol. If your address has enough Bitcoin to warrant being attacked by a quantum computer, then you are more likely to lose your Bitcoin before the necessary changes take effect.
* Still requires changes to current code or a reserved opcode. Obviously it won't be possible to allow BLISS without doing some alteration to the Bitcoin protocol, and it is up to the community to decide which choice is best.

### Considerations
* One big consideration for this method is whether or not the Bitcoin Community should decide to phase out ECDSA entirely. The obvious drawback of phasing out ECDSA is that anybody with 'old' Bitcoin will no longer be able to use them. Even setting some period of time to switch over will still leave some people with Bitcoin stuck in addresses no longer accepted. Essentially, this would be a collective decision by the community to burn a portion of the entire Bitcoin into unspendable addresses. However, if ECDSA is used alongside BLISS then we need to start to consider whether or not they will be used together, alternatively, or through some other means. For example, should 'new' standard Locking and Unlocking scripts assume that both ECDSA and BLISS are going to be included, or the default be BLISS with ECDSA still being available? By maintaining ECDSA, no address will ever be locked out of using their Bitcoin, but they do end up taking the risk of having their Bitcoin stolen in the case of a quantum attack.
* Also, we need to consider what was brought up in the previous post due to the size of BLISS keys and signatures. The limits on transactions are drastically decreased once BLISS is added in, and the blockchain will only grow even larger. It makes sense to increase the standard block size from 1MB to something larger, but that is another major problem being currently debated today. Suffice to say, even if BLISS secures Bitcoin in technicality, it poses other social and economic problems within Bitcoin.

## Final Thoughts
Protecting Bitcoin from potential attacks of quantum computers may seem like overkill, but when the foundation and value of Bitcoin is reliant on the security of its algorithms, we need to guarantee that those algorithms get replaced if proven to be insecure. Looking through alternative signature schemes, we decided to go with the Bimodal Lattice Signature Scheme (BLISS) as our intended post-quantum algorithm replacement for ECDSA. We have analyzed the necessary changes required from Bitcoin, the effects that it will have on the blockchain, and different steps to integrate BLISS into Bitcoin. Implementing these changes will definitely be considered drastic and have lasting effects on Bitcoin and the blockchain, which is why so much effort has been put into analyzing such effects. This concludes my first semester of research in Post-quantum Cryptocurrencies. Next semester I will be looking into implementing a local prototype of BLISS integrated Bitcoin using btcd (https://github.com/btcsuite/btcd) and verifying the security, speed, and size requirements. Also I will be looking at possible alternative means of generating Gaussian curves to prevent side-channel attacks on the cache as brought up in the previously mentioned paper (http://www.leongb.nl/wp-content/uploads/2016/03/Thesis.pdf). Hopefully by the end of next semester, the prototype will be up to the standard that it could technically be fully implemented into the main blockchain.

## Other Sources
https://en.bitcoin.it/wiki/Proof_of_burn
https://en.bitcoin.it/wiki/Softfork
https://en.bitcoin.it/wiki/OP_RETURN
https://en.bitcoin.it/wiki/Script
https://bitcoin.org/en/developer-guide
http://www.siliconian.com/blog/16-bitcoin-blockchain/26-deconstructing-bitcoin-transaction-part-2-op-return
http://www.soroushjp.com/2014/12/20/bitcoin-multisig-the-hard-way-understanding-raw-multisignature-bitcoin-transactions/
http://bitcoin.stackexchange.com/questions/9678/what-is-script-hash-address-exactly-and-how-does-it-work
http://chimera.labs.oreilly.com/books/1234000001802/ch05.html


---
## BLISS Bitcoin Protocol Specifications
### 10/28/16 - 11/11/16

## Required Changes
As stated in the last post, we are going to have to end up adding an opcode to Script in order to properly check BLISS signatures. This opcode will be known as `OP_CHECKBLISS` which will take a public key from BLISS and a signature from BLISS to verify if they match together. The problem now is deciding how we are going to protect our Bitcoin **before** the Bitcoin community decides to switch over to use the BLISS protocol. At the moment, we have 2 options, both of which have their pros and cons.

First, we can 'burn' our Bitcoin by running a Locking Script with BLISS integration on some previously non-BLISS protected Bitcoin. Assuming that a node and the blockchain are willing to accept this transaction, your Bitcoin will now be completely locked into that new account without possibility of removal _UNTIL_ the Bitcoin community does a fork to allow for using BLISS Unlocking and Locking Scripts. The benefits of running this is a complete guarantee that your Bitcoin cannot be stolen by a quantum computer, regardless of the state of the current blockchain's integration with BLISS. The obvious negative of this is your Bitcoin is completely unusable until the fork to the new protocol, hence 'burning' your Bitcoin. In the absolute worst case scenario, the Bitcoin community never forks to the new protocol leaving your Bitcoin stuck with an unlockable output because of your attempted protection against quantum attack.

Second, we can keep the original protocol, but add in our BLISS integration as a part of the message included in transactions. For example, we generate a BLISS private key and use that private key to create our ECDSA private key, which is then used to create a new public address to send Bitcoin to. This address is sent money using the old P2PKH Locking Script, but we also end up adding our BLISS public address into the message, but don't integrate it into the Locking Script. We could say with that message "Please don't validate transactions from this address without verifying the BLISS signature". When the Bitcoin community forks to the new Locking and Unlocking scripts, all of these 'new' transactions already contain the BLISS integration, which means that they can be quickly ported over to the new protocol without required switching. The benefits of this form is that your Bitcoin will be still completely spendable throughout the entire process, but has the added benefit of the BLISS protection when the community switches over. The negatives of this are that a message saying "Please don't steal this" wouldn't particularly stop somebody intending to steal Bitcoin with a quantum computer, and your Bitcoin is not 'technically' safe from a quantum attack until the Bitcoin community integrates with BLISS.


## Consequences of BLISS Integration
Now that we understand what are the possibilities for switching over to BLISS, what are going to be the consequences? With new Unlocking and Locking scripts, we introduce yet another form of possible transactions in Bitcoin. These transactions will be only useful assuming a BLISS private key can be generated and used by the users, which will require a change of software for many Bitcoin clients, assuming they desire the protection from quantum attack. In terms of previous addresses and previous transactions, we cannot end up blocking old Unlocking and Locking scripts, because then you would be 'deleting' old Bitcoin by leaving them in addresses to never be used again. This means that we **must** be able to use the old Locking and Unlocking scripts, even if they are technically not secure from quantum attack if the transaction does not use change addresses. It will be up to the individual owners of Bitcoin to switch over to the new protocol, with the risk of not switching being the possibility of having your Bitcoin stolen.

In terms of actual requirements set by Bitcoin for a **standard transaction**, are as follows:

* The transaction must be smaller than 100,000 bytes. That’s around 200 times larger than a typical single-input, single-output P2PKH transaction.

* Each of the transaction’s signature scripts must be smaller than 1,650 bytes. That’s large enough to allow 15-of-15 multisig transactions in P2SH using compressed public keys.

* The transaction’s signature script must only push data to the script evaluation stack. It cannot push new opcodes, with the exception of opcodes which solely push data to the stack.

Assuming that we are using BLISS-IV (see previous blog post for BLISS document containing protocol specifications), we are going to have a signature size of 6.5KB, a private key size of 3KB, and a public key size of 7KB. If you do not already know, 1KB is approximately 1000 bytes. This means that adding in any of the BLISS keys or signatures for a single transaction between two parties is completely doable, albeit limiting on the current protocol. As of now, the average single input to single output transaction will require 500 bytes or 0.5KB. By adding on the public key and signature from BLISS, we will be requiring approximately 14KB per transaction (assuming single to single), this would include the 500 bytes from normal ECDSA, 6.5KB for the signature, and 7KB for the public key. At the current standard, this means that we will only be able to fit around 7 inputs and outputs to a transaction, assuming they are all using the BLISS protocol. This is a pretty huge jump in terms of size requirements for Bitcoin (~200 to ~7 is a factor of nearly 30). What about speed requirements? The only thing that we will be verifying is the signature of ECDSA and the signature of BLISS. The act of signing in BLISS-IV requires about 0.375ms while a ECDSA-256 requires about 0.106ms. Luckily signing is not actually required by a node for a list of transactions, signing is only done by the individual users. Verification for BLISS-IV requires 0.032ms while a ECDSA-256 requires about 0.384ms. This means that we can verify about 31,000 transactions in a second with BLISS-IV at the same time that we could verify about 2,500 in a second transactions with ECDSA-256. This is extremely useful when integrating into the Bitcoin protocol because then we can know that speed will not be an issue created from this integration. However, notably the size will likely become an issue, requiring possibly an increase later in the future, especially considering that there are arguments going on currently about the maximum blocksize of 1MB being too small (only allowing for about 3-4 transactions per second).

Obviously we are going to have to look more at the consequences of real-time use of BLISS once integrated into Bitcoin, which is why I will eventually be creating a local implementation for testing purposes and setting benchmarks. It's also entirely possible that an even better post-quantum signature scheme may arrive in the future with more security, less size requirements, and faster signing times. However, at the moment, BLISS is what we are looking towards for post-quantum security for Bitcoin. 

## Other Sources
https://bitcoin.org/en/developer-guide#non-standard-transactions
https://en.bitcoin.it/wiki/Protocol_documentation
https://en.bitcoin.it/wiki/Block_size_limit_controversy

---
## Bitcoin Locking and Unlocking Script Alterations
### 10/14/16 - 10/28/16

## Current Structure
The current Bitcoin protocol is implemented in GoLang but it also has it's own Scripting language. This Scripting language is a number of Opcodes which are *NOT* Turing complete because there is no need for the language to be more complex than it needs to be, but also because a non-Turing complete language won't have loops and therefore won't have the case of the Halting Problem. If you do not know what the Halting Problem is, it is simply finding out whether or not a program will finish or not, and the introduction of loops can make a program go on infinitely. If the Bitcoin Scripting language doesn't have loops, then there is a guarantee that no program in the Bitcoin software will get stuck on a script that loops on forever. In the past there were a few Opcodes from the Bitcoin Scripting language that were removed for security reasons. For example, there used to be a simple `OP_RETURN` which had the potential to break the entire structure of Bitcoin because by passing it along with a 1 one could validate any transaction without the public key or signature (Obviously a security flaw in software intended to hold monetary value).

## The Current Scripting Language
How does the current Bitcoin Scripting Language (literally known as `Script`) work? It is a simple stack-based protocol that moves from left to right. What this means is that a user will give a Script as a list of instructions which will move items on and off of a stack. In terms of actual uses, if an Unlocking and Locking script are given, then a final stack with True on top (assuming a correct implementation of scripts) will grant access to transferred Bitcoin. The Script will contain, as stated before, opcodes, also known as `Words` which have the ability to manipulate the current stack. The words themselves look like `OP_NOP`, `OP_PUSHDATA1`, etc. but have numbers (opcodes) associated with them such as `97`, `76`, etc. respectively. There are some very complex checks of cryptography implemented into some of these words including public key verification with signatures (`OP_CHECKSIG`), RIPEMD-160 (`OP_RIPEMD160) and SHA256 (`OP_SHA256`) hashing, and verifying multiple public keys with multiple signatures (`OP_CHECKMULTISIG`). These are the core of the scripting language and are mostly what define the 5 sets of valid Locking and Unlocking Scripts.

What are these Locking and Unlocking Scripts anyway? These are the scripts, when in combination, that validate transactions on the Bitcoin blockchain. The Locking Script is a combination of hashing, the public Bitcoin address, and checks for the signature (Specifically, `OP_DUP OP_HASH160 <pubKeyHash> OP_EQUALVERIFY OP_CHECKSIG` in one case). This Locking Script is often called `scriptPubKey`. For each Locking Script, there must be a valid Unlocking script that combines the signature and public key of the sending party (For the previous example, `<sig> <pubKey>` are passed). This Unlocking Script is often called `scriptSig`. With these two scripts, transactions are able to be validated on the stack by verifying the signature and public key hash, and without those the transaction won't be validated. Specifically, the Locking script is a type of "lock" placed on the outputs of a transaction that won't allow the output to be spent without the verification of the "key" (i.e. Unlocking Script). Every Bitcoin client intending to verify all the transactions must take the given Locking Script on the inputs run the Unlocking Script (given with the input itself) and execute the scripts together. However, there is not just one way to run Locking and Unlocking Scripts, in fact there are multiple. The one referenced previously is known as a Pay-to-Public-Key-Hash (P2PKH) because it is paying to the hash of a public key. This is predominantly used in the blockchain, but there do exist other payment methods like Pay-to-Public-Key (P2PK) and Multi-Signature.

One large thing of note is that P2PK will become completely invalidated if quantum computers become available. The reasoning behind this is that P2PK assumes that the outputs are locked with the Public Key (Like so, `<Public Key A> OP_CHECKSIG`) which in a "quantum world" would be 'easy' to break and find the private key for. If somebody wants to steal money from this address they would have to simply give the signature from the private key (Like so, `<Signature from Private Key A>`) which would grant access to the stored Bitcoin. At least in the P2PKH instance, the Locking script runs on top of the Bitcoin address (a hashed version of the public key), which would make it 'hard' to find the public key, and therefore 'hard' to break and find the private key. Therefore, up until the Unlocking Script is run (and the Public Key is broadcasted by the intended recipient), any Bitcoin outputs will be safe from quantum-attack *ASSUMING* that the Public Key has not yet been broadcasted.

## BLISS Integration

As talked about in our previous post, BLISS will be our desired post-quantum signature for the Bitcoin protocol. The big question is how we can ensure that this signature is being verified alongside the original signature. We want to be able to create Scripts that simultaneously verify the ECDSA signature as well as the BLISS signature, and guarantee a failure if either fails (specifically the case when ECDSA succeeds and BLISS fails, which would be a sign of a quantum-attack). The largest problem right now is simply the fact that the `Script` does not yet contain Opcodes for verifying a BLISS signature. Until an Opcode for this verification is available, there can be no unlocking or locking scripts that can ensure security against quantum computers. However, in this case, I am going to move forward assuming that there exists a word known as `OP_CHECKBLISS` which when given a stack of `<PUB KEY BLISS> <SIG BLISS>` will return a result of `TRUE` on the stack (essentially verifying that the public key shown is generated from the same private key of the signature).

We can now go about determining what would be the best course of action for creating a valid Unlocking and Locking Script with the given Opcode. Simply, we will be having our Unlocking script have both our Public Keys and Signatures from ECDSA and BLISS, and our Locking Script will have our verifications first of the ECDSA then of the BLISS, and finally comparing those two outputs to determine if they are both true (a simple AND instruction). With this in place, we can ensure that any transaction containing Locked with these Scripts will only be unlocked with a valid and quantum-secure BLISS signature.

## Interesting Things/Previous Research

It turns out that earlier this year a man by the name of Leon Groot Bruinderink released a paper on possible side-channel attacks on an implementation of post-quantum Bitcoin that uses the BLISS protocol. That paper can be found here (http://www.leongb.nl/wp-content/uploads/2016/03/Thesis.pdf). This is an extremely interesting look at the possible types of attacks on our potential implementation of BLISS into Bitcoin, as well as the possible ways to prevent such attacks.

## Other Sources
- http://chimera.labs.oreilly.com/books/1234000001802/ch05.html#p2pkh
- https://en.bitcoin.it/wiki/Script

---

## Post-Quantum Implementation into Bitcoin Analysis
### 9/30/16 - 10/14/16

## Risks of Non-implementation
As stated previously, the ECDSA algorithm is no longer “hard” using a modification of Shor’s algorithm. This means that any service currently using the ECDSA algorithm has the potential to be hacked or broken by a quantum computer. In terms of the Bitcoin protocol, what would be the consequences in the breaking of the ECDSA algorithm? The Bitcoin protocol uses ECDSA to generate public and private key pairs which are used to sign transactions and verifying those same transactions. The private key is specifically used to sign any transaction that you choose to put onto the block chain. This means that you can create a transaction designated that you will send 1 Bitcoin to person X, followed by signing a signature of that transaction with your private key Pr. The Bitcoin network in turn sees that transaction and verifies your given signature with your publically available public key Pu. As long as the signature can be verified with the public key, then the transaction will most likely go through and onto the block chain. The security of ECDSA relies on the fact that it is provably “hard” to calculate the private key Pr from the available public key P¬u. However, a functioning quantum computer would be able to relatively easily derive the private key from any given public key. This control of a private key does not give total control over the user’s Bitcoins, but it does pose risks for some users. Specifically, this poses a risk to users who choose to only stick with one private key, as in users that use the same Bitcoin address repeatedly. However, the Bitcoin protocol protects your Bitcoin address with the usage of multiple hash functions used on the public key to generate the user’s Bitcoin address. The protocol is as follows:

What this generally means for Bitcoin users is that your address will not be the source of your leaked private key (assuming a functioning quantum computer). So all that means is that we don’t have to give away our public key, right? Well, the problem is that nobody can verify your transaction signatures without your publically broadcasted public key. This means that until you broadcast your public key, your address is perfectly safe from quantum attacks (assuming that the hashing functions SHA256 and RIPEMD-160 remain quantum resistant, but that would lead to even larger problems). However, once your public key is broadcast to verify a transaction, any leftover Bitcoin in that address is at risk of being attacked and robbed by a quantum computer. Once your transaction and public key are available, a maligned entity with a quantum computer can brute force your private key, impersonate you with it, create a transaction transferring any leftover Bitcoin to their own address, and get off scot-free (assuming the Bitcoin community would be unwilling to reverse the transaction). However, there is yet another problem present with the introduction of a quantum computer. The Bitcoin community is built up from a multitude of nodes throughout the world who receive and send Bitcoin transactions to the Bitcoin miners that eventually place your transactions in the block chain. If one of those nodes that you attempt to send to has a quantum computer, they could receive your intended transaction, forge your private key, create a new transaction changing the intended address, and send it on its way to the mining pools. So what is a solution to this? Simply make every Bitcoin address a one-time use address. While this principle is carried out quite often in the Bitcoin block chain, there are still many cases of businesses, organizations, and the like who reuse a Bitcoin address for ease of use. In fact, there exist many Bitcoin addresses on the block chain that are holding hundreds of Bitcoin and have already publically broadcast their public key. However, this solution assumes that every Bitcoin node and Bitcoin mining pool are trustworthy and do not have quantum computers. If any receiving node or mining pool is malicious and owns a functioning quantum computer, they can spend your Bitcoin even before your transaction becomes technically available to the public! While the majority of Bitcoin may be safe, any security risk to the protocol must be mitigated as soon as possible, assuming a future of working quantum computers.

## The Protocol
We have discussed previously about the possible post-quantum algorithms that can be applied for use in the Bitcoin protocol, and have come down to using the BLISS protocol as our quantum-resistant algorithm. However, the problem that we are presented with is integrating this into the current protocol while still maintaining the previous functionality, also known as backwards compatibility. If we required a hard fork from the entire Bitcoin community, we would see a split similar to that of Ethereum earlier this past summer. Many users would not consider the threat of quantum computers large enough to switch over to using a new protocol entirely, while some would like to ensure the overall security before the likelihood of an attack. Therefore, we pose the notion that we derive the private ECDSA key from our private quantum key, and requiring the verification of both the ECDSA signature and quantum signature. Early adopters of this protocol would be required to put out a message to the block chain stating something along the lines of “do not accept any transactions from this address without the verification of my quantum signature”. Eventually, once enough traction is gained, or when the first attack on ECDSA is discovered, Bitcoin miners will only add transactions with the quantum signatures in addition to the ECDSA signatures. In the worst case, nobody will use a quantum resistant Bitcoin protocol until an attack is carried out. However, this is a huge risk considering once again why Ethereum split earlier (http://www.coindesk.com/ethereum-classic-explained-blockchain/). If Bitcoin users only wait until they are attacked, they run the risk of losing a majority of the value in Bitcoin from a major split between users. By implementing an entirely backwards-compatible AND quantum resistant protocol into Bitcoin, we can slowly integrate quantum resistance into Bitcoin while not alienating users who do not yet wish to join. The problems that we see with integrating this is maintaining the functionality as well as minimizing the weight of this new protocol. Noticeably many new quantum-resistant signatures require massive key sizes as well as significantly longer signing and verification times. Seeing as the Bitcoin community is already having problems with scaling up transaction volume, adding additional unnecessary weight from massive keys and long waiting times would be infeasible. This is where the BLISS protocol comes into play. As far as my research has come to, the BLISS protocol is the most viable quantum-resistant protocol because of the significantly smaller key sizes and signing/verification times in comparison to other quantum-resistant signature algorithms. Similar to a previous table here is a more in depth look at the requirements behind the BLISS protocol:

A good comparison between different quantum-resistant signature schemes and current schemes was produced which at least shows the viability of BLISS:

As can be seen from the chart above, BLISS is the best performing protocol of all the tested algorithms, although it does obviously worse in terms of key size compared to current protocols. In terms of creating an ECDSA private key from a BLISS private key, we would have to hash the BLISS private key to produce our required ECDSA private key. While the hashing obviously has its benefits from the quantum-resistance, it is required to scale down the larger size of the BLISS key. The actual creation of a private key is creating 2 matrices that satisfy the BLISS protocol such that it follows the following scheme:

Once our two keys A and S are generated, we can hash them down with a chosen hashing scheme (SHA-256, BLAKE-256, HAVAL etc.) that guarantees an ECDSA sized private key (256 bits). I intend to look into creating an example of this for a later post. In the best case, we can ensure the complete security of Bitcoin, while still maintaining the current functionality of the protocol.

## Other Sources
- http://www.bitcoinnotbombs.com/bitcoin-vs-the-nsas-quantum-computer/
- https://bitcoinmagazine.com/articles/bitcoin-is-not-quantum-safe-and-how-we-can-fix-1375242150
- http://www.smithandcrown.com/8655/
- http://www.nicolascourtois.com/bitcoin/thesis_Di_Wang.pdf#page=27
- http://csrc.nist.gov/groups/ST/post-quantum-2015/presentations/session9-oneill-maire.pdf
- http://www.etsi.org/images/files/ETSIWhitePapers/QuantumSafeWhitepaper.pdf

---

## Introduction to Quantum Signatures
### 9/17/16 - 9/30/16

## Background
Digital signatures are used pretty much everywhere on the internet, specifically for Authentication, Integrity, and Non-repudiation. A lot of the security of the net relies on the fact that these digital signatures are truly secure and won’t be ‘easily’ broken. With the coming potential of quantum computers, there is a growing need for quantum secure algorithms and signatures alike. Many quantum-resistant encryption algorithms have been proposed, but most have not been designed to support signatures. In this post, I am going to look into the applications of some quantum signatures and their usefulness in current technologies.

## Lattice Signatures
I'm first going to look at Lattice-Based cryptography, based off of the fact that it is often the most cited and used form of post-quantum cryptography. In future posts, we'll get more into how lattice cryptography works, but for now we will describe some current signature schemes.

### Ring-LWE Signatures
(http://eprint.iacr.org/2011/537.pdf)

Ring Learning with Error is based on the arithmetic of polynomials with coefficients from a finite field. In 2011, Lyubashevsky came up with the application of the Ring-LWE structure for signatures based off of the worst-case hardness of the Shortest Independent Vectors Problem (SIVP) in general lattices. In his paper, he looks at the Small Integer Solution (SIS) problem and its application to a signature scheme based off of its provable hardness. The requirements behind the signature are a secret key of m x k matrix of S random integers and a signer must pick an m-dimensional vector y from a distribution D which then is used to compute c such that ( c = H(Ay,m) ) which is then used to compute z such that ( z = Sc+y ). The signature that the signer will output is (z,c), put there is a probability that a signature will not be outputted, at which point the signer will run the algorithm until a signature is outputted. The signature length and key size can change based off of the signers choosing, but there are certain security implications if those values are too small. Later in the paper, applications of SIS and LWE define a Ring Variant of a SIS signature which is provably improved from the previous two signature schemes. Specifically, the scheme defines a secret key s1,…, sγ where every coefficient of si is chosen uniformly and independently from a predefined range. A public key is created (a1,…, aγ, t) where each ai is uniformly random and t is defined as the sum of each multiple of ai and si. A message is signed, which can be verified easily with the public key. This scheme is based on the provable hardness of the ring variant of the SIS problem. This final ring variant is the Ring-LWE Signature provided by Lyubashevsky.

### GGH Signatures
(http://eprint.iacr.org/1996/016.ps)

The Goldreich-Goldwasser-Halevi signature scheme was published in 1997 and is based on the Closest Vector Problem (CVP) which is also a lattice based problem. Signers show their message using a lattice, which the verifier then verifies on the different lattice point sufficiently close to the previously designated message point of the same lattice. The GGH signature scheme is more based on an encryption algorithm but the application to signatures is perfectly viable, while difficult.

### NTRUSign
(http://grouper.ieee.org/groups/1363/WorkingGroup/presentations/NTRUSign-2005-08.ppt)

NTRUSign is based on the GGH signature scheme which was put into publication in 2003, but was revised with parameter recommendations in 2005. This signature scheme uses 2 short polynomials in a ring which are then used to place the intended message in a 2N-dimensional space. One of the problems presented with NTRUSign is the fact that a transcript of signatures can leak information about the private key. After the creators reformed their parameter recommendations in 2005, they suggested that 20^30 signatures were needed to possibly find the private key, however a recent 2012 analysis of NTRUSign proved that they only needed about a couple thousand signatures to provably find the private key.

### BLISS
(http://eprint.iacr.org/2013/383.pdf)

Finally, and most importantly, in 2013 the BLISS protocol was introduced as a solution to all the previously broken signature schemes introduced. BLISS is very similar to Ring-LWE in structure, but has some notable changes which allow for it to (so far) not give away the secret key over consistent usage. The signature and verification algorithm are shown below:
![](http://imgur.com/rVMwR7l)

However, there were optimizations introduced later in the publication, specifically defining the BLISS signature scheme:
![](http://imgur.com/Ld96OYB)

Using the BLISS signature scheme for future signatures look to prove quite useful, however compared to current schemes there are noticeable drawbacks:
![](http://imgur.com/34MHKtO)

The BLISS scheme noticeably takes more space than RSA or ECDSA, but this is the most applicable and secure implementation of lattice-based cryptography so far. 

## Multivariate Signatures
This type of cryptography is based off of multivariate polynomials over a finite field. One of the ‘benefits’ of multivariate cryptography is the fact that it is extremely useful in the application of signatures, but all encryption schemes using multivariate cryptography have failed. In this post, I intend on going strictly into the viability of Lattice Signatures, but I am adding the types of signature schemes I may look at in the future.

### CyclicRainbow
(https://eprint.iacr.org/2010/424.pdf)

### MI-T-HFE
(https://eprint.iacr.org/2015/890.pdf)

### GUI
(http://csrc.nist.gov/groups/ST/post-quantum-2015/papers/session1-ding-paper.pdf)

## Hash-Based Signatures
Hash signatures have been around since around 1970 ever since Ralph Merkle came up with the idea of the Merkle Trees as an alternative to RSA and DSA. The interesting thing about hash signatures are that the use of hashing is already widely used, but has been proven to be quantum resistant unlike some of the other current encryption methods. This means that using hash-based signatures could prove easier to implement into future technologies simply because people are more used to them. Similarly to the previous section, I will go into this more on future posts.

### Merkle
(https://www.emsec.rub.de/media/crypto/attachments/files/2011/04/becker_1.pdf)

### Lamport
(http://csrc.nist.gov/groups/ST/hash/documents/preneel_nist_v2.pdf)

## Code-Based Signatures
Code-Based encryption schemes are based on the general hardness of decoding a general linear code. McEliece and Niederreiter encryption algorithms were discovered in the late 20th century, but the problem with code-based signatures is there will be very large key sizes required. Because of this problem, the code-based cryptography has only seen potential application to encryption schemes and not as much into signature schemes.

### Code-based Signatures
(https://arxiv.org/pdf/1312.4265v1.pdf)

### Stern's Scheme
(https://eprint.iacr.org/2014/163.pdf)

## Post-Quantum Signatures Conclusion
There exist quite a number of signature schemes that have proven to be quantum resistant. In terms of applications, the most likely forms we may use will be Lattice-Based or Multivariate signature schemes. However, the biggest problem that we will see in the implementation of quantum resistant signatures are the large key sizes in comparison to current key sizes. This means that many protocols will need to be changed to work with the new signature schemes, as introduced in the suggestion of Transport Layer Security (TLS) and Internet Key Exchange (IKE). Another problem that remains to be solved is the fact that many of these quantum resistant algorithms are not guaranteed to be quantum resistant. To better explain, the only reason why some algorithms like RSA and ECDSA are provably broken by quantum computing is because of the creation of Shor’s Algorithm which shows that quantum will break the foundation of prime factorization. The current post-quantum algorithms rely on both a NP-Hard problem as well as not having a quantum algorithm that breaks that hardness. While there currently do not exist any solutions to Lattice-Based or Multivariate problems with quantum computing, that does not mean that there will not exist such algorithms in the future. The security of all these quantum-resistant schemes are based on the assumption that there will not be an algorithm found to break the schemes. Regardless of this, striving to improve the security of the internet and technology is paramount. These quantum algorithms hold just as much security as current cryptographic algorithms do today, because they are based on NP-Hard problems that have no solutions capable of breaking the security of the algorithms.

## Other Sources
- https://eprint.iacr.org/2004/297.pdf
- http://eprint.iacr.org/2013/383.pdf
- http://www.di.ens.fr/~ducas/NTRUSign_Cryptanalysis/DucasNin guyen_Learning.pdf