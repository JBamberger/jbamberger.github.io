---
layout: post
title:  "Website with Flask, Gunicorn and Nginx on CentOS"
date:   2018-01-22 13:35:00 +0200
categories: python web flask gunicorn nginx
---

This page contains info on the setup of a Flask application with Gunicorn as app
server and nginx as webserver. The app will run with Python 3 on CentOS 7.

## App setup

Install Python 3 with the following commands:

```sh
sudo yum -y update
sudo yum -y install yum-utils
sudo yum -y groupinstall development
sudo yum -y install https://centos7.iuscommunity.org/ius-release.rpm # community repo
sudo yum -y install python36u # use whatever version fits your needs
sudo yum -y install python36u-pip
sudo yum -y install python36u-devel
```

Create a virtual environment and install dependencies as well as gunicorn:

```sh
python3.6 -m venv <appdir-name>
source <appdir-name>/bin/activate # enable the environment
# if you have requirements.txt use it, otherwise install the dependencies manually
pip install -r <appdir-name>/requirements.txt
pip install gunicorn
deactivate # quit the virtual environment
```

## Configure Nginx

Edit your nginx configuration (most likely in `/etc/nginx/nginx.conf`) and add
the following server:

```nginx
server {
    listen 80;
    client_max_body_size 4G;

    server_name <your-domain> www.<your-domain>;

    keepalive_timeout 5;

    root <path-to-app>/static; # path to wherever you keep static files

    location / {
        # serve static files directly and use app as fallback if not found
        try_files $uri @proxy_to_app;
    }

    location @proxy_to_app {
        # setup reverse proxy to gunicorn
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://unix:/run/gunicorns/socket;
    }

    # serve error pages from static files
    error_page 500 502 503 504 /500.html;
    location = /500.html {
        root <path-to-app>/static;
    }
}
```

## Configure systemd

Create a new systemd socket file at `/etc/systemd/system/gunicorn.socket` with
the following content.

```conf
[Unit]
Desctiption=gunicorn socket

[Socket]
ListenStream=/run/gunicorns/socket

[Install]
WantedBy=sockets.target
```

Now create the systemd service file at `/etc/systemd/system/gunicorn.service`
and add the following content:

```conf
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
PIDFile=/run/gunicorn/pid
User=<server-user>
Group=<server-group>
RuntimeDirectory=gunicorn
WorkingDirectory=<path-to-app>
ExecStart=<path-to-app>/bin/gunicorn  app:app  \
        --pid /run/gunicorn/pid   \
        --bind unix:/run/gunicorns/socket
ExecReload=/bin/kill -s HUP $MAINPID
ExecStop=/bin/kill -s TERM $MAINPID
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```

Start and enable gunicorn:

```sh
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```

## Gunicorn logs

You can display the logs with `sudo journalctl -u gunicorn`.

## References

- [How To Install Python 3 and Set Up a Local Programming Environment on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-install-python-3-and-set-up-a-local-programming-environment-on-centos-7)
- [How To Set Up Django with Postgres, Nginx, and Gunicorn on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-set-up-django-with-postgres-nginx-and-gunicorn-on-ubuntu-18-04)
- [Deploying Gunicorn](https://docs.gunicorn.org/en/stable/deploy.html)
