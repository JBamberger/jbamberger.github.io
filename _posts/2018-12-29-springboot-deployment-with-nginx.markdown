---
layout: post
title:  "Spring Boot deployment with nginx"
date:   2018-12-29 16:05:00 +0200
categories: Development
tags: Java Spring Boot Gradle Nginx CentOS systemd
---

This post will show how to configure a CentOS server for deploying a Spring Boot
application build with Gradle. The networking is passed trough nginx which can
be used to run multiple servers on the same host or serve static traffic faster.

## Configure nginx

To configure nginx add the following server declaration to your
`/etc/nginx/conf.d/nginx.conf` file.

```nginx
server {
        listen 80;
        listen [::]:80;

        server_name <your-domain>;

        location / {
             proxy_pass http://localhost:8080/;
             proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
             proxy_set_header X-Forwarded-Proto $scheme;
             proxy_set_header X-Forwarded-Port $server_port;
        }
}
```

This will pass the traffic through to the locally running app server.

Then test the configuration and restart nginx.

``` sh
sudo nginx -t # test the new configuration for valid syntax
sudo systemctl restart nginx
```

## Configure deployment in Gradle

First off, we need to define the server information and credentials. Log into
the server and create a user `deployment`. Assign the `sudo` privilege to the
newly created user. Then generate a SSH key `deployment` locally and add it to
the allowed keys on the server.

Edit the user `gradle.properties` file at `~/.gradle/gradle.properties` (or
create it if it doesn't exist) and add the following:

```conf
server_key_deployment_passphrase=<deployment-ssh-key-passphrase>
server_user_deployment_passphrase=<deployment-user-passphrase>
```

Now we need to set up Gradle to deploy the application. To do so we add the
`org.hidetake.ssh` plugin which allows us to perform SSH operations:

```groovy
plugins {
    id 'org.hidetake.ssh' version '2.9.0'
}
```

Then we configure our server access properties:

```groovy
remotes {
    webServer {
        host = '<your-ip>'
        user = 'deployment'
        port = <your-ssh-port>
        identity = file("${System.properties['user.home']}/.ssh/deployment")
        passphrase = server_key_deployment_passphrase
        sudoPassword = server_user_deployment_passphrase
        errorStream = System.err
        outputStream = System.out
    }
}
```

Note how the passphrases are loaded from the properties file, such that they do
not end up in version control by accident.

Next we define the build and deployment directories:

```groovy
def remote_dir = '/var/www/<yourapp>'        // server deployment location
def local_dir = 'build/libs'                 // local artefact output directory
def deploy_file = "<yourapp>-${version}.jar" // artefact name
```

Now we add a new task to perform the deployment. The task requires that the
artefact is build and checked (i.e. test cases are ran). Then it connects to the
server and pushes the newly built file. The deployed application versions exist
all side by side. The currently active one is linked to `<yourapp>.jar`. Then
the app server is restarted.

```groovy
task('deploy') {
  dependsOn build, test, check
  doLast {
    ssh.run {
      session(remotes.webServer) {
        put from: "${projectDir}/${local_dir}/${deploy_file}", into: remote_dir
        execute "ln -sf ${remote_dir}/${deploy_file} ${remote_dir}/<yourapp>.jar"
        executeSudo('systemctl restart <yourapp>.service', pty: true)
      }
    }
  }
}
```

## Register the application with systemd

The new application needs to be registered with systemd such that it can be
controlled easily. To do so, create a unit file at
`/etc/systemd/system/<yourapp>.service` and write the following content inside.

```config
[Unit]
Description=Spring Boot Application
After=syslog.target
After=network.target[Service]
User=<serveruser>
Type=simple

[Service]
ExecStart=/usr/bin/java -jar /var/www/<yourapp>/<yourapp>.jar
Restart=always
StandardOutput=syslog
StandardError=syslog
SyslogIdentifier=<yourapp>

[Install]
WantedBy=multi-user.target
```

Then start the app server:

```bash
sudo systemctl start <yourapp>
sudo systemctl status <yourapp>
```

## Perform a local test

Build the uber-jar containing all of the necessary jars, including the app
server (e.g. Tomcat) by invoking

```sh
./gradlew build
```

Now run it with `java -jar <yourapp>.jar` and test it.

## Perform the deployment

To perform a deployment to the server you need to invoke the new Gradle task `deploy`:

```sh
./gradlew deploy
```

## References

- [Deploy Spring Boot Applications with an NGINX Reverse Proxy](https://www.linode.com/docs/development/java/how-to-deploy-spring-boot-applications-nginx-ubuntu-16-04/)
