# Installation

## 🖥️System Requirements

\- OS: Linux\
\- CPU: 4 Core(s)\
\- Memory: 8GB\
\- Storage: 300GB

You can get your server here : ![🖥](https://web.telegram.org/a/blank.8dd283bceccca95a48d8.png)[Click Here to Rent VPS from €4.50/month](https://www.dpbolvw.net/bi103shqnhp465665C69E46ABE79DB?sid=CNID-R)

## 🛠️Manual Installation <a href="#install-binary" id="install-binary"></a>

This guide is created under the assumption you are using Ubuntu 22.04 LTS\
If you use other OS, please modify the commands accordingly

### Set environment

```sh
export INSTALLATION_DIR=${HOME}/appl
export DAEMON_NAME=junctiond
export DAEMON_HOME=${HOME}/.junction
export SERVICE_NAME=junctiond-testnet
export MONIKER="YOUR_NODE_NAME_HERE"
export WALLET="YOUR_WALLET_NAME_HERE"
```

DAEMON\_HOME is reference only, you can set  wherever you want.

### Write env to .profile

{% code fullWidth="false" %}
```bash
echo 'export DAEMON_NAME=${DAEMON_NAME}' >> ~/.profile
echo 'export DAEMON_HOME=${DAEMON_HOME}' >> ~/.profile
echo 'export DAEMON_ALLOW_DOWNLOAD_BINARIES=true' >> ~/.profile
echo 'export DAEMON_RESTART_AFTER_UPGRADE=true' >> ~/.profile
echo 'export DAEMON_LOG_BUFFER_SIZE=512' >> ~/.profile

source ~/.profile
```
{% endcode %}

{% hint style="info" %}
This step is optional, but we recommend you to do it for convenience
{% endhint %}

### Install dependencies

Please refer to [server-preparation.md](../../basics/server-preparation.md "mention")

### Prepare installation

```sh
mkdir -p ${INSTALLATION_DIR}/bin
mkdir -p ${DAEMON_HOME}/cosmovisor/genesis/bin
mkdir -p ${DAEMON_HOME}/cosmovisor/upgrades
```

### Install and Setup Warden Daemon

<pre class="language-sh" data-overflow="wrap"><code class="lang-sh">cd ${INSTALLATION_DIR}

#Download Junction Daemon and basic setup
wget https://github.com/airchains-network/junction/releases/download/v0.1.0/junctiond
chmod +x ${DAEMON_NAME}
<strong>mv ${DAEMON_NAME} ${INSTALLATION_DIR}/bin
</strong>
#Download and install cosmovisor
wget https://github.com/cosmos/cosmos-sdk/releases/download/cosmovisor%2Fv1.5.0/cosmovisor-v1.5.0-linux-amd64.tar.gz
tar -xvzf cosmovisor-v1.5.0-linux-amd64.tar.gz

#Copy Binaries
cp cosmovisor ${INSTALLATION_DIR}/bin/cosmovisor
cp ${INSTALLATION_DIR}/bin/${DAEMON_NAME} ${DAEMON_HOME}/cosmovisor/genesis/bin
sudo ln -s ${INSTALLATION_DIR}/bin/cosmovisor /usr/local/bin/cosmovisor -f
sudo ln -s ${DAEMON_HOME}/cosmovisor/genesis ${DAEMON_HOME}/cosmovisor/current -f
sudo ln -s ${DAEMON_HOME}/cosmovisor/current/bin/${DAEMON_NAME} /usr/local/bin/${DAEMON_NAME} -f
</code></pre>

#### Check wardend version

```sh
junctiond --home ${DAEMON_HOME} version
```

### Create or Restore Wallet

```sh
#If you want to create new wallet
junctiond --home ${DAEMON_HOME} keys add ${WALLET}
```

```sh
#If you already have wallet and want to use same phrase
junctiond --home ${DAEMON_HOME} keys add ${WALLET} --recover
```

{% hint style="info" %}
You will be prompted to "Enter your bip39 mnemonic", paste your phrase and press \[ENTER]
{% endhint %}

#### Check your wallet

```sh
junctiond --home ${DAEMON_HOME} keys list
```

### Setup pruning config

```sh
sed -i \
  -e 's|^pruning *=.*|pruning = "custom"|' \
  -e 's|^pruning-keep-recent *=.*|pruning-keep-recent = "100"|' \
  -e 's|^pruning-keep-every *=.*|pruning-keep-every = "0"|' \
  -e 's|^pruning-interval *=.*|pruning-interval = "10"|' \
  ${DAEMON_HOME}/config/app.toml
```

{% hint style="info" %}
The number shown only for reference. \
You can change the number for each parameter as you want
{% endhint %}

### Setting up Cosmovisor

```bash
sudo tee /etc/systemd/system/junctiond-testnet.service > /dev/null <<EOF  
[Unit]
Description=Airchain Testnet Daemon (cosmovisor)
After=network-online.target

[Service]
User=$USER
ExecStart=$(which cosmovisor) run start --home ${DAEMON_HOME}
Restart=always
RestartSec=3
LimitNOFILE=4096
Environment="DAEMON_NAME=${DAEMON_NAME}"
Environment="DAEMON_HOME=${DAEMON_HOME}"
Environment="DAEMON_ALLOW_DOWNLOAD_BINARIES=true"
Environment="DAEMON_RESTART_AFTER_UPGRADE=true"
Environment="DAEMON_LOG_BUFFER_SIZE=512"

[Install]
WantedBy=multi-user.target
EOF
```

#### Enable the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable junctiond-testnet.service
sudo systemctl start junctiond-testnet.service
```

#### Check service log

```bash
sudo journalctl -xfu junctiond-testnet
```

### Cleanup

```bash
rm -f cosmovisor-v1.5.0-linux-amd64.tar.gz
rm -f README.md CHANGELOG.md LICENSE readme.md cosmovisor
```

## ⚙️Automatic Installation

You can visit our repository and follow the instruction there

[https://github.com/CryptoNodeID/airchain](https://github.com/CryptoNodeID/airchain)
