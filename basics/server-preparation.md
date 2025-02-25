# Server Preparation

## Installing Dependencies

We will need to install these dependencies and common tools before we can continue to the next steps.\
Simply by executing below commands :&#x20;

```sh
sudo apt -q update
sudo apt -qy install curl git jq lz4 build-essential snapd unzip nginx wget
sudo apt -qy upgrade
```

{% hint style="info" %}
This guide is made by assuming you're using Ubuntu 22.04 LTS. You will need to modify to match your OS accordingly
{% endhint %}

## Install Docker (Optional)

```sh
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove -qy $pkg; done

sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -qy docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl start docker
```

### Set to run docker as a non-root user

```sh
sudo chmod 666 /var/run/docker.sock
sudo usermod -aG docker <your_username>
```
