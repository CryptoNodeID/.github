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

You can create your validator by using command below :

```bash
crossfid --home ${DAEMON_HOME} tx staking create-validator \
  --amount=9900000000000000000000mpx \
  --pubkey=$(crossfid --home ${DAEMON_HOME} tendermint show-validator) \
  --moniker="${MONIKER}" \
  --identity=${DISPLAY_PIC} \
  --details="${DETAILS}" \
  --website="${WEBSITE}" \
  --security-contact="${CONTACT}" \
  --chain-id="crossfi-evm-testnet-1" \
  --commission-rate="0.10" \
  --commission-max-rate="0.20" \
  --commission-max-change-rate="0.01" \
  --min-self-delegation="1000000" \
  --gas="auto" \
  --gas-prices="10000000000000mpx" \
  --gas-adjustment=1.5 \
  --from=$WALLET
```

{% hint style="info" %}
When specifying commission parameters, the `commission-max-change-rate` is used to measure % _point_ change over the `commission-rate`. E.g. 1% to 2% is a 100% rate increase, but only 1 percentage point.
{% endhint %}

It's possible that you won't have enough MPX to be part of the active set of validators in the beginning. Users are able to delegate to inactive validators (those outside of the active set) using the [XFI Console](https://xficonsole.com/) You can confirm that you are in the validator set by using a third party explorer like [XFI Scan.](https://xfiscan.com/validators)

### Edit Validator

You can edit your validator details using command below :

{% hint style="warning" %}
Before you continue, you can edit your environment variable, or you can edit manually
{% endhint %}

```bash
crossfid --home ${DAEMON_HOME} tx staking edit-validator
  --moniker="${MONIKER}" \
  --website="${WEBSITE}" \
  --identity=${DISPLAY_PIC} \
  --details="${DETAILS}" \
  --chain-id="crossfi-evm-testnet-1" \
  --gas="auto" \
  --gas-prices="10000000000000mpx" \
  --gas-adjustment=1.5 \
  --from=<key_name> \
  --commission-rate="0.10"
```

### Unjail Validator

When a validator is "jailed" for downtime, you must submit an `Unjail` transaction from the operator account in order to be able to get block proposer rewards again (depends on the zone fee distribution).

```bash
crossfid tx slashing unjail \
 --from=${WALLET} \
 --chain-id="crossfi-evm-testnet-1"
```

### Halting Your Validator

When attempting to perform routine maintenance or planning for an upcoming coordinated upgrade, it can be useful to have your validator systematically and gracefully halt. You can achieve this by either setting the `halt-height` to the height at which you want your node to shutdown or by passing the `--halt-height` flag to `crossfid`. The node will shutdown with a zero exit code at that given height after committing the block.

#### Get last block height

{% code overflow="wrap" %}
```sh
curl -sX GET "https://crossfi-testnet-api.cryptonode.id/cosmos/base/tendermint/v1beta1/blocks/latest" -H  "accept: application/json" | jq '.block.last_commit.height'
```
{% endcode %}

#### Halt Validator

```sh
crossfid start --halt-height HALT_HEIGHT
```

{% hint style="info" %}
replace HALT\_HEIGHT  with number shown from result of [#get-last-block-height](validator-setup.md#get-last-block-height "mention") and add some (like 100 or 500)
{% endhint %}
