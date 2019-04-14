---
layout: post
title:  "LetsEncrypt with Certbot and Cloudflare"
date:   2019-01-19 22:13:00 +0200
categories: lets-encrypt certbot cloudflare
---

This page shows how to configure [Certbot](https://certbot.eff.org/) with a domain that uses Couldflare DNS. This allows us to issue wildcard certificates.

It assumes, that you've already set up your server and can establish an ssh connection. My server is running CentOS, so you'll have to replace `yum` with the appropriate package manager for your system, for Ubuntu this it is `apt-get`.

If you haven't installed Certbot go to <https://certbot.eff.org/> and follow the instructions.

## Cloudflare-DNS plugin for Certbot

Next, we'll need to install the Cloudflare DNS plugin for Certbot:

First, we need the Python package manager pip.

```bash
sudo yum install python-pip
```

which we can use to install the Certbot plugin

```bash
sudo pip install certbot-dns-cloudflare
```

## Set up the cerdentials file

Create a new file `cloudflare.ini` in `~/.secrets/certbot/cloudflare.ini` and write your Cloudflare account credentials down as follows:

```ini
# Cloudflare API credentials used by Certbot
dns_cloudflare_email = <your-email-address>
dns_cloudflare_api_key = <your-api-key>
```

You can find your Cloudflare API key by logging into Cloudflare, and then going to _My Profile_. There you can copy the _Global API Key_.

Because these credentials can be used to perform arbitrary actions on your Cloudflare account, it is essential to set the correct access permissions. You can restrict the access to your account by running `chmod 600 ~/.secrets/certbot/cloudflare.ini`. This will also remove write permission for you, so make sure to finish editing beforehand.

## Issuing the certificate

If you are using Certbot / LetsEncrypt for the first time, you'll be asked to create an account in the next step. Just follow the steps and everything should work.

To issue the certificate one command is sufficient:

```bash
sudo certbot certonly \
    --dns-cloudflare \
    --dns-cloudflare-credentials ~/.secrets/certbot/cloudflare.ini \
    --preferred-challenges dns-01 \
    -d <cert-names>
```

Replace the last line with all names you want to be included in the certificate. You can separate multiple entries with `,`.

Example: `example.com, www.example.com, something.example.com`

A wildcard can be used by including an `*` in the name: `*.example.com`.

The active certificate will be linked into `/etc/letsencrypt/live`. You can now configure your server to use the certificate.

## Automatic renewal

Because LetsEncrypt certificates are valid for only 90 days, it is a good idea to schedule the certificate renewal automatically, such that there is always a valid certificate available.
This can be done for example by creating a cron job:

Run `crontab -e` and then enter the following line:

```bash
0 */12 * * * root test -x /usr/bin/certbot -a \! -d /run/systemd/system && perl -e 'sleep int(rand(3600))' && certbot -q renew
```

## Useful links

- LetsEncrypt: <https://letsencrypt.org/>
- Certbot: <https://certbot.eff.org/>
- Cloudflare DNS plugin: <https://certbot-dns-cloudflare.readthedocs.io/en/stable/>
- Test your SSL configuration: <https://www.ssllabs.com/ssltest/>