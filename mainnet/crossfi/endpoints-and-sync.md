# Endpoints and Sync

RPC: [https://crossfi-mainnet-rpc.cryptonode.id](https://crossfi-mainnet-rpc.cryptonode.id)

API: [https://crossfi-mainnet-api.cryptonode.id](https://crossfi-mainnet-api.cryptonode.id)

{% code title="Peers" %}
```sh
1985bfd22cf1baeb5aaae6b40b13632c28303bd1@sentry2.cryptonode.id:30656
```
{% endcode %}

## State Sync

```sh
DAEMON_HOME="${HOME}/.mineplex-chain"
SERVICE_NAME="crossfi-mainnet"

sudo systemctl stop ${SERVICE_NAME}

cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/data/priv_validator_state.json.backup
cp ${DAEMON_HOME}/config/priv_validator_key.json ${DAEMON_HOME}/config/priv_validator_key.json.backup

crossfid tendermint unsafe-reset-all --home ${DAEMON_HOME}

PEERS="44ca9e03a9c016ce148d19e877a8652662bbd71b@crossfi-mainnet-peer.cryptonode.id:30656,94bd757a9f002e5ec72f1f7d0a1f96eec0d49f1b@3.95.243.5:30656,3f3d80c93d3af57ff3ca5dfe45ba2b523fa8f056@mainnet-crossfi.konsortech.xyz:16656,2c8951227c667c8833e2930bc07ce1a9f0acbe28@seed-v2.mineplex.io:30656,e9fd5cca36b36d6646cfa65ff72b2f22abec4667@46.101.138.73:30656"
SNAP_RPC="https://crossfi-mainnet-rpc.cryptonode.id:443"

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
sudo journalctl -fu ${SERVICE_NAME}
```

## Snapshot Restore

```sh
DAEMON_HOME=$HOME/.crossfi-mainnet
SERVICE_NAME=crossfi-mainnet
NETWORK=crossfi-mainnet

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
