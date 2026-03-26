
---
### Initial Analysis

```
index=botsv1 | stats count by sourcetype
```

![[Scenario 1 - Walkthrough, 1.png]]

Giving this screenshot to GPT to understand the type of data present.

 **Windows / Endpoint Telemetry**
- **`WinRegistry`**  
    Windows Registry data (keys/values).  
    **Use:** Persistence (Run keys, Services), config changes, malware artifacts.
- **`XmlWinEventLog:Microsoft-Windows-Sysmon/Operational`**  
    Sysmon logs (process, network, file, registry, injection, etc.).  
    **Use:** Core endpoint telemetry. Process creation (Event ID 1), network (3), file create (11), registry (13), injection (8).
- `wineventlog`
    Has windows logs collected from the host for Security ,System, Application, and more. 

**Firewall / Network Security (FortiGate)**
- **`fgt_event`**  
    FortiGate system/security events (logins, config changes, IPS alerts).  
    **Use:** Admin activity, security alerts, device-level events.
- **`fgt_traffic`**  
    Traffic logs (allowed/blocked connections).  
    **Use:** Network flows, lateral movement, C2 traffic, exfil paths.
- **`fgt_utm`**  
    Unified Threat Management logs (AV, IPS, web filtering).  
    **Use:** Detected threats, malware hits, exploit attempts.

**Web / Application**
- **`iis`**  
    Microsoft IIS web server logs.  
    **Use:** Web exploitation (e.g., webshells, CVEs), suspicious URIs, attacker IPs.

**Vulnerability Scanning**
- **`nessus:scan`**  
    Nessus scan results.  
    **Use:** Known vulnerabilities, attack surface context (not attacker activity itself).

**Zeek (Bro) Network Telemetry (`stream:*`)**
These are structured network logs (deep visibility into protocols):
- **`stream:dns`**  
    DNS queries/responses.  
    **Use:** Domain lookups, C2 domains, DNS tunneling.
- **`stream:http`**  
    HTTP requests/responses.  
    **Use:** Web activity, payload delivery, user agents, URIs.
- **`stream:dhcp`**  
    DHCP assignments.  
    **Use:** IP ↔ MAC mapping, device tracking.
- **`stream:icmp`**  
    ICMP traffic.  
    **Use:** Ping sweeps, network discovery.
- **`stream:ip`**  
    Raw IP-level metadata.  
    **Use:** General traffic visibility (less detailed than protocol logs).
- **`stream:ldap`**  
    LDAP queries.  
    **Use:** AD enumeration, recon, privilege discovery.
- **`stream:mapi`**  
    Exchange MAPI traffic.  
    **Use:** Mail access patterns, potential Exchange abuse.
- **`stream:sip`**  
    VoIP signaling.  
    **Use:** Rarely used unless VoIP attack or abuse.
- **`stream:smb`**  
    SMB file sharing activity.  
    **Use:** Lateral movement, file transfers, admin shares.
- **`stream:snmp`**  
    SNMP traffic.  
    **Use:** Network device enumeration.
- **`stream:tcp`**  
    TCP connection metadata.  
    **Use:** Connection-level tracking (who talked to whom).

**Network Security Monitoring**
- **`suricata`**  
    IDS/IPS alerts from Suricata.  
    **Use:** Signature-based detections (exploits, malware traffic, scans).

**Infrastructure / General Logs**
- **`syslog`**  
    Generic logs from network devices, Linux systems, etc.  
    **Use:** Broad—depends on source (auth logs, system events, network gear).


---
### 101. What is the likely IP address of someone from the Po1s0n1vy group scanning imreallynotbatman.com for web application vulnerabilities?

Scanning a website for vulnerabilities means we need to look at the following source types:
- `suricata` -> Might trigger an alert when scanning activity is detected
- `stream:http` -> See the HTTP requests sent to the `imnotreallybatman.com` site

