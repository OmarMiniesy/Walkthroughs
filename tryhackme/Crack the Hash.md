
#hash #hashcat #johntheripper #cracking 

---

1. `48bb6e862e54f2a795ffc4e541caed4d`

> Using [crackstation.net](https://crackstation.net/), we paste the hash and get the result : `easy`.
###### :: `easy` .

2. `CBFDAC6008F9CAB4083784CBD1874F76618D2A97`

> Using [crackstation.net](https://crackstation.net/), we paste the hash and get the result: `password123`.
###### :: `password123` .

3. `1C8BFE8F801D79745C4631D09FFF36C82AA37FC4CCE4FC946683D7B336B63032`
> Using [crackstation.net](https://crackstation.net/), we paste the hash and get the result: `letmein`.
###### :: `letmein` .

4. `$2y$12$Dwt1BZj6pcyc3Dy1FWZ5ieeUznr71EeNkJkUlypTsgbX1H68wsRom`
> First thing i do, is i analyze the hash type using a [hash analyzer](https://www.tunnelsup.com/hash-analyzer/) tool.
> We see that the type of hash is `bcrypt`.
> Search in hashcat for the `bcrypt` cracking mode, see that it is for `-m 3200`.

> We also filter `rockyou.txt` for 4 character words as by the hint using this command:
``` shell
grep -E '^.{4}$' /usr/share/wordlists/rockyou.txt > output.txt 
```
> Also add the hash to a file called `pass.txt`, and we run the hashcat tool.
```shell
hashcat -m 3200 -a 0 -O pass output.txt              
```
> After a while, we get the password: `bleh`.
###### :: `bleh` .

5. `279412f945939ba78ce0758d3fd83daa`

> Using [crackstation.net](https://crackstation.net/), we paste the hash and get the result : `Eternity22`.
###### :: `Eternity22` .

6. `F09EDCB1FCEFC6DFB23DC3505A882655FF77375ED8AA2D1C13F640FCCC2D0C85`
> Using [crackstation.net](https://crackstation.net/), we paste the hash and get the result : `paule`.
###### :: `paule` .

7. `1DFECA0C002AE40B8619ECF94819CC1B`

> Using [crackstation.net](https://crackstation.net/), we paste the hash and get the result : `n63umy8lkf4i`.
###### :: `n63umy8lkf4i` .

8. `$6$aReallyHardSalt$6WKUTqzq.UQQmrm0p/T7MPpMbGNnzXPMAXi4bJMl9be.cfi3/qxIf.hsGpS41BqMhSrHVXgMpdjS6xeKZAs02.`
> Look in the [hashcat example](https://hashcat.net/wiki/doku.php?id=example_hashes) page for the hash matching the one we have.
> See that it has `-m 1800`.

> Add the hash to a file called `pass` and start the attack with the `rockyou.txt` wordlist.
```
hashcat -m 1800 -a 0 -O pass /usr/share/wordlists/rockyou.txt
```

> We see the result after a while `waka99`.
###### :: `waka99` .

9. `e5d8870e5bdd26602cab8dbe07a942c8669e56d6` with salt `tryhackme`.
> First thing i do is that i analyze the hash type using a [hash analyzer](https://www.tunnelsup.com/hash-analyzer/) tool, and we see it is `SHA1`.
> Since we have a salt, we add it to the end separated by a `:`.

> Look in the [hashcat example](https://hashcat.net/wiki/doku.php?id=example_hashes) page for the hash matching the one we have.
> See that it has `-m 110` as it is `SHA1` with hash and a salt after it.
> Running hashcat with the `hash:salt` in the `pass` file and using `rockyou.txt` as a wordlist.
```
hashcat -m 110 -a 0 -O pass /usr/share/wordlists/rockyou.txt
```

> Doesn't work.
> Trying the other modes for `SHA-1` with a salt, and we get that `-m 160` is the one that works.
> The result is: `481616481616`.
###### :: `481616481616` .

---

