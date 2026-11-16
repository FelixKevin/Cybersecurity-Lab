---
module: 3
order: 1
moduleTitle: "3: Routing and NAT"
published: false
---

# Module 3: Routing and NAT

After module 2 the zones of Halden & Co are separated, and nobody can work. In this module you connect them again through one device: a router. You learn how a packet finds its way to another network, how a private network reaches the internet, and how the internet reaches one server inside.

### Learning objectives

After this module you can:

- predict from a routing table where a host sends a packet;
- configure a static route and a default route, and explain the difference;
- route between VLANs with one router interface on a trunk;
- show with two captures what NAT changes in a packet;
- forward one port from the internet to a server in the DMZ.

### What you need

| What | Requirement |
| --- | --- |
| Your lab from module 2 | Four machines, with the snapshot `m2-zones` |
| Lab files | `halden-fw01.ova` and `halden-web01.ova`, from the course files on Canvas |
| Running at the same time | Six machines |

The steps for the router are written for OPNsense 26.7.

[CONTROLEREN] Both appliances do not exist yet. The requirements are in the review notes.

### Prepare your lab

You add two machines and one cable for yourself.

| Machine | Role | Connected to |
| --- | --- | --- |
| `halden-fw01` | Router, and from module 4 on the firewall | The internet, and `port3` of `halden-sw02` |
| `halden-web01` | The public web server with the client portal | `port2` of `halden-sw02` |
| Kali, third adapter | Your management laptop | `port3` of `halden-sw01` |

1. Shut down all machines.
2. Import `halden-fw01.ova` and `halden-web01.ova` with **File > Import Appliance**.
3. In the network settings of `halden-sw02`, change the internal network of **Adapter 3** to `trunk-sw02-fw01`. The router is on the other end of that cable.
4. In the network settings of Kali, enable **Adapter 3**, attach it to **Internal Network** and name the network `sw01-port3`.
5. Start all six machines.

The cable to the router carries all zones, so `port3` of `halden-sw02` becomes a trunk. On `halden-sw02`:

```bash
sudo ovs-vsctl clear port port3 tag
sudo ovs-vsctl set port port3 trunks=10,20,40,99
```

Update your port plan: `port3` of `halden-sw02` is now a trunk to `halden-fw01`.

`port3` of `halden-sw01` is in the Management zone. Give the new interface of Kali an address there:

```bash
sudo nmcli connection add type ethernet ifname eth2 con-name halden-mgmt ipv4.method manual ipv4.addresses 10.20.99.50/24 ipv4.never-default yes ipv6.method disabled
sudo nmcli connection up halden-mgmt
```

Open a browser in Kali and go to `https://10.20.99.1`. The browser warns about the certificate: the router made its own. Accept it and log in with the user `root`. You now see the dashboard of OPNsense.

[CONTROLEREN] The password of `root`, the adapter names of both appliances, and the interface name `eth2` in Kali.

[NIET GETEST] The `nmcli` commands on the current Kali image.

From now on Kali plays two roles. Through `eth1` it is a workstation in the Clients zone. Through `eth2` it is the laptop of an administrator in the Management zone.

## 3.1 How a packet finds its way

In module 2 you saw that Kali and the server could not reach each other across VLANs. Putting them in different IP ranges would not have helped. A host that wants to reach another network needs someone to hand the packet to.

A **router** is a device that is connected to several networks and passes packets from one to the other. Every host and every router decides where to send a packet with its **routing table**: a list of networks, each with the way to reach it. One line of that table is a **route**.

### Read the routing table of Kali

In a terminal in Kali:

```bash
ip route
```

```output
default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100
10.20.30.0/24 dev eth1 proto kernel scope link src 10.20.30.50 metric 101
10.20.99.0/24 dev eth2 proto kernel scope link src 10.20.99.50 metric 102
```

[NIET GETEST] This output on Kali. The order and the metrics can differ.

Read each line as a sentence. The last three say: "this network is on this interface". Such a route is a **connected route**. It appears by itself as soon as an interface has an address.

