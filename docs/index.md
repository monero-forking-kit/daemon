# Documentation for Monero Forking Kit

<WORK IN PROGRESS>

* Setup your repo
    * Fork Monero-starting-kit
    * Install dependencies
    * Compile the project
* Modify cryptonote_config.h
    * Network parameters
    * Money supply
    * Genesis block
* Setup seed nodes
    * Create 2 virtual machines
    * Create DNS records
    * Update the code for this 
    * Compile 
* Launch your blockchain
    * Start the 2 nodes
    * Start mining
    * Bonus: start a mining pool

## Setup your repo

### Fork monero-starting-kit
The first thing to do is to fork the repo on github. You probably don't want to keep `monero-forking-kit` as the name of your repo. You can change the name in your repo settings. Then, you are ready to clone your forked repo on your local machine:
```
git clone https://github.com/<yourusername>/<yourreponame>.git
```

### Install dependencies
Follow the instructions in the `README.md` to install the dependencies on your machine, then compile the project. It should compile without problems. If not, you need to figure this out before continuing.

### Compile the project
To make sure everything is fine, try to compile the project by first jumping into your repo directory, and launching:
```
make
```
It should successfully complete the compilation process. Next, you will have to modify some parts of the c++ code. It is a good idea to try to compile your code after each change. If you break anything, you will realize it quickly and it will be easier to fix.


## Setup seed nodes
When a deamon (node in networking context) start, it needs to connect to the peer-to-peer network of the blockchain. More specifically, it needs to connect to other nodes already connected to the network, which in turn requires to know the ip addresses of those nodes. Seed nodes solve this problem, by keeping lists of connected nodes, and sending those lists to new nodes when they want to connect to the network.

Monero already have its nodes, but we cant use them. Each blockchain must have its own seed nodes. So you need to setup your n

ckchain is mostly decentralized, to solve this problem we need a centralized organization to coordinate
You need at least 2 nodes running on 2 different ips. The easiest way to do this would be to create 2 virtual machines wi

### Create 2 virtual machines
The daemon runs on different linuxes, but I recommends using Ubuntu 16.04. The installation works the required libraries work well, and the daemon compiles without problems. For other OS, like some versions of CentOS, you can easily spend days debugging the installation of some c++ libraries like boost, so I will not recommend trying that.

Once your VMs are ready, you need to make sure that the `P2P_DEFAULT_PORT` specified in your `src/cryptonote_config.h` is open for incoming and outgoing connections. If you also want to make your nodes available for wallets-only or other application that want to query the blockchain without storing a full blockchain, you will need to also open the `RPC_DEFAULT_PORT` for incoming connections. 

### Create DNS records
When the

```

## Modify cryptonote_config.h 
`src/cryptonote_config.h` contains most of the parameters used by the daemon.

If you are unsure what to modify, leave some .

* Network Parameters
*

* Money Supply

### Money Supply

When you start your blockchain, there are no coins. As time goes by, miners find new blocks and are rewarded with new coins created out of nowhere. Those coins are given to miners in so-called *coinbase transactions*. The amount of new coins created depends on the money supply parameters. 



You can use the [Emission calculator tool](https://cryptonotestarter.org/inner.html) of [cryptonotestarter.org](https://cryptonotestarter.org) to play with parameters

** Note 1: Keep in mind that all amounts used in the money supply parameters are expressed in basic units, which is equivalent to the concept of "cent" for dollars, as a comparison. For Monero, 1 Monero is equal to 10^12 basic unit.**

** Note 2: Keep in mind that numbers used have to fit in the c++ uint64 type, which is 2^63-1. You could still use a MONEY_SUPPLY amount superior to this number, but several years after the launch of your coin your blockchain will run into a big bug and will stop working. So be careful.**


#### Main Emission
* **Money_SUPPLY**: The total amount of coins which will be created during the main emission phase.
* **EMISSION_SPEED**: 
* **

#### Tail Emission
Tail emission occur once the *MONEY_SUPPLY* amount of coins has been created. To continue to incentivize the network to mine, a residual creation of coins goes on indefinetly. The parameters
A



Coins will be created by the miners during all the lifetime of your coin. The main emmission phase is when most of the coin will be mined
The Money Supply parameters determine the 
```
MONEY_SUPPLY
EMISSION_SPEED_FACTOR_PER_MINUTE  
#define EMISSION_SPEED_FACTOR_PER_MINUTE                (20)
#define FINAL_SUBSIDY_PER_MINUTE                        ((uint64_t)300000000000) // 3 * pow(10, 11)
```

### Money Supply
 ipsum

## Test 2
lorem ipsum

### Genesis Block

The genesis block is the first block of the blockchain. When the daemon start, if it does not find any blocks locally or on the network it will try to build the genesis block. The genesis block will use the hardcoded genesis trasnsaction (`GENESIS_TX`). Any change to the genesis transation requires to re-compile the daemon.

If you haven't changed any parameters related to the money supply, you don't need to change anything. You will not be able to recover the coins mined in the genesis block, but it is negligible.

Otherwise, congratulation, you gained the privilege to pull your hairs and delve into the wonderful world of binary formats! Fortunately for you, I have already been there before and laid the groundwork. Actually, the only required change is the amount of the coin base transaction, which needs to follow 

You will need to understand:
* How
* varint encoding

In Monero, the genesis transaction is:
```
"013c01ff0001ffffffffffff03029b2e4c0281c0b02e7c53291a94d1d0cbff8883f8024f5142ee494ffbbd08807121017767aafcde9be00dcfd098715ebcf7f410daebc582fda69d24a28e9d0bc890d1"
```

Below is a breakdown of all the fields, including the coinbase amount
For a thorough analysis of the


```
VALUE            NAME          TYPE     DESCRIPTION
=====            =====         ====     ===========
"01"             tx_version    varint&ast;   Transaction format version
"3c"             unlock_time   varint   Unlock time after which mined coined can be used 

                                        -- Inputs Section --
"01"             input_num     varint   Number of inputs (1 for coinbase transactons)
                                        --> Array Of Inputs (see [CNS004, 3.2](https://cryptonote.org/cns/cns004.txt))
"ff"             input_type    byte     Type of input (0xff for  base transactions)
"00"             height        varint   Height of the block which contains the transaction
                                        <--

                                        -- Outputs Section --
"01"             output_num    varint   Number of outputs
                                        --> Array of inputs (see [CNS004, 3.3](https://cryptonote.org/cns/cns004.txt))
**"ffffffffffff03" amount        varint   The value of the coinbase transaction (Decimal value: 17592186044415)**
"02"             output_type   varint   Type of output (to key, 0x02)
"9b2e4c0281c0b02e7c53291a94d1d0cbff8883f8024f5142ee494ffbbd088071"
                 key           pubkey   32 bytes public key representing the destination of this output
                                        <-- 
                                          
                                        -- Extra Section (see [CNS005](https://cryptonote.org/cns/cns004.txt))
"21"             extra_size    varint   Number of bytes in the Extra field 
                                        --> Array of extra fields
"01"             tag           byte     Sub-field tag (transaction publick key, 0x01)
"7767aafcde9be00dcfd098715ebcf7f410daebc582fda69d24a28e9d0bc890d1"
                 data          pubkey   Transaction pubkey (32 bytes)
```
&ast; varint (CNS003, 3.3](https://cryptonote.org/cns/cns003.txt)
