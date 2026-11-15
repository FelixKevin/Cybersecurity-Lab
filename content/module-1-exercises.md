---
module: 1
order: 2
tocMode: page
tocLabel: Exercises
published: false
---

# Exercises module 1

You make every exercise in your own lab, with Kali and `halden-srv01` running. The core exercises E1 to @1 and the case belong together: who can do them has reached the learning objectives of module 1.

## E1 Where is it? {#e1 .easy}

A colleague sends you a screenshot of Wireshark and asks where to find things. You answer from your own capture.

Open `m1-ping.pcapng` from [1.1](./module-1.md#1_1) and select the first echo request. Find the four values below.

Answer with a table of four rows: the value, and the pane and layer where you found it.

- the MAC address of Kali;
- the IP address of the destination;
- the TTL of the packet;
- the size of the whole frame in bytes.

## E2 Say it in a filter {#e2 .easy}

A filter is a sentence in a small language. Translate five sentences.

Write one display filter for each description. Test every filter in Wireshark: the bar has to turn green.

Answer with a list of five filters.

1. All packets to or from 10.20.30.10.
2. All DNS traffic.
3. All packets that 10.20.30.50 sent.
4. All traffic on TCP port 22.
5. All packets that are ARP or ICMP.

## M1 The first minute {#m1}

The bakery Brood & Zo has one laptop behind the counter. The owner wants to know what it does on the network when nobody touches it. You answer the same question for your Kali machine.

Start a capture on `eth0` and do nothing in Kali for 60 seconds. Stop the capture and choose **Statistics > Protocol Hierarchy**.

Answer with a table of at most five rows: the protocol, the number of packets, and one sentence on why Kali sends it. Write "unknown" when you cannot explain a protocol after five minutes of searching.

## M2 Two languages {#m2}

The accountant Van Damme captures on a laptop with Wireshark and on a server with tcpdump. The same question needs a different filter in each tool.

Complete the table. Each row describes the same traffic in both kinds of filter.

Answer with the completed table.

| Traffic | Display filter | Capture filter |
| --- | --- | --- |
| To or from 10.20.30.10 | | `host 10.20.30.10` |
| On port 443 | `tcp.port == 443` | |
| DNS lookups | `dns` | |
| Everything except SSH | | `not port 22` |
| From or to the lab network | | `net 10.20.30.0/24` |

## H1 Who does Kali talk to? {#h1 .hard}

The dentists of Tandartsen Noord installed new software. Nobody knows which servers it contacts. Before you analyse their network, you practise the method on your own machine.

Start a capture on `eth0` and run the command below. Stop the capture when the command is done.

```bash
sudo apt update
```

Work out which servers Kali contacted, and decide whether someone on the same network could read what was exchanged.

Answer with:

- a table with one row per server: name, IP address, port, and whether the content is plaintext or encrypted;
- three sentences with your conclusion, each with the packet number that supports it.

Hint: start with **Statistics > Conversations**, and use the DNS responses to turn addresses into names.

## @1 Wireshark without a window {#home1 .home}

Wireshark comes with a command-line version, `tshark`. It reads the same display filters.

Read the manual page with `man tshark`. Find out how you read a capture file and apply a display filter in one command. Run that command on `m1-ping.pcapng` so that it shows only the echo replies.

Answer with the command and its output.

## C1 Halden & Co: the baseline {#c1 .case}

Before you change anything on Halden's network, you record how it behaves today. Later modules compare against this baseline.

Starting point: your lab as you left it after module 0. Restore the snapshot `m0-clean` on both machines if you are not sure of its state.

Capture one exchange for each of the five protocols from [1.4](./module-1.md#1_4) and [1.5](./module-1.md#1_5): ARP, DHCP, DNS, the TCP handshake and the TLS handshake. Save each one in its own file, named `m1-arp.pcapng`, `m1-dhcp.pcapng` and so on.

Answer with a table of five rows: the file name, the interface, the display filter that isolates the exchange, and the packet numbers of the exchange.

## C2 Halden & Co: what an eavesdropper learns {#c2 .case}

Nadia wants to know what someone could learn about a workstation without touching it. You answer with the captures from [C1 Halden & Co: the baseline](#c1).

For each of the five captures, write down what the exchange gives away to someone who can read it.

Answer with a table of five rows: the protocol, what it reveals, and the packet number and field that shows it.

## C3 Halden & Co: a note for Tom {#c3 .case}

Tom said that the intranet does not need encryption, because it is internal. You have the capture from [1.3](./module-1.md#1_3) that shows what that means.

Write a note to Tom of at most 150 words. He is not a network specialist, and he has ten minutes.

Answer with a note that contains:

- what you tested, in one sentence;
- what you found, with the name of the capture file and the stream number;
- what it means for Halden & Co, linked to one of the three fears from [0.1](./module-0.md#0_1);
- one recommendation.

## Extra exercises {#extra}

These exercises are optional. They repeat the module at different levels.

## X1 Count the pings (Easy) {#x1}

Start a capture on `eth1` and run `ping -c 5 10.20.30.10`. Apply a display filter that shows only the echo requests.

Answer with the filter and the number of packets it shows.

## X2 One capture, three filters (Medium) {#x2}

Start a capture on `eth1` with the capture filter `host 10.20.30.10`. Ping the server, request the intranet page with `curl` and log in over SSH. Stop the capture.

Write three display filters that each show one of the three activities and nothing else.

Answer with the three filters and the number of packets each one shows.

## X3 The handshake from both sides (Hard) {#x3}

Capture the same `curl http://10.20.30.10/` twice at the same moment: with Wireshark on Kali and with tcpdump on `halden-srv01`. Copy the server capture to Kali and compare the three packets of the handshake in both files.

Answer with a table that lists, for each of the three packets, what is identical in both captures and what differs. Explain each difference in one sentence.
