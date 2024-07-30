# Cheat Sheet

If you use our auto-installation, we've included several scripts under `./scripts` folder like:

```sh
./list_keys.sh
./check_balance.sh
./create_validator.sh
./unjail_validator.sh
./check_validator.sh
./start_wardend.sh
./check_log.sh
```

Here's other useful commands you might need

## ⚙️Service operations <a href="#service-operations" id="service-operations"></a>

{% tabs %}
{% tab title="Check logs" %}
```sh
sudo journalctl -fu symphonyd
```
{% endtab %}

{% tab title="Reload daemon" %}
```bash
sudo systemctl daemon-reload
```
{% endtab %}

{% tab title="Enable service" %}
```sh
sudo systemctl enable symphonyd
```
{% endtab %}

{% tab title="Disable service" %}
```sh
sudo systemctl disable symphonyd
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Start service" %}
```sh
sudo systemctl start symphonyd
```
{% endtab %}

{% tab title="Stop service" %}
```sh
sudo systemctl stop symphonyd
```
{% endtab %}

{% tab title="Restart service" %}
```sh
sudo systemctl status symphonyd
```
{% endtab %}

{% tab title="Check service status" %}
```sh
sudo systemctl status symphonyd
```
{% endtab %}
{% endtabs %}

## 🖥️Node operations

{% tabs %}
{% tab title="Overall status" %}
```
symphonyd status | jq
```
{% endtab %}

{% tab title="Node info" %}
```
symphonyd status 2>&1 | jq .NodeInfo
```
{% endtab %}

{% tab title="Sync info" %}
```
symphonyd status 2>&1 | jq .SyncInfo
```
{% endtab %}
{% endtabs %}

## 🗝️Key Management

{% tabs %}
{% tab title="Add new wallet" %}
```
symphonyd keys add $WALLET
```
{% endtab %}

{% tab title="Restore wallet" %}
```
symphonyd keys add $WALLET --recover
```
{% endtab %}

{% tab title="List all wallet" %}
```
symphonyd keys list
```
{% endtab %}

{% tab title="Check balance" %}
```
symphonyd q bank balances $(symphonyd keys show $WALLET -a)
```
{% endtab %}
{% endtabs %}

## 💱Transaction operations

Withdraw all rewards

{% code overflow="wrap" %}
```sh
symphonyd tx distribution withdraw-all-rewards \
--from $WALLET \
--chain-id symphony-testnet-2 \
--gas auto \
--gas-adjustment 1.5 \
--fees 800note -y
```
{% endcode %}

Withdraw rewards and commission from your validator

```sh
symphonyd tx distribution withdraw-rewards $(symphonyd keys show $WALLET --bech val -a) \
--from $WALLET \
--commission \
--chain-id symphony-testnet-2 \
--gas auto \
--gas-adjustment 1.5 \
--fees 800note -y
```

Self delegate

{% code overflow="wrap" %}
```sh
symphonyd tx staking delegate $(symphonyd keys show $WALLET --bech val -a) 1000000note \
--from $WALLET \
--chain-id symphony-testnet-2 \
--gas auto \
--gas-adjustment 1.5 \
--fees 800note -y
```
{% endcode %}

Redelegate

```sh
symphonyd tx staking redelegate <FROM_VALOPER_ADDRESS> <TO_VALOPER_ADDRESS> 1000000note \
--from $WALLET \
--chain-id symphony-testnet-2 \
--gas auto \
--gas-adjustment 1.5 \
--fees 800note -y
```

Unbond

{% code overflow="wrap" %}
```sh
symphonyd tx staking unbond $(symphonyd keys show $WALLET --bech val -a) 1000000note \
--from $WALLET \
--chain-id symphony-testnet-2 \
--gas auto --gas-adjustment 1.5 \
--fees 800note -y
```
{% endcode %}

Transfer token

{% code overflow="wrap" %}
```sh
symphonyd tx bank send $WALLET <TO_WALLET_ADDRESS> 1000000note \
--gas auto \
--gas-adjustment 1.5 \
--fees 800note -y
```
{% endcode %}

## ✅Validator operations

Create validator

Check [validator-setup.md](../warden-protocol/validator-setup.md "mention")

{% hint style="info" %}
Can follow this guide to get your keybase pgp : [https://docs.harmony.one/home/network/validators/managing-a-validator/adding-a-validator-logo](https://docs.harmony.one/home/network/validators/managing-a-validator/adding-a-validator-logo)
{% endhint %}

Unjail Validator

```bash
symphonyd tx slashing unjail \
--from $WALLET \
--chain-id symphony-testnet-2 \
--fees=800note -y
```

## 🏛️Governance

Query Proposal List

```sh
symphonyd query gov proposals
```

Vote

```sh
symphonyd tx gov vote 1 yes \
--from $WALLET \
--chain-id symphony-testnet-2 \
--gas auto \
--gas-adjustment 1.5 \
--fees=800note -y
```

{% hint style="warning" %}
vote value can be `yes`, `no`, `no_with_veto` and `abstain`
{% endhint %}
