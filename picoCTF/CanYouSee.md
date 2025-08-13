
---

Downloading the file and unzipping the content, we get a `ukn_reality.jpg`.
- Since this is an image, the first thing I do is run `exiftool` to check its metadata.

```bash
└─$ exiftool ukn_reality.jpg 
ExifTool Version Number         : 13.00
File Name                       : ukn_reality.jpg
Directory                       : .
File Size                       : 2.3 MB
File Modification Date/Time     : 2024:03:11 20:05:53-04:00
File Access Date/Time           : 2025:08:11 13:55:02-04:00
File Inode Change Date/Time     : 2025:08:11 13:55:01-04:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : inches
X Resolution                    : 72
Y Resolution                    : 72
XMP Toolkit                     : Image::ExifTool 11.88
Attribution URL                 : cGljb0NURntNRTc0RDQ3QV9ISUREM05fYjMyMDQwYjh9Cg==
Image Width                     : 4308
Image Height                    : 2875
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 4308x2875
Megapixels                      : 12.4
```

We see the `Attribution URL` field has a value that looks like it is base64 encoded:
```
cGljb0NURntNRTc0RDQ3QV9ISUREM05fYjMyMDQwYjh9Cg==
```

Decoding this from base64 we get the flag:
```bash
└─$ echo "cGljb0NURntNRTc0RDQ3QV9ISUREM05fYjMyMDQwYjh9Cg==" | base64 -d
picoCTF{ME74D47A_HIDD3N_b32040b8}
```

---
