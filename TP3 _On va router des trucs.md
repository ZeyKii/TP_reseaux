# TP3 : On va router des trucs

### 1. Echange ARP

🌞**Générer des requêtes ARP**

```
[fmaxance@localhost ~]$ ping 10.3.1.12
PING 10.3.1.12 (10.3.1.12) 56(84) bytes of data.
64 bytes from 10.3.1.12: icmp_seq=1 ttl=64 time=0.516 ms
64 bytes from 10.3.1.12: icmp_seq=2 ttl=64 time=0.513 ms
64 bytes from 10.3.1.12: icmp_seq=3 ttl=64 time=0.288 ms
64 bytes from 10.3.1.12: icmp_seq=4 ttl=64 time=0.605 ms
64 bytes from 10.3.1.12: icmp_seq=5 ttl=64 time=0.383 ms
64 bytes from 10.3.1.12: icmp_seq=6 ttl=64 time=0.531 ms
```

```
[fmaxance@John ~]$ ip -s n s
10.3.1.12 dev enp0s8 lladdr 08:00:27:e8:fc:9f used 25/23/4 probes 1 STALE
10.3.1.10 dev enp0s8 lladdr 0a:00:27:00:00:03 ref 1 used 0/0/1 probes 1 DELAY
```

```
[fmaxance@Marcel ~]$ ip -s n s
10.3.1.10 dev enp0s8 lladdr 0a:00:27:00:00:03 ref 1 used 0/0/2 probes 4 DELAY
10.3.1.11 dev enp0s8 lladdr 08:00:27:93:6c:7d used 41/41/11 probes 4 STALE
```

```
[fmaxance@Marcel ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:e8:fc:9f brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.12/24 brd 10.3.1.255 scope global noprefixroute enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fee8:fc9f/64 scope link
       valid_lft forever preferred_lft forever
```

### 2. Analyse de trames

🌞**Analyse de trames**

[source](./tp3_arp.pcap)

## II. Routage

| Machine  | `10.3.1.0/24` | `10.3.2.0/24` |
|----------|---------------|---------------|
| `router` | `10.3.1.254`  | `10.3.2.254`  |
| `john`   | `10.3.1.11`   | no            |
| `marcel` | no            | `10.3.2.12`   |
```schema
   john                router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └───┘    └─────┘    └───┘    └─────┘
```

### 1. Mise en place du routage

🌞**Activer le routage sur le noeud `router`**

```
[fmaxance@Routeur ~]$ sudo firewall-cmd --list-all
[sudo] password for fmaxance:
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: enp0s8 enp0s9
  sources:
  services: cockpit dhcpv6-client ssh
  ports:
  protocols:
  forward: yes
  masquerade: yes
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:
```

```
[fmaxance@Routeur ~]$ sudo firewall-cmd --get-active-zone
public
  interfaces: enp0s9 enp0s8
```

```
[fmaxance@Routeur ~]$ sudo firewall-cmd --add-masquerade --zone=public
success
```

```
[fmaxance@Routeur ~]$ sudo firewall-cmd --add-masquerade --zone=public --permanent
success
```

🌞**Ajouter les routes statiques nécessaires pour que `john` et `marcel` puissent se `ping`**

```
[fmaxance@John ~]$ ip r s
10.3.1.0/24 dev enp0s8 proto kernel scope link src 10.3.1.11 metric 100
10.3.2.0/24 via 10.3.1.254 dev enp0s8
```

```
[fmaxance@Marcel ~]$ ip r s
10.3.1.0/24 via 10.3.2.254 dev enp0s8
10.3.2.0/24 dev enp0s8 proto kernel scope link src 10.3.2.12 metric 100
```

```
[fmaxance@John ~]$ ping 10.3.2.12
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=0.785 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.771 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=0.822 ms
64 bytes from 10.3.2.12: icmp_seq=4 ttl=63 time=0.887 ms
```

### 2. Analyse de trames

🌞**Analyse des échanges ARP**

```
[fmaxance@John ~]$ ip -s n s
10.3.1.10 dev enp0s8 lladdr 0a:00:27:00:00:03 ref 1 used 0/0/0 probes 4 REACHABLE
10.3.1.254 dev enp0s8 lladdr 08:00:27:3a:d2:16 used 120/120/104 probes 4 STALE
```

```
[fmaxance@Marcel ~]$ ip -s n s
10.3.2.254 dev enp0s8 lladdr 08:00:27:a2:a0:f7 used 202/200/182 probes 1 STALE
10.3.2.10 dev enp0s8 lladdr 0a:00:27:00:00:04 ref 1 used 2/2/4 probes 4 DELAY
```

```
[fmaxance@Routeur ~]$ ip -s n s
10.3.2.12 dev enp0s9 lladdr 08:00:27:e8:fc:9f used 138/138/96 probes 4 STALE
10.3.1.11 dev enp0s8 lladdr 08:00:27:93:6c:7d used 135/133/116 probes 1 STALE
10.3.1.10 dev enp0s8 lladdr 0a:00:27:00:00:03 ref 1 used 0/0/3 probes 4 DELAY
```

