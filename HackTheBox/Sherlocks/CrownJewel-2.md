


---

Downloading the files and converting them to CSV using Eric Zimmerman's EvtxECmd.
```
../../Tools/EvtxECmd/EvtxECmd.exe -f SYSTEM.evtx --csv . --csvf "sys-csv"
../../Tools/EvtxECmd/EvtxECmd.exe -f SECURITY.evtx --csv . --csvf "sec-csv"
../../Tools/EvtxECmd/EvtxECmd.exe -f APPLICATION.evtx --csv . --csvf "app-csv"
```
- These files can then be loaded into Timeline Explorer for grouping and easier navigation.

For the `APPLICATION` logs, the output is not nice, as the payload columns are all empty, so I will convert it as is to JSON using Chainsaw to be able to navigate easily and read data properly.
```
../../Tools/chainsaw/target/release/chainsaw dump APPLICATION.evtx --json -o ./json-app
```

First thing I did was obtain all of the Event IDs in all of the event logs and paste them into Claude to get a brief overview of the types of events logged in each file.

---
### Task 1

For the Microsoft Shadow Copy Service to enter a running state, an event will be logged with this exact timestamp.
 - We can find this in the System log under Event ID `7036`. Adding a filter in Timeline Explorer for that EventID to only show these events.
- Add a filter in the `Payload Data1` column for services that have the `Shadow` string.

![[CrownJewel-2, a.png]]

We see that the first time it was ran it stopped 3 minutes later, and the second time, it ran without stopping.
- The time here is `2024-05-15 05:39:55`.

> When utilizing ntdsutil.exe to dump NTDS on disk, it simultaneously employs the Microsoft Shadow Copy Service. What is the most recent timestamp at which this service entered the running state, signifying the possible initiation of the NTDS dumping process?: `2024-05-15 05:39:55`.

---
### Task 2

To find the dumped NTDS file, we need to trace the attacker's steps.
- We know they have a domain admin account in their hands, so we need to find out its logon session ID and trace its activity.
- We also know they are using `ntdsutil` to dump the database.

We can search in the security logs for the process `ntdsutil` and see how it was created and if it did any security-related actions.
- Searching for `ntdsutil` in the security logs, we get 118 events with Event ID `4799` for security group enumeration.

We see that the `ntdsutil` process was used to enumerate the Administrators and Backup Operators groups in the exact time  the volume shadow copy service starts - `05:39:55`.
- We also see the `SubjectLogonId` -> `0x8DE3D`.
- We also see the Username and user SID of the compromised account: `FORELA\Administrator (S-1-5-21-3239415629-1862073780-2394361899-500)`
- This is the logon session of the attacker which we can use to trace with their actions.
- We also see the process ID of the `ntdsutil` process is -> `0xF64`.

![[CrownJewel-2, b.png]]

![[CrownJewel-2, c.png]]

Now, searching in the security logs for the attacker's `SubjectLogonId` to see their activity, we get this:

![[CrownJewel-2, d.png]]

We see that this activity takes place before the volume shadow copy service runs, so we can assume this is how the attacker managed to obtain the credentials of the compromised domain admin account.
- We see the persistence mechanism in place by the attacker (scheduled task).
- Tracing the event in line 60, we see credentials were returned for the exact SID we have found to belong to the attacker.

I jumped to the application logs we have open in Timeline Explorer and hit a `CTRL+F` for `ntds` to see if any of the logs have anything useful about it.
- I also added a time filter on the Time Created column to see the events logged on the day the shadow service was created to view activity relevant to the attack.

![[CrownJewel-2, e.png]]

And we hit a gold mine!

![[CrownJewel-2, f.png]]

From these event IDs, the useful ones are the ones i have open.
- By the way, I grouped by Event ID by dragging it to the top to be able to see this view.

Claude gives a quick summary of the event logs we see:
- `216` - **Database location changed** — AD detected ntds.dit was accessed from a non-standard path;
- `325` - **Database created at path** — logs the exact file path of the newly created ntds.dit;
- `326` and `327` are database opened and closed.

Therefore, we see in the `325` EventID row in the exact same timestamp of the service starting, after by 1 second, the location of the new dumped database : `C:\Windows\Temp\dump_tmp\Active Directory\ntds.dit`.

> Identify the full path of the dumped NTDS file.: `C:\Windows\Temp\dump_tmp\Active Directory\ntds.dit`.

---
### Task 3

We saw this and commented on it, `2024-05-15 05:39:56`, saying how it is super close to the volume starting, meaning its highly relevant.

> When was the database dump created on the disk?: `2024-05-15 05:39:56`.

---
### Task 4

Looking at the screenshot we used to answer question 3, we can see event ID `327` which means the database engine detached the database, and so released its handle on the database. 
- That's the point at which the file is no longer locked by the engine and is considered a complete, usable copy.

We note the time here is `2024-05-15 05:39:58`.

> When was the newly dumped database considered complete and ready for use? :`2024-05-15 05:39:58`

---
### Task 5

I usually hide the `provider` column in Timeline Explorer as it takes up space and I rarely need it, but it contains the answer to this question:

![[CrownJewel-2, g.png]]

We see here the provider is `ESENT`.

> Event logs use event sources to track events coming from different sources. Which event source provides database status data like creation and detachment? `ESENT`.

---
### Task 6

We found this before while tracking the actions of the attacker in Task 3.
- But since it is stated here outright, we can determine that the Event ID needed for group enumeration is `4799`.
- Filtering the security logs in Timeline Explorer to `4799` and looking only at the events where the process responsible if `ntdsutil.exe`, we get the answer:

![[CrownJewel-2, h.png]]

![[CrownJewel-2, i.png]]

We see that the 2 groups are:
- Administrators
- Backup Operators

> When ntdsutil.exe is used to dump the database, it enumerates certain user groups to validate the privileges of the account being used. Which two groups are enumerated by the ntdsutil.exe process? Give the groups in alphabetical order joined by comma space.: `Administrators, Backup Operators`

---
### Task 7

We captured the `SubjectLogonId` as `0x8DE3D` above, and we can also capture it by opening the field of `SubjectLogonId` in any of the fields of the group enumeration events we saw in the previous task.

Now, we need to find the first time it logged in, so we can simply CTRL F for it in the Security Logs and focus on the event IDs that are related to logon activity.
- I see no events related to logon in the security logs.

So I went to the system logs, and found that there is an Event ID `7001` for user logon and it only captures the `UserSID`.
- Copying the attacker's user SID from the same event for the group enumeration, we see it is: `S-1-5-21-3239415629-1862073780-2394361899-500`

CTRL+F in the system logs, we see 1 record show up:

![[CrownJewel-2, j.png]]

Showing the logon time of : `2024-05-15 05:36:31`.

> Now you are tasked to find the Login Time for the malicious Session. Using the Logon ID, find the Time when the user logon session started. : `2024-05-15 05:36:31`.

---
