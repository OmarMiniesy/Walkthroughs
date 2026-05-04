
---
### Initial Analysis

Before starting the analysis, I converted the `evtx` files to CSV to be easily parsed by Timeline Explorer using the EvtxECmd tool.

```
../../Tools/EvtxECmd/EvtxECmd.exe -f ./Microsoft-Windows-Powershell.evtx --csv . --csvf "powershell-csv"
```

We see that the events present in the `powershell` logs are:

![[Phantom Check, 1.png]]
- `400` - Start of PowerShell session
- `403` - End of PowerShell session
- `600` - PowerShell provider started.
- `800` - Execution details, commands run. 


```
 ../../Tools/EvtxECmd/EvtxECmd.exe -f ./Windows-Powershell-Operational.evtx --csv . --csvf "operational-csv"
```

![[Phantom Check, 2.png]]

The interesting `operational` logs are:
- `4103` - Contains the execution output and the commands.
- `4104` - Script Block Logging. 

---
### Task 1 & 2

By going through the `operational` logs in TimeLine Explorer and filtering by Event ID, we skim through the `4103` events.
- We are looking for anything related to `WMI`

![[Phantom Check, 3.png]]

We see there are multiple `Get-WmiObject` commands being executed.
- Adding a filter on `Payload Data1` to be equal to that value to analyze in details:

![[Phantom Check, 4.png]]

Analyzing the `Payload Data6` column, we see:

![[Phantom Check, 5.png]]

- `Win32_ComputerSystem`
- `SELECT * FROM MSAcpi_ThermalZoneTemperature`

Inspecting PayloadData6 thoroughly, we see that the first WMI class is used to obtain the Manufacturer, and the second is used to obtain the current temperature of the machine.
- These are known malware anti-reverse engineering techniques to detect for virtual machines.
- Check out these [notes](https://github.com/OmarMiniesy/Notes/blob/master/Defensive%20Security/Malware%20Analysis/Anti-Reverse%20Engineering.md).

> Which WMI class did the attacker use to retrieve model and manufacturer information for virtualization detection?: `Win32_ComputerSystem`.

> Which WMI query did the attacker execute to retrieve the current temperature value of the machine?: `SELECT * FROM MSAcpi_ThermalZoneTemperature`

---
### Task 3

Going through Event ID `4103`, nothing speaks out, however, note that the events we found earlier for the WMI class took place around `2025-04-09 09:19 and 09:20`
- Jumping to Event ID `4104`, we see that actual scripts being run and its code.
- Looking for the `function` keyword and jumping to the same time range.

![[Phantom Check, 6.png]]

We see that only 1 event exists, and we see the function `Check-VM`, which is pretty suspicious.

> The attacker loaded a PowerShell script to detect virtualization. What is the function name of the script?: `Check-VM`.

---
### Task 4

Going through the script, we see that it is detecting several types of Virtualization throughout the script.

Scrolling through, we see:

![[Phantom Check. 7.png]]

> Which registry key did the above script query to retrieve service details for virtualization detection?: `HKLM:\SYSTEM\ControlSet001\Services`.

---
### Task 5

Scrolling to the VirtualBox part of the script, we see:

![[Phantom Check, 8.png]]

> The VM detection script can also identify VirtualBox. Which processes is it comparing to determine if the system is running VirtualBox?: `vboxservice.exe, vboxtray.exe`

---
### Task 6

I remember seeing this in the `4013` events.
- Skimming through again in `PayloadData 6` we see 1 event that stands out:

![[Phantom Check, 9-1.png]]

Opening it, we see

![[Phantom Check, 10.png]]

> The VM detection script prints any detection with the prefix 'This is a'. Which two virtualization platforms did the script detect?: `hyper-v, vmware`.

---


