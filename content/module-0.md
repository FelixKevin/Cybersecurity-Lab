---
module: 0
order: 1
moduleTitle: "0: Getting started"
published: false
---

# Module 0: Getting started

In this course you build a company network, attack it and then detect your own attacks. All of that happens in a lab on your own laptop. In this module you meet the company, agree on the rules and get the lab running.

### Learning objectives

After this module you can:

- describe Halden & Co and name the assets the firm wants to protect;
- decide for a given action whether it is inside or outside the scope of the lab;
- run two virtual machines on an isolated network in VirtualBox;
- take a snapshot of a virtual machine and restore it.

### What you need

Check this list before you start. The steps in this module are written for VirtualBox 7.2 on Windows 11, Linux and macOS on Intel.

| What | Requirement |
| --- | --- |
| Processor | 64-bit Intel or AMD (x86-64), with virtualisation switched on |
| Memory | 16 GB RAM |
| Free disk space | 60 GB |
| Operating system | Windows 11, a recent Linux distribution or macOS on Intel |
| VirtualBox | Version 7.2 or newer |
| Kali Linux | The current prebuilt image for VirtualBox, 64-bit |
| Archive tool | A tool that opens `.7z` files, such as 7-Zip |
| Lab file | `halden-srv01.ova`, from the course files on Canvas |

[CONTROLEREN] Disk space, and where students download `halden-srv01.ova`.

::: warning
**Watch out:** a laptop with an ARM processor cannot run this lab. That includes every Mac with Apple silicon (M1 or newer) and Windows laptops with a Snapdragon processor. Contact your lecturer before the first lesson if this is your situation.
:::

[CONTROLEREN] The fallback for students with an ARM laptop.

## 0.1 Meet Halden & Co

Halden & Co is an accounting firm in Brussels with 25 employees. The firm keeps the books for about 400 small businesses and self-employed clients. It also handles their tax returns and payroll.

Halden & Co is a fictional company. We use it in every module of this course, so it pays to know it well.

### Who you are

You work for a small security consultancy. Halden & Co has hired your company for the next twelve weeks. Your assignment: rebuild their network so that it is segmented, tested and monitored.

Your two contacts at the firm are:

- **Nadia El Amrani**, managing partner. She decides about budget and wants to know what a risk means for her clients.
- **Tom Peeters**, office manager. He looks after IT next to his real job, and he is glad you are here.

### What Halden & Co owns

| Asset | Where it lives | Why it matters |
| --- | --- | --- |
| Client files (annual accounts, tax returns) | File server in the office | Confidential, and clients trust the firm with them |
| Payroll data | File server in the office | Personal data of thousands of employees of clients |
| Accounting software | On every workstation | Without it nobody can work |
| Staff workstations | 25 desks, a mix of Windows and Linux | The way into everything else |
| Client portal | One public web server | Clients upload their invoices here |

### What worries them

Nadia names three fears, in this order:

1. Client data ends up with someone who should not have it.
2. The office cannot work in the weeks before a tax deadline.
3. Ransomware locks the file server.

### The network today

Every device at Halden & Co sits on one network: workstations, the file server, the printer and the web server. The network was set up years ago by a relative of one of the partners. It has worked ever since, so nobody has looked at it since.

A network without internal borders is called a flat network. You learn in module 2 why that is a problem, and you fix it there.

What this means for you: every design choice you make in this course needs a reason that Nadia would accept. "Because the file server holds payroll data" is a reason. "Because the tutorial said so" is not.

## 0.2 Rules of engagement

In this course you use the tools that real attackers use. A port scanner does not know whether it is pointed at your lab or at the campus network. You have to know.

Professional testers solve this with a written agreement. The **rules of engagement** are the agreement that states what a tester may attack, when and how. The list of systems that may be attacked is the **scope**. Everything that is not on that list is out of scope.

### The rules of this lab

1. **Attack only machines in your own lab.** In scope are the virtual machines you run for this course, on the lab network on your own laptop.
2. **Everything else is out of scope.** That includes the campus network, the Wi-Fi at home, devices of other students and every address on the internet.
3. **Read the target address before you press Enter.** Most accidents are a typing mistake in an IP address.
4. **Keep lab machines on the lab network.** Never connect a deliberately vulnerable machine to a network that other people use.
5. **Share techniques, not victims.** You may discuss what you learn. You do not test it on someone to prove a point.

In module 11 the scope grows by one network: that of the team assigned to you. Until then, rule 1 means your own laptop only.

::: warning
**Watch out:** breaking into a computer system without permission is a criminal offence in Belgium. That is also true when you only look around and change nothing. "It was for a course" is not permission.
:::

The legal side returns in depth in Penetration Testing. For this course the five rules are enough, and they are not optional.

