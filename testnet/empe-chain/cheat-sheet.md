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
sudo journalctl -fu emped
```
{% endtab %}

{% tab title="Reload daemon" %}
```bash
sudo systemctl daemon-reload
```
{% endtab %}

{% tab title="Enable service" %}
```sh
sudo systemctl enable emped
```
{% endtab %}

{% tab title="Disable service" %}
```sh
sudo systemctl disable emped
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Start service" %}
```sh
sudo systemctl start emped
```
{% endtab %}

{% tab title="Stop service" %}
```sh
sudo systemctl stop emped
```
{% endtab %}

{% tab title="Restart service" %}
```sh
sudo systemctl status emped
```
{% endtab %}

{% tab title="Check service status" %}
```sh
sudo systemctl status emped
```
{% endtab %}
{% endtabs %}

## 🖥️Node operations

{% tabs %}
{% tab title="Overall status" %}
```
emped status | jq
```
{% endtab %}

{% tab title="Node info" %}
```
emped status 2>&1 | jq .NodeInfo
```
{% endtab %}

{% tab title="Sync info" %}
```
emped status 2>&1 | jq .SyncInfo
```
{% endtab %}
{% endtabs %}

## 🗝️Key Management

{% tabs %}
{% tab title="Add new wallet" %}
```
emped keys add $WALLET
```
{% endtab %}

{% tab title="Restore wallet" %}
```
emped keys add $WALLET --recover
```
{% endtab %}

{% tab title="List all wallet" %}
```
emped keys list
```
{% endtab %}

{% tab title="Check balance" %}
```
emped q bank balances $(emped keys show $WALLET -a)
```
{% endtab %}
{% endtabs %}

## 💱Transaction operations

Withdraw all rewards

{% code overflow="wrap" %}
```sh
emped tx distribution withdraw-all-rewards \
--from $WALLET \
--chain-id empe-testnet-2 \
--gas auto \
--gas-adjustment 1.5 \
--fees 20uempe -y
```
{% endcode %}

Withdraw rewards and commission from your validator

```sh
emped tx distribution withdraw-rewards $(emped keys show $WALLET --bech val -a) \
--from $WALLET \
--commission \
--chain-id empe-testnet-2 \
--gas auto \
--gas-adjustment 1.5 \
--fees 20uempe -y
```

Self delegate

{% code overflow="wrap" %}
```sh
emped tx staking delegate $(emped keys show $WALLET --bech val -a) 1000000uempe \
--from $WALLET \
--chain-id empe-testnet-2 \
--gas auto \
--gas-adjustment 1.5 \
--fees 20uempe -y
```
{% endcode %}

Redelegate

```sh
emped tx staking redelegate <FROM_VALOPER_ADDRESS> <TO_VALOPER_ADDRESS> 1000000uempe \
--from $WALLET \
--chain-id empe-testnet-2 \
--gas auto \
--gas-adjustment 1.5 \
--fees 20uempe -y
```

Unbond

{% code overflow="wrap" %}
```sh
emped tx staking unbond $(emped keys show $WALLET --bech val -a) 1000000uempe \
--from $WALLET \
--chain-id empe-testnet-2 \
--gas auto --gas-adjustment 1.5 \
--fees 20uempe -y
```
{% endcode %}

Transfer token

{% code overflow="wrap" %}
```sh
emped tx bank send $WALLET <TO_WALLET_ADDRESS> 1000000uempe \
--gas auto \
--gas-adjustment 1.5 \
--fees 20uempe -y
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
emped tx slashing unjail \
--from $WALLET \
--chain-id empe-testnet-2 \
--fees=20uempe -y
```

## 🏛️Governance

Query Proposal List

```sh
emped query gov proposals
```

Vote

```sh
emped tx gov vote 1 yes \
--from $WALLET \
--chain-id empe-testnet-2 \
--gas auto \
--gas-adjustment 1.5 \
--fees=20uempe -y
```

{% hint style="warning" %}
vote value can be `yes`, `no`, `no_with_veto` and `abstain`
{% endhint %}
