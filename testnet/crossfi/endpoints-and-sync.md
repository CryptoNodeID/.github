# Endpoints and Sync

RPC: [https://crossfi-testnet-rpc.cryptonode.id](https://crossfi-testnet-rpc.cryptonode.id)

API: [https://crossfi-testnet-api.cryptonode.id](https://crossfi-testnet-api.cryptonode.id)&#x20;

JSON-RPC: [https://crossfi-testnet-jsonrpc.cryptonode.id](https://crossfi-testnet-jsonrpc.cryptonode.id)&#x20;

{% code title="Peers" %}
```sh
d5bd5ea9c3fab054d6cd1fee92fc3ac79827f391@crossfi-testnet-peer.cryptonode.id:20656
```
{% endcode %}

## State Sync

```sh
DAEMON_HOME=${HOME}/.crossfi-testnet
SERVICE_NAME=crossfi-testnet

sudo systemctl stop ${SERVICE_NAME}

cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/data/priv_validator_state.json.backup
cp ${DAEMON_HOME}/config/priv_validator_key.json ${DAEMON_HOME}/config/priv_validator_key.json.backup

crossfid tendermint unsafe-reset-all --home ${DAEMON_HOME}

PEERS="d5bd5ea9c3fab054d6cd1fee92fc3ac79827f391@crossfi-testnet-peer.cryptonode.id:20656,66bdf53ec0c2ceeefd9a4c29d7f7926e136f114a@crossfi-testnet-peer.itrocket.net:36656,4b6c13b8820fd6c1bcf5e36c3097a1b64e4e3b8c@testnet-crossfi.konsortech.xyz:11656"
SNAP_RPC="https://crossfi-testnet-rpc.cryptonode.id:443"

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ${DAEMON_HOME}/config/config.toml 
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height);
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000));
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

## Snapshot Restore

```sh
DAEMON_HOME=$HOME/.crossfi-testnet
SERVICE_NAME=crossfi-testnet
NETWORK=crossfi-testnet

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
