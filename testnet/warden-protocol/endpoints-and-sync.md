# Endpoints and Sync

RPC: [https://warden-testnet-rpc.cryptonode.id](https://warden-testnet-rpc.cryptonode.id)

API: [https://warden-testnet-api.cryptonode.id](https://warden-testnet-api.cryptonode.id)



{% code title="Peers" %}
```sh
895efd33011b12c75d75e83382329cc510fb0464@warden-testnet-peer.cryptonode.id:24656
```
{% endcode %}

## State Sync

```sh
DAEMON_HOME="${HOME}/.warden"
SERVICE_NAME="wardend"

sudo systemctl stop ${SERVICE_NAME}

cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/data/priv_validator_state.json.backup
cp ${DAEMON_HOME}/config/priv_validator_key.json ${DAEMON_HOME}/config/priv_validator_key.json.backup

wardend tendermint unsafe-reset-all --home ${DAEMON_HOME}

PEERS="895efd33011b12c75d75e83382329cc510fb0464@warden-testnet-peer.cryptonode.id:24656,ddb4d92ab6eba8363bab2f3a0d7fa7a970ae437f@sentry-1.buenavista.wardenprotocol.org:26656,c717995fd56dcf0056ed835e489788af4ffd8fe8@sentry-2.buenavista.wardenprotocol.org:26656,e1c61de5d437f35a715ac94b88ec62c482edc166@sentry-3.buenavista.wardenprotocol.org:26656,b14f35c07c1b2e58c4a1c1727c89a5933739eeea@warden-testnet-peer.itrocket.net:18656,00c0b45d650def885fcbcc0f86ca515eceede537@testnet-warden.konsortech.xyz:15656,61446070887838944c455cb713a7770b41f35ac5@37.60.249.101:26656,0be8cf6de2a01a6dc7adb29a801722fe4d061455@65.109.115.100:27060,8288657cb2ba075f600911685670517d18f54f3b@65.108.231.124:18656,dc0122e37c203dec43306430a1f1879650653479@37.27.97.16:26656,6fb5cf2179ca9dd98ababd1c8d29878b2021c5c3@146.19.24.175:26856"
SNAP_RPC="https://warden-testnet-rpc.cryptonode.id:443"

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
DAEMON_HOME=$HOME/.warden
SERVICE_NAME=wardend
NETWORK=warden-testnet

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
