# Installation

## 🖥️System Requirements

\- OS: Windows (as Launcher only) / Ubuntu 20.04 (amd64) / macOS 12+ (both Intel and Apple Silicon) / Ubuntu 22.04 need workaround \
\- CPU: 4 CPU Cores \
\- Memory: 4 GB RAM \
\- Storage: 160 GB disk space \
\- Network: 100 Mbps internet connection

## 📽️Video Installation Instruction

You can follow instruction in this video to install your Humanode Validator node

{% embed url="https://youtu.be/AAkVG_g73-g" %}
Source: [https://www.youtube.com/watch?v=AAkVG\_g73-g](https://www.youtube.com/watch?v=AAkVG\_g73-g)
{% endembed %}

### Special instruction for Ubuntu 22.04 user

When you encounter error while generating address, you can execute this command in your host server and continue with the installation

```sh
mkdir $HOME/opt && cd $HOME/opt
wget https://www.openssl.org/source/openssl-1.1.1o.tar.gz
tar -zxvf openssl-1.1.1o.tar.gz
cd openssl-1.1.1o
./config && make && make test
mkdir $HOME/opt/lib
mv $HOME/opt/openssl-1.1.1o/libcrypto.so.1.1 $HOME/opt/lib/
mv $HOME/opt/openssl-1.1.1o/libssl.so.1.1 $HOME/opt/lib/
sudo ln -s $HOME/opt/lib/libssl.so.1.1 /usr/lib/libssl.so.1.1
sudo ln -s $HOME/opt/lib/libssl.so.1.1 /usr/lib64/libssl.so.1.1
sudo ln -s $HOME/opt/lib/libcrypto.so.1.1 /usr/lib/libcrypto.so.1.1
sudo ln -s $HOME/opt/lib/libcrypto.so.1.1 /usr/lib64/libcrypto.so.1.1
```
