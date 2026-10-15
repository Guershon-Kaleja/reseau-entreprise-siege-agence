# Plan d'adressage

## IPv4 : découpage VLSM de 172.16.0.0/22

| LAN | Besoin | Réseau | Masque | Passerelle | Switch | Plage d'hôtes | Broadcast |
|---|---|---|---|---|---|---|---|
| Hôtes (siège) | 100 | 172.16.0.0/25 | 255.255.255.128 | .1 | .2 | .3 à .126 | .127 |
| Wi-Fi (agence) | 60 | 172.16.0.128/26 | 255.255.255.192 | .129 | aucune (AP) | .130 à .190 | .191 |
| Filaire (agence) | 40 | 172.16.0.192/26 | 255.255.255.192 | .193 | .194 | .195 à .254 | .255 |
| Serveurs (siège) | 12 | 172.16.1.0/28 | 255.255.255.240 | .1 | .2 | .3 à .14 | .15 |
| Lien routeur-routeur | 2 | 172.16.1.16/30 | 255.255.255.252 | | | .17 et .18 | .19 |

## Interfaces des routeurs

### RsiegeAdmin (siège)

| Interface | Rôle | IPv4 | IPv6 |
|---|---|---|---|
| Gi0/0 | LAN Serveurs | 172.16.1.1/28 | 2001:db8:beef:101::1/64, FE80::1 |
| Gi0/1 | LAN Hôtes | 172.16.0.1/25 | 2001:db8:beef:100::1/64, FE80::1 |
| Gi0/2 | Lien vers Ragence | 172.16.1.17/30 | 2001:db8:beef:300::1/64, FE80::1 |

### Ragence (agence)

| Interface | Rôle | IPv4 | IPv6 |
|---|---|---|---|
| Gi0/0 | LAN Filaire | 172.16.0.193/26 | 2001:db8:beef:201::1/64, FE80::1 |
| Gi0/1 | LAN Wi-Fi | 172.16.0.129/26 | 2001:db8:beef:200::1/64, FE80::1 |
| Gi0/2 | Lien vers RsiegeAdmin | 172.16.1.18/30 | 2001:db8:beef:300::2/64, FE80::2 |

## Switches (gestion IPv4 uniquement)

| Switch | Interface | IPv4 | Passerelle par défaut |
|---|---|---|---|
| SlanHotes | VLAN 1 | 172.16.0.2/25 | 172.16.0.1 |
| SsiegeServeurs | VLAN 1 | 172.16.1.2/28 | 172.16.1.1 |
| SlanFilaire | VLAN 1 | 172.16.0.194/26 | 172.16.0.193 |

## Serveurs (LAN Serveurs)

| Serveur | IPv4 (fixe) | IPv6 prévue | IPv6 obtenue (SLAAC) |
|---|---|---|---|
| DHCP 1 | 172.16.1.3 | 2001:db8:beef:101::3 | 2001:DB8:BEEF:101:2D0:BAFF:FE37:6079 |
| DHCP 2 | 172.16.1.4 | 2001:db8:beef:101::4 | 2001:DB8:BEEF:101:20C:85FF:FEC2:81A7 |
| DNS 1 | 172.16.1.5 | 2001:db8:beef:101::5 | 2001:DB8:BEEF:101:260:70FF:FE2C:3984 |
| DNS 2 | 172.16.1.6 | 2001:db8:beef:101::6 | 2001:DB8:BEEF:101:2D0:58FF:FE08:3B08 |
| HTTP 1 | 172.16.1.7 | 2001:db8:beef:101::7 | 2001:DB8:BEEF:101:2D0:FFFF:FE36:3928 |
| HTTP 2 | 172.16.1.8 | 2001:db8:beef:101::8 | 2001:DB8:BEEF:101:201:C9FF:FE69:AC9A |

Les adresses IPv6 statiques prévues n'ont pas pu être conservées dans Packet Tracer (voir la section « Limites » du README) : les serveurs utilisent SLAAC. Le tableau « prévue » est la configuration cible sur du matériel réel.

## Pools DHCPv4 (sur chaque serveur DHCP)

| LAN | Serveur 1 (`172.16.1.3`) | Serveur 2 (`172.16.1.4`) | Réserve |
|---|---|---|---|
| Hôtes | .3 à .72 (70) | .73 à .102 (30) | .103 à .126 |
| Wi-Fi | .130 à .171 (42) | .172 à .189 (18) | .190 |
| Filaire | .195 à .222 (28) | .223 à .234 (12) | .235 à .254 |

Le LAN Serveurs n'a pas de pool : adresses fixes. Chaque pool distribue la passerelle du LAN, le masque et un serveur DNS.

## IPv6 : plan hiérarchique de 2001:db8:beef::/48

| Zone | Préfixe |
|---|---|
| Siège (résumé) | 2001:db8:beef:100::/56 |
| LAN Hôtes | 2001:db8:beef:100::/64 |
| LAN Serveurs | 2001:db8:beef:101::/64 |
| Agence (résumé) | 2001:db8:beef:200::/56 |
| LAN Wi-Fi | 2001:db8:beef:200::/64 |
| LAN Filaire | 2001:db8:beef:201::/64 |
| Lien routeur-routeur | 2001:db8:beef:300::/64 |

## Routes statiques

| Routeur | Destination | Prochain saut |
|---|---|---|
| RsiegeAdmin | 172.16.0.128/25 | 172.16.1.18 |
| RsiegeAdmin | 2001:db8:beef:200::/56 | 2001:db8:beef:300::2 |
| Ragence | 172.16.0.0/25 | 172.16.1.17 |
| Ragence | 172.16.1.0/28 | 172.16.1.17 |
| Ragence | 2001:db8:beef:100::/56 | 2001:db8:beef:300::1 |

## Noms DNS

| Nom | Adresse |
|---|---|
| test.com | 172.16.1.8 |