Trying the `suricata` source type, I wrote this query to show the types of alerts triggered and by whom:
```
index=botsv1 sourcetype="suricata" event_type=alert
| stats values(alert.signature), count by src_ip, dest_ip
```
- `event_type=alert` to return only the alerts.
- `values(alert.signature)` to showcase the type of alert
- `count by src_ip, dest_ip` to see the alerts resulted based on which source IP that triggered it and to whom the request was for.

We see mainly 3 IPs that resulted in alerts:
- `192.168.2.50`
- `192.168.250.100`
- `40.80.148.42` -> This one has so many alerts and CVEs and has scanning done by `Acunetix` as well.

![[Scenario 1 - Walkthrough, a.png]]

Now, we need to know which one was targeting the `imnotreallybatman.com` website.
- We need to figure out the IP address of the `imnotreallbatman.com` site.

There are many ways we can prove this, we can jump to the `http` stream and filter for these IP addresses, and check the requests URL and the destination IP, and then map them.
```
index=botsv1 sourcetype="stream:http" batman
| stats values(url), count by src_ip, dest_ip
```
- Here, we filter for all HTTP requests that contain `batman` in it.
- Then, we display the `URL` and see the source and destination IPs.

![[Scenario 1 - Walkthrough, b.png]]

From here, it is obvious, that the `40.80.148.42` sends so many requests to the `192.168.250.70` IP address and to the `imnotreallybatman.com` website.
- Therefore, we know that the IP address of `imnotreallybatman.com` is `192.168.250.70`.

Now, we know from the Suricata query that the `40.80.148.42` IP triggered a lot of alerts when sending requests to the `.70` IP, and we know that the `.70` IP is that of our website.

> If we also take a look at the `http_user_agent`, the `form_data`, and the `uri`, we see malicious payloads being sent from that `40.80` IP.

Scanning behavior also showcases multiple source ports to avoid being blocked by firewalls, which we can prove here as well: by showing the source port of these requests.
```
index=botsv1 sourcetype="stream:http" src_ip=40.80.148.42 dest_ip=192.168.250.70
| stats values(src_port)
```
- We see that the attacker used ports from `49204 - 49501`, all directed to the `dest_port` of `80`.

###### **ANSWER**: `40.80.148.42`

---
### 102.What company created the web vulnerability scanner used by Po1s0n1vy? Type the company name. (For example "Microsoft" or "Oracle")

We can obtain the name usually from the `user_agent` header, however, it wasn't that easy:
```
index=botsv1 sourcetype="stream:http" src_ip=40.80.148.42 dest_ip=192.168.250.70
|stats values(http_user_agent)
```

![[Scenario 1 - Walkthrough, c.png]]

We can refer to the Suricata alerts which we retrieved earlier, as they should contain the name of the software causing the scans.
- This time, we can focus for only alerts that stem from our identified IP address.
- Moreover, we can also look for `scan` related alerts by adding the `scan` keyword and hoping it hits.

```
index=botsv1 sourcetype="suricata" src_ip=40.80.148.42 dest_ip=192.168.250.70 scan
|stats values(alert.signature)
```

![[Scenario 1 - Walkthrough, d.png]]

We see here the name `Acunetix`, after doing a quick search, it is the one we are looking for.
- We can also remove the `scan` keyword and see the types of vulnerabilities that were alerted on. These match with the payloads present in the HTTP requests being sent.

###### **ANSWER**: `Acunetix`.

---
### 103. What content management system is imreallynotbatman.com likely using? (Please do not include punctuation such as . , ! ? in your answer. We are looking for alpha characters only.)

I saw earlier while investigating `joomla` a lot, and it is a content management system.
- However, we can confirm this again by running a query to see the URLs being accessed on the `imnotreallybatman.com` website.

```
index=botsv1 sourcetype="stream:http" src_ip=40.80.148.42 dest_ip=192.168.250.70
| table _time, url, uri, request
```

![[Scenario 1 - Walkthrough, d-1.png]]

We see that `joomla` is being accessed on our internal `250.70` IP and on our `imnotreallybatman.com` website.

###### **ANSWER**: `joomla`.

---
### 104. What is the name of the file that defaced the imreallynotbatman.com website? Please submit only the name of the file with extension (For example "notepad.exe" or "favicon.ico")

