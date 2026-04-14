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

