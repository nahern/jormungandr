---
title: "How to send money from the faucet to an account address?"
author: ["alejandro garcia"]
draft: false
---

Follow the instructions below or watch this video tutorial: [Jormungandr send transactions](https://youtu.be/6YFoitp-hsw)

On this tutorial we are going to go beyond simply setting up a node to actually transfer tokens from one account (the faucet) to another.
The faucet was created by the `bootstrap` script, and configured in the `genesis.yaml` file.
Now we are going to create a different account by ourselves and transfer funds to it.

It's important to note that there are two ways to do this by: account address and by UTXO.
In this tutorial we cover the first one.


## Creating a receiver account 

Now we are going to create an account that will receive funds from the faucet

<a id="code-snippet--creating-receiver-account"></a>
```bash
jcli key generate --type=Ed25519Extended > receiver_secret.key
cat receiver_secret.key | jcli key to-public > receiver_public.key
jcli address account --testing $(cat receiver_public.key) | tee receiver_account.txt
```

```text
ta1s5gpyg7tnvzjyq0t46xehcu5h2k2rwkjmgladja9ahlljygz0azcwvd45x2
```

with the receiver account, we can now send funds.


## Withdrawing from the faucet 

Now we are going to use the faucet-send-money.sh script that the bootstrap script created for us:

<a id="code-snippet--withdraw-from-faucet"></a>
```bash
./faucet-send-money.sh $(cat receiver_account.txt) 1000
```

```text
## Sending 1000 to ta1s5gpyg7tnvzjyq0t46xehcu5h2k2rwkjmgladja9ahlljygz0azcwvd45x2
discrimination: testing
account: ed25519_pk1zqfz8jumq53qr6aw3kd789964jsm45k68ltvhf0dllu3zqnlgkrsjgzg86
Success!
```

It will show a Success! message **but** this is a partial success. It means that the transaction was successfully created and submitted to the node. Next step is for the node to check the transaction and to include it (or not) to the blockchain. So next we need to wait for a new block to be created in order for the transaction to take effect.
Keep in mind, that blocks are created differently depending on the selected consensus mode (BFT or Genesis).


## Checking that the transaction is in the blockchain 

<a id="code-snippet--checking-transaction-in-blockchain"></a>
```bash
jcli rest v0 message logs -h http://127.0.0.1:8443/api
```

```text
---
- fragment_id: cd9f68517222d70537fd9dda056c4ca3036b57fc0733b907ce66deee36d4ac51
  last_updated_at: "2019-06-20T23:20:01.764334883+00:00"
  received_at: "2019-06-20T23:20:01.764334797+00:00"
  received_from: Rest
  status: Pending
```

If you do it immediately you will see a status of Pending. Wait and try again until the transaction is **InABlock**.
The waiting (slot) time is variable in the Genesis consensus and fixed in a BFT consensus.

<a id="code-snippet--checking-transaction-in-blockchain-InABlock"></a>
```bash
sleep 20
jcli rest v0 message logs -h http://127.0.0.1:8443/api
```

```text
---
- fragment_id: cd9f68517222d70537fd9dda056c4ca3036b57fc0733b907ce66deee36d4ac51
  last_updated_at: "2019-06-20T23:22:44.043796512+00:00"
  received_at: "2019-06-20T23:20:01.764334797+00:00"
  received_from: Rest
  status:
    InABlock:
      date: "0.75"
```

Now the transaction was accepted by the node and included into block 49.


## Reviewing the faucet and receiver balances 


### Checking the receiver account balance 

Let's check the balance of the faucet account

<a id="code-snippet--receiver-account-balance"></a>
```bash
jcli rest v0 account get $(cat receiver_account.txt) -h  http://127.0.0.1:8443/api
```

```text
---
counter: 0
delegation: ~
value: 1000
```

We see that we have the 1,000 tokens we sent.


### Checking the faucet account balance 

```bash
jcli rest v0 account get $FAUCET_ACCOUNT -h  http://127.0.0.1:8443/api
```

```text
---
counter: 1
delegation: 935e3de07b841d0ad3d854de88bad3d956299017a96be3fc32f7a1b404ffbe92
value: 999998990
```

Notice how the transaction `counter` has incremented and that there are less tokens than expected. It is because we withdraw 1,000 tokens plus 10 tokens to pay the transaction fee.

The transaction fee, was configured inside the `genesis.yaml` file in `linear_fees`.

The above steps concludes the basic usage of the self node.
