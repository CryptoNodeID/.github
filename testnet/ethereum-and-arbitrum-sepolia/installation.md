# Installation

## 🖥️System Requirements

\- OS: Linux\
\- CPU: 6 Core(s)\
\- Memory: 24GB\
\- Storage: 2TB

You can get your server here : ![🖥](https://web.telegram.org/a/blank.8dd283bceccca95a48d8.png)[Click Here to Rent VPS from €4.50/month](https://www.dpbolvw.net/bi103shqnhp465665C69E46ABE79DB?sid=CNID-R)

## 💭Prerequisites

Have Docker installed in your system, if you don't, please follow [this guide](../../basics/server-preparation.md#install-docker-optional)

## 🛠️Installation <a href="#install-binary" id="install-binary"></a>

{% hint style="success" %}
This guide is created under the assumption you are using Ubuntu 22.04 LTS\
If you use other OS, please modify the commands accordingly
{% endhint %}

### ⟠ Ethereum

#### Clone our GitHub repo

```sh
git clone https://github.com/CryptoNodeID/sepolia-eth.git
cd sepolia-eth
```

#### Prepare the directories

```sh
mkdir -p $HOME/sepolia-eth/geth-data
mkdir -p $HOME/sepolia-eth/prysm-data
mkdir -p $HOME/sepolia-eth/jwt
```

#### Prepare .env file to prevent permission issue

```bash
echo "UID=$(id -u)" >> .env
echo "GID=$(id -g)" >> .env
```

#### Run ETH Sepolia Docker

```sh
docker compose up -d
```

#### Check Log

```sh
docker compose logs -fn 100
```

{% hint style="success" %}
Based on our setup, your ETH Sepolia RPC can be accessed via port 22545

Try with this command:

{% code overflow="wrap" %}
```sh
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' -H "Content-Type: application/json" http://127.0.0.1:22545
```
{% endcode %}
{% endhint %}

Notable Ports:

* 22545 -> RPC endpoint
* 22546 -> WS endpoint
* 22551 -> Auth endpoint
* 22500 -> Beacon endpoint

### ⟠ Arbitrum

{% hint style="warning" %}
Before preparing Arbitrum, better to wait until ETH node completed the sync
{% endhint %}

#### Clone our GitHub repo

```sh
git clone https://github.com/CryptoNodeID/sepolia-arb.git
cd sepolia-eth
```

#### Prepare the directories

```sh
mkdir -p $home/sepolia-arb/arb-data
```

#### Prepare .env file to prevent permission issue

```bash
echo "UID=$(id -u)" >> .env
echo "GID=$(id -g)" >> .env
```

#### Run ETH Sepolia Docker

```sh
docker compose up -d
```

#### Check Log

```sh
docker compose logs -fn 100
```

{% hint style="success" %}
Based on our setup, your ARB Sepolia RPC can be accessed via port 22557

Try with this command:

{% code overflow="wrap" %}
```sh
curl -X POST --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}' -H "Content-Type: application/json" http://127.0.0.1:22557
```
{% endcode %}
{% endhint %}

Notable Ports:

* 22557 -> RPC endpoint
* 22558 -> WS endpoint
