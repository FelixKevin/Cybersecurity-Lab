---
module: 4
order: 1
moduleTitle: "4: Firewalls"
published: false
---

# Module 4: Firewalls

After module 3 every packet between two zones passes `halden-fw01`, and the router lets all of it through. In this module you turn the router into a firewall. You decide which traffic may cross a border, you block everything else, and you write the rule set down so that someone else can maintain it.

### Learning objectives

After this module you can:

- read and write a firewall rule, and predict which rule a packet matches;
- explain why a reply passes a stateful firewall without a rule of its own;
- build a rule set on the principle of default deny;
- protect a DMZ so that a compromised public server cannot reach the inside;
- document a rule set with a reason and a test for every rule.

### What you need

| What | Requirement |
| --- | --- |
| Your lab from module 3 | Six machines, with the snapshot `m3-routed` |
| The traffic table | From [2.4](./module-2.md#2_4) |

The steps are written for OPNsense 26.7. You configure the firewall from Kali, in the browser on `https://10.20.99.1`.

## 4.1 What a firewall decides

In module 3 a workstation could open an SSH session to the file server, and the web server could start a connection to a workstation. Nobody at Halden & Co needs either. The router had no way to tell wanted traffic from unwanted traffic, because nobody had told it.

A **firewall** is a device that decides for traffic that passes whether it may continue. It decides with **rules**. All rules together are the **rule set**.

### Read the one rule you have

Go to **Firewall > Rules** and choose **Floating** in the interface filter. You see the rule that came with the appliance:

| Action | Interface | Protocol | Source | Destination | Port | Description |
| --- | --- | --- | --- | --- | --- | --- |
| Pass | any | any | any | any | any | Allow everything (module 3) |

Every rule answers the same questions. Where does the traffic enter? Where does it come from? Where does it go, and to which port? What happens then? This rule answers "anything" to all of them.

Disable that rule and apply. Then ping the file server from Kali:

```output
2 packets transmitted, 0 received, 100% packet loss, time 1021ms
```

A firewall without rules lets nothing through. You come back to that in [4.3](#4_3).

### Write your first rule

Rules belong to the interface where the traffic enters the firewall. Traffic from a workstation enters on the Clients interface.

1. In **Firewall > Rules**, choose **Clients** in the interface filter and add a rule.
2. Set **Action** to `Pass`, **Source** to `Clients net` and **Destination** to `any`.
3. Tick **Log**, set the description to `Clients: allow everything (temporary)` and save. Apply the changes.

The ping works again. Open **Firewall > Log Files > Live View** and ping once more. You see a green line for each packet the rule passed.

### The first match wins

Add a second rule on Clients: **Action** `Block`, **Protocol** `TCP`, **Source** `Clients net`, **Destination** `10.20.20.10`, destination port `22`, description `Clients: no SSH to the file server`. Move it above the first rule and apply.

```bash
ssh -o ConnectTimeout=5 consultant@10.20.20.10
```

```output
ssh: connect to host 10.20.20.10 port 22: Connection timed out
```

`curl http://10.20.20.10/` still shows the intranet. Now move the block rule below the pass rule and apply. SSH works again.

The firewall reads the rules of an interface from top to bottom and stops at the first rule that matches. A rule below a broader rule never gets a turn. Put the block rule back on top.

### Block or reject

Change the action of the block rule to `Reject` and try again:

```output
ssh: connect to host 10.20.20.10 port 22: Connection refused
```

| Action | What the firewall does | What the sender notices |
| --- | --- | --- |
| Pass | Lets the traffic through | A connection |
| Block | Drops the packet in silence | Nothing, until a timeout |
| Reject | Drops the packet and says so | An error at once |

Use `Block` for traffic from networks you do not trust: an attacker learns nothing and loses time. `Reject` is kinder to your own users, who get an error instead of a frozen screen. Set the rule back to `Block`.

[CONTROLEREN] All labels in 4.1 for OPNsense 26.7.

[NIET GETEST] On the lab machines. The behaviour of rule order, block and reject comes from a simulation; the `ssh` messages were not run.

What this means for you: when traffic is blocked that should pass, or the other way round, find the first rule it matches. The answer is in the order, not in the rule you were looking at.

### Check yourself 4.1 – What a firewall decides

```text
Sectie: 4.1
Type: meerkeuze
Vraag: The Clients interface has these rules, in this order. A workstation opens an SSH session to 10.20.20.10. What happens? Choose one answer.
Code:
  1. Pass   Clients net -> any          any port
  2. Block  Clients net -> 10.20.20.10  port 22
Opties:
  A. The session is allowed by rule 1
  B. The session is blocked by rule 2
  C. The session is blocked, because block rules always win
  D. Both rules are applied, and the last one decides
Juist: A
Feedback A: Correct. The firewall stops at the first rule that matches, and rule 1 matches everything (see 4.1).
Feedback B: Rule 2 never gets a turn. Rule 1 comes first and matches the same traffic (see 4.1).
Feedback C: The action of a rule does not give it priority. Only the order does (see 4.1).
Feedback D: The firewall stops at the first match and does not read on (see 4.1).

Sectie: 4.1
Type: koppelen
Vraag: Match each action with what the sender notices.
Paren:
  Pass → The connection works
  Block → Nothing happens, until a timeout
  Reject → An error arrives at once
Feedback: Block drops the packet in silence. Reject drops it and tells the sender (see 4.1).

Sectie: 4.1
Type: meerkeuze
Vraag: A workstation in the Clients zone connects to the web server in the DMZ. On which interface do you write the rule that allows it? Choose one answer.
Opties:
  A. Clients, because the traffic enters the firewall there
  B. DMZ, because the traffic goes there
  C. WAN, because the web server is public
  D. On both Clients and DMZ
Juist: A
Feedback A: Correct. A rule belongs to the interface where the connection enters the firewall (see 4.1).
Feedback B: The DMZ interface is where this traffic leaves the firewall. Rules there are for connections that the web server starts (see 4.1).
Feedback C: This connection does not pass the WAN interface at all (see 4.1).
Feedback D: One rule is enough, on the interface where the connection enters. Why the answer needs no rule, you see in 4.2 (see 4.1).
```

## 4.2 Stateful filtering

Look at what you built in [4.1](#4_1). The Clients interface has a pass rule. The Servers interface has no rule at all. And still the file server answers the ping of Kali. The answer enters the firewall on an interface where nothing is allowed.

### The firewall remembers

When a rule passes the first packet of a connection, the firewall writes down who talks to whom. That record is a **state**, and the list of all of them is the **state table**. Every later packet that fits a state passes without a look at the rules. A firewall that works this way is a **stateful firewall**.

Ping the file server from Kali and go to **Firewall > Diagnostics > States**. Search for `10.20.20.10`. You see the state of your ping: the two addresses, the protocol and the interface.

[CONTROLEREN] The label of the page and of the search field in OPNsense 26.7.

### Who may start

A state belongs to one connection, in the direction in which it started. Test the other direction. Log in on the console of `halden-srv01` and ping Kali:

```bash
ping -c 2 10.20.10.50
```

```output
2 packets transmitted, 0 received, 100% packet loss, time 1021ms
```

Kali can ping the server. The server cannot ping Kali. For the server's ping there is no state, so the firewall looks at the rules of the Servers interface, and there are none.

| Connection | Enters on | Rule | Result |
| --- | --- | --- | --- |
| Kali starts, the server answers | Clients | Pass on Clients | Works: the answer fits the state |
| The server starts | Servers | None | Blocked |

A rule does not say "these two machines may talk". It says "this side may start a conversation with that side". That difference is the most useful tool you have in this module.

### Why that matters

Without states you would need a second rule for the answers: from the server to the workstations, on every port above 1023. That rule would also let the server start connections on those ports. A stateful firewall lets answers through and nothing else.

States also exist for protocols without a connection of their own. For a ping or a DNS query the firewall remembers the request for a short time and lets the matching answer through.

::: warning
**Watch out:** a state outlives a rule change. If you block something that is running, the open connection can keep working until its state ends. Test a new block rule with a new connection.
:::

[NIET GETEST] On the lab machines. The one-way behaviour comes from a simulation with the same addresses.

What this means for you: for every piece of allowed traffic, ask who starts. The web server has to answer the internet. It never has to start a conversation with the inside.

### Check yourself 4.2 – Stateful filtering

```text
Sectie: 4.2
Type: meerkeuze
Vraag: The only rule on the firewall is "Pass, Clients net to Servers net" on the Clients interface. A server tries to open a connection to a workstation. What happens? Choose one answer.
Opties:
  A. It is blocked: no state exists and the Servers interface has no rule
  B. It is allowed: the rule covers traffic between the two zones
  C. It is allowed: answers always pass a stateful firewall
  D. It is blocked: servers can never start connections
Juist: A
Feedback A: Correct. This is a new connection that enters on Servers. There is no state for it and no rule that allows it (see 4.2).
Feedback B: A rule allows one side to start. It does not open the border in both directions (see 4.2).
Feedback C: This is not an answer. It is the first packet of a new connection (see 4.2).
Feedback D: Servers can start connections when a rule allows it. Here no rule does (see 4.2).

Sectie: 4.2
Type: meerkeuze
Vraag: Kali downloads a large file from the intranet. Halfway, you add a rule that blocks port 80 from Clients to the server. The download continues. Why? Choose one answer.
Opties:
  A. The connection already has a state, and packets that fit a state skip the rules
  B. Block rules only work after a restart of the firewall
  C. The download uses another port than 80
  D. The rule is on the wrong interface
Juist: A
Feedback A: Correct. A state outlives a rule change. Test with a new connection (see 4.2).
Feedback B: A rule works as soon as you apply it, for new connections (see 4.2).
Feedback C: The download runs over the same connection to port 80 on which it started (see 4.2).
Feedback D: The Clients interface is where this connection entered. The rule is in the right place and comes too late for this one connection (see 4.2).

Sectie: 4.2
Type: meerdere antwoorden
Vraag: What does a stateful firewall record in a state? Choose all correct answers.
Opties:
  A. The source and destination address
  B. The protocol and the ports
  C. The password of the user
  D. The content of the files that pass
Juist: A, B
Feedback A: Correct. A state describes who talks to whom (see 4.2).
Feedback B: Correct. Together with the addresses they identify one connection (see 4.2).
Feedback C: A state describes a connection, not what is said inside it (see 4.2).
Feedback D: The firewall in this module looks at addresses, protocols and ports. It does not read the content (see 4.2).
```

## 4.3 Default deny

Your Clients interface has two rules: block SSH to the file server, then allow everything. Tomorrow you think of something else to block, and next week of another thing. You will never finish that list, because it has to contain everything an attacker could try.

Turn the question round. **Default deny** means that everything is blocked unless a rule allows it. The list of what Halden's workstations need is short, and you already wrote it in [2.4](./module-2.md#2_4).

OPNsense works this way without your help. When no rule matches, the firewall blocks. You saw it in [4.1](#4_1) when you disabled the only rule.

### Name things first

Two values come back in many rules. An **alias** is a name for a list of addresses, networks or ports that you can use in rules.

Go to **Firewall > Aliases** and add two:

| Name | Type | Content | Description |
| --- | --- | --- | --- |
| `Internal_networks` | Network(s) | `10.20.0.0/16` | All zones of Halden & Co |
| `Web_ports` | Port(s) | `80`, `443` | HTTP and HTTPS |

### The rule set of the Clients zone

Delete the two rules on the Clients interface and build these four. Tick **Log** on each one.

| # | Action | Protocol | Source | Destination | Port | Description |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Pass | TCP | Clients net | `10.20.20.10` | 80 | Clients: intranet |
| 2 | Pass | TCP | Clients net | `10.20.20.10` | 445 | Clients: client files |
| 3 | Pass | TCP/UDP | Clients net | This Firewall | 53 | Clients: DNS |
| 4 | Pass | TCP | Clients net | not `Internal_networks` | `Web_ports` | Clients: web |

For rule 4, choose `Internal_networks` as the destination and tick **Invert Destination**. The rule then matches every destination outside Halden's own zones, which is the internet. Without the inversion you would write `any`, and a workstation could reach the web server and the firewall's own pages too.

[CONTROLEREN] All labels in 4.3 for OPNsense 26.7, including the alias types and the inversion of the destination.

### Test what works and what does not

| Test from Kali | Expected | Why |
| --- | --- | --- |
| `curl http://10.20.20.10/` | The intranet page | Rule 1 |
| `ssh -o ConnectTimeout=5 consultant@10.20.20.10` | Timeout | No rule |
| `curl -m 5 http://10.20.40.10/` | Timeout | No rule: staff do not need the DMZ |
| `curl -I https://example.com` | An answer | Rules 3 and 4 |
| `ping -c 2 1.1.1.1` | 100% packet loss | No rule |

The last line may surprise you. Nobody decided to block ping to the internet. Nobody decided to allow it, and under default deny that is enough.

```output
curl: (28) Connection timed out after 5002 milliseconds
```

[NIET GETEST] On the lab machines. The five results come from a simulation of the same rule set; the wording of the `curl` message depends on the version.

### Find what you forgot

Something will stop working that someone needs. Open **Firewall > Log Files > Live View** and filter on the action `block`. Each red line shows the interface, the source, the destination and the port. That is all the information you need to decide whether a rule is missing.

Decide, do not just add. A blocked ping is a question for Tom: do the workstations need it? If so, it gets a rule and a reason.

::: warning
**Watch out:** default deny also covers traffic that leaves. A workstation that may reach only ports 80 and 443 on the internet is a poorer base for an attacker than one that may reach everything.
:::

What this means for you: a rule set under default deny is a list of decisions. Everything on it was allowed by someone, for a reason. Everything else is blocked without anyone having to think of it.

### Check yourself 4.3 – Default deny

```text
Sectie: 4.3
Type: meerkeuze
Vraag: An interface has two rules: "Pass TCP port 80" and "Pass TCP port 443". A workstation behind it sends a DNS query to a server on the internet. What happens under default deny? Choose one answer.
Opties:
  A. It is blocked, because no rule allows it
  B. It is allowed, because no rule blocks it
  C. It is allowed, because DNS is needed for web traffic
  D. It is blocked, because DNS is always blocked
Juist: A
Feedback A: Correct. Under default deny, traffic that matches no rule is blocked (see 4.3).
Feedback B: That would be default allow. A firewall with default deny needs a rule to let traffic through (see 4.3).
Feedback C: The firewall does not know what a workstation needs. It applies the rules it has (see 4.3).
Feedback D: Nothing is blocked "always". DNS is blocked here because nobody allowed it (see 4.3).

Sectie: 4.3
Type: meerkeuze
Vraag: Rule 4 allows Clients to reach "not `Internal_networks`" on the web ports. Why not destination `any`? Choose one answer.
Opties:
  A. `any` would also allow the web ports of every other zone of Halden & Co
  B. `any` is not a valid destination in a pass rule
  C. `any` would block the internet
  D. `any` is slower for the firewall
Juist: A
Feedback A: Correct. With `any`, a workstation could reach port 80 and 443 in the DMZ, in Management and on the firewall itself (see 4.3).
Feedback B: `any` is valid. It allows more than this rule should (see 4.3).
Feedback C: `any` includes the internet. The problem is everything else it includes (see 4.3).
Feedback D: The reason is what the rule allows, not how fast it is (see 4.3).

Sectie: 4.3
Type: categoriseren
Vraag: The Clients zone has the four rules of this section. Is the traffic from a workstation allowed or blocked?
Indeling:
  HTTP to the intranet on 10.20.20.10 → Allowed
  HTTPS to a website on the internet → Allowed
  A DNS query to the firewall → Allowed
  SSH to 10.20.20.10 → Blocked
  HTTP to the web server 10.20.40.10 → Blocked
  A ping to 1.1.1.1 → Blocked
Feedback: Allowed is what one of the four rules names: ports 80 and 445 on the file server, DNS on the firewall, and the web ports outside Halden's networks. Everything else is blocked (see 4.3).
```

## 4.4 The DMZ

`halden-web01` is the one machine at Halden & Co that anyone on the internet may talk to. Sooner or later someone finds a weakness in it. The question for your design is what that person can do next.

A **DMZ** is a zone for servers that the internet can reach, cut off from the internal zones. The idea is to plan for the loss of what stands in it. If the web server falls, the attacker should be standing in an empty room.

### Who starts, again

Go back to the traffic table of [2.4](./module-2.md#2_4). One line said: "DMZ to Servers, handing over the uploaded invoices". With what you know from [4.2](#4_2), that line is a problem. It lets the most exposed machine start a connection to the most valuable one.

Turn it round. The file server fetches the invoices from the web server, over SSH. The traffic is the same, and the one who starts is the machine you trust.

| From | To | Port | Who starts |
| --- | --- | --- | --- |
| Internet | `halden-web01` | 80 | The internet |
| `halden-srv01` | `halden-web01` | 22 | The file server |
| `halden-web01` | Any internal zone | — | Nobody: not allowed |

### The rules

On the **DMZ** interface:

| # | Action | Protocol | Source | Destination | Port | Description |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Pass | TCP/UDP | DMZ net | This Firewall | 53 | DMZ: DNS |
| 2 | Block | any | DMZ net | `Internal_networks` | any | DMZ: never to the inside |
| 3 | Pass | TCP | DMZ net | not `Internal_networks` | `Web_ports` | DMZ: updates |

On the **Servers** interface:

| # | Action | Protocol | Source | Destination | Port | Description |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | Pass | TCP | `10.20.20.10` | `10.20.40.10` | 22 | Servers: fetch uploaded invoices |
| 2 | Pass | TCP/UDP | Servers net | This Firewall | 53 | Servers: DNS |
| 3 | Pass | TCP | Servers net | not `Internal_networks` | `Web_ports` | Servers: updates |

Tick **Log** on every rule. Rule 2 of the DMZ blocks what default deny would block anyway. You write it down because you want its log lines: a web server that tries to reach the inside is not a mistake, it is an alarm. You use those lines in module 9.

### Make the opening visible

In [3.5](./module-3.md#3_5) the port forward passed its own traffic, without a rule that you can see in the rule set. Open the rule in **Firewall > NAT > Destination NAT (Port Forward)** and set **Firewall Rule** to `Register rule`. A rule for port 80 to 10.20.40.10 now appears under **Firewall > Rules** on the WAN interface, where it belongs: with all other decisions.

[CONTROLEREN] All labels in 4.4 for OPNsense 26.7, in particular the options of **Firewall Rule**.

### Test the empty room

| Test | From | Expected |
| --- | --- | --- |
| `http://localhost:8080` | Your own laptop | The client portal |
| `curl -m 5 http://10.20.20.10/` | The console of `halden-web01` | Timeout |
| `ping -c 1 10.20.10.50` | The console of `halden-web01` | 100% packet loss |
| `ssh consultant@10.20.40.10` | The console of `halden-srv01` | A login prompt |
| `curl -m 5 http://10.20.40.10/` | The console of `halden-srv01` | Timeout: only port 22 is allowed |

After the second and third test, look at the live log. You see the lines of rule 2, with the address of the web server as the source.

[NIET GETEST] On the lab machines. All five results come from a simulation of the same rule set.

::: warning
**Watch out:** the DMZ protects the inside from the web server. It does not protect the web server. The portal is as reachable and as vulnerable as before, and you look at that in modules 5 and 7.
:::

What this means for you: you can now answer Nadia's first fear with a design. If the public server is taken over, the client files are not one step away. They are behind a border that the web server may not cross.

### Check yourself 4.4 – The DMZ

```text
Sectie: 4.4
Type: meerkeuze
Vraag: Uploaded invoices have to get from the web server in the DMZ to the file server. Which design is the safest? Choose one answer.
Opties:
  A. The file server connects to the web server and fetches them
  B. The web server connects to the file server and delivers them
  C. Both servers may connect to each other
  D. The web server stores them on a workstation
Juist: A
Feedback A: Correct. The trusted machine starts, so no rule lets the DMZ open a connection to the inside (see 4.4).
Feedback B: That needs a rule from the DMZ to the Servers zone. A compromised web server could use it (see 4.4).
Feedback C: This allows more than either one-way design and has the weakness of the worst of them (see 4.4).
Feedback D: That lets the web server start a connection into the Clients zone, which is also the inside (see 4.4).

Sectie: 4.4
Type: meerkeuze
Vraag: Rule 2 on the DMZ interface blocks traffic to the internal networks. Default deny would block it too. Why write the rule? Choose one answer.
Opties:
  A. To get a clear log line when the web server tries to reach the inside
  B. Because default deny does not work on a DMZ
  C. Because block rules are faster than default deny
  D. To allow the answers of the web server
Juist: A
Feedback A: Correct. The traffic is blocked either way. The rule gives the attempt a name in the log (see 4.4).
Feedback B: Default deny works on every interface (see 4.3).
Feedback C: Speed is not the reason. Visibility is (see 4.4).
Feedback D: Answers pass because of the state of the connection. They need no rule (see 4.2).

Sectie: 4.4
Type: meerdere antwoorden
Vraag: An attacker takes over `halden-web01`. With the rule set of this section, what can the attacker do from that machine? Choose all correct answers.
Opties:
  A. Change the client portal
  B. Download tools from the internet over HTTPS
  C. Open a connection to the file server
  D. Scan the workstations
Juist: A, B
Feedback A: Correct. The DMZ protects the inside, not the web server itself (see 4.4).
Feedback B: Correct. Rule 3 allows the web ports to the internet, for updates. An attacker can use the same rule (see 4.4).
Feedback C: Rule 2 blocks every connection from the DMZ to the internal networks (see 4.4).
Feedback D: The Clients zone is part of the internal networks, so rule 2 blocks that too (see 4.4).
```

## 4.5 Documenting a rule set

Your rule set has eleven rules of your own and works. In a year it has forty. Tom finds one that allows port 8443 to an address nobody recognises, and he does not dare to remove it. A rule that nobody can explain stays for ever.

### The description field is not enough

You gave every rule a description. That tells Tom what the rule is for. It does not tell him who asked for it, why, or how to check that it still works. For that you keep a **rule register**: a document with one row per rule, its reason and its test.

| Interface | # | Rule | Reason | Asked by | Test |
| --- | --- | --- | --- | --- | --- |
| Clients | 1 | Pass TCP 80 to `10.20.20.10` | Staff use the intranet | Tom | `curl http://10.20.20.10/` from a workstation shows the page |
| Clients | 2 | Pass TCP 445 to `10.20.20.10` | Staff open client files | Tom | A workstation opens a share |
| DMZ | 2 | Block to `Internal_networks` | A compromised web server must not reach the inside | You | `curl` from `halden-web01` to `10.20.20.10` times out, and the log shows the block |
| WAN | 1 | Pass TCP 80 to `10.20.40.10` | Clients upload invoices | Nadia | `http://localhost:8080` shows the portal |

The other rules follow the same pattern. The reason links a rule to an asset or to one of Nadia's fears. If you cannot fill in the reason, the rule should not exist.

### Three checks for a rule set

1. **Every rule has a reason and a test.** The register is complete when it has as many rows as the firewall has rules.
2. **Every test has been run.** A pass rule has a test that works, and where it matters a test that fails: SSH to the file server from a workstation.
3. **No rule is wider than its reason.** "Staff use the intranet" does not need every port of the file server.

### Rules that nobody uses

The firewall counts how often each rule matches. In **Firewall > Rules**, click **Inspect** to see those numbers. A pass rule that has matched nothing in three months is a question for the person who asked for it.

### Keep the configuration itself

1. Go to **System > Configuration > Backups** and download the configuration. You get one XML file with every setting of the firewall.
2. Store the file on your own laptop, with the date in its name: `halden-fw01-m4.xml`.
3. Shut down all six machines and take a snapshot of each with the name `m4-firewall`.

[CONTROLEREN] The labels of **Inspect** and of the backup page in OPNsense 26.7.

::: warning
**Watch out:** the backup contains the whole design of the network and the password hashes of the firewall. Treat it like the client files: do not mail it and do not put it in a public repository.
:::

What this means for you: at the end of this course you hand over a network, not a set of screenshots. The rule register is what lets Tom change a rule in a year and know what he is doing.

### Check yourself 4.5 – Documenting a rule set

```text
Sectie: 4.5
Type: meerkeuze
Vraag: During a review you find a pass rule for port 8443 to an address in the Servers zone. It has no description and is not in the rule register. What do you do first? Choose one answer.
Opties:
  A. Find out who uses it, with the counters and the log, and ask that person for the reason
  B. Delete it at once
  C. Leave it: it has always been there
  D. Add a description "unknown" and move on
Juist: A
Feedback A: Correct. The counters and the log tell you whether and by whom the rule is used. Then it gets a reason, or it goes (see 4.5).
Feedback B: The rule may carry something the office needs. Find out first, then decide (see 4.5).
Feedback C: That is how a rule set fills up with rules nobody can explain (see 4.5).
Feedback D: A description without a reason changes nothing. The rule still has no owner and no test (see 4.5).

Sectie: 4.5
Type: meerdere antwoorden
Vraag: What belongs in a row of the rule register? Choose all correct answers.
Opties:
  A. The reason for the rule
  B. A test that shows the rule works
  C. The person who asked for the rule
  D. The password of the firewall
Juist: A, B, C
Feedback A: Correct. A rule without a reason should not exist (see 4.5).
Feedback B: Correct. The test lets someone else check the rule later (see 4.5).
Feedback C: Correct. That is the person to ask when the rule is reviewed (see 4.5).
Feedback D: Credentials never belong in documentation that is shared (see 4.5).

Sectie: 4.5
Type: meerkeuze
Vraag: The reason of a rule is "staff use the intranet". The rule allows every port from Clients to the file server. What is wrong? Choose one answer.
Opties:
  A. The rule is wider than its reason: the intranet needs one port
  B. Nothing: the reason is filled in
  C. The rule is on the wrong interface
  D. The rule should be a block rule
Juist: A
Feedback A: Correct. A rule should allow what its reason needs, and no more (see 4.5).
Feedback B: A reason is necessary and not enough. The rule also has to fit it (see 4.5).
Feedback C: Traffic from workstations enters on Clients. The interface is right (see 4.1).
Feedback D: The intranet has to be reachable. The rule has to become narrower, not disappear (see 4.5).
```

## Summary

New terms in this module:

- **firewall**: a device that decides for traffic that passes whether it may continue;
- **rule** and **rule set**: one decision of a firewall, and all of them together;
- **state** and **state table**: the record of one allowed connection, and the list of all of them;
- **stateful firewall**: a firewall that lets every packet that fits a state pass without a look at the rules;
- **default deny**: everything is blocked unless a rule allows it;
- **alias**: a name for a list of addresses, networks or ports;
- **DMZ**: a zone for servers that the internet can reach, cut off from the internal zones;
- **rule register**: a document with one row per rule, its reason and its test.

The rules to remember:

- A rule belongs to the interface where the connection enters the firewall.
- The first rule that matches decides. Order is part of the rule set.
- A rule says who may start. The answer passes because of the state.
- Allow what is needed and let default deny block the rest.
- The DMZ never starts a connection to the inside.
- Every rule has a reason, an owner and a test.

## Common mistakes

**Putting the rule on the interface where the traffic leaves.** You want workstations to reach the intranet and add the rule on Servers. Nothing changes. The connection enters on Clients, so the rule goes there.

**Writing a rule for the answer.** You add "Servers to Clients, any port" so that replies can pass. They already could, because of the state. The extra rule only lets the server start connections.

**Placing a specific rule under a broad one.** The block for SSH sits below "allow everything" and never matches. Read the rule set from top to bottom, as the firewall does.

**Using `any` as the destination for internet traffic.** "Clients to any, port 443" also opens port 443 in Management and on the firewall itself. Use the inverted alias for everything outside your own networks.

**Testing a block with a connection that is already open.** The old connection keeps its state and keeps working. Start a new one.

**Testing only what should work.** The intranet opens, so the rule set is "done". Half of the tests in a firewall project are supposed to fail. Run those too.

**Letting the DMZ deliver to the inside.** It looks the same as fetching, and it needs a rule from the most exposed machine to the most valuable one. Let the inside fetch.

**Adding a rule because something stopped working.** The log shows a block and you allow it, without asking whether it was needed. Under default deny every rule is a decision, with a name behind it.

::: oefening
[Exercises module 4](./module-4-exercises.md)
:::
