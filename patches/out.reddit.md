Reddit tracks the links you open with a redirect to out.reddit.com. This patch bypasses the redirect without visiting the site.

**NOTE:** This will not bypass opening links from comments!

Search for ```.class public final Lcom/reddit/domain/model/OutboundLink;```

Replace the getUrl method in the same file with this:

```smali
# get the "url" query in the out.reddit.com link and return that instead.
.method public final getUrl()Ljava/lang/String;
    .locals 5

    iget-object v4, p0, Lcom/reddit/domain/model/OutboundLink;->url:Ljava/lang/String;

    const-string v1, "https://out.reddit.com"
    if-eqz v4, :return

    .line 1
    invoke-virtual {v4, v1}, Ljava/lang/String;->startsWith(Ljava/lang/String;)Z

    move-result v1

    const/4 v2, 0x0

    if-eqz v1, :return

    .line 2
    invoke-static {v4}, Landroid/net/Uri;->parse(Ljava/lang/String;)Landroid/net/Uri;

    move-result-object v0

    const-string v3, "url"

    invoke-virtual {v0, v3}, Landroid/net/Uri;->getQueryParameter(Ljava/lang/String;)Ljava/lang/String;

    move-result-object v0

    if-eqz v0, :return

    const-string v3, "UTF-8"

    .line 3
    invoke-static {v0, v3}, Ljava/net/URLDecoder;->decode(Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;
    move-result-object v0
    return-object v0

    :return
    return-object v4
.end method
```
