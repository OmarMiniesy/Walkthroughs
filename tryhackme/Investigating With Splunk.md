---

---

---

Visiting the IP address and opening the Splunk interface.
- To check how many events are present in the `main` index, we search for that index and specify the time range of `all time`.

![](./screenshots/splunk-1.png)

###### How many events were collected and Ingested in the index **main**? : `12256`

Next, we need to find the user that was created and on which host.
- Looking at the fields tab, we see that for all the events, there is only 1 host, therefore, we know where the user was created.

![](./screenshots/splunk-2.png)

Now that we have the host, we need to look and see when and how a user was created.
- I thought to look through the `CommandLine` field to see if any command was done to add a new user. (not the best approach).

![](./screenshots/splunk-3.png)

We see that a command was executed to add a user called `A1berto`.

The right way to do this was to first check the `SourceType`, which we see is only 1 in the fields tab, which is `eventlogs`.
- Therefore, the logs we have are searchable using windows event logs event IDs.

Searching online for the windows event log ID for user creation, [event ids](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/), we see that the user creation ID is `4720`.

![](./screenshots/splunk-4.png)

We can filter our search in Splunk using this value for the `EventID` field.

![](./screenshots/splunk-5.png)

We see that 1 event is returned, Investigating that event, we see the following:

![](./screenshots/splunk-6.png)

###### On one of the infected hosts, the adversary was successful in creating a backdoor user. What is the new username? : `A1berto`

To tackle the next point, I started to search for Windows Event Log Event IDs for registry modifications but to no avail.
- However, there is a Sysmon EventID for registry modifications, which is 13.

Searching for that ID, we see 1143 events. Not good.
- To narrow down our search, I added `A1berto` in the search, and voila, we get 1 hit.

![](./screenshots/Splunk-7.png)

###### On the same host a registry key was also updated regarding the new backdoor user. What is the full path of that registry key? : `HKLM\SAM\SAM\Domains\Account\Users\Names\A1berto`

To answer the next question, we need to find the user that is being impersonated.
- Taking a look at all the users present in the logs, I found both `A1berto` and `Alberto`, where the first is the one the attacker is creating.
- Hence, the second user with the correct spelling is the one being impersonated.

###### Examine the logs and identify the user that the adversary was trying to impersonate : `Alberto`

