# Routing using NGINX

## Prerequisites

You will need NGINX installed and your Node running before you continue with this.\
For NGINX, it is already covered in [server-preparation.md](server-preparation.md "mention")\
For Nodes, you can look for the guides in [testnets](../testnets/ "mention") section

## Create per subdomain NGINX config

In this section, we will use [crossfi](../testnets/crossfi/ "mention") as our reference project

### API

#### Creating Your NGINX Configuration File

The first step involves creating a new configuration file specific to your subdomain. This file will dictate how NGINX handles incoming requests to your API. Use the following command to create a new configuration file for your project's API subdomain:

```sh
sudo nano /etc/nginx/sites-available/crossfi-testnet-api
```

This command opens up a text editor within your terminal. If you prefer a different text editor, feel free to substitute `nano` with your editor of choice, such as `vim` or `gedit`.

#### Configuring NGINX for Your API Subdomain

With the file open, it's time to input the necessary configuration for your API subdomain. Below is a template specifically designed for a Node.js application. Make sure you understand each directive and modify it to suit your application's specific needs, especially the `proxy_pass` directive, which should point to your application's running port:

```nginx
server {
    listen 80;
    server_name crossfi-testnet-api.cryptonode.id;

    location / {
        add_header Access-Control-Allow-Origin *;
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Expose-Headers Content-Length;
        
        proxy_pass http://127.0.0.1:1317;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

{% hint style="info" %}
Change `cryptonode.id` in `server_name` section to your own domain and the `proxy_pass` to your API port, in this case default port `1317`
{% endhint %}

This configuration sets up your server to listen on port 80, which is the default port for HTTP. The `server_name` should match your project's subdomain. The `proxy_pass` directive is crucial as it tells NGINX where to forward requests that hit this subdomain. In this example, it's set to the localhost on port 1317, which is the default port for many Node.js applications.

### RPC

#### Creating Your NGINX Configuration File

Just like configuring the API subdomain, setting up an RPC subdomain requires a similar approach but points to a different server application port. This setup is crucial for applications that interact with your server using the Remote Procedure Call (RPC) protocol. Below is a tailored NGINX configuration for an RPC server setup. Remember to modify the `proxy_pass` directive to match your RPC server's listening port:

```sh
sudo nano /etc/nginx/sites-available/crossfi-testnet-rpc
```

#### Configuring NGINX for Your RPC Subdomain

```nginx
server {
    listen 80;
    server_name crossfi-testnet-rpc.cryptonode.id;
    
    location / {
        proxy_pass http://127.0.0.1:26657;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

{% hint style="warning" %}
Again, change `cryptonode.id` to your own domain and the `proxy_pass` to your RPC port, in this case default port 26657
{% endhint %}

### JSONRPC

#### Creating Your NGINX Configuration File

Below is a tailored NGINX configuration for an JSONRPC server setup. Remember to modify the `proxy_pass` directive to match your JSONRPC server's listening port:

```sh
sudo nano /etc/nginx/sites-available/crossfi-testnet-jsonrpc
```

#### Configuring NGINX for Your JSONRPC Subdomain

```nginx
server {
    listen 80;
    server_name crossfi-testnet-jsonrpc.cryptonode.id;
    
    location / {
        proxy_pass http://127.0.0.1:8545;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

{% hint style="danger" %}
Again, change `cryptonode.id` to your own domain and the `proxy_pass` to your JSONRPC port, in this case default port 8545
{% endhint %}

## Activating Your Configuration

After tailoring the configuration file to your needs and saving your changes, the next step is linking this file to the `sites-enabled` directory to activate it:

```sh
sudo ln -s /etc/nginx/sites-available/crossfi-testnet-* /etc/nginx/sites-enabled/
```

This command creates a symbolic link between the `sites-available` and `sites-enabled` directories, effectively enabling your configuration.

Finally, test the NGINX configuration for syntax errors:

```sh
sudo nginx -t
```

If the test passes without issues, reload NGINX to apply the changes:

```sh
sudo systemctl reload nginx
```

Your subdomains are now set up and should be accessible via the specified subdomain. This setup enhances your project's structure by clearly distinguishing between different parts of your application, making it easier for developers and users to interact with your endpoints.

But, all of your endpoints still using HTTP which is not secure. We will setup SSL for all of our endpoint in the next section
