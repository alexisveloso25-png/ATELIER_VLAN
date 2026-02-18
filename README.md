# 🧩 TP VLAN & Masques — Comprendre la segmentation réseau

> Atelier pratique pour comprendre le rôle des VLAN et des masques IP dans la segmentation d’un réseau.

---

# 🎯 Objectifs pédagogiques

À la fin de ce TP, vous devrez savoir :

✅ Expliquer le rôle d’un VLAN  
✅ Comprendre le lien VLAN ↔ sous-réseau IP  
✅ Calculer et utiliser des masques IP  
✅ Mettre en place un trunk 802.1Q  
✅ Tester la communication intra/inter-VLAN

---

# 🧠 Rappel théorique simple

## 📌 VLAN
Un VLAN est un **réseau logique** sur un même switch physique.

👉 Chaque VLAN = un domaine de broadcast distinct  
👉 Chaque VLAN = un sous-réseau IP différent

---

## 📌 Masque de sous-réseau
Le masque définit :
- La partie réseau  
- La partie hôte  

| Masque | Nb hôtes |
|------|---------|
   /24 | 254 |
   /25 | 126 |
   /26 | 62 |

---

# 🗺️ Topologie du TP

```
         VLAN 10         VLAN 20
        192.168.10.0/24 192.168.20.0/24

PC1 -------- SW1 -------- R1 -------- PC3
              |  (trunk)
              |
             PC2
```

---

# 🧩 Matériel Packet Tracer

- 1 routeur (2911)  
- 1 switch (2960)  
- 3 PC  

---

# 🌐 Plan d’adressage

## VLAN 10

| Équipement | IP |
|----------|------|
PC1 | 192.168.10.10/24 |
PC2 | 192.168.10.20/24 |
GW | 192.168.10.1 |

---

## VLAN 20

| Équipement | IP |
|----------|------|
PC3 | 192.168.20.10/24 |
GW | 192.168.20.1 |

---

# 🔹 PARTIE 1 — Création des VLAN

Sur le switch :

```
enable
conf t
vlan 10
name ADMIN
vlan 20
name USERS
```

---

# 🔹 PARTIE 2 — Affectation des ports

```
interface f0/1
switchport mode access
switchport access vlan 10

interface f0/2
switchport mode access
switchport access vlan 10

interface f0/3
switchport mode access
switchport access vlan 20
```

---

# 🔹 PARTIE 3 — Trunk vers le routeur

```
interface f0/24
switchport mode trunk
```

---

# 🔹 PARTIE 4 — Router-on-a-stick

Sur R1 :

```
interface g0/0.10
encapsulation dot1Q 10
ip address 192.168.10.1 255.255.255.0

interface g0/0.20
encapsulation dot1Q 20
ip address 192.168.20.1 255.255.255.0

interface g0/0
no shutdown
```

---

# 🔹 PARTIE 5 — Configuration IP des PC

Configurer IP + passerelle selon le plan d’adressage.

---

# 🧪 TESTS

## Test 1 — Intra-VLAN
PC1 → PC2  
👉 Doit fonctionner

* * <img width="1247" height="512" alt="image" src="https://github.com/user-attachments/assets/7f4b4faa-aaa0-47e6-a571-3656d380411c" />
 * *  

---

## Test 2 — Inter-VLAN
PC1 → PC3  
👉 Fonctionne uniquement grâce au routeur

* * <img width="1194" height="536" alt="image" src="https://github.com/user-attachments/assets/d3b94398-bf8b-4e08-aff4-0768c7b459f2" />
 * *  
  
---

# ❓ Questions de réflexion

1. Pourquoi PC1 ne voit-il pas PC3 sans routeur ? ->
   
   Parce qu'ils sont dans des VLAN différents et des sous-réseaux différents. Un switch seul ne sait pas faire passer des données d'un VLAN à un autre (il les isole totalement). Il faut un routeur pour faire le "pont" entre les deux réseaux IP.

2. Quel rôle joue le masque /24 ? ->

   Il sert à définir la taille du réseau. Le /24 (255.255.255.0) indique que les trois premiers nombres de l'IP (ex: 192.168.10) correspondent au nom du réseau, et le dernier nombre au PC. Cela permet au PC de savoir si une adresse est dans son réseau ou s'il doit passer par la passerelle.
   
3. Que se passe-t-il si VLAN 10 et VLAN 20 ont le même réseau IP ? ->

   Ça crée un conflit de routage. Le routeur ne pourra pas configurer deux interfaces avec le même réseau (il affichera une erreur). Même sans routeur, les PC ne se verraient pas car le switch bloquerait la communication entre les deux VLANs.
   
4. Pourquoi un trunk est-il nécessaire ? ->

   Parce qu'on a plusieurs VLANs mais un seul câble entre le switch et le routeur. Le Trunk permet de faire passer tous les VLANs sur ce même câble en le (tagging 802.1Q) chaque paquet pour que le routeur sache à quel VLAN il appartient.

---

# ⭐ Travail sur les Masques

Changer VLAN 10 en :

```
192.168.10.0/25
```
Capture de changement du masque vlan 10

* * <img width="522" height="147" alt="image" src="https://github.com/user-attachments/assets/fa1d906b-59d6-4078-888d-3f9978bb4754" /> * *
 

Questions :
- Combien d’hôtes max ?   Il y a 128 adresses au total (2*7) dont ce qui donne 126 hôtes utilisables.
- Quelle plage IP valide ?  La plage va de 192.168.10.1 à 192.168.10.126.
- Peut-on encore communiquer avec VLAN 20 ?
  Oui, tant que le routeur est configuré avec la nouvelle passerelle en /25. Le changement de masque dans un VLAN n'empêche pas le routage vers un autre VLAN, car c'est le routeur qui fait le lien entre les deux.

---

# 🚀 Extensions

- Ajouter VLAN 30  
- Mettre un DHCP par VLAN

  Vérification : Le PC a bien reçu l'adresse IP du DHCP Vlan 30

 * * <img width="1461" height="825" alt="image" src="https://github.com/user-attachments/assets/a6f93a1a-d4a9-4fd9-bb9d-d76fc7273d1d" /> * *

  
---

# 📝 Évaluation (/20)

| Critère | Points |
|--------|-------|
VLAN créés correctement | 4 |
Ports bien affectés | 2 |
Trunk opérationnel | 4 |
Inter-VLAN fonctionnel | 4 |
Travail sur les masques | 4 |  
Extention | 2 |  
  
# ✅ Fin du TP

Si vous savez expliquer :
> "Pourquoi deux VLAN ne communiquent pas sans routeur ?"

Alors vous avez compris la segmentation réseau 👍
