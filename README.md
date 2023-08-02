# Cosmos--Node--Setup-testnet
basic introduction and knowledge on cosmos-sdk/network . A task given to set up a cosmos node on their public network and thus learning about a new platform and implementing the things learnt and showcase it in the best way possible 

providing the necessary instructions for logging into the live public testnet.
we would be discussing the set up of node in testnet in one of three different ways: 

# 1)Build Tools

Install build tools and Go.

sudo apt-get update

sudo apt-get install -y make gcc

wget https://go.dev/dl/go1.20.3.linux-amd64.tar.gz

sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.20.3.linux-amd64.tar.gz

export PATH=$PATH:/usr/local/go/bin

# 2)Install and Configure

You must use the script below to install and set up the Gaia binaries. The version of Gaia running on the Cosmos Hub Public Testnet is 10.0.2 (opens new window).

cd $HOME

git clone https://github.com/cosmos/gaia

cd gaia

-> To sync from genesis, comment out the next line.

git checkout v10.0.2

->To sync from genesis, uncomment the next line and skip the State Sync Setup section.

-git checkout v6.0.4

make install

export PATH=$PATH:$HOME/go/bin

gaiad init custom_moniker

# 3)Prepare genesis file

cd $HOME

wget https://github.com/cosmos/testnets/raw/master/public/genesis.json.gz

gzip -d genesis.json.gz

mv genesis.json $HOME/.gaia/config/genesis.json

-> Set minimum gas price & peers

cd $HOME/.gaia/config

sed -i 's/minimum-gas-prices = ""/minimum-gas-prices = "0.0025uatom"/' app.toml

sed -i 's/seeds = ""/seeds = "639d50339d7045436c756a042906b9a69970913f@seed-01.theta-testnet.polypore.xyz:26656,3e506472683ceb7ed75c1578d092c79785c27857@seed-02.theta-

testnet.polypore.xyz:26656"/' config.toml

# 4) Sync Setup(state)

You must set up a trust height and trust hash in order to use state sync. Depending on when you join the network, these will change depending on the current block height.

In this case, the trust height would be 12,562,000 and the trust hash would be 6F958861E1FA409639C8F2DA899D09B9F50A66DBBD49CE021A2FF680FA8A9204. The block explorer reports the current block height as 12,563,326.

cd $HOME/.gaia/config

sed -i 's/enable = false/enable = true/' config.toml

sed -i 's/trust_height = 0/trust_height = <BLOCK_HEIGHT>/' config.toml

sed -i 's/trust_hash = ""/trust_hash = "<BLOCK_HASH>"/' config.toml

sed -i 's/rpc_servers = ""/rpc_servers = "http:\/\/state-sync-01.theta-testnet.polypore.xyz:26657,http:\/\/state-sync-02.theta-testnet.polypore.xyz:26657"/' config.toml

# 5)Cosmovisor (Setup)

A process manager called Cosmovisor keeps an eye on any incoming chain upgrade requests in the governance module. When a proposal is accepted, Cosmovisor can instantly download the updated binary, switch to it when the chain binary reaches the upgrade height, and then restart the daemon.

.gaia

└── cosmovisor

    └── genesis
    
        └── bin
            
            └── gaiad

go install github.com/cosmos/cosmos-sdk/cosmovisor/cmd/cosmovisor@v1.3.0

mkdir -p ~/.gaia/cosmovisor/genesis/bin

cp ~/go/bin/gaiad ~/.gaia/cosmovisor/genesis/bin/

# 6)service file(creation)

Running gaiad or cosmovisor with the --x-crisis-skip-assert-invariants flag is advised by Cosmos Hub. Operators are likely to notice rounding error when extracting rewards from the validator when verifying for invariants. They are anticipated.

[Unit]

Description=Cosmovisor service

After=network-online.target

[Service]

User=root

ExecStart=/root/go/bin/cosmovisor run start --x-crisis-skip-assert-invariants --home /root/.gaia

Restart=no

LimitNOFILE=4096

Environment='DAEMON_NAME=gaiad'

Environment='DAEMON_HOME=/root/.gaia'

Environment='DAEMON_ALLOW_DOWNLOAD_BINARIES=true'

Environment='DAEMON_RESTART_AFTER_UPGRADE=true'

Environment='DAEMON_LOG_BUFFER_SIZE=512'

Environment='UNSAFE_SKIP_BACKUP=true'

[Install]

WantedBy=multi-user.target

# 7)Start the Service

Reload the systemd manager configuration

systemctl daemon-reload

systemctl restart systemd-journald

for cosmovisor

systemctl enable cosmovisor.service

systemctl start cosmovisor.service

# 8) validator creation and upgrading the node 

A folder with the same name as the upgrade given in the software upgrade proposal will be the first place Cosmovisor looks for the new binary if the environment variable DAEMON_ALLOW_DOWNLOAD_BINARIES is set to false. The expected folder structure for the v11 update would like the following:

gaia

└── cosmovisor

    ├── current
    
    ├── genesis
    
    │   └── bin
    
    |       └── gaiad
    
    └── upgrades
    
        └── v11
        
            └── bin
            
                └── gaiad

Prepare the upgrade directory

Copy

mkdir -p ~/.gaia/cosmovisor/upgrades/v11/bin

Download and install the new binary version.


Copy

cd $HOME/gaia

git pull

git checkout v11.0.0-rc0

make install

# 9)Copy the new binary to the v11 upgrade directory

cp ~/go/bin/gaiad ~/.gaia/cosmovisor/upgrades/v11/bin/gaiad

The gaiad binary will be stopped when the upgrade height is achieved by Cosmovisor, which will then restart after copying the new binary to the current/bin folder. The node should begin syncing blocks with the updated binary within a short period of time.






