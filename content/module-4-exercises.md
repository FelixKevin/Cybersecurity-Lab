---
module: 4
order: 2
tocMode: page
tocLabel: Exercises
published: false
---

# Exercises module 4

You make the exercises E1 to @1 on paper or in your own lab. The case continues the work on Halden & Co. Who can do the core exercises and the case has reached the learning objectives of module 4.

## E1 Which rule decides? {#e1 .easy}

The bookshop Het Leeshuis has a firewall with these rules on the interface of its shop network, 192.168.10.0/24.

```output
1. Block  TCP      Shop net -> 192.168.20.5   port 22
2. Pass   TCP      Shop net -> 192.168.20.5   any port
3. Pass   TCP/UDP  Shop net -> This Firewall  port 53
4. Pass   TCP      Shop net -> any            port 443
```

Work out for each packet from a till in the shop network which rule decides, and what happens.

Answer with a table of five rows: the packet, the rule number or "none", and pass or block.

1. TCP to 192.168.20.5, port 22
2. TCP to 192.168.20.5, port 3306
3. UDP to the firewall, port 53
4. TCP to 93.184.215.14, port 80
5. TCP to 192.168.30.9, port 443

## E2 Block or reject? {#e2 .easy}

The same bookshop asks which action to use for traffic it does not allow.

Choose `Block` or `Reject` for each situation.

Answer with a table of three rows: the situation, the action and one sentence with the reason.

1. Connections from the internet to ports the shop does not offer.
2. A till that tries to reach a printer in another zone, which staff sometimes do by mistake.
3. A public web server that tries to open a connection to the office network.

## M1 Who starts? {#m1}

The physiotherapy practice Kine Plus has three flows of data between zones. For each one, two designs are possible.

Decide for each flow which machine should start the connection, so that the less trusted zone never starts a connection into the more trusted one.

Answer with a table of three rows: the flow, the machine that starts, and the interface on which the pass rule goes.

1. Appointments that patients book on the public website have to reach the agenda server in the Servers zone.
2. The backup server in the Management zone has to store a copy of the agenda server.
3. A camera in the Cameras zone has to deliver its recordings to a recorder in the Servers zone.

## M2 Three mistakes {#m2}

The administrator of the garage Autobedrijf Peeters wrote this rule set and asks why it does not do what he wants. Workstations are in the Office zone. He wants them to reach the stock application on 10.5.20.8, port 8080, and websites on the internet. Nothing else should be possible.

```output
Interface Office:
1. Pass   TCP      Office net -> any         port 80, 443
2. Block  TCP      Office net -> 10.5.20.8   port 22

Interface Servers:
1. Pass   TCP      Office net -> 10.5.20.8   port 8080
2. Pass   any      Servers net -> Office net any port
```

Find the three mistakes.

Answer with a table of three rows: the rule, what is wrong with it, and the rule you would write instead. Then give the corrected rule set.

## H1 A rule set from a zone design {#h1 .hard}

The architecture firm Atelier Dubois has four zones behind one firewall.

| Zone | Range | What is in it |
| --- | --- | --- |
| Office | 10.30.1.0/24 | Twelve workstations |
| Storage | 10.30.2.0/24 | A NAS on 10.30.2.10 with all drawings, reachable on port 445 |
| Guests | 10.30.3.0/24 | Phones and laptops of visitors |
| Cameras | 10.30.4.0/24 | Four cameras and a recorder on 10.30.4.10 |

The partners want this: staff work on the NAS and browse the web. Guests get the internet and nothing else. The office manager watches the camera images on the web page of the recorder, port 443. The cameras themselves need nothing outside their zone.

Answer with:

- the rule set per interface, as tables like the ones in [4.3](./module-4.md#4_3);
- for each zone, one test that should work and one that should fail;
- the rule you find most doubtful, and what you would ask the partners about it.

## @1 What is knocking? {#home1 .home}

Your firewall blocks traffic all the time that you never thought about.

Start your lab, leave Kali alone for ten minutes and open **Firewall > Log Files > Live View** with a filter on the action `block`.

Answer with a table of at most five rows, one per kind of blocked traffic: the interface, the source, the destination and port, and your best explanation of what it is. Mark each row with "leave blocked" or "ask Tom".

## C1 Halden & Co: the rule register {#c1 .case}

The rule set of [4.3](./module-4.md#4_3) and [4.4](./module-4.md#4_4) works. Tom cannot maintain it yet.

Starting point: your lab with the snapshot `m4-firewall` from [4.5](./module-4.md#4_5).

Complete the rule register for every rule on the Clients, Servers, DMZ and WAN interfaces.

Answer with a table that has one row per rule, with the columns of [4.5](./module-4.md#4_5): interface, number, rule, reason, asked by and test. The register is complete when it has as many rows as the firewall has rules on those interfaces.

## C2 Halden & Co: the test report {#c2 .case}

A register says what a rule is for. A test report shows that the firewall does it.

Run every test from your register. Add at least four tests that have to fail: one per interface.

Answer with a table of at least fifteen rows: the test, the machine you ran it from, the expected result, the result you saw and the evidence. For a test that has to fail, the evidence is the line from the live log that shows the block.

## C3 Halden & Co: a request from Tom {#c3 .case}

Tom writes: "The partners want to open client files from home. Can you forward port 445 to the file server? It is just one port."

Answer with a reply to Tom of at most 150 words that contains:

- your decision;
- the reason, linked to one of Nadia's fears from [0.1](./module-0.md#0_1) and to the design of [4.4](./module-4.md#4_4);
- what you propose to look at instead, in one sentence. You do not have to build it.

## Extra exercises {#extra}

These exercises are optional. They repeat the module at different levels.

## X1 One rule, six fields (Easy) {#x1}

The printer of Halden & Co gets the address 10.20.10.200 in the Clients zone. The file server has to send scan jobs to it on port 9100.

Answer with the rule as a table of field and value: interface, action, protocol, source, destination and port.

## X2 Ping, yes or no (Medium) {#x2}

After [4.3](./module-4.md#4_3) the workstations cannot ping the internet. Tom says that he uses ping every day to check whether the connection works.

Answer with your decision, the rule if you allow it, and the row for the rule register.

## X3 Tighten Management (Hard) {#x3}

The Management interface has one rule that allows everything from the Management zone. That zone holds the admin interfaces of the switches and the firewall, and the laptop of the administrator.

Replace the rule by a rule set under default deny. Make sure you can still reach the web interface of the firewall before you apply it.

Answer with:

- the new rule set of the Management interface;
- the test that shows you did not lock yourself out;
- one sentence on what you would do if you had locked yourself out.
