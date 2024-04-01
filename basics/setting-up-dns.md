# Setting up DNS

## Prerequisites

### Domain

You will need to have your own domain name first before we can continue.\
If you don't have your domain yet, you can [Get your domain here (starting from $0)](https://www.jdoqocy.com/he102cy63y5LNMNNMTNQVLNNQOVMQO?sid=cryptonode.id)

### Cloudflare

You need to [sign up for Cloudflare account](https://www.cloudflare.com/en-gb/plans/) if you don't have it already. Free plan is enough

## DNS Management

We will use Cloudflare as our DNS Manager. You can plan ahead for which DNS do you want to use.

In this case, we will use our crossfi setup as example.

### Cloudflare Setup

After you registered to Cloudflare, you will see your dashboard, click on "**Add a site**" to continue

<figure><img src="../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Fill in your domain name (example: cryptonode.id), then click continue

<figure><img src="../.gitbook/assets/image (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Cloudflare will prompt you to update your nameserver settings to their nameserver, you can set it up in your domain provider

<figure><img src="../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

After your nameserver propagated, go to DNS > Records, then click on "Add record" as shown below on the bottom right corner

<figure><img src="../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

Fill in the required details\
Select A for the Type, fill in your subdomain in Name field, and your server IPv4\
Finally, set the Proxy status to DNS only by clicking on it

<figure><img src="../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

After everything is done, it will look like this

<figure><img src="../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

Now you have set up your domain, but it won't work yet, let's continue to setup our server
