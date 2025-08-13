
---

After downloading the file, we run the `file` command:

```bash
└─$ file disko-2.dd      
disko-2.dd: DOS/MBR boot sector; 
partition 1 : ID=0x83, start-CHS (0x0,32,33), end-CHS (0x3,80,13), startsector 2048, 51200 sectors; 
partition 2 : ID=0xb, start-CHS (0x3,80,14), end-CHS (0x7,100,29), startsector 53248, 65536 sectors
```

We see there are 2 partitions in the output.
- After doing some research, we can gain better understanding of the disk partitions and their sizes using the `fdisk` command and using the `-l` flag.

```bash
└─$ fdisk -l ./disko-2.dd 
Disk ./disko-2.dd: 100 MiB, 104857600 bytes, 204800 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x8ef8eaee

Device        Boot Start    End Sectors Size Id Type
./disko-2.dd1       2048  53247   51200  25M 83 Linux
./disko-2.dd2      53248 118783   65536  32M  b W95 FAT32
```

We see clearly 3 pieces of information:
- The `Units`, or basically the size of 1 *sector* which is `512` bytes.
- For each device listed, there is a `Start` index.
- For each device listed, there is a `Sectors` count.

> Using these 3 pieces of information, we can pinpoint what device we want to zoom in on by specifying the start location, the number of sectors we want to read, and the size of each sector to read.

The question said we need to investigate the `Linux` partition. To extract this partition, we need these 3 numbers:
- Sector Size: `512` bytes.
- Start: `2048`.
- Sectors: `51200`.

Writing a command that extracts only this and saves it to another file:
```bash
dd if=disko-2.dd of=linux.dd bs=512 skip=2048 count=51200
```
- We give an input file through `if` of `disko-2.dd`.
- An output file of `linux.dd` which will hold the partition we extract through `of`.
- The sector size, or block size, of `512` through `bs`.
- The starting index of the partition through `skip` of `2048`.
- The number of sectors to read through `count` of `51200`.

Now, we can run `strings` on the file `linux.dd` and look for the flag:
```bash
└─$ strings linux.dd | grep pico
picoCTF{4_P4Rt_1t_i5_a93c3ba0}
```

---

