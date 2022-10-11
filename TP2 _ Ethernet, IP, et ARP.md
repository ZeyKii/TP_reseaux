# TP2 : Ethernet, IP, et ARP

🌞 **Mettez en place une configuration réseau fonctionnelle entre les deux machines**

```
PS C:\WINDOWS\system32> Get-NetAdapter | FL

Name                       : Ethernet
InterfaceDescription       : Realtek PCIe GbE Family Controller
InterfaceIndex             : 17
MacAddress                 : D8-BB-C1-AA-C0-E7
MediaType                  : 802.3
PhysicalMediaType          : 802.3
InterfaceOperationalStatus : Up
AdminStatus                : Up
LinkSpeed(Gbps)            : 1
MediaConnectionState       : Connected
ConnectorPresent           : True
DriverInformation          : Driver Date 2021-07-14 Version 1168.1.714.2021 NDIS 6.85
```

```
PS C:\WINDOWS\system32> New-NetIPAddress -InterfaceIndex 17 -IPAddress 192.168.137.2 -AddressFamily IPv4 -PrefixLength 22
```

```
PS C:\WINDOWS\system32> ipconfig

Configuration IP de Windows


Carte Ethernet Ethernet :

   Suffixe DNS propre à la connexion. . . :
   Adresse IPv4. . . . . . . . . . . . . .: 192.168.137.2
   Masque de sous-réseau. . . . . . . . . : 255.255.252.0
   Passerelle par défaut. . . . . . . . . :
```

- Adresse Réseau : 192.168.136.0
- Broadcast : 192.168.139.255

🌞 **Prouvez que la connexion est fonctionnelle entre les deux machines**

```
PS C:\WINDOWS\system32> ping 192.168.137.1

Envoi d’une requête 'Ping'  192.168.137.1 avec 32 octets de données :
Réponse de 192.168.137.1 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=2 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=1 ms TTL=128

Statistiques Ping pour 192.168.137.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 2ms, Moyenne = 1ms
```

🌞 **Wireshark it**

[capture](./ping%20on%20alan.pcapng)

# II. ARP my bro

🌞 **Check the ARP table**
```
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.137.2 --- 0x11
  Adresse Internet      Adresse physique      Type
  192.168.137.1         08-8f-c3-51-69-8e     dynamique
  192.168.139.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```

```
MAC binome : 192.168.137.1         08-8f-c3-51-69-8e     dynamique
```

```
MAC gateway :
PS C:\WINDOWS\system32> arp -a 10.33.19.254

Interface : 10.33.16.79 --- 0x13
  Adresse Internet      Adresse physique      Type
  10.33.19.254          00-c0-e7-e0-04-4e     dynamique
```

🌞 **Manipuler la table ARP**

```
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.137.2 --- 0x11
  Adresse Internet      Adresse physique      Type
  192.168.137.1         08-8f-c3-51-69-8e     dynamique
  192.168.139.255       ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```

```
PS C:\WINDOWS\system32> arp -d
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.137.2 --- 0x11
  Adresse Internet      Adresse physique      Type
  224.0.0.22            01-00-5e-00-00-16     statique
```
```
PS C:\WINDOWS\system32> ping 192.168.137.1

Envoi d’une requête 'Ping'  192.168.137.1 avec 32 octets de données :
Réponse de 192.168.137.1 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=1 ms TTL=128
Réponse de 192.168.137.1 : octets=32 temps=1 ms TTL=128

Statistiques Ping pour 192.168.137.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 1ms, Moyenne = 1ms
```

```
PS C:\WINDOWS\system32> arp -a

Interface : 192.168.137.2 --- 0x11
  Adresse Internet      Adresse physique      Type
  192.168.137.1         08-8f-c3-51-69-8e     dynamique
  224.0.0.22            01-00-5e-00-00-16     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
```

🌞 **Wireshark it**

[capture](./ping%20alan%20arp.pcapng)

Broadcast (soi-même) envoi à toute les personnes présente sur le réseau pour chercher une seule personne et la deuxième est de l'unicast (la machine recherchée) donc elle répond directement à notre machine.

# III. DHCP you too my brooo

🌞 **Wireshark it**

[capture](./trames%20DORA.pcapng)