Took me a while and many failed attempts.. However, while searching for `POST` requests in the `stream:http` sourcetype, i noticed something.
- I was searching for `POST` requests to our victim server, `.250.70`, that would contain files uploaded there, and hence, get my answer.

```
index=botsv1 sourcetype=stream:http http_method=POST dest = 192.168.250.70
| stats values(url), values(form_data), count by c_ip
```

Running this, expecting to see the `40.80.142` IP address (I did ), I also saw the `23.33.63.114` IP address sending requests with many usernames and passwords. 
- 412 attempts to be exact.

![[Scenario 1 - Walkthrough, e.png]]

So now, another IP address is in our roster.
- We need to find our more about this IP address.

```
index=botsv1 sourcetype=stream:http 23.22.63.114
```

Sifting through the fields on the left:

We see that there are events where the server is the one acting as the client and sending requests to our malicious machine?
- This is weird. Why would our server initiate connections to that server? This hints at the possibility of a web shell being present, or C2 taking place.
- However, they were only 3 requests. So we can easily investigate.
![[Scenario 1 - Walkthrough, f.png]]

Adding this field as an extra filter:
```
index=botsv1 sourcetype=stream:http 23.22.63.114 c_ip="192.168.250.70"
```

We see by sifting through the fields on the left:
- Destination port of `1337` - weird.
- A `uri` of `poisonivy-is-coming-for-you-batman.jpeg`.

![[Scenario 1 - Walkthrough, f-1.png]]

We see that it is sending a `GET` request to this website: `prankglassinebracket.jumpingcrab.com` at port `1337` and sending for this image.

If we look at the first event, we see it is a huge event, possibly carrying the image we saw and having these fields:

![[Scenario 1 - Walkthrough, g.png]]
- We see that the destination server is `SimpleHTTP/0.6 Python` .

Now that we have *fetched* this image and it is downloaded at our server, how did this happen?
- How did the server execute a GET request to download this image from this malicious IP address which was seen attempting to login to the admin page with so many credentials? Possibly a brute force attack?
- I noted the time of this request being sent is `3:19:11 8/10/16`. So we are looking for events that happened before that time where it resulted in the server sending out a GET request.

> Spent some time but it was tough. I figured out a process running `3791.exe`, traced its parent to the `w3ws` process, which is the webserver. I then saw that it spawned multiple malicous child processes that execute commands via php -> there was a webshell or some sort of RCE present. 

> So i traced how the `3791.exe` file was uploaded through stream:http, and saw a post request by `40.80.` IP. Saw that it managed to also view the contents of the directory. I then filtered on the cookie it used, and saw that this cookie had access to the admin page on the website. The first request with this cookie was a post with one of the credentials from the brute force, hence, they managed to find the right creds.

> Then, i saw some playing around with joomla plugins and uploading files and executing commands, i think they exploited some cve on joomla after finding admin access that allows them RCE.

> Need to go over this again, draft it out, and understand it properly. IT GETS UNRAVELED IN THE COMING QUESTIONS.

###### **ANSWER**: `poisonivy-is-coming-for-you-batman.jpeg`.

---
### 105: This attack used dynamic DNS to resolve to the malicious IP. What fully qualified domain name (FQDN) is associated with this attack?

The malicious IP here is the one needed to fetch the image we found - `23.22.63.114`, hence, the domain is the one that has the request sent to: `prankglassinebracket.jumpingcrab.com`.

We can confirm that by checking the `stream:dns` sourcetype and viewing the query & response for our IP address.
```
index=botsv1 sourcetype="stream:dns" 23.22.63.114
```

![[Scenario 1 - Walkthrough, h.png]]

###### **ANSWER**: `prankglassinebracket.jumpingcrab.com`

---
### 106: What IP address has Po1s0n1vy tied to domains that are pre-staged to attack Wayne Enterprises?

This is the IP address we have been dealing with: `23.22.63.114`.

###### **ANSWER**: `23.22.63.114`.

---
### 107: Based on the data gathered from this attack and common open source intelligence sources for domain names, what is the email address that is most likely associated with Po1s0n1vy APT group?


