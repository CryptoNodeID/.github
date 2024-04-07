# Endpoints and Sync

RPC: [https://crossfi-testnet-rpc.cryptonode.id](https://crossfi-testnet-rpc.cryptonode.id)

API: [https://crossfi-testnet-api.cryptonode.id](https://crossfi-testnet-api.cryptonode.id)&#x20;

JSON-RPC: [https://crossfi-testnet-jsonrpc.cryptonode.id](https://crossfi-testnet-jsonrpc.cryptonode.id)&#x20;

{% code title="Peers" %}
```sh
9a0bacb836102d71abd685651d8edb4b3694224b@crossfi-testnet-peer.cryptonode.id:26656
```
{% endcode %}

## State Sync

```sh
sudo systemctl stop crossfi-testnet

cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/data/priv_validator_state.json.backup
cp ${DAEMON_HOME}/config/priv_validator_key.json ${DAEMON_HOME}/config/priv_validator_key.json.backup

crossfid tendermint unsafe-reset-all --home ${DAEMON_HOME}

PEERS="9a0bacb836102d71abd685651d8edb4b3694224b@crossfi-testnet-peer.cryptonode.id:26656"
SNAP_RPC="https://crossfi-testnet-rpc.cryptonode.id:443"

sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" ${DAEMON_HOME}/config/config.toml 
LATEST_HEIGHT=$(curl -s $SNAP_RPC/block | jq -r .result.block.header.height);
BLOCK_HEIGHT=$((LATEST_HEIGHT - 1000));
TRUST_HASH=$(curl -s "$SNAP_RPC/block?height=$BLOCK_HEIGHT" | jq -r .result.block_id.hash) 

sed -i \
  -e "s|^  enable *=.*|  enable = "true"|" \
  -e "s|^  rpc_servers *=.*|  rpc_servers = \"$SNAP_RPC,$SNAP_RPC\"|" \
  -e "s|^  trust_height *=.*|  trust_height = $BLOCK_HEIGHT|" \
  -e "s|^  trust_hash *=.*|  trust_hash = \"$TRUST_HASH\"|" \
  -e "s|^  seeds *=.*|  seeds = \"\"|" \
  ${DAEMON_HOME}/config/config.toml
  
mv ${DAEMON_HOME}/data/priv_validator_state.json.backup ${DAEMON_HOME}/data/priv_validator_state.json
mv ${DAEMON_HOME}/config/priv_validator_key.json.backup ${DAEMON_HOME}/config/priv_validator_key.json

sudo systemctl restart crossfi-testnet && sudo journalctl -u crossfi-testnet -f
```
