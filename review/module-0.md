# Review module 0

Written in English on the structure of the Schrijfwijzer (chapters 9, 10, 11 and 17). The Dutch-specific language rules (T1–T6, tables 13A–13D) do not apply; the register, perspective and readability rules (R, P, L) are applied to the English text.

## New terms

| Term | Definition at introduction |
| --- | --- |
| rules of engagement | the agreement that states what a tester may attack, when and how |
| scope | the list of systems that may be attacked |
| virtual machine (VM) | a computer that exists only as software and runs inside another computer |
| hypervisor | the program that runs virtual machines |
| host / guest | your laptop / a virtual machine on it |
| Kali Linux | a Linux distribution that comes with the tools security testers use |
| internal network | a network that exists only inside VirtualBox, between the virtual machines attached to it |
| snapshot | a saved state of a virtual machine that you can return to later |

"Flat network" is mentioned in 0.1 without bold: it is introduced as a term in module 2.

## Decisions made while writing (confirm or change)

- **Hypervisor: VirtualBox 7.2**, the same for every student. Free, and one set of steps for Windows, Linux and Intel macOS.
- **Kali: the official prebuilt VirtualBox image**, not an installation from ISO.
- **Addressing in module 0:** lab network `halden-lab`, 10.20.30.0/24, Kali on .50, `halden-srv01` on .10. Provisional: the real addressing plan comes with the zone design in module 2.
- **Kali keeps a NAT adapter** next to the lab network, for updates and tools. The server has only the lab network.
- **Case characters:** Nadia El Amrani (managing partner) and Tom Peeters (office manager, IT on the side). 25 employees, about 400 clients.
- **No exercise file and no "Common mistakes" section** for module 0. It is a setup module (M1); the final check (M4) and "Not working?" (M5) take their place. The module therefore does not end with an exercise link block.
- **Check yourself only after 0.2 and 0.5.** Sections 0.1, 0.3 and 0.4 have no concept to test that is not checked by doing the steps.
- **Quiz type juist/fout is avoided** as long as the tool shows "Juist" and "Fout" to students.
- `quizzes.addAnswerHint` is set to `false` in `course.config.json`: the tool would otherwise append the Dutch sentence "Kies één antwoord." The English hint is written in every question.

## Open questions

- **ARM laptops:** what is the fallback for students with Apple silicon or Snapdragon? The module only tells them to contact the lecturer.
- **`halden-srv01.ova` has to be built.** Minimum for module 0: a small Linux server, static address 10.20.30.10/24, one adapter on internal network `halden-lab`, answers ping. Which services it runs depends on modules 5 to 8.
- **Where do students download the appliance?** An OVA is probably too large for `content/files/`; the text says "the course files on Canvas".
- **Hyper-V:** switching it off breaks WSL 2 and Docker Desktop, which students use in Full-Stack Development and later DevOps. Accept the slow mode, or prescribe a way to switch?
- **Screenshots:** none yet (M6). To add once the steps are tested on VirtualBox 7.2: adapter 2 settings, the Snapshots tool.
- **Sanctions and policy** in 0.2: reference to the EhB ICT policy, and what happens after a violation.
- **Changelog (S4):** where does this course keep it?
- **Feedback links:** the tool turns "zie 2.4" in quiz feedback into a link. "see 0.2" is probably not recognised, so it stays plain text until the tool supports English.

## Markers

- [CONTROLEREN] What you need: free disk space (60 GB) and the download location of `halden-srv01.ova`.
- [CONTROLEREN] What you need: fallback for ARM laptops.
- [CONTROLEREN] 0.2: reference to the EhB acceptable-use policy and consequences.
- [CONTROLEREN] 0.3, Install VirtualBox, step 4: menu label for the version number.
- [CONTROLEREN] 0.4, Add Kali, steps 3 to 7: labels in VirtualBox 7.2, and whether Expert mode is needed for Adapter 2.
- [CONTROLEREN] 0.4, Import the server: depends on how `halden-srv01.ova` is built.
- [NIET GETEST] 0.4, Give Kali an address: the `nmcli` commands and the interface name `eth1`.
- [CONTROLEREN] 0.5, Take a snapshot: location and labels of the Snapshots tool in VirtualBox 7.2.
- [NIET GETEST] Final check: the ping output.
- [CONTROLEREN] Not working?: the Hyper-V row.

Nothing in this module was run: no VirtualBox or Kali was available while writing. Every step needs one pass on a real laptop per operating system.
