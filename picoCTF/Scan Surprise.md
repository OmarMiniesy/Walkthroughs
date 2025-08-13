
---

Downloading the image and opening it, we see that it is a QR code.
- Doing a quick google search, we can obtain the content of the QR code using something called `zbar-tools`.

Installing it first:
```bash
sudo apt install zbar-tools
```

Then, running the tool `zbarimg` on the target image, we get the flag:
```bash
|─$ zbarimg flag.png 
QR-Code:picoCTF{p33k_@_b00_7843f77c}
scanned 1 barcode symbols from 1 images in 0 seconds
```

---
