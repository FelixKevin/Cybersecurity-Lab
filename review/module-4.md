# Review module 4

Written in English on the structure of the Schrijfwijzer (chapters 9, 11, 12A and 17).

## New terms

| Term | Definition at introduction |
| --- | --- |
| firewall | a device that decides for traffic that passes whether it may continue |
| rule / rule set | one decision of a firewall / all rules together |
| state / state table | the record of who talks to whom for one allowed connection / the list of all of them |
| stateful firewall | a firewall in which every packet that fits a state passes without a look at the rules |
| default deny | everything is blocked unless a rule allows it |
| alias | a name for a list of addresses, networks or ports that you can use in rules |
| DMZ | a zone for servers that the internet can reach, cut off from the internal zones |
| rule register | a document with one row per rule, its reason and its test |

"DMZ" was used as the name of a zone since 2.4 and is introduced as a term here.

## Decisions made while writing (confirm or change)

- **Students disable the allow-all rule in 4.1 and see the firewall block everything**, then write their first pass rule themselves. Default deny is experienced first and named in 4.3.
- **Rules are written per interface, on the interface where the connection enters.** Floating rules are only met as the place where the allow-all rule lives.
- **The hand-over of invoices is turned round in 4.4:** the file server fetches from the web server over SSH, instead of the DMZ delivering to the Servers zone as the traffic table of 2.4 said. The text presents this as an improvement that follows from 4.2.
- **The final rule set has eleven rules:** four on Clients, three on Servers, three on DMZ and one on WAN. Management keeps the single allow-all rule of the appliance; tightening it is exercise X3.
- **Rule 2 on the DMZ is an explicit, logged block** that default deny would also enforce. It prepares the detection work of module 9.
- **The port forward of 3.5 gets a visible rule** on the WAN interface, by changing its firewall rule option.
- **Workstations lose ping to the internet** as a deliberate surprise of default deny. Whether to allow it is exercise X2.
- **Tests use `curl`, `ssh` and `ping`.** Port scanning with nmap is kept for module 5.
- **The configuration backup is stored on the host** as part of the evidence, with a warning about what it contains.
- The three questions left open after module 3 (static-route example, shell or web interface for captures, DHCP for Clients) were not answered yet; this module does not depend on them.

## Added requirements for the appliances

- `halden-fw01.ova`: a pass rule on the Management interface that keeps the web interface reachable when the floating allow-all rule is disabled. Without it students lock themselves out in the first step of 4.1.
- `halden-fw01.ova`: logging of the default block rule switched on (the default), so that 4.3 shows blocked traffic in the live log.
- `halden-fw01.ova`: the DNS resolver answers on all zone interfaces (rule 3 of each zone points at "This Firewall").
- `halden-web01.ova`: an SSH server and the account `consultant`, for the fetch test in 4.4.
- `halden-srv01.ova`: an SSH client and `curl`.
- `halden-web01.ova` and `halden-srv01.ova`: console login for `consultant`, and `curl` and `ping` installed.
- The file server offers nothing on port 445. Rule 2 of Clients is therefore not testable in the lab; the register in 4.5 names "a workstation opens a share" as its test. Either add a file share to the appliance or accept that this rule has no working test.

## Open questions

- **Labels.** OPNsense 26.7 moved firewall rules to a new interface. Every step in this module names fields (Action, Source, Destination, Log, Invert Destination, Inspect, Register rule) that have to be checked against the appliance.
- 4.3: the destination inversion is the least intuitive step of the module. Is "not `Internal_networks`" clear enough, or should the text use an explicit block rule followed by "pass to any" instead?
- 4.4: the DMZ may reach the internet on the web ports for updates. An attacker can use that too, and the quiz says so. Keep it, or restrict updates to a named mirror?
- A file share on `halden-srv01` would make rule 2 of Clients testable and gives module 7 a realistic target. Add it to the appliance?
- Example solutions (O6) are not written yet.

## Markers

- [CONTROLEREN] 4.1: all labels for OPNsense 26.7.
- [NIET GETEST] 4.1: rule order, block and reject on the lab machines; the `ssh` messages.
- [CONTROLEREN] 4.2: the label of the states page and its search field.
- [NIET GETEST] 4.2: the one-way behaviour on the lab machines.
- [CONTROLEREN] 4.3: all labels, including alias types and the inversion of the destination.
- [NIET GETEST] 4.3: the five test results on the lab machines.
- [CONTROLEREN] 4.4: all labels, in particular the options of the firewall rule field of a port forward.
- [NIET GETEST] 4.4: the five test results on the lab machines.
- [CONTROLEREN] 4.5: the labels of Inspect and of the backup page.

## What was checked

The rule sets were run in a simulation: Linux network namespaces with the lab addresses, and nftables with a drop policy, connection tracking and the same rules.

- No rules: 100% packet loss. One pass rule on Clients: Kali reaches the server, and the server cannot ping Kali.
- A block for port 22 above the pass rule: timeout, and port 80 still works. The same block below the pass rule: the connection is allowed. Reject instead of block: an error at once.
- The four rules of Clients: intranet works; port 22 on the server, the DMZ web server and ping to the internet are blocked; the web ports to an outside address work.
- The DMZ and Servers rules: the web server cannot reach the file server or Kali; the port forward from outside still shows the portal; the file server reaches port 22 of the web server and not port 80.
- The OPNsense menu paths and the meaning of pass, block and reject were compared with the current manual page on firewall rules.

Not checked: anything in OPNsense itself, and the `ssh` commands.
