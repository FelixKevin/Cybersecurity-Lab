---
module: 1
order: 1
moduleTitle: "1: Reading the wire"
published: false
---

# Module 1: Reading the wire

Everything that happens on a network happens in packets, and packets can be recorded and read. In this module you record the traffic of your lab and learn to find what matters in it. You need that skill in every later module: to check a firewall rule, to see an attack and to prove what happened.

### Learning objectives

After this module you can:

- capture traffic on a chosen interface with Wireshark and save it as evidence;
- write display filters and capture filters that isolate one host, port or protocol;
- reconstruct a conversation between two hosts from a capture;
- explain the exchanges of ARP, DHCP, DNS, TCP and TLS from the packets;
- capture traffic on a server without a graphical interface with tcpdump.

## 1.1 Capturing traffic

In module 0 you pinged `halden-srv01` and got three replies. That tells you the server is alive. It does not tell you what your laptop sent, what came back, or what else happened on the wire. To know that, you have to look at the traffic itself.

A **packet** is one unit of data that travels over a network. A **packet capture** is a recording of the packets that pass one network interface. The tool that records and decodes them is a **packet analyser**. The one you use most in this course is **Wireshark**.

### Your first capture

Start Kali and `halden-srv01`. Work in Kali.

1. Start Wireshark from the application menu, or type `wireshark &` in a terminal. You now see a list of interfaces.
2. Double-click `eth1`, the interface on the lab network. The capture starts, and the list stays almost empty: nothing is happening on `halden-lab`.
3. In a terminal, send three pings to the server:

```bash
ping -c 3 10.20.30.10
```

4. In Wireshark, click the red square to stop the capture. You now see eight packets, or six if Kali already knew where the server was.

[NIET GETEST] Starting Wireshark as the user `kali` and capturing without extra permissions on the current Kali image.

### What you are looking at

The window has three panes.

| Pane | Shows | Use it to |
| --- | --- | --- |
| Packet list | One line per packet: number, time, source, destination, protocol, info | Find the packet you want |
| Packet details | The selected packet, decoded layer by layer | Read what a field means |
| Packet bytes | The same packet as raw bytes | See what was really sent |

The packet list of your capture looks like this:

| No. | Source | Destination | Protocol | Info |
| --- | --- | --- | --- | --- |
| 1 | PCSSystemtec_… | Broadcast | ARP | Who has 10.20.30.10? Tell 10.20.30.50 |
| 2 | PCSSystemtec_… | PCSSystemtec_… | ARP | 10.20.30.10 is at 08:00:27:… |
| 3 | 10.20.30.50 | 10.20.30.10 | ICMP | Echo (ping) request |
| 4 | 10.20.30.10 | 10.20.30.50 | ICMP | Echo (ping) reply |

