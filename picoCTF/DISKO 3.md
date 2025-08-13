
---

After downloading the file and unzipping it, we can list the files present in the disk image using the `flist` command:
```bash
└─$ fls disko-3.dd 
d/d 4:  log
v/v 3225859:    $MBR
v/v 3225860:    $FAT1
v/v 3225861:    $FAT2
V/V 3225862:    $OrphanFiles
```

We see there are many files present, but this `log` directory, as it has `d/d`, seems interesting.

Researching how to access it, we have to mount this image so that we can navigate inside of it.
- To do that, we create a folder that we will use to mount the disk image inside, its like a folder that we will use to store the contents of the entire disk image.
- Then, we mount the image in that folder using the `-o loop` option to treat the file as if it were a real disk.

```bash
mkdir ./disk-3
sudo mount -o loop disko-3.dd ./disk-3
```

After running this, we can now navigate to the directory we created and view the files inside.
- We `cd` into the mounted folder and into the `log` folder and list contents, we see there is a zip file called `flag.zip`.

```
┌──(kali㉿kali)-[~/Desktop/disk-3/log]
└─$ ls -lah       
total 1.6M
drwxr-xr-x 11 root root 3.5K Mar 31 11:32 .
drwxr-xr-x  3 root root  512 Dec 31  1969 ..
-rwxr-xr-x  1 root root    0 Mar 31 11:31 alternatives.log
-rwxr-xr-x  1 root root  194 Mar 31 11:31 alternatives.log.2.gz
drwxr-xr-x  2 root root 1.5K Mar 31 11:31 apt
-rwxr-xr-x  1 root root    0 Mar 31 11:31 boot.log
-rwxr-xr-x  1 root root 7.4K Mar 31 11:31 boot.log.1
-rwxr-xr-x  1 root root 7.0K Mar 31 11:31 boot.log.5
-rwxr-xr-x  1 root root 7.3K Mar 31 11:31 boot.log.6
-rwxr-xr-x  1 root root 293K Mar 31 11:31 daemon.log
-rwxr-xr-x  1 root root  24K Mar 31 11:31 debug
-rwxr-xr-x  1 root root 3.3K Mar 31 11:31 dpkg.log.1
-rwxr-xr-x  1 root root    0 Mar 31 11:31 dpkg.log.2.gz
-rwxr-xr-x  1 root root 158K Mar 31 11:31 dpkg.log.4.gz
-rwxr-xr-x  1 root root 254K Mar 31 11:31 dpkg.log.5.gz
-rwxr-xr-x  1 root root  32K Mar 31 11:31 faillog
-rwxr-xr-x  1 root root   78 Jul 17 11:06 flag.gz
drwxr-xr-x  3 root root  512 Mar 31 11:31 inetsim
drwxr-xr-x  3 root root 1.0K Mar 31 11:31 installer
drwxr-xr-x  3 root root  512 Mar 31 11:31 journal
-rwxr-xr-x  1 root root  19K Mar 31 11:31 kern.log.3.gz
-rwxr-xr-x  1 root root  19K Mar 31 11:31 kern.log.4.gz
-rwxr-xr-x  1 root root 286K Mar 31 11:31 lastlog
drwxr-xr-x  2 root root 1.0K Mar 31 11:31 lightdm
-rwxr-xr-x  1 root root   59 Mar 31 11:31 macchanger.log.4.gz
drwxr-xr-x  2 root root  512 Mar 31 11:31 mysql
drwxr-xr-x  2 root root  512 Mar 31 11:31 private
drwxr-xr-x  2 root root  512 Mar 31 11:31 stunnel4
-rwxr-xr-x  1 root root  46K Mar 31 11:31 syslog.3.gz
-rwxr-xr-x  1 root root 283K Mar 31 11:31 syslog.4.gz
drwxr-xr-x  2 root root  512 Mar 31 11:31 sysstat
-rwxr-xr-x  1 root root  195 Mar 31 11:31 vmware-network.1.log
-rwxr-xr-x  1 root root  195 Mar 31 11:31 vmware-network.2.log
-rwxr-xr-x  1 root root  195 Mar 31 11:31 vmware-network.3.log
-rwxr-xr-x  1 root root  195 Mar 31 11:31 vmware-network.4.log
-rwxr-xr-x  1 root root  193 Mar 31 11:31 vmware-network.5.log
-rwxr-xr-x  1 root root  193 Mar 31 11:31 vmware-network.6.log
-rwxr-xr-x  1 root root  195 Mar 31 11:31 vmware-network.7.log
-rwxr-xr-x  1 root root  195 Mar 31 11:31 vmware-network.8.log
-rwxr-xr-x  1 root root    0 Mar 31 11:31 vmware-network.log
-rwxr-xr-x  1 root root 3.0K Mar 31 11:31 vmware-vmsvc-root.1.log
-rwxr-xr-x  1 root root 3.1K Mar 31 11:31 vmware-vmsvc-root.2.log
-rwxr-xr-x  1 root root 3.3K Mar 31 11:31 vmware-vmsvc-root.3.log
-rwxr-xr-x  1 root root 2.3K Mar 31 11:31 vmware-vmsvc-root.log
-rwxr-xr-x  1 root root  20K Mar 31 11:31 vmware-vmtoolsd-root.log
-rwxr-xr-x  1 root root  86K Mar 31 11:31 wtmp
-rwxr-xr-x  1 root root  21K Mar 31 11:31 Xorg.0.log
-rwxr-xr-x  1 root root  21K Mar 31 11:31 Xorg.0.log.old
```

Unzipping the file and opening it we see the flag:

```bash
sudo gunzip flag.gz
cat flag
```

We get the flag:
```
picoCTF{n3v3r_z1p_2_h1d3_26d4f233}
```

---
