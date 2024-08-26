
---

The idea here is that there is a checksum provided.
- This checksum is used to make sure that the hash is correct.

After connecting to the machine using `ssh`:
```bash
ssh -p 51031 ctf-player@rhea.picoctf.net
```

and specifying the password:
```text
f3b61b38
```

We see a set of files:
```bash
ctf-player@pico-chall$ ls
checksum.txt  decrypt.sh  files
```

The `checksum.txt` holds the checksum value, and the `decrypt.sh` has a script that can decrypt a hash.
- The `decrypt.sh` will only work on the file that has the correct hash.
- We can identify which file has the correct hash by comparing it with the value in the `checksum.txt` file.

To do so, we can find the checksum of all the files in the `files` directory:
```bash
sha256sum files/*
```

> This outputs the checksums of all the files. The file we want will have the checksum that is similar to the one in the `checksum.txt` file.

We can use the `grep` tool to do that with the value of the checksum:

```bash
ctf-player@pico-chall$ cat checksum.txt
fba9f49bf22aa7188a155768ab0dfdc1f9b86c47976cd0f7c9003af2e20598f7
```

```bash
sha256sum files/* | grep fba9f49bf22aa
fba9f49bf22aa7188a155768ab0dfdc1f9b86c47976cd0f7c9003af2e20598f7  files/87590c24
```

We see that the file that has this hash is: `files/87590c24`. 
- Using the `decrypt.sh` script on that file will work, producing the flag
```bash
ctf-player@pico-chall$ ./decrypt.sh files/87590c24 
picoCTF{trust_but_verify_87590c24}
```

---
