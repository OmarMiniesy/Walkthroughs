
---
### Setup

I wanted to use Eric Zimmerman's tools to solve this challenge, so I used [EvtxECmd](https://github.com/EricZimmerman/evtx) to parse the `.evtx` file provided and Timeline Explorer to go through the output.
- Both of these tools can be downloaded from [here](ericzimmerman.github.io/#!index.md).

So, running `EvtxECmd` on the file we have using this command:
```
.\EvtxECmd.exe -f .\Security.evtx --csv . --csvf ".\csv-security"
```
- This basically indicates the input file using `-f` and where to output the csv file, by specifying directory using `--csv` and the filename using `--csvf`.

We get the following output which shows a summary of the Event IDs present and their counts, along with our output file that we can now parse using Timeline Explorer.

![[cmp-2.png]]

Opening timeline explorer, i then dragged the **Event ID** column up so that we group by this column, making our analysis easier, so we can quickly jump and locate what type of event we need.
- We can later drop this column again if we need to analyze by sequence of events or time after locating something interesting.

![[Campfire-2, 1.png]]

> I think now, after doing this quick setup, we can start analyzing the Sherlock

### Analysis

##### Understanding AS-REProasting 

This Sherlock is concerned with the AS-REProasting attack, where:
- An attacker targets a user account that **does not** have *pre-authentication* enabled. 
- As a result, the attacker does not need to prove their identity when performing an *AS-REQ* to the KDC.
- The attacker does not need to provide an encrypted timestamp in the `pA-ENC-TIMESTAMP` field in the *AS-REQ*.
- The KDC responds with *AS-REP* that contains a part encrypted with the target user's pasword.
- The attacker then cracks this password.

##### Detecting AS-REProasting

To detect this, we need to look for the following:
- Event ID `4768` which represents the *AS-REQ*, or a request for a *TGT*.
- Filter more for where `PreAuthType` is `0` to indicate no pre-authentication is needed.
- Filter more for `ticket encryption type`s that are crackable, like `0x17` which is for `RC4-HMAC`.

### Task 1

To locate when the attacker requested the Kerberos ticket for the vulnerable user, we need to look for the `4768` Event ID logs.
- We can do this easily since we grouped by Event ID in timeline explorer:

![[Campfire-2, 2.png]]

We see there are only 11 logs, what we can do next, is then look for hints of our attack based on the criteria mentioned [[#Detecting AS-REProasting|Above]].
- Scrolling through we see one row with timestamp `2024-05-29 06:36:40` that stands out as it has `RC4-HMAC` encryption AND it has pre-authentication disabled. 
	- Check out Payload Data4 column and Payload Data6 column.
- Going to the last column and viewing the Payload, we see:

![[Campfire-2, 3-1.png]]

Therefore, we can say that this is the event where the TGT was requested to perform the AS-REProasting attack.

> When did the ASREP Roasting attack occur, and when did the attacker request the Kerberos ticket for the vulnerable user? : `2024-05-29 06:36:40`

### Task 2

Using the screenshot above, we see the value for the `TargetUserName` field is `arthur.kyle`.

> Please confirm the User Account that was targeted by the attacker: `arthur.kyle`.

### Task 3

> What was the SID of the account?: `S-1-5-21-3239415629-1862073780-2394361899-1601`

### Task 4

We can locate the IP address of the machine that was used to request the TGT also from the same screenshot.

> It is crucial to identify the compromised user account and the workstation responsible for this attack. Please list the internal IP address of the compromised asset to assist our threat-hunting team: `172.17.79.129`

### Task 5

Now that we know a TGT was obtained from the above IP, we need to check if this ticket was then used again.
- Since the KDC returns this ticket back to the IP that requested it, we can simply search for the above IP address in our logs.

We see that the next 4 logs after the TGT request are done by that same IP, and we see a user name in the User Name column:

 ![[Campfire-2, 4.png]]

If we open the event in line `75` that is of a TGS being requested, we see that the `TargetUserName` field, is actually representative of the user account that requests this `TGS`.
- Connecting the dots, we see that the attacker performed AS-REP roasting against the account **arthur.kyle** to obtain a crackable AS-REP response. 
- By correlating Kerberos activity from the same source IP, we can identify the user logged into the attacking workstation.

![[Campfire-2, 5.png]]

Studying this further, we see the attacker is requesting a TGS for the Domain Controller  !!
- The user `happy.grunwald` is requesting a TGS for the Domain Controller and it works! 
- The events following this one showcase that network share objects were accessed, also by the same IP address.

> We do not have any artifacts from the source machine yet. Using the same DC Security logs, can you confirm the user account used to perform the ASREP Roasting attack so we can contain the compromised account/s? : `happy.grunwald`.

---
### Summary 

- The attacker was operating from the workstation **172.17.79.129**, which generated the Kerberos activity seen in the domain controller logs.
    
- From this machine, the attacker performed **AS-REP Roasting** by requesting a Kerberos authentication ticket (**Event ID 4768**) for the account **arthur.kyle**, which had **Kerberos pre-authentication disabled**.
    
- Because pre-authentication was disabled, the domain controller returned an **AS-REP encrypted with the user’s password hash**, allowing the attacker to capture data that could be **cracked offline to recover the user’s password**.
    
- By correlating events from the same source IP address, we observed **Event ID 4769** shortly after, showing normal Kerberos activity originating from the same machine.
    
- This event revealed that **happy.grunwald** was the user account logged into the attacking workstation at the time of the attack.
    
- Therefore, **arthur.kyle** was the **victim account targeted for AS-REP roasting**, while **happy.grunwald** was the **compromised account used by the attacker to perform the attack from workstation 172.17.79.129**.

---
