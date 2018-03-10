# Access Control lists - A straightforward guide
Have you always been searching for a simple explanation of the Access Control Lists ?
>~~No, I don't care about it.~~

> Yes, Of course!

## Getting Started

ACL (Access Control List) offers the possibility to create a kind of white/black list. It acts as a filter.

As a light switch does, an ACL can permit the flow or deny it.

By default, an ACL blocks the flow.

It is possible to create two kinds of list :
- Standard
- Extended

Standard ones are the simple ones while extended are more complex.

##  Define
- To create an ACL, make it in two parts :
   - First one, define the ACL,
   - the second one, apply it on an interface.


At the definition, give a number to your ACL to name it. Standart and Extented have different number ranges.

|   |Standart ACLs|Extended ACLs|
|----|-------------|-------------|
|access-list-number|0 to 99 & 1300 to 1999|100 to 199 & 2000 to 2699|

>Don't pay attention to high numbers as [1300->1999] and [2000->2699], it won't be usefull.

### Wildcard
As a mask does, Wildcards are usefull to identify a network.


### Standard list
```
access-list [access-list-number] [permit/deny]
[host/source source-wildcard|any]
```

The three examples below are exactly the same.
```
access-list 28 deny 192.168.90.36
```
```
access-list 28 deny 192.168.90.36 0.0.0.0
```
> *If there are only zeros, the Wildcard can be omitted.*
```
access-list 28 deny host 192.168.90.36
```
**There may be multiple ways to write an ACL.**

### Extended list
IP concerns all trafics.



## Apply

interface <interface>
ip access-group number {in|out}

```
Router(config)# interface <interface>
Router(config-if)# ip access-group number {in|out}
```

You have to apply an **Standard** ACL **the closest to the destination**.
You have to apply an **Extended** ACL **the closest to the source**.

## Examples

### Simple example Standard ACL

On the interface f0/1 out, block the traffic from a network (_192.168.0.0/22_) but allow 2 subnetworks (_192.168.0.0/24_ & _192.168.1.0/24_).
```
Router(config)# acces-list 10 permit 192.168.0.0 0.0.0.255
Router(config)# acces-list 10 permit 192.168.1.0 0.0.0.255
Router(config)# acces-list 10 deny 192.168.0.0 0.0.3.255
Router(config)# interface f0/1
Router(config-if)# ip access-group 10 out
```
To find the widmash, look at the mask of the network and write it with bits :
11111111.11111111.11111100.00000000
-> 255.255.252.0
then
255-252 = 3

### Problem 1 & 2

|         Names     |    IPs   |    Gateways|
|-------------------|---------|--|
|Melvin's PC| 172.16.70.35  |  172.16.70.1|
| Franck's PC| 172.16.70.32 |172.16.70.1|
| Jim's PC| 192.168.90.36  | 172.16.90.2|
| Kathy's PC| 192.168.90.38|    172.16.90.2|

The Router has three interfaces : E0, S0 and E1.

|Interfaces| Networks|
|---|---|
|E0|172.16.70.0|
|S0|210.30.28.0|
|E1|192.168.90.0|

> **Tips** : Make a schema to illustrate

1. Write a standard access list to block Melvin’s PC from sending information to Kathy’s PC ; but will allow all other traffic.
```
Router(config)# access-list 11 deny 172.16.70.35
Router(config)# access-list 11 permit any
Router(config)# interface e1
Router(config-if)# ip access-group 11 out
```

 2. Write a standard access list to block Jim’s PC from sending information to network 172.16.70.0; but will allow all other traffic from the 192.168.90.0 network and from the 210.30.28.0 network to reach the 172.16.70.0 network. Deny all other traffic.

```
Router(config)# access-list 28 deny 192.168.90.36
Router(config)# access-list 28 permit 192.168.90.0 0.0.0.255
Router(config)# access-list 28 permit 210.30.28.0 0.0.0.255
Router(config)# interface e0
Router(config-if)# ip access-group 28 out
```
Use the E0 to regulate from others interfaces.

### Problem 3

|         Names     |    IPs   |    Gateways|
|-------------------|---------|--|
|Johnn's PC| 172.16.70.35  |  172.16.70.1|
| Gail's PC| 172.16.70.32 |172.16.70.1|
| Mike's PC| 192.168.90.36  | 172.16.90.2|
| Celeste's PC| 192.168.90.38|    172.16.90.2|

The Router has two interfaces : FA0 & F01.
|Interfaces| Networks|
|---|---|
|FA0|172.16.70.0|
|FA1|192.168.90.0|

