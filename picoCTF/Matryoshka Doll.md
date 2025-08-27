
---

To extract files in images, we can use the `binwalk` tool with the `-e` flag.

```bash
binwalk -e dolls.jpg
```

We get an output folder. We then `cd` into the folder `base images`, and do the same command again on the image:

```bash
binwalk -e 2_c.jpg
```

We get another output folder, we `cd` again and do the exact same command.

Keep repeating until the output file is the `flag.txt` file.

```bash
picoCTF{ac0072c423ee13bfc0b166af72e25b61}
```

---
