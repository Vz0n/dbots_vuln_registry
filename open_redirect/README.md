Many sites, after some process like logging-in, getting into a tab... etc. sends to you a redirect to the page that you were before starting the process. They often save the page that you were before in attributes like `redirect`, `to`, `go`... etc. On a URL that could be represented as:

`/login?redirect=/dashboard`

When there is no validation of that `redirect` parameter, you can put something like `https://google.com` instead, and when you end the process, you will be redirected to Google. There is no restriction for the redirect so it's "open". On Discord bots this is a bit useful for phishing attacks as the redirect will be issued after a user authorizes your application; they may trust the new page as the trusted application redirected them to it.

On formal Bug Bounty platforms like HackerOne or BugCrowd, this bug by itself is considered generally out of scope or informative as the impact is just phishing, which is something that you can easily prevent by looking where you are at. Most times, this bug is chained with another application function(s) to prove real harm.

The cases here aren't interesting at all. It's just to keep record.
