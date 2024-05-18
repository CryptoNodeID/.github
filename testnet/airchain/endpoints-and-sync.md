# Endpoints and Sync

RPC: [https://airchain-testnet-rpc.cryptonode.id](https://airchain-testnet-rpc.cryptonode.id)

API: [https://airchain-testnet-api.cryptonode.id](https://airchain-testnet-api.cryptonode.id)



{% code title="Peers" %}
```sh
df2a56a208821492bd3d04dd2e91672657c79325@airchain-testnet-peer.cryptonode.id:27656
```
{% endcode %}

## State Sync

```sh
DAEMON_HOME="${HOME}/.junction"
SERVICE_NAME="junctiond"

sudo systemctl stop ${SERVICE_NAME}

cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/data/priv_validator_state.json.backup
cp ${DAEMON_HOME}/config/priv_validator_key.json ${DAEMON_HOME}/config/priv_validator_key.json.backup

junctiond tendermint unsafe-reset-all --home ${DAEMON_HOME}

PEERS="df2a56a208821492bd3d04dd2e91672657c79325@airchain-testnet-peer.cryptonode.id:27656,aeaf101d54d47f6c99b4755983b64e8504f6132d@airchain-testnet-peer.dashnode.org:28656,47f61921b54a652ca5241e2a7fc4ed8663091e89@airchains-testnet-peer.itrocket.net:19656,04e2fdd6ec8f23729f24245171eaceae5219aa91@airchains-testnet-seed.itrocket.net:19656,2d1ea4833843cc1433e3c44e69e297f357d2d8bd@5.78.118.106:26656"
SNAP_RPC="https://airchain-testnet-rpc.cryptonode.id:443"

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

## Snapshot Restore

```sh
DAEMON_HOME=$HOME/.junction
SERVICE_NAME=junctiond
NETWORK=airchain-testnet

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
