# Endpoints and Sync

{% tabs %}
{% tab title="RPC" %}
```
https://galactica-testnet-rpc.cryptonode.id
```
{% endtab %}

{% tab title="API" %}
```
https://galactica-testnet-api.cryptonode.id
```
{% endtab %}

{% tab title="JSON-RPC" %}
```
https://galactica-testnet-jsonrpc.cryptonode.id
```
{% endtab %}
{% endtabs %}

{% code title="Peers" %}
```sh
cd3e3746b83a33c857a97be4986f16425e03b880@galactica-testnet-peer.cryptonode.id:25656
```
{% endcode %}

## State Sync

```sh
DAEMON_HOME="${HOME}/.galactica"
SERVICE_NAME="galacticad"

sudo systemctl stop ${SERVICE_NAME}

cp ${DAEMON_HOME}/data/priv_validator_state.json ${DAEMON_HOME}/data/priv_validator_state.json.backup
cp ${DAEMON_HOME}/config/priv_validator_key.json ${DAEMON_HOME}/config/priv_validator_key.json.backup

galacticad tendermint unsafe-reset-all --home ${DAEMON_HOME}

PEERS="cd3e3746b83a33c857a97be4986f16425e03b880@galactica-testnet-peer.cryptonode.id:25656,c9993c738bec6a10cfb8bb30ac4e9ae6f8286a5b@galactica-testnet-peer.itrocket.net:11656,f3cd6b6ebf8376e17e630266348672517aca006a@46.4.5.45:27456,6b846b316d704d78f3f9ca46d86cebc5a22de8ae@161.97.111.249:26656,232050092a90b94b002f34a79de8a3859ed13ee7@5.104.108.184:26656,9990ab130eac92a2ed1c3d668e9a1c6e811e8f35@148.251.177.108:27456,c722e6dc5f762b0ef19be7f8cc8fd67cdf988946@49.12.96.14:26656,e926f2e20568e61646558a2b8fd4a4e46d77903f@141.95.110.124:26656,3afb7974589e431293a370d10f4dcdb73fa96e9b@157.90.158.222:26656,dc4ed6e614725dffc41874e762a1b1ce4cdc3304@161.97.74.20:46656,e38c22e44893e75e388f3bcea2a075124d75ccd3@89.110.89.244:26656,8949fb771f2859248bf8b315b6f2934107f1cf5a@168.119.241.1:26656"
SNAP_RPC="https://galactica-testnet-rpc.cryptonode.id:443"

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
DAEMON_HOME=$HOME/.galactica
SERVICE_NAME=galacticad
NETWORK=galactica-testnet

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
