
### Reflected XSS with AngularJS sandbox escape without strings : EXPERT

---

> See that the input dummy text is inside an AngularJs sandbox.
> To escape, we need to create a string using the constructor and `charAt` functions, as well as the `[].join` method. This tricks the sandbox into thinking this is not a dangerous expression.
> However, we cannot use `eval` to create strings, so we need to create a string as given in the explanation before the lab.

> First, we create an empty string and escape from the angular sandbox.
```
toString().constructor.prototype.charAt=[].join;
```

> Then we concatenate to that string we found a string but built from the `fromCharCode` function by giving the ASCII codes of characters.
```
toString().constructor.prototype.charAt=[].join;[1]|orderBy:toString().constructor.fromCharCode(120,61,97,108,101,114,116,40,49,41)=1
```
> This ASCII translates to `x=alert(1)`. Which is the payload we need to execute.


> Before we use the payload, we need to encode the `=` character to `%3d`.
> Paste this payload after the search parameter: `search=mins&<PAYLOAD>`.

---

The exploit uses `toString()` to create a string without using quotes. It then gets the `String` prototype and overwrites the `charAt` function for every string. This effectively breaks the AngularJS sandbox. Next, an array is passed to the `orderBy` filter. We then set the argument for the filter by again using `toString()` to create a string and the `String` constructor property. Finally, we use the `fromCharCode` method generate our payload by converting character codes into the string `x=alert(1)`. Because the `charAt` function has been overwritten, AngularJS will allow this code where normally it would not.
