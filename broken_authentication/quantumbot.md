## The token with two states
*Fixed on: 26/07/2026*

[Website](https://quantum-bot.net) | [Discord](https://discord.gg/DPp36vkWTF)

It's a small multi-purpose bot. It has an economy system, levelsm, AI and other usual stuff that you see on multi-purpose bots.

![Portal](assets/quantumbot1.png)


Inspecting requests, I noticed that the Discord Bearer token issued by the app was being used to identify users in the website:

```bash
GET /api/dashboard/user/settings HTTP/1.1
Host: quantum-bot.net
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:152.0) Gecko/20100101 Firefox/152.0
Accept: application/json, text/plain, */*
Accept-Language: en-US,en;q=0.9
Accept-Encoding: gzip, deflate, br
Authorization: Bearer MTMwOTgyNjQ3ODE4NzM0ODA2OA.**************
... [snip]
```

So I tried to change the token for one issued by my application, and the server was still identifying me correctly. The same issue of [John-bot](johnbot.md), but on this I actually needed a token with the `guilds` scope, because it seems that the backend was trying to find the guild using this for permission checks.

https://github.com/user-attachments/assets/2afd044a-7e89-42f3-8e49-95b76602e9c1

The interesting thing is that, by inspecting the assets I saw some endpoints that only the bot dev can use, and with just the `identify` scope you can access things that are only yours (like your profile settings). Tricking the owner of the bot may be something feasible here actually.

The dev fixed it quickly.







