
### Reflected XSS in a JavaScript URL with some characters blocked : EXPERT

---

> Checking where the injection point is, and it is in the `postId` parameter in the URL.

![expertCode](./screenshots/expert.png)

> Cleaning up this code to properly understand it:

![cleanCode](./screenshots/clean.png)

> Whatever we insert goes in the body section after the `/post%3fpostId%3d` which is the url encoding of `/post?postId=`.
> Now what is inserted is 5, which grabs the post with ID = 5.

> Our payload should find a way to break out of that string, and then call the alert function with the string `1337`.
> However, we know that the `()` are encoded, so we can't call the function normally.
> We need the function to get called by the code itself.

> First, we need to break out of that string.
```
postId=5&'},
```
> A single qoute to break from the string, and then the curly braces and the comma to add another item inside `fetch` function.
> This transforms the request to look like this.

![newReq](./screenshots/newreq.png)

> Now we can directly write JavaScript. Since the `()` are encoded, we need to find a different way to call the `alert` function.
> To do this, we use the `throw` function, and create an arrow function with no parameters to do the job. 
> Inside it, we call the alert function and the string 1337.
```
postId=5&'},x=a=>{throw onerror=alert,1337},
```
> We added a new variable called `x` that contains the function `a`.
> Function `a` has the logic to throw and call the alert. Adding the parameters after a comma.

> However this won't work because the space will be encoded. So we need to find a way to insert a space.
```
postId=5&'},x=a=>{throw/**/onerror=alert,1337},
```
> Done using the multi line comment. It forces a space like a new line.

> The javascript code now looks like this.
![newercode](./screenshots/newercode.png)

> Now we need to call `x`.
> We can set it to be equal to a function that is called automatically in javascript such as the `toString()` function, that is used whenever an object is to be used as a string.
> And then attempt to write code that will use the `toString` or `x` function.
```
postId=5&'},x=a=>{throw/**/onerror=alert,1337},toString=x,window+'',{_:'
```
> The window object we add to it an empty string, so javascript attempts to convert it to a string using the toString function. 
> This automatically calls our x function since toString=x.
> We then closed off the final `}` and used a dummy variable to get rid of the `'`.

![img](./screenshots/final.png)

> After submitting the paylod, we have to click on the `back to blog` button since this payload is in the `href` link attribute of that button.
> The alert pops up and the lab is complete.

---
