
#### Ransomware

After the excitement of yesterday, Alice has started to settle into her new job. Sadly, she realizes her new colleagues may not be the crack cybersecurity team that she was led to believe before she joined. Looking through her incident ticketing queue she notices a “critical” ticket that was never addressed. Shaking her head, she begins to investigate. Apparently on August 24th Bob Smith (using a Windows 10 workstation named we8105desk) came back to his desk after working-out and found his speakers blaring (click below to listen), his desktop image changed (see below) and his files inaccessible.

Alice has seen this before... ransomware. After a quick conversation with Bob, Alice determines that Bob found a USB drive in the parking lot earlier in the day, plugged it into his desktop, and opened up a word document on the USB drive called "Miranda_Tate_unveiled.dotm". With a resigned sigh she begins to dig into the problem...

Ransomware warning: https://assets.ctfassets.net/v6zg15is9ny1/6CCvnndD4tmwub6yVorZ7r/78b2d6d11651779080b758d8c7594d91/cerber-sample-voice.mp3

Alices journal: https://assets.ctfassets.net/v6zg15is9ny1/61mwyKjugG9TCfJFFu42iV/b207b79bcd52c33ae6067ad51022b2c4/Alice-Journal.html.pdf

Mission document: https://assets.ctfassets.net/v6zg15is9ny1/1Rr1oPQ0JHwBsNFKqNNUPJ/560deb3f9b6fe236bc977a34de0d8c73/mission_document.html.pdf

| #   | Question                                                                                                                                                                                                                                     |
| --- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 200 | What was the most likely IP address of we8105desk on 24AUG2016?                                                                                                                                                                              |
| 201 | Amongst the Suricata signatures that detected the Cerber malware, which one alerted the fewest number of times? Submit ONLY the signature ID value as the answer. (No punctuation, just 7 integers.)                                         |
| 202 | What fully qualified domain name (FQDN) does the Cerber ransomware attempt to direct the user to at the end of its encryption phase?                                                                                                         |
| 203 | What was the first suspicious domain visited by we8105desk on 24AUG2016?                                                                                                                                                                     |
| 204 | During the initial Cerber infection a VB script is run. The entire script from this execution, pre-pended by the name of the launching .exe, can be found in a field in Splunk. What is the length in characters of the value of this field? |
| 205 | What is the name of the USB key inserted by Bob Smith?                                                                                                                                                                                       |
| 206 | Bob Smith's workstation (we8105desk) was connected to a file server during the ransomware outbreak. What is the IP address of the file server?                                                                                               |
| 207 | How many distinct PDFs did the ransomware encrypt on the remote file server?                                                                                                                                                                 |
| 208 | The VBscript found in question 204 launches 121214.tmp. What is the ParentProcessId of this initial launch?                                                                                                                                  |
| 209 | The Cerber ransomware encrypts files located in Bob Smith's Windows profile. How many .txt files does it encrypt?                                                                                                                            |
| 210 | The malware downloads a file that contains the Cerber ransomware cryptor code. What is the name of that file?                                                                                                                                |
| 211 | Now that you know the name of the ransomware's encryptor file, what obfuscation technique does it likely use?                                                                                                                                |

