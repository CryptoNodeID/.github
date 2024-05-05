# Endpoints and Sync

RPC: [https://side-testnet-rpc.cryptonode.id](https://side-testnet-rpc.cryptonode.id)

API: [https://side-testnet-api.cryptonode.id ](https://side-testnet-api.cryptonode.id)

{% code title="Peers" %}
```sh
9de5775684739f62e314b178a45db8fb0ab092ba@side-testnet-peer.cryptonode.id:22656
```
{% endcode %}

## State Sync

```sh
DAEMON_HOME=${HOME}/.side
SERVICE_NAME=sided

sudo systemctl stop ${SERVICE_NAME}

cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/data/priv_validator_state.json.backup
cp ${DAEMON_HOME}/config/priv_validator_key.json ${DAEMON_HOME}/config/priv_validator_key.json.backup

sided tendermint unsafe-reset-all --home ${DAEMON_HOME}

PEERS="739b859cd76a5d4f05d868b7bdf3f826bf185e4c@side-testnet-peer.cryptonode.id:22656,bbbf623474e377664673bde3256fc35a36ba0df1@side-testnet-peer.itrocket.net:45656,1ace1dc4d8968f6946d0ede46e6c10b2eefb60bb@65.109.78.52:26656,996c8e0d0c331c19984c543f6a3ec8520131fb7e@95.164.3.79:34656,85cfebdb59615a1bf427106a32b30c91568fd52a@135.181.216.54:3450,351db3719747088c9a980685a7ca40e84e7211e5@65.109.107.172:26656,dc09bc2843f6097d145d79c232d2b2749f1e88ce@37.27.48.77:29656,a435b4f6bd8a3372d5a3a96cb781418c20913391@173.249.59.66:22656,cbd5e3faeead45960c3ce1388454025445f820da@65.108.73.186:26656,9c14080752bdfa33f4624f83cd155e2d3976e303@65.108.231.124:45656"
SNAP_RPC="https://side-testnet-rpc.cryptonode.id:443"

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
DAEMON_HOME=$HOME/.side
SERVICE_NAME=sided
NETWORK=side-testnet

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