[CONTROLEREN] Add a reference to the EhB policy on acceptable use of ICT, and state what happens when a student breaks these rules.

### Check yourself 0.2 – Rules of engagement

```text
Sectie: 0.2
Type: categoriseren
Vraag: Is the target in scope or out of scope for this lab?
Indeling:
  A virtual machine on the lab network on your laptop → In scope
  The Kali machine of the student next to you → Out of scope
  The router at your student room → Out of scope
  The public website of a company you find online → Out of scope
  The campus Wi-Fi → Out of scope
Feedback: Only the virtual machines in your own lab are in scope. Everything else is out of scope, also when you own the device or when the owner is a friend (see 0.2).

Sectie: 0.2
Type: meerkeuze
Vraag: You want to scan 10.20.30.10 in your lab, but you type 10.20.3.10 and press Enter. The scan starts. What do you do first? Choose one answer.
Opties:
  A. Stop the scan
  B. Let the scan finish and check afterwards whether the address exists
  C. Shut down your laptop
  D. Start a second scan on the right address
Juist: A
Feedback A: Correct. You stop first, because the address is out of scope. Then you check what happened and tell your lecturer (see 0.2).
Feedback B: Every packet you send to an address outside your lab is out of scope. You do not wait until the scan is done (see 0.2).
Feedback C: That stops the scan, but you also lose the terminal that shows what you sent. Stop the scan itself and keep the evidence (see 0.2).
Feedback D: The wrong scan keeps running while you start the next one. Stop it first (see 0.2).

Sectie: 0.2
Type: meerdere antwoorden
Vraag: A friend asks you to test the Wi-Fi of the café where she works. Which statements are correct? Choose all correct answers.
Opties:
  A. The café is out of scope for this course
  B. Your friend can give you permission, because she works there
  C. Without permission from the owner, the test is a criminal offence
  D. It is allowed as long as you change nothing
Juist: A, C
Feedback A: Correct. The scope of this course is your own lab (see 0.2).
Feedback B: An employee does not own the network. Only the owner can give permission, and you want it in writing (see 0.2).
Feedback C: Correct. Access without permission is an offence in Belgium (see 0.2).
Feedback D: Looking around without permission is also an offence, even when you change nothing (see 0.2).
```

## 0.3 Your hypervisor

You need several computers for this course: an attacker, a server, later a firewall. Buying them is one option. Running them as software on the laptop you already have is cheaper.

A **virtual machine (VM)** is a computer that exists only as software and runs inside another computer. The program that runs virtual machines is the **hypervisor**. Your laptop is the **host**, and each virtual machine on it is a **guest**.

In this course every student uses the same hypervisor: VirtualBox. It is free and it runs on Windows, Linux and macOS on Intel.

### Check that virtualisation is on

A hypervisor needs a processor feature called VT-x (Intel) or AMD-V (AMD). On most laptops it is on. Check it before you install anything.

On Windows:

1. Press Ctrl+Shift+Esc to open Task Manager.
2. Open the **Performance** tab and select **CPU**. You now see a line **Virtualization: Enabled**.

On Linux, run this command in a terminal:

```bash
lscpu | grep Virtualization
```

```output
Virtualization:                       VT-x
```

On an AMD processor you see `AMD-V`. On a Mac with an Intel processor the feature is always on.

Do you see **Disabled**, or no output at all? Go to "Not working?" at the end of this module.

### Install VirtualBox