| ordre | type trame  | IP source | MAC source              | IP destination | MAC destination            |
|-------|-------------|-----------|-------------------------|----------------|----------------------------|
| 1     | Requête ARP | x         | `marcel` `08:00:27:93:6c:7d` | x              | Broadcast `ff:ff:ff:ff:ff` |
| 2     | Réponse ARP | x         | `serveur` `08:00:27:3a:d2:16`                       | x              | `marcel` `08:00:27:93:6c:7d`    |
| 3     | Ping        | `10.3.1.11`         | `08:00:27:93:6c:7d`                       | `10.3.2.12`              | `08:00:27:3a:d2:16`                          |
| 4     | Pong        | `10.3.2.12`         | `08:00:27:3a:d2:16`                       | `10.3.1.11`              | `08:00:27:93:6c:7d`                          |

[source](tp3_routage_marcel.pcap)

### 3. Accès internet

🌞**Donnez un accès internet à vos machines**

  ```
  [fmaxance@John ~]$ sudo ip route add default via 10.3.1.254 dev enp0s8
  [fmaxance@Marcel ~]$ sudo ip route add default 10.3.2.254 dev enp0s8

  [fmaxance@John ~]$ ping 8.8.8.8
  PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
  64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=21.9 ms
  64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=22.4 ms
  64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=22.4 ms

  [fmaxance@John ~]$ ping google.com
  PING google.com (142.250.179.78) 56(84) bytes of data.
  64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=1 ttl=112 time=21.4 ms
  64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=2 ttl=112 time=23.1 ms
  64 bytes from par21s19-in-f14.1e100.net (142.250.179.78): icmp_seq=3 ttl=112 time=22.0 ms

  ```

🌞**Analyse de trames**

| ordre | type trame | IP source          | MAC source              | IP destination | MAC destination |     |
|-------|------------|--------------------|-------------------------|----------------|-----------------|-----|
| 1     | ping       | `john` `10.3.1.11` | `john` `08:00:27:93:6c:7d` | `8.8.8.8`      | `08:00:27:3a:d2:16`               |     |
| 2     | pong       | `8.8.8.8`                | `08:00:27:3a:d2:16`                     | `10.3.1.11`            | `08:00:27:93:6c:7d`             |  |

[source](tp3_routage_internet.pcap)

## III. DHCP

On reprend la config précédente, et on ajoutera à la fin de cette partie une 4ème machine pour effectuer des tests.

| Machine  | `10.3.1.0/24`              | `10.3.2.0/24` |
|----------|----------------------------|---------------|
| `router` | `10.3.1.254`               | `10.3.2.254`  |
| `john`   | `10.3.1.11`                | no            |
| `bob`    | oui mais pas d'IP statique | no            |
| `marcel` | no                         | `10.3.2.12`   |

```schema
   john               router              marcel
  ┌─────┐             ┌─────┐             ┌─────┐
  │     │    ┌───┐    │     │    ┌───┐    │     │
  │     ├────┤ho1├────┤     ├────┤ho2├────┤     │
  └─────┘    └─┬─┘    └─────┘    └───┘    └─────┘
   john        │
  ┌─────┐      │
  │     │      │
  │     ├──────┘
  └─────┘
```

### 1. Mise en place du serveur DHCP

🌞**Sur la machine `john`, vous installerez et configurerez un serveur DHCP** (go Google "rocky linux dhcp server").

```
[fmaxance@John ~]$ sudo systemctl status dhcpd
● dhcpd.service - DHCPv4 Server Daemon
     Loaded: loaded (/usr/lib/systemd/system/dhcpd.service; disabled; vendor preset: disabled)
     Active: active (running) since Mon 2022-10-24 11:24:23 CEST; 5min ago
       Docs: man:dhcpd(8)
             man:dhcpd.conf(5)
   Main PID: 11591 (dhcpd)
     Status: "Dispatching packets..."
      Tasks: 1 (limit: 11118)
     Memory: 4.6M
        CPU: 6ms
     CGroup: /system.slice/dhcpd.service
             └─11591 /usr/sbin/dhcpd -f -cf /etc/dhcp/dhcpd.conf -user dhcpd -group dhcpd --no-pid

Oct 24 11:24:33 John dhcpd[11591]: DHCPDISCOVER from 08:00:27:fc:89:e9 via enp0s8
Oct 24 11:24:34 John dhcpd[11591]: DHCPOFFER on 10.3.1.200 to 08:00:27:fc:89:e9 (Bob) via enp0s8
Oct 24 11:24:34 John dhcpd[11591]: DHCPREQUEST for 10.3.1.200 (10.3.1.11) from 08:00:27:fc:89:e9 (Bob) via enp0s8
Oct 24 11:24:34 John dhcpd[11591]: DHCPACK on 10.3.1.200 to 08:00:27:fc:89:e9 (Bob) via enp0s8
Oct 24 11:24:54 John dhcpd[11591]: DHCPDISCOVER from 08:00:27:93:6c:7d via enp0s8
Oct 24 11:24:55 John dhcpd[11591]: DHCPOFFER on 10.3.1.201 to 08:00:27:93:6c:7d via enp0s8
Oct 24 11:24:55 John dhcpd[11591]: DHCPREQUEST for 10.3.1.201 (10.3.1.11) from 08:00:27:93:6c:7d via enp0s8
Oct 24 11:24:55 John dhcpd[11591]: DHCPACK on 10.3.1.201 to 08:00:27:93:6c:7d via enp0s8
Oct 24 11:29:33 John dhcpd[11591]: DHCPREQUEST for 10.3.1.200 from 08:00:27:fc:89:e9 (Bob) via enp0s8
Oct 24 11:29:33 John dhcpd[11591]: DHCPACK on 10.3.1.200 to 08:00:27:fc:89:e9 (Bob) via enp0s8
```

