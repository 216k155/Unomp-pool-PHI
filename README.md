#### Unified NOMP Supported PHI Algo
====================================

#### Requirements
-----------------

#### 1) Downloading & Installing

Clone the repository and run `npm update` for all the dependencies to be installed:

```bash
sudo apt-get install build-essential libssl-dev npm nodejs nodejs-legacy
curl https://raw.githubusercontent.com/creationix/nvm/v0.16.1/install.sh | sh
source ~/.profile
nvm install 0.10.25
nvm use 0.10.25
git clone https://github.com/216k155/Unomp-pool-PHI.git phi-pool
cd unomp
npm update
```

#### 2) Configuration

##### Portal config
Inside the `config_example.json` file, ensure the default configuration will work for your environment, then copy the file to `config.json`.

Explanation for each field:
````javascript
{
    /* Specifies the level of log output verbosity. Anything more severe than the level specified
       will also be logged. */
    "logLevel": "debug", //or "warning", "error"
    
    /* By default UNOMP logs to console and gives pretty colors. If you direct that output to a
       log file then disable this feature to avoid nasty characters in your log file. */
    "logColors": true, 


    /* The UNOMP CLI (command-line interface) will listen for commands on this port. For example,
       blocknotify messages are sent to UNOMP through this. */
    "cliPort": 17117,

    /* By default 'forks' is set to "auto" which will spawn one process/fork/worker for each CPU
       core in your system. Each of these workers will run a separate instance of your pool(s),
       and the kernel will load balance miners using these forks. Optionally, the 'forks' field
       can be a number for how many forks will be spawned. */
    "clustering": {
        "enabled": true,
        "forks": "auto"
    },
    
    /* Pool config file will inherit these default values if they are not set. */
    "defaultPoolConfigs": {
    
        /* Poll RPC daemons for new blocks every this many milliseconds. */
        "blockRefreshInterval": 1000,
        
        /* If no new blocks are available for this many seconds update and rebroadcast job. */
        "jobRebroadcastTimeout": 55,
        
        /* Disconnect workers that haven't submitted shares for this many seconds. */
        "connectionTimeout": 600,
        
        /* (For MPOS mode) Store the block hashes for shares that aren't block candidates. */
        "emitInvalidBlockHashes": false,
        
        /* This option will only authenticate miners using an address or mining key. */
        "validateWorkerUsername": true,
        
        /* Enable for client IP addresses to be detected when using a load balancer with TCP
           proxy protocol enabled, such as HAProxy with 'send-proxy' param:
           http://haproxy.1wt.eu/download/1.5/doc/configuration.txt */
        "tcpProxyProtocol": false,
        
        /* If under low-diff share attack we can ban their IP to reduce system/network load. If
           running behind HAProxy be sure to enable 'tcpProxyProtocol', otherwise you'll end up
           banning your own IP address (and therefore all workers). */
        "banning": {
            "enabled": true,
            "time": 600, //How many seconds to ban worker for
            "invalidPercent": 50, //What percent of invalid shares triggers ban
            "checkThreshold": 500, //Perform check when this many shares have been submitted
            "purgeInterval": 300 //Every this many seconds clear out the list of old bans
        },
        
        /* Used for storing share and block submission data and payment processing. */
        "redis": {
            "host": "127.0.0.1",
            "port": 6379,
            "db": 0, /* redis db select, usefull for multi-node cluster replicas */
	    "password": ""  /* "" - no password, or any non blank "redispassword" to enable it */
        }
    },

    /* This is the front-end. Its not finished. When it is finished, this comment will say so. */
    "website": {
        "enabled": true,
        /* If you are using a reverse-proxy like nginx to display the website then set this to
           127.0.0.1 to not expose the port. */
        "host": "0.0.0.0",
        /* Title you want for your site. */
        "siteTitle": "UNOMP Beta",
        "port": 8080,
        /* Used for displaying stratum connection data on the Getting Started page. */
        "stratumHost": "pool.unomp.org",
        "stats": {
            /* Gather stats to broadcast to page viewers and store in redis for historical stats
               every this many seconds. */
            "updateInterval": 15,
            /* How many seconds to hold onto historical stats. Currently set to 24 hours. */
            "historicalRetention": 43200,
            /* How many seconds worth of shares should be gathered to generate hashrate. */
            "hashrateWindow": 300
        },
        /* Not done yet. */
        "adminCenter": {
            "enabled": true,
            "password": "password"
        }
    },

    /* Redis instance of where to store global portal data such as historical stats, proxy states,
       ect.. */
    "redis": {
        "host": "127.0.0.1",
        "port": 6379,
        "db": 0, /* redis db select, usefull for multi-node cluster replicas */
        "password": ""  /* similar "" - no password, or any non blank  "redispassword" to enable it */
    },


    /* With this switching configuration, you can setup ports that accept miners for work based on
       a specific algorithm instead of a specific coin. Miners that connect to these ports are
       automatically switched a coin determined by the server. The default coin is the first
       configured pool for each algorithm and coin switching can be triggered using the
       cli.js script in the scripts folder.  */
       
    "switching": {
        "switch1": {
            "enabled": false,
            "algorithm": "sha256",
            "ports": {
                "3333": {
                    "diff": 10,
                    "varDiff": {
                        "minDiff": 16,
                        "maxDiff": 512,
                        "targetTime": 15,
                        "retargetTime": 90,
                        "variancePercent": 30
                    }
                }
            }
        },
        "switch2": {
            "enabled": false,
            "algorithm": "scrypt",
            "ports": {
                "4444": {
                    "diff": 10,
                    "varDiff": {
                        "minDiff": 16,
                        "maxDiff": 512,
                        "targetTime": 15,
                        "retargetTime": 90,
                        "variancePercent": 30
                    }
                }
            }
        },
        "switch3": {
            "enabled": false,
            "algorithm": "x11",
            "ports": {
                "5555": {
                    "diff": 0.001
                }
            }
        }
    },

    "profitSwitch": {
        "enabled": false,
        "updateInterval": 600,
        "depth": 0.90,
        "usePoloniex": true,
        "useBittrex": true
    }
}
````


