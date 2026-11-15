---
module: 2
order: 2
tocMode: page
tocLabel: Exercises
published: false
---

# Exercises module 2

You make the exercises E1 to @1 on paper or in your own lab. The case continues the work on Halden & Co. Who can do the core exercises and the case has reached the learning objectives of module 2.

## E1 Who hears the broadcast? {#e1 .easy}

The bike shop Velo Marolles has one switch with eight ports. Ports 1 to 5 are in VLAN 10, ports 6 and 7 are in VLAN 20, and port 8 is in VLAN 30. A device on port 2 sends an ARP request.

Work out which ports receive that request, and how many broadcast domains the switch has.

Answer with the list of ports and one number.

## E2 Access or trunk? {#e2 .easy}

A new colleague at your consultancy has to cable a small office. For each connection, the colleague asks which kind of port to configure.

Decide for each connection whether the switch port is an access port or a trunk port. Give the VLAN or the VLANs it needs, using the zone table from [2.4](./module-2.md#2_4).

Answer with a table of five rows: the connection, the mode and the VLANs.

1. A workstation of an accountant.
2. The cable to a second switch that has workstations and servers.
3. The file server.
4. The cable to a second switch that has only workstations.
5. The public web server.

## M1 Read the switch {#m1}

The notary office Claes & Partners asks you why two of their machines cannot reach each other. Their administrator sends you the configuration of the only switch.

```output
    Bridge br0
        Port port1
            tag: 10
            Interface port1
        Port port2
            tag: 20
            Interface port2
        Port port3
            tag: 10
            Interface port3
        Port port4
            Interface port4
```

A PC is on `port1`, a NAS on `port2`, a second PC on `port3` and a printer on `port4`. All four have an address in the same range.

Answer with:

- a table that shows for each pair of devices whether they can reach each other;
- the one port you would change first, and why, in two sentences.

## M2 One VLAN too few {#m2}

The trunk between two switches of the print shop Drukkerij Lemmens carries `trunks=10,20`. The owner adds a camera system in VLAN 50, with cameras on the first switch and the recorder on the second. The recorder shows no images.

Rebuild the situation in your lab: use `port1` of `halden-sw01` and `port1` of `halden-sw02` as the two camera ports, and reproduce the fault. Then fix it. Restore the configuration of [2.5](./module-2.md#2_5) when you are done.

Answer with:

- the command that fixes the fault, and the switch or switches where you run it;
- the two lines of `tcpdump` output on the trunk that show the fix works.

## H1 Zones for an architect {#h1 .hard}

The architecture firm Atelier Dubois has twelve employees and asks you for a zone design. This is what they have:

- twelve workstations with expensive design software;
- a NAS with all project drawings, which is also their only backup;
- a plotter and two printers;
- guest Wi-Fi for clients who visit;
- four security cameras with a recorder;
- one switch and one internet connection.

Design the zones. You decide how many there are.

Answer with:

- a zone table like the one in [2.4](./module-2.md#2_4): zone, VLAN, who lives there and why it is its own zone;
- a table of the traffic that has to cross between zones;
- one risk that your design does **not** solve, in two sentences.

## @1 What happens to frames without a tag? {#home1 .home}

A trunk carries tagged frames. A device that does not know VLANs sends frames without a tag. What a trunk port does with those depends on a setting.

Read the section on `vlan_mode` in the manual page of the Open vSwitch database: `man ovs-vswitchd.conf.db`. Find the four modes a port can have.

Answer with a table of four rows: the mode, and one sentence in your own words on what the port does with a frame without a tag.

## C1 Halden & Co: the port plan {#c1 .case}

Tom has to maintain the switches after you leave. He needs a document that matches what is configured.

Starting point: your lab with the snapshot `m2-zones` from [2.5](./module-2.md#2_5).

Run `sudo ovs-vsctl show` on both switches and compare the output with the port plan in [2.5](./module-2.md#2_5).

Answer with:

- the port plan as a table, with one extra column: the line of the output that confirms each row;
- the output of both switches as text.

## C2 Halden & Co: proof that the zones hold {#c2 .case}

A port plan shows what you intended. Nadia wants to know that it works.

Run the three tests from [2.5](./module-2.md#2_5). During test 2, capture the ping on the trunk port of `halden-sw01` and save it as `m2-trunk.pcap`. You cannot copy the file to Kali yet, because the switches have no address. Read it on the switch with `sudo tcpdump -e -n -r m2-trunk.pcap`.

Answer with a table of three rows: the test, the expected result, the result you saw, and the evidence. For test 2 the evidence is the line of the capture that shows the VLAN tag.

## C3 Halden & Co: the design in three sentences {#c3 .case}

Nadia approves budgets, not VLAN IDs. She asks why her office needs four zones when it worked fine with one.

Answer with a note of at most 120 words. It names each of her three fears from [0.1](./module-0.md#0_1) and says which zone makes that fear smaller. It also states honestly what does not work yet after this module.

## Extra exercises {#extra}

These exercises are optional. They repeat the module at different levels.

## X1 Move a desk (Easy) {#x1}

An accountant of Halden & Co moves to the free desk on `port2` of `halden-sw01`. A technician needs that port for a management laptop for one afternoon.

Answer with the command that puts `port2` in the Management zone, and the command that puts it back.

## X2 A zone for the printer (Medium) {#x2}

The printer of Halden & Co is in the Clients zone. Printers are often badly maintained and hard to update.

Decide whether the printer should get its own zone.

Answer with your decision, two arguments for it and the strongest argument against it.

## X3 Three switches (Hard) {#x3}

Halden & Co rents a second floor. It gets its own switch with workstations and one management port. The new switch connects to `halden-sw01`, not to the server room.

Answer with:

- the port plan of the new switch;
- the changes to the port plan of `halden-sw01`;
- the list of VLANs on each of the two trunks, with a reason for every VLAN you leave out.
