
---
### Initial Analysis of PCAP

First thing I did was open the `pcap` file using Wireshark, then I enabled name resolution for the network address:

![Reaper-1.png](./pics/Reaper-2.png)

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

> We see that mainly SMB traffic is present as well as ARP traffic.

Opening the PCAP now, we see that the first packets we see are all ARP packets being sent out by the IP address `172.17.79.135` as if trying to discover the network, is sent to the entire IP address network range.
- This is the machine whose name is `D`.
- We get a few responses, mainly for the Forela-Wkstn001 we discovered, as well as the domain controller, and a few other virtual machines.

##### Assessing `NBNS` - NetBIOS

We also see scattered throughout the protocol `NBNS` protocol sending out *Refresh* packets.
- Doing a quick search, Refresh packets are used to ensure that the computer maintains the same name-IP mapping.
- This is a protocol that serves a similar function to DNS, by mapping IP addresses to machine names.
- From here, we can confirm the following intuition we found earlier from Wireshark about the computer names and their IP addresses.

Opening a packet from these and expanding the NBNS section, we see the following:

![[Reaper-5.png]]

This shows that the name of the machine `FORELA-WKSTN-001` is refreshing its IP address of `172.17.79.129`. We see the following IPS:
- `172.17.79.129` -> `FORELA-WKSTN-001`
- `172.17.79.136` -> `FORELA-WKSTN-002`
- `172.17.79.1` -> `PK-DEV-JSTARK`
- `172.17.79.4` -> `DC`

We also notice how in the `NBNS` packets, a few them are sent out by the `172.17.79.135` machine we though was suspicious for discovering the network, which had the name `D`.
- We see that it sends out a few `NBNS` query packets to destination IPs, as if to determine their information:

![[Reaper-6.png]]

If we open the first packet for example, we see that it sends a NBNS query to the `.1` IP address:
- This query is of type `NBSTAT` which is a legitimate query type to retrieve the NetBIOS names registered on a host.
- The query name consisting of the `*<00><00>...` is the NetBIOS wildcard name which returns the status of all the NetBIOS names registered on the host.
- This means that the `.135` machine is asking the `.1` machine about all of the NetBIOS names it has and their status, which adds to the discovery of the network.

![[Reaper-8.png]]

The response from the `.1` IP address is then:

![[Reaper-7.png]]

This shows the name of the machine being `PK-DEV-JSTARK`, is part of a local windows `WORKGROUP`, and has the *SMB* service as shown by the `<20>` suffix. 

> This is the case for the other machines it has reconned as shown in this table:

| Hostname & IP     | IP            | Domain / Workgroup | NetBIOS Names Returned                                                                | Roles / Services Identified                                                                                                      |
| ----------------- | ------------- | ------------------ | ------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| **PK-DEV-JSTARK** | 172.17.79.1   | WORKGROUP          | PK-DEV-JSTARK `<00>`  <br>WORKGROUP `<00>`  <br>PK-DEV-JSTARK `<20>`                  | `<00>` Workstation service  <br>`<20>` SMB Server service                                                                        |
| **WKSTN001**      | 172.17.79.129 | FORELA             | FORELA `<00>`  <br>WKSTN001 `<00>`  <br>WKSTN001 `<20>`                               | `<00>` Workstation service  <br>`<20>` SMB Server service                                                                        |
| **DC01**          | 172.17.79.4   | FORELA             | DC01 `<00>`  <br>FORELA `<00>`  <br>FORELA `<1C>`  <br>DC01 `<20>`  <br>FORELA `<1B>` | `<00>` Workstation service  <br>`<20>` SMB Server service  <br>`<1C>` Domain Controllers group  <br>`<1B>` Domain Master Browser |

Ok, i got a bit carried away. I think its time to answer the questions lol.

---
### Task 1

> What is the IP Address for Forela-Wkstn001?: `172.17.79.129`

---
### Task 2

> What is the IP Address for Forela-Wkstn002?: `172.17.79.136`

---
### Task 3

Knowing that the suspicious IP is this one `172.17.79.135` due to its weird activities and its name and not being involved in the domain, we can assume that it is the attacker machine.

Now, we can look through the `pcap` at the *SMB* conversations and look for our IP address.

![[Reaper-10.png]]

We see here that the suspicious IP tries to connect to both `.136` and `.129` with the `FORELA\arthur.kyle` username, and then we scroll down a bit, we see both IPs are communicating via SMB with the `.135` machine, indicating the connection works.
- Hence, the attacker has access to this user and is using it to connect to different workstations (WKSTN-1) and (WKSTN-2) and reading data.

![[Reaper-11.png]]

> What is the username of the account whose hash was stolen by attacker? `arthur.kyle`.

We can then navigate to the `evtx` file to check if the `.135` managed to successfully login using the `arthur.kyle` to either machines.
- We can do that by filtering for the `4624` Event ID in Event Viewer, and navigating through the login events.

![[Reaper-12-1.png]]

