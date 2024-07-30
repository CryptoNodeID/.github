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

### Get Testnet EMPE

You can obtain testnet tokens to fund your address from official EMPE faucet:

[https://faucet-testnet.empe.io/#/](https://faucet-testnet.empe.io/#/)

You can verify your balance with this command:

```sh
symphonyd query bank balances $(symphonyd keys show $WALLET -a)
```

### Create Validator

Obtain your validator public key by running the following command:

```sh
symphonyd tendermint show-validator
```

The output will be similar to this (with a different key):

{% code overflow="wrap" %}
```json
{"@type":"/cosmos.crypto.ed25519.PubKey","key":"lR1d7YBVK5jYijOfWVKRFoWCsS4dg3kagT7LB9GnG8I="}
```
{% endcode %}

You can create your validator by using command below :

```bash
symphonyd tx staking create-validator \
  --amount=100000uempe \
  --pubkey=$(symphonyd tendermint show-validator) \
  --moniker=${MONIKER} \
  --identity=${DISPLAY_PIC} \
  --website=${WEBSITE} \
  --security-contact=${CONTACT} \
  --details=${DETAILS} \
  --chain-id=empe-testnet-2 \
  --commission-rate="0.05" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --gas="auto" \
  --min-self-delegation="10000" \
  --fees=800note \
  --from=${WALLET}
```

{% hint style="info" %}
When specifying commission parameters, the `commission-max-change-rate` is used to measure % _point_ change over the `commission-rate`. E.g. 1% to 2% is a 100% rate increase, but only 1 percentage point.
{% endhint %}

### Unjail Validator

When a validator is "jailed" for downtime, you must submit an `Unjail` transaction from the operator account in order to be able to get block proposer rewards again (depends on the zone fee distribution).

```bash
emped tx slashing unjail \
--from=${WALLET} \
--chain-id=empe-testnet-2 \
--fees=800note
```

### Halting Your Validator

When attempting to perform routine maintenance or planning for an upcoming coordinated upgrade, it can be useful to have your validator systematically and gracefully halt. You can achieve this by either setting the `halt-height` to the height at which you want your node to shutdown or by passing the `--halt-height` flag to `symphonyd`. The node will shutdown with a zero exit code at that given height after committing the block.

#### Get last block height

{% code overflow="wrap" %}
```sh
curl -sX GET "https://symphony-testnet-api.cryptonode.id/cosmos/base/tendermint/v1beta1/blocks/latest" -H  "accept: application/json" | jq '.block.last_commit.height'
```
{% endcode %}

#### Halt Validator

```sh
symphonyd start --halt-height HALT_HEIGHT
```

{% hint style="info" %}
replace HALT\_HEIGHT  with number shown from result of [#get-last-block-height](validator-setup.md#get-last-block-height "mention") and add some (like 100 or 500)
{% endhint %}
