
---
### Overview

Downloading the files, we see several `evtx` files that can be used:

![[LogJammer 1.png]]

---
### Task 1

Looking for successful logons, means we need to look for Event ID `4624` in the `Security.evtx` file.
- Using the `chainsaw` tool with the `search` functionality and JSON parsing we can find the login times:

```bash
./chainsaw search "4624" ./Security.evtx --json \
| jq -c .[] \
| grep CyberJunkie \
| jq '.Event.System.TimeCreated_attributes.SystemTime'
```

- The first line searches in the Security file for the `4624` event ID which is for successful logins and outputs in `json`.
- The second line simply formats the JSON output such that each event is written in 1 line, making `grep` easier.
- The third line is grepping for the `CyberJunkie` user.
- The fourth line returns the timestamp needed.

![[LogJammer, 2.png]]

From here we can see that the time of first login for that user is `27/03/2023 14:37:09`

> When did the cyberjunkie user first successfully log into his computer? (UTC): `27/03/2023 14:37:09`.

---
### Task 2

To understand the firewall logs, i ran the below command using Chainsaw to figure out the event IDs present in the `Windows Firewall-Firewall.evtx`.

```
./chainsaw search "" ./Windows\ Firewall-Firewall.evtx --json 2> /dev/null \
| jq '.[] | .Event.System.EventID' \
| sort | uniq -c | sort -rn
```

- The first line grabs everything in the file by matching on an empty string and returns it as JSON, and throwing any noise to `/dev/null`
- The second line returns only the Event IDs
- The third line sorts and counts the Event IDs.

![[LogJammer, 3.png]]

We see there are the following events:
- `2004` - A firewall rule being added <- what we need. 
- `2006` - A firewall rule being deleted.
- `2005` - A firewall rule being modified. 

We can now filter on the `2004` Event ID and observe.

```
../../Tools/chainsaw/target/release/chainsaw search "2004" ./Windows\ Firewall-Firewall.evtx --json | jq .[]
```
- Skimming through, we see that the user name is not recorded, but we see there is a `Event.EventData.ModifyingUser`, and it has the SID of the user.

We need to obtain the SID of the CyberJunkie user when they first logged in.
- Running the same command we used above to obtain the Time of login, we simply change the field we need to be that of the target user SID.

```
../../Tools/chainsaw/target/release/chainsaw search "4624" ./Security.evtx --json | jq -c .[] | grep CyberJunkie | jq '.Event.EventData.TargetUserSid'
```

![[LogJammer, 4.png]]

We see the SID is:
```
S-1-5-21-3393683511-3463148672-371912004-1001
```

We can now use this while looking for the events in the firewall logs.

```
 ../../Tools/chainsaw/target/release/chainsaw search "2004" ./Windows\ Firewall-Firewall.evtx --json | jq .[] | jq -c | grep S-1-5-21-3393683511-3463148672-371912004-1001 | jq .
```
- Looking at the events, we see that we are mainly interested with the `RuleName` field which can be found under the `Event.EventData.RuleName`.

Fixing the query above to return the rule names, we see:

```
../../Tools/chainsaw/target/release/chainsaw search "2004" ./Windows\ Firewall-Firewall.evtx --json | jq .[] | jq -c | grep S-1-5-21-3393683511-3463148672-371912004-1001 | jq '.Event.EventData.RuleName' | sort | uniq -c | sort -rn
```

![[LogJammer, 5.png]]

Notice the obviously malicious firewall rule being added.

> The user tampered with firewall settings on the system. Analyze the firewall event logs to find out the Name of the firewall rule added?: `Metasploit C2 Bypass`.

---
### Task 3

To identify more details about this specific rule, we can filter on it:

```
../../Tools/chainsaw/target/release/chainsaw search "2004" ./Windows\ Firewall-Firewall.evtx --json | jq '.[] | select(.Event.EventData.RuleName == "Metasploit C2 Bypass")'
```
- The `select` statement can be used to match a field with a specific value.

![[LogJammer, 6.png]]

We see that the Direction is 2, which is equivalent to *outbound*.

> Whats the direction of the firewall rule?: `outbound`.

---
### Task 4

Changing the audit policy is logged using Event ID `4719` and is visible in the `security.evtx` log file.
- Running `chainsaw` to look for this Event ID:

```
../../Tools/chainsaw/target/release/chainsaw search "4719" ./Security.evtx --json \
| jq .[]
```

![[LogJammer, 6-1.png]]

Opening the Microsoft documentation for the `4719` Event ID, we [see](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4719) that we can identify the subcategory using the `SubcategoryGuid` field.
- `0CCE9227-69AE-11D9-BED3-505054503030`.

To identify its value, we can run this command on `cmd` or PowerShell as given in the documentation and `grep` on the value we have.
```
auditpol /list /subcategory:* /v | Select-String 0CCE9227-69AE-11D9-BED3-505054503030
```

![[LogJammer, 7.png]]

We get the following subcategory.

> The user changed audit policy of the computer. Whats the Subcategory of this changed policy?: `Other Object Access Events`.

---
### Task 5

To look for scheduled task creations, we can look for Event ID `4698` in the `security.evtx` file.

```
../../Tools/chainsaw/target/release/chainsaw search "4698" ./Security.evtx --json | jq .[]
```

Running this, we get the output:

![[LogJammer, 8.png]]

Scanning through, we see the user `CyberJunkie` is responsible and we see that it has the name `HTB-AUTOMATION`.

> The user "cyberjunkie" created a scheduled task. Whats the name of this task?: `HTB-AUTOMATION`.

