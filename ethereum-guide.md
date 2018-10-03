# Hands-On Ethereum Guide
By [Maximilian Schiedermeier](https://liris.cnrs.fr/page-membre/maximilian-schiedermeier),  
[Email](mailto:maximilian.schiedermeier@insa-lyon.fr), [Public Encryption Key](https://pgp.mit.edu/pks/lookup?op=vindex&search=0x7010339C4391A4F3)  
[Liris labs](https://liris.cnrs.fr/), [DRIM](https://liris.cnrs.fr/en/team/drim), [INSA Lyon](https://www.insa-lyon.fr/).

*Note: This file is under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/) License.*

## Machine Setup

### Tools

For building an Ethereum based Java-Dapp (Distributed Application), you need three things:

 * [Geth](https://github.com/ethereum/go-ethereum/wiki/geth): Command line interface for running a full ethereum node.
 * [Web3J](https://web3j.io/): Java library for integrating with nodes on Ethereum blockchains.
 * [Solidity](https://solidity.readthedocs.io/en/v0.4.25/): Language for implementing smart contracts, designed to target the Ethereum Virtual Machine.

### UNIX (Like) Installation

#### Ubuntu / Raspbian

 * Install geth:  

```bash
    sudo apt-get install software-properties-common
    sudo add-apt-repository -y ppa:ethereum/ethereum
    sudo apt-get update
    sudo apt-get install ethereum  
```

*(Gentoo: ```emerge -a net-p2p/go-ethereum````)*

 * Download Web3J, extract and create terminal reference:  

```bash
    echo "[Downloading...]"  
    wget https://github.com/web3j/web3j/releases/download/v3.5.0/web3j-3.5.0.zip  

    echo "[Extracting...]"  
    unzip web3j-3.5.0.zip  
    rm web3j-3.5.0.zip  

    echo "[Linking...]"  
    echo '#!/bin/bash' >> ~/bin/web3j  
    echo '~/web3j-3.5.0/bin/web3j "$@"' >> ~/bin/web3j  
    chmod +x ~/bin/web3j
```

*Note: Make sure the ~/bin/ directory is in your [PATH](https://opensource.com/article/17/6/set-path-linux).*


 * Install solidity:  

```bash
    sudo apt-get install solc
```

*(Gentoo: ```emerge -a dev-lang/solidity```)*


#### Mac

 * Install [Brew](https://brew.sh/), the mac package manager.  

```bash
    /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

 * Install geth:

```bash
    brew update  
    brew upgrade  
    brew tap ethereum/ethereum  
    brew install ethereum  
```

 * Download Web3J, extract and create terminal reference:  

*Same as Linux instructions above, but first install the download tool, using ```brew install wget```*

 * Install solidity:  

```bash
    brew install solidity  
    brew linkapps solidity  
```

### Windows

You have three options, but I strongly recommend the third:  
 * Find a way to install the tools for windows, adapt all unix-scripts that show up in this guide.  
*You will be slower than other teams, without our support and you will have to invest extra time outside the regular cursus.*  

 * Run Linux within your Windows installation, using a virtualization software. 
*This will be more complicated than the original guide, as soon as we get to networking. You will not get any help from us on that behalf.*
    * Windows 10: You can use [HyperV](https://www.windowscentral.com/how-run-linux-distros-windows-10-using-hyper-v)  
    * Windows XP/Vista/7/8: Use [VirtualBox](http://www.psychocats.net/ubuntu/virtualbox)  

 * [Install Linux natively](https://builtvisible.com/the-ubuntu-installation-guide/) on your PC. You can install it next to your Windows installation.

### Files

All files relevant to the following guide reside in a special directory, the *ethereum base directory*. Your blockchain data and affiliated accounts reside in that directory. If you delete it you start from scratch.  
Mac users: Your *ethereum base directory* is: ```~/Library/Ethereum/```  
Linux users: Your *ethereum base directory* is: ```~/.ethereum/```  

For now it is empty but as soon as you interact with a blockchain you will see the following structure:  

        .ethereum/
            keystore/	
                .... // Account data for the main chain  
            yourblockchainnamewillbehere/  
                geth  
                    ... // Actual blockchain data of that chain  
                history  
                keystore  
                    ... // Account data of that chain  

## Chains

### Genesis Block

By default all ```geth``` commands are executed scoped on the main ethereum blockchain. As interacting with the main chain implicitly means spending *real money*, we will advise geth to operate on our own private chain.
To do so we first need to create a new blockchain. Each blockchain starts with a genesis block, the chains first block. The following steps show you how to create a new chain, based on a manually coded json representation of a genesis block.  
*Note: When working on the main chain this step is not necessary. The main chain genesis block is hard coded into geth.*  

Navigate to your *ethereum base directory* and create a file named ```genesis.json```.  
Paste the following content into that file. **Copy & Paste is not enough, you still have to modify the content afterwards!**

```json
    {
        "alloc": {},
        "coinbase": "0x0000000000000000000000000000000000000000",
        "config": {
       	    "chainId": 1608199012345,
       	    "eip155Block": 0,
       	    "eip158Block": 0,
       	    "homesteadBlock": 0
        },
        "difficulty": "0x20000",
        "extraData": "",
        "gasLimit": "0x2fefd8",
        "mixhash": "0x0000000000000000000000000000000000000000000000000000000000000000",
        "nonce": "0x0000000000000042",
        "parentHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
        "timestamp": "0x00"
    }
```

Essentially, this description nails down the characteristics of your new custom blockchain. However in a network, there must not be two different chains using the same *chainId*. To avoid that, edit the *chainId* value and fill in a number that is sufficiently high and unique, e.g. your birthday + 12345, then save the file. This is the only modification needed.

For an explanation of the remaining fields, [look here](https://ethereum.stackexchange.com/questions/2376/what-does-each-genesis-json-parameter-mean).

### Blockchain instantiation

Next instantiate a blockchain, using this genesis block:  

        cd ~/.ethereum
        geth --datadir="yourChainName" init genesis.json

**Do not use the identifier "yourChainName"**  (use your own unique name, e.g. "sharedchainBinome42")  
Once executed, you should see a line alike: ```Successfully wrote genesis state```

Also, this will create a new subdirectory in your *ethereum base directory*. If you delete it, your blockchain and all associated accounts disappear, too.

## Accounts

*More information on accounts [here](https://github.com/ethereum/go-ethereum/wiki/Managing-your-accounts).*

Every interaction with a blockchain takes place on behalf of an account. So in order to do anything with your newly created blockchain you now need at least one account.  
**Hint:**
By default all geth commands are bound to the default chain. This involves the creation of new accounts. You can override this behavior by providing a path to your own chain. Use the ```--datadir /some/path/``` switch. You can also use relative paths. A good practice is to ```cd``` into your *ethereum base dir* and then only provide the name of your chain as parameter.

### Querying

At any given moment you can list the available accounts, using:  

        geth account list  
        WARN [04-11|14:19:13] No etherbase set and no accounts found as default  

As you have not yet added an account you will see the above warning. So lets see how to add a new account  

*Note: Unless another account is explicitly specified, geth uses your oldest account as default.*

### Creation

The ```geth account new``` command adds a new account. When prompted enter the same password twice:  

```bash
    geth account new  
    Your new account is locked with a password. Please give a password. Do not forget this password.  
    Passphrase:  
    Repeat passphrase:  
    Address: {67b7b218cb53611612e2983cf2fa45ce01540cf9}  
```

The last line indicates the unique address of your new account. When later transferring ether between accounts we need this address. This address is also referred to as the coinbase.  
If you now check the content of your ~/.ethereum directory the newly created account will show up there.

        schieder@chartreuse:.ethereum $ tree  
        .  
        --- keystore  
        ------ UTC--2018-04-12T11-56-57.442014523Z--67b7b218cb53611612e2983cf2fa45ce01540cf9  

Of course you do not need to inspect the file system to find your new account you can also retrieve a list of accounts using:  

        geth account list  
        Account #0: {564a2f909e604268d865654086a36f19ac915b07} keystore:///home/schieder/.ethereum/keystore/UTC--2018-04-11T13-03-56.222342097Z--564a2f909e604268d865654086a36f19ac915b07
        

## Balance

Each account has an amount of ether attached. This amount can be considered as *funds*. Stakeholders who want to deposit something into the blockchain must pay a tiny amount of ether to do so.  
Without ether, accounts are very limited in their actions.

*Additional information and commands related to mining can be found [here](https://github.com/ethereum/go-ethereum/wiki/mining) and [here](https://github.com/turboprep/geth-tutorial).*

### Querying

Querrying is the act of resolving the amount of ether attached to an account, regarding the blockchains most recent state. To look that amount up, first open a geth console, using:

        geth --nodiscover console  

*Note: By nature ```geth``` is designed for P2P scenarios. Unless you specify otherwise, geth will look out for other ethereum nodes and accept incoming connections. The following examples are altogether commands where you operate on your own local chain and explicitly do not want any P2P functionality. Tell ```geth``` to not interact with other nodes, by launching it using the ```--nodiscover``` option.*

*Note: When launching geth with the ```console``` argument you effectively start a new inner console. To kill a running command, use ```Ctrl-C```. ```Ctrl-D``` brings you back to the parent bash.*

To then print the current balance in *Wei*, type:  


        eth.getBalance("0x...hashofyouraccounthere")  

*Note: 1.000.000.000.000.000.000 Wei = 1 Ether*

If you prefer a direct conversion into *Ether*, wrap your command with a call to web3j.  

```javascript
    web3.fromWei(eth.getBalance(eth.coinbase), "ether")  
```


By now you have not done any mining. Therefore you will see a sad output of ```0``` either way.

### Mining

Mining is the most straightforward way to obtain ether. Essentially it means to invest computational power to keep the blockchain alive. To append the chain by a new block we need to solve a puzzle. The miner who first resolves it is immediately rewarded by a single ether.

You have two options for mining. 

 * Open a geth console and type ```miner.start()```

 * Launch geth with the ```--mine``` option.  

#### DAG generation

The first time you start to mine, geth will build a [directed acyclic graph](https://github.com/ethereum/wiki/wiki/Mining#ethash-dag). Having a DAG is precondition to running ethereums [proof-of-work](https://cointelegraph.com/explained/proof-of-work-explained) algorithm and therefore also to mining itself. Building the DAG is memory intense. In the context of ethereum it serves as an integrated protection against [ASIC mining](https://www.digitaltrends.com/computing/what-is-an-asic-miner/).  
Throughout the DAG generation you will repeatedly see some log: ```Generating DAG in progress```. Just be patient until the DAG generation is complete. This step is only necessary the very first time you start to mine.

#### Reward

Whenever your miner found a block, you will see the following:  

```bash
    Commit new mining work  
    Successfully sealed new block  
```

This implicitly means one ether was added to your account.

#### Multi-Cores

By default geth mines with only a single CPU core. (You can argue about [mining on the main chain with CPU power](https://bitcoin.stackexchange.com/questions/5608/how-does-one-calculate-the-profitability-of-gpu-mining), but on a private chain this is totally reasonable.)  
Anyhow you can slightly speed up the "Obtaining Ether" process by telling ```geth``` to use more CPU cores for the mining.  

```bash
    geth --mine --minerthreads=4
```
#### Account Selection

If you have more then one account you should specify which account you want to mine for. If you don't, obtained ether is attributed to the default account. You can query this using geth's console:   

```bash
    eth.coinbase
```

To override the target account, specify a custom etherbase when launching ```geth```:

```bash
    --etherbase '0xa4d8e9cae4d04b093aac82e6cd355b6b963fb7ff'  
```

Alternatively you can refer to the index of the account. The order corresponds to their timestamps creation. You can also look things up using:  

```bash
    geth account list
```

### Transferring Ether

Another way to boost an accounts balance is to transfer ether from another account. Before actually transferring the ether, the sending account must be unlocked. In the following example the first account at index ```0``` was unlocked for 15 seconds, using the password ```abc123```. The seconds command sends ```42``` ether from account at index ```0``` to account at index ```1```.

```bash
    > web3.personal.unlockAccount(web3.personal.listAccounts[0],"abc123", 15)
    true
    > eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[1], value: web3.toWei(42, "ether")})
    INFO [09-26|09:52:24.639] Setting new local account                address=0x3c7b081d3e608525A2efB821e80d597Cef7a435c
    INFO [09-26|09:52:24.640] Submitted transaction                    fullhash=0x08da052064eb4ffb20396cfef0f83e2ba09c932db26a3788f770638ea1200754 recipient=0xd929a310108EA93d2D82DF4A74763eD1316F94Fb
    "0x08da052064eb4ffb20396cfef0f83e2ba09c932db26a3788f770638ea1200754"
```

*Note: Instead of specifying source and target account by ```eth.accounts[x]``` you can always use hash-adresses wrapped in ticks: ```'0x3c7b081d3e608525a2efb821e80d597cef7a435c'```*  

As you remember querying the balance only tells the amount of ether based on the *current blockchain state*. For your transfer to be visible in the account listing, it must still become part of the blockchain. Precisely the blockchain must be appended by a new block to contain the tranfer. As you will read [a little later](#transactions), transfering ether is one of three possible *transaction* types. From the moment of their registration, transactions first remain in a pending state where they are only queued for persistence. It is only when a new block is mined that they are eventually integrated into the blockchain. Precisely they are the payload of that new block.  
**tl;dr: Your transfer has no visible effect until you mine at least one more block.**

If you forget to unlock the senders account or exceed the authorized time frame (here 15 seconds), the transfer will be rejected.
```bash
    > eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[1], value: web3.toWei(42, "ether")})
    Error: authentication needed: password or unlock
        at web3.js:3143:20
        at web3.js:6347:15
        at web3.js:5081:36
        at <anonymous>:1:1
```

## Custom Functions

In the previous sections you already used ```geth console``` to access geths functions from the built in command line. You can extend the supplied set by your our own custom functions. Following the consoles javascript syntax you can then dynamically load and call functionality defined in external files.  

Loading such a file is fairly easy, simply type in your console:  

```javascript
    loadScript("/path/to/your/custom/function.js")
```  

The following example prints a list of all account balances. See [here](https://ethereum.gitbooks.io/frontier-guide/listing_accounts.html) for the original.

```javascript
    function checkAllBalances() {  
        var totalBal = 0;  
        for (var acctNum in eth.accounts) {  
            var acct = eth.accounts[acctNum];  
            var acctBal = web3.fromWei(eth.getBalance(acct), "ether");  
            totalBal += parseFloat(acctBal);  
            console.log("  eth.accounts[" + acctNum + "]: \t" + acct + " \tbalance: " + acctBal + " ether");  
        }  
        console.log("  Total balance: " + totalBal + " ether");  
    };  
```

Once you have imported a script, you can directly call functions defined within. In regard to the above listing, this would be:  

        checkAllBalances();

## Basic Programmatic Access

Thus far all described blockchain interactions were executed manually. This means whenever you accessed the blockchain, you did so using either the ```geth``` command or its built-in console. But geth can also expose its functionality and work as a proxy. The two most outstanding ways to achieve this are:  

 * Using *RPCs* (Remote Procedure Calls)  
Geth acts as a proxy on a given port and ip. Can be seen as the more generic one, for it allows accessing a remote node over a network connection.

 * Using *IPC* (Inter Process Communication)
Geth opens a unix socket for other processes running on the same machine. There are ways to later expose the socket over the network, but usually this method is only used for local access.

This guide covers *RPC* access.

*Note: You are still working on a local chain with only a single node. The network functionality described here only concerns the communication with local programs that use your geth node as an entry point.*  

### Preparations

Before using ```geth```-*RPC* functionality from a high-level programming language, you must ensure that...

 * ... you have access to an account with some ether.
 * ... you have access to an *RPC-enabled* geth node.

#### Have an account with some ether

By now you should already have this covered. If not, create your [private blockchain](#chains), create an [account](#accounts), then [mine](#mining) until you have about 20 ether. 

#### Run an HTTP-enabled geth node

As *RPC* is a network-protocol, you could send your HTTP commands to any available *RPC*-enabled node. Unfortunately in the context of a private local chain there are none around. You have to operate your own node for all proxied access. Remember, this node must keep running throughout the entire execution of your program.  

```bash
    geth --rpcapi personal,db,eth,net,web3 --rpc --mine --minerthreads=4 console  
```

The ```--rpcapi``` and  ```--rpc``` arguments advise ```geth``` is to power up the *RPC interface*.  

For the exact meaning of the remaining parameters, see [here](https://ethereum.stackexchange.com/questions/41112/what-do-these-geth-rpcapi-parameters-mean-db-and-net) and [here](https://github.com/ethereum/go-ethereum/wiki/Management-APIs)  

*Note: As mentioned in the ["Transferring Ether"](#transferring-ether) section, a blockchain without miners is "dead". If you forget the ```--mine``` option, your Dapp will not be able to execute actions that internally require [transactions](#transactions). If you use synchronous API-calls it will even block.*


#### Test accessing your node

You can easily test access to your node using ```curl```. More information [here](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getbalance).

```bash
    curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x3c7b081d3e608525a2efb821e80d597cef7a435c", "latest"],"id":1}' http://localhost:8545 
```

*Note: Reply to the above balance request is the amount in Wei in [hexadecimal format](https://www.rapidtables.com/convert/number/hex-to-decimal.html).*

### Java Access

This section demonstrates the minimal setup towards a Java driven blockchain access.

#### Dependencies

Though vanilla Java provides utilities for HTTP communication, we will use the ```web3j``` library to abstract this layer away from our code. 

Easiest way is to set up your java project to use a [build automation tool](https://en.wikipedia.org/wiki/Build_automation) that resolves the dependencies for you. In your IDE, [create a new maven project](https://www.jetbrains.com/help/idea/maven-support.html). To then integrate ```web3j``` into your project, add the following dependency to your ```pom.xml```: 

```xml 
    <dependencies>
    ...
        <dependency>
            <groupId>org.web3j</groupId>
            <artifactId>core</artifactId>
            <version>3.5.0</version>
        </dependency>
    ...
    </dependencies>
```

*Note: Some IDEs, (e.g. IntelliJ) do not detect changes to your maven configuration files by default. This means that your java code cannot use external libraries, even if you added them to your ```pom.xml```. Look out for a corresponding popup and select "[enable auto-imports](http://testerhq.com/2013/09/18/do-enable-auto-import-in-intellij-for-maven-projects-need-to-be-imported/)".*

#### Code

The simplest example once more is to transfer ether between accounts. To implement this with java/web3j you need to do three things:  .

 * Connect to a local node.  
Default access is ```127.0.0.1:8545```. Make sure the port is not blocked by your firewall or another application.

 * Load credentials for an existing account, using its physical location and password.  
You can look up the location by ```cd```ing to your *ethereum base dir*, then running the ```tree``` command. ([Mac installation](https://rschu.me/list-a-directory-with-tree-command-on-mac-os-x-3b2d4c4a4827))

 * Trigger the actual sending of ether.  
This line may block a while depending on the cpu-power of your background miner. Typical response time is 15-20 seconds.

Combined we end up with the following code snippet:  
```java
        // Connect to local node
        Web3j web3 = Web3j.build(new HttpService());  // defaults to http://localhost:8545/

        // Load credentials for accessing wallet of source account
        Credentials credentialsWallet1 = WalletUtils.loadCredentials(SOURCE_ACCOUNT_PASSWORD, LOCATION_SOURCE_ACCOUNT);

        // Transfer specified amount of ether to target account
        Transfer.sendFunds(web3, credentialsWallet1, TARGET_ACCOUNT, BigDecimal.valueOf(AMOUNT), Convert.Unit.ETHER).sendAsync().get();
```

*Note: Full demo code available [here](https://kartoffelquadrat.eu:5050/maex/transfer-ether-demo).*

Also, you will be able to track the execution of your program on the debug output of your geth node. Between the mining messages you should see:  
```bash
    INFO [09-26|14:43:33.350] Submitted transaction                    fullhash=0x08026e37dcdec6790da17773f226c3e9190305f9b63f9c0aa0b025280057e2de recipient=0xd929a310108EA93d2D82DF4A74763eD1316F94Fb
```

You can directly store the return value of the third line in a transaction receipt object, within in your java code. More information on using the web3j API and many great code examples [here](https://web3j.readthedocs.io/en/latest/getting_started.html).

## Smart Contracts


A smart contract is a program that resides in the ethereum blockchain. Ethereum provides strong guarantees for its integrity and we can therfore trust the correctness of generated outputs.

*Note: "Correctness" applies from a security point of view, here. We stipulate that computed outputs correlate to deposited code. But semantically a smart contract is still prone to errors.*

*More information and code examples [here](https://web3j.readthedocs.io/en/latest/smart_contracts.html)*

### Transactions

You can not use smart contracts without a basic understanding of transactions. Transactions are the payload of blocks. With "[Transferring Ether](#transferring-ether)" you have already gotten to know the first and most common type for transactions. 
Ethereum defines only two more types:

 * Deposit of a new smart contract in the blockchain.  
 * Calling a previously deposited smart contract.  

Committed transactions remain in a *pending state* until they have been embedded into a newly mined block. So apart from the transactions being already integrated into the blockchain there is a dynamic pool of *unpersisted transactions*. These are all transactions that are scheduled for integration but not yet part of a newly mined block. This is why miners keep the chain alive. Though miners extend the blockchain for the sake of being rewarded, their actual contribution is the integration of pending transactions into the blockchain. 

### Workflow

Developing and using a smart contracts always follows the same pattern.

 * [Implement](#implementation-of-the-smart-contract-logic)  
 * [Compile](#compilation-of-solidity-code-to-binaries)  
 * [Generate Java wrappers](#generation-of-a-java-wrapper-for-smart-contract-binaries)  
 * [Deploy](#deployment-of-a-smart-contract)  
 * [Call](#interact-with-a-smart-contract)  

The following subsections illustrate the process on the example of a Java Dapp that first deposits a smart contract "mirror". It is a minimal smart contract that merely offers a single function "reflect". When reflect is called it simply returns the received input string. After the deployment the Dapp calls the contract with ```Hello, smart-world!``` and prints the return value. Complete sources of a demo implementation can be found [here](https://kartoffelquadrat.eu:5050/maex/smart-contract-demo). 

*Note: As this guide aims at Java Dapp development, the used directory layout matches a [maven-project structure](https://maven.apache.org/guides/getting-started/).*

#### Implementation of the smart contract logic

Smart contracts are coded, using the [Solidity](https://solidity.readthedocs.io/en/v0.4.25/introduction-to-smart-contracts.html) language. No need for an IDE to do this, just open [your favorite text editor](https://www.vim.org/).

This is simple example of a smart contract named ```mirror``` with a single function ```reflect```. As the name suggests, the function simply returns its input back to the caller.

```solidity
    pragma solidity ^0.4.17;
    contract mortal {
        address owner;
        constructor() public { owner = msg.sender; }
        function kill() public { if (msg.sender == owner) selfdestruct(owner); }
    }
        
    // contract with a single function that simply returns the input argument back to the caller
    contract mirrorcontract is mortal {
        function reflect(string _input) public pure returns (string) {
     	    return _input;
        }
    }  
```

*See [here](https://solidity.readthedocs.io/en/v0.4.25/introduction-to-smart-contracts.html) for more information on the Solidity language and its syntax.*

Default relative location of smart contract source file in a Java-Dapp:  

```bash
    .
    ├── README.md
    ├── contracts
    │   └── solidity
    │       └── Mirror.sol	<--- Place your contract here.
    ├── pom.xml
    ├── src
    │   └── ...
    └── target
        └── ...
```


#### Compilation of solidity code to binaries

Smart contracts are executed by the [Ethereum Virtual Machine](https://www.mycryptopedia.com/ethereum-virtual-machine-explained/) (EVM). However, the EVM can not interpret solidity sources. You have to compile your contract into binaries first.

 * ```cd``` into your projects ```solidity``` directory.  

 * Compile your smart contract:  
```bash
    solc Mirror.sol --bin --abi --optimize -o ../build
```
This will generate and fill a ```build``` directory.

By now your project structure should resemble:

```bash
    .
    ├── README.md
    ├── contracts
    │   ├── build              <--- This directory was generated (contract binaries)
    │   │   ├── mirrorcontract.abi
    │   │   ├── mirrorcontract.bin
    │   │   ├── mortal.abi
    │   │   └── mortal.bin
    │   └── solidity
    │       └── Mirror.sol	
    ├── pom.xml
    ├── src
    │   └── ...
    └── target
        └── ...
```

#### Generation of a Java wrapper for smart contract binaries

Java does not know how to handle these binaries. But you can embed them into java wrappers. Wrappers are not coded manually, but generated from the binaries, using the ```web3j``` command-line tool.

 * ```cd``` into your projects ```build``` directory.  
 * Generate the wrappers with:  

```bash
    web3j solidity generate ./mirrorcontract.bin ./mirrorcontract.abi -p fr.insa.drim.schieder.etherdemo.hellosmartworld -o ../../src/main/java/
```

*Note: Don't forget to adapt the _bin and _abi file names, as well as your package name.*

You will find a new java class in your sources:   
```bash
    .
    ├── README.md
    ├── contracts
    │   └── ...
    ├── pom.xml
    ├── src
    │   ├── main
    │   │   ├── java
    │   │   │   └── fr
    │   │   │       └── insa
    │   │   │           └── drim
    │   │   │               └── schieder
    │   │   │                   └── etherdemo
    │   │   │                       └── hellosmartworld
    │   │   │                           ├── ...
    │   │   │                           └── Mirrorcontract.java <--- This file is the generated wrapper
    │   │   └── resources
    │   └── test
    │       └── ...
    └── target
        └── ...
```

If you inspect the content of that file you will actually find the smart contract binary in a final String named ```BINARY```.

*Note: Never edit the content of generated files.*

#### Deployment of a smart contract

The java code for deploying a smart contract in the blockchain is similar to transferring ether:
 
* You connect to a node.  
* You authenticate into an account.  
* You effectuate a transaction.  

As you see only the third line differs from the [previous java snippet](#java-access):  

```java
    // Connect to local node
    Web3j web3 = Web3j.build(new HttpService());  // defaults to http://localhost:8545/

    // Load credentials for accessing wallet of source account
    Credentials credentials = WalletUtils.loadCredentials(SOURCE_ACCOUNT_PASSWORD, LOCATION_SOURCE_ACCOUNT);

    // Deploy the contract in the blockchain
    Mirrorcontract.deploy(web3, credentials, DefaultGasProvider.GAS_PRICE, DefaultGasProvider.GAS_LIMIT).send();
```

*Note: The last line returns a Mirrorcontract instance that you can use for further interactions.*  

Once more you can track the effect of your transaction in geths logs:  
```bash
    INFO [09-26|17:46:20.708] Setting new local account                address=0x3c7b081d3e608525A2efB821e80d597Cef7a435c
    INFO [09-26|17:46:20.708] Submitted contract creation              fullhash=0xf6e29728627250aace189814f4bb069ecd4a9d4cc38aa040f0a17e7a8a37ccfc contract=0x6b557c7a68C2e9919c2Be1c3B4772e792b6F44E9
```

### Interact with a smart contract

Web3j abstracts the entire asynchronous communication with geth away from you and even simulated a synchronous behavior. This way you can interact with the smart contract as if it were operating on a local object.

To interact with a smart contract, you can do simple method call on the contract-object you received upon deployment:

```java
    // Will send a string to the mirror-contract, living on the blockchain, 
    // wait for the EVM to run the code, then print the contract's reply to
    // console.
    System.out.println(mirrorContract.reflect("Hello, smart-world!")
```

## Clusters

So far all instructions were scoped on single-machine setups. However blockchains are distributed technologies and gain their primary advantages (integrity, availability) from the fact that they can be spread out and operated within large networks.  
This section indicates how to set up new chains in multi-machine networks.

*More information on clusters and their setup [here](https://github.com/ethereum/go-ethereum/wiki/Setting-up-private-network-or-local-cluster), [here](https://ethereum.stackexchange.com/questions/13547/how-to-set-up-a-private-network-and-connect-peers-in-geth), [here](http://iotbl.blogspot.com/2017/03/setting-up-private-ethereum-testnet.html) and [here](https://ethereum.stackexchange.com/questions/12436/how-to-communicate-with-a-remote-node).*

### Physical link

Before starting with whatsoever P2P configuration, ensure your network configuration is working on a [physical layer](https://en.wikipedia.org/wiki/Physical_layer).

 * Ensure machines have unique IPs  
 * Ensure machines are in the same subnet  
 * Ensure dedicated ports are open, no port used twice 
 * Test ICMP connection  

Further illustrations refer to the following two-machine demo setup:
```
    +---------------------+                      +---------------------+
    |      MACHINE I      |                      |      MACHINE II     | 
    +=====================+                      +=====================+
    | Host: "Chartreuse"  |                      | Host: "Coral"       |
    | Dev: eth0           |                      | Dev: eth0           |
    | IP: 192.168.6.1/24  +========-RJ45-========+ IP: 192.168.6.2/24  |
    | Port: 30301         |                      | Port: 30302         |
    | OS: Mac Os 10.13.6  |                      | OS: Gentoo 4.12.11  |
    +---------------------+                      +---------------------+
```

### Common genesis

The most important thing when setting up a new blockchain, is using the **exact** same genesis block on all machines. Semantic equivalence is not sufficient, the files must be bitwise identical.  
Best practice is to write the file on one machine and then transfer it to the other over the network.

 * [Write a genesis json file](#genesis-block) on *Machine I*. Make sure to use a unique chain id.

 * Transfer the file to *Machine II*
```bash
    scp genesis.json schieder@192.168.6.2:/tmp
```  

 * Verify the hashes are identical.  
```bash
    # Mac:
    md5 genesis.json

    # Linux:
    md5sum genesis.json
```
 * Init a new blockchain on **both*** machines, using that genesis file.
```bash
    geth --datadir="nameOfYourChainGoesHere" init shared-genesis.json
```

### Enodes

Next you need to synchronize your machines. This can be achieved with enodes. An enode is a unique address that serves as entry point to a running ethereum node.

#### Lookup

To query the enode address of *Machine I*, launch a geth console for your initialized blockchain. Parameters are a little different for a P2P setup.

```bash
 geth --datadir="nameOfYourChainGoesHere" --networkid 1608199012345 --verbosity=4 --ipcdisable --port 30301 --rpcport 8101 --rpcaddr 192.168.6.1 --netrestrict 192.168.6.0/24 console
```

*Note: Adapt datadir, networkid, port, rpcport, rpcaddr to your setup.*

Quick overview of parameter changes, compared to single machine setup:  
 * ```--verbosity=4```: Low number means less output, higher more. 4 is a reasonable setting for getting problems printed without having your entire screen flooded with logs.
 * Remove ```--nodiscover``` -> you want to use P2P functionality, right ;-)
 * ```--ipcdisable``` -> no need for interprocess communication, we want the nodes to interact over LAN
 *  ```--port 303XY```, where X is count of your hexanome and Y the count of your PCs (so first pc is 1, second 2 , ...) -> reason is we must not have two machines with the same port for the P2P to work.
 *  ```--rpcport 81XY```, same reason (remote procedure calls need a unique target)
 * ```--rpcaddr 192.168.a.b``` , matching the physical address of your ethernet card.
 *  ```--netrestrict 192.168.10.0/24``` because we want to start a P2P net only in the LAN, noit share it with the entire world.


Once geth is running, type:
```bash
> admin.nodeInfo.enode  
"enode://175cb35c728eb654608a22117f59851c4c45cd7eaddeab3b3af4a0694a3389ee3e6e12d009bb20da7188987ea00ac3c79a040879559000d6c53ef81cb0df4b51@[::]:30301?discport=0"
```

*Note: The substring ```[::]``` at the end substitutes the localhost address ```127.0.0.1```. If you want to use the enode address from another machine, it has to be replaced by Machine I's IP address: ```192.168.6.1```.*

#### Connect

On *Machine II*, launch geth slightly modified arguments::

```bash
    geth --datadir="sharedchain" --networkid 1608199012345 --verbosity=4 --ipcdisable --port 30302 --rpcport 8102 --rpcaddr 192.168.10.2 --netrestrict 192.168.10.0/24 console
```

*Note: Adapt datadir, networkid, port, rpcport, rpcaddr according to  your setup.*

Next connect to the enode running on *machine I*:  
```bash
    > admin.addPeer("enode://175cb35c728eb654608a22117f59851c4c45cd7eaddeab3b3af4a0694a3389ee3e6e12d009bb20da7188987ea00ac3c79a040879559000d6c53ef81cb0df4b51@192.168.5.1:30301?discport=0")
```

Verify the logs of *machine I*. You should see something like:  
```bash
    DEBUG[09-19|13:51:54] Ethereum peer connected                  id=175cb35c728eb654 conn=staticdial name=Geth/v1.8.14-stable/gentoo-amd64/go1.10.3
```

Also on *machine II* verify if the enode of *machine I* is listed as peer:
```bash
    > admin.peers
    [{
        caps: ["eth/62", "eth/63"],
        id: "175cb35c728eb654608a22117f59851c4c45cd7eaddeab3b3af4a0694a3389ee3e6e12d009bb20da7188987ea00ac3c79a040879559000d6c53ef81cb0df4b51",
        name: "Geth/v1.8.14-stable/darwin-amd64/go1.10.3",
        network: {
          localAddress: "192.168.6.2:30302",
          remoteAddress: "192.168.6.1:30301"
        },
        protocols: {
            eth: {
                difficulty: 3825280,
                head: "0x96127b538059d60a5b757c5da074e05d510d60f6cfb001f79dfd23fad9296351",
                version: 63
            }
        }
}]
```

## Summing up

Congratulations, you have made it through this guide and achieved the following competences:

 * Running geth nodes
 * Creating accounts and private chains
 * Setting up decentralized local blockchain networks
 * Coding smrt contracts
 * Programmatically interacting with the blockchain.

As such you are now ready to develop your first Dapp. 
