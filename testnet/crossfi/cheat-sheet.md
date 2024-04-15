# Cheat Sheet

If you use our auto-installation, we've included several scripts under `./scripts` folder like:

```sh
./list_keys_testnet.sh
./check_balance_testnet.sh
./create_validator_testnet.sh
./unjail_validator_testnet.sh
./check_validator_testnet.sh
./start_crossfi_testnet.sh
./check_log_testnet.sh
```

Here's other useful commands you might need

## ⚙️Service operations <a href="#service-operations" id="service-operations"></a>

{% tabs %}
{% tab title="Check logs" %}
```sh
sudo journalctl -fu crossfi-testnet
```
{% endtab %}

{% tab title="Reload daemon" %}
```bash
sudo systemctl daemon-reload
```
{% endtab %}

{% tab title="Enable service" %}
```sh
sudo systemctl enable crossfi-testnet
```
{% endtab %}

{% tab title="Disable service" %}
```sh
sudo systemctl disable crossfi-testnet
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Start service" %}
```sh
sudo systemctl start crossfi-testnet
```
{% endtab %}

{% tab title="Stop service" %}
```sh
sudo systemctl stop crossfi-testnet
```
{% endtab %}

{% tab title="Restart service" %}
```sh
sudo systemctl status crossfi-testnet
```
{% endtab %}

{% tab title="Check service status" %}
```sh
sudo systemctl status crossfi-testnet
```
{% endtab %}
{% endtabs %}

## 🖥️Node operations

### Node info

```
crossfid status 2>&1 | jq .NodeInfo
```

### Sync info

```
crossfid status 2>&1 | jq .SyncInfo
```

### Add new wallet

```
crossfid keys add $WALLET
```

### Restore wallet

```
crossfid keys add $WALLET --recover
```

### List all wallets

```
crossfid keys list
```

### Check balance

```
crossfid q bank balances $(crossfid keys show $WALLET -a)
```

## 💱Transaction operations

Withdraw all rewards

{% code overflow="wrap" %}
```
crossfid tx distribution withdraw-all-rewards --from $WALLET --chain-id crossfi-evm-testnet-1 --gas auto --gas-adjustment 1.5 --gas-prices 5000000000000mpx
```
{% endcode %}

Self delegate

{% code overflow="wrap" %}
```
crossfid tx staking delegate $(crossfid keys show $WALLET --bech val -a) 1000000mpx --from $WALLET --chain-id crossfi-evm-testnet-1 --gas auto --gas-adjustment 1.5 --gas-prices 5000000000000mpx -y
```
{% endcode %}

Unbond

{% code overflow="wrap" %}
```
crossfid tx staking unbond $(crossfid keys show $WALLET --bech val -a) 1000000mpx --from $WALLET --chain-id crossfi-evm-testnet-1 --gas auto --gas-adjustment 1.5 --gas-prices 5000000000000mpx -y
```
{% endcode %}

Transfer token

{% code overflow="wrap" %}
```
crossfid tx bank send $WALLET_ADDRESS <TO_WALLET_ADDRESS> 1000000mpx --gas auto --gas-adjustment 1.5 --gas-prices 5000000000000mpx -y
```
{% endcode %}

## ✅Validator operations

Create validator

{% code overflow="wrap" %}
```sh
crossfid tx staking create-validator \
--amount 1000000mpx \
--from $WALLET \
--commission-rate 0.1 \
--commission-max-rate 0.2 \
--commission-max-change-rate 0.01 \
--min-self-delegation 1 \
--pubkey $(crossfid tendermint show-validator) \
--moniker "$MONIKER" \
--identity "put your keybase pgp here" \
--details "anything you want to say here" \
--chain-id crossfi-evm-testnet-1 \
--gas auto --gas-adjustment 1.5 --gas-prices 5000000000000mpx \
-y
```
{% endcode %}

{% hint style="info" %}
Can follow this guide to get your keybase pgp : [https://docs.harmony.one/home/network/validators/managing-a-validator/adding-a-validator-logo](https://docs.harmony.one/home/network/validators/managing-a-validator/adding-a-validator-logo)
{% endhint %}

Edit validator

{% code overflow="wrap" %}
```sh
crossfid tx staking edit-validator \
--commission-rate 0.1 \
--new-moniker "$MONIKER" \
--identity "put your keybase pgp here" \
--details "anything you want to say here" \
--from $WALLET \
--chain-id crossfi-evm-testnet-1 \
--gas auto --gas-adjustment 1.5 --gas-prices 5000000000000mpx \
-y
```
{% endcode %}

{% hint style="info" %}
You can leave any field blank if you don't want to change it
{% endhint %}
