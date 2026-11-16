# Review module 3

Written in English on the structure of the Schrijfwijzer (chapters 9, 11, 12A and 17).

## New terms

| Term | Definition at introduction |
| --- | --- |
| router | a device that is connected to several networks and passes packets from one to the other |
| routing table / route | a list of networks, each with the way to reach it / one line of that table |
| connected route | a route for a network that is on one of your own interfaces |
| default gateway | the router that a host uses for everything else |
| hop | each router a packet passes |
| static route | a route that an administrator adds by hand |
| next hop | the router you hand the packet to |
| default route | the route for every destination that has no more specific route |
| inter-VLAN routing | one router interface on a trunk, with a VLAN interface for every zone |
| private address / public address | an address from the ranges reserved for use inside organisations / an address that is reachable on the internet |
| NAT | a router replaces an address in a packet that passes |
| PAT | translating the port together with the address |
| port forwarding / destination NAT | a router sends connections that arrive on one of its own ports on to a server inside |

## Decisions made while writing (confirm or change)

- **The router is OPNsense (`halden-fw01`), from this module on.** It is the device that becomes the firewall in module 4, so students configure one box across two modules instead of replacing a router. The alternative, a plain Linux router in module 3, would have been fully testable here but is thrown away a week later.
- **The appliance arrives with a rule that allows everything.** Module 3 is about routing; the firewall rules are module 4. The text says so in 3.3.
- **A "Prepare your lab" block before 3.1** holds the import and cabling, so that the five sections stay about the concepts.
- **Kali gets a third adapter in the Management zone** and is from now on both a workstation (Clients) and the administrator's laptop (Management). That bridges two zones on one machine, which the zone design forbids for real workstations; the text names the two roles.
- **Kali loses its direct internet connection** (adapter 1, cable disconnected) and reaches the internet through `halden-fw01`.
- **`port3` of `halden-sw02` changes from a Management access port to the trunk to the router.** VirtualBox shows four adapters per machine, so there was no free port.
- **Static routes are taught on Kali with the provider's network (10.0.2.0/24) as destination.** In a network with one router no static route is needed, so any example is somewhat artificial; H1 gives the real use case with two routers.
- **A DMZ web server (`halden-web01`) is added now**, because port forwarding needs a target in the DMZ. The lab runs six machines from here on.
- **The "internet" for the port forward is the student's own laptop**, through a VirtualBox port-forwarding rule in front of the router. Two forwards in a row, which the text uses as illustration.
- **The old range 10.20.30.0/24 is retired.** Addresses: Kali 10.20.10.50 and 10.20.99.50, file server 10.20.20.10, web server 10.20.40.10, router .1 in every zone.
- **DHCP for the Clients zone is not configured.** Kali has a static address.

## Requirements for `halden-fw01.ova`

- OPNsense 26.7, 2 GB RAM, two paravirtualised adapters: adapter 1 on NAT (WAN, DHCP), adapter 2 on internal network `trunk-sw02-fw01`.
- A Management interface prebuilt: VLAN 99 on the second adapter (`vlan0.99`), address 10.20.99.1/24. No untagged interface on the trunk.
- Web interface reachable on the Management interface, preferably only there (System > Settings > Administration, listen interfaces). Otherwise the redirect on port 80 of the WAN address may get in the way of 3.5.
- `root` with a password that students receive.
- One floating rule that passes all traffic on all interfaces, with a description that says it is replaced in module 4.
- On the WAN interface: "Block private networks" and "Block bogon networks" switched off. The lab's WAN is a private range, and the port forward arrives from 10.0.2.2.
- Source NAT in automatic mode (the default).
- The text assumes the interface names `vtnet0` (WAN) and `vtnet1` (trunk).

## Requirements for `halden-web01.ova`

- Small Linux, one adapter on internal network `sw02-port2`.
- Static address 10.20.40.10/24, gateway 10.20.40.1.
- Web server on port 80 without encryption with a page titled "Halden & Co client portal".
- Account `consultant` with sudo for `tcpdump`; SSH on port 22 (used in X3).

## Added requirements for `halden-srv01.ova`

- NetworkManager, with the connection of its adapter named `lan`.
- `consultant` may run `nmcli` with sudo.

## Open questions

- **OPNsense is renaming its NAT pages.** Port forwarding became "Destination NAT" in 26.1 and outbound NAT is becoming "Source NAT" in 26.7. The labels in 3.2, 3.4 and 3.5 follow the current manual, but they have to be checked against the exact version in the appliance, and again each year.
- Six machines: is the memory use acceptable in practice? Estimated 9 GB for the guests (Kali 4, router 2, servers and switches 3).
- 3.2: the example static route points at the provider's network and works only because the router translates. Acceptable, or prefer a different destination?
- 3.4: students work in the shell of the router as `root`. Acceptable, or use the packet capture page of the web interface instead?
- Should the Clients zone get DHCP from the router in this module, as a real office would have?
- Example solutions (O6) are not written yet.

## Markers

- [CONTROLEREN] What you need: both appliances do not exist yet.
- [CONTROLEREN] Prepare your lab: the password of `root`, adapter names, the interface name `eth2`.
- [NIET GETEST] Prepare your lab: the `nmcli` commands on Kali.
- [NIET GETEST] 3.1: the routing table of Kali.
- [CONTROLEREN] 3.2, Give the router a leg: all labels for OPNsense 26.7 and the parent interface.
- [NIET GETEST] 3.2: commands and outputs on Kali and OPNsense.
- [CONTROLEREN] 3.3, Move the file server: the connection name `lan` and sudo rights.
- [NIET GETEST] 3.3: outputs on the lab machines.
- [NIET GETEST] 3.4: both captures and the interface name `vtnet0`.
- [CONTROLEREN] 3.5: all labels for VirtualBox 7.2 and OPNsense 26.7.
- [NIET GETEST] 3.5: both capture lines.

## What was checked

The routing and NAT behaviour was run in a simulation: Linux network namespaces for Kali, the router, the provider, the file server and the web server, with the lab addresses, IP forwarding and nftables for source and destination NAT.

- Without a gateway: `ping: connect: Network is unreachable`. With a static route to 10.0.2.0/24: the provider's router answers and 1.1.1.1 stays unreachable. With a default route: both answer.
- A server with an address and no gateway: the capture on the server shows the requests and no replies. With the gateway: replies with `ttl=63`.
- `traceroute -n` shows the router as hop 1 and the server as hop 2.
- Source NAT: the two captures in 3.4 are real output (10.20.10.50 inside, 10.0.2.15 outside).
- Destination NAT: the two lines in 3.5 are real output, and a request to the outside address returned the portal page.
- The OPNsense menu paths were compared with the current manual pages for VLAN devices and NAT.

Not checked: anything in OPNsense or VirtualBox, the `nmcli` commands, and the NetworkManager format of the routing table on Kali.
