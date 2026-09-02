## Configuring everything from everywhere
*Fixed on: 25/08/2026*

[Website](https://kena.bot) | [Discord](https://discord.gg/kenabot-910907192989843548)

This is a music bot (on its way to becoming multi-purpose) that is widely used across Hispanic Discord communities. One of the featured users of this bot is the [Gatitos World 2](https://discord.gg/gatitos2) server.

![Portal](assets/kenabot1.png)

There are 8 bots (or Kenas) that you can use in your server, but only four of them are publicly available (1 to 4). Your guild must be whitelisted to use the other four. And I'll go with the first one.

![Kenas list](assets/kenabot2.png)

So I started watching the dashboard, and while inspecting requests I noticed that the site was using [Remix](https://remix.run/) in the backend, because responses to actions were getting returned in a serialized format:

```bash
HTTP/1.1 200 OK
Date: Tue, 25 Aug 2026 14:20:15 GMT
Content-Type: text/x-script; charset=utf-8
X-Powered-By: Express
X-Remix-Response: yes
Cf-Cache-Status: DYNAMIC

[{"_1":2,"_7":8,"_10675":10676},"root",{"_3":4}, ... [snip]
```

On the main bot (the blue one, if it's still not clear), there were only two modules available (greeter and music). Following the requests inspection I noticed that the target bot id to configure was in a query parameter of the actions:

```bash
GET /es/guilds/{guild.id}/music.data?bot={bot.id} HTTP/1.1
Host: dash.kena.bot
Accept: */*
Cookie: ...[snip]
```

It doesn't seem interesting, but what if it's used in some request to an API? Sounds out of nowhere but, I added a `#` (`%23` encoded) at the end of the bot ID and everything was still ok and... there was a very weird thing hidden in plain sight in the serialized response:

```json
"__v",0,"_id","\"6a11982bd25e9c6adbaaa6ed\"","allowed_modules",[10690,10691,10692,10693,10694,10695,10696,10697],"music","greeter","bot_id","910965586622816326","collect_guild_stats",true,"whitelisted_guilds",[],"createdAt","2026-05-23T12:06:03.732Z","custom_name","default_enabled_modules",[10690],"default_visible_modules",[],"embed_color","guild_stats",{"_10712":10713,"_10714":10715},"added",155,"kicked",91,"hidden_module_testers",{"_10693":10718,"_10692":10724,"_10697":10725,"_10696":10726,"_10695":10727},[10719,10720,10721,10722,10723],"",[10721,10723,10720],[10720,10721,10723],[10720,10721,10723],[10720,10723],"hidden_modules",[10692,10693,10695,10696,10697],"language","max_guilds","mode","prod","owner_id","prefix","premium_id","presence",{"_10739":10740,"_10741":10742,"_10743":10744,"_10745":10746},"activity_name","kena.bot","activity_type","Playing","activity_url","","status","online","token","updatedAt","2026-08-25T15:11:59.893Z","visibility","public"
```

Which, after some thought, I deserialized into this:

```jsonc
{
    "_id":"6a11982bd25e9c6adbaaa6ed",
    "allowed_modules":["music", "greeter"],
    "bot_id":"910965586622816326",
    "collect_guild_stats":true,
    "custom_name":null,
    "default_enabled_modules":["music"],
    "default_visible_modules":[],
    "whitelist_guilds":[],
    "embed_color":null,
    "guild_stats":{
        // ... [snip]
    },
    "added":155,
    "kicked":91,
    "hidden_module_testers":{
        // ... [snip]
    },
    "hidden_modules":["<hidden>"],
    "language":null,
    "max_guilds":null,
    "mode":"prod",
    "owner_id":null,
    "prefix":null,
    "premium_id":null,
    "presence":{
        "activity_name":"kena.bot",
        "activity_type":"Playing",
        "activity_url":"",
        "status":"online",
    },
    "token":null,
    "visibility":"public",
    "createdAt":"2026-05-23T12:06:03.732Z",
    "updatedAt":"2026-08-25T15:11:59.893Z",
    "__v":0
}
```

So it seems that the backend is fetching (w/ `GET`) an internal API with that ID and there is no validation on it. I first got excited on the `token` field because this bot also offers hosting custom instances, and it would mean that probably tokens are stored in there, but it seems that there was a dashboard check against the returned `whitelist_guilds` if the `visibility` was not public, making the dashboard return a 403 with `bot-not-whitelisted` if I try to fetch data from a non public bot. But if this API is used to get data from the guild, it is also used to *save data*?

The route `/es/guilds/{guild.id}/settings.data?bot={bot.id}` is used to configure who can access the dashboard, the bot profile and some of its commands. The `POST` request made to this endpoint has the following body:

```json
{
    "prefix":"String",
    "language":"OneOf<Locales>",
    "timezone":"Timezone",
    "embed_color":"String",
    "enabled_embed_color":"Boolean",
    "permissions":{
        "admin_roles":"Array<Snowflake>",
        "admin_users":"Array<Snowflake>",
        "module_grants":"ModuleGrant"
    },
    "commands":{
        "help":{
            "enabled":"Boolean"
        },
        "commands":{
            "enabled":"Boolean"
        },
        "config":{
            "enabled":"Boolean"
        }
    }
}
```

So if my last proposition is true, it would mean that there is an endpoint of the internal API that's used to configure guilds, which I found as `<initial-path>/{bot.id}/guilds/{guild.id}` (with `GET` it returned the guild settings). I tested this settings request with that endpoint pointing to another guild (`?bot={bot.id}/guilds/{guild.id}%23`)... and it worked:

https://github.com/user-attachments/assets/d2068b9d-1646-4119-9090-671b5fd5a457

You can modify and get various things on that internal API (even the same bot object), but as the dashboard was validating `POST` requests structure, you were limited to send requests with that method to endpoints which accepts a body similar to the ones used to modify modules and guild settings.

The devs fixed it quickly.