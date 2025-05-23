# VM Method

## 💭Prerequisites

1. Dedicated Server. You can get affordable [dedicated server with high performance here](https://billing.fiberstate.com/aff.php?aff=185), if you plan to run multiple nodes
2. &#x20;You have the server ready by following [this guide](../../../advance/proxmox-setup-with-1-public-ip.md)

## 🛠️Installation <a href="#install-binary" id="install-binary"></a>

Follow the [manual installation](../installation.md) guide in 1 of the VM, until <kbd>Verify Installation</kbd> part

Clone your VM

<figure><img src="../../../.gitbook/assets/image (21).png" alt=""><figcaption></figcaption></figure>

Edit <kbd>.env</kbd> with your existing key or generate new key

After that, simply start the service

```sh
sudo systemctl start cortensor
```
