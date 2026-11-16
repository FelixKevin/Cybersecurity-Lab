---
module: 3
order: 2
tocMode: page
tocLabel: Exercises
published: false
---

# Exercises module 3

You make the exercises E1 to @1 on paper or in your own lab. The case continues the work on Halden & Co. Who can do the core exercises and the case has reached the learning objectives of module 3.

## E1 Which way out? {#e1 .easy}

A laptop in the flower shop Fleur de Lys has this routing table.

```output
default via 192.168.1.1 dev wlan0
192.168.1.0/24 dev wlan0 proto kernel scope link src 192.168.1.37
192.168.50.0/24 via 192.168.1.250 dev wlan0
```

Work out for each destination where the laptop sends the packet: straight to the destination, to 192.168.1.1 or to 192.168.1.250.

Answer with a table of four rows: the destination, the route that matches and where the packet goes.

1. 192.168.1.80
2. 192.168.50.12
3. 8.8.8.8
4. 192.168.5.12

## E2 Private or public? {#e2 .easy}

A log file of a web server lists the addresses of its visitors. Some of them cannot come from the internet.

Decide for each address whether it is private or public.

Answer with a table of six rows: the address, private or public, and for a private address the range it belongs to.

1. 10.200.3.4
2. 172.20.10.2
3. 172.15.0.9
4. 192.168.178.20
5. 192.169.1.1
6. 8.8.4.4

## M1 The printer that never answers {#m1}

The dentists of Tandartsen Noord moved their printer to a new zone. It has the address 10.8.30.20/24. The workstations are in 10.8.10.0/24 with the gateway 10.8.10.1, and the router also has the address 10.8.30.1. A workstation pings the printer and gets no reply. A capture on the port of the printer shows the echo requests.

Work out what is wrong without touching the printer.

Answer with:

- the most likely cause, in one sentence;
- the one observation in the text that rules out a problem on the workstation or the router;
- the setting you would change, and its new value.

## M2 Three captures {#m2}

A student of last year made three captures of one ping and lost the notes that say where each was taken. The ping went from a workstation 10.20.10.50 to 9.9.9.9, through a router with the outside address 10.0.2.15.

```output
A  IP 10.20.10.50 > 9.9.9.9: ICMP echo request, id 41, seq 1, length 64
B  IP 10.0.2.15 > 9.9.9.9: ICMP echo request, id 41, seq 1, length 64
C  IP 9.9.9.9 > 10.20.10.50: ICMP echo reply, id 41, seq 1, length 64
```

Decide for each line on which side of the router it was captured, inside or outside.

Answer with a table of three rows: the line, the side, and the address that told you.

## H1 A second office {#h1 .hard}

The architecture firm Atelier Dubois opens a second office in Ghent. Each office has its own router. The two routers are connected to each other by a leased line.

| | Brussels | Ghent |
| --- | --- | --- |
| Office network | 10.30.1.0/24 | 10.30.2.0/24 |
| Router, office side | 10.30.1.1 | 10.30.2.1 |
| Router, leased line | 172.16.0.1/30 | 172.16.0.2/30 |
| Internet connection | Yes | No |

A workstation in Ghent has to reach the NAS in Brussels and the internet.

Answer with:

- the routing table of each router: for every route the network, the next hop or "connected", and whether it is a connected, static or default route;
- the gateway of a workstation in Ghent;
- the list of routers that a packet from Ghent to the internet passes, in order.

## @1 How traceroute finds the hops {#home1 .home}

In [3.3](./module-3.md#3_3) `traceroute` listed the router between Kali and the file server. It has no special access to that router. It finds it with a trick that uses the TTL.

Read the description in `man traceroute`. Then capture on `eth1` while you run `traceroute -n 10.20.20.10`, and look at the TTL of the packets Kali sends and at the packets that come back from 10.20.10.1.

Answer with three sentences that explain the trick, each with the number of a packet in your capture that shows it.

## C1 Halden & Co: the addressing plan {#c1 .case}

Tom needs to know which machine has which address, and which address a new machine should get.

Starting point: your lab with the snapshot `m3-routed` from [3.5](./module-3.md#3_5).

Write down the addressing plan of the network as it is now.

Answer with:

- a table with one row per zone: the zone, the VLAN, the range and the address of the gateway;
- a table with one row per machine and interface: the machine, the interface, the zone, the address and the gateway;
- the routing table of `halden-fw01`, copied from **System > Routes > Status**, with one sentence per line on where that route comes from.

## C2 Halden & Co: proof that it routes {#c2 .case}

Nadia's staff can work again. You show that with tests that have an expected result.

Run each test from Kali and keep the evidence.

Answer with a table of five rows: the test, the expected result, the result you saw and the evidence.

| Test | Evidence to collect |
| --- | --- |
| Kali reaches the file server | The output of `traceroute -n` |
| Kali reaches the web server | The TTL of the ping reply |
| Kali reaches the internet through the router | The output of `traceroute -n 1.1.1.1`, first hop |
| The router translates the address of Kali | Two capture lines, inside and outside |
| The internet reaches the client portal | A screenshot of the browser on your laptop, and the capture line on `halden-web01` |

## C3 Halden & Co: where things stand {#c3 .case}

You are three weeks into the assignment. Nadia asks for a short status.

Answer with a note of at most 150 words that contains:

- what works again since this module, in words she would use;
- the one opening from the internet that exists, and why;
- an honest sentence on what is **not** protected yet, and in which module that changes.

## Extra exercises {#extra}

These exercises are optional. They repeat the module at different levels.

## X1 Ask the kernel (Easy) {#x1}

Run `ip route get` in Kali for the addresses 10.20.10.1, 10.20.20.10, 10.20.99.1 and 9.9.9.9.

Answer with a table of four rows: the address, the interface and the next hop, or "direct" when there is none.

## X2 A zone for guests (Medium) {#x2}

Halden & Co wants a Guests zone for visitors, with VLAN 50 and the range 10.20.50.0/24. Guests need the internet and nothing else.

Answer with the list of changes, in the order you would make them: on the router, on each switch, and in your documents. You do not have to build it.

## X3 The second port (Hard) {#x3}

The supplier of the client portal asks for SSH access to `halden-web01` from the internet, on port 2222 on the outside.

Build the port forward in your lab and test it from your own laptop. Remove it again afterwards.

Answer with:

- the rule in VirtualBox and the rule in OPNsense, as two tables of field and value;
- the capture line on `halden-web01` that shows the forwarded connection;
- two sentences of advice to Tom on whether this port forward should stay.
