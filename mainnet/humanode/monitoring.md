# Monitoring

<figure><img src="../../.gitbook/assets/image (61).png" alt=""><figcaption><p>sample alert</p></figcaption></figure>

This simple script will monitor your node and send alert to your telegram chat/group when your bioauth is about to end or already in inactive state.

## Installation

### Prepare Telegram API Key

go to Telegram, search for [@BotFather](http://t.me/BotFather)&#x20;

<figure><img src="../../.gitbook/assets/image (62).png" alt="" width="224"><figcaption></figcaption></figure>

type in `/start` to interact with @BotFather and create your bot with `/newbot` command

after that type `/token` to get your API Key

### Install script

Copy and save this script below with .sh format in your **Humanode server**&#x20;

```shell
#!/bin/bash
telegram_token='<YOUR_API_KEY>'
telegram_group='<YOUR_TELEGRAM_GROUP>'
telegram_user_tag="@<YOUR_TELEGRAM_USERNAME>"
# Stop editing
# Script starts here
server_ip=$(curl -s https://api.ipify.org)
telegram_bot="https://api.telegram.org/bot${telegram_token}/sendMessage"
status=$(curl -s http://127.0.0.1:9933 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"bioauth_status","params":[],"id":1}' | jq '.result')

if [ "$(echo "$status" | tr '[:upper:]' '[:lower:]')" == "$(echo '"inactive"' | tr '[:upper:]' '[:lower:]')" ]; then
  message="🚨Your Humanode (${server_ip}) is not active, please proceed to do re-authentication ${telegram_user_tag}"
  else
  current_timestamp=$(date +%s)
  expires_at=$(curl -s http://127.0.0.1:9933 -X POST -H "Content-Type: application/json" -d '{"jsonrpc":"2.0","method":"bioauth_status","params":[],"id":1}' | jq '.result.Active.expires_at')
  difference=$(( (expires_at / 1000 - current_timestamp) / 60 ))

  if (( difference > 25 && difference < 31 )); then
    message="🟡Your Humanode (${server_ip}) will be deactivated in 30 minutes, please prepare for re-authentication ${telegram_user_tag}"
  elif (( difference > 0 && difference < 6 )); then
    message="🔴Your Humanode (${server_ip}) will be deactivated in 5 minutes, please prepare for re-authentication ${telegram_user_tag}"
  else
    message="NULL"
  fi
fi
if [ "$message" != "NULL" ]; then
  curl -X POST -H "Content-Type:multipart/form-data" -F chat_id=${telegram_group} -F text="${message}" ${telegram_bot}
fi

```

{% hint style="danger" %}
Replace `<YOUR_API_KEY>` with the key you get from previous step
{% endhint %}

{% hint style="warning" %}
For `telegram_group` you can change it to your own chat/group.
{% endhint %}

{% hint style="danger" %}
Replace `telegram_user_tag` with your own username
{% endhint %}

After that, don't forget to give execute permission by using this command below

```sh
chmod ug+x /path/to/your/script.sh
```

### Set cronjob

You need to install the cronjob to make this alert work.

Open cron editor by executing this:

```sh
crontab -e
```

After that, put this line into the last line of crontab and then save it

```sh
*/5 * * * * /path/to/your/script.sh
```

{% hint style="info" %}
This cron will run the script every 5 minutes
{% endhint %}

{% hint style="danger" %}
Remember to replace `/path/to/your/script.sh` to your actual script location
{% endhint %}