```
[fmaxance@Bob ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:fc:89:e9 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.200/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 545sec preferred_lft 545sec
    inet6 fe80::a00:27ff:fefc:89e9/64 scope link
       valid_lft forever preferred_lft forever
```

🌞**Améliorer la configuration du DHCP**

```
[fmaxance@Bob ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:fc:89:e9 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.200/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 320sec preferred_lft 320sec
    inet6 fe80::a00:27ff:fefc:89e9/64 scope link
       valid_lft forever preferred_lft forever


[fmaxance@Bob ~]$ ping 10.3.2.12 -c 3
PING 10.3.2.12 (10.3.2.12) 56(84) bytes of data.
64 bytes from 10.3.2.12: icmp_seq=1 ttl=63 time=0.763 ms
64 bytes from 10.3.2.12: icmp_seq=2 ttl=63 time=0.839 ms
64 bytes from 10.3.2.12: icmp_seq=3 ttl=63 time=0.971 ms

--- 10.3.2.12 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2028ms
rtt min/avg/max/mdev = 0.763/0.857/0.971/0.085 ms


[fmaxance@Bob ~]$ ping 8.8.8.8 -c 3
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=22.9 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=22.9 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=22.1 ms

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2003ms
rtt min/avg/max/mdev = 22.121/22.658/22.936/0.380 ms


[fmaxance@Bob ~]$ ping 10.3.1.11 -c 3
PING 10.3.1.11 (10.3.1.11) 56(84) bytes of data.
64 bytes from 10.3.1.11: icmp_seq=1 ttl=64 time=0.313 ms
64 bytes from 10.3.1.11: icmp_seq=2 ttl=64 time=0.566 ms
64 bytes from 10.3.1.11: icmp_seq=3 ttl=64 time=0.409 ms

--- 10.3.1.11 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2031ms
rtt min/avg/max/mdev = 0.313/0.429/0.566/0.104 ms


[fmaxance@Bob ~]$ ip r s
default via 10.3.1.254 dev enp0s8 proto dhcp src 10.3.1.200 metric 100
10.3.1.0/24 dev enp0s8 proto kernel scope link src 10.3.1.200 metric 100


[fmaxance@Bob ~]$ dig

; <<>> DiG 9.16.23-RH <<>>
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 55827
;; flags: qr rd ra ad; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;.                              IN      NS

;; ANSWER SECTION:
.                       86419   IN      NS      a.root-servers.net.
.                       86419   IN      NS      b.root-servers.net.
.                       86419   IN      NS      c.root-servers.net.
.                       86419   IN      NS      d.root-servers.net.
.                       86419   IN      NS      e.root-servers.net.
.                       86419   IN      NS      f.root-servers.net.
.                       86419   IN      NS      g.root-servers.net.
.                       86419   IN      NS      h.root-servers.net.
.                       86419   IN      NS      i.root-servers.net.
.                       86419   IN      NS      j.root-servers.net.
.                       86419   IN      NS      k.root-servers.net.
.                       86419   IN      NS      l.root-servers.net.
.                       86419   IN      NS      m.root-servers.net.

;; Query time: 25 msec
;; SERVER: 8.8.8.8#53(8.8.8.8)
;; WHEN: Mon Oct 24 12:03:01 CEST 2022
;; MSG SIZE  rcvd: 239

[fmaxance@Bob ~]$ ping google.com
PING google.com (216.58.214.78) 56(84) bytes of data.
64 bytes from fra15s10-in-f14.1e100.net (216.58.214.78): icmp_seq=1 ttl=112 time=21.3 ms
64 bytes from fra15s10-in-f14.1e100.net (216.58.214.78): icmp_seq=2 ttl=112 time=24.4 ms
64 bytes from par10s39-in-f14.1e100.net (216.58.214.78): icmp_seq=3 ttl=112 time=22.4 ms
```

### 2. Analyse de trames

🌞**Analyse de trames**

[source](tp3_dhcp.pcap)

```
[fmaxance@Bob ~]$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
2: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:fc:89:e9 brd ff:ff:ff:ff:ff:ff
    inet 10.3.1.200/24 brd 10.3.1.255 scope global dynamic noprefixroute enp0s8
       valid_lft 509sec preferred_lft 509sec
    inet 10.3.1.202/24 brd 10.3.1.255 scope global secondary dynamic enp0s8
       valid_lft 617sec preferred_lft 617sec
    inet6 fe80::a00:27ff:fefc:89e9/64 scope link
       valid_lft forever preferred_lft forever
```
