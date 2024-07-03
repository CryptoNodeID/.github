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
sudo journalctl -fu wardend
```
{% endtab %}

{% tab title="Reload daemon" %}
```bash
sudo systemctl daemon-reload
```
{% endtab %}

{% tab title="Enable service" %}
```sh
sudo systemctl enable wardend
```
{% endtab %}

{% tab title="Disable service" %}
```sh
sudo systemctl disable wardend
```
{% endtab %}
{% endtabs %}

{% tabs %}
{% tab title="Start service" %}
```sh
sudo systemctl start wardend
```
{% endtab %}

{% tab title="Stop service" %}
```sh
sudo systemctl stop wardend
```
{% endtab %}

{% tab title="Restart service" %}
```sh
sudo systemctl status wardend
```
{% endtab %}

{% tab title="Check service status" %}
```sh
sudo systemctl status wardend
```
{% endtab %}
{% endtabs %}

## 🖥️Node operations

{% tabs %}
{% tab title="Overall status" %}
```
wardend status | jq
```
{% endtab %}

{% tab title="Node info" %}
```
wardend status 2>&1 | jq .NodeInfo
```
{% endtab %}

{% tab title="Sync info" %}
```
wardend status 2>&1 | jq .SyncInfo
```
{% endtab %}
{% endtabs %}

## 🗝️Key Management

{% tabs %}
{% tab title="Add new wallet" %}
```
wardend keys add $WALLET
```
{% endtab %}

{% tab title="Restore wallet" %}
```
wardend keys add $WALLET --recover
```
{% endtab %}

{% tab title="List all wallet" %}
```
wardend keys list
```
{% endtab %}

{% tab title="Check balance" %}
```
wardend q bank balances $(crossfid keys show $WALLET -a)
```
{% endtab %}
{% endtabs %}

## 💱Transaction operations

Withdraw all rewards

{% code overflow="wrap" %}
```
wardend tx distribution withdraw-all-rewards --from $WALLET --chain-id buenavista-1 --gas auto --gas-adjustment 1.5 --fees 500uward -y
```
{% endcode %}

Self delegate

{% code overflow="wrap" %}
```
wardend tx staking delegate $(crossfid keys show $WALLET --bech val -a) 1000000mpx --from $WALLET --chain-id buenavista-1 --gas auto --gas-adjustment 1.5 --fees 500uward -y
```
{% endcode %}

Unbond

{% code overflow="wrap" %}
```
wardend tx staking unbond $(crossfid keys show $WALLET --bech val -a) 1000000mpx --from $WALLET --chain-id buenavista-1 --gas auto --gas-adjustment 1.5 --fees 500uward -y
```
{% endcode %}

Transfer token

{% code overflow="wrap" %}
```
wardend tx bank send $WALLET <TO_WALLET_ADDRESS> 1000000mpx --gas auto --gas-adjustment 1.5 --fees 500uward -y
```
{% endcode %}

## ✅Validator operations

Create validator

Check [validator-setup.md](validator-setup.md "mention")

{% hint style="info" %}
Can follow this guide to get your keybase pgp : [https://docs.harmony.one/home/network/validators/managing-a-validator/adding-a-validator-logo](https://docs.harmony.one/home/network/validators/managing-a-validator/adding-a-validator-logo)
{% endhint %}

Unjail Validator

<pre class="language-bash"><code class="lang-bash">wardend tx slashing unjail \
--from $WALLET \
--chain-id buenavista-1 \
<strong>--fees=500uward -y 
</strong></code></pre>