Packets 5 to 8 repeat the last two lines. You asked for three pings and got eight packets: before Kali could send anything, it first had to find the server. You read that exchange in [1.4](#1_4).

Select packet 3 and look at the packet details. You see four lines that you can open: Frame, Ethernet II, Internet Protocol Version 4 and Internet Control Message Protocol. Those are the layers of the TCP/IP model from Computer Networks, in the order they are wrapped around the data.

### Save it

Choose **File > Save As** and save the capture as `m1-ping.pcapng`. A **capture file** holds the recorded packets, so that you or someone else can open them again later.

Name every capture file like your snapshots: module number first, then what it shows. Copy the files you need for your portfolio to the host (see [0.5](./module-0.md#0_5)).

::: warning
**Watch out:** a capture contains everything that passed, including passwords and personal data. Capture only on your own lab machines. Recording traffic of other people on a shared network is out of scope (see [0.2](./module-0.md#0_2)).
:::

What this means for you: from now on "it works" is not evidence. A capture file that shows the request and the reply is.

### Check yourself 1.1 – Capturing traffic

```text
Sectie: 1.1
Type: koppelen
Vraag: Match each pane of Wireshark with what it shows.
Paren:
  Packet list → One line per captured packet
  Packet details → The selected packet, decoded layer by layer
  Packet bytes → The selected packet as raw bytes
Feedback: The list is for finding a packet, the details are for reading it, and the bytes show what was really sent (see 1.1).

Sectie: 1.1
Type: meerkeuze
Vraag: You start a capture on `eth0` and ping 10.20.30.10. The ping gets replies, but the capture shows no ICMP packets. What is the most likely cause? Choose one answer.
Opties:
  A. The lab network is on another interface
  B. Wireshark cannot decode ICMP
  C. The server blocks packet captures
  D. Ping packets are too small to capture
Juist: A
Feedback A: Correct. Traffic to 10.20.30.10 leaves through `eth1`. A capture shows only the interface you chose (see 1.1).
Feedback B: Wireshark decodes ICMP, as you saw in your first capture (see 1.1).
Feedback C: A capture happens on your own interface. The other side cannot prevent it (see 1.1).
Feedback D: Every packet that passes the interface is captured, whatever its size (see 1.1).

Sectie: 1.1
Type: ordenen
Vraag: Put the layers in the order Wireshark shows them in the packet details of a ping request, from top to bottom.
Volgorde:
  Frame
  Ethernet II
  Internet Protocol Version 4
  Internet Control Message Protocol
Feedback: Wireshark starts with the frame as it was captured and then works inwards: Ethernet, IP and then ICMP (see 1.1).
```

## 1.2 Filters

Your first capture had eight packets. Start a capture on `eth0`, open a browser in Kali and load one news site. Within seconds you have a few thousand packets. Nobody finds one ping in that by scrolling.

A **display filter** hides every captured packet that does not match a condition. The packets are still in the capture; you only stop looking at them.

### Display filters

Type a filter in the bar above the packet list and press Enter. The bar turns green when the filter is valid and red when it is not.

| You want to see | Display filter |
| --- | --- |
| Only ping traffic | `icmp` |
| Everything to or from the server | `ip.addr == 10.20.30.10` |
| Only packets sent by Kali | `ip.src == 10.20.30.50` |
| Web traffic without encryption | `tcp.port == 80` |
| Pings sent by Kali | `ip.src == 10.20.30.50 && icmp` |
| Name lookups and address lookups | `dns \|\| arp` |
| Everything except the server | `!(ip.addr == 10.20.30.10)` |

The general form is:

```text
<field> <operator> <value>
```

A protocol name on its own, such as `icmp`, is also a filter: it matches every packet that contains that protocol. You combine conditions with `&&` (and), `||` (or) and `!` (not).

You do not have to remember field names. Select a packet, right-click a field in the packet details and choose **Apply as Filter > Selected**. Wireshark writes the filter for you.

### Capture filters

A display filter works after the capture. A **capture filter** decides which packets are recorded at all. You set it before you start, in the field under the interface list.

| You want to record | Capture filter |
| --- | --- |
| Only traffic to or from the server | `host 10.20.30.10` |
| Only web traffic without encryption | `port 80` |
| Everything on the lab network | `net 10.20.30.0/24` |
| Only name lookups | `udp port 53` |
| Everything except SSH | `not port 22` |

The general form is:

```text
<qualifier> <value>
```

Qualifiers are words such as `host`, `net` and `port`. You combine them with `and`, `or` and `not`.

::: warning
**Watch out:** the two kinds of filter have different syntax. `ip.addr == 10.20.30.10` is a display filter, `host 10.20.30.10` is a capture filter. Each one is an error in the other field.
:::

### Which one do you use?

Use a display filter by default. You can change it as often as you like, and nothing is lost.

Use a capture filter when the capture would otherwise become too large, or when you may only record certain traffic. What a capture filter drops is gone: there is no way to look at it afterwards.

What this means for you: an empty packet list does not mean there was no traffic. Check first whether a filter is still active.

### Check yourself 1.2 – Filters

```text
Sectie: 1.2
Type: categoriseren
Vraag: Is it a display filter or a capture filter?
Indeling:
  ip.addr == 10.20.30.10 → Display filter
  tcp.port == 80 → Display filter
  dns || arp → Display filter
  host 10.20.30.10 → Capture filter
  not port 22 → Capture filter
  net 10.20.30.0/24 → Capture filter
Feedback: Display filters use field names with an operator, such as `ip.addr ==`. Capture filters use qualifiers such as `host`, `net` and `port` (see 1.2).

Sectie: 1.2
Type: meerkeuze
Vraag: Which display filter shows only the packets that 10.20.30.50 sent to port 80? Choose one answer.
Opties:
  A. `ip.src == 10.20.30.50 && tcp.dstport == 80`
  B. `ip.addr == 10.20.30.50 || tcp.port == 80`
  C. `host 10.20.30.50 and port 80`
  D. `ip.src == 10.20.30.50 && tcp.port != 80`
Juist: A
Feedback A: Correct. Both conditions must be true: the source is Kali and the destination port is 80 (see 1.2).
Feedback B: `||` means or. This also shows every packet of Kali on other ports, and port 80 traffic of other hosts (see 1.2).
Feedback C: This is capture filter syntax. In the display filter bar it turns red (see 1.2).
Feedback D: `!=` selects the packets that do **not** use port 80 (see 1.2).

Sectie: 1.2
Type: meerkeuze
Vraag: You captured for ten minutes with the capture filter `port 80`. Afterwards you want to check the DNS lookups from those ten minutes. What can you do? Choose one answer.
Opties:
  A. Nothing: the DNS packets were never recorded
  B. Clear the capture filter and the DNS packets appear
  C. Apply the display filter `dns`
  D. Open the capture file again without a filter
Juist: A
Feedback A: Correct. A capture filter decides what is recorded. DNS uses port 53, so those packets are not in the file (see 1.2).
Feedback B: Changing the capture filter only affects the next capture (see 1.2).
Feedback C: A display filter can only show packets that are in the capture. These are not (see 1.2).
Feedback D: The file contains only what was recorded, however you open it (see 1.2).
```

## 1.3 Following a conversation

Tom mentions that the staff use an intranet page on `halden-srv01`. He adds that it is internal, so it does not need encryption. You can test that claim in two minutes.

A filter shows you packets. A request and its answer are spread over many packets, mixed with other traffic. You want to read them as one exchange. Wireshark calls all packets of one TCP connection a **stream**.

### Read the intranet page off the wire

1. Start a capture on `eth1`.
2. In a terminal in Kali, request the page:

```bash
curl http://10.20.30.10/
```

3. Stop the capture. You see about ten packets with the protocols TCP and HTTP.
4. Right-click any of those packets and choose **Follow > TCP Stream**.

You now see a window with the whole exchange as text. The request of Kali is in red, the answer of the server in blue:

```output
GET / HTTP/1.1
Host: 10.20.30.10
User-Agent: curl/8.15.0
Accept: */*

HTTP/1.1 200 OK
Content-Type: text/html

<html>
<head><title>Halden & Co intranet</title></head>
...
```

[NIET GETEST] The exact request and response depend on the Kali image and on the web server in `halden-srv01.ova`.

Data that anyone who captures it can read is **plaintext**. The intranet page is plaintext, and so is everything a colleague types into a form on it.

When you close the window, the filter bar contains `tcp.stream eq 0`. Wireshark numbers the streams in a capture from 0. Change the number to walk through the other conversations.

### Who talked to whom?

For a large capture you first want an overview. Choose **Statistics > Conversations**. You get one line per pair of hosts, with the number of packets and bytes in each direction.

This is the fastest way to answer the first question about any capture: which machines are in it, and which pair exchanged the most data?

::: info
You read this page because you captured on your own interface, where your own request passed. Reading the traffic of other machines takes one more step, which you take in module 6. On a flat network like Halden's that step is small.
:::

What this means for you: "it is internal" is not a security measure. You now have a capture file that shows Tom why.

### Check yourself 1.3 – Following a conversation

```text
Sectie: 1.3
Type: meerkeuze
Vraag: After **Follow > TCP Stream** the filter bar shows `tcp.stream eq 3`. What does the packet list show now? Choose one answer.
Opties:
  A. All packets of one TCP connection
  B. The third packet of the capture
  C. All packets on port 3
  D. Three TCP connections
Juist: A
Feedback A: Correct. Wireshark numbers every TCP connection in the capture. The filter shows the packets of the connection with number 3 (see 1.3).
Feedback B: The number is the number of the stream, not of a packet (see 1.3).
Feedback C: A port filter looks like `tcp.port == 3`. The stream number has nothing to do with ports (see 1.3).
Feedback D: The filter shows one connection: the one with number 3. Numbering starts at 0, so it is the fourth (see 1.3).

Sectie: 1.3
Type: meerdere antwoorden
Vraag: A colleague at Halden & Co logs in on a page that uses HTTP without encryption. Someone captures that traffic. What can that person read? Choose all correct answers.
Opties:
  A. The address of the page
  B. The username and password
  C. The content of the page the server sends back
  D. Files on the hard disk of the colleague
Juist: A, B, C
Feedback A: Correct. The request line and the Host header are plaintext (see 1.3).
Feedback B: Correct. Form data travels in the request, as plaintext (see 1.3).
Feedback C: Correct. The response is plaintext too (see 1.3).
Feedback D: A capture shows only what travels over the network. Files that are never sent are not in it (see 1.3).
```

## 1.4 ARP, DHCP and DNS up close

A workstation at Halden & Co that starts in the morning cannot send anything useful yet. It first needs an address, the way to its neighbours and the address behind a name. Three protocols arrange that. You know them from Computer Networks as diagrams.

Here you read each one in a capture, and you note what it gives away. You do the same three things every time: start a capture, run the command that triggers the protocol, and apply the display filter.

### ARP: who has this address?

Capture on `eth1`. Make Kali forget where the server is and ping it once:

```bash
sudo ip neigh flush dev eth1
ping -c 1 10.20.30.10
```

Display filter: `arp`. You see two packets. Kali asks the whole network "Who has 10.20.30.10?", and the server answers with its MAC address.

Open the first packet. The destination is Broadcast: every machine on the network receives the question. Nothing in the answer proves that it comes from the real server. You use that weakness in module 6.

### DHCP: can I have an address?

Capture on `eth0`. Disconnect and reconnect that interface:

```bash
sudo nmcli device disconnect eth0
sudo nmcli device connect eth0
```

Display filter: `dhcp`. You see up to four packets: Discover, Offer, Request and ACK. Kali asks whether anyone hands out addresses, a server offers one, Kali requests it and the server confirms.

Do you see only Request and ACK? Then Kali remembered its previous address and skipped the first two steps.

Open the ACK and look at the options. Next to the address, the server also tells Kali which router and which DNS server to use. Whoever answers DHCP requests decides where a workstation sends its traffic.

### DNS: what is the address of this name?

Capture on `eth0` and look up a name:

```bash
dig example.com
```

Display filter: `dns`. You see a query and a response. Open the query: the name `example.com` is in it as plaintext.

A list of the DNS queries of a workstation is close to a list of everything its user did that day.

[NIET GETEST] All commands in 1.4 and the number of packets they produce, on the current Kali image in VirtualBox.

What this means for you: none of these three protocols checks who is answering. In module 6 you abuse that, and in module 9 you write rules that notice the abuse.

### Check yourself 1.4 – ARP, DHCP and DNS

```text
Sectie: 1.4
Type: ordenen
Vraag: Put the packets in the order a workstation sends or receives them when it requests an address with DHCP.
Volgorde:
  Discover
  Offer
  Request
  ACK
Feedback: The workstation asks who hands out addresses, a server offers one, the workstation requests it and the server confirms (see 1.4).

Sectie: 1.4
Type: koppelen
Vraag: Match each protocol with the question it answers.
Paren:
  ARP → Which MAC address belongs to this IP address?
  DHCP → Which IP address can I use?
  DNS → Which IP address belongs to this name?
Feedback: ARP links an IP address to a MAC address, DHCP hands out addresses and DNS translates names (see 1.4).

Sectie: 1.4
Type: meerkeuze
Vraag: Kali sends the ARP request "Who has 10.20.30.10?". Which machines on `halden-lab` receive that packet? Choose one answer.
Opties:
  A. Every machine on the network
  B. Only the machine with the address 10.20.30.10
  C. Only the router
  D. No machine, until the address is known
Juist: A
Feedback A: Correct. The destination of an ARP request is Broadcast, so every machine on the network receives it (see 1.4).
Feedback B: Kali does not know yet where that machine is. That is why it has to ask everyone (see 1.4).
Feedback C: ARP works inside one network and does not need a router (see 1.4).
Feedback D: The request is the way to find out the address, so it has to reach the other machines first (see 1.4).
```

## 1.5 TCP and TLS up close

The workstation now has an address and knows where the server is. To exchange data, it opens a connection, and for most websites it encrypts that connection. You read both steps in a capture, as in [1.4](#1_4).

### TCP: the three-way handshake

Capture on `eth1` and request the intranet page again:

```bash
curl http://10.20.30.10/
```

Display filter: `tcp.flags.syn == 1`. You see two packets, marked `[SYN]` and `[SYN, ACK]`. Clear the filter and look at the packet right after them: an `[ACK]` from Kali.

Those three packets are the **three-way handshake**, the exchange that opens every TCP connection. Kali proposes a connection, the server accepts, Kali confirms. Only then does the HTTP request follow.

A server that answers a SYN with a SYN-ACK tells you that the port is open. Port scanners are built on that one fact, as you see in module 5.

### TLS: the same page, encrypted

Capture on `eth0` and request a page over HTTPS:

```bash
curl -s -o /dev/null https://example.com
```

Display filter: `tls.handshake`. The first packet is the Client Hello. Follow the TCP stream, as in [1.3](#1_3). This time the window shows unreadable data: the content is encrypted.

Now open the Client Hello in the packet details and look for the extension `server_name`. The name `example.com` is there, as plaintext.

Encryption hides what you send, not who you talk to. The IP addresses, the DNS query and the server name stay visible.

[NIET GETEST] All commands in 1.5 and the number of packets they produce, on the current Kali image in VirtualBox.

What this means for you: when you look at encrypted traffic, you can still say who talked to whom, when and how much. For a defender that is often enough to spot something wrong.

### Check yourself 1.5 – TCP and TLS

```text
Sectie: 1.5
Type: meerdere antwoorden
Vraag: A workstation opens `https://portal.example.com`. Someone captures the traffic. What can that person read? Choose all correct answers.
Opties:
  A. The IP address of the server
  B. The name `portal.example.com`
  C. The password the user types on the page
  D. The content of the page
Juist: A, B
Feedback A: Correct. IP addresses are never encrypted by TLS (see 1.5).
Feedback B: Correct. The name is visible in the DNS query and in the `server_name` extension of the Client Hello (see 1.5).
Feedback C: The password travels inside the encrypted connection (see 1.5).
Feedback D: The page is sent inside the encrypted connection (see 1.5).

Sectie: 1.5
Type: meerkeuze
Vraag: You send a SYN to port 22 of a server and get a SYN-ACK back. What do you know? Choose one answer.
Opties:
  A. Something is listening on port 22
  B. Your login was accepted
  C. The server runs Linux
  D. Port 22 is closed
Juist: A
Feedback A: Correct. A SYN-ACK is the second step of the handshake: the server accepts connections on that port (see 1.5).
Feedback B: The handshake comes before any login. No credentials were sent yet (see 1.5).
Feedback C: Every operating system answers this way. The handshake alone does not tell you which one it is (see 1.5).
Feedback D: A closed port does not answer with a SYN-ACK (see 1.5).
```

## 1.6 tcpdump on a server

So far you captured on Kali, with a window and a mouse. The interesting traffic at Halden & Co passes the file server, and that machine has no screen and no desktop. You need a tool that works in a terminal.

**tcpdump** is a packet analyser for the command line. It captures with the same capture filters you learned in [1.2](#1_2), and it writes files that Wireshark opens.

### Log in on the server

Tom has created an account for you on `halden-srv01`. Connect from a terminal in Kali:

```bash
ssh consultant@10.20.30.10
```

The first time, SSH asks whether you trust this server. Type `yes`. Then enter the password. You now see a prompt that ends in `halden-srv01`.

[CONTROLEREN] The account `consultant`, its password and its sudo rights have to exist in `halden-srv01.ova`.

### Capture in the terminal

First find the name of the interface:

```bash
sudo tcpdump -D
```

```output
1.enp0s3 [Up, Running, Connected]
2.any (Pseudo-device that captures on all interfaces) [Up, Running]
3.lo [Up, Running, Loopback]
```

Now capture five packets on that interface:

```bash
sudo tcpdump -i enp0s3 -n -c 5
```

Five lines appear at once, and all of them are SSH. You are watching your own session: every line tcpdump prints is sent to you over SSH, which is new traffic to print.

The fix is a capture filter. Leave out your own SSH traffic:

```bash
sudo tcpdump -i enp0s3 -n -c 5 not port 22
```

Now tcpdump waits. Open a second terminal in Kali and run `curl http://10.20.30.10/`. The first terminal shows:

```output
14:02:11.341802 IP 10.20.30.50.49288 > 10.20.30.10.80: Flags [S], seq 1672673210, win 64240, options [mss 1460,sackOK,TS val 3136385979 ecr 0,nop,wscale 7], length 0
14:02:11.341821 IP 10.20.30.10.80 > 10.20.30.50.49288: Flags [S.], seq 553797594, ack 1672673211, win 65160, options [mss 1460,sackOK,TS val 1806727562 ecr 3136385979,nop,wscale 7], length 0
14:02:11.341838 IP 10.20.30.50.49288 > 10.20.30.10.80: Flags [.], ack 1, win 502, options [nop,nop,TS val 3136385979 ecr 1806727562], length 0
14:02:11.341906 IP 10.20.30.50.49288 > 10.20.30.10.80: Flags [P.], seq 1:78, ack 1, win 502, options [nop,nop,TS val 3136385979 ecr 1806727562], length 77: HTTP: GET / HTTP/1.1
14:02:11.341912 IP 10.20.30.10.80 > 10.20.30.50.49288: Flags [.], ack 78, win 509, options [nop,nop,TS val 1806727562 ecr 3136385979], length 0
```

[NIET GETEST] The interface name and this output on `halden-srv01`. The line format was checked with tcpdump 4.99 on another machine.

Each line is one packet: time, source address and port, destination address and port, and the TCP flags. `[S]` is a SYN, `[S.]` a SYN-ACK and `[.]` an ACK. The first three lines are the three-way handshake from [1.5](#1_5), this time seen from the server.

The general form is:

```text
tcpdump -i <interface> <options> <capture filter>
```

| Option | Meaning |
| --- | --- |
| `-D` | List the interfaces |
| `-i <interface>` | Capture on this interface |
| `-n` | Show addresses and ports as numbers, without looking up names |
| `-c <count>` | Stop after this number of packets |
| `-w <file>` | Write the packets to a capture file |
| `-r <file>` | Read a capture file |

### Take the capture home

Reading packets in a terminal works for a quick check. For analysis you write them to a file and open that file in Wireshark.

1. On the server, capture to a file:

```bash
sudo tcpdump -i enp0s3 -w m1-server.pcap not port 22
```

2. In the second terminal, run `curl http://10.20.30.10/` once more.
3. Back on the server, press Ctrl+C. tcpdump reports how many packets it captured.
4. Type `exit` to leave the server.
5. In Kali, copy the file and open it:

```bash
scp consultant@10.20.30.10:m1-server.pcap .
wireshark m1-server.pcap &
```

You now see the same conversation as in [1.3](#1_3), captured at the other end of the cable.

::: warning
**Watch out:** with `-w`, tcpdump shows nothing on the screen while it captures. That is normal. Without a filter and without `-c`, the file keeps growing until the disk is full.
:::

What this means for you: you can now capture on any machine you can log in to, also when it has no desktop. From module 4 on, that machine is usually the firewall.

### Check yourself 1.6 – tcpdump

```text
Sectie: 1.6
Type: koppelen
Vraag: Match each tcpdump option with what it does.
Paren:
  -i → Chooses the interface to capture on
  -n → Shows addresses and ports as numbers
  -c → Stops after a number of packets
  -w → Writes the packets to a file
Feedback: A typical capture on a server combines all four, followed by a capture filter (see 1.6).

Sectie: 1.6
Type: meerkeuze
Vraag: You are logged in over SSH and run `sudo tcpdump -i enp0s3 -n`. The screen fills with lines without end. Why? Choose one answer.
Opties:
  A. tcpdump captures the SSH traffic that carries its own output
  B. The server is under attack
  C. The option `-n` repeats every packet
  D. The interface is broken
Juist: A
Feedback A: Correct. Every printed line travels to you over SSH, and that is new traffic to print. Add `not port 22` (see 1.6).
Feedback B: A busy screen is not evidence of an attack. Look at the port first: it is 22, your own session (see 1.6).
Feedback C: `-n` only stops tcpdump from translating addresses into names (see 1.6).
Feedback D: A broken interface shows nothing. This one shows your own session (see 1.6).

Sectie: 1.6
Type: meerkeuze
Vraag: In tcpdump output you see `Flags [S.]`. Which packet is this? Choose one answer.
Opties:
  A. A SYN-ACK
  B. A SYN
  C. An ACK
  D. The end of a connection
Juist: A
Feedback A: Correct. `S` is SYN, and the dot stands for ACK (see 1.6).
Feedback B: A SYN without ACK is shown as `[S]` (see 1.6).
Feedback C: An ACK on its own is shown as `[.]` (see 1.6).
Feedback D: The end of a connection is shown with `F` for FIN, or `R` for reset (see 1.6).
```

## Summary

New terms in this module:

- **packet**: one unit of data that travels over a network;
- **packet capture**: a recording of the packets that pass one network interface;
- **packet analyser**: a tool that records and decodes packets, such as **Wireshark** and **tcpdump**;
- **capture file**: a file that holds recorded packets;
- **display filter**: hides the captured packets that do not match a condition;
- **capture filter**: decides which packets are recorded at all;
- **stream**: all packets of one TCP connection;
- **plaintext**: data that anyone who captures it can read;
- **three-way handshake**: the exchange of SYN, SYN-ACK and ACK that opens a TCP connection.

The rules to remember:

- A capture shows one interface. Choose the one the traffic passes.
- Display filters look like `ip.addr == 10.20.30.10`, capture filters like `host 10.20.30.10`.
- What a capture filter drops is gone. Use a display filter unless you have a reason.
- Encryption hides the content, not the addresses, the DNS query or the server name.
- On a server you capture with tcpdump, with `not port 22` when you are logged in over SSH.
- Name your capture files and copy your evidence to the host.

## Common mistakes

**Capturing on the wrong interface.** You capture on `eth0`, ping the server and see nothing. Traffic to 10.20.30.0/24 leaves through `eth1`. Check the address of the target and choose the interface on that network.

**Typing a capture filter in the display filter bar.** `host 10.20.30.10` turns the bar red. In the display filter bar you write `ip.addr == 10.20.30.10`.

**Reading an empty list as "no traffic".** The filter `dns` from ten minutes ago is still active, so your ping is hidden. Clear the filter bar before you draw a conclusion.

**Using a capture filter that is too narrow.** You record with `port 80` and afterwards want to see the DNS lookup that came first. It was never recorded. Capture wide, and narrow down with display filters.

**Believing that HTTPS hides the destination.** The page is encrypted, but the Client Hello still shows `server_name: example.com`. Anyone on the path knows which site was visited.

**Forgetting `not port 22` on a server.** tcpdump prints without end, and the capture file fills with your own SSH session.

**Leaving the evidence in the virtual machine.** You restore a snapshot and the capture files are gone. Copy them to the host first (see [0.5](./module-0.md#0_5)).

::: oefening
[Exercises module 1](./module-1-exercises.md)
:::