---
### Task 6 & 7

Further inspecting the `TaskContent` in the above screenshot, we see 

![[LogJammer, 9.png]]

The file 

```
C:\Users\CyberJunkie\Desktop\Automation-HTB.ps1
```

being run with the arguments of

![[LogJammer, 10.png]]

```
-A cyberjunkie@hackthebox.eu
```

> Whats the full path of the file which was scheduled for the task?: `C:\Users\CyberJunkie\Desktop\Automation-HTB.ps1`.

> What are the arguments of the command?: `-A cyberjunkie@hackthebox.eu`.

---
### Task 8 & 9

Heading over to the `windows defender` logs and running the command we ran earlier to identify the Event IDs present:

```
 ../../Tools/chainsaw/target/release/chainsaw search "" ./Windows\ Defender-Operational.evtx --json 2> /dev/null | jq '.[] | .Event.System.EventID' | sort | uniq -c | sort -rn
```

![[LogJammer, 11.png]]

Doing some research, we identify that Event IDs:
- `1116` - malware detected.
- `1117` - defender took actions against malware.

Trying the first the `1116` Event ID by using the following query to identify the relevant data.

```
../../Tools/chainsaw/target/release/chainsaw search "1116" ./Windows\ Defender-Operational.evtx --json | jq .[]
```

From here, notice that the `Event.EventData.Path` field contains the path of the file that was detected by the antivirus.

Running the following query to identify the exact file names and what was detected throughout, we see:

```
../../Tools/chainsaw/target/release/chainsaw search "1116" ./Windows\ Defender-Operational.evtx --json | jq '.[].Event.EventData.Path'
```

![[LogJammer, 12.png]]

We directly spot `SharpHound`. the tool used for recon.

We can also see the file and its location:
```
C:\Users\CyberJunkie\Downloads\SharpHound-v1.1.0.zip
```


> The antivirus running on the system identified a threat and performed actions on it. Which tool was identified as malware by antivirus?: `Sharphound`.

> Whats the full path of the malware which raised the alert?: `C:\Users\CyberJunkie\Downloads\SharpHound-v1.1.0.zip`

---
### Task 10

To identify the action taken, we can now filter for the `1117` Event ID and grep for `SharpHound` and look at the events that are returned.

```
../../Tools/chainsaw/target/release/chainsaw search "1117" ./Windows\ Defender-Operational.evtx --json | jq .[] -c | grep SharpHound | jq .
```

Skimming through the results, we see they are only 2 events, and we can spot the action taken in the `Action Name` field.

![[LogJammer, 13.png]]

> What action was taken by the antivirus?: `Quarantine`.

---
### Task 11

To view the powershell events, we can now use the `powershell operational` log file.

```
 ../../Tools/chainsaw/target/release/chainsaw search "" ./Powershell-Operational.evtx --json 2> /dev/null | jq '.[] | .Event.System.EventID' | sort | uniq -c | sort -rn
```

![[LogJammer, 14.png]]

We know that Event IDs `4104` and `4103` are used to log commands and script blocks.
- Checking `4103` for the commands run.

Get several results, trying to filter them by grepping on the `CyberJunkie` user then on the `Automation-HTB.ps1` file to narrow down the results, we get:

```
../../Tools/chainsaw/target/release/chainsaw search "4103" ./Powershell-Operational.evtx --json | jq -c | grep CyberJunkie | jq -c '.[].Event.EventData.Payload' | grep Automation-HTB | jq .
```

![[LogJammer, 15.png]]

From here, we see many commands, including:
- `Resolve-Path`
- `Test-Path`
- `GetStreamHash`
- `Get-FileHash`, which is the one that stands out as it deals with the `HTB-Automation.ps1` file.

Going over to the `4014` event ID and identifying the needed field to be `Event.EventData.ScriptBlockText` and looking for the `Get-FileHash` command

```
../../Tools/chainsaw/target/release/chainsaw search "4104" ./Powershell-Operational.evtx --json | jq .[] -c | grep Get-FileHash
```

![[LogJammer, 16.png]]

We see the command that was executed in full.

```
Get-FileHash -Algorithm md5 .\Desktop\Automation-HTB.ps1
```

> The user used Powershell to execute commands. What command was executed by the user?: `Get-FileHash -Algorithm md5 .\Desktop\Automation-HTB.ps1`.

---
### Task 12

doing some research, the following Event Ids are important:
- `1102` for the `security` log wipes
- `104` for the `system` log wipes

For the security log, the following is output:
```
../../Tools/chainsaw/target/release/chainsaw search 1102 ./Security.evtx --skip-e
rrors --json | jq .
```

![[LogJammer, 17.png]]
- We see that a log file was cleared by the `CyberJunkie` user, but we don't see the log file name.
- We also note down the timestamp is `2023-03-27T14:36:45.307731Z`.

For the system log, there are a lot of events, so we can filter based on the timestamp as well

```
../../Tools/chainsaw/target/release/chainsaw search "" ./System.evtx --skip-errors --json \
  | jq '.[] | select(
      .Event.System.EventID == 104
      and
      .Event.System.TimeCreated_attributes.SystemTime >= "2023-03-27T14:36:45.307731Z"
    )'
```

This returns exactly one event, that of the firewall logs, which match with the changes made to the firewall rules.

![[LogJammer, 20.png]]


> We suspect the user deleted some event logs. Which Event log file was cleared?: `Microsoft-Windows-Windows Firewall With Advanced Security/Firewall`.

---
