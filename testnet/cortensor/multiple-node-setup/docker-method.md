# Docker Method

{% hint style="warning" %}
This guide is quoted from [https://docs.aldebaranode.xyz/guide/testnet/cortensor/installation](https://docs.aldebaranode.xyz/guide/testnet/cortensor/installation)

All rights reserved to [Aldebaranode](https://aldebaranode.xyz)
{% endhint %}

### Prerequisites

Ensure that you meet the hardware requirements to run the Cortensor network. For detailed information, you can visit the [official website](https://docs.cortensor.network/getting-started/quick-start-guide#hardware-requirements).

### Step 1 - Install Docker

Run the following command to uninstall all conflicting packages:

{% code overflow="wrap" %}
```sh
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
```
{% endcode %}

Set up Docker’s apt repository.

{% code overflow="wrap" %}
```sh
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \  
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```
{% endcode %}

Install the Docker packages.

{% code overflow="wrap" %}
```sh
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
{% endcode %}

### Step 2 - Clone Installer Repository

Install Git to clone the installer repository.

```sh
sudo apt-get install git -y
```

{% code overflow="wrap" %}
```sh
git clone https://github.com/mrheed/cortensor-installer.git && cd cortensor-installer
```
{% endcode %}

### Step 3 - Config Docker Compose

Run the build.sh script. The script will do the following:

* Build the Dockerfile for the Cortensor image
* Generate the docker-compose.yaml file for you to adjust your total nodes

```sh
bash build.sh
```

After the docker-compose.yaml is generated, you need to modify the environment variables for the cortensor section:

* **RPC\_URL**: Your ARB Sepolia RPC URL, which you can obtain by running your own node or using a service provider like Ankr, Alchemy, or Infura.
* **PUBLIC\_KEY**: The EVM address of your miner node.
* **PRIVATE\_KEY**: The EVM private key of your miner node.

![Env Variable](https://docs.aldebaranode.xyz/_next/image?url=%2F_next%2Fstatic%2Fmedia%2Fenv.ccadbbcb.png\&w=1080\&q=75)

### Step 4 - Run the Nodes

Ensure that the working directory is within the installer folder.

Run your nodes using this command

```sh
docker compose up -d
```

Run this command to check all the container logs

```sh
docker compose logs --tail 100 -f
```

Run this command to stop the nodes

```sh
docker compose down
```

Run this command to delete all nodes

```sh
docker compose rm
```
