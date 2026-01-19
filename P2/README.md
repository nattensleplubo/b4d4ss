#  <ins> P2. VXLAN </ins>

# 🔹 Qu’est-ce que VXLAN ?
## 1. Défintion
**VXLAN (Virtual Extensible LAN)** est une technologie de virtualisation de reseau qui permet d’étendre **un réseau Ethernet(L2)** au-dessus d’un **réseau IP(L3)**.

**--->** **VXLAN** permet à des machines situées sur des réseaux IP différents de communiquer comme si elles étaient sur le même switch Ethernet. Vise à résoudre des problèmes d'évolutivité associés au déploiement du cloud computing.

## 2. Problème des réseaux traditionnels (Pourquoi VXLAN a été créé)
### 🔸 Limites des VLANs

Les VLANs classiques présentent plusieurs limitations :

- 🔢 **Limite de 4094 VLANs**
- 🧱 Fonctionnent uniquement en **Layer 2**
- 📉 Difficultés de **scalabilité**
- 🌳 **Spanning Tree** complexe
- 🏢 Peu adaptés aux **datacenters modernes**

---

### 🔸 Évolution des datacenters

Avec l’arrivée de :

- 🖥️ La **virtualisation**
- 📦 Les **conteneurs**
- ☁️ Le **cloud**
- 🔄 Les **migrations de VM**

👉 Il devenait nécessaire de :

- Étendre le **Layer 2 au-dessus du Layer 3**
- Disposer de **millions de réseaux isolés**
- Avoir une infrastructure **scalable et dynamique**

---

### 🔹 Problématique

Un réseau Ethernet (L2) **ne traverse pas naturellement un réseau IP (L3)**.  
Or, dans les environnements modernes (datacenters, cloud), il est parfois nécessaire de conserver un **réseau L2 étendu**, même si l’infrastructure physique est basée sur du L3.

---

## 3. VXLAN : principe général

**VXLAN (Virtual eXtensible LAN)** est une technologie qui permet d’encapsuler des **trames Ethernet (L2)** à l’intérieur de paquets **UDP/IP (L3)**.

VXLAN crée un **réseau overlay (virtuel)** au-dessus d’un **réseau underlay (physique)**.

---

### 🔹 VTEP

Un **VTEP (VXLAN Tunnel Endpoint)** est un équipement qui :
- encapsule les trames Ethernet en VXLAN
- décapsule les trames reçues
- fait le lien entre L2 (overlay) et L3 (underlay)

Dans ce projet, **Linux joue le rôle de VTEP**.

---
## 🧩 Objectif de la Partie 2 (P2)

Conformément à l’énoncé du projet BADASS, la Partie 2 consiste à mettre en place
un réseau **VXLAN sans EVPN**, en deux étapes obligatoires :

1. **VXLAN en mode statique (unicast)**  
   - Les VTEP distants sont configurés manuellement
   - Le tunnel VXLAN est point-à-point
   - Permet de comprendre le fonctionnement de base de VXLAN

2. **VXLAN en mode dynamique multicast (flood & learn)**  
   - Les VTEP ne sont plus configurés explicitement
   - La découverte des pairs se fait via un groupe multicast
   - Les adresses MAC sont apprises dynamiquement

Ces deux modes sont implémentés sur **la même topologie**, afin de comparer leurs
fonctionnements, leurs avantages et leurs limites.

Cette Partie 2 sert de base fonctionnelle avant l’introduction d’un plan de contrôle
avec **BGP EVPN** en Partie 3.

---

## 4️⃣ Fichiers de configuration : host, routeur statique et routeur dynamique

Dans la Partie 2 du projet BADASS, on distingue plusieurs types de fichiers de configuration :
- les fichiers **host**
- les fichiers **routeur statique**
- les fichiers **routeur dynamique**

⚠️ Important :  
La distinction *statique / dynamique* concerne :
- le **routage IP (Layer 3)**
- **ET** le mode de fonctionnement de VXLAN (unicast ou multicast)

Cependant, VXLAN reste **sans plan de contrôle** en P2.

---

## 4.1 Fichiers `host`

Les fichiers `host` décrivent des **machines finales**.

### 🔹 Rôle d’un host
Un host :
- génère du trafic (ping, tests)
- possède une adresse IP
- est connecté à un routeur/VTEP
- ne fait **ni routage, ni VXLAN**

Le host est **totalement passif** du point de vue réseau.

### 🔹 Contenu typique
- configuration d’interface réseau
- adresse IP
- passerelle par défaut

📌 Le host ne connaît **ni VXLAN, ni tunnels, ni topologie globale**.