The first line is different. It says: "for everything else, hand the packet to 10.0.2.2". The router that a host uses for that is its **default gateway**.

### Ask where a packet goes

You do not have to work out the answer yourself. Ask the kernel:

```bash
ip route get 10.20.99.1
ip route get 1.1.1.1
```

```output
10.20.99.1 dev eth2 src 10.20.99.50 uid 1000
    cache
1.1.1.1 via 10.0.2.2 dev eth0 src 10.0.2.15 uid 1000
    cache
```

The first address is in a connected network: Kali sends the packet straight to it. The second matches only the default line, so the packet goes `via` the gateway.

When several routes match, the most specific one wins: the route whose network is the smallest. A route for 10.20.99.0/24 beats the default route, which matches every address.

### What "via" means on the wire

Think back to ARP in [1.4](./module-1.md#1_4). For a destination in a connected network, Kali asks for the MAC address of the destination. For any other destination, Kali asks for the MAC address of the gateway.

The packet that leaves has the IP address of the final destination and the MAC address of the router. The router receives the frame, reads the IP address, looks in its own routing table and repeats the process. Each router a packet passes is a **hop**.

What this means for you: when something is unreachable, read the routing table before anything else. It tells you where the packet goes, and that is often not where you thought.

### Check yourself 3.1 – How a packet finds its way

```text
Sectie: 3.1
Type: meerkeuze
Vraag: A host has this routing table. Where does it send a packet for 10.20.20.10? Choose one answer.
Code:
  default via 10.20.10.1 dev eth1
  10.20.10.0/24 dev eth1 proto kernel scope link src 10.20.10.50
Opties:
  A. To the router 10.20.10.1
  B. Straight to 10.20.20.10
  C. Nowhere: there is no route
  D. To every device on eth1
Juist: A
Feedback A: Correct. 10.20.20.10 is not in 10.20.10.0/24, so only the default route matches. The packet goes to the gateway (see 3.1).
Feedback B: A host sends straight to the destination only when it is in a connected network. 10.20.20.10 is not in 10.20.10.0/24 (see 3.1).
Feedback C: The default route matches every address that has no more specific route (see 3.1).
Feedback D: A packet for one address is sent to one next device, not to all of them (see 3.1).

Sectie: 3.1
Type: meerkeuze
Vraag: Kali sends a packet to 1.1.1.1 through its gateway 10.20.10.1. Which addresses are in the frame that leaves Kali? Choose one answer.
Opties:
  A. Destination IP 1.1.1.1 and destination MAC of the gateway
  B. Destination IP 10.20.10.1 and destination MAC of the gateway
  C. Destination IP 1.1.1.1 and destination MAC of 1.1.1.1
  D. Destination IP 10.20.10.1 and destination MAC of 1.1.1.1
Juist: A
Feedback A: Correct. The IP address stays that of the final destination. The MAC address is that of the next device (see 3.1).
Feedback B: If the destination IP were the gateway, the gateway would keep the packet for itself (see 3.1).
Feedback C: Kali cannot learn the MAC address of a machine in another network. ARP works only inside one network (see 3.1).
Feedback D: Neither is right: the IP address is the final destination, the MAC address is the gateway (see 3.1).

Sectie: 3.1
Type: koppelen
Vraag: Match each term with its description.
Paren:
  Connected route → A network that is on one of your own interfaces
  Default gateway → The router for everything without a more specific route
  Hop → One router that a packet passes
  Routing table → The list of networks and the way to reach each one
Feedback: A host needs only connected routes and a default gateway. Routers in between repeat the same lookup at every hop (see 3.1).
```

## 3.2 Static and default routes

Halden's workstations get their own range, 10.20.10.0/24. A workstation with only an address in that range knows one network. It has to learn the way to the rest.

### Give the router a leg in the Clients zone

The router is connected to the trunk. To take part in a VLAN, it needs an interface in that VLAN with an address in the range of the zone. That address becomes the gateway of the zone. Work in the browser, on `https://10.20.99.1`.

1. Go to **Interfaces > Devices > VLAN** and add an entry: **Device** `vlan0.10`, **Parent** `vtnet1`, **VLAN tag** `10`, **Description** `Clients`. Save and apply.
2. Go to **Interfaces > Assignments**. Choose `vlan0.10`, give it the description `Clients` and add it.
3. Open **Interfaces > [Clients]**. Tick **Enable Interface**, set **IPv4 Configuration Type** to **Static IPv4** and **IPv4 address** to `10.20.10.1` with `24`. Save and apply.
4. Go to **System > Routes > Status**. You now see a line for 10.20.10.0/24 on `vlan0.10`: the connected route of the router.

[CONTROLEREN] All labels in these steps for OPNsense 26.7, and the name of the parent interface.

### Move Kali into the zone

Until now Kali had its own way to the internet, through adapter 1. A real workstation at Halden & Co has no such shortcut. Pull that cable: in the network settings of Kali, untick **Cable Connected** under **Adapter 1**.

Then give `eth1` an address in the Clients range:

```bash
sudo nmcli connection modify halden-lab ipv4.addresses 10.20.10.50/24
sudo nmcli connection up halden-lab
```

Try to reach the router of the internet provider, which sits behind `halden-fw01`:

```bash
ping -c 2 10.0.2.2
```

```output
ping: connect: Network is unreachable
```

No packet left Kali. Its routing table holds only connected routes, and 10.0.2.2 is in none of them.

### A static route: one more network

Tell Kali that the provider's network is behind the router:

```bash
sudo ip route add 10.0.2.0/24 via 10.20.10.1
ping -c 2 10.0.2.2
```

```output
PING 10.0.2.2 (10.0.2.2) 56(84) bytes of data.
64 bytes from 10.0.2.2: icmp_seq=1 ttl=63 time=0.701 ms
64 bytes from 10.0.2.2: icmp_seq=2 ttl=63 time=0.500 ms
```

A **static route** is a route that an administrator adds by hand. The router you hand the packet to is the **next hop**. The general form is:

```text
ip route add <network> via <next hop>
```

The route covers one network and nothing more. `ping -c 1 1.1.1.1` still answers `Network is unreachable`.

### A default route: everything else

You could add a static route for every network on the internet. There is a shorter way. Remove the static route and give Kali a default gateway:

```bash
sudo ip route del 10.0.2.0/24
sudo nmcli connection modify halden-lab ipv4.gateway 10.20.10.1 ipv4.dns 10.20.10.1
sudo nmcli connection up halden-lab
ping -c 2 1.1.1.1
```

```output
PING 1.1.1.1 (1.1.1.1) 56(84) bytes of data.
64 bytes from 1.1.1.1: icmp_seq=1 ttl=54 time=14.2 ms
64 bytes from 1.1.1.1: icmp_seq=2 ttl=54 time=13.8 ms
```

A **default route** is the route for every destination that has no more specific route. `ip route` now shows it as `default via 10.20.10.1 dev eth1`.

[NIET GETEST] The commands and outputs of 3.2 on Kali and OPNsense. The routing behaviour and the error messages come from a simulation with the same addresses; the times and the TTL of 1.1.1.1 are examples.

Kali reaches the internet through `halden-fw01`, although its address is private. Why that works, you see in [3.4](#3_4).

::: warning
**Watch out:** the next hop has to be in a network that the host is connected to. A route `via 10.20.20.1` on a workstation in 10.20.10.0/24 is refused: the workstation cannot hand a packet to a router it cannot reach.
:::

What this means for you: a workstation needs one default route. Static routes belong on routers that know more than one way out, and on hosts that are a deliberate exception.

### Check yourself 3.2 – Static and default routes

```text
Sectie: 3.2
Type: meerkeuze
Vraag: A workstation has the address 10.20.10.50/24 and no gateway. You ping 10.20.20.10 and get `Network is unreachable`. What happened to the packet? Choose one answer.
Opties:
  A. It was never sent
  B. The router dropped it
  C. The server ignored it
  D. It was sent to every device in the network
Juist: A
Feedback A: Correct. No route matches the destination, so the host has nowhere to send the packet (see 3.2).
Feedback B: The packet did not reach the router. The host itself reports the error (see 3.2).
Feedback C: A server that ignores a ping gives a timeout, not this message (see 3.2).
Feedback D: A host does not broadcast a packet when it has no route. It refuses to send it (see 3.2).

Sectie: 3.2
Type: categoriseren
Vraag: Is the description about a static route or about a default route?
Indeling:
  Covers one network → Static route
  `ip route add 10.0.2.0/24 via 10.20.10.1` → Static route
  Covers every destination without a more specific route → Default route
  `default via 10.20.10.1 dev eth1` → Default route
  The usual and only route on a workstation → Default route
Feedback: A static route names one network. The default route catches everything that no other route matches (see 3.2).

Sectie: 3.2
Type: meerkeuze
Vraag: A host has these two routes. Which one is used for a packet to 10.0.2.2? Choose one answer.
Code:
  default via 10.20.10.1 dev eth1
  10.0.2.0/24 via 10.20.10.254 dev eth1
Opties:
  A. The route to 10.0.2.0/24, because it is more specific
  B. The default route, because it comes first
  C. Both, in turn
  D. Neither, because two routes conflict
Juist: A
Feedback A: Correct. The most specific matching route wins, whatever its place in the table (see 3.1).
Feedback B: The order of the lines does not decide. The size of the network does (see 3.1).
Feedback C: One packet follows one route (see 3.1).
Feedback D: Several routes may match. That is normal: the default route matches everything (see 3.1).
```

## 3.3 Routing between zones

Kali can reach the internet, and Halden's accountants still cannot open a single client file. The file server has to move into its zone, and the router needs a leg there too.

One router interface on a trunk, with a VLAN interface for every zone, is the usual way to connect VLANs. It is called **inter-VLAN routing**. You built the first leg in [3.2](#3_2). The other two follow the same four steps.

### Two more legs

Repeat the steps of [3.2](#3_2) for these zones:

| Zone | Device | VLAN tag | Address of the router |
| --- | --- | --- | --- |
| Servers | `vlan0.20` | 20 | 10.20.20.1/24 |
| DMZ | `vlan0.40` | 40 | 10.20.40.1/24 |

**System > Routes > Status** now shows four connected routes, one per zone, and a default route to the provider. The router knows every network of Halden & Co without a single static route.

### Move the file server

`halden-srv01` is on a port in VLAN 20, with an address from the old flat network. Log in on its console as `consultant`. You change the address of the server, so an SSH session would cut itself off.

```bash
sudo nmcli connection modify lan ipv4.addresses 10.20.20.10/24
sudo nmcli connection up lan
```

[CONTROLEREN] The connection name `lan` and the sudo rights of `consultant` on `halden-srv01`.

Ping the server from Kali:

```bash
ping -c 2 10.20.20.10
```

```output
PING 10.20.20.10 (10.20.20.10) 56(84) bytes of data.

--- 10.20.20.10 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1018ms
```

Kali has a route. The router has a route. Capture on the server while you ping again:

```bash
sudo tcpdump -i enp0s3 -n -c 2 icmp
```

```output
10:04:43.406911 IP 10.20.10.50 > 10.20.20.10: ICMP echo request, id 269, seq 1, length 64
10:04:44.407885 IP 10.20.10.50 > 10.20.20.10: ICMP echo request, id 269, seq 2, length 64
```

The requests arrive. No reply leaves. The server wants to answer 10.20.10.50 and has no route to that network: you gave it an address and no gateway.

```bash
sudo nmcli connection modify lan ipv4.gateway 10.20.20.1
sudo nmcli connection up lan
```

```output
64 bytes from 10.20.20.10: icmp_seq=1 ttl=63 time=0.110 ms
64 bytes from 10.20.20.10: icmp_seq=2 ttl=63 time=0.075 ms
```

::: warning
**Watch out:** a reply is a packet too, and it needs its own route back. When a ping fails, check the routing table at both ends.
:::

### See the hop

The server answers with `ttl=63`. Inside one network it was 64. Every router lowers the TTL of a packet by one, so the number tells you how many hops a reply passed.

`traceroute` uses that to list the routers on a path:

```bash
traceroute -n 10.20.20.10
```

```output
traceroute to 10.20.20.10 (10.20.20.10), 30 hops max, 60 byte packets
 1  10.20.10.1  0.039 ms  0.004 ms  0.004 ms
 2  10.20.20.10  0.059 ms  0.062 ms  0.045 ms
```

One hop: the router. Then the destination. The web server already has its address and gateway, so `curl http://10.20.40.10/` shows the client portal.

[NIET GETEST] On the lab machines. The outputs of 3.3 come from a simulation with the same addresses and routes.

### What you have now, and what you do not have

Every zone reaches every other zone again. That looks like the flat network you started with, and there is one difference: every packet between two zones now passes `halden-fw01`.

The router lets everything through. `halden-fw01.ova` comes with one rule that allows all traffic, so that you can study routing without a firewall in the way. A workstation can still open an SSH session to the file server, and the web server can still start a connection to a workstation.

What this means for you: you have built the place where decisions can be made. In module 4 you make them.

### Check yourself 3.3 – Routing between zones

```text
Sectie: 3.3
Type: meerkeuze
Vraag: A workstation pings a server in another zone. A capture on the server shows the echo requests, and no replies. What is the most likely cause? Choose one answer.
Opties:
  A. The server has no route back to the network of the workstation
  B. The workstation has no default gateway
  C. The trunk does not carry the VLAN of the server
  D. The router has no interface in the zone of the server
Juist: A
Feedback A: Correct. The requests arrive, so the way there works. The reply needs its own route, and the server has none (see 3.3).
Feedback B: Without a gateway the workstation would not send the request at all, and the server would see nothing (see 3.2).
Feedback C: Then the requests would not reach the server (see 2.3).
Feedback D: Then the router could not deliver the requests to the server (see 3.3).

Sectie: 3.3
Type: numeriek
Vraag: A server sends replies with a TTL of 64. A workstation receives them with `ttl=61`. How many routers did the reply pass?
Juist: 3
Feedback: Every router lowers the TTL by one: 64 − 61 = 3 (see 3.3).

Sectie: 3.3
Type: meerdere antwoorden
Vraag: The router of Halden & Co has VLAN interfaces in Clients, Servers and DMZ. What does a new server in the Servers zone need to reach a workstation? Choose all correct answers.
Opties:
  A. An address in 10.20.20.0/24
  B. The gateway 10.20.20.1
  C. A static route on the router to 10.20.20.0/24
  D. The gateway 10.20.10.1
Juist: A, B
Feedback A: Correct. The server needs an address in the range of its zone (see 3.3).
Feedback B: Correct. The gateway is the address of the router in the server's own zone (see 3.3).
Feedback C: The router is connected to that network. A connected route appears by itself (see 3.1).
Feedback D: That is the router's address in the Clients zone. A host can only use a gateway in its own network (see 3.2).
```

## 3.4 NAT and PAT

In [3.2](#3_2) Kali pinged 1.1.1.1 from the address 10.20.10.50 and got an answer. That should not work. Thousands of companies use 10.20.10.50 inside their own network, so no router on the internet knows a way back to yours.

Some address ranges are reserved for use inside organisations. An address from those ranges is a **private address**. Routers on the internet do not forward packets for them. An address that is reachable on the internet is a **public address**.

| Private ranges |
| --- |
| 10.0.0.0/8 |
| 172.16.0.0/12 |
| 192.168.0.0/16 |

### See the address change

Capture the same ping at two places: before the router and behind it. On Kali:

```bash
sudo tcpdump -i eth1 -n -c 2 icmp
```

On the console of `halden-fw01`, log in as `root` and choose **8) Shell**. Capture on the interface towards the internet:

```bash
tcpdump -i vtnet0 -n -c 2 icmp
```

In a second terminal in Kali, run `ping -c 1 1.1.1.1`. Kali sees:

```output
10:04:59.547204 IP 10.20.10.50 > 1.1.1.1: ICMP echo request, id 286, seq 1, length 64
10:04:59.547251 IP 1.1.1.1 > 10.20.10.50: ICMP echo reply, id 286, seq 1, length 64
```

The router sees, on its outside:

```output
10:04:59.547239 IP 10.0.2.15 > 1.1.1.1: ICMP echo request, id 286, seq 1, length 64
10:04:59.547249 IP 1.1.1.1 > 10.0.2.15: ICMP echo reply, id 286, seq 1, length 64
```

[NIET GETEST] On Kali and OPNsense, including the interface name `vtnet0`. Both captures come from a simulation of the same translation.

It is the same packet, with another source address. **NAT** (network address translation) means that a router replaces an address in a packet that passes. Here the router puts its own outside address in place of Kali's. The answer comes back to the router, which puts Kali's address back and delivers it.

You find the rule in **Firewall > NAT > Source NAT (Outbound)**. The mode is **Automatic Source NAT rule generation**: the router translates every internal network without a rule from you.

### 25 workstations, one address

All workstations leave through the same outside address. When two of them open the same website, the answers come back to one address. The router tells them apart by port.

| Inside | After translation | Destination |
| --- | --- | --- |
| 10.20.10.50, port 49288 | 10.0.2.15, port 51022 | 1.1.1.1, port 443 |
| 10.20.10.51, port 49288 | 10.0.2.15, port 38817 | 1.1.1.1, port 443 |

Translating the port together with the address is **PAT** (port address translation). The router remembers every translation for as long as the connection lives. It is the kind of NAT that almost every network uses.

### The lab is honest about one thing

The outside address of your router, 10.0.2.15, is itself private. In the lab, VirtualBox plays the internet provider and translates once more, and your router at home does it a third time. At Halden & Co the outside address would be a public one.

::: warning
**Watch out:** NAT is not a security measure. It hides addresses, and it happens to drop packets it has no translation for. It does not decide what is allowed. That is the job of the firewall rules in module 4.
:::

What this means for you: a server on the internet sees one address for the whole office. If that server reports abuse from Halden's address, only the records on `halden-fw01` can tell you which workstation it was. You need that in module 8.

### Check yourself 3.4 – NAT and PAT

```text
Sectie: 3.4
Type: categoriseren
Vraag: Is the address private or public?
Indeling:
  10.20.10.50 → Private
  192.168.1.20 → Private
  172.16.5.4 → Private
  1.1.1.1 → Public
  172.32.0.1 → Public
  193.191.0.10 → Public
Feedback: Private are 10.0.0.0/8, 172.16.0.0/12 and 192.168.0.0/16. The range 172.16.0.0/12 ends at 172.31.255.255, so 172.32.0.1 is public (see 3.4).

Sectie: 3.4
Type: meerkeuze
Vraag: Kali (10.20.10.50) pings 1.1.1.1 through a router with the outside address 10.0.2.15. Which source address does a capture on the outside interface of the router show for the request? Choose one answer.
Opties:
  A. 10.0.2.15
  B. 10.20.10.50
  C. 10.20.10.1
  D. 1.1.1.1
Juist: A
Feedback A: Correct. The router replaces the source address with its own outside address (see 3.4).
Feedback B: That is the address before translation, as a capture on Kali shows it (see 3.4).
Feedback C: That is the inside address of the router. It is the gateway of Kali and never appears in the packet (see 3.1).
Feedback D: That is the destination of the request (see 3.4).

Sectie: 3.4
Type: meerkeuze
Vraag: Two workstations open the same website at the same moment, through the same public address. How does the router know which answer belongs to which workstation? Choose one answer.
Opties:
  A. It gave each connection its own source port and remembers which one belongs to whom
  B. The website sends the private address back in the answer
  C. It sends every answer to both workstations
  D. It looks at the MAC address of the website
Juist: A
Feedback A: Correct. That is PAT: the port is translated together with the address (see 3.4).
Feedback B: The website never saw the private address. It was replaced before the packet left (see 3.4).
Feedback C: Each answer goes to one workstation, the one whose translation matches (see 3.4).
Feedback D: MAC addresses do not cross a router. The router sees only the MAC address of the provider's router (see 3.1).
```

## 3.5 Port forwarding

Halden's clients upload their invoices to the client portal on `halden-web01`. That server has the private address 10.20.40.10, and the internet sees one address: the outside of `halden-fw01`. A connection that starts on the internet arrives at the router, which has no translation for it and no idea where it should go.

**Port forwarding** means that a router sends connections that arrive on one of its own ports on to a server inside. The router replaces the destination address this time, which is why it is also called **destination NAT**.

### Your laptop is the internet

In the lab, VirtualBox stands between your router and the outside world, where the provider would be. Tell it to pass one port from your own laptop to the router. Your laptop then plays a client of Halden & Co on the internet.

1. Open the network settings of `halden-fw01`, **Adapter 1**, and click **Port Forwarding**.
2. Add a rule: **Name** `portal`, **Protocol** TCP, **Host IP** `127.0.0.1`, **Host Port** `8080`, **Guest Port** `80`.

### Forward the port on the router

1. In OPNsense, go to **Firewall > NAT > Destination NAT (Port Forward)** and add a rule.
2. Set **Interface** to `WAN`, **Protocol** to `TCP`, **Destination** to `WAN address` and the destination port to `80`.
3. Set **Redirect Target IP** to `10.20.40.10` and **Redirect Target Port** to `80`.
4. Set **Firewall Rule** to `Pass` and the description to `Client portal`. Save and apply.

[CONTROLEREN] All labels in both lists, for VirtualBox 7.2 and OPNsense 26.7.

On your own laptop, not in Kali, open a browser and go to `http://localhost:8080`. You now see the client portal of Halden & Co.

### Follow the connection

A capture of the first packet at two places shows what the router did. On the outside of the router:

```output
10:05:06.014601 IP 10.0.2.2.42704 > 10.0.2.15.80: Flags [S], seq 1772677268, win 64240, options [mss 1460,sackOK,TS val 3843533177 ecr 0,nop,wscale 10], length 0
```

On `halden-web01`:

```output
10:05:06.014629 IP 10.0.2.2.42704 > 10.20.40.10.80: Flags [S], seq 1772677268, win 64240, options [mss 1460,sackOK,TS val 3843533177 ecr 0,nop,wscale 10], length 0
```

[NIET GETEST] On the lab machines. Both lines come from a simulation of the same rule.

The source is the same in both lines. The destination changed from the router to the web server. Compare that with [3.4](#3_4), where the source changed and the destination stayed.

| | Changes | Used for |
| --- | --- | --- |
| Source NAT | The source address | Inside reaches the internet |
| Destination NAT | The destination address | The internet reaches one server inside |

### A hole on purpose

A port forward is an opening from the internet into the network. You made it on purpose, and it deserves three limits.

- Forward to the DMZ only. That zone exists for this.
- Forward one port, the one the service needs.
- Write down every port forward and why it exists.

Tom may ask one day to forward a port straight to the file server, "so the partners can work from home". The request is reasonable. That solution is not: it puts the client files one open port away from the internet.

The portal in the lab speaks HTTP, so that you can read it in a capture. You know from [1.3](./module-1.md#1_3) what that means. A real portal uses HTTPS on port 443.

### Save the state

Shut down all six machines and take a snapshot of each with the name `m3-routed`.

What this means for you: every port forward is a line in your report to the client. For each one you can say which server, which port and which reason.

### Check yourself 3.5 – Port forwarding

```text
Sectie: 3.5
Type: categoriseren
Vraag: Is it source NAT or destination NAT?
Indeling:
  A workstation opens a website on the internet → Source NAT
  The router replaces a private source address with its own → Source NAT
  A client on the internet opens the client portal → Destination NAT
  The router replaces its own address with that of a server inside → Destination NAT
  A port forward → Destination NAT
Feedback: Source NAT lets the inside reach the internet. Destination NAT lets the internet reach one server inside (see 3.5).

Sectie: 3.5
Type: meerkeuze
Vraag: A port forward sends port 80 of the router (10.0.2.15) to 10.20.40.10. A client 10.0.2.2 connects. Which addresses does a capture on the web server show for the first packet? Choose one answer.
Opties:
  A. Source 10.0.2.2, destination 10.20.40.10
  B. Source 10.0.2.15, destination 10.20.40.10
  C. Source 10.0.2.2, destination 10.0.2.15
  D. Source 10.20.40.1, destination 10.20.40.10
Juist: A
Feedback A: Correct. Destination NAT replaces the destination. The source stays the client (see 3.5).
Feedback B: The router does not replace the source in a port forward. The web server sees who the client is (see 3.5).
Feedback C: Those are the addresses before translation, on the outside of the router (see 3.5).
Feedback D: The address of the router in the DMZ is the gateway of the web server. It does not appear in the packet (see 3.1).

Sectie: 3.5
Type: meerkeuze
Vraag: Tom asks you to forward port 445 from the internet to the file server, so that partners can open files at home. What is the best answer? Choose one answer.
Opties:
  A. No: that exposes the file server to the internet, and remote access needs another solution
  B. Yes: one port is a small risk
  C. Yes, if you use a different port number on the outside
  D. No: a router can forward only one port, and port 80 is taken
Juist: A
Feedback A: Correct. You forward to the DMZ only. The need is real, and the answer to it is not a hole to the Servers zone (see 3.5).
Feedback B: The port leads straight to the client files and the payroll data, the assets Halden fears most to lose (see 3.5).
Feedback C: Another number hides nothing. A scan of the outside address finds the port in seconds, as you see in module 5 (see 3.5).
Feedback D: A router can forward many ports. The reason to refuse is the risk, not a limit of the device (see 3.5).
```

## Summary

New terms in this module:

- **router**: a device that is connected to several networks and passes packets from one to the other;
- **routing table** and **route**: the list of networks with the way to reach each one, and one line of it;
- **connected route**: a route for a network on one of your own interfaces;
- **default gateway**: the router a host uses for everything else;
- **hop**: one router that a packet passes;
- **static route** and **next hop**: a route added by hand, and the router it points to;
- **default route**: the route for every destination without a more specific route;
- **inter-VLAN routing**: one router interface on a trunk, with a VLAN interface per zone;
- **private address** and **public address**: an address for use inside organisations, and one that is reachable on the internet;
- **NAT**: a router replaces an address in a packet that passes;
- **PAT**: the port is translated together with the address;
- **port forwarding** or **destination NAT**: a router sends connections on one of its own ports on to a server inside.

The rules to remember:

- A host sends straight to a connected network and hands everything else to its gateway.
- The most specific matching route wins.
- A reply needs a route too. Check both ends.
- Every router lowers the TTL by one.
- Source NAT changes where a packet comes from, destination NAT where it goes to.
- NAT is not a security measure.
- Forward ports to the DMZ only, one port at a time, and write each one down.

## Common mistakes

**Giving a server an address and no gateway.** The requests arrive and the replies never leave. A capture on the server shows it in ten seconds.

**Using a gateway in another network.** A workstation in 10.20.10.0/24 gets the gateway 10.20.20.1. The host cannot reach that router, so the route is refused. The gateway is always the router's address in the host's own zone.

**Reading only one routing table.** You check Kali, find a correct default route and conclude that the router is broken. The table at the other end is half of the path.

**Leaving the shortcut in place.** Adapter 1 of Kali is still connected, so Kali reaches the internet directly and your tests of `halden-fw01` prove nothing. Check with `traceroute -n 1.1.1.1` that the first hop is 10.20.10.1.

**Forgetting the trunk.** The router has an interface in VLAN 40 and the trunk on `halden-sw02` carries only 10 and 20. The interface is up and nothing answers. A new zone is a change on the router and on every trunk on the path.

**Calling NAT a firewall.** "We are behind NAT" says that addresses are translated. It says nothing about what is allowed. In this module the router allows everything.

**Forwarding a port to the wrong zone.** A port forward to 10.20.20.10 works as well as one to the DMZ, which is the problem. The router does not know which zone is meant to be public. You do.

::: oefening
[Exercises module 3](./module-3-exercises.md)
:::
