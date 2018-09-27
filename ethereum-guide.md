# Hands-On   E-T-H-E-R-E-U-M    Guide
By [Maximilian Schiedermeier](https://liris.cnrs.fr/page-membre/maximilian-schiedermeier),  
[Email](mailto:maximilian.schiedermeier@insa-lyon.fr), [Public Encryption Key](https://pgp.mit.edu/pks/lookup?op=vindex&search=0x7010339C4391A4F3)

*Note: This file is under [Attribution-NonCommercial-ShareAlike 4.0 International](https://creativecommons.org/licenses/by-nc-sa/4.0/) License.*

## Machine Setup

### Utils

For building an Ethereum based Java-Dapp (Distributed Application), you need three things:

 * [Geth](https://github.com/ethereum/go-ethereum/wiki/geth): Command line interface for running a full ethereum node.
 * [Web3J](https://web3j.io/): Java library for integrating with nodes on Ethereum blockchains.
 * [Solidity](https://solidity.readthedocs.io/en/v0.4.25/): Language for implementing smart contracts, designed to target the Ethereum Virtual Machine.

### Installation

#### Linux

*Note: The ```emerge -a``` commands are for gentoo. Ubuntu users, click the links below for equivalent ```apt-get install``` commands.*

 * Install geth:  

```bash
    emerge -a net-p2p/go-ethereum
```

*[Ubuntu equivalent](https://github.com/ethereum/go-ethereum/wiki/Installation-Instructions-for-Ubuntu)*

 * Download Web3J, extract and create terminal reference:  

```bash
    wget https://github.com/web3j/web3j/releases/download/v3.5.0/web3j-3.5.0.zip  
    unzip web3j-3.5.0.zip  
    rm web3j-3.5.0.zip  
    echo '#!/bin/bash' >> ~/bin/web3j  
    echo '~/web3j-3.5.0/bin/web3j "$@"' >> ~/bin/web3j  
    chmod +x ~/bin/web3j
```

*Note: Make sure the ~/bin/ directory is in your [PATH](https://opensource.com/article/17/6/set-path-linux).*


 * Install solidity:  

```bash
    emerge -a dev-lang/solidity
```

*[Ubuntu equivalent](https://solidity.readthedocs.io/en/v0.4.21/installing-solidity.html)*

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
*-Same as Linux instructions above-*

 * Install solidity:  

```bash
    brew install solidity  
    brew linkapps solidity  
```

#### Windows

 * Erase your hard drive
 * Install [Ubuntu](https://builtvisible.com/the-ubuntu-installation-guide/)

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
*Note: When working on the main chain this step is obsolete. The main chain genesis block is hard coded into geth.*  

Navigate to your *ethereum base directory* and create a file named ```genesis.json```.  
Paste the following content into that file. **C&P is not enough, you still have to modify the content afterwards!**

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

Essentially, this description nails down the characteristics of your new custom blockchain. However in a network, there must not be two different chains using the same *chainId*. To avoid that, edit the *chainId* value and fill in a number that is sufficiently high and unique, e.g. your birthday + 12345, then save the file.

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

Every intercation with a blockchain is effectuated on beahlf of an account. So in order to do anything with your newly created blockchain you now need at least one account.  
**Hint:**
By default all geth commands are bound to the default chain. This involves the creation of new accounts. You can override this behaviour by providing a path to your own chain. Use the ```--datadir /some/path/``` switch. You can also use relative paths. A good practive is to ```cd``` into your *ethereum base dir* and then only provide the name of your chain as parameter.

### Querying

At any given moment you can list the available accounts, using:  

        geth account list  
        WARN [04-11|14:19:13] No etherbase set and no accounts found as default  

As you have not yet added an account you will see the above warning. So lets see how to add a new account  

### Creation

The ```geth account new``` command adds a new account. When prompted enter the same password twice:  

```bash
    geth account new  
    Your new account is locked with a password. Please give a password. Do not forget this password.  
    Passphrase:  
    Repeat passphrase:  
    Address: {67b7b218cb53611612e2983cf2fa45ce01540cf9}  
```

The last line indicates the unique address of your new account. When later transfering ether between accounts we need this address. This address is also referred to as the coinbase.  
If you now check the content of your ~/.ethereum directory the newly created account will show up there.

        schieder@chartreuse:.ethereum $ tree  
        .  
        --- keystore  
        ------ UTC--2018-04-12T11-56-57.442014523Z--67b7b218cb53611612e2983cf2fa45ce01540cf9  

Of course you do not need to inspect the file system to find your new account you can also retrieve a list of accounts using:  

        geth account list  
        Account #0: {564a2f909e604268d865654086a36f19ac915b07} keystore:///home/schieder/.ethereum/keystore/UTC--2018-04-11T13-03-56.222342097Z--564a2f909e604268d865654086a36f19ac915b07
        

## Balance

*Additional information and commands realted to mining can be found [here](https://github.com/ethereum/go-ethereum/wiki/mining) and [here](https://github.com/turboprep/geth-tutorial).*

Each account has an amount of ether attached. This amount can be considered as *funds*. Stakeholders who want to deposit something into the blockchain must pay a tiny amount of ether to do so.  
Without ether, accounts are very limited in their actions.

### Querrying

Querrying is the act of resolving the amount of ether attached to an account, regarding the blockchains most recent state. To look that amount up, first open a geth console, using:

        geth --nodiscover console  

*Note: By nature ```geth``` is designed for P2P scenarios. Unless you specify otherwise, geth will look out for other ethereum nodes and accept incoming connections. The following examples are altogether commands where you operate on your own local chain and explicitly do not want any P2P functionality. Tell ```geth``` to not interact with other nodes, by launching it using the ```--nodiscover``` option.*

*Note: When launching geth with the ```console``` argument you effectively start a new inner console. To kill a running command, use ```Ctr-C```. ```Ctr-D``` brings you back to the parent bash.*

To then print the current balance in *Wei*, type:  

        eth.getBalance("0x...hashofyouraccounthere")  

If you prefer a direct conversion into *Ether*, wrap your command with a call to web3j.  

```javascript
    web3.fromWei(eth.getBalance(eth.coinbase), "ether")  
```

*Note: 1.000.000.000.000.000.000 Wei = 1 Ether*

By now you have not done any mining. Therefore you will see a sad output of ```0``` either way.

### Mining

Mining is the most straighforward way to obtain ether. In essential it means to invest computational power to keep the blockchain alive. Appending the chain by new blocks is nothing a puzzle. The miner who first resolves it is immediately rewarded by a single ether.

You have two options for mining. 

 * Open a geth console and type ```miner.start()```

 * Launch geth with the ```--mine``` option.  

#### DAG generation

The first time you start to mine, geth will build a [DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph) representation of the chain in the RAM. As your private chain consists of nothing but a genesis block, this goes super fast. If later you start mining on huge public chains this step will take some minutes.  

Throughout the DAG generation you will repeatedly see some log: ```Generating DAG in progress```

#### Reward

Whenever your miner found a block, you will see the following:  

```bash
    Commit new mining work  
    Successfully sealed new block  
```

This implicitly means one ether was added to your account.

#### Multi-Cores

By default geth mines with only a single CPU core. (You can argue about [mining on the main chain with CPU power](https://bitcoin.stackexchange.com/questions/5608/how-does-one-calculate-the-profitability-of-gpu-mining), but on a private chain this is totaly reasonable.)  
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

### Transfering Ether

Another way to boost an accounts balance is to transfere ether from another account. Before actually transfering the ether, the sending account must be unlocked. In the following example the first account at index ```0``` was unlocked for 15 seconds, using the password ```abc123```. The seconds command sends ```42``` ether from account at index ```0``` to account at index ```1```.

```bash
    > web3.personal.unlockAccount(web3.personal.listAccounts[0],"abc123", 15)
    true
    > eth.sendTransaction({from: eth.accounts[0], to: eth.accounts[1], value: web3.toWei(42, "ether")})
    INFO [09-26|09:52:24.639] Setting new local account                address=0x3c7b081d3e608525A2efB821e80d597Cef7a435c
    INFO [09-26|09:52:24.640] Submitted transaction                    fullhash=0x08da052064eb4ffb20396cfef0f83e2ba09c932db26a3788f770638ea1200754 recipient=0xd929a310108EA93d2D82DF4A74763eD1316F94Fb
    "0x08da052064eb4ffb20396cfef0f83e2ba09c932db26a3788f770638ea1200754"
```

As you remember querrying the balance only tells the amount of ether based on the *current blockchain state*. For your transfer to be visible in the account listing, it still has to "arrive" in the blockchain. For that to happen it must still be "wrapped" by a new block. As you will read [a little later](#transactions), transfering ether is one of three possible *transaction* types. From the moment of their registration, transactions first remain in a pending state where they are only queued for persistence. It is only when a new block is mined that they are eventually integrated into the blockchain. Precisely they are the payload of that new block.  
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

*Note: Instead of specifying source and target account by ```eth.accounts[x]``` you can always use hash-adresses wrapped in ticks: ```'0x3c7b081d3e608525a2efb821e80d597cef7a435c'```*



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

This far all described blockchain interactions were effectuated manually. This means whenever you accessed the blockchain, you did so used either the ```geth``` command or its built-in console. But geth can also expose its functionality and work as a proxy. The two most outstanding ways to achieve this are:  

 * Using *RPCs* (Remote Procedure Calls)  
Geth acts as a proxy on a given port and ip. Can be seen as the more generic one, for it allows accessing a remote node over a network connection.

 * Using *IPC* (Inter Process Communication)
Geth opens a unix socket for other processes running on the same machine. There are ways to later expose the socket over the network, but usually this method is only used for local access.

This guide covers *RPC* access, only.

### Preparations

Before using ```geth```s *RPC* functionality from a high-level programming language, you must ensure that...

 * ... you have access to an account with some ether.
 * ... you have access to an *RPC-enabled* geth node.

#### Have an account with some ether

By now you should already have this covered. If not, create your [private blockchain](#chains), create an [account](#accounts), then [mine](#mining) until you have about 20 ether. 

#### Run an HTTP-enabled geth node

As *RPC* is a network-protocol, you could send your HTTP commands to any availbale *RPC*-enabled node. Unfortunately in the context of a private local chain there are none around. You have to operate your own node for all proxied access. Remember, this node must keep running throughout the entire execution of your program.  

```bash
    geth --rpcapi personal,db,eth,net,web3 --rpc --mine --minerthreads=4 console  
```

The ```--rpcapi``` and  ```--rpc``` arguments advise ```geth``` is to power up the *RPC interface*.  

For the exact meaning of the remaining parameters, see [here](https://ethereum.stackexchange.com/questions/41112/what-do-these-geth-rpcapi-parameters-mean-db-and-net) and [here](https://github.com/ethereum/go-ethereum/wiki/Management-APIs)  

*Note: As mentioned in the ["Transfering Ether"](#transfering-ether) section, a blockchain without miners is "dead". If you forget the ```--mine``` option, your Dapp not be able to effectuate actions that internally require [transactions](#transactions). If you use synchronous API-calls it will even block.*


#### Test accessing your node

You can easily test access to your node using ```curl```. More information [here](https://github.com/ethereum/wiki/wiki/JSON-RPC#eth_getbalance).

```bash
    curl -H "Content-Type: application/json" -X POST --data '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0x3c7b081d3e608525a2efb821e80d597cef7a435c", "latest"],"id":1}' http://localhost:8545 
```

*Note: Reply to the above balance request is the amount in Wei in [hexadecimal format](https://www.rapidtables.com/convert/number/hex-to-decimal.html).*

### Java Access

This section demonstrates the minimal setup towards a Java driven blockchain access..

#### Dependencies

Though vanilla Java provides utilities for HTTP communication, we will use the ```web3j``` library to abstract this layer away from our code. 

Easiest way is to set up your java project to use a [build automation tool](https://en.wikipedia.org/wiki/Build_automation) that resolves the dependencies for you. In your IDE, [create a new maven project](https://www.youtube.com/watch?v=pt3uB0sd5kY). To then integrate ```web3j``` into your project, add the following dependency to your ```pom.xml```: 

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

*Everything you need to know any many more code examples [here](https://web3j.readthedocs.io/en/latest/smart_contracts.html)*

A smart contract is a program that resides in the blockchain. We therefore hold strong guarantees for its integrity and can trust into the correctness of generated outputs.

*Note: "Correctness" applies from a security point of view, here. We stipulate that computed outputs correlate to deposited code. But semantically a smart contract is still prone to errors.*

### Transactions

You can not use smart contracts without a basic understanding of transactions. Transactions are the payload of blocks. With "[Transfering Ether](#transfering-ether)" you have already gotten to know the first and most common type for transactions. 
Ethereum defines only two more types:

 * Deposit of a new smart contract in the blockchain.  
 * Calling a previously deposited smart contract.  

Commited transactions remain in a *pending state* until they have been embedded into a newly mined block. So apart from the transactions being already integrated into the blockchain there also is a dynamic pool of *unpersisted transactions*. This is why miners keep the chain alive. Though miners extend the blockchain for the sake of being rewarded, their actual contribution is the embedding of pending transactions into the blockchain. 

### The procedure

Depositing and using a SC runs in 5 stages:

 * Code a smart contract in solitude.
 * Compile the solitude code into binaries.
 * Wrap the binaries in Java.
 * Make a transaction that persists the binaries in the blockchain.
 * Use further transaction s to interact with the deposited smart contract.

### Coding a smart contract

This is simple example of a solidity code that receives a string and buffers it until replaced by a subsequent call.

        pragma solidity ^0.4.17;
        contract mortal {
          address owner;
          function mortal() public { owner = msg.sender; }
          function kill() public { if (msg.sender == owner) selfdestruct(owner); }
        }

        contract buffer is mortal {
          string buffer;
          // constructor
          function accumulator(string _initbuffer) public {
            buffer = _initbuffer;
          }

          // getter
          function greet(string _addendum) public constant returns (string) {
            buffer = _addendum;
            return buffer;
          }
        }  

### Compiling a smart contract

Solidity code by itsel is neither executable by the EVM (ethereum virtual machine) not usable for your web3J-java project. Integration runs in two stages:

 * Compile the solidity code into EJM binaries.

Presuming that you have an Intellij-wbe3J project at ```~/IdeaProjects/web3Jdemo/``` you should place your solidity code at ```~/IdeaProjects/web3Jdemo/contracts/solidity/MyContract.sol``` 

Next ```cd``` into that directory then compile your contract. Make sure the output directory is set to ```../build```  

```bash
    solc MyContract.sol --bin --abi --optimize -o ../build
```

This will generate four files:  (Correlates to the ```contract``` signatures of your solitude file.)

 * ```../build/mortal.bin```  
 * ```../build/mortal.abi```  
 * ```../build/buffer.bin```  
 * ```../build/buffer.abi```  

**TODO: get rid of warnings at compilation**
**TODO: adapt command and filenames (buffer - helloworld)**

### Build JAVA wrappers

Do not code the java wrappers yourself. Web3J handles that for you. If not yet done, [download the web3J command line tools](https://github.com/web3j/web3j/releases/tag/v1.0.4).  
```cd``` into the directory with the genereated binary blobs:  
```cd ~/IdeaProjects/web3Jdemo/contracts/build/```  

Then generate the Java wrappers, using:  

```bash
    web3j solidity generate ./buffer.bin ./buffer.abi -p your.funny.package.goes.here -o ../../src/main/java/
```  

Your IDE should then display a generated java class in the specified pacakge. If you inspect the file you will find the binary blobs integrated. This a line that looks alike:  

```java
    private static final String BINARY = "6060604052341561000f57600080fd5b6040516103203803806103208339810160405280805160008054600160a060020a03191633600160a060020a03161790559190910190506001818051610059929160200190610060565b50506100fb565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106100a157805160ff19168380011785556100ce565b828001600101855582156100ce579182015b828111156100ce5782518255916020019190600101906100b3565b506100da9291506100de565b5090565b6100f891905b808211156100da57600081556001016100e4565b90565b6102168061010a6000396000f30060606040526004361061004b5763ffffffff7c010000000000000000000000000000000000000000000000000000000060003504166341c0e1b58114610050578063cfae321714610065575b600080fd5b341561005b57600080fd5b6100636100ef565b005b341561007057600080fd5b610078610130565b60405160208082528190810183818151815260200191508051906020019080838360005b838110156100b457808201518382015260200161009c565b50505050905090810190601f1680156100e15780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b6000543373ffffffffffffffffffffffffffffffffffffffff9081169116141561012e5760005473ffffffffffffffffffffffffffffffffffffffff16ff5b565b6101386101d8565b60018054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156101ce5780601f106101a3576101008083540402835291602001916101ce565b820191906000526020600020905b8154815290600101906020018083116101b157829003601f168201915b5050505050905090565b602060405190810160405260008152905600a165627a7a72305820634445fe931ac7fa310264a5cbf9085974d4af61c1d3215e4e090c23a994529d0029";
```

### Persist a smart contract

#### Old school

How to do it with geth

#### Programmatic

Saving the SC in the chain is straightforward. Load ```web3j``` and some valid account credentials (of the person who submits the SC) into local variables, then persist using the SCs ```deploy(...)``` method.

```java
        // Load web3j
        Web3j web3 = Web3j.build(new HttpService());  // defaults to http://localhost:8545/

        // Load credentials:
        Credentials credentials = WalletUtils.loadCredentials(
          "accountpassword",
          "/home/yourname/.ethereum/keystore/UTC--2018-04-12T12-55-28.874090257Z--ac3fbbf1b4649104f44377b3c33f401c9ac3f8df");
          
        // Persist the contract, using web3j and the credentials
        Buffer contract = Buffer.deploy(
          web3, credentials, GAS_PRICE, GAS_LIMIT,
          "Hello blockchain world!").send();
```

When executing the code you geth node will immediately log the persistence of your contract:

```bash
    INFO [09-26|17:46:20.708] Setting new local account                address=0x3c7b081d3e608525A2efB821e80d597Cef7a435c
    INFO [09-26|17:46:20.708] Submitted contract creation              fullhash=0xf6e29728627250aace189814f4bb069ecd4a9d4cc38aa040f0a17e7a8a37ccfc contract=0x6b557c7a68C2e9919c2Be1c3B4772e792b6F44E9
```

### Interact with a smart contract

Two components:

 * Sending information / calling SC
 * Receiving reply

Finally you want to send values to the SC and receive return values (just as you would use any other function).  
Note: Though the procedure described below suggest a synchronous characteristic, it is actually not. In a first transaction the params get passed to the SC. The SC then by iself writes a second tansaction with the result to the BC. It is merely web3j that hides this complexity from you and emulates a ```synchronous``` execution. That is to say with the code below, calling and receiving will block until some miners have actually eternalized the related transactions in the chain.

#### Sending

To send params you use the function you defined in your solitude file, e.g. ```greet("SomeStringAsParam")```. The actual call then looks like this:  
```contract.buffer("SomeStringAsParam").send();```  
Note: ```contract``` is the reference you rceived upon persistence in the BC. ```send()``` is the explicit call to persist this parameter-passing transaction in the BC.

#### Receiving

Receiving is even simplet. It merely is the return value of the call that passed a value to the SC. So to recover a String from the buffer SC, you would write:  
```String theReturnValue = contract.buffer().send("SomeInputValue");```  

In case you want to access the contract from another agent you need to load it first from the BC. To do so, use it's unique address.  
It has to be exposed by the entity that deposited the SC in the chain:
```contractInstance.getContractAddress();```  
The string returned by this call, looks like:  
```0xda87ac931d237a2fe1ef8e45abf79244c69e6a06```  
Use this sting in the second agent to load the contract:  
```GeneratedContractClass.load("0xda87ac931d237a2fe1ef8e45abf79244c69e6a06", web3, credentials, GAS_PRICE, GAS_LIMIT);```

## Clusters

## C L U S T E R S...
##Network setup:
##[Actually two options: P2P, every machine own node, and RPC, only one mode, accessed remotely]
##1) create A SINGLE private chain, genesis block etc...
##-> if no "--datadir" is provided this instatiates in ~/Library/Ethereum/geth...  otherwise it replicates structure in specified dir:
##toto/
├── geth
│   ├── chaindata
│   │   ├── 000001.log
│   │   ├── CURRENT
│   │   ├── LOCK
│   │   ├── LOG
│   │   └── MANIFEST-000000
│   └── lightchaindata
│       ├── 000001.log
│       ├── CURRENT
│       ├── LOCK
│       ├── LOG
│       └── MANIFEST-000000
└── keystore"
##2) create private network that will SHARE this BC.
##geth --datadir /Users/maex/Desktop/toto/ --networkid 1608199012345
##networkid SHOULD be something unique to not conflict with existing networks -> irrelevant in LAN (?)
##3) Access the blockchain:
##3.1) same machine: (IPC, inter process communication)
##> personal.newAccount()
##Passphrase:
##Repeat passphrase:
##"0x2e9d394a1c51568bab2ff61eb073bbe13415aadf"
##> eth.getBalance("0x2e9d394a1c51568bab2ff61eb073bbe13415aadf")
##mine:
##> miner.start()
##...wait until DAG complete, something commited.
##> miner.stop()
##> eth.getBalance("0x2e9d394a1c51568bab2ff61eb073bbe13415aadf")
3.2) from remote machine (RPC, remote procedure calls)
-> CLuster: https://github.com/ethereum/go-ethereum/wiki/Setting-up-private-network-or-local-cluster
https://ethereum.stackexchange.com/questions/12436/how-to-communicate-with-a-remote-node
Power up: (unsure if this works?)
geth --datadir /Users/maex/Desktop/toto/ --networkid 1608199012345 --rpc --rpcport "8545" --rpcaddr "0.0.0.0" --rpccorsdomain "*"
https://ethereum.stackexchange.com/questions/13547/how-to-set-up-a-private-network-and-connect-peers-in-geth
http://iotbl.blogspot.com/2017/03/setting-up-private-ethereum-testnet.html
https://github.com/ethereum/go-ethereum/wiki/Setting-up-private-network-or-local-clusterGENESIS: -> shared chain can be any string (!), chain id can be any big random number!
MAKE SURE TO USE THE SAME CONFIG FOR YOUR GETH COMMANDS! (chainname, id)
WARNING II -> the genesis file has to be the exact same file, by means of hashing - semantically identical but differently son files DO NOT WORK!
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
INIT:
Mac: cd Library/Ethereum/
Linux: cd .ethereum
        geth --datadir="sharedchain" init shared-genesis.json
Expected structure:
maex@Chartreuse:Ethereum $ tree sharedchain/
sharedchain/
├── geth
│   ├── chaindata
│   │   ├── 000002.log
│   │   ├── CURRENT
│   │   ├── CURRENT.bak
│   │   ├── LOCK
│   │   ├── LOG
│   │   └── MANIFEST-000003
│   └── lightchaindata
│       ├── 000001.log
│       ├── CURRENT
│       ├── LOCK
│       ├── LOG
│       └── MANIFEST-000000
└── keystore
One ONE client (and only one!), start geth in console mode, create account, obtain some ether:
geth --datadir="sharedchain" --networkid 1608199012345 --nodiscover console
> personal.newAccount("maxou1")
"0xed0ae7db863e9ce9afa43f028b596f4af06b20ad"
> miner.start(1)
Check balance, then abort geth
> web3.fromWei(eth.getBalance(eth.coinbase), "ether")
_NOTE: tree command should now list your account hash in the "keystore" of YOUR chain (sharedchain)!
└── sharedchain
    ├── geth
    │   ├── LOCK
    │   ├── chaindata
    │   │   ├── 000004.ldb
    │   │   ├── 000007.ldb
    │   │   ├── 000008.log
    │   │   ├── CURRENT
    │   │   ├── CURRENT.bak
    │   │   ├── LOCK
    │   │   ├── LOG
    │   │   └── MANIFEST-000009
    │   ├── lightchaindata
    │   │   ├── 000001.log
    │   │   ├── CURRENT
    │   │   ├── LOCK
    │   │   ├── LOG
    │   │   └── MANIFEST-000000
    │   ├── nodekey
    │   └── transactions.rlp
    ├── history
    └── keystore
FIND OUT ENODE ID OF YOUR RUNNING GETH -> P2p handshake entry point (other nodes can attach to this.)
> admin.nodeInfo.enode
"enode://175cb35c728eb654608a22117f59851c4c45cd7eaddeab3b3af4a0694a3389ee3e6e12d009bb20da7188987ea00ac3c79a040879559000d6c53ef81cb0df4b51@[::]:30301?discport=0"

NOW GO OFFLINE (you are starting a p2p connection. unless you have a good firewall you should not expose your PC to the entire wollt. Thats why we're using old school cable LAN for out network in class)
Before connecting from other geth entity -> restart geth with following changes:
add "--verbosity=4" Lowe number means less output, higher more. 4 is a reasonable setting for getting problems printed without having your entire screen flooded with logs.
remove "--nodiscover" -> you want to use P2P functionality, right ;-)
add "--ipcdisable" -> no need for interprocess communication, we want the nodes to interact over LAN
add "--port 303XY", where X is count of your hexanome and Y the count of your PCs (so first pc is 1, second 2 , ...) -> reason is we must not have two machines with the same port for the P2P to work.
add "--rpcport 81XY", same reason (remote procedure calls need a unique target)
add "--rpcaddr 192.168.a.b" , matching the physical address of your ethernet card.
add "--netrestrict 192.168.10.0/24" because we want to start a P2P net only in the LAN, noit share it with the entire world.
so finally you should have (mac, machine 1)
 geth --datadir="sharedchain" --networkid 1608199012345 --verbosity=4 --ipcdisable --port 30301 --rpcport 8101 --rpcaddr 192.168.10.1 --netrestrict 192.168.10.0/24 console
and (linux, machine 2)
 geth --datadir="sharedchain" --networkid 1608199012345 --verbosity=4 --ipcdisable --port 30302 --rpcport 8102 --rpcaddr 192.168.10.2 --netrestrict 192.168.10.0/24 console

Alright last thing you need to do is to sync with an existing node. Therefore we use the enode-id we queried before:
But careful! -> the enode string still contains a self reference "[::]" which is nothing else but "127.0.0.1", the localhost loopback ip. For the node who printed this string it was correct, but if we want to connect from another node it has to be changed for the actual ip. So if the first machine hat 192.168.5.1, we can now connect using the enode id:
"enode://175cb35c728eb654608a22117f59851c4c45cd7eaddeab3b3af4a0694a3389ee3e6e12d009bb20da7188987ea00ac3c79a040879559000d6c53ef81cb0df4b51@192.168.5.1:30301?discport=0"
Simply type:
> admin.addPeer("enode://175cb35c728eb654608a22117f59851c4c45cd7eaddeab3b3af4a0694a3389ee3e6e12d009bb20da7188987ea00ac3c79a040879559000d6c53ef81cb0df4b51@192.168.5.1:30301?discport=0")
Check the log outputs for error / success messages.
Ideal you see something like:
DEBUG[09-19|13:51:54] Ethereum peer connected                  id=175cb35c728eb654 conn=staticdial name=Geth/v1.8.14-stable/darwin-amd64/go1.10.3
This was the output from the linux machine connecting to a geth node on a mac -> "darwin"
Check the output of:
> admin.peers
If it returns an empty list: "[]" it did not work, if you see
[{
    caps: ["eth/62", "eth/63"],
    id: "175cb35c728eb654608a22117f59851c4c45cd7eaddeab3b3af4a0694a3389ee3e6e12d009bb20da7188987ea00ac3c79a040879559000d6c53ef81cb0df4b51",
    name: "Geth/v1.8.14-stable/darwin-amd64/go1.10.3",
    network: {
      localAddress: "192.168.5.2:33094",
      remoteAddress: "192.168.5.1:30301"
    },
    protocols: {
      eth: {
        difficulty: 3825280,
        head: "0x96127b538059d60a5b757c5da074e05d510d60f6cfb001f79dfd23fad9296351",
        version: 63
      }
    }
}]
You're good to go. :-)


mac
geth --datadir="sharedchain" --networkid 1608199012345 --verbosity=3 --ipcdisable --port 30301 --rpcport 8101 --rpcaddr 192.168.5.1 --nodiscover console

lin
geth --datadir="sharedchain" --networkid 1608199012345 --verbosity=3 --ipcdisable --port 30302 --rpcport 8102 --rpcaddr 192.168.5.2 --nodiscover console

FIREWALL SETTINGS???

