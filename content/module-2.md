---
module: 2
order: 1
moduleTitle: "2: Segmentation"
published: false
---

# Module 2: Segmentation

Halden & Co has one network in which every device can reach every other device. In this module you split that network into zones with VLANs. You learn how a switch keeps zones apart, how zones cross a cable between two switches, and how you decide which zones a company needs.

### Learning objectives

After this module you can:

- explain why a flat network makes an incident worse;
- assign switch ports to VLANs and predict which hosts can still reach each other;
- configure a trunk between two switches and show the VLAN tag in a capture;
- design zones for a small company and give a reason for each zone;
- document and verify the port configuration of a switch.

### What you need

This module adds two switches to your lab. Both come from the same file.

| What | Requirement |
| --- | --- |
| Your lab from module 1 | Kali and `halden-srv01`, with the snapshot `m0-clean` |
| Lab file | `halden-sw01.ova`, from the course files on Canvas |
| Running at the same time | At most four machines: Kali, `halden-srv01` and two switches |

[CONTROLEREN] `halden-sw01.ova` does not exist yet. The requirements are in the review notes.

## 2.1 The flat network problem

In module 1 you sat at a workstation of Halden & Co, read the intranet and logged in on the server over SSH. Nothing stopped you, and nothing was supposed to: on Halden's network every device can talk to every other device.

A network like that is a **flat network**: a network without internal borders. It is the easiest network to build, which is why so many small companies have one.

### One network, one problem for everyone