Scrolling through the logon events, we see one that stands out from all other events by having a different *Security ID* of *NULL SID*.
- Going through it more we see that the *Account Name* is `arthur.kyle` and the Source Network Address is `172.17.79.135`, which is our attacker machine.
- However, what's weird, is that the name of the workstation being used is `FORELA-WKSTN002`, and we know that it has a different IP address, that being `172.17.79.136`.

![[Reaper-13.png]]

> We also notice that the method of login is *NTLM*, which is different than the other methods 

Doing a quick research, we can reach the following conclusion:
- This is an indication that the credentials for the `arthur.kyle` user was stolen, and at attempt at authentication took place with a spoofed or incorrect workstation name.
- This takes place when hacking tools are used.
- The credentials were stolen and then used them to authenticate using *NTLM*.
- We also see that it is *logon type 3*, meaning it is a network type logon, and is usually the case in lateral movement techniques with stolen credentials.

But how where the credentials stolen? I'm going to take a step back.

---
### Stealing the Creds

From what we know from the event log, we know that the stolen user is `arthur.kyle`.
- But from where was this account stolen?
- How did the `.135` get into contact and obtain that user? From which workstation?

Going through the `pcap`, we see that the `.136` sends out DNS requests to the DC at `.4` asking for a domain that does not exist...... `D`
- So what does the `.135` attacker machine do, it responds when `LLMNR` protocol is used, and spoofs a response, claiming to be the machine with name `D`. ***AHAAAAA***.
- This is the attack that takes place here [[Noxious]].

![[Reaper-14.png]]

So now, the `.136` machine, our victim, thinks that `.135` is the nonexistent domain it is searching for, meaning the attacker now acts as a man in the middle.
- So when the `.136` tries to authenticate with the `.135` machine, the attacker manages to steal the credentials.

If we dive deeper, we see here that the victim machine, `.136` tries to then authenticate to the `.135` machine, by requesting a TGS from the `.4` machine, our Domain Controller.
- However, this fails! Because the `.135` machine, whose service name is `D`, doesn't actually exist!

![[Reaper-15.png]]

- We see the `tgs-req` being sent, and then just the packet after it, we see an error, that the principal name is unknown.
- If we expand the TGS-REQ packet, we will see the service being requested is D, and this doesn't exist in the directory.

So then, windows defaults to NTLM authentication afterwards, and this is where the attacker manages to capture the credentials of our victim user `arthur.kyle` present on the `.136` machine!!!

| Packet | Direction | Message                                      | Meaning                           |
| ------ | --------- | -------------------------------------------- | --------------------------------- |
| 1195   | 136 → 135 | **Session Setup Request, NTLMSSP_NEGOTIATE** | Client begins NTLM authentication |
| 1196   | 135 → 136 | **NTLMSSP_CHALLENGE**                        | Attacker sends NTLM challenge     |
| 1197   | 136 → 135 | **NTLMSSP_AUTH, User: FORELA\arthur.kyle**   | Client sends NTLM authentication  |
| 1198   | 135 → 136 | Session Setup Response                       | Authentication completed          |
| 1199   | 136 → 135 | Tree Connect `\\D\IPC$`                      | SMB share access                  |
In packet `1197`, when the client sends the NTLM authentication, the attacker manages to lay his hands on the credentials of the `arthur.kyle` user.

Right after that, we see the attacker then try to connect to the `.129` machine with the same user!!!
- LATERAL MOVEMENT !

---
### Task 4

> What is the IP Address of Unknown Device used by the attacker to intercept credentials? `172.17.79.135`

---
### Task 5

We now know the victim is the machine with IP ending in `.136`.
- Filtering for this IP.

After following the `pcap`, we can then see that the `.136` manages to then connect with the `.4` Domain Controller, and then requests access to the `\DC01\Trip` file on the DC.
- The attacker is not present here.

![[Reaper-16.png]]

> What was the fileshare navigated by the victim user account?: `\\DC01\Trip`.

---
### Task 6

Opening the log we found earlier, we can see the source port present in the log.

> What is the source port used to logon to target workstation using the compromised account?: `40252`.

---
### Task 7

Also in the log we found earlier.

> What is the Logon ID for the malicious session?: `0x64A799`.

---
### Task 8 

We reached this conclusion above, where there was a mismatch between the IP and the actual name on that IP. Found in the log.
- `172.17.79.135`
- `FORELA-WKSTN002`

> The detection was based on the mismatch of hostname and the assigned IP Address. What is the workstation name and the source IP Address from which the malicious logon occur?: `FORELA-WKSTN002, 172.17.79.135`.

---
### Task 9

This can be found in the *details* section of the log under the *TimeCreated* record.

> At what UTC time did the the malicious logon happen?: `2024-07-31 04:55:16`

---
### Task 10

I went to the event log and filtered for event log `5140` which represents file share access.

![[Reaper-16-1.png]]

> What is the share Name accessed as part of the authentication process by the malicious tool used by the attacker?: `\\*\IPC$`

---
