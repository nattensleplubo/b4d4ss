# BADASS PART 1

## GNS3 configuration with Docker

The main goal of this part is first of all, getting our hands on GNS3 and setting it up for it to work with docker images.

For the VM we chose **Fedora** since it's packages are up do date and easier to install than Debian.

---

The subject asks us to have two machines communicating with one another: One router and one "host" machine. Each of them will have their own docker image.

The **host** machine must have **busybox**.

The **router** must have : 
    - A software for packet routing, in our case : JSP A DEMANDER/!\
    - The service **BGPD** active and configured
    - The service **OSPFD** actuve and configured
    - An **IS-IS** routing engine service
    - **busybox** or an alternative

**busybox**: Small executable that is a shell with small but functional versions of most of the cricical system commands built into it. The intent is to provide as much critical functionality of a unix command line env. as possible in a minimal size for embedded sytems.
**BGP** : Protocol that shares reachability information along with the path a route takes (through autonomous systems). Its main purpose is to decode the best route between autonomous systems, large networks owned by ISPs, companies or cloud providers.
**BGPD**: Border Gateway Protocol routing Daemon. bgpd is a BGP daemon which manages the network routing table. Its main purpose if to exchange information concerning "network reachability" with other BGP systems
**OSPF** : Routing protocol for IP (internet protocol) networks that routers use to share information about network topology. Its used to automatically find the best path for data to travel between networks within a large organization or ISP
**OSPFD**: ospfd is an Open Shortest Path First daemon which manages routing tables.
**IS-IS**: Intermediate System to Intermediate System is a link state interior routing protocol similar in purpose to OSPF. It's used to route traffic within a large network.

---

## The Docker images
### The Router Image
```dockerfile
FROM quay.io/frrouting/frr:9.1.0

RUN apk update && apk upgrade

RUN touch /etc/frr/vtysh.conf && \
    sed -i /etc/frr/daemons -e 's/bgpd=no/bgpd=yes/' && \
    sed -i /etc/frr/daemons -e 's/ospfd=no/ospfd=yes/' && \
    sed -i /etc/frr/daemons -e 's/isisd=no/isisd=yes/'
```

### The host image
```dockerfile
FROM alpine:3.19

RUN apk update && apk add --no-cache \
    busybox \
	bash

CMD ["sh"]
```

### Lancer la simulation dans GNS3
```sh
docker build -t router:latest -f _rencarna-2 . && \
docker build -t host:latest -f _rencarna-1_host . && \
gns3 &
```

### Verifier la configuration de l'image docker sur GNS3

Importer maintenant l'image docker dans l'application
Ouvrir un terminal auxiliaire
Regarder si la configuration des composants sont bien active dans le fichier /etc/frr/daemons
