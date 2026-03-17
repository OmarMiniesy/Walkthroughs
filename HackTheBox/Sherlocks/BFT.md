
---
### Setup

First thing to do is to convert the `$MFT` file in the `C` directory of the downloaded zip to a csv file using the [MFTECmd](https://download.ericzimmermanstools.com/net9/MFTECmd.zip) tool.

```
.\MFTECmd.exe -f '$MFT' --csv . --csvf "csv-mft"
```
- The `-f` flag is used to specify the path to the `$MFT` input file.
- The `--csv` flag is used to specify the output directory.
- The `--csvf` flag is used to specify the output csv file.

Now, we can open Timeline Explorer, also by Eric Zimmerman, and open the output `csv-mft` file in it to begin analysis.

I also downloaded the [HxD](https://mh-nexus.de/en/downloads.php?product=HxD20) hex editor to be used later.

---
### Task 1

We need to look for a `.zip` file that was created in the timeframe of February 13.
- We can do that by filtering the `Extension` column on `.zip`
- Focusing our attention to the entries that pop up on this day.

![[BFT-1.png]]

We see that we get 4 files only that have a parent path related to the `simon.stark` user.
- Checking the file names, we see that we are only interested in `invoices.zip` and `Stage-20240213T093324Z-001.zip`.
- So which one is the one downloaded?

![[BFT-2.png]]

From here, we can see that the `Stage` file came before at `16:34`, and that the `invoices.zip` file comes after it by approx 1 hour, and is located inside the `Stage` folder.
- This points out that the `stage.zip` file came before and *contains* inside it the `invoices.zip`.

> Simon Stark was targeted by attackers on February 13. He downloaded a ZIP file from a link received in an email. What was the name of the ZIP file he downloaded from the link?: `Stage-20240213T093324Z-001.zip`.

---
### Task 2

The Zone Identifier is an alternate data stream (ADS) that is used to signify how the file came to be on the host.
- It contains information about the IP and links it came from, which can be from the internet or from the local machine.

> The Zone Identifier data is located in `filename:Zone.Identifier` file, which in our case will be: `Stage-20240213T093324Z-001.zip:Zone.Identifier`.

To look for this, we can directly search for this filename in the text search:

![[BFT-3.png]]

We see the `HostUrl` that the download comes from.

> Examine the Zone Identifier contents for the initially downloaded ZIP file. This field reveals the HostUrl from where the file was downloaded, serving as a valuable Indicator of Compromise (IOC) in our investigation/analysis. What is the full Host URL from where this ZIP file was downloaded? : `https://storage.googleapis.com/drive-bulk-export-anonymous/20240213T093324.039Z/4133399871716478688/a40aecd0-1cf3-4f88-b55a-e188d5c1c04f/1/c277a8b4-afa9-4d34-b8ca-e1eb5e5f983c?authuser`

---
### Task 3

Since a malicious zip file was downloaded, we can assume that it contains some other files that probably do some bad stuff.
- We can check the timestamp of when that malicious file was downloaded, and then browse for files that have been created that have a parent path containing the `Stage-20240213T093324Z-001` folder.

We identified the creation time of the `stage` zip file as `16:34`.
- Adding a filter for the parent path to contain the `stage` folder and looking at the files that were created around the stated timestamp should prove useful.

![[BFT-4.png]]

We see a `.bat` file found in the `stage` folder under the `\invoice\invoices\` folder, which stands out because `.bat` file is used to contain executable instructions.
- We also know that the drive used is the `C` drive as the `$MFT` file was found inside the `C` drive in the downloaded question zip file for the sherlock.

> What is the full path and name of the malicious file that executed malicious code and connected to a C2 server?: `C:\Users\simon.stark\Downloads\Stage-20240213T093324Z-001\Stage\invoice\invoices\invoice.bat`

### Task 4

![[BFT-6.png]]

> Analyze the $Created0x30 timestamp for the previously identified file. When was this file created on disk?: `2024-02-13 16:38:39`.

###  Task 5

To get the *hex offset* of a file in the MFT record, we have to locate its **Entry Number** in the MFT.
- The entry number is just its index.
- Since each entry has a size of 1024 bytes, we need to multiply the index we get with 1024 to get its actual address.
- This number is the byte address of that MFT record. We simply need to convert it to hex using a hex converter to get the hex offset.

![[BFT-5.png]]

We have the entry number of `23436`.
- Multiply by 1024 to get `23998464`.
- Convert to hex to `0x16E3000`.

> Finding the hex offset of an MFT record is beneficial in many investigative scenarios. Find the hex offset of the stager file from Question 3. `16E3000`.

### Task 6

This file has a size of `286`, hence, it is stored directly in the MFT table, and is called a resident file.

![[BFT-7.png]]

However, this is not visible in timeline explorer.
- We need to utilize a hex viewer to view the actual `$MFT` table and locate this file using the hex address we found earlier of `16E3000`.

Opening the HxD tool and pressing `CTRL+G` to jump to our address of `16E3000`, we can see the entry for our file.

![[BFT-8.png]]

We see our file and we see that it communicates using `HTTP` to `43.204.110.203:6666`.

> Each MFT record is 1024 bytes in size. If a file on disk has smaller size than 1024 bytes, they can be stored directly on MFT File itself. These are called MFT Resident files. During Windows File system Investigation, its crucial to look for any malicious/suspicious files that may be resident in MFT. This way we can find contents of malicious files/scripts. Find the contents of The malicious stager identified in Question3 and answer with the C2 IP and port : `43.204.110.203:6666`.

---