1. Go to [virtualbox.org](https://www.virtualbox.org/wiki/Downloads) and download the package for your operating system.
2. Run the installer and keep the default options. On Windows the installer warns that your network connection drops for a moment. That is normal.
3. Start VirtualBox. You now see the VirtualBox Manager, with an empty list of machines.
4. Choose **Help > About VirtualBox**. You now see the version number. It starts with 7.2 or higher.

[CONTROLEREN] Menu labels in step 4 for VirtualBox 7.2 on each operating system.

What this means for you: when something breaks in the lab, the question is always "on the host or in the guest?". Get used to naming which one you mean.

## 0.4 Importing the lab

The lab starts small: one attacker and one server, connected by a network that leads nowhere else. You add machines in later modules.

| Machine | Role | Address | Network |
| --- | --- | --- | --- |
| `kali` | Your workstation as a tester | 10.20.30.50 | NAT and `halden-lab` |
| `halden-srv01` | A server at Halden & Co | 10.20.30.10 | `halden-lab` |

**Kali Linux** is a Linux distribution that comes with the tools security testers use. `halden-lab` is an **internal network**: a network that exists only inside VirtualBox, between the virtual machines that are attached to it.

Kali has a second connection, through NAT, so that it can download updates. The server has no way out.

### Add Kali

1. Go to [kali.org/get-kali](https://www.kali.org/get-kali/), choose **Virtual Machines** and download the 64-bit image for VirtualBox.
2. Extract the `.7z` file to a folder where it can stay for the whole semester. You now see a folder with a `.vbox` file and a `.vdi` file.
3. In VirtualBox, choose **Machine > Add** and select the `.vbox` file. You now see the Kali machine in the list.
4. Select the machine and open **Settings**. Switch the settings window to **Expert** mode.
5. Under **System**, set **Base Memory** to 4096 MB.
6. Under **Network**, open **Adapter 2**. Tick **Enable Network Adapter**, set **Attached to** to **Internal Network** and type `halden-lab` as the name.
7. Click **OK**. In the details of the machine you now see two adapters: NAT and Internal Network 'halden-lab'.

[CONTROLEREN] Labels in steps 3 to 7 for VirtualBox 7.2, including whether Expert mode is needed for Adapter 2.

::: warning
**Watch out:** the name of an internal network is case-sensitive. `halden-lab` and `Halden-Lab` are two different networks, and machines on them cannot see each other.
:::

### Import the server

1. Download `halden-srv01.ova` from the course files on Canvas.
2. In VirtualBox, choose **File > Import Appliance**, select the file and click **Finish**. You now see `halden-srv01` in the list.
3. Open **Settings > Network** for this machine. Check that **Adapter 1** is attached to **Internal Network** with the name `halden-lab`.
4. Start `halden-srv01`. After about a minute you see a login prompt. You do not log in: from now on you approach this machine over the network.

[CONTROLEREN] `halden-srv01.ova` does not exist yet. Steps 2 to 4 depend on how it is built.

### Give Kali an address on the lab network

Nothing hands out addresses on `halden-lab` yet, so you set the address of Kali by hand.

1. Start the Kali machine and log in with the user `kali` and the password `kali`.
2. Open a terminal and run the two commands below. Kali asks for your password at the first one.

```bash
sudo nmcli connection add type ethernet ifname eth1 con-name halden-lab ipv4.method manual ipv4.addresses 10.20.30.50/24 ipv6.method disabled
sudo nmcli connection up halden-lab
```

3. Check the result:

```bash
ip -brief address show eth1
```

```output
eth1             UP             10.20.30.50/24
```

[NIET GETEST] The `nmcli` commands and the interface name `eth1` on the current Kali image.

Copy the first command for now. You learn what an interface and a `/24` mean for a design in module 2 and module 3.

### Check that the lab is isolated

The server must be reachable from Kali and from nowhere else. Test the second half from the host. Open a terminal on your own laptop, not in Kali.

On Windows:

```bash
ping 10.20.30.10
```

On Linux or macOS:

```bash
ping -c 3 10.20.30.10
```

You now see timeouts or `100% packet loss`. A failed ping is the result you want here: your own laptop cannot reach the server.

What this means for you: an attack on 10.20.30.10 can only come from a machine on `halden-lab`. That is what makes it safe to break things there.

## 0.5 Snapshots and resets

In the coming weeks you break this lab on purpose, and sometimes by accident. Reinstalling a machine every time costs an evening. You need a way back that takes a minute.

A **snapshot** is a saved state of a virtual machine that you can return to later. It records the disk and the settings of the machine at that moment.

### Take a snapshot

1. Shut down the Kali machine: in Kali, choose **Shut Down** from the menu at the top right.
2. In the VirtualBox Manager, select the machine and open its **Snapshots** tool.
3. Click **Take**, name the snapshot `m0-clean` and click **OK**. You now see `m0-clean` in the list, with **Current State** under it.
4. Repeat steps 1 to 3 for `halden-srv01`. Use **Machine > ACPI Shutdown** to shut it down.

[CONTROLEREN] Where the Snapshots tool sits in VirtualBox 7.2, and the labels in steps 2 to 4.

Start the name of every snapshot with the module number. In week 9 you want to know which of your seven snapshots is the one from before the attack.

### Restore a snapshot

Try it once now, while nothing depends on it.

1. Start Kali, log in and create a file on the desktop:

```bash
touch ~/Desktop/break-me.txt
```

2. Shut down Kali.
3. In the **Snapshots** tool, select `m0-clean` and click **Restore**. Untick the option to create a snapshot of the current state, then confirm.
4. Start Kali and log in. The file `break-me.txt` is gone.

::: warning
**Watch out:** restoring a snapshot removes everything that happened in the machine after it. That includes your notes, screenshots and capture files. Keep the evidence for your portfolio on the host, never only inside a virtual machine.
:::

A snapshot is not a backup. It sits on the same disk as the machine, so a broken or lost laptop takes both.

### Check yourself 0.5 – Snapshots

```text
Sectie: 0.5
Type: meerkeuze
Vraag: You take the snapshot `m6-before-attack`. Afterwards you run an attack and save a capture file inside Kali. Then you restore `m6-before-attack`. What happens to the capture file? Choose one answer.
Opties:
  A. It is gone
  B. It is still in the same folder
  C. It moves to the host
  D. It stays, but it is empty
Juist: A
Feedback A: Correct. Restoring brings the disk back to the moment of the snapshot. Everything from after that moment is removed (see 0.5).
Feedback B: The disk returns to its state at the moment of the snapshot. The file did not exist then (see 0.5).
Feedback C: VirtualBox never copies files from a guest to the host on its own. Copy your evidence to the host yourself, before you restore (see 0.5).
Feedback D: A restore does not empty files. It returns the whole disk to the earlier state, so the file is not there at all (see 0.5).

Sectie: 0.5
Type: meerdere antwoorden
Vraag: Which statements about snapshots are correct? Choose all correct answers.
Opties:
  A. A snapshot lets you return a virtual machine to an earlier state
  B. A snapshot protects your work when your laptop is stolen
  C. A snapshot of Kali also saves the state of `halden-srv01`
  D. A clear name tells you later which state a snapshot holds
Juist: A, D
Feedback A: Correct. That is what a snapshot is for (see 0.5).
Feedback B: A snapshot sits on the same disk as the machine. It is not a backup (see 0.5).
Feedback C: A snapshot belongs to one machine. You take one for every machine you want to be able to reset (see 0.5).
Feedback D: Correct. Start the name with the module number (see 0.5).
```

## Final check

Your setup is ready when this one command works. Start `halden-srv01` and Kali, and run it in a terminal in Kali:

```bash
ping -c 3 10.20.30.10
```

```output
PING 10.20.30.10 (10.20.30.10) 56(84) bytes of data.
64 bytes from 10.20.30.10: icmp_seq=1 ttl=64 time=0.612 ms
64 bytes from 10.20.30.10: icmp_seq=2 ttl=64 time=0.498 ms
64 bytes from 10.20.30.10: icmp_seq=3 ttl=64 time=0.521 ms

--- 10.20.30.10 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2031ms
```

Your times differ. What counts is the last line: three packets sent, three received.

[NIET GETEST] The ping output, until `halden-srv01.ova` exists.

## Not working?

These are the problems we expect most often.

| What you see | Cause | What to do |
| --- | --- | --- |
| **Virtualization: Disabled** in Task Manager, or an error about VT-x or AMD-V when a machine starts | Virtualisation is switched off in the firmware of your laptop | Restart, open the BIOS or UEFI settings and switch on Intel VT-x or AMD-V (also called SVM). The key to get there differs per brand. |
| A machine starts but is very slow, and VirtualBox shows a turtle icon at the bottom right | Windows uses the processor feature for Hyper-V | In **Turn Windows features on or off**, untick **Hyper-V**, **Virtual Machine Platform** and **Windows Hypervisor Platform**. Then restart. |
| The installer on Windows asks for the Microsoft Visual C++ Redistributable | A component that VirtualBox needs is missing | Install the latest Visual C++ Redistributable from the Microsoft website and run the installer again. |
| `eth1` does not exist in Kali | Adapter 2 is not enabled | Shut down Kali and repeat step 6 of "Add Kali" in [0.4](#0_4). |
| The ping in the final check shows `Destination Host Unreachable` | The two machines are not on the same internal network, or Kali has no address | Compare the network name of both machines, letter by letter. Then check the address with `ip -brief address show eth1`. |
| The ping from the host in [0.4](#0_4) gets replies | Another device on your own network uses 10.20.30.10 | Your lab is still isolated. Tell your lecturer, so that you can agree on a different range. |
| Everything is slow as soon as two machines run | The host is out of memory | Close your browser and other heavy programs. Never run more machines than the module asks for. |

[CONTROLEREN] The Hyper-V row: names of the Windows features, and whether switching them off is acceptable on student laptops that use WSL or Docker Desktop.

## Summary

New terms in this module:

- **rules of engagement**: the agreement that states what a tester may attack, when and how;
- **scope**: the list of systems that may be attacked;
- **virtual machine (VM)**: a computer that exists only as software and runs inside another computer;
- **hypervisor**: the program that runs virtual machines;
- **host** and **guest**: your laptop, and a virtual machine on it;
- **Kali Linux**: a Linux distribution that comes with the tools security testers use;
- **internal network**: a network that exists only inside VirtualBox;
- **snapshot**: a saved state of a virtual machine that you can return to.

The rules to remember:

- Attack only machines in your own lab. Everything else is out of scope.
- Read the target address before you press Enter.
- Take a snapshot before you change or attack something, and start its name with the module number.
- Keep your evidence on the host.
