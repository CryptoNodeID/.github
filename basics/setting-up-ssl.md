# Setting up SSL

## Prerequisite

You need to have `snapd` installed.\
This step already covered in [server-preparation.md](server-preparation.md "mention")

And remove any previous certbot version installed by executing command below:

```sh
sudo apt-get remove certbot
```

## Setting up CertBot

### CertBot Installation

We will use CertBot as our SSL manager\
You can install it by using below commands:

```sh
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo snap set certbot trust-plugin-with-root=ok
```

### CertBot DNS Plugin

Since we're using Cloudflare as our DNS Manager, we need to install Cloudflare plugin using command below:

```sh
sudo snap install certbot-dns-cloudflare
```

{% hint style="info" %}
You will need to install another plugin if you use other DNS manager.\
visit [Official CertBot Guide](https://eff-certbot.readthedocs.io/en/latest/using.html#dns-plugins) for more details
{% endhint %}

### Get your credential API Token

To obtain your Cloudflare API token, follow these steps:

1. Log in to your Cloudflare account.
2. Go to the **Profile** section.
3. Select **API Tokens**.
4. Click on **Create Token**.
5. For this case, you only need "Edit zone DNS" permissions.
6. Once you've configured the token, click **Continue to summary** and then **Create Token**.
7. Make sure to copy your new API token and securely store it; **you won’t be able to see it again.**

{% hint style="info" %}
Refer to the [official Cloudflare documentation](https://developers.cloudflare.com/api/tokens/create) for detailed steps and information.
{% endhint %}

After you obtained your Cloudflare API Token, you need to put it in a file called `certbot.ini` in your server with format like below:

```
dns_cloudflare_api_token = 0123456789abcdef0123456789abcdef01234567
```

you can use your text editor or this command below to write it into a file directly

```sh
echo 'dns_cloudflare_api_token = 0123456789abcdef0123456789abcdef01234567' > ~/certbot.ini
```

### Generate SSL

After you finished previous steps, now we need to generate the SSL.\
You can do it by executing below command:

```sh
certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials ~/certbot.ini \
  -d *.cryptonode.id
```

{% hint style="info" %}
It will generate a wildcard SSL. If you want a subdomain specific, you can replace the \* symbol with your subdomain (e.g. crossfi-testnet-api.cryptonode.id)
{% endhint %}

{% hint style="info" %}
The default location of the certificates are `/etc/letsencrypt/live/cryptonode.id/fullchain.pem` for the public key and `/etc/letsencrypt/live/cryptonode.id/privkey.pem` for the private key
{% endhint %}

### Re-Configure NGINX to use SSL

In [routing-using-nginx.md](routing-using-nginx.md "mention"), we've already set up our domain to point to correct endpoint. But it's still using HTTP. Now, we will set it up so we can use HTTPS.

Edit your config by opening your previous config with this command

```sh
sudo nano /etc/nginx/sites-available/crossfi-testnet-api
```

And replace the config with this

```nginx
server {
    listen 80;
    server_name crossfi-testnet-api.cryptonode.id;
    return 301 https://$host$request_uri;
}
server {
    listen 443 ssl;

    server_name crossfi-testnet-api.cryptonode.id;

    ssl_certificate /etc/letsencrypt/live/cryptonode.id/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/cryptonode.id/privkey.pem;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 5m;
    ssl_ecdh_curve secp384r1;
    ssl_stapling on;
    ssl_stapling_verify on;

    add_header 'Access-Control-Allow-Methods' 'GET,POST,OPTIONS,PUT,DELETE,PATCH';
    add_header 'Access-Control-Allow-Headers' 'Authorization,Accept,Origin,DNT,X-CustomHeader,Keep-Alive,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Content-Range,Range';

    location / {
        limit_req zone=ip burst=12 delay=8;
        proxy_pass http://127.0.0.1:1317;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

Repeat the same process for other endpoints and you will have all of your endpoints with SSL enabled
