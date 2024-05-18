# Endpoints and Sync

{% tabs %}
{% tab title="RPC" %}
```
https://initia-testnet-rpc.cryptonode.id
```
{% endtab %}
{% endtabs %}

{% code title="Peers" %}
```sh
e3b22ed3046af612cb88140017cb1ac1ea881254@initia-testnet-peer.cryptonode.id:28656,5f934bd7a9d60919ee67968d72405573b7b14ed0@t-seed-initia.dashnode.org:29656,5e3d8e15692bad4dd30ad85ef0116ecf3483de88@initia-testnet-peer.nodem0rt.xyz:19656
```
{% endcode %}

## State Sync

```sh
DAEMON_HOME="${HOME}/.initia"
SERVICE_NAME="initiad"

sudo systemctl stop ${SERVICE_NAME}

cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/data/priv_validator_state.json.backup
cp ${DAEMON_HOME}/config/priv_validator_key.json ${DAEMON_HOME}/config/priv_validator_key.json.backup

initiad tendermint unsafe-reset-all --home ${DAEMON_HOME} --keep-addr-book

PEERS="e3b22ed3046af612cb88140017cb1ac1ea881254@initia-testnet-peer.cryptonode.id:28656,5f934bd7a9d60919ee67968d72405573b7b14ed0@t-seed-initia.dashnode.org:29656,5e3d8e15692bad4dd30ad85ef0116ecf3483de88@initia-testnet-peer.nodem0rt.xyz:19656,a63a6f6eae66b5dce57f5c568cdb0a79923a4e18@peer-initia-testnet.trusted-point.com:26628,aee7083ab11910ba3f1b8126d1b3728f13f54943@initia-testnet-peer.itrocket.net:11656,cd69bcb00a6ecc1ba2b4a3465de4d4dd3e0a3db1@initia-testnet-seed.itrocket.net:51656,b027527aa552c6f292143a2adc61bfa23ecb3896@194.163.170.106:51656,ab948b87097b6474663e0132ac7360676f7030cd@62.169.26.15:26656,49da32b984143181ae5cae6564aba3a150624d7d@194.180.176.225:26656,d1d43cc7c7aef715957289fd96a114ecaa7ba756@testnet-seeds.nodex.one:24510,bbf8ef70a32c3248a30ab10b2bff399e73c6e03c@initia-testnet.rpc.nodex.one:24556"
SNAP_RPC="https://initia-testnet-rpc.cryptonode.id:443"

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ${DAEMON_HOME}/config/config.toml 
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height);
BLOCK_HEIGHT=$((LATEST_HEIGHT - 5000));
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) 

sed -i \
  -e "s|^.*enable *=.*|enable = "true"|" \
  -e "s|^.*rpc_servers *=.*|rpc_servers = \"$SNAP_RPC,$SNAP_RPC\"|" \
  -e "s|^.*trust_height *=.*|trust_height = $BLOCK_HEIGHT|" \
  -e "s|^.*trust_hash *=.*|trust_hash = \"$TRUST_HASH\"|" \
  -e "s|^.*seeds *=.*|seeds = \"\"|" \
  ${DAEMON_HOME}/config/config.toml
  
mv ${DAEMON_HOME}/data/priv_validator_state.json.backup ${DAEMON_HOME}/data/priv_validator_state.json
mv ${DAEMON_HOME}/config/priv_validator_key.json.backup ${DAEMON_HOME}/config/priv_validator_key.json

sudo systemctl restart ${SERVICE_NAME}
sudo journalctl -fu ${SERVICE_NAME} --no-hostname -o cat
```

{% hint style="danger" %}
ONLY USE 1 of the SYNC method ‼️ if you already run [#state-sync](endpoints-and-sync.md#state-sync "mention"), wait for it and check the log, don't run [#snapshot-restore](endpoints-and-sync.md#snapshot-restore "mention").

If you choose to use [#snapshot-restore](endpoints-and-sync.md#snapshot-restore "mention"), you don't need to execute [#state-sync](endpoints-and-sync.md#state-sync "mention")
{% endhint %}

## Snapshot Restore

```sh
DAEMON_HOME=$HOME/.initia
SERVICE_NAME=initiad
NETWORK=initia-testnet

sudo systemctl stop ${SERVICE_NAME}
cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/priv_validator_state.json.backup
rm -rf ${DAEMON_HOME}/data
mkdir -p ${DAEMON_HOME}/data

SNAP_NAME=$(curl -s https://snapshot.cryptonode.id/${NETWORK}/ | egrep -o ">${NETWORK}-snapshot.*\.tar.lz4" | tr -d ">")
curl https://snapshot.cryptonode.id/${NETWORK}/${SNAP_NAME} | lz4 -dc - | tar -xf - -C ${DAEMON_HOME}/data
mv ${DAEMON_HOME}/priv_validator_state.json.backup ${DAEMON_HOME}/data/priv_validator_state.json

sudo systemctl restart ${SERVICE_NAME}
sudo journalctl -fu ${SERVICE_NAME} --no-hostname -o cat
```
