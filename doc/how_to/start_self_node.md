---
title: "How to start a self node?"
author: ["alejandro garcia"]
draft: false
---

Follow the instructions below or watch this video tutorial: [Jormungandr bootstrappig](https://youtu.be/M%5F7ZJKQnv%5FY)


## Execute the bootstrap script 

Use the bootstrap script to start a single self node.

Create directory `jor-test/self-node` to store the configurations

```bash
mkdir -p ~/jor-test/self-node
cd ~/jor-test/self-node
```

Now let's execute the bootstrap script. Use `tee` command since you want to display and save the output to file.

<a id="code-snippet--initial-configuration"></a>
```bash
~/jor-test/jormungandr/scripts/bootstrap | tee initial-configuration.txt
```

```text
########################################################

* Consensus: genesis
* REST Port: 8443
* Slot duration: 10

########################################################

* CLI version: jcli 0.2.1
* NODE version: jormungandr 0.2.1

########################################################

faucet account: ta1skv9h5fmwvd2s55adv7edn7tk62xt3t34zgw5g4ue2g2n0t94wtq6hfgmcn
  * public: ed25519_pk1npdazwmnr25998tt8ktvljakj3ju2udgjr4z90x2jz5m6edtjcxs0uyel6
  * secret: ed25519e_sk1uzea43a80qjaklfc3f99a7fp30lecec6tv95ny39ngl0ag58epp4gs0ffknqh84zh7nuxmau3c5cd7mefj6epvh5cshmr9p3md9m8fq9ehu6g
  * amount: 1000000000

pool id: 935e3de07b841d0ad3d854de88bad3d956299017a96be3fc32f7a1b404ffbe92
block-0 hash: d70495af81ae8600aca3e642b2427327cb6001ec4d7a0037e96a00dabed163f9

To start the node:
  jormungandr --genesis-block ./block-0.bin --config ./config.yaml --secret ./pool-secret1.yaml
To connect using CLI REST:
  jcli rest v0 <CMD> --host "http://127.0.0.1:8443/api"
For example:
  jcli rest v0 node stats get -h "http://127.0.0.1:8443/api"
```

It's important to note that the `bootstrap` script has several parameters the most import ones are:

-p
: Setting the port for the  REST api. By default it's 8443

-b
: Start an Ouroboros **BFT** blockchain. BFT is a good blockchain to use for debugging purposes, since slots are warranted to last 10 seconds.

-g
: Start an Ouroboros **Genesis** blockchain. It's the newest protocol. Slot duration is variable.

To check the other parameters just do a

```text
bootstrap -h
```


## Review the files that were created by the bootstrap script 

The files that the bootstrap script created are:

```bash
ls -l
```

```text
total 32
-rw-r--r-- 1 agarciafdz agarciafdz  544 jun 20 18:10 block-0.bin
-rw-r--r-- 1 agarciafdz agarciafdz  261 jun 20 18:10 config.yaml
-rwxr-xr-x 1 agarciafdz agarciafdz 3107 jun 20 18:10 create-account-and-delegate.sh
-rwxr-xr-x 1 agarciafdz agarciafdz 3017 jun 20 18:10 faucet-send-certificate.sh
-rwxr-xr-x 1 agarciafdz agarciafdz 2990 jun 20 18:10 faucet-send-money.sh
-rw-r--r-- 1 agarciafdz agarciafdz 1067 jun 20 18:10 genesis.yaml
-rw-r--r-- 1 agarciafdz agarciafdz 1040 jun 20 18:10 initial-configuration.txt
-rw-r--r-- 1 agarciafdz agarciafdz 2237 jun 20 18:10 pool-secret1.yaml
```

These files include:

block-0.bin
: The encoded version of the `genesis.yaml` file

config.yaml
: Contains the configuration options of the **node**. Not to be confused with the configuration of the blockchain (in genesis.yaml)

create-account-and-delegate.sh
: used to demonstrate delegat

faucet-send-certificate.sh
: Script created by bootstrap and will be used when we need to delegate stake

faucet-send-money.sh
: Another script created by bootstrap. It sends money from the faucet to another account.

genesis.yaml
: Configuration options of the **blockchain**.

initial-configuration.txt:
: This was created by the `tee` command to save relevant keys used later.

pool-secret1.yaml
: The configuration file of the stakepool (in this case we only have one stakepool).


## Starting the self node 

The bootstrap script also suggests the command to run a node.
In this example, we are going to send the log messages to a file with `&>my_node.log`, so that we can analyze them later.
We will run the process in the background with `&`.

<a id="code-snippet--starting-jormungandr"></a>
```bash
jormungandr --genesis-block ./block-0.bin --config ./config.yaml --secret ./pool-secret1.yaml &> my_node.log &
```

you can check what the log contains with the `tail` command:

<a id="code-snippet--tail-installation"></a>
```bash
tail my_node.log

```

```text
Jun 20 18:13:33.306 INFO storing blockchain in '"/home/agarciafdz/jor-test/self-node/jormungandr-storage-test/blocks.sqlite"', task: init
Jun 20 18:13:33.371 WARN no gRPC peers specified, skipping bootstrap, task: bootstrap
Jun 20 18:13:33.373 INFO starting task, task: client-query
Jun 20 18:13:33.373 INFO starting task, task: network
Jun 20 18:13:33.373 INFO our node id: 86467880508638223666813496146945725477, task: network
Jun 20 18:13:33.373 INFO adding P2P Topology module: trusted-peers, task: network
Jun 20 18:13:33.374 INFO start listening and accepting gRPC connections on 127.0.0.1:8299, task: network
Jun 20 18:13:33.374 INFO preparing, task: leadership
Jun 20 18:13:33.374 INFO starting, task: leadership
Jun 20 18:13:33.374 INFO starting, sub_task: End Of Epoch Reminder, task: leadership
```


## Checking the initial balance 

Now that we have a node running let's check the initial balance of the faucet account.

In case you forgot the data, you can check the initial configuration.txt file.

<a id="code-snippet--cat-initial-configuration"></a>
```bash
cat initial-configuration.txt
```

```text
########################################################

* Consensus: genesis
* REST Port: 8443
* Slot duration: 10

########################################################

* CLI version: jcli 0.2.1
* NODE version: jormungandr 0.2.1

########################################################

faucet account: ta1skv9h5fmwvd2s55adv7edn7tk62xt3t34zgw5g4ue2g2n0t94wtq6hfgmcn
  * public: ed25519_pk1npdazwmnr25998tt8ktvljakj3ju2udgjr4z90x2jz5m6edtjcxs0uyel6
  * secret: ed25519e_sk1uzea43a80qjaklfc3f99a7fp30lecec6tv95ny39ngl0ag58epp4gs0ffknqh84zh7nuxmau3c5cd7mefj6epvh5cshmr9p3md9m8fq9ehu6g
  * amount: 1000000000

pool id: 935e3de07b841d0ad3d854de88bad3d956299017a96be3fc32f7a1b404ffbe92
block-0 hash: d70495af81ae8600aca3e642b2427327cb6001ec4d7a0037e96a00dabed163f9

To start the node:
  jormungandr --genesis-block ./block-0.bin --config ./config.yaml --secret ./pool-secret1.yaml
To connect using CLI REST:
  jcli rest v0 <CMD> --host "http://127.0.0.1:8443/api"
For example:
  jcli rest v0 node stats get -h "http://127.0.0.1:8443/api"
```

There we can see that the initial private key is:

```text
 ed25519e_sk1uzea43a80qjaklfc3f99a7fp30lecec6tv95ny39ngl0ag58epp4gs0ffknqh84zh7nuxmau3c5cd7mefj6epvh5cshmr9p3md9m8fq9ehu6g
```

And the faucet account is:

```text
 ta1skv9h5fmwvd2s55adv7edn7tk62xt3t34zgw5g4ue2g2n0t94wtq6hfgmcn
```

With that information we can check the original balance in the account.

<a id="code-snippet--check-faucet-balance"></a>
```bash
jcli rest v0 account get $FAUCET_ACCOUNT -h  http://127.0.0.1:8443/api
```

```text
---
counter: 0
delegation: 935e3de07b841d0ad3d854de88bad3d956299017a96be3fc32f7a1b404ffbe92
value: 1000000000
```

It should be the same that we created with bootstrap, 1000000000 tokens.
