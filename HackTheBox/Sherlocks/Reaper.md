
---
### Initial Analysis of PCAP

First thing I did was open the `pcap` file using Wireshark, then I enabled name resolution for the network address:

![Reaper-1.png](Reaper-2.png)

> That way, we can determine if Wireshark mapped the names of the machines with their IP addresses by inspecting the packets present.

Opening the Statistics tab, then choosing endpoints, and navigating to IPv4, we can now observe the machines and their IP addresses:

![Reaper-2.png](./pics/Reaper-2.png)

I sorted by the *Packets* tab, and clicked on the left checkbox for Name resolution, we can see that we see the first 2 IP addresses and their names:
- `172.17.79.4 : dc01.forela.local` -> This is the Domain Controller
- `172.17.79.129 : Forela-Wkstn001.forela.local` -> This is the interesting workstation
- `172.17.79.135 : D` -> This name is weird, lets keep it in note.

![Reaper-3.png|350](./pics/Reaper-3.png)![Reaper-4.png|300](./pics/Reaper-4.png)

Next, I opened the *Protocol Hierarchy* tab under *Statistics* as well to check out the type of communication present in the `pcap`:

![Reaper-04.png](./pics/Reaper-04.png)