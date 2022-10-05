# TP1 - Premier pas réseau

## I. Exploration locale en solo

### 1. Affichage d'informations sur la pile TCP/IP locale

\
**Interface Wifi**
Nom : Intel(R) Wi-Fi 6 AX201 160MHz
MAC : 84-14-4D-10-5E-AF
IP : 10.33.16.63

**Interface Ethernet**
Nom : Realtek PCIe GbE Family Controller
MAC : D8-BB-C1-AA-C0-E7
IP : Média déconnecté

\
**Gateway**
Commande utilisée : *ipconfig*
IP : 10.33.19.254

\
**Table ARP**
Commande utilisée : *arp -a -N 10.33.16.63*
MAC : 00-c0-e7-e0-04-4e

\
**Interface Graphique**
![](https://i.imgur.com/5tWkSei.png)


### 2. Modifications des informations
### A. Modification d'adresse IP (part 1)

🌞 Utilisez l'interface graphique de votre OS pour **changer d'adresse IP** :


![](https://i.imgur.com/KoYVxO1.png)


🌞 **Il est possible que vous perdiez l'accès internet.** Que ce soit le cas ou non, expliquez pourquoi c'est possible de perdre son accès internet en faisant cette opération.

On perd l'accès car l'IP est déjà utilisé par quelqu'un d'autre, comme on est deuxième donc pas prioritaire, nous n'avons plus accès à Internet.

# II. Exploration locale en duo (avec Alan Bourdoncle)

## 3. Modification d'adresse IP

🌞 **Modifiez l'IP des deux machines pour qu'elles soient dans le même réseau**

![](https://i.imgur.com/AcLlKbQ.png)

🌞 **Vérifier à l'aide d'une commande que votre IP a bien été changée**

``````
PS C:\Users\maxfe> ipconfig

Configuration IP de Windows


Carte Ethernet Ethernet :

   Suffixe DNS propre à la connexion. . . :
   Adresse IPv6 de liaison locale. . . . .: fe80::a0b1:a3a9:115e:ca4a%17
   Adresse IPv4. . . . . . . . . . . . . .: 10.10.10.96 <------
   Masque de sous-réseau. . . . . . . . . : 255.255.255.0
``````

🌞 **Vérifier que les deux machines se joignent**

``````
PS C:\Users\maxfe> ping 10.10.10.69

Envoi d’une requête 'Ping'  10.10.10.69 avec 32 octets de données :
Réponse de 10.10.10.69 : octets=32 temps=1 ms TTL=128
Réponse de 10.10.10.69 : octets=32 temps=1 ms TTL=128
Réponse de 10.10.10.69 : octets=32 temps=1 ms TTL=128
Réponse de 10.10.10.69 : octets=32 temps=2 ms TTL=128

Statistiques Ping pour 10.10.10.69:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 1ms, Maximum = 2ms, Moyenne = 1ms
``````

🌞 **Déterminer l'adresse MAC de votre correspondant**

``````
PS C:\Users\maxfe> arp -a

Interface : 10.10.10.96 --- 0x11
  Adresse Internet      Adresse physique      Type
  10.10.10.69           08-8f-c3-51-69-8e     dynamique <------
  10.10.10.255          ff-ff-ff-ff-ff-ff     statique
  224.0.0.22            01-00-5e-00-00-16     statique
  224.0.0.251           01-00-5e-00-00-fb     statique
  224.0.0.252           01-00-5e-00-00-fc     statique
  239.255.255.250       01-00-5e-7f-ff-fa     statique
  255.255.255.255       ff-ff-ff-ff-ff-ff     statique
``````

## 4. Utilisation d'un des deux comme gateway

🌞**Tester l'accès internet**

```
PS C:\Users\maxfe> ping 1.1.1.1

Envoi d’une requête 'Ping'  1.1.1.1 avec 32 octets de données :
Réponse de 1.1.1.1 : octets=32 temps=28 ms TTL=54
Réponse de 1.1.1.1 : octets=32 temps=25 ms TTL=54
Réponse de 1.1.1.1 : octets=32 temps=25 ms TTL=54
Réponse de 1.1.1.1 : octets=32 temps=28 ms TTL=54

Statistiques Ping pour 1.1.1.1:
    Paquets : envoyés = 4, reçus = 4, perdus = 0 (perte 0%),
Durée approximative des boucles en millisecondes :
    Minimum = 25ms, Maximum = 28ms, Moyenne = 26ms
```

🌞 **Prouver que la connexion Internet passe bien par l'autre PC**

```
PS C:\Users\maxfe> tracert 192.168.137.1

Détermination de l’itinéraire vers TelosGVNG [192.168.137.1]
avec un maximum de 30 sauts :

  1     1 ms     1 ms     1 ms  TelosGVNG [192.168.137.1]

Itinéraire déterminé.
```

## 5. Petit chat privé

🌞 **sur le PC *serveur*** avec par exemple l'IP 192.168.1.1

```
PS C:\Users\alanw\netcat-1.11> .\nc.exe -l -p 8888
```

🌞 **sur le PC *client*** avec par exemple l'IP 192.168.1.2

```
Windows PowerShell
Copyright (C) Microsoft Corporation. Tous droits réservés.

Installez la dernière version de PowerShell pour de nouvelles fonctionnalités et améliorations ! https://aka.ms/PSWindows

PS C:\Users\maxfe\Desktop\netcat-win32-1.11\netcat-1.11> .\nc.exe 192.168.137.1 8888
[fmaxance] : Salut !
[balan] : Salut
[balan] : ça dit quoi ?
[fmaxance] : rien de spécial mise à part que j'aime les pommes et toi ?
[balan] : bah écoute j'aime les pommes aussi
[fmaxance] : SUPER en revoir
```

🌞 **Visualiser la connexion en cours**

- sur tous les OS, il existe une commande permettant de voir les connexions en cours
- ouvrez un deuxième terminal pendant une session `netcat`, et utilisez la commande correspondant à votre OS pour repérer la connexion `netcat` :

```
PS C:\Users\alanw> netstat -a -n -b

  TCP    192.168.137.1:8888     192.168.137.2:56320    ESTABLISHED
 [nc.exe]
```

🌞 **Pour aller un peu plus loin**

Sans IP définie : 
```
PS C:\Users\alanw\netcat-1.11> ./nc.exe -l -p 8888

PS C:\Users\alanw> netstat -a -n -b | Select-String 8888

  TCP    0.0.0.0:8888           0.0.0.0:0              LISTENING
```
N'importe qui peut se connecter sur le serveur car l'IP n'est pas définie.

\
Avec IP définie : 
```
PS C:\Users\alanw\netcat-1.11> ./nc.exe -l -p 8888 -s 192.168.137.1
PS C:\Users\alanw> netstat -a -n -b | Select-String 8888

  TCP    192.168.137.1:8888     0.0.0.0:0              LISTENING
```

## 6. Firewall

🌞 **Activez et configurez votre firewall**

![](https://i.imgur.com/LBUWX1Z.png)
  
# III. Manipulations d'autres outils/protocoles côté client

## 1. DHCP

🌞**Exploration du DHCP, depuis votre PC**

```
PS C:\Users\maxfe> ipconfig /all

Carte réseau sans fil Wi-Fi :

   Suffixe DNS propre à la connexion. . . :
   Description. . . . . . . . . . . . . . : Intel(R) Wi-Fi 6 AX201 160MHz
   Adresse physique . . . . . . . . . . . : 84-14-4D-10-5E-AF
   DHCP activé. . . . . . . . . . . . . . : Oui
   Configuration automatique activée. . . : Oui
   Adresse IPv6 de liaison locale. . . . .: fe80::4d16:36f5:df08:3d88%19(préféré)
   Adresse IPv4. . . . . . . . . . . . . .: 10.33.16.79(préféré)
   Masque de sous-réseau. . . . . . . . . : 255.255.252.0
   Bail obtenu. . . . . . . . . . . . . . : mercredi 5 octobre 2022 09:06:37
   Bail expirant. . . . . . . . . . . . . : jeudi 6 octobre 2022 09:06:36 <------
   Passerelle par défaut. . . . . . . . . : 10.33.19.254
   Serveur DHCP . . . . . . . . . . . . . : 10.33.19.254 <------
   IAID DHCPv6 . . . . . . . . . . . : 159650893
   DUID de client DHCPv6. . . . . . . . : 00-01-00-01-29-05-CC-02-D8-BB-C1-AA-C0-E7
   Serveurs DNS. . .  . . . . . . . . . . : 8.8.8.8
                                       8.8.4.4
                                       1.1.1.1
   NetBIOS sur Tcpip. . . . . . . . . . . : Activé
```

## 2. DNS

🌞** Trouver l'adresse IP du serveur DNS que connaît votre ordinateur**

```
PS C:\Users\maxfe> ipconfig /all

Carte réseau sans fil Wi-Fi :

   Serveurs DNS. . .  . . . . . . . . . . : 8.8.8.8
                                       8.8.4.4
                                       1.1.1.1
```

🌞 Utiliser, en ligne de commande l'outil `nslookup` (Windows, MacOS) ou `dig` (GNU/Linux, MacOS) pour faire des requêtes DNS à la main

```
PS C:\Users\maxfe> nslookup google.com
Serveur :   dns.google
Address:  8.8.8.8

Réponse ne faisant pas autorité :
Nom :    google.com
Addresses:  2a00:1450:4007:81a::200e
          216.58.214.78
```

```
PS C:\Users\maxfe> nslookup ynov.com
Serveur :   dns.google
Address:  8.8.8.8

Réponse ne faisant pas autorité :
Nom :    ynov.com
Addresses:  2606:4700:20::681a:be9
          2606:4700:20::681a:ae9
          2606:4700:20::ac43:4ae2
          104.26.11.233
          104.26.10.233
          172.67.74.226
```

Les deux domaines utilisent le même DNS ("8.8.8.8") pour faire des requêtes.

On a effectuer ces requêtes auprès de l'adresse IP 8.8.8.8

```
PS C:\Users\maxfe> nslookup 231.34.113.12
Serveur :   dns.google
Address:  8.8.8.8

*** dns.google ne parvient pas à trouver 231.34.113.12 : Non-existent domain
```

```
PS C:\Users\maxfe> nslookup 78.34.2.17
Serveur :   dns.google
Address:  8.8.8.8

Nom :    cable-78-34-2-17.nc.de
Address:  78.34.2.17
```

Pour la première Ip ("231.34.113.12") le DNS de google n'a pas trouver de nom de domaine.
Pour la deuxième Ip ("78.34.2.17") le DNS de google a trouver un domaine au nom de : cable-78-34-2-17.nc.de

# IV. Wireshark

🌞 Utilisez le pour observer les trames qui circulent entre vos deux carte Ethernet. Mettez en évidence :

- un `ping` entre vous et votre passerelle

![](https://i.imgur.com/FdVxEQ4.png)

- un `netcat` entre vous et votre mate, branché en RJ45

![](https://i.imgur.com/fXsZu4l.png)

- une requête DNS. Identifiez dans la capture le serveur DNS à qui vous posez la question.

![](https://i.imgur.com/9v8O7xz.png)

🌞 **Wireshark it**

- déterminez à quelle IP et quel port votre PC se connecte quand vous regardez une vidéo Youtube

![](https://i.imgur.com/nxCyFqS.png)

IP : 10.33.16.79
Port : 61652