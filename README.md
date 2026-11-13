[atelier 2.docx](https://github.com/user-attachments/files/34820277/atelier.2.docx)# reseaux-packet-tracer1
# 07 – Infrastructure multi-sites : VLAN, DHCP, OSPF, NAT/PAT et tunnel GRE


## Objectif
Concevoir et configurer une infrastructure à deux bâtiments (A et B) reliés à un réseau opérateur : **VLAN et routage inter-VLAN**, **DHCP**, **OSPF**, **NAT/PAT** et **tunnel GRE**.

## Plan d'adressage (extraits)
| Élément | Réseau |
|---------|--------|
| Bâtiment A (VLAN 10, 20, 30) | 192.168.10.0/24, 192.168.20.0/24, 192.168.30.0/24 |
| Bâtiment B (VLAN 40, 50, 60) | 192.168.40.0/24, 192.168.50.0/24, 192.168.60.0/24 |
| SWN3A ↔ RE1 | 10.0.0.0/30 |
| SWN3B ↔ RE1 | 10.0.1.0/30 |
| Réseau serveurs (serveur Web) | 192.168.5.0/24 |
| R0 ↔ RE2 | 212.217.7.8/30 |
| R0 ↔ RI | 212.217.12.0/30 |
| LAN de R0 (DNS, internet.com) | 212.217.26.0/24 |
| Tunnel GRE RE1 ↔ RE2 | 172.16.0.0/30 |

![Topologie](captures/07_topologie.png)
<img width="1288" height="382" alt="image" src="https://github.com/user-attachments/assets/d8ddd291-19d6-4995-8f54-a845365766bd" />
<img width="710" height="376" alt="image" src="https://github.com/user-attachments/assets/c0c8f2e0-5ee8-4463-8721-8f143db9d134" />


## Démarche
1. **Switches (SWN3A, SWN3B)** : création des VLAN, ports access, trunks entre switches et vers les petits switches, liens routés (`no switchport`), interfaces SVI comme passerelles, `ip routing`
2. **DHCP** : un pool par VLAN directement sur les switches (passerelle en `.254`) ; DHCP aussi sur RE2
3. **Routeur RE1** : interfaces LAN / serveurs / WAN, **NAT statique** (redirection du port 80 vers le serveur Web `192.168.5.10`) et **PAT** pour la sortie vers Internet
4. **Routeurs R0 / RI / RE2** : liens série (`clock rate` sur le port DCE), adressage, tests de ping
5. **Serveurs** : DNS Internet et serveur `internet.com`
6. **Routage dynamique OSPF** sur RE1, RE2, R0 et RI
7. **Tunnel GRE** entre RE1 et RE2 *(le sujet cite aussi EIGRP : [préciser si je l'ai configuré])*

## Configuration clé (extraits)
```
! Switch couche 3
ip routing
interface vlan 10
 ip address 192.168.10.254 255.255.255.0
ip dhcp pool DHCP_VLAN10
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.254

! NAT statique + PAT sur RE1
ip nat inside source static tcp 192.168.5.10 80 interface <WAN> 80
ip nat inside source list <ACL> interface <WAN> overload

! Liaison série (côté DCE)
interface serial 0/3/0
 clock rate 64000
```
*(Compléter `<WAN>` et `<ACL>` avec mes valeurs réelles.)*

## Vérifications
```
show ip interface brief
show ip route
show ip ospf neighbor
show ip nat translations
show ip nat statistics
ping 212.217.26.10        ! DNS Internet depuis un PC du bâtiment
```


## Ce que j'ai retenu
- Les VLAN sont routés directement sur des switches de niveau 3 avec des SVI ; le DHCP peut y être servi par VLAN.
- Le NAT statique publie un service interne (Web, port 80) ; le PAT permet à tout le LAN de sortir avec une seule adresse publique.
- Un tunnel GRE relie deux sites distants via un réseau intermédiaire ; OSPF assure la propagation des routes.

📁 Fichier : 

[atelier 2.docx](https://github.com/user-attachments/files/34820282/atelier.2.docx)

