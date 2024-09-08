
---

Taking the hint, I cloned the entire website to the local machine using:

```bash
wget -m http://saturn.picoctf.net:64806/
```

Then, I started to go through the source code but got nothing useful, so I resorted to using `grep` to find the flag.

```bash
grep -r "picoCTF"
```
- Run this command in the folder of the website, and the flag is produced....

```bash
css/style.css:/** banner_main picoCTF{1nsp3ti0n_0f_w3bpag3s_8de925a7} **/
```

The flag:

```text
picoCTF{1nsp3ti0n_0f_w3bpag3s_8de925a7}
```

---
