# Installation

### Update system & install prerequisites. <a href="#update-system-and-install-prerequisites" id="update-system-and-install-prerequisites"></a>

Copy

```
sudo apt update
sudo apt upgrade
sudo apt install -y curl git jq lz4 build-essential unzip
bash <(curl -s "https://raw.githubusercontent.com/MANTRA-Finance/public/main/go_install.sh")
source .bash_profile
```

### Install binary. <a href="#install-binary" id="install-binary"></a>

Copy

```
mkdir bin
cd bin
source ~/.profile

wget https://github.com/MANTRA-Finance/public/raw/main/mantrachain-hongbai/mantrachaind-linux-amd64.zip
unzip mantrachaind-linux-amd64.zip
rm -rf mantrachaind-linux-amd64.zip
```

### Install cosmwasm library <a href="#install-cosmwasm-library" id="install-cosmwasm-library"></a>

More information can be found in here: [https://github.com/CosmWasm/wasmvm](https://github.com/CosmWasm/wasmvm)

Copy

```
sudo wget -P /usr/lib https://github.com/CosmWasm/wasmvm/releases/download/v1.3.1/libwasmvm.x86_64.so
```

### Initialise node <a href="#initialise-node" id="initialise-node"></a>

Copy

```
mantrachaind init <your-moniker> --chain-id mantra-hongbai-1
```

* where `<your-moniker>`is the moniker and can be any valid string name (e.g. `val-01`),
* and `mantra-hongbai-1` is the Chain ID and must be exactly as specified.

### Download "genesis.json" - HONGBAI <a href="#download-genesis.json-hongbai" id="download-genesis.json-hongbai"></a>

Obtain the `genesis.json` for the MANTRA Hongbai Chain (Testnet).

Copy

```
curl -Ls https://github.com/MANTRA-Finance/public/raw/main/mantrachain-hongbai/genesis.json > $HOME/.mantrachain/config/genesis.json
```

Update the `config.toml` with the seed node and the peers for the MANTRA Hongbai Chain (Testnet).

Copy

```
CONFIG_TOML="$HOME/.mantrachain/config/config.toml"
SEEDS="d6016af7cb20cf1905bd61468f6a61decb3fd7c0@34.72.142.50:26656"
PEERS="da061f404690c5b6b19dd85d40fefde1fecf406c@34.68.19.19:26656,20db08acbcac9b7114839e63539da2802b848982@34.72.148.3:26656"
sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $CONFIG_TOML
sed -i.bak -e "s/^seeds =.*/seeds = \"$SEEDS\"/" $CONFIG_TOML
external_address=$(wget -qO- eth0.me)
sed -i.bak -e "s/^external_address *=.*/external_address = \"$external_address:26656\"/" $CONFIG_TOML
sed -i 's|^minimum-gas-prices *=.*|minimum-gas-prices = "0.0002uom"|g' $CONFIG_TOML
sed -i 's|^prometheus *=.*|prometheus = true|' $CONFIG_TOML
sed -i -e "s/^filter_peers *=.*/filter_peers = \"true\"/" $CONFIG_TOML
```

### Install Cosmovisor <a href="#install-cosmovisor" id="install-cosmovisor"></a>

`cosmovisor` is a process manager for Cosmos SDK application binaries that automates application binary switch at chain upgrades.

It polls the `upgrade-info.json` file that is created by the x/upgrade module at upgrade height, and then can automatically download the new binary, stop the current binary, switch from the old binary to the new one, and finally restart the node with the new binary.

Copy

```
go install cosmossdk.io/tools/cosmovisor/cmd/cosmovisor@v1.4.0
mkdir -p ~/.mantrachain/cosmovisor/genesis/bin
mkdir -p ~/.mantrachain/cosmovisor/upgrades
cp ~/bin/mantrachaind ~/.mantrachain/cosmovisor/genesis/bin
```

### Configure "mantrachaind" as a Service <a href="#configure-mantrachaind-as-a-service" id="configure-mantrachaind-as-a-service"></a>

The `systemd` service manager allows the `mantrachaind` binary to run as a service, instead of as a command-line application. (See [https://systemd.io](https://systemd.io/) for more information.)

Copy

```
sudo tee /etc/systemd/system/mantrachaind.service > /dev/null << EOF
[Unit]
Description=Mantra Node
After=network-online.target
[Service]
User=$USER
ExecStart=$(which cosmovisor) run start
Restart=on-failure
RestartSec=3
LimitNOFILE=10000
Environment="DAEMON_NAME=mantrachaind"
Environment="DAEMON_HOME=$HOME/.mantrachain"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=false"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="UNSAFE_SKIP_BACKUP=true"
[Install]
WantedBy=multi-user.target
EOF
```

### Starting, stoping and restarting service <a href="#starting-stoping-and-restarting-service" id="starting-stoping-and-restarting-service"></a>

Copy

```
#reload, enable and start
sudo systemctl daemon-reload
sudo systemctl enable mantrachaind
sudo systemctl start mantrachaind

#stop
sudo systemctl stop mantrachaind

#restart
sudo systemctl restart mantrachaind

#logs
sudo journalctl -xefu mantrachaind

#logs - filtered on block height lines
sudo journalctl -xefu mantrachaind -g ".*txindex"
```

Once started, the node will take some time to sync with the blockchain.

Visit [https://explorer.hongbai.mantrachain.io/mantrachain](https://explorer.hongbai.mantrachain.io/mantrachain) to see the current height of the blockchain. Use the `journalctl` command to check on the node's progress.

## ⚙️Automatic Installation

You can visit our repository and follow the instruction there

[https://github.com/CryptoNodeID/mantrachain](https://github.com/CryptoNodeID/mantrachain)