##### Coin config
Inside the `coins` directory, ensure a json file exists for your coin. If it does not you will have to create it.
Here is an example of the required fields:
````javascript
{
    "name": "Litecoin",
    "symbol": "LTC",
    "algorithm": "scrypt",

    /* Magic value only required for setting up p2p block notifications. It is found in the daemon
       source code as the pchMessageStart variable.
       For example, litecoin mainnet magic: http://git.io/Bi8YFw
       And for litecoin testnet magic: http://git.io/NXBYJA */
    "peerMagic": "fbc0b6db", //optional
    "peerMagicTestnet": "fcc1b7dc" //optional

    //"txMessages": false, //options - defaults to false

    //"mposDiffMultiplier": 256, //options - only for x11 coins in mpos mode
}
````

For additional documentation how to configure coins and their different algorithms
see [these instructions](//github.com/sigwo/node-merged-pool#module-usage).


##### Pool config
Take a look at the example json file inside the `pool_configs` directory. Rename it to `yourcoin.json` and change the
example fields to fit your setup.

Description of options:

````javascript
{
    "enabled": true, //Set this to false and a pool will not be created from this config file
    "coin": "litecoin.json", //Reference to coin config file in 'coins' directory
    "auxes": [
        {
            "coin": "viacoin.json",
            "daemons": [
                {
                    "host": "127.0.0.1",
                    "port": 4001,
                    "user": "user",
                    "password": "password"
                }
            ]
        }
    ],
    "address": "mi4iBXbBsydtcc5yFmsff2zCFVX4XG7qJc", //Address to where block rewards are given

    /* Block rewards go to the configured pool wallet address to later be paid out to miners,
       except for a percentage that can go to, for examples, pool operator(s) as pool fees or
       or to donations address. Addresses or hashed public keys can be used. Here is an example
       of rewards going to the main pool op and a pool co-owner. Can also be set for mandatory 
       donation coins like GRE and DMD. */
    "rewardRecipients": {
        "n37vuNFkXfk15uFnGoVyHZ6PYQxppD3QqK": 1.5, //1.5% goes to pool op
        "mirj3LtZxbSTharhtXvotqtJXUY7ki5qfx": 0.5 //0.5% goes to a pool co-owner
    },

    "paymentProcessing": {
        "enabled": true,

        /* Every this many seconds get submitted blocks from redis, use daemon RPC to check
           their confirmation status, if confirmed then get shares from redis that contributed
           to block and send out payments. */
        "paymentInterval": 30,

        /* Minimum number of coins that a miner must earn before sending payment. Typically,
           a higher minimum means less transactions fees (you profit more) but miners see
           payments less frequently (they dislike). Opposite for a lower minimum payment. */
        "minimumPayment": 0.01,

        /* This daemon is used to send out payments. It MUST be for the daemon that owns the
           configured 'address' that receives the block rewards, otherwise the daemon will not
           be able to confirm blocks or send out payments. */
        "daemon": {
            "host": "127.0.0.1",
            "port": 4000,
            "user": "testuser",
            "password": "testpass"
        }
    },

    /* Each pool can have as many ports for your miners to connect to as you wish. Each port can
       be configured to use its own pool difficulty and variable difficulty settings. varDiff is
       optional and will only be used for the ports you configure it for. */
    "ports": {
        "3032": { //A port for your miners to connect to
            "diff": 32, //the pool difficulty for this port

            /* Variable difficulty is a feature that will automatically adjust difficulty for
               individual miners based on their hashrate in order to lower networking overhead */
            "varDiff": {
                "minDiff": 8, //Minimum difficulty
                "maxDiff": 512, //Network difficulty will be used if it is lower than this
                "targetTime": 15, //Try to get 1 share per this many seconds
                "retargetTime": 90, //Check to see if we should retarget every this many seconds
                "variancePercent": 30 //Allow time to very this % from target without retargeting
            }
        },
        "3256": { //Another port for your miners to connect to, this port does not use varDiff
            "diff": 256 //The pool difficulty
        }
    },

    /* More than one daemon instances can be setup in case one drops out-of-sync or dies. */
    "daemons": [
        {   //Main daemon instance
            "host": "127.0.0.1",
            "port": 4000,
            "user": "testuser",
            "password": "testpass"
        }
    ],

    /* This allows the pool to connect to the daemon as a node peer to receive block updates.
       It may be the most efficient way to get block updates (faster than polling, less
       intensive than blocknotify script). It requires the additional field "peerMagic" in
       the coin config. */
    "p2p": {
        "enabled": false,

        /* Host for daemon */
        "host": "127.0.0.1",

        /* Port configured for daemon (this is the actual peer port not RPC port) */
        "port": 19333,

        /* If your coin daemon is new enough (i.e. not a shitcoin) then it will support a p2p
           feature that prevents the daemon from spamming our peer node with unnecessary
           transaction data. Assume its supported but if you have problems try disabling it. */
        "disableTransactions": true
    }
}

````

You can create as many of these pool config files as you want (such as one pool per coin you which to operate).
If you are creating multiple pools, ensure that they have unique stratum ports.

For more information on these configuration options see the [pool module documentation](https://github.com/UNOMP/node-merged-pool#module-usage)



##### [Optional, recommended] Setting up blocknotify
1. In `config.json` set the port and password for `blockNotifyListener`
2. In your daemon conf file set the `blocknotify` command to use:
```
node [path to cli.js] [coin name in config] [block hash symbol]
```
Example: inside `dogecoin.conf` add the line
```
blocknotify=node /home/unomp/scripts/cli.js blocknotify dogecoin %s
```

Alternatively, you can use a more efficient block notify script written in pure C. Build and usage instructions
are commented in [scripts/blocknotify.c](scripts/blocknotify.c).


#### 3) Start the portal

```bash
node init.js
```

###### Optional, highly-recommended enhancements for your awesome new mining pool server setup:
* Use something like [forever](https://github.com/nodejitsu/forever) or [ndt](https://github.com/snailjs/ndt) to keep the node script running
in case the master process crashes.
* Use something like [redis-commander](https://github.com/joeferner/redis-commander) to have a nice GUI
for exploring your redis database.
* Use something like [logrotator](http://www.thegeekstuff.com/2010/07/logrotate-examples/) to rotate log 
output from UNOMP.
* Use [New Relic](http://newrelic.com/) to monitor your UNOMP instance and server performance.


#### Upgrading
When updating UNOMP to the latest code its important to not only `git pull` the latest from this repo, but to also update
the `merged-pooler` and `node-multi-hashing` modules, and any config files that may have been changed.
* Inside your UNOMP directory (where the init.js script is) do `git pull` to get the latest UNOMP code.
* Remove the dependenices by deleting the `node_modules` directory with `rm -r node_modules`.
* Run `npm update` to force updating/reinstalling of the dependencies.
* Compare your `config.json` and `pool_configs/coin.json` configurations to the latest example ones in this repo or the ones in the setup instructions where each config field is explained. You may need to modify or add any new changes.

Donations
---------
Below is my donation address. The original [credits](//github.com/sigwo/unified-node-open-mining-portal/blob/master/CREDITS.md) are listed here because I felt scammy if I totally removed them. They no longer are supporting the current development effort. Please donate to:

* BTC: `19svwpxWAhD4zsfeEnnxExZgnQ46A3mrt3`

Donors (email me to be added):
* [elitemobb from altnuts.com](http://altnuts.com)
* [mike from minerpools.com](https://minerpools.com)
* [gentamicin from themultipool.com](http://themultipool.com)

[Credits](//github.com/sigwo/unified-node-open-mining-portal/blob/master/CREDITS.md)
-------

License
-------
Released under the GNU General Public License v2

http://www.gnu.org/licenses/gpl-2.0.html
