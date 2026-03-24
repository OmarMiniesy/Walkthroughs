
---

First thing I did was downloaded the `evtx` and converted them to CSV using [EvtxECmd](https://github.com/EricZimmerman/evtx).

```powershell
./Tools/EvtxECmd/EvtxECmd.exe -f ./Microsoft-Windows-NTFS.evtx --csv . --csvf "NTFS-csv"

./Tools/EvtxECmd/EvtxECmd.exe -f ./SECURITY.evtx --csv . --csvf "SECURITY-csv"

./Tools/EvtxECmd/EvtxECmd.exe -f ./SYSTEM.evtx --csv . --csvf "SYSTEM-csv"
```


We also have a `MFT` file which we can also handle using the [MFTECmd](https://github.com/EricZimmerman/MFTECmd) tool to convert it to a CSV file as well.
```PowerShell
./Tools/MFTECmd/MFTECmd.exe -f C/\$MFT --csv . --csvf MFT-CSV
```

---
### Task 1

The Event ID when a service enters a running state from a google search is `7036` and it is found in the `System` logs.

Opening the `SYSTEM` logs using Timeline Explorer:
```PowerShell
./Tools/TimelineExplorer/TimelineExplorer.exe ./SYSTEM-csv
```

We can then filter the `EventID` column for the `7036` ID.

![[CrownJewel-1,1.png|393]]

We can then look at `Payload Data1` column and we will see the Volume Shadow Copy service name.

![[CrownJewel-1,2.png]]

We see the time is `2024-05-14 03:42:16`, and that its status is currently `running`, therefore, it is the time it started.

> Attackers can abuse the vssadmin utility to create volume shadow snapshots and then extract sensitive files like NTDS.dit to bypass security mechanisms. Identify the time when the Volume Shadow Copy service entered a running state.: `2024-05-14 03:42:16`.

---
### Task 2

We need to look for an event where User groups are enumerated by the volume shadow copy process.
- Doing a quick google search for events that log enumeration of user groups, we see the Event IDs 4799 and 4798 belonging to the `SECURITY` log.

Filtering for these Event IDs in the Security log in Timeline Explorer, we see only the Event ID `4799`.
- Now, we have to identify the ones started by the process for Volume Shadow Copies, also called `VSSVC.exe`.


![[CrownJewel-1,3.png]]

We see that the ones we need are the ones in the middle, and the two groups that were enumerated are:
- `Backup Operators`
- `Administrators`

While the machine account responsible was the `DC01$` account.

> When a volume shadow snapshot is created, the Volume shadow copy service validates the privileges using the Machine account and enumerates User groups. Find the two user groups the volume shadow copy process queries and the machine account that did it.: `Administrators, Backup Operators, DC01$`

---
### Task 3

In the same Security log rows, we see the Caller Process ID but in hex, which we can then convert to decimal.

![[CrownJewel-1,4.png]]

The process ID is `0x1190` and converting it to decimal using any website we get the value `4496`.

> Identify the Process ID (in Decimal) of the volume shadow copy service process. `4496`.

---
### Task 4

For this one, I navigated to the `NTFS` log as I felt it was the most necessary, then, I grouped by all the Event IDs in Timeline Explorer, and sent a screenshot to Claude to tell me the description of each Event ID.

![[CrownJewel-1, 5.png]] 

![[CrownJewel-1, 6.png]]

From here, we can see that the none of these are related to volume mounts ! 
- Also, the view of the data is not friendly in Timeline Explorer, so I want to view it in a better format.
- Viewing the data in the Payload column, I see there are several nice columns like the `DeviceName` and the `DeviceGuid`.
- I want to extract the data from the event log file and obtain all instances of this data to try and figure out if I can extract something useful.

To be able to have proper visibility on all the data quickly, instead of using a SIEM, I used `chainsaw` to convert the given `evtx` file to JSON:
```
./chainsaw dump Microsoft-Windows-NTFS.evtx --json -o json-ntfs
```

Now, we can filter for the `DeviceName` fields, but first, we need to know where it exists in the JSON structure:
```
cat json-ntfs | jq '.[]' -c | grep "DeviceName"
```
- This command prints each event on one line to allow us to `grep` using the `-c` flag.
- I then `grep` for this field to see which events have it, and see how can we move forward.

The last event i get is:
```
{"Event_attributes":{"xmlns":"http://schemas.microsoft.com/win/2004/08/events/event"},"Event":{"System":{"Provider_attributes":{"Name":"Microsoft-Windows-Ntfs","Guid":"3FF37A1C-A68D-4D6E-8C9B-F79E8B16C482"},"EventID":303,"Version":1,"Level":4,"Task":8,"Opcode":2,"Keywords":"0x4000000000000020","TimeCreated_attributes":{"SystemTime":"2024-05-14T03:46:47.503852Z"},"EventRecordID":130,"Correlation":null,"Execution_attributes":{"ProcessID":1736,"ThreadID":1436},"Channel":"Microsoft-Windows-Ntfs/Operational","Computer":"DC01.forela.local","Security_attributes":{"UserID":"S-1-5-18"}},"EventData":{"VolumeCorrelationId":"06C4A997-CCA8-11ED-A90F-000C295644F9","VolumeIdLength":0,"VolumeId":"","VolumeLabelLength":0,"VolumeLabel":"","DeviceNameLength":33,"DeviceName":"\\Device\\HarddiskVolumeShadowCopy1","DeviceGuid":"00000000-0000-0000-0000-000000000000","VendorIdLength":0,"VendorId":"","ProductIdLength":0,"ProductId":"","ProductRevisionLength":0,"ProductRevision":"","DeviceSerialNumberLength":0,"DeviceSerialNumber":"","BusType":0,"AdapterSerialNumberLength":0,"AdapterSerialNumber":"","Vcb":"0xffff800f93fd41b0","ProcessId":1736,"ProcessName":"svchost.exe","DismountReason":"User request"}}}
```

Distinctly, this stands out because I see the `DeviceName` is `\\Device\\HarddiskVolumeShadowCopy1`, which is probably what we need.
- I note that EventID, `303`, and now time to visualize all the existing device names.

To do that, we need to understand the structure of the JSON file.
```
cat json-ntfs | jq '.[]' -c | grep "DeviceName" | jq .
```
- By simply adding the `jq .` to the end, we now return it in proper JSON format.

We note:
- `EventID` is at : `Event.System.EventID`
- `DeviceName` is at: `Event.EventData.DeviceName`
- `DeviceGuid` is at: `Event.EventData.DeviceGuid` its all 0s??
- `VolumeCorrelationId` is at: `Event.EventData.VolumeCorrelationId`

![[CrownJewel-1, 7.png]]

Now, we can see all of the `DeviceName`s present, and locate the `DeviceName` we need, and then use that to get its `DeviceGuid`.

```
cat json-ntfs | jq '.[]' | jq '.Event.EventData.DeviceName' | sort | uniq -c | sort -n
```

We get this output:

![[CrownJewel-1, 8.png]]

We see that the interesting device name is the first one, the `ShadowCopy1`.
- Now, we need to see which EventIDs had this device name, to understand the attacker flow and what happened.

We know from the question that the attacker mounted this volume as apparent by the question, but let's see it in action.

To extract all events with this volume name:
```
cat json-ntfs | jq '.[] | select(.Event.EventData.DeviceName == "\\Device\\HarddiskVolumeShadowCopy1")' | jq '.Event.System.EventID'
```

I did this to identify which events contained this volume of ours in question:

![[CrownJewel-1, 9.png]]

I then pasted the output of the 5 events to Claude understand what happened:

| Time            | EID     | What happened                                                           |
| --------------- | ------- | ----------------------------------------------------------------------- |
| 03:44:22.812931 | **4**   | Shadow copy volume **dismounted** — stats logged                        |
| 03:44:22.813035 | **9**   | Volume **surprise-removed** (unexpected disconnect)                     |
| 03:44:22.813045 | **10**  | Transaction log (`$LogFile`) cache info logged                          |
| 03:46:47.502329 | **300** | Self-healing / dismount initiated by `svchost.exe` — **"User request"** |
| 03:46:47.503852 | **303** | Dismount progress/completion — same process                             |

We see that it shows the *dismount process*, not the mounting itself.
- We then figure out that the mounting Event ID is in the SYSTEM log, `98`.
- However, there is no mounting done after the timestamp of running of the volume shadow copy service, and there is no mounting at all for this volume name. weird.

Ok, time to step back. We need to figure out which volume is the right one !
- We already have these other volumes present which we found above by running this query:

```
cat json-ntfs | jq '.[]' | jq '.Event.EventData.DeviceName' | sort | uniq -c | sort -n
```

We get this output:

![[CrownJewel-1, 8.png]]

We also know that the attacker started the volume shadow copy service at `2024-05-14 03:42:16`.

We can check for each volume the timestamps of the event IDs that happened for them using this command:

- For `harddiskvolume4`:
```
cat json-ntfs | jq -c '.[] | select(.Event.EventData.DeviceName == "\\Device\\HarddiskVolume4") | {EventID
: .Event.System.EventID, Time: .Event.System.TimeCreated_attributes.SystemTime}'
```

![[CrownJewel-1, 11.png]]

None of the events take place after our timestamp, so this volume is ruled out.

- For `harddisvolume3`:
```
cat json-ntfs | jq -c '.[] | select(.Event.EventData.DeviceName == "\\Device\\HarddiskVolume3") | {EventID: .Event.System.EventID, Time: .Event.System.TimeCreated_attributes.SystemTime}'
```

![[CrownJewel-1, 12.png]]

The same goes for this volume. Doing this for the `HarddiskVolumeShadowCopy1`, we get convincing event times!!

![[CrownJewel-1, 13.png]]

This is our culprit.
- We need to obtain its ID.
- Removing the final pipe command to print the event ID and the timestamp, we can look for its ID.

We found on top the following keys:
- `DeviceGuid` is at: `Event.EventData.DeviceGuid` its all 0s??
- `VolumeCorrelationId` is at: `Event.EventData.VolumeCorrelationId`

So i think the answer will be for the correlation id.

```
cat json-ntfs | jq -c '.[] | select(.Event.EventData.DeviceName == "\\Device\\HarddiskVolume3") | .Event.EventData.VolumeCorrelationId'
```

We get the value of `17A28535-1E81-4F9F-8B4A-85BB7474B0C9`.

> Find the assigned Volume ID/GUID value to the Shadow copy snapshot when it was mounted.: `{17A28535-1E81-4F9F-8B4A-85BB7474B0C9}`.

---
### Task 5

To identify disk related artifacts, we need to parse the MFT.
- We already converted it to csv using the MFTEcmd tool, we need to open it in Timeline Explorer.

Now, we need to look for a dump of the `NTDS.dit` database.
- we can easily do that by looking for files present with the `.dit` extension.

![[CrownJewel-1, 13-1.png]]

By filtering on the `Extension` column for `.dit` values, we get this odd file:
```
.\Users\Administrator\Documents\backup_sync_dc\ntds.dit
```

We know that this MFT was from the `C` drive as was present in the artifacts folder in the Sherlock, so we can construct the path to begin with `C:\`.

> Identify the full path of the dumped NTDS database on disk.: `C:\Users\Administrator\Documents\backup_sync_dc\ntds.dit`

---
### Task 6

Check the `Created0x10` column to get the value: `2024-05-14 03:44:22`

> When was newly dumped ntds.dit created on disk?: `2024-05-14 03:44:22`

---
### Task 7

I see that the dump of the NTDS database was in the `backup_sync_dc` folder.
- We can scan the contents of this folder and see if the registry was dumped alongside it.

By adding a filter in the Parent Path column:

![[CrownJewel-1, f.png]]

We see:

![[CrownJewel-1, x.png]]

The SYSTEM hive was dumped with a size of `17563648`.

> A registry hive was also dumped alongside the NTDS database. Which registry hive was dumped and what is its file size in bytes?: `SYSTEM, 17563648`.

---
