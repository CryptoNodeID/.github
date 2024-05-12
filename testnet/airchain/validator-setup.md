# Validator Setup

## 📶Running a Validator

### Set environment

```bash
export MONIKER="YOUR_NODE_NAME_HERE"
export WALLET="YOUR_WALLET_NAME_HERE"
export DETAILS="YOUR_NODE_INFORMATION_HERE"
export WEBSITE="YOUR_WEBSITE_HERE"
export DISPLAY_PIC="YOUR_KEYBASE_IMAGE_PGP"
export CONTACT="YOUR_CONTACT"
```

{% hint style="info" %}
WEBSITE, DETAILS, DISPLAY\_PIC and CONTACT are optional

for DISPLAY\_PIC please refer to this : [Harmony Guide](https://docs.harmony.one/home/network/validators/managing-a-validator/adding-a-validator-logo)
{% endhint %}

### Write env to .profile

```bash
echo 'export MONIKER=${MONIKER}' >> ~/.profile
echo 'export WALLET=${WALLET}' >> ~/.profile
echo 'export DETAILS=${DETAILS}' >> ~/.profile
echo 'export WEBSITE=${WEBSITE}' >> ~/.profile
echo 'export DISPLAY_PIC=${DISPLAY_PIC}' >> ~/.profile
echo 'export CONTACT=${CONTACT}' >> ~/.profile

source ~/.profile
```

{% hint style="info" %}
This step is optional, but we recommend you to do it for convenience
{% endhint %}

### Create Validator

Obtain your validator public key by running the following command:

```sh
junctiond comet show-validator
```

The output will be similar to this (with a different key):

{% code overflow="wrap" %}
```json
{"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="}
```
{% endcode %}

Create a file called `validator.json` and put the details below:

```json
{    
    "pubkey": {"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="},
    "amount": "4900000amf",
    "moniker": "your-node-moniker",
    "identity": "your-keybase-pgp",
    "website": "optional website for your validator",
    "security": "optional security contact for your validator",
    "details": "optional details for your validator",
    "commission-rate": "0.1",
    "commission-max-rate": "0.2",
    "commission-max-change-rate": "0.01",
    "min-self-delegation": "1"
}
```

OR you can use this command to generate `validator.json`

```sh
tee validator.json > /dev/null <<EOF
{
    "pubkey": $(junctiond comet show-validator),
    "amount": "4900000amf",
    "moniker": "${MONIKER}",
    "identity": "${DISPLAY_PIC}",
    "website": "${WEBSITE}",
    "security": "${CONTACT}",
    "details": "${DETAILS}",
    "commission-rate": "0.1",
    "commission-max-rate": "0.2",
    "commission-max-change-rate": "0.01",
    "min-self-delegation": "1"
}
EOF
```

You can create your validator by using command below :

```bash
junctiond tx staking create-validator validator.json \
    --from=${WALLET}  \
    --chain-id=junction \
    --fees=500amf
```

{% hint style="info" %}
When specifying commission parameters, the `commission-max-change-rate` is used to measure % _point_ change over the `commission-rate`. E.g. 1% to 2% is a 100% rate increase, but only 1 percentage point.
{% endhint %}

### Unjail Validator

When a validator is "jailed" for downtime, you must submit an `Unjail` transaction from the operator account in order to be able to get block proposer rewards again (depends on the zone fee distribution).

```bash
junctiond tx slashing unjail \
 --from=${WALLET} \
 --chain-id="junction" \
 --fees=500amf
```

### Halting Your Validator

When attempting to perform routine maintenance or planning for an upcoming coordinated upgrade, it can be useful to have your validator systematically and gracefully halt. You can achieve this by either setting the `halt-height` to the height at which you want your node to shutdown or by passing the `--halt-height` flag to `wardend`. The node will shutdown with a zero exit code at that given height after committing the block.

#### Get last block height

{% code overflow="wrap" %}
```sh
curl -sX GET "https://airchain-testnet-api.cryptonode.id/cosmos/base/tendermint/v1beta1/blocks/latest" -H  "accept: application/json" | jq '.block.last_commit.height'
```
{% endcode %}

#### Halt Validator

```sh
junctiond start --halt-height HALT_HEIGHT
```

{% hint style="info" %}
replace HALT\_HEIGHT  with number shown from result of [#get-last-block-height](validator-setup.md#get-last-block-height "mention") and add some (like 100 or 500)
{% endhint %}