---
### 108: What IP address is likely attempting a brute force password attack against imreallynotbatman.com?

```
index=botsv1 sourcetype=stream:http http_method=POST dest = 192.168.250.70
| stats values(url), values(form_data), count by c_ip
```

We saw that earlier: `23.22.63.114`

---
### 109: What is the name of the executable uploaded by Po1s0n1vy? Please include file extension. (For example, "notepad.exe" or "favicon.ico")

We found it earlier, it is the `3791.exe`, but let's look for it again.

The executable uploaded -> We need to look for `POST` requests in the `stream:http`.
- We can then filter for the malicious IPs we already know:
	- `40.80.148.42`
	- `23.22.63.114`

But this got so many events.

We need to change the approach.
- Since it is an executable, we can guess that it was executed on the server.
- We know that the server has an IP address of `192.168.250.70`.

We can open the `sysmon` logs and check for `EventCode` 1, and look for executables that were run that seem suspicious and in suspicious directories.

```
index=botsv1 sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational EventCode=1
| stats values(cmdline), values(host), count by ParentImage, Image
| sort + count
```
- This query obtains for each parent and process, the command line that was executed by the process and on which host this took place.
- We then sort by ascending order to view the things that were executed the least amount of times first.
- We see a lot of suspicious stuff, but we then stumble upon an image: `C:\inetput\wwroot\joomla\3791.exe` and on the `we1149srv` host.

![[Scenario 1 - Walkthrough, i.png]]

This is most definitely the file we are looking for, but we need to confirm first that this host is the one that has the IP of `192.168.250.70` because this is our webserver.
- We can deduce it is from looking at the path of the image, which says it is the webserver.
- We can also deduce it by looking at Event ID 3 for network connections, and checking the source IP for events that take place on the `we1149srv` host.

```
index=botsv1 sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational EventCode=3 Computer="we1149srv.waynecorpinc.local"
```

![[Scenario 1 - Walkthrough, j.png]]

Therefore, this file we found is on the webserver.
- We can then move to the `http:stream` sourcetype and confirm that the `3791.exe` was indeed uploaded there by our attacker.

```
index=botsv1 sourcetype=stream:http 3791
```

We see 2 events returned, and we see:

![[Scenario 1 - Walkthrough, k.png|166]] 

![[Scenario 1 - Walkthrough, l.png|166]]


Confirming the attacker and our server.

Looking at the second event with time `8/10/16 2:52:47.035 PM`, we see:

![[Scenario 1 - Walkthrough, m.png]]

Our file being uploaded, and the magic bytes `MZ` confirming it is an executable.
- I also noted down some values from this event that I think are useful:

- `Cookie: 7598a3465c906161e060ac551a9e0276=9qfk2654t4rmhltilkfhe7ua23` - Can be used to trace this session and how they managed to upload the file.
- `Referer: http://imreallynotbatman.com/joomla/administrator/index.php?option=com_extplorer&tmpl=component` - Looks like they managed to get access to the admin page and abused something there related to `com_extplorer` or something.

> We need to trace 

###### **ANSWER:** `3791.exe`

---
### 110: What is the MD5 hash of the executable uploaded?

To obtain that, we need to go to the event where it was executed, and it should contain the hash there in one of the `EventCode 1` fields.

```
index=botsv1 sourcetype=XmlWinEventLog:Microsoft-Windows-Sysmon/Operational EventCode=1 Image=*3791*
```

We get 1 event as output, and viewing the `hashes` field on the left:

![[Scenario 1 - Walkthrough, n.png]]

We get the MD5: `AAE3F5A29935E6ABCC2C2754D12A9AF0`

###### **ANSWER**: `AAE3F5A29935E6ABCC2C2754D12A9AF0`

---
### 111: GCPD reported that common TTPs (Tactics, Techniques, Procedures) for the Po1s0n1vy APT group, if initial compromise fails, is to send a spear phishing email with custom malware attached to their intended target. This malware is usually connected to Po1s0n1vys initial attack infrastructure. Using research techniques, provide the SHA256 hash of this malware.

