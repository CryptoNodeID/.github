# Endpoints and Sync

RPC: [https://warden-testnet-rpc.cryptonode.id](https://warden-testnet-rpc.cryptonode.id)

API: [https://warden-testnet-api.cryptonode.id](https://warden-testnet-api.cryptonode.id)



{% code title="Peers" %}
```sh
f1e730c741c7edb89e4610e2f24993c5ca2e028b@sentry1.cryptonode.id:22656,66ac611ba87753e92f1e5d792a2b19d4c5080f32@sentry2.cryptonode.id:22656
```
{% endcode %}

## State Sync

```sh
DAEMON_HOME="${HOME}/.empe-chain"
SERVICE_NAME="emped"

sudo systemctl stop ${SERVICE_NAME}

cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/priv_validator_state.json.backup
cp ${DAEMON_HOME}/config/priv_validator_key.json ${DAEMON_HOME}/priv_validator_key.json.backup

emped tendermint unsafe-reset-all --home ${DAEMON_HOME}

PEERS="f1e730c741c7edb89e4610e2f24993c5ca2e028b@sentry1.cryptonode.id:22656,66ac611ba87753e92f1e5d792a2b19d4c5080f32@sentry2.cryptonode.id:22656,edfc10bbf28b5052658b3b8b901d7d0fc25812a0@193.70.45.145:26656,4bd60dee1cb81cb544f545589b8dd286a7b3fd65@149.202.73.140:26656,149383fab60d8845c408dce7bb93c05aa1fd115e@54.37.80.141:26656"
SNAP_RPC="https://empe-testnet-rpc.cryptonode.id:443"

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ${DAEMON_HOME}/config/config.toml 
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height);
LATEST_HEIGHT=$(echo "$LATEST_HEIGHT" | awk '{printf "%d000\n", $0 / 1000}')
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000));
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) 

sed -i \
  -e "s|^.*enable *=.*|enable = "true"|" \
  -e "s|^.*rpc_servers *=.*|rpc_servers = \"$SNAP_RPC,$SNAP_RPC\"|" \
  -e "s|^.*trust_height *=.*|trust_height = $BLOCK_HEIGHT|" \
  -e "s|^.*trust_hash *=.*|trust_hash = \"$TRUST_HASH\"|" \
  -e "s|^.*seeds *=.*|seeds = \"\"|" \
  ${DAEMON_HOME}/config/config.toml
  
mv ${DAEMON_HOME}/priv_validator_state.json.backup ${DAEMON_HOME}/data/priv_validator_state.json
mv ${DAEMON_HOME}/priv_validator_key.json.backup ${DAEMON_HOME}/config/priv_validator_key.json

sudo systemctl restart ${SERVICE_NAME}
sudo journalctl -fu ${SERVICE_NAME}
```

## Snapshot Restore

```sh
DAEMON_HOME=$HOME/.empe-chain
SERVICE_NAME=emped
NETWORK=empe-testnet

sudo systemctl stop ${SERVICE_NAME}
cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/priv_validator_state.json.backup
rm -rf ${DAEMON_HOME}/data
mkdir -p ${DAEMON_HOME}/data

SNAP_NAME=$(curl -s https://snapshot.cryptonode.id/${NETWORK}/ | egrep -o ">${NETWORK}-snapshot.*\.tar.lz4" | tr -d ">")
curl https://snapshot.cryptonode.id/${NETWORK}/${SNAP_NAME} | lz4 -dc - | tar -xf - -C ${DAEMON_HOME}/data
mv ${DAEMON_HOME}/priv_validator_state.json.backup ${DAEMON_HOME}/data/priv_validator_state.json

sudo systemctl restart ${SERVICE_NAME}
sudo journalctl -fu ${SERVICE_NAME}
```
