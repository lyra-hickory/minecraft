# Server Setup

This page is for documenting the server setup, file locations, server settings, etc etc...

Basically the goal is that following this page someone unfamiliar with the project should be able to replicate it.


## Index

[Requirements and Setup](#requirements)


[Service File](#service-file)


[DDNS Setup](#ddns)


## Getting Started

### System Setup

#### Assumptions:
- working linux distro
- internet access
- root access


### Requirements:

- `openjdk` | Latest : `25.0.4`
    - `apt install openjdk-25-jdk`

- `papermc`   | Latest : `26.2#112`
    - [papermc download link](https://papermc.io/downloads/paper)


### Server Root:

Since this is an independent service that will be running I am putting it here:

```
/opt/minecraft/
```

### Setup:

1. Create our working directory

    mkdir /opt/minecraft

2. Move Paper into this folder

    mv paper.jar /opt/minecraft

3. First run to build out the directory

    java -Xmx2G -Xms2G -jar paper.jar nogui

4. Accept EULA, update [server.properties](../configs/server.properties) to match linked file

5. Setup the [System Service](#service-file)

6. Now you can run the server as a service and it should just work from here on~

    systemctl start minecraft

7. Done!


# Service File

Currently the service file lives here:

```
/etc/systemd/system/minecraft.service
```

TODO: enable the service file directly with `systemctl enable /opt/minecraft/minecraft.service`?

## Overview

So a quick overview, this is the setup for managing the systemd process

## Management commands reference

Start / Stop

```
systemctl start minecraft
```

```
systemctl stop minecraft
```

Actively logs the output

```
journalctl -u minecraft -f
```

Reloads the daemon

```
systemctl daemon-reload
```

Symlink

```
systemctl enable minecraft
```

## User setup

```
useradd -r -m -s /bin/false minecraft
```

```
chown -R minecraft:minecraft /opt/minecraft
```

## Config

You can view the minecraft service file here: [minecraft.service](../configs/minecraft.service)


# DDNS

A quick overview of the DDNS setup.

## cloudflare-ddns

I opted to run the ddns daemon in a docker compose:

```
/opt/cloudflare-ddns/docker-compose.yml
```

Config:

```
services:
  cloudflare-ddns:
    image: favonia/cloudflare-ddns:latest
    network_mode: host
    restart: always
    user: "1000:1000"
    environment:
      - CLOUDFLARE_API_TOKEN=${CLOUDFLARE_API_TOKEN}
      - DOMAINS=play.puppy-den.xyz
      - PROXIED=false
      - UPDATE_CRON=@every 5m
      - IP6_PROVIDER=none
```

Also add the `.env` file with the `CLOUDFLARE_API_TOKEN` in the same folder

```
docker compose up -d
```

That's it! Check if it ran:

```
docker compose logs
```

Should auto start on boot now as well

