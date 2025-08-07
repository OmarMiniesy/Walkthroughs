
---

After downloading the file, I ran `file` to determine the filetype.

```bash
file disko-1.dd.gz
```

The output showed us that it is a `gzip` file.
- To decompress `gzip`, we use the tool `gunzip`.

```bash
gunzip disko-1.dd.gz
```

We then run the `strings` command on the output file:
```bash
strings disko0-1.dd
```
- We get a long output in the terminal.

Using `CTRL+SHIFT+F` or whatever method to find in the previous output, I searched for `picoCTF` and we get the flag:
```bash
picoCTF{1t5_ju5t_4_5tr1n9_c63b02ef}
```

---
