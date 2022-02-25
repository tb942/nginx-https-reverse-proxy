# Local nginx https reverse proxy

## Introduction

This repo is the config for my local nginx instance that provides https to my [portainer instance](https://github.com/tb942/Portainer-config) and my [grafana internet monitoring instance](https://github.com/tb942/internet-monitoring/tree/personal-use).

This nginx instance is set up for the following domains to be DNS pointed to it:

* `portainer.lan`
* `speed.lan`

I do this forwarding using pihole (ofc).

To facilitate the https you need certificates for the domains from a trusted CA. I run my own local CA with [mkcert](https://github.com/FiloSottile/mkcert) which works well for me.

The certificates and private key need putting in a `./certs` directory. They are then mounted into the container in the `/run/secrets` directory using docker secrets.

The `server` blocks are configured to only listen on the 192.168.0.192 IP and their relevant ports. This means that nothing on the `main` network can talk to it. If you change the IP that it's going to live on then you need to replace this IP in the nginx.conf file.

## Setup

### Public IP address

I don't want this to just exist on a port on my raspberry pi's IP, so I created a macvlan network so that the container gets it's own IP address.

To do this run:

```bash
docker network create -d macvlan \
  --subnet=192.168.0.0/24 \
  --ip-range=192.168.0.192/26 \
  --gateway=192.168.0.1 \
  -o parent=eth0 pub_net
```

This assumes that you are using the `192.168.0.0/24` subnet, with `192.168.0.1` as the gateway, and you should set `192.168.0.191` as the upper limit of your DHCP address range to prevent conflicting IP allocations.

### Internal services network

I don't want the other services to be accessible without going through this nginx instance, so I run them on a shared external docker network and don't expose any ports publicly.

I use the network name `main`, you can call it whatever you want.

```bash
docker network create main
```

This is the same network that my [internet monitoring](https://github.com/tb942/internet-monitoring/tree/personal-use) and [portainer](https://github.com/tb942/Portainer-config) instances exist on.

### Deployment

```bash
docker-compose up -d
```

### Updating

```bash
docker-compose pull
docker-compose up -d
```