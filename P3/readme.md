# Principe
## BGP EVPN c'est quoi ?
BGP EVPN (Ethernet VPN) est une technologie de réseau qui combine BGP (Border Gateway Protocol) avec des extensions pour créer des réseaux virtuels Ethernet à grande échelle.

## Cas d'usage courants

EVPN est très utilisé dans les data centers pour interconnecter des serveurs et créer des réseaux overlay avec VXLAN (VXLAN-EVPN étant une combinaison très populaire). Les opérateurs télécoms l'utilisent aussi pour fournir des services VPN Ethernet à leurs clients entreprises, remplaçant progressivement les anciennes technologies comme VPLS.

### Comment faire le setup ?

Premierement il faudra setup la spine :
```sh
vtysh << EOF

# Mode Config

conf t
# Nom du routeur(spine)
hostname _rencarna-1
no ipv6 forwarding

# Underlay
# Lien vers leaf1
interface eth0
 ip address 10.1.1.1/30
# Lien vers leaf2
interface eth1
 ip address 10.1.1.5/30
# Lien vers leaf3
 interface eth2
 ip address 10.1.1.9/30

# Loopback
interface lo
 ip address 1.1.1.1/32

 # BGP (control plane EVPN)
router bgp 1
# Création d’un peer-group iBGP
 neighbor ibgp peer-group
# Tous les voisins sont dans le même AS (iBGP)
 neighbor ibgp remote-as 1
# Utilisation de la loopback comme source BGP
 neighbor ibgp update-source lo
bgp listen range 1.1.1.0/29 peer-group ibgp

# EVPN
address-family l2vpn evpn
 neighbor ibgp activate
 neighbor ibgp route-reflector-client
exit-address-family

# OSPF
router ospf
 network 0.0.0.0/0 area 0

 # VTY
line vty
EOF
```

Puis ensuite setup les leaves (juste pour router 2)
```sh
# Creates bridge br0 (virtual switch)
ip link add br0 type bridge
# Brings the bridge up (activates it)
ip link set dev br0 up
# Creates a VXLAN interface named vxlan10
ip link add vxlan10 type vxlan id 10 dstport 4789
# Activates the vxlan10 interface
ip link set dev vxlan10 up
# Adds vxlan10 to the bridge
brctl addif br0 vxlan10
# Adds the local physical interface (eth1) into the bridge
brctl addif br0 eth1

# Enters the FRRouting shell
vtysh
# Enters configuration mode
conf t
# Sets the host name
hostname rencarna-2
# Disables ipv6 packet forwarding
no ipv6 forwarding

# Enters configuration eth0
interface eth0
# Assigns ip address to eth0
ip address 10.1.1.2/30
# Enables OSPF on this interface 
ip ospf area 0

# Configures the loopback interface
interface lo
# Assigns a stable router id (This ip identifies this VTEP)
ip address 1.1.1.2/32
# Advertises the loopback into OSPF
ip ospf area 0

# Starts BGP
router bgp 1
# Defines a BGP neighbour 
neighbor 1.1.1.1 remote-as 1
# Forces BGP to use source ip = loopback (1.1.1.2)
neighbor 1.1.1.1 update-source lo

# Enters the EVPN BGP family
address-family l2vpn evpn
# Activates the neighbour for EVPN
neighbor 1.1.1.1 activate
# Advertises all local VNIs to BGP
advertise-all-vni
exit-address-family

# Starts the OSPF routing process
router ospf

```

Ensuite nous viendrons donner une ip a chaque hosts.

Et nous pouvons ping les hosts entre eux.