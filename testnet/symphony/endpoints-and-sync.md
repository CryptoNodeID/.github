# Endpoints and Sync

RPC: [https://symphony-testnet-rpc.cryptonode.id](https://symphony-testnet-rpc.cryptonode.id)

API: [https://symphony-testnet-api.cryptonode.id](https://symphony-testnet-api.cryptonode.id)



{% code title="Peers" %}
```sh
a495c12eac37c0eb92ae8b2af5bcc7ede5534772@sentry1.cnd.biz.id:23656
```
{% endcode %}

## State Sync

```sh
DAEMON_HOME="${HOME}/.symphonyd"
SERVICE_NAME="symphonyd"

sudo systemctl stop ${SERVICE_NAME}

cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/priv_validator_state.json.backup
cp ${DAEMON_HOME}/config/priv_validator_key.json ${DAEMON_HOME}/priv_validator_key.json.backup

symphonyd tendermint unsafe-reset-all --home ${DAEMON_HOME}

PEERS="a495c12eac37c0eb92ae8b2af5bcc7ede5534772@sentry1.cnd.biz.id:23656"
SNAP_RPC="https://symphony-testnet-rpc.cryptonode.id:443"

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
DAEMON_HOME=$HOME/.symphonyd
SERVICE_NAME=symphonyd
NETWORK=symphony-testnet

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