Think back to the ARP request in [1.4](./module-1.md#1_4). Its destination was Broadcast, so every machine on the network received it. The group of devices that receive each other's broadcasts is a **broadcast domain**. At Halden & Co that is all of them: 25 workstations, the file server, the printer and the public web server.

### Hear a question that was not meant for you

You can see this in your lab, which is still flat. You need two terminals in Kali.

1. In the first terminal, capture one ARP packet on the lab network:

```bash
sudo tcpdump -i eth1 -n -c 1 arp
```

2. In the second terminal, log in on the server and let it look for an address that does not exist:

```bash
ssh consultant@10.20.30.10
ping -c 1 10.20.30.99
```

The ping fails, as it should. The first terminal now shows:

```output
14:26:31.954640 ARP, Request who-has 10.20.30.99 tell 10.20.30.10, length 28
```

[NIET GETEST] On Kali and `halden-srv01`. The output comes from a simulation of the same network.

The server asked a question, and Kali received it. Kali was not the sender and not the target. On a flat network every workstation hears what every other device is looking for.

### What an infected workstation can reach

Now take the second fear of Nadia: ransomware. Someone at the front desk opens the wrong attachment. What can that workstation reach?

| From the infected workstation | Reachable on a flat network? |
| --- | --- |
| The other 24 workstations | Yes |
| The file server with client files and payroll | Yes |
| The printer | Yes |
| The public web server | Yes |
| The configuration page of the switch | Yes |

An attacker who moves from the first machine to the next ones is making a **lateral movement**. On a flat network there is nothing in the way.

### The fix: borders inside the network

**Segmentation** means that you split a network into parts and control the traffic between those parts. Each part is a **zone**: a group of devices with the same task and the same need for protection.

Segmentation does not stop the first infection. It decides how far the damage can spread afterwards. A workstation that cannot reach the file server cannot encrypt it.

You could give every zone its own switch and its own cables. For a company of 25 people that is expensive, and it breaks the day someone changes desks. In [2.2](#2_2) you get the same result on the hardware that is already there.

What this means for you: when you look at a network, your first question is no longer "does it work?". It is "what can this machine reach that it does not need?".

### Check yourself 2.1 – The flat network problem

```text
Sectie: 2.1
Type: meerkeuze
Vraag: A workstation on a flat network is infected with ransomware. What does segmentation change about that situation? Choose one answer.
Opties:
  A. It limits which other devices the workstation can reach
  B. It prevents the workstation from being infected
  C. It removes the ransomware from the workstation
  D. It encrypts the traffic between the devices
Juist: A
Feedback A: Correct. Segmentation does not stop the first infection. It limits how far the damage spreads (see 2.1).
Feedback B: The attachment is still opened. Segmentation works after that moment, not before it (see 2.1).
Feedback C: Segmentation is a property of the network. It changes nothing on the infected machine (see 2.1).
Feedback D: Segmentation separates traffic. Encryption is a different measure (see 2.1).

Sectie: 2.1
Type: koppelen
Vraag: Match each term with its description.
Paren:
  Flat network → A network without internal borders
  Broadcast domain → The devices that receive each other's broadcasts
  Lateral movement → An attacker moves from one machine to the next
  Zone → A group of devices with the same task and need for protection
Feedback: On a flat network there is one broadcast domain and no zone, so lateral movement meets no border (see 2.1).

Sectie: 2.1
Type: numeriek
Vraag: A company has one flat network with 30 workstations, 2 servers and 3 printers. How many broadcast domains does it have?
Juist: 1
Feedback: A flat network is one broadcast domain, however many devices are on it (see 2.1).
```

## 2.2 VLANs

Halden & Co has one switch on the office floor. Workstations and the server are plugged into it. You want two separate networks, and you are not getting a second switch.

A **switch** connects the devices of one network and delivers each frame to the right port. A **VLAN** (virtual LAN) is a separate network inside a switch: ports in the same VLAN can exchange frames, ports in different VLANs cannot. Each VLAN has a number, the **VLAN ID**, from 1 to 4094.

### Put a switch in your lab

Until now `halden-lab` connected your two machines directly. You replace it with a switch and two cables. In VirtualBox, a cable is an internal network with exactly two machines on it.

1. Shut down Kali and `halden-srv01`.
2. Import `halden-sw01.ova` with **File > Import Appliance**. You now see `halden-sw01` in the list, with four network adapters.
3. Open the network settings of Kali. Change the name of the internal network of **Adapter 2** from `halden-lab` to `sw01-port1`.
4. Open the network settings of `halden-srv01`. Change the name of the internal network of **Adapter 1** to `sw01-port2`.
5. Start all three machines.

| Cable (internal network) | One end | Other end |
| --- | --- | --- |
| `sw01-port1` | Kali, adapter 2 | `halden-sw01`, `port1` |
| `sw01-port2` | `halden-srv01`, adapter 1 | `halden-sw01`, `port2` |

Log in on the console of `halden-sw01` with the user `netadmin`. You work in the window of the switch itself, the way an administrator connects a cable to the console port of a real switch.

[CONTROLEREN] The account `netadmin`, its password and the adapter names in the appliance.

Look at the configuration:

```bash
sudo ovs-vsctl show
```

```output
d64098d6-936a-443c-b797-bc5dc5d6bd40
    Bridge br0
        Port port1
            Interface port1
        Port port2
            Interface port2
        Port port3
            Interface port3
        Port port4
            Interface port4
        Port br0
            Interface br0
                type: internal
    ovs_version: "3.3.9"
```

The switch runs **Open vSwitch**, switch software for Linux. `br0` is the switch, and `port1` to `port4` are its ports. No port has a VLAN yet. The order of the ports in the output can differ. In Kali, run `ping -c 2 10.20.30.10`: you get replies. The network is still flat, now with a switch in the middle.

### Assign ports to VLANs

Put both ports in VLAN 20:

```bash
sudo ovs-vsctl set port port1 tag=20
sudo ovs-vsctl set port port2 tag=20
```

Ping again from Kali. It still works: both ports are in the same VLAN. Now move the port of Kali to VLAN 10:

```bash
sudo ovs-vsctl set port port1 tag=10
```

```output
PING 10.20.30.10 (10.20.30.10) 56(84) bytes of data.

--- 10.20.30.10 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1014ms
```

Nothing changed on Kali or on the server. They have the same addresses and the same cables. The switch no longer passes frames between them.

A port that belongs to one VLAN is an **access port**. The device on it does not know that VLANs exist. The general form is:

```text
ovs-vsctl set port <port> tag=<vlan id>
```

To remove a port from its VLAN, use `ovs-vsctl clear port <port> tag`. Open vSwitch saves every change at once: the configuration survives a restart.

::: warning
**Watch out:** the two machines are in the same IP range and still cannot reach each other. A VLAN works below IP addresses. In a finished design every VLAN gets its own range, and you do that in module 3.
:::

Other switches use other commands for the same idea. On most hardware switches every port starts as an access port in VLAN 1. What you learn here is the concept; the syntax you look up per brand.

Set `port1` back to VLAN 20 before you continue, and check that the ping works again.

What this means for you: a VLAN is a decision per port. The question "which zone is this device in?" has the same answer as "which VLAN is its port in?".

### Check yourself 2.2 – VLANs

```text
Sectie: 2.2
Type: meerkeuze
Vraag: A switch has this configuration. Which hosts can ping each other? Choose one answer.
Code:
  port1  tag: 10   host A  10.20.30.11
  port2  tag: 20   host B  10.20.30.12
  port3  tag: 10   host C  10.20.30.13
Opties:
  A. Only A and C
  B. A, B and C
  C. Only A and B
  D. No hosts
Juist: A
Feedback A: Correct. Ports in the same VLAN exchange frames. `port1` and `port3` are both in VLAN 10 (see 2.2).
Feedback B: The hosts are in the same IP range, but host B is in another VLAN. The switch does not pass frames between VLANs (see 2.2).
Feedback C: A and B are next to each other on the switch, but in different VLANs. The position of a port does not matter (see 2.2).
Feedback D: A and C are in the same VLAN, so they can reach each other (see 2.2).

Sectie: 2.2
Type: meerkeuze
Vraag: Which command puts `port3` in VLAN 40? Choose one answer.
Opties:
  A. `sudo ovs-vsctl set port port3 tag=40`
  B. `sudo ovs-vsctl set port port3 vlan=40`
  C. `sudo ovs-vsctl set port 40 tag=port3`
  D. `sudo ovs-vsctl set vlan 40 port=port3`
Juist: A
Feedback A: Correct. The form is `ovs-vsctl set port <port> tag=<vlan id>` (see 2.2).
Feedback B: Open vSwitch calls the VLAN of an access port its `tag` (see 2.2).
Feedback C: The name of the port comes first, then the VLAN ID as the value of `tag` (see 2.2).
Feedback D: You configure a port, not a VLAN. A VLAN exists as soon as a port uses its number (see 2.2).

Sectie: 2.2
Type: meerdere antwoorden
Vraag: You move the port of a workstation from VLAN 20 to VLAN 10. What changes on the workstation itself? Choose all correct answers.
Opties:
  A. Nothing changes on the workstation
  B. Its IP address changes
  C. Its MAC address changes
  D. It can reach other devices than before
Juist: A, D
Feedback A: Correct. The VLAN is a setting of the switch port. The workstation does not know about it (see 2.2).
Feedback B: The switch does not change addresses. The workstation keeps the address it had (see 2.2).
Feedback C: A MAC address belongs to the network adapter and does not depend on the switch (see 2.2).
Feedback D: Correct. It now exchanges frames with the ports in VLAN 10, and no longer with those in VLAN 20 (see 2.2).
```

## 2.3 Trunks and tagging

The file server of Halden & Co does not stand on the office floor. It is in a small server room, on a second switch. Workstations in VLAN 10 and servers in VLAN 20 exist on both sides, and there is one cable between the two switches.

One cable has to carry several VLANs, and the switch at the other end has to know which frame belongs to which VLAN. The sending switch adds a label to each frame. That label is the **VLAN tag**: four extra bytes in the frame that hold the VLAN ID. A port that carries tagged frames of several VLANs is a **trunk port**, and the link between two such ports is a **trunk**.

### Add the second switch

1. Shut down `halden-srv01`.
2. Import `halden-sw01.ova` a second time. In the import window, change the name to `halden-sw02` and choose **Generate new MAC addresses for all network adapters**.
3. In the network settings of `halden-sw02`, rename the internal networks of adapters 1 to 3 to `sw02-port1`, `sw02-port2` and `sw02-port3`.
4. Give **Adapter 4** of both switches the same internal network: `trunk-sw01-sw02`. For `halden-sw01` you can change this while it runs, in **Devices > Network > Network Settings** of its window.
5. Move the server: change **Adapter 1** of `halden-srv01` to `sw02-port1`.
6. Start `halden-sw02` and `halden-srv01`.

[CONTROLEREN] Labels in steps 2 and 4 for VirtualBox 7.2.

| Cable (internal network) | One end | Other end |
| --- | --- | --- |
| `sw01-port1` | Kali | `halden-sw01`, `port1` |
| `trunk-sw01-sw02` | `halden-sw01`, `port4` | `halden-sw02`, `port4` |
| `sw02-port1` | `halden-srv01` | `halden-sw02`, `port1` |

Log in on `halden-sw02` and give it its own name, so that you always see which switch you are typing on:

```bash
sudo hostnamectl set-hostname halden-sw02
```

Log out and in again. The prompt now ends in `halden-sw02`.

### Configure the trunk

On `halden-sw02`, put the port of the server in VLAN 20:

```bash
sudo ovs-vsctl set port port1 tag=20
```

On both switches, make `port4` a trunk for VLAN 10 and VLAN 20:

```bash
sudo ovs-vsctl set port port4 trunks=10,20
```

In Kali, run `ping -c 2 10.20.30.10`. You get replies: Kali is in VLAN 20 on the first switch, the server is in VLAN 20 on the second, and the trunk carries VLAN 20 between them.

The general form is:

```text
ovs-vsctl set port <port> trunks=<vlan id>,<vlan id>
```

### See the tag

On `halden-sw01`, capture on the trunk port while you ping from Kali. The option `-e` makes tcpdump show the Ethernet header.

```bash
sudo tcpdump -i port4 -e -n -c 2 vlan
```

```output
14:20:58.261184 08:00:27:74:1b:32 > 08:00:27:df:2f:4f, ethertype 802.1Q (0x8100), length 102: vlan 20, p 0, ethertype IPv4 (0x0800), 10.20.30.50 > 10.20.30.10: ICMP echo request, id 627, seq 1, length 64
14:20:58.261322 08:00:27:df:2f:4f > 08:00:27:74:1b:32, ethertype 802.1Q (0x8100), length 102: vlan 20, p 0, ethertype IPv4 (0x0800), 10.20.30.10 > 10.20.30.50: ICMP echo reply, id 627, seq 1, length 64
```

Each frame says `802.1Q` and `vlan 20`. **802.1Q** is the standard that describes the tag. Now capture the same ping on the access port:

```bash
sudo tcpdump -i port1 -e -n -c 2 icmp
```

```output
14:21:02.272329 08:00:27:74:1b:32 > 08:00:27:df:2f:4f, ethertype IPv4 (0x0800), length 98: 10.20.30.50 > 10.20.30.10: ICMP echo request, id 634, seq 1, length 64
14:21:02.272711 08:00:27:df:2f:4f > 08:00:27:74:1b:32, ethertype IPv4 (0x0800), length 98: 10.20.30.10 > 10.20.30.50: ICMP echo reply, id 634, seq 1, length 64
```

No tag, and the frame is four bytes shorter: 98 instead of 102. The switch adds the tag when a frame leaves through a trunk port and removes it before the frame reaches a device.

[NIET GETEST] Both captures on the appliances in VirtualBox. Commands and output come from a simulation with Open vSwitch 3.3.9; the MAC addresses are examples.

### Only what is needed

Change the trunk on `halden-sw01` so that it carries only VLAN 10, and ping again:

```bash
sudo ovs-vsctl set port port4 trunks=10
```

The ping fails. VLAN 20 no longer crosses the cable. Set the trunk back to `trunks=10,20`.

::: warning
**Watch out:** in Open vSwitch a port without `tag` and without `trunks` is a trunk port that carries every VLAN. A port you forgot to configure is the most generous port on the switch. In [2.5](#2_5) you give every unused port a safe place.
:::

What this means for you: list on every trunk only the VLANs that have devices on both sides. A VLAN that is not on the trunk cannot leak over it.

### Check yourself 2.3 – Trunks and tagging

```text
Sectie: 2.3
Type: categoriseren
Vraag: Does the description fit an access port or a trunk port?
Indeling:
  Belongs to exactly one VLAN → Access port
  Connects a workstation → Access port
  Sends frames without a VLAN tag → Access port
  Carries frames of several VLANs → Trunk port
  Connects two switches → Trunk port
  Sends frames with a VLAN tag → Trunk port
Feedback: An access port belongs to one VLAN and the device on it sees normal frames. A trunk port carries several VLANs and marks each frame with a tag (see 2.3).

Sectie: 2.3
Type: meerkeuze
Vraag: Host A is on `sw01` in VLAN 20. Host B is on `sw02` in VLAN 20. The trunk between the switches has `trunks=10,40` on both sides. Can A ping B? Choose one answer.
Opties:
  A. No, because the trunk does not carry VLAN 20
  B. Yes, because both hosts are in the same VLAN
  C. Yes, because a trunk carries every VLAN
  D. No, because hosts on different switches can never be in the same VLAN
Juist: A
Feedback A: Correct. The frames of VLAN 20 are not allowed on the trunk, so they never reach the other switch (see 2.3).
Feedback B: The same VLAN ID on both switches is not enough. The trunk between them has to carry that VLAN (see 2.3).
Feedback C: This trunk has a list, and VLAN 20 is not on it (see 2.3).
Feedback D: A VLAN can span several switches. That is what a trunk is for (see 2.3).

Sectie: 2.3
Type: numeriek
Vraag: An untagged frame is 98 bytes long. How many bytes is the same frame on a trunk, with a VLAN tag?
Juist: 102
Feedback: The tag adds four bytes to the frame (see 2.3).
```

## 2.4 Designing zones

You can now build any number of VLANs. The harder question is which ones Halden & Co needs. Ten zones are not safer than four if nobody can explain what the other six are for.

### Start from the assets

Go back to the table of assets in [0.1](./module-0.md#0_1), and to what Nadia fears. Group the devices by two questions: what do they need to reach, and how bad is it when they are compromised?

| Zone | VLAN | Planned range | Who lives there | Why its own zone |
| --- | --- | --- | --- | --- |
| Clients | 10 | 10.20.10.0/24 | The 25 workstations | Where staff open mail and browse: the most likely place for a first infection |
| Servers | 20 | 10.20.20.0/24 | The file server and internal services | Holds the client files and the payroll data |
| DMZ | 40 | 10.20.40.0/24 | The public web server | Reachable from the internet, so the most exposed machine |
| Management | 99 | 10.20.99.0/24 | The admin interfaces of switches and, later, the firewall and the log host | Who controls these devices controls every other zone |

The public zone is called a DMZ. You learn what makes it special in module 4.

The planned ranges skip 10.20.30.0/24 on purpose. That is the range of the old flat network, and it stays in use until you move the machines in module 3.

### Five rules for a zone design

1. **Start from what you protect, not from the devices.** A zone exists because of an asset or a risk.
2. **Put together what needs the same access and the same protection.** All workstations do the same job, so they share a zone.
3. **Give everything that is reachable from the internet its own zone.** It will be attacked first.
4. **Separate management.** The interfaces where you configure the network do not belong where the users are.
5. **Be able to explain every zone in one sentence.** A zone without a reason is a rule set nobody maintains.

### Who needs to talk to whom

A zone design is half the work. The other half is the list of traffic that has to cross the borders.

| From | To | Needed for |
| --- | --- | --- |
| Clients | Servers | Opening client files, the intranet |
| Internet | DMZ | Clients who upload invoices |
| DMZ | Servers | Handing over the uploaded invoices |
| Management | Every zone | Administration |
| Every zone | Management | Nothing |

Every row that is not in this table should not be possible. A workstation has no reason to open a connection to the web server's operating system, and the web server has no reason to start a connection to a workstation.

::: warning
**Watch out:** a VLAN separates, and that is all it does. After this module nothing can cross a border, including the traffic in the table. You open the crossings in module 3 and guard them in module 4.
:::

What this means for you: you can defend this design to Nadia with her own three fears. If a zone does not make one of them smaller, it does not belong in the design.

### Check yourself 2.4 – Designing zones

```text
Sectie: 2.4
Type: categoriseren
Vraag: In which zone of Halden & Co does the device belong?
Indeling:
  The laptop of an accountant → Clients
  The PC at the front desk → Clients
  The file server with payroll data → Servers
  The web server where clients upload invoices → DMZ
  The admin interface of a switch → Management
Feedback: Workstations share the Clients zone, the confidential data sits in Servers, what the internet can reach goes in the DMZ, and admin interfaces go in Management (see 2.4).

Sectie: 2.4
Type: meerkeuze
Vraag: Tom proposes to put the public web server in the Servers zone, "because it is a server". What is the strongest argument against that? Choose one answer.
Opties:
  A. It is reachable from the internet, so a compromise would put an attacker next to the client files
  B. Web servers need a different VLAN ID than file servers
  C. A zone can hold only one server
  D. The web server uses more bandwidth than the file server
Juist: A
Feedback A: Correct. The most exposed machine and the most valuable data should not share a zone (see 2.4).
Feedback B: A VLAN ID is a number you choose. It says nothing about the kind of device (see 2.4).
Feedback C: A zone holds every device with the same task and the same need for protection (see 2.4).
Feedback D: Zones are about access and risk, not about the amount of traffic (see 2.4).

Sectie: 2.4
Type: meerdere antwoorden
Vraag: Which traffic does the design of Halden & Co need? Choose all correct answers.
Opties:
  A. From Clients to Servers, to open client files
  B. From the internet to the DMZ, to upload invoices
  C. From the DMZ to Clients, to show the portal to staff
  D. From Clients to Management, to browse the switch
Juist: A, B
Feedback A: Correct. Staff work with the files on the file server (see 2.4).
Feedback B: Correct. That is what the portal is for (see 2.4).
Feedback C: The web server has no reason to start a connection to a workstation (see 2.4).
Feedback D: Nothing needs to reach Management from another zone. Administration starts from Management (see 2.4).
```

## 2.5 Building Halden's zones

You have a design on paper and two switches that still carry the test setup from [2.3](#2_3). Now you build the real thing, and you leave behind something Tom can read when you are gone.

### The port plan

Write down what every port is for before you type a command. This table is the **port plan**: the list of every switch port with its VLAN and what is connected to it.

| Switch | Port | Mode | VLAN | Connected to |
| --- | --- | --- | --- | --- |
| `halden-sw01` (office floor) | `port1` | Access | 10 | Workstation (Kali) |
| | `port2` | Access | 999 | Not in use |
| | `port3` | Access | 99 | Management |
| | `port4` | Trunk | 10, 20, 40, 99 | `halden-sw02` |
| `halden-sw02` (server room) | `port1` | Access | 20 | `halden-srv01` |
| | `port2` | Access | 40 | Public web server (later) |
| | `port3` | Access | 99 | Management |
| | `port4` | Trunk | 10, 20, 40, 99 | `halden-sw01` |

### Unused ports go to the parking VLAN

`port2` of `halden-sw01` is a free desk. Nobody uses it today. A free port in the Clients zone is an invitation: anyone who plugs in a laptop is on the network of the workstations.

A **parking VLAN** is a VLAN without devices that is not on any trunk. You put every unused port in it. A device on such a port has a link and reaches nothing. At Halden & Co the parking VLAN is 999.

The day someone starts at that desk, you move the port to VLAN 10 and update the port plan. That is one command, and it is a decision instead of a default.

### Configure both switches

On `halden-sw01`:

```bash
sudo ovs-vsctl set port port1 tag=10
sudo ovs-vsctl set port port2 tag=999
sudo ovs-vsctl set port port3 tag=99
sudo ovs-vsctl set port port4 trunks=10,20,40,99
```

On `halden-sw02`:

```bash
sudo ovs-vsctl set port port1 tag=20
sudo ovs-vsctl set port port2 tag=40
sudo ovs-vsctl set port port3 tag=99
sudo ovs-vsctl set port port4 trunks=10,20,40,99
```

### Verify against the plan

A configuration you did not check is a guess. On each switch, compare the output with your port plan, line by line:

```bash
sudo ovs-vsctl show
```

```output
d64098d6-936a-443c-b797-bc5dc5d6bd40
    Bridge br0
        Port port1
            tag: 10
            Interface port1
        Port port2
            tag: 999
            Interface port2
        Port port3
            tag: 99
            Interface port3
        Port port4
            trunks: [10, 20, 40, 99]
            Interface port4
        Port br0
            Interface br0
                type: internal
    ovs_version: "3.3.9"
```

Every port has either a `tag` or a `trunks` line. A port with neither is the forgotten port from [2.3](#2_3). VLAN 999 is on no trunk, so the parking VLAN stays inside each switch.

Then test the behaviour. A test has an expected result that you write down before you run it.

| Test | From Kali | Expected | Why |
| --- | --- | --- | --- |
| 1 | `ping -c 2 10.20.30.10` | 100% packet loss | Kali is in Clients, the server in Servers |
| 2 | Move `port1` of `halden-sw01` to VLAN 20, ping again | Replies | Same VLAN, and the trunk carries it |
| 3 | Move `port1` back to VLAN 10, ping again | 100% packet loss | The border is back |

Test 2 matters as much as test 1. A ping that fails proves little: a cable in the wrong network fails too. Only the pair "works in the same VLAN, fails across VLANs" shows that the VLANs are the cause.

### Save the state

1. Shut down all four machines.
2. Take a snapshot of each one with the name `m2-zones`.

Halden's staff could not work on this network: no workstation reaches the file server. That is the correct result for this module, and the first thing you fix in module 3.

What this means for you: the port plan and the test table are part of the work, not paperwork after it. They are what you hand to the client, and what you check your firewall against in module 4.

### Check yourself 2.5 – Building Halden's zones

```text
Sectie: 2.5
Type: meerkeuze
Vraag: You check a switch against the port plan. Which port needs your attention? Choose one answer.
Code:
  Port port1
      tag: 10
  Port port2
      tag: 10
  Port port3
  Port port4
      trunks: [10, 20, 40, 99]
Opties:
  A. `port3`, because it has no `tag` and no `trunks`
  B. `port4`, because it carries four VLANs
  C. `port2`, because it has the same VLAN as `port1`
  D. No port: the configuration is complete
Juist: A
Feedback A: Correct. In Open vSwitch a port without `tag` and without `trunks` carries every VLAN (see 2.5).
Feedback B: A trunk with a list of the VLANs it needs is what you want (see 2.5).
Feedback C: Several access ports can share a VLAN. All workstations are in VLAN 10 (see 2.5).
Feedback D: One port has no configuration, and that has a meaning in Open vSwitch (see 2.5).

Sectie: 2.5
Type: meerkeuze
Vraag: Kali is in VLAN 10 and the server in VLAN 20. Your ping fails, as expected. Why do you also test with both in the same VLAN? Choose one answer.
Opties:
  A. To show that the VLANs cause the failure, and not a broken cable or a wrong address
  B. To make the capture file larger
  C. Because a ping only works the second time
  D. To check that the server is switched on, which is all the test shows
Juist: A
Feedback A: Correct. A ping can fail for many reasons. The pair of tests isolates the VLAN as the cause (see 2.5).
Feedback B: The size of a capture is not a goal. The test has to prove something (see 2.5).
Feedback C: A ping does not need a second attempt when the network is configured correctly (see 2.5).
Feedback D: It shows more than that: the whole path through both switches and the trunk works (see 2.5).

Sectie: 2.5
Type: meerkeuze
Vraag: A visitor plugs a laptop into a free desk port that is in the parking VLAN. What can the laptop reach? Choose one answer.
Opties:
  A. Nothing
  B. The workstations, because the desk is on the office floor
  C. Only the internet
  D. Every VLAN, because the port is not in use
Juist: A
Feedback A: Correct. The parking VLAN has no other devices and is on no trunk (see 2.5).
Feedback B: The place of the desk does not matter. The VLAN of the port decides, and that is not the VLAN of the workstations (see 2.5).
Feedback C: The parking VLAN leads nowhere, also not to the internet (see 2.5).
Feedback D: That is what happens with a port without configuration. A port in the parking VLAN has a `tag`, so it belongs to one VLAN (see 2.5).

Sectie: 2.5
Type: ordenen
Vraag: Put the steps of building a segmented network in the order of this module.
Volgorde:
  Design the zones from the assets
  Write the port plan
  Configure the switches
  Compare the configuration with the port plan
  Test the behaviour
Feedback: You decide first and type afterwards. Checking has two parts: the configuration against the plan, and the behaviour against what you expect (see 2.5).
```

## Summary

New terms in this module:

- **flat network**: a network without internal borders;
- **broadcast domain**: the group of devices that receive each other's broadcasts;
- **lateral movement**: an attacker moves from the first machine to the next ones;
- **segmentation**: splitting a network into parts and controlling the traffic between them;
- **zone**: a group of devices with the same task and the same need for protection;
- **switch**: connects the devices of one network and delivers each frame to the right port;
- **VLAN** and **VLAN ID**: a separate network inside a switch, and its number;
- **access port**: a port that belongs to one VLAN;
- **VLAN tag** and **802.1Q**: four extra bytes in a frame that hold the VLAN ID, and the standard that describes them;
- **trunk port** and **trunk**: a port that carries tagged frames of several VLANs, and the link between two such ports;
- **Open vSwitch**: switch software for Linux;
- **port plan**: the list of every switch port with its VLAN and what is connected to it;
- **parking VLAN**: a VLAN without devices and on no trunk, for unused ports.

The rules to remember:

- Segmentation does not stop the first infection. It limits how far it spreads.
- Ports in the same VLAN can exchange frames. Ports in different VLANs cannot, whatever their IP addresses are.
- A trunk carries only the VLANs on its list. Put only the VLANs on it that both sides need.
- Every zone has a reason you can say in one sentence.
- Unused ports go to the parking VLAN.
- Write the port plan first, then configure, then check both the configuration and the behaviour.

## Common mistakes

**Leaving a port without configuration.** `port3` has no `tag` and no `trunks`, so in Open vSwitch it carries every VLAN. Give every port a `tag`. Ports nobody uses go to the parking VLAN, 999.

**Configuring one end of a trunk.** `halden-sw01` has `trunks=10,20` and `halden-sw02` was forgotten. Configure both ends with the same list, and check both with `ovs-vsctl show`.

**Expecting a VLAN to follow the IP range.** Two hosts in 10.20.30.0/24 cannot ping each other, and you look for a wrong address. They are in different VLANs. Check the port before you check the address.

**Typing on the wrong switch.** Both switches come from the same file and look the same. Give each its own hostname and read the prompt before you press Enter.

**A typing mistake in the name of an internal network.** `sw01-port1` on Kali and `sw01-prt1` on the switch are two cables that go nowhere. Compare the names letter by letter, as in [0.4](./module-0.md#0_4).

**Testing only that traffic is blocked.** A ping that fails proves nothing on its own. Show that it works inside the VLAN and fails across VLANs.

**Thinking the work is done.** The zones are separated, and nothing decides yet which traffic may cross. Without modules 3 and 4, segmentation is a network where nobody can work.

::: oefening
[Exercises module 2](./module-2-exercises.md)
:::