* Write an extended access list to block the 172.16.70.0 network from receiving information from Mike’s computer at 192.168.90.36. Block the lower half of the ip addresses from 192.168.90.0 network from reaching Gail’s computer at 172.16.70.32. Permit all other traffic.

->Destinataire est à la fin de l'extended ACL
```
Router(config)# access-list 111 deny ip 192.168.90.36 0.0.0.0 172.16.70.0 0.0.0.255
Router(config)# access-list 111 deny ip 192.168.90.0 0.0.0.127 172.16.70.32 0.0.0.0
Router(config)# access-list 111 permit ip any any
Router(config)# interface fa1
Router(config-if)# ip access-group 111 in
```
### Problem 4
|         Names  |    IPs   |    Gateways|
|-------------------|---------|--|
|Cindy's PC| 172.16.70.35  |  172.16.70.1|
| Ralph's PC| 172.16.70.32 |172.16.70.1|
| Bob's PC| 192.168.90.36  | 172.16.90.2|
| Barbra's PC| 192.168.90.38|    172.16.90.2|
| Router A| 192.16.20.5|   x|
| Router B| 192.18.50.10|    x|


The Router A has two interfaces : E0 & S0.
The Router B has two interfaces : S1 & E1.
S0 and S1 are linked together.

|Interfaces| Networks|
|---|---|
|E0|192.16.20.0|
|E1|192.18.50.0|

* Write an extended access list to permit the 192.16.20.0 network to receive packets from the 192.18.50.0 network. Deny all other Traffic from 192.18.50.0 network.

```
RouterB(config)# access-list 112 permit ip 192.18.50.0 0.0.0.255 192.16.20.0 0.0.0.255
RouterB(config)# access-list 112 deny ip any any
RouterB(config)# interface e1
RouterB(config-if)# ip access-group 112 in
```

### Problem 5

|         Names   |    IPs   |    Gateways|
|-------------------|---------|--|
|PC1| 192.168.207.26 | 192.168.207.25|
|WEB server1| 192.168.207.27 | 192.168.207.25|
|WEB server2| 210.128.50.11| 210.128.50.10|
|PC2| 210.128.50.12| 210.128.50.10|
| Router A| 192.168.207.25|   x|
| Router B| 210.128.50.10|    x|


The Router A has two interfaces : E0 & S0.
The Router B has two interfaces : S1 & E1.
S0 and S1 are linked together.

|Interfaces| Networks|
|---|---|
|E0|192.168.207.0|
|E1| 210.128.50.0|

Write an extended access list to deny HTTP traffic intended for web server 192.168.207.27, but will permit all other HTTP traffic to reach the only the 192.168.207.0 network. Deny all other IP traffic to 192.168.207.0 network.
```
RouterB(config)# access-list 112 deny ftp any 192.168.207.27
RouterB(config)# access-list 112 permit ftp any 192.168.207.0 0.0.0.255
RouterB(config)# interface e0
RouterB(config-if)# ip access-group 112 out
```
> No need to add this line "RouterB(config)# access-list 112 deny ip any 192.168.207.0 0.0.0.255" Because create an ACL deny all trafic by default.

### Problem 6
Write an access list to permit Denise’s and Bob’s computers to telnet into Router B. Deny all other telnet traffic.

|         Names    |    IPs   |    Gateways|
|-------------------|---------|--|
|Celeste| 192.30.76.145 | 172.20.70.1|
|Bob| 192.30.76.155 | 172.20.70.1|
|Peggy| 192.168.33.210| 192.168.33.1|
|Denise| 192.168.33.214| 192.168.33.1|


The Router A has three interfaces : E0, E1 & S0.

|Interfaces| IP|
|---|---|
|E0| 172.20.70.1|
|E1| 10.250.4.0|
|S0|x|

The Router B has three interfaces : E0, E1 &S1

|Interfaces| IP|
|---|---|
|E0| 172.16.16.0|
|E1| 192.168.33.1|
|S1|x|

S0 and S1 are linked together.

```
RouterB(config)# access-list 50 permit 192.168.33.214
RouterB(config)# access-list 50 permit 192.30.76.155
RouterB(config)# line vty 0 4
RouterB(config-if)# ip access-class 50 in
```

> Pay attention to the two lasts lines, they correspond to the telnet protocol.

### Reverse problem

---

## Sources

[Source 1](https://www.cisco.com/c/en/us/support/docs/security/ios-firewall/23602-confaccesslists.html#acltypes)
