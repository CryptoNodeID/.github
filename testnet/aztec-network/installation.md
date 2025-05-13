# Installation

## 🖥️System Requirements

\- OS: Linux\
\- CPU: 8 Core(s)\
\- Memory: 16GB\
\- Storage: 1TB SSD\
\- Network: 25Mbps Minimum

## 💭Prerequisites

Have Docker installed in your system, if you don't, please follow [this guide](../../basics/server-preparation.md#install-docker-optional)

## 🛠️Installation <a href="#install-binary" id="install-binary"></a>

{% hint style="success" %}
This guide is created under the assumption you are using Ubuntu 22.04 LTS\
If you use other OS, please modify the commands accordingly
{% endhint %}

1.  Install <kbd>aztec</kbd> cli

    <pre class="language-bash"><code class="lang-bash"><strong>bash -i &#x3C;(curl -s https://install.aztec.network)
    </strong></code></pre>
2.  Create folder for easier management

    ```bash
    mkdir -p aztec-docker/data
    ```
3.  Go inside the folder and create docker-compose.yml

    ```bash
    cd aztec-docker && vi docker-compose.yml
    ```
4.  Put this <kbd>yml</kbd> content in <kbd>docker-compose.yml</kbd> and modify accordingly

    ```yaml
    name: aztec-sequencer
    services:
      aztec-sequencer:
        container_name: aztec-sequencer
        restart: unless-stopped
        image: aztecprotocol/aztec:alpha-testnet
        environment:
          ETHEREUM_HOSTS: "<your ETH RPC>"
          L1_CONSENSUS_HOST_URLS: "<your ETH Beacon>"
          DATA_DIRECTORY: /data
          VALIDATOR_PRIVATE_KEY: 0x<your private key hash>
          COINBASE: 0x<your address>
          P2P_IP: <your IP address>
          LOG_LEVEL: debug
        entrypoint: >
          sh -c 'node --no-warnings /usr/src/yarn-project/aztec/dest/bin/index.js start --network alpha-testnet start --node --archiver --sequencer'
        ports:
          - 40400:40400/tcp
          - 40400:40400/udp
          - 8080:8080
        volumes:
          - ./data:/data
    ```
5. Send some <kbd>Sepolia ETH</kbd> to your wallet address to get started
6.  Start the service with this command

    ```bash
    docker compose up -d
    ```

{% hint style="success" %}
To check log, run&#x20;

```bash
docker compose logs -fn 100
```

To stop:

```bash
docker compose down
```
{% endhint %}
