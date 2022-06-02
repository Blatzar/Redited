## Comment ads

### Method 1
*This one is not properly tested yet*

Search for `Error loading comment ads` in the whole project, the file should have `.source "RedditCommentsPageAdRepository.kt"` in the top.

In the same function as `Error loading comment ads` is located in return null, like what's shown below.
```smali
# virtual methods
.method public a(ILjava/lang/String;Ljava/lang/String;Ljava/lang/String;ZLwS/d;)Ljava/lang/Object;
    .locals 9
    .annotation system Ldalvik/annotation/Signature;
        value = {
            "(I",
            "Ljava/lang/String;",
            "Ljava/lang/String;",
            "Ljava/lang/String;",
            "Z",
            "LwS/d<",
            "-",
            "Lcom/reddit/domain/model/Link;",
            ">;)",
            "Ljava/lang/Object;"
        }
    .end annotation

    # Add these two lines at the top of the function like this to return null
    const/4 p1, 0x0
    return-object p1
```

### Method 2

Rename all occurances of `api/comments_page_ads` to something else, this will make the network requests to fetch the ads fail :)