---

## 4.2 Fichiers `routeur statique`

Les fichiers de **routeur statique** décrivent des routeurs IP utilisant :
- des **routes statiques**
- une configuration manuelle du routage

### 🔹 Rôle du routeur statique
Un routeur statique :
- assure le **routage IP (L3)** manuellement
- participe au **réseau underlay**
- peut aussi jouer le rôle de **VTEP VXLAN**

### 🔹 Contenu typique
- configuration des interfaces IP
- routes définies manuellement (`ip route add`)
- configuration VXLAN statique (si VTEP)

📌 Le routeur statique **ne découvre rien automatiquement**.

---

## 4.3 Fichiers `routeur dynamique`

Les fichiers de **routeur dynamique** décrivent des routeurs IP utilisant un **protocole de routage dynamique**.

### 🔹 Rôle du routeur dynamique
Un routeur dynamique :
- apprend automatiquement les routes IP
- met à jour sa table de routage
- simplifie la gestion du réseau underlay

👉 En P2, ce routage dynamique sert **uniquement au Layer 3**.

### 🔹 Contenu typique
- configuration des interfaces IP
- activation d’un protocole de routage dynamique
- aucune logique VXLAN dynamique

En P2, VXLAN est implémenté :
- soit en **mode statique unicast**
- soit en **mode dynamique multicast (flood & learn)**

Dans les deux cas, VXLAN ne dispose **d’aucun plan de contrôle** (pas d’EVPN).
---

## 4.4 Différence entre routeur statique et dynamique (P2)

| Critère | Routeur statique | Routeur dynamique |
|------|----------------|------------------|
| Type | Routage IP | Routage IP |
| Découverte des routes | Manuelle | Automatique |
| Protocole | Aucun | OSPF / RIP / équivalent |
| VXLAN | Statique | Statique |
| Rôle principal | Simple | Scalable |

---

## 4.5 Pourquoi utiliser les deux en P2 ?

La Partie 2 utilise **les deux types de routeurs** afin de :
- comparer routage manuel vs automatique
- comprendre le rôle du **routage IP sous VXLAN**
- montrer que VXLAN fonctionne **au-dessus de n’importe quel underlay L3**
- préparer la transition vers la Partie 3

---

## 4.6 Résumé clair

- **Host** : machine finale, aucune intelligence réseau
- **Routeur statique** : routage IP manuel + VXLAN statique
- **Routeur dynamique** : routage IP automatique + VXLAN statique

👉 En P2 :
- dynamique = **L3**
- VXLAN = **toujours statique**

## 📄 ROUTER – VXLAN statique (P2)

Ce fichier configure **Router 2** comme **VTEP (VXLAN Tunnel Endpoint)** dans la Partie 2 du projet BADASS.

Son objectif est de :
- participer au **réseau underlay IP (Layer 3)**
- établir un **tunnel VXLAN statique**
- étendre un **réseau Ethernet (Layer 2)** entre des hosts distants

---

## 🔹 Interface `eth0` – Underlay IP (L3)

```bash
ip link set eth0 up
ip addr add 10.1.1.2/24 dev eth0
```

- `eth0` est l’interface connectée au **réseau IP underlay**
- Une adresse IP lui est assignée pour transporter les paquets VXLAN
- Cette interface ne transporte **pas directement** le trafic Ethernet des hosts

Sans cette interface IP fonctionnelle, aucun tunnel VXLAN ne peut être établi.

---

## 🔹 Bridge `br0` – Switch Ethernet virtuel (L2)

```bash
ip link add br0 type bridge
ip link set br0 up
```

- `br0` est un **bridge Linux**
- Il se comporte comme un **switch Ethernet de niveau 2**
- Il permet de regrouper plusieurs interfaces dans un même domaine de broadcast

Le bridge est indispensable pour relier les interfaces locales et VXLAN.

---

## 🔹 Interface `vxlan10` – Tunnel VXLAN statique

```bash
ip link add vxlan10 type vxlan \
  id 10 \
  dev eth0 \
  local 10.1.1.2 \
  remote 10.1.1.1 \
  dstport 4789
```

- `vxlan10` est l’interface VXLAN
- `id 10` est le **VNI (VXLAN Network Identifier)**
- `dev eth0` indique que l’underlay utilise l’interface IP `eth0`
- `local` correspond à l’adresse IP du VTEP local
- `remote` correspond à l’adresse IP du VTEP distant, défini manuellement
- `dstport 4789` est le port UDP standard utilisé par VXLAN

Le tunnel est **statique**, ce qui signifie que les pairs VXLAN sont configurés explicitement.

