# ` 🌐 `︲Infrastructure Réseau (Switch L3 & Routeur)

<p align="center">
  <img src="https://img.shields.io/badge/Cisco-ISR4221-blue?style=for-the-badge&logo=cisco&logoColor=white">
  <img src="https://img.shields.io/badge/Cisco-3750v2_L3-orange?style=for-the-badge&logo=cisco&logoColor=white">
</p>

---

## `  🌐  ` ︲ `  🗺️  `︲Shéma Réseau

```mermaid
graph LR
    subgraph Internet
    WAN["WAN<br/>172.16.0.0/16"]
    end

    WAN -- "DHCP <--> Port 1 (out)" --> FW[("Stormshield")]

    subgraph Management
    FW -- "Port 2 <--> Réseau" --> ADMIN["Admin<br/>172.30.0.0"]
    end

    subgraph LAME[Lame Serveur]
    NIC_DMZ["Carte réseau<br/>Haut à Droite (DMZ)"]
    NIC_LAN["Carte réseau<br/>Bas à Droite (LAN)"]
    end

    subgraph DMZ_Zone[Zone DMZ]
    FW -- "Port 3 <--> 192.168.0.0/24" --> NIC_DMZ
    end

    FW -- "Port 4 <--> Port 0/0/0<br/>Réseau: 172.19.0.2/16" --> R1(("Routeur Cisco<br/>.1"))
    
    R1 -- "Port 0/0/1 <--> Port Fa1/0/22<br/>VLAN 5<br/>" --> SW["Switch L3<br/>.254"]

    subgraph LAN_Segmentation[Segmentation LAN & Affectation des Ports]
    direction TB
    SW --- V1["VLAN 1<br/>192.168.1.0/24<br/>(Aucun port affecté)"]
    SW --- V2["VLAN 2<br/>10.0.0.0/8<br/>Ports: Fa1/0/1 à 1/0/4"]
    SW --- V3["VLAN 3<br/>172.18.0.0/16<br/>Ports: Fa1/0/5 à 1/0/8"]
    SW --- V4["VLAN 4 (LAN)<br/>192.168.0.0/24<br/>Ports: Fa1/0/9 à 1/0/12"]
    end

    V4 --- NIC_LAN

    style WAN fill:#f9f,stroke:#333,stroke-width:2px
    style FW fill:#f66,stroke:#333,stroke-width:2px
    style R1 fill:#69f,stroke:#333,stroke-width:2px
    style SW fill:#9f9,stroke:#333,stroke-width:2px
    style V4 fill:#dfd,stroke:#333
    style ADMIN fill:#eee,stroke:#333,stroke-dasharray: 5 5
    style LAME fill:#fff2cc,stroke:#d6b656,stroke-width:2px
```

---

## ` 📌 `︲Vue rapide

```

Internet
|
[ISR4221]
|
[SW L3 3750]
|
|── VLAN 2 → 10.0.0.0/8
|── VLAN 3 → 172.18.0.0/16
|── VLAN 4 → 192.168.0.0/24 (DHCP)
|── VLAN 1 → 192.168.1.0/24 (Mgmt)

```

---

## ` 📡 `︲Routeur (ISR4221)

### Interfaces
- `Gi0/0/0` → WAN  
  `172.19.0.2` → Gateway `172.19.0.1`

- `Gi0/0/1` → LAN  
  `172.17.0.1` → vers switch (VLAN 5)

---

### NAT
- NAT overload actif côté WAN (`Gi0/0/0`)
- Réseaux autorisés :
  - `10.0.0.0/8`
  - `172.18.0.0/16`
  - `192.168.0.0/24`
  - `192.168.1.0/24`
  - `172.17.0.0/16`

---

### Routage
- Default route → `172.19.0.1`
- Routes internes : (Permettent le retour vers le Switch de niveau 3)
  - `10.0.0.0/8` → `172.17.0.255`
  - `172.18.0.0/16` → `172.17.0.255`
  - `192.168.0.0/24` → `172.17.0.255`

⚠️ Next-hop en `.255` (broadcast) → à vérifier selon design réel

---

### Accès / sécurité
- SSH only
- HTTP off
- CDP off

---

## ` 🖧 `︲Switch L3 (3750v2)

### VLANs

| VLAN | IP | Réseau | Ports |
|------|--------|--------|------|
| 1 | 192.168.1.250 | Mgmt | - |
| 2 | 10.0.0.255 | 10.0.0.0/8 | Fa1/0/1-4 |
| 3 | 172.18.0.255 | 172.18.0.0/16 | Fa1/0/5-8 |
| 4 | 192.168.0.254 | 192.168.0.0/24 | Fa1/0/9-12 |
| 5 | 172.17.0.255 | 172.17.0.0/16 | Fa1/0/22 (uplink routeur) |

---

### Routage
- `ip routing` actif
- Default route → `172.17.0.1` (routeur)

---

### DHCP
- Serveur DHCP : Sur serveur AD → `192.168.0.1` (VLAN 4)
- Helper configuré sur :
  - VLAN 2 : 10.0.0.1 à 10.0.0.254, Passerelle : 10.0.0.255, Serveur DNS : 192.168.0.1
  - VLAN 3 : 10.0.0.1 à 10.0.0.254, Passerelle : 10.0.0.255, Serveur DNS : 192.168.0.1

---

### Accès
- HTTP / HTTPS actifs
- Certificat auto-signé

---

## ` 🔁 `︲Flux réseau

### Interne
- Routage inter-VLAN assuré par le switch

### Cheminement d'un client pour acceder à Internet :
1. Client → VLAN
2. Switch → routeur (`172.17.0.1`)
3. NAT → sortie WAN

---

---
