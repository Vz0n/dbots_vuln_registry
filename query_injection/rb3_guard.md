## Silently dumping your db
*Fixed on: ??/11/2025*

[Website](https://www.rb3guard.com) | [Discord](https://discord.gg/fHmcxDWdjQ)

This bot is like Wick; anti-raid and those things.

To access a dashboard, you send a request to `/dash/:guild_id`... being curious, I tried to put an ID like `12345'`, and the server stopped responding, only throwing a `504 Gateway timeout`.

But when I added a plus sign and another single quote (`1234' + '5`), the request was still working, so I suspected that there was some sort of injection. Then I made my request like this:

```
1234' + if(substr(database(),1,1)='r','1',false) + '5
```

And with the `r` the server would throw a `302 Found`, but with any other character, it will throw the same timeout as before. That's a blind SQL injection.

I reported it to the dev, and he said that he was going to fix it later. On this date, I decided to look at it again and indeed, it's fixed.