# Installation

## Prerequisites

### Install Node and Yarn

```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
nvm install node
nvm install-latest-npm
npm install --global yarn
```

## Install

### Clone from our repo

```sh
git clone https://github.com/CryptoNodeID/explorer.git
```

{% hint style="warning" %}
Would be better if you fork from our repo and clone it from yours so you can modify it freely
{% endhint %}

### Build

```sh
cd explorer
yarn --ignore-engines && yarn build
```

the build result will be under `dist` folder in the current working directory

## Setup nginx

Just like in [basics](../../basics/ "mention")segment ( [routing-using-nginx.md](../../basics/routing-using-nginx.md "mention") ), we're going to create config for explorer.\
In assumption you've know how to and done [setting-up-ssl.md](../../basics/setting-up-ssl.md "mention"), here's the commands and config example:

```
sudo nano /etc/nginx/sites-available/explorer
```

```
server {
    listen 443 ssl;
    listen [::]:443 ssl;
    server_name explorer.cryptonode.id;

    ssl_certificate /etc/letsencrypt/live/explorer.cryptonode.id/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/explorer.cryptonode.id/privkey.pem;

    root ~/explorer/dist;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ /index.html =404;
    }
}
```

```
ln -s /etc/nginx/sites-available/explorer /etc/nginx/sites-enabled/explorer
```

{% hint style="info" %}
You might need to modify the nginx config to follow your server configuration like `ssl_certificate` , `ssl_certificate_key` and `root` location where you put the installation of your explorer.
{% endhint %}
