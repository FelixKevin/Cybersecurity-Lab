# Review module 2

Written in English on the structure of the Schrijfwijzer (chapters 9, 11, 12A and 17).

## New terms

| Term | Definition at introduction |
| --- | --- |
| flat network | a network without internal borders |
| broadcast domain | the group of devices that receive each other's broadcasts |
| lateral movement | an attacker moves from the first machine to the next ones |
| segmentation | splitting a network into parts and controlling the traffic between those parts |
| zone | a group of devices with the same task and the same need for protection |
| switch | connects the devices of one network and delivers each frame to the right port |
| VLAN / VLAN ID | a separate network inside a switch / its number, 1 to 4094 |
| access port | a port that belongs to one VLAN |
| VLAN tag | four extra bytes in the frame that hold the VLAN ID |
| trunk port / trunk | a port that carries tagged frames of several VLANs / the link between two such ports |
| 802.1Q | the standard that describes the tag |
| Open vSwitch | switch software for Linux |
| port plan | the list of every switch port with its VLAN and what is connected to it |

"Flat network" was mentioned without bold in 0.1 and is introduced here. "DMZ" is used in 2.4 as the name of a zone and is introduced as a term in module 4.

## Decisions made while writing (confirm or change)

- **The switch is Open vSwitch on a small Linux VM**, not a Linux bridge and not an emulated hardware switch. Reasons: vendor-neutral, three short commands cover access ports and trunks, the configuration survives a restart without extra files, and it is used in practice (virtualisation platforms, cloud networking).
- **A second switch** (`halden-sw02`, the server room) is added in 2.3, because a trunk needs two VLAN-aware devices. It is the same appliance imported twice. From this module on the lab runs four machines.
- **A "cable" is a VirtualBox internal network with two machines on it**, named after the switch port (`sw01-port1`, `trunk-sw01-sw02`). The network `halden-lab` from module 0 is no longer used.
- **Addresses do not change in this module.** Kali and the server stay in 10.20.30.0/24 and are separated by VLAN only. The move to one range per zone is module 3.
- **Zone plan:** Clients VLAN 10 (10.20.10.0/24), Servers VLAN 20 (10.20.20.0/24), DMZ VLAN 40 (10.20.40.0/24), Management VLAN 99 (10.20.99.0/24). VLAN 30 is skipped so that the old flat range 10.20.30.0/24 is not confused with a zone.
- **The lab ends in a state where nothing crosses the zones.** That is deliberate and is said in the text; module 3 opens the crossings.
- **Native VLAN is not covered here.** It is the topic of the @home exercise and returns with VLAN hopping in module 6.
- **Switch administration is on the VM console** in this module. Remote management over VLAN 99 needs addresses and comes later.

## Requirements for `halden-sw01.ova`

- Small Linux (512 MB RAM is enough), Open vSwitch installed and started at boot.
- One bridge `br0` with four ports named `port1` to `port4` (rename the interfaces, for example with systemd `.link` files), no `tag` and no `trunks` set.
- Four VirtualBox adapters on internal networks `sw01-port1` to `sw01-port4`.
- **Promiscuous mode "Allow All" on all four adapters.** Without it VirtualBox does not hand frames for other MAC addresses to the VM, and the switch forwards nothing.
- **An adapter type that passes 802.1Q tags.** Some emulated Intel adapters in VirtualBox strip VLAN tags; the paravirtualised adapter (virtio-net) is the usual choice. This has to be tested: section 2.3 depends on it.
- Account `netadmin` with sudo, `tcpdump` installed.
- Console keyboard layout: Belgian AZERTY is what most students have. Decide on the layout, or tell students how to change it.
- Importing the appliance twice must give a working second switch (new MAC addresses, hostname changed by the student).

## Open questions

- 2.1 has no hands-on step. Add a short capture that shows a broadcast of another machine arriving on Kali, or leave it conceptual?
- 2.3, step 4: changing the internal network of a running VM. If this is unreliable in VirtualBox 7.2, shut down `halden-sw01` first.
- 2.5: unused ports. The text puts the free desk port in VLAN 10. Should there be a separate "parking" VLAN for unused ports, as is common practice?
- The switches cannot hand over capture files until they have an address (C2 reads the file on the switch). Acceptable, or give the switches a management address already in this module?
- Example solutions (O6) are not written yet.

## Markers

- [CONTROLEREN] What you need: `halden-sw01.ova` does not exist yet.
- [CONTROLEREN] 2.2, Put a switch in your lab: the account `netadmin`, its password and the adapter names.
- [CONTROLEREN] 2.3, Add the second switch: labels in steps 2 and 4 for VirtualBox 7.2.
- [NIET GETEST] 2.3, See the tag: both captures on the appliances in VirtualBox.

## What was checked

All Open vSwitch commands in the text, quizzes and exercises were run in a simulation: Open vSwitch 3.3.9 with the userspace datapath, two bridges, hosts in network namespaces with the lab addresses.

- Flat switch: ping works. Both ports in VLAN 20: works. VLAN 10 against VLAN 20: 100% packet loss.
- Trunk with `trunks=10,20` on both sides: works. Trunk with only VLAN 10: fails.
- A port without `tag` and without `trunks` carries tagged frames of every VLAN.
- The two tcpdump outputs in 2.3 are real output of that simulation (frame length 102 with tag, 98 without); times and MAC addresses were replaced by examples.
- The output of `ovs-vsctl show` is real, with the line `datapath_type: netdev` removed and the ports put in order.

Not checked: anything in VirtualBox, in particular whether tagged frames pass between two VMs on an internal network.
