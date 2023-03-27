# ScroogeCoin
ScroogeCoin implementation in Python 3. This currency is defined in the book "Bitcoin and Cryptocurrency Technologies" (Princeton University)

## High level comments on the currency
**Scrooge**
- Scrooge is a trusted entity that creates coins, manages the blockchain and approves payments.

**Coin creation**
- Only Scrooge can create coins.

**Payment**
- Whoever owns a coin can transfer it on to someone else.
- Payments must be signed by all the owners whose coins are consumed in the transaction.
- Scrooge verifies signatures and checks double-spending before approving the transaction.
- When a coin is consumed it is deleted and other coin with the new owner is created.
- The amount of created coins in a payment must be equal to the amout of consumed coins.

**Blockchain**
- Transactions are inserted by Scrooge into the blockchain.
- Scrooge publishes the blockchain along with the signature of the last block.

**Wallet**
- A wallet is identified by the sha256 hash of its public key.

**Signing and hashing**
- *ecdsa* is used for signing.
- *sha256* is used for hashing.

## Dependencies
- [ecdsa](https://github.com/warner/python-ecdsa)

## Usage example
**Creating Scrooge**

A wallet is assigned to Scrooge when it is created. The blockchain is created too, and the genesis block contains a transaction that creates a coin assigned to Scrooge's wallet. 
```
scrooge = Scrooge()
print(scrooge.wallet)
print(scrooge.blockchain)
```

**Output**

```
Wallet
------------------------------
Id: 6d5a0eab2d03a8a7d033f02ddaf3688196d26c51cafebdba331910ca06f232dd
------------------------------

Blockchain 
------------------------------
Block: 0	Hash previous block: None
TransID: 0	Type: Coin creation

Created coins: 
Num: 0, Value: 1, Wallet id: 6d5a0eab2d03a8a7d033f02ddaf3688196d26c51cafebdba331910ca06f232dd
------------------------------
```

**Coin creation**

Coins have an id, a value, and the wallet id of the owner. The coin id is assigned by Scrooge when the transaction that creates the coin is inserted into the blockchain. This id is composed by its transaction id and the correlative number of the coin in the transaction.
```
 wallet_1 = Wallet()
 coins = [
     Scroogecoin(value=200, wallet_id=wallet_1.id),
     Scroogecoin(value=500, wallet_id=wallet_1.id)
 ]
 scrooge.create_coins(coins)
 print(scrooge.blockchain)                
```

**Output**

```
Blockchain 
------------------------------
Block: 0	Hash previous block: None
TransID: 0	Type: Coin creation

Created coins: 
Num: 0, Value: 1, Wallet id: 6d5a0eab2d03a8a7d033f02ddaf3688196d26c51cafebdba331910ca06f232dd
------------------------------
Block: 1	Hash previous block: 7c2d2f27a859b498315e58c4cffb026582be82006c0d39b37a4f26151c86db73
TransID: 1	Type: Coin creation

Created coins: 
Num: 0, Value: 200, Wallet id: 26a9a4697e671a6ee06ae447536cb35ad1cf6ba39df80cf21f791bea175799c1
Num: 1, Value: 500, Wallet id: 26a9a4697e671a6ee06ae447536cb35ad1cf6ba39df80cf21f791bea175799c1
------------------------------
``` 

**Payment**

The following code executes a 600 scroogecoins payment from wallet_1 to wallet_2. The payment is created and sent to Scrooge along with the verifying key and the signature of the wallet_1. 
```
wallet_2 = Wallet()
pay_coin = Scroogecoin(value=600, wallet_id=wallet_2.id)
payment = Payment(created_coins=[pay_coin], consumed_coins=created_coins)
signature = wallet_1.sign(encoded_hash_object(payment))
payment_result = scrooge.process_payment(
    payment, [(wallet_1.verifying_key, signature)]
)
print(scrooge.blockchain)
```
**Output**

The output shows that one coin had to be "devided" to pay the 600 scroogecoins. The payment of the block 2 represents the coin division, in which the source wallet is equal to the destination wallet, and a coin with value 500 was devided into two coins with values 400 and 100. The block 3 contains the transfer between wallet_1 and wallet_2. 
```
Blockchain 
------------------------------
Block: 0	Hash previous block: None
TransID: 0	Type: Coin creation

Created coins: 
Num: 0, Value: 1, Wallet id: 6d5a0eab2d03a8a7d033f02ddaf3688196d26c51cafebdba331910ca06f232dd
------------------------------
Block: 1	Hash previous block: 7c2d2f27a859b498315e58c4cffb026582be82006c0d39b37a4f26151c86db73
TransID: 1	Type: Coin creation

Created coins: 
Num: 0, Value: 200, Wallet id: 26a9a4697e671a6ee06ae447536cb35ad1cf6ba39df80cf21f791bea175799c1
Num: 1, Value: 500, Wallet id: 26a9a4697e671a6ee06ae447536cb35ad1cf6ba39df80cf21f791bea175799c1
------------------------------
Block: 2	Hash previous block: 1a13477397e4572ecd6f60c34b681d46a5c1ddb4570ee21c9b3cd4bf983ec8ba
TransID: 2	Type: Payment

Consumed coins: 
Num: 1, Value: 500, Wallet id: 26a9a4697e671a6ee06ae447536cb35ad1cf6ba39df80cf21f791bea175799c1

Created coins: 
Num: 0, Value: 400, Wallet id: 26a9a4697e671a6ee06ae447536cb35ad1cf6ba39df80cf21f791bea175799c1
Num: 1, Value: 100, Wallet id: 26a9a4697e671a6ee06ae447536cb35ad1cf6ba39df80cf21f791bea175799c1
------------------------------
Block: 3	Hash previous block: d8b94a19925f9b59bf15ea6fbf63d10a867ae398f8e43738ec2ed3a73abbd0fb
TransID: 3	Type: Payment

Consumed coins: 
Num: 0, Value: 200, Wallet id: 26a9a4697e671a6ee06ae447536cb35ad1cf6ba39df80cf21f791bea175799c1
Num: 0, Value: 400, Wallet id: 26a9a4697e671a6ee06ae447536cb35ad1cf6ba39df80cf21f791bea175799c1

Created coins: 
Num: 0, Value: 200, Wallet id: 5407beb09e3a47b00948451d4d49ed0137adadfb308094b14399d04a5148c13b
Num: 1, Value: 400, Wallet id: 5407beb09e3a47b00948451d4d49ed0137adadfb308094b14399d04a5148c13b
------------------------------

```