---

## 🔹 Activation et intégration de VXLAN dans le bridge

```bash
ip link set vxlan10 up
ip link set vxlan10 master br0
```

- L’interface VXLAN est activée
- Elle est attachée au bridge `br0`
- Le tunnel VXLAN devient un **port du switch virtuel**

Toute trame Ethernet entrant dans le bridge peut être encapsulée dans VXLAN.

---

## 🔹 Interface `eth1` – Connexion vers le host

```bash
ip link set eth1 up
ip link set eth1 master br0
```

- `eth1` est l’interface reliée au host local
- Elle est rattachée au bridge `br0`
- Le host rejoint le **réseau Ethernet étendu**

Pour le host, la communication est totalement transparente :  
il se comporte comme s’il était connecté à un switch local.

---

## 🧠 Résumé du fonctionnement

- `eth0` : transport IP (underlay – L3)
- `vxlan10` : tunnel VXLAN statique
- `br0` : switch Ethernet virtuel
- `eth1` : interface du host

```text
Host ── eth1 ── br0 ── vxlan10 === IP underlay === vxlan10 ── br0 ── eth1 ── Host
```

Router 2 agit comme un **pont entre un réseau Ethernet local et un réseau VXLAN**, en s’appuyant sur une infrastructure IP routée.

## 📄 ROUTER  – VXLAN multicast (P2)

Ce fichier configure **Router 1** comme **VTEP (VXLAN Tunnel Endpoint)** utilisant **VXLAN en mode multicast** dans la Partie 2 du projet BADASS.

Dans ce mode :
- les VTEP ne connaissent pas leurs pairs à l’avance
- la découverte des MAC distantes se fait par **flooding**
- le multicast remplace la configuration statique des VTEP distants

---

## 🔹 Interface `eth0` – Underlay IP (L3)

```bash
ip link set eth0 up
ip addr add 10.1.1.1/24 dev eth0
```

- `eth0` est l’interface connectée au **réseau IP underlay**
- Une adresse IP lui est assignée pour transporter le trafic VXLAN
- Cette interface sert uniquement au transport IP (UDP)

Sans underlay IP fonctionnel, VXLAN ne peut pas encapsuler les trames Ethernet.

---

## 🔹 Bridge `br0` – Switch Ethernet virtuel (L2)

```bash
ip link add br0 type bridge
ip link set br0 up
```

- `br0` est un **bridge Linux**
- Il agit comme un **switch Ethernet de niveau 2**
- Il regroupe les interfaces locales et VXLAN dans un même domaine L2

Le bridge permet de relier le host local au réseau VXLAN.

---

## 🔹 Interface `vxlan10` – VXLAN multicast

```bash
ip link add vxlan10 type vxlan \
  id 10 \
  dev eth0 \
  group 239.1.1.1 \
  dstport 4789
```

- `vxlan10` est l’interface VXLAN
- `id 10` correspond au **VNI (VXLAN Network Identifier)**
- `dev eth0` indique l’interface underlay utilisée
- `group 239.1.1.1` définit l’adresse **multicast IP**
- `dstport 4789` est le port UDP standard VXLAN

En mode multicast :
- les trames inconnues sont envoyées au groupe multicast
- tous les VTEP abonnés reçoivent le trafic
- les adresses MAC sont apprises dynamiquement

---

## 🔹 Activation et intégration de VXLAN dans le bridge

```bash
ip link set vxlan10 up
ip link set vxlan10 master br0
```

- L’interface VXLAN est activée
- Elle est attachée au bridge `br0`
- Le tunnel VXLAN devient un **port du switch virtuel**

Cela permet au bridge de transmettre les trames Ethernet via VXLAN.

---

## 🔹 Interface `eth1` – Connexion vers le host

```bash
ip link set eth1 up
ip link set eth1 master br0
```

- `eth1` est l’interface connectée au host local
- Elle est rattachée au bridge `br0`
- Le host rejoint le **réseau Ethernet étendu**

Le host communique comme s’il était sur un switch local.

---

## 🧠 Résumé du fonctionnement

- `eth0` : transport IP (underlay – L3)
- `vxlan10` : VXLAN multicast (flood & learn)
- `br0` : switch Ethernet virtuel
- `eth1` : interface du host

```text
Host ── eth1 ── br0 ── vxlan10
                  ↓
            Multicast 239.1.1.1
                  ↓
           Autres VTEP VXLAN
```

Router 1 étend un réseau Ethernet Layer 2 au-dessus d’un réseau IP Layer 3 en utilisant **VXLAN multicast**, sans configuration explicite des VTEP distants.
