# Review module 1

Written in English on the structure of the Schrijfwijzer (chapters 9, 11, 12A and 17).

## New terms

| Term | Definition at introduction |
| --- | --- |
| packet | one unit of data that travels over a network |
| packet capture | a recording of the packets that pass one network interface |
| packet analyser | the tool that records and decodes packets |
| Wireshark | the packet analyser with a graphical interface used in this course |
| capture file | a file that holds the recorded packets |
| display filter | hides every captured packet that does not match a condition |
| capture filter | decides which packets are recorded at all |
| stream | all packets of one TCP connection |
| plaintext | data that anyone who captures it can read |
| three-way handshake | the exchange that opens every TCP connection |
| tcpdump | a packet analyser for the command line |

## Requirements for `halden-srv01.ova` that this module adds

- A web server on port 80 without encryption, serving a small "Halden & Co intranet" page (1.3, 1.4, 1.5).
- SSH on port 22 with an account `consultant` and a password that students receive (1.5).
- `sudo` rights for `consultant`, at least for `tcpdump` (1.5).
- `tcpdump` installed (1.5).
- The module assumes the interface is called `enp0s3`; adjust the text to the real name.

Module 0 says "you do not log in" on the server console. Module 1 introduces the account over SSH; that is consistent, but the password still has to reach students somehow.

## Decisions made while writing (confirm or change)

- **Exercises follow the variant for courses without programming (12A):** E1, E2, M1, M2, H1, @1, a case with three parts and three extra exercises. The programming variant (example input and output per exercise) does not fit lab work.
- **The case exercises C1 to C3 are the portfolio deliverable** of this module (annotated captures plus a short note). Say so if the portfolio should be a separate assignment.
- **E, M, H and @home use other small businesses** for context (N4), but students carry them out in their own lab, because there is no other data set.
- **Exercise file name:** `module-1-exercises.md` instead of `-oefeningen.md`.
- **Traffic to the internet (DHCP, DNS, TLS) is captured on `eth0`**, the NAT adapter of Kali. This depends on an internet connection and on `example.com`.
- **The classic `!=` pitfall is left out.** Since Wireshark 4.0, `ip.addr != x` means the same as `!(ip.addr == x)`.
- **No screenshots yet.** The three panes (1.1) and the Follow TCP Stream window (1.3) are the two places where one would help most.
- **No example solutions yet (O6).**

## Open questions

- 1.4 is the longest section (five protocols, about 600 words). Split into two sections, or keep as one?
- 1.4, DHCP: on a reconnect Kali may send only Request and ACK. The text explains this, but a guaranteed full exchange would be better. Is there a reliable way to force a Discover?
- 1.5: should students also learn `tcpdump -r` here, or is opening the file in Wireshark enough?
- H1 depends on which mirrors `apt update` contacts and on whether they use HTTP or HTTPS. The answer differs per student and per day. Acceptable for a Hard exercise?
- Cross-reference to "Computer Networks" in 1.1 and 1.4: check the course name against the final curriculum (S2).

## Markers

- [NIET GETEST] 1.1, Your first capture: starting Wireshark as `kali` and capturing without extra permissions.
- [NIET GETEST] 1.3: the request and response of the intranet page.
- [NIET GETEST] 1.4: all commands and the number of packets they produce.
- [CONTROLEREN] 1.5, Log in on the server: the account `consultant`, its password and sudo rights.
- [NIET GETEST] 1.5, Capture in the terminal: the interface name and the output on `halden-srv01`.

## What was checked

- Every display filter in the text, the quizzes and the exercises was parsed with tshark 4.2.2 (syntax only).
- Every capture filter was compiled with tcpdump 4.99.4 (`tcpdump -d`).
- The tcpdump line format and the flags `[S]`, `[S.]`, `[.]`, `[P.]` come from a real capture of an HTTP request on a loopback interface; addresses and ports in the text are rewritten to the lab values.
- Not checked: anything in the Wireshark window, anything on Kali, and anything that needs `halden-srv01`.
