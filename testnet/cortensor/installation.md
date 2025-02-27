# Installation

## 🖥️System Requirements

\- OS: Linux\
\- CPU: 6 Core(s)\
\- Memory: 12GB\
\- Storage: 100GB

You can get your server here : ![🖥](https://web.telegram.org/a/blank.8dd283bceccca95a48d8.png)[Click Here to Rent VPS from €4.50/month](https://www.dpbolvw.net/bi103shqnhp465665C69E46ABE79DB?sid=CNID-R)

## 💭Prerequisites

You need to have 2000 $COR staked per node for 84 days  in [Cortensor staking platform](https://stake.cortensor.network/)

{% hint style="warning" %}
If you want to run 5, then stake 10k $COR
{% endhint %}

## 🛠️Installation <a href="#install-binary" id="install-binary"></a>

{% hint style="success" %}
This guide is created under the assumption you are using Ubuntu 24.04 LTS\
If you use other OS, please modify the commands accordingly
{% endhint %}

{% hint style="warning" %}
It's recommended to run the command using <kbd>root</kbd> user
{% endhint %}

#### Clone Installer <a href="#clone-installer" id="clone-installer"></a>

```sh
git clone https://github.com/cortensor/installer
cd installer
```

#### Install Dependencies

```sh
./install-docker-ubuntu.sh
./install-ipfs-linux.sh
./install-linux.sh
```

#### Copy Installation folder to <kbd>deploy</kbd> user

```sh
cp -Rf ./installer /home/deploy/installer
chown -R deploy:deploy /home/deploy/installer
```

#### Login as <kbd>deploy</kbd> user and verify the installation

```sh
ls -al /usr/local/bin/cortensord
ls -al $HOME/.cortensor/bin/cortensord
ls -al /etc/systemd/system/cortensor.service
ls -al $HOME/.cortensor/bin/start-cortensor.sh
ls -al $HOME/.cortensor/bin/stop-cortensor.sh
docker version
ipfs version
```

#### Generate Key

```sh
/usr/local/bin/cortensord ~/.cortensor/.env tool gen_key
```

after this step, contact Cortensor admin via [Discord](https://discord.gg/cortensor) and send them your <kbd>Public key</kbd> for whitelist

```
Staking Address: 0x.....E2A1 (your wallet that you use for staking)
Mining Address:  0x.....C8D3 (your node public key)
Discord: yourdiscord
Telegram: @yourtelegram
```

Only after confirmation from Admin, you can proceed to register, verify and start the node

```sh
/usr/local/bin/cortensord ~/.cortensor/.env tool register
/usr/local/bin/cortensord ~/.cortensor/.env tool verify
```

#### Start the service

```sh
sudo systemctl start cortensor
```
