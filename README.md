# Doc-Technique-SwitchL3-Routeur

---

## Explication sur le fichier "`config-router-complet`"

C'est la configuration d'un **routeur Cisco** (modèle ISR 4221) qui fait le lien entre un réseau interne et Internet.

---

## Ce que fait le routeur

### 1. 🌐 Connexion Internet
Le routeur a **deux "portes"** :
- Une porte **vers Internet** (`GigabitEthernet0/0/0`) avec l'IP `172.19.0.2`
- Une porte **vers le réseau local** (`GigabitEthernet0/0/1`) avec l'IP `172.17.0.1`

---

### 2. 🔁 Le NAT (Partage de connexion)
Le routeur fait du **NAT**, c'est comme la box Internet à la maison.

> Tous les appareils du réseau local partagent **une seule adresse IP publique** pour aller sur Internet

Les réseaux qui ont le droit de passer par Internet :
- `192.168.1.0/24`
- `10.0.0.0/8`
- `172.18.0.0/16`
- `192.168.0.0/24`
- `172.17.0.0/16`

---

### 3. 🗺️ Les routes (GPS du routeur)
Le routeur sait où envoyer les données grâce à des **routes statiques** :

| Destination | Direction |
|-------------|-----------|
| Tout le reste (Internet) | `172.19.0.1` |
| Réseau `10.x.x.x` | `172.17.0.255` |
| Réseau `172.18.x.x` | `172.17.0.255` |
| Réseau `192.168.0.x` | `172.17.0.255` |

---

### 4. 🔐 La sécurité
- La connexion à distance se fait uniquement en **SSH** (connexion chiffrée)
- Le **HTTP** (non sécurisé) est désactivé
- Le **CDP** est désactivé (le routeur ne se présente pas aux autres équipements)

---

## C'est quoi le fichier "`config-switch-complet`" ?

C'est la configuration d'un **switch Cisco** (modèle Catalyst 3750v2-24TS) qui est le **cœur du réseau local**. Il découpe le réseau en plusieurs zones séparées appelées **VLANs**.

---

## Ce que fait le switch

### 1. 🗂️ Les VLANs (Les zones du réseau)
Le switch divise le réseau en **5 zones distinctes** :

| VLAN | Ports concernés | Adresse IP du switch | Réseau |
|------|----------------|---------------------|--------|
| VLAN 1 | (gestion) | `192.168.1.250` | `192.168.1.0/24` |
| VLAN 2 | Fa1/0/1 → 1/0/4 | `10.0.0.255` | `10.0.0.0/8` |
| VLAN 3 | Fa1/0/5 → 1/0/8 | `172.18.0.255` | `172.18.0.0/16` |
| VLAN 4 | Fa1/0/9 → 1/0/12 | `192.168.0.254` | `192.168.0.0/24` |
| VLAN 5 | Fa1/0/22 | `172.17.0.255` | `172.17.0.0/16` |

---

### 2. 📡 Le DHCP Helper (Distributeur d'adresses IP)
Les VLANs 2 et 3 utilisent un **serveur DHCP distant** à l'adresse `192.168.0.1`

> Concrètement, quand un appareil se connecte sur ces ports, il demande une adresse IP. Le switch **redirige cette demande** vers le serveur DHCP qui se trouve dans le VLAN 4

---

### 3. 🗺️ Le Routage
- Le switch fait du **routage entre les VLANs** (`ip routing` activé)
- Toutes les connexions vers l'extérieur passent par `172.17.0.1` (le routeur)

---

### 4. 🔐 La Sécurité
- Possède un **certificat SSL auto-signé** pour les connexions sécurisées
- Le **HTTP et HTTPS** sont activés (interface web d'administration)
- Utilise le **Spanning Tree (PVST)** pour éviter les boucles réseau

---

## ⚠️ Points faibles notés

| # | Problème | Détail |
|---|----------|--------|
| 1 | **HTTP activé** | Connexion non chiffrée possible sur l'interface web |
| 2 | **Pas de mot de passe** | Accès VTY sans authentification (`login` sans password) |
| 3 | **Pas de chiffrement** | `no service password-encryption` |
| 4 | **Ports inutilisés** | Les ports `Fa1/0/13` à `Fa1/0/24` (sauf 22) ne sont pas désactivés ⚠️ |
| 5 | **IOS ancienne** | Version `12.2` très ancienne, potentiellement des failles de sécurité |

---

## Schéma simplifié

```
Internet
   |
[Routeur 172.17.0.1]
   |
[Switch - VLAN 5 : 172.17.0.255]
   |
   |── VLAN 2 : Réseau 10.0.0.0/8     (Ports 1-4)
   |── VLAN 3 : Réseau 172.18.0.0/16  (Ports 5-8)
   |── VLAN 4 : Réseau 192.168.0.0/24 (Ports 9-12) ← Serveur DHCP
   |── VLAN 1 : Réseau 192.168.1.0/24 (Gestion)
```

---

