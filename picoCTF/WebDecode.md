
---

Opening the link and inspecting all pages, the `about.html` page has a suspicious string:
```text
cGljb0NURnt3ZWJfc3VjYzNzc2Z1bGx5X2QzYzBkZWRfMTBmOTM3NmZ9
```

Giving this string to [cyberchef](https://gchq.github.io/CyberChef) and using the magic decode, we see that it was base64 encoded, and this is its output when decoded:
```text
picoCTF{web_succ3ssfully_d3c0ded_10f9376f}
```

---
