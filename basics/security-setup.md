---
description: This is basic security setup that we use to protect our servers
---

# Security Setup

## 💭Prerequisites

Have an application to create keypair. In this case, we will use [MobaXterm](https://mobaxterm.mobatek.net)

## Application steps

### 1. Create Key Pair

Go to mobaxterm and open MobaKeyGen

&#x20;

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

Choose EdDSA and change the dropdown to Ed25519 > Generate

<figure><img src="../.gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

When generating, you will be asked to move your mouse randomly inside the red highlighted part.

After generating, you will see screen like below

<figure><img src="../.gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

you can change the <kbd>key comment</kbd> as you like, and save both keys securely. <kbd>privatekey</kbd> is the one that you will use to authenticate to your server later.

The item marked in green is the one we'll use in the next step.

### 2. Script Modification and execution

The provided script sets up a new user and enhances server security through several measures. It installs essential packages and configures SSH for secure access by disabling root login and password authentication, allowing only public key authentication. Fail2ban is installed and configured to protect against brute-force attacks by temporarily banning IPs with failed login attempts. The modified SSH configuration restricts various forwarding options, reducing the attack surface. These combined steps provide enhanced protection against unauthorized access.

{% hint style="danger" %}
This script will prevent you to login as root directly and only allow whitelisted user to login with PRIVATEKEY, NOT PASSWORD.

DO WITH CARE OR YOU WILL LOSE ACCESS TO YOUR SERVER. YOU'VE BEEN WARNED!
{% endhint %}

Modify below script accordingly, change the <kbd>PUB\_KEY</kbd> and <kbd>USER</kbd> part based on your needs, re-check the script and execute.

```sh
PUB_KEY="YOUR PUBLIC KEY HERE"
USER="YOUR USERNAME HERE"

sudo apt update -qy
sudo apt install -qy git jq lz4 build-essential unzip net-tools ca-certificates fail2ban
sudo apt upgrade -qy
sudo useradd -m -d /home/$USER -s /bin/bash $USER
sudo echo "$USER ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
sudo mkdir -p /home/$USER/.ssh
sudo chown -R $USER:$USER /home/$USER
sudo echo "$PUB_KEY" >> /home/$USER/.ssh/authorized_keys
sudo sed -i -e '/^\(#\|\)PermitRootLogin/s/^.*$/PermitRootLogin no/' /etc/ssh/sshd_config
sudo sed -i -e '/^\(#\|\)PasswordAuthentication/s/^.*$/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i -e '/^\(#\|\)KbdInteractiveAuthentication/s/^.*$/KbdInteractiveAuthentication no/' /etc/ssh/sshd_config
sudo sed -i -e '/^\(#\|\)ChallengeResponseAuthentication/s/^.*$/ChallengeResponseAuthentication no/' /etc/ssh/sshd_config
sudo sed -i -e '/^\(#\|\)MaxAuthTries/s/^.*$/MaxAuthTries 2/' /etc/ssh/sshd_config
sudo sed -i -e '/^\(#\|\)AllowTcpForwarding/s/^.*$/AllowTcpForwarding no/' /etc/ssh/sshd_config
sudo sed -i -e '/^\(#\|\)X11Forwarding/s/^.*$/X11Forwarding no/' /etc/ssh/sshd_config
sudo sed -i -e '/^\(#\|\)AllowAgentForwarding/s/^.*$/AllowAgentForwarding no/' /etc/ssh/sshd_config
sudo sed -i -e '/^\(#\|\)AuthorizedKeysFile/s/^.*$/AuthorizedKeysFile .ssh\/authorized_keys/' /etc/ssh/sshd_config
sudo sed -i "\$a AllowUsers ${USER}" /etc/ssh/sshd_config
sudo printf "[sshd]\nenabled = true\nbanaction = iptables-multiport\nbackend = systemd\nbantime = 1h\nbantime.increment = true\nbantime.factor = 24\nbantime.maxtime = 5w\n\n[recidive]\nenabled = true\nlogpath = /var/log/fail2ban.log\nbanaction = %%(banaction_allports)s\nbantime = -1\nfindtime = 86400\nmaxretry = 6" > /etc/fail2ban/jail.local
sudo systemctl restart fail2ban
sudo systemctl reload ssh
```

{% hint style="success" %}
Now your server is slightly secured. Not that we make it bullet proof, but at least we reduced some of the risks of being hacked. 😁
{% endhint %}
