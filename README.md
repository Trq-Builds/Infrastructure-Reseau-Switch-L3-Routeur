# ` 🌐 `︲Documentation : Infrastructure Réseau (Switch L3 & Routeur)

<p align="center">
  <img src="https://img.shields.io/badge/Cisco-ISR4221-blue?style=for-the-badge&logo=cisco&logoColor=white">
  <img src="https://img.shields.io/badge/Cisco-Catalyst_3750v2-orange?style=for-the-badge&logo=cisco&logoColor=white">
  <img src="https://img.shields.io/badge/Réseau-VLANs-green?style=for-the-badge">
  <img src="https://img.shields.io/badge/NAT-Enabled-purple?style=for-the-badge">
</p>

---

Cette documentation présente la configuration complète d’une infrastructure réseau basée sur un **routeur Cisco ISR 4221** et un **switch de couche 3 Catalyst 3750v2**.

Objectif : comprendre l’architecture, le routage, le cloisonnement réseau (VLAN) et les mécanismes de communication entre les différents segments.

---

## ` 📑 `︲Sommaire

1. [`📡`︲Présentation du routeur](#routeur)

2. [`🔁`︲Fonctionnement du NAT](#nat)
3. [`🗺️`︲Routage](#routage)

4. [`🔐`︲Sécurité du routeur](#securite-routeur)
5. [`🖧`︲Présentation du switch L3](#switch)

6. [`🗂️`︲VLANs](#vlans)

7. [`📡`︲DHCP Helper](#dhcp)

8. [`🔁`︲Routage inter-VLAN](#intervlan)

9. [`🔐`︲Sécurité du switch](#securite-switch)

10. [`⚠️`︲Analyse des faiblesses](#faiblesses)

11. [`📐`︲Schéma réseau](#schema)

---

<a id="routeur"></a>
# ` 📡 `︲Présentation du routeur

---

> [!NOTE]
> Le routeur assure la **connexion entre le réseau interne et Internet**.  
> Il agit comme point de sortie unique pour tous les équipements internes.

---

### ` ⚙️ `︲Interfaces réseau

Le routeur possède **deux interfaces principales** :

- ` 🌐 `︲**GigabitEthernet0/0/0 (WAN)**  
  → Adresse IP : `172.19.0.2`  
  → Connectée à Internet  

- ` 🏠 `︲**GigabitEthernet0/0/1 (LAN)**  
  → Adresse IP : `172.17.0.1`  
  → Connectée au switch (réseau interne)

---

> [!TIP]
> Le routeur joue exactement le même rôle qu’une **box Internet**, mais avec un contrôle total sur le routage et la sécurité.

---

<a id="nat"></a>
# ` 🔁 `︲Fonctionnement du NAT

---

> [!NOTE]
> Le NAT (**Network Address Translation**) permet à plusieurs machines privées de partager une seule adresse IP publique.

---

### ` 📡 `︲Principe

- Les machines internes utilisent des IP privées
- Le routeur remplace ces IP par **son IP publique**
- Il maintient une table de correspondance

---

### ` 📊 `︲Réseaux autorisés

- `192.168.1.0/24`
- `10.0.0.0/8`
- `172.18.0.0/16`
- `192.168.0.0/24`
- `172.17.0.0/16`

---

> [!IMPORTANT]
> Sans NAT, aucune machine interne ne peut accéder à Internet.

---

<a id="routage"></a>
# ` 🗺️ `︲Routage

---

> [!NOTE]
> Le routeur utilise des **routes statiques** pour savoir où envoyer les paquets.

---

### ` 📊 `︲Table de routage

| Destination | Passerelle |
|------------|-----------|
| Internet (défaut) | `172.19.0.1` |
| `10.0.0.0/8` | `172.17.0.255` |
| `172.18.0.0/16` | `172.17.0.255` |
| `192.168.0.0/24` | `172.17.0.255` |

---

> [!TIP]
> La route par défaut est critique :  
> sans elle → aucune sortie vers Internet.

---

<a id="securite-routeur"></a>
# ` 🔐 `︲Sécurité du routeur

---

### ` 🛡️ `︲Mesures appliquées

- ` 🔐 `︲Connexion distante via **SSH uniquement**
- ` ❌ `︲HTTP désactivé
- ` 👻 `︲CDP désactivé (pas de visibilité réseau)

---

> [!TIP]
> Désactiver CDP évite de donner des infos aux attaquants sur ton infra.

---

<a id="switch"></a>
# ` 🔻 `︲Présentation du switch L3

---

> [!NOTE]
> Le switch est le **cœur du réseau local**.  
> Il segmente le réseau via des VLANs et gère le routage interne.

---

<a id="vlans"></a>
# ` 🗂️ `︲VLANs

---

> [!NOTE]
> Les VLANs permettent de **séparer logiquement le réseau**.

---

### ` 📊 `︲Configuration

| VLAN | Ports | IP Switch | Réseau |
|------|------|----------|--------|
| VLAN 1 | Gestion | `192.168.1.250` | `192.168.1.0/24` |
| VLAN 2 | Fa1/0/1 → 4 | `10.0.0.255` | `10.0.0.0/8` |
| VLAN 3 | Fa1/0/5 → 8 | `172.18.0.255` | `172.18.0.0/16` |
| VLAN 4 | Fa1/0/9 → 12 | `192.168.0.254` | `192.168.0.0/24` |
| VLAN 5 | Fa1/0/22 | `172.17.0.255` | `172.17.0.0/16` |

---

> [!IMPORTANT]
> Chaque VLAN est un **réseau isolé**.  
> Sans routage → aucune communication entre eux.

---

<a id="dhcp"></a>
# ` 📡 `︲DHCP Helper

---

> [!NOTE]
> Le DHCP Helper permet aux VLANs d’utiliser un serveur DHCP distant.

---

### ` ⚙️ `︲Fonctionnement

- VLAN 2 et 3 → pas de serveur DHCP local
- Le switch redirige les requêtes vers :
  → `192.168.0.1` (VLAN 4)

---

> [!TIP]
> Sans DHCP Helper → aucun client ne reçoit d’IP dans ces VLANs.

---

<a id="intervlan"></a>
# ` 🔁 `︲Routage inter-VLAN

---

> [!NOTE]
> Le switch effectue le routage entre VLANs grâce à `ip routing`.

---

### ` ⚙️ `︲Principe

- Communication interne directe entre VLANs
- Sortie Internet via :
  → `172.17.0.1` (routeur)

---

> [!IMPORTANT]
> Le switch agit comme un **routeur interne**.

---

<a id="securite-switch"></a>
# ` 🔐 `︲Sécurité du switch

---

### ` 🛡️ `︲Mesures en place

- Certificat SSL auto-signé
- HTTP + HTTPS activés
- Spanning Tree (PVST)

---

> [!WARNING]
> HTTP activé = surface d’attaque inutile.

---

<a id="faiblesses"></a>
# ` ⚠️ `︲Analyse des faiblesses

---

| # | Problème | Impact |
|---|---------|-------|
| 1 | HTTP activé | Accès non sécurisé |
| 2 | Pas de mot de passe VTY | Accès distant libre |
| 3 | Pas de chiffrement | Mots de passe visibles |
| 4 | Ports ouverts | Risque d’intrusion |
| 5 | IOS obsolète | Failles potentielles |

---

> [!WARNING]
> En production, cette configuration est **clairement vulnérable**.

---

<a id="schema"></a>
# ` 📐 `︲Schéma réseau

---

```

Internet
|
[Routeur 172.17.0.1]
|
[Switch L3 - VLAN 5]
|
|── VLAN 2 → 10.0.0.0/8
|── VLAN 3 → 172.18.0.0/16
|── VLAN 4 → 192.168.0.0/24 (DHCP)
|── VLAN 1 → 192.168.1.0/24 (Gestion)

```

---

> [!TIP]
> Architecture classique :
> segmentation + routage centralisé + sortie unique.

---

## ` 🧠 `︲Conclusion

---

> [!NOTE]
> Cette infrastructure met en place :

- Cloisonnement réseau via VLANs
- Routage inter-VLAN efficace
- Accès Internet via NAT
- Centralisation du trafic

---

> [!IMPORTANT]
> Base solide pour un réseau d’entreprise…  
> mais **sécurité à corriger avant toute mise en production**.

---

