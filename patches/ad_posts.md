## Ad posts

### Metod 1 (Easy but broken)

Ad posts are baked in the normal post requests so to remove post ads one needs to break the parsing.
To break the ad posts simply rename all occurances of `"AdPost"` to something else (*keep the quoutation marks though*),
this will remove all ad posts in the main feed, but does not remove posts in the custom feeds :(

### Method 2

We can edit the Duplicates filter to also filter out promoted posts :)

Seach for `RemoveDuplicatesFilter.kt` it will contain a single method which filters out posts, edit it like this:

```smali
# virtual methods
.method public b(Ljava/util/List;LTf/l;)Ljava/util/List;
    # Change this to a 5
    .locals 5
    .annotation system Ldalvik/annotation/Signature;
        value = {
            "(",
            "Ljava/util/List<",
            "+TT;>;",
            "LTf/l<",
            "TT;>;)",
            "Ljava/util/List<",
            "TT;>;"
        }
    .end annotation

    .line 1
    new-instance v0, Ljava/util/HashSet;

    invoke-direct {v0}, Ljava/util/HashSet;-><init>()V

    .line 2
    new-instance v1, Ljava/util/ArrayList;

    invoke-direct {v1}, Ljava/util/ArrayList;-><init>()V

    .line 3
    invoke-interface {p1}, Ljava/lang/Iterable;->iterator()Ljava/util/Iterator;

    move-result-object p1
    
    # cond_0 here is effectively a label for continue in the loop
    :cond_0
    :goto_0
    invoke-interface {p1}, Ljava/util/Iterator;->hasNext()Z

    move-result v2

    if-eqz v2, :cond_1

    invoke-interface {p1}, Ljava/util/Iterator;->next()Ljava/lang/Object;

    move-result-object v2

    .line 4
    move-object v3, v2

    check-cast v3, Lcom/reddit/domain/model/ILink;

    # Start here
    # After v3 is cast as an ILink we can check if it's promoted and then filter it out
    # if ((v3 as ILink).getPromoted()) continue
    invoke-virtual {v3}, Lcom/reddit/domain/model/ILink;->getPromoted()Z
    move-result v4
    if-nez v4, :cond_0
    # End of custom code

    .line 5
    invoke-virtual {v3}, Lcom/reddit/domain/model/ILink;->getUniqueId()Ljava/lang/String;

    move-result-object v3

    .line 6
    invoke-virtual {v0, v3}, Ljava/util/HashSet;->add(Ljava/lang/Object;)Z

    move-result v3

    if-eqz v3, :cond_0

    .line 7
    invoke-virtual {v1, v2}, Ljava/util/ArrayList;->add(Ljava/lang/Object;)Z

    goto :goto_0

    .line 8
    :cond_1
    new-instance p1, Ljava/util/ArrayList;

    invoke-direct {p1}, Ljava/util/ArrayList;-><init>()V

    .line 9
    invoke-virtual {v1}, Ljava/util/ArrayList;->iterator()Ljava/util/Iterator;

    move-result-object v0

    :cond_2
    :goto_1
    invoke-interface {v0}, Ljava/util/Iterator;->hasNext()Z

    move-result v1

    if-eqz v1, :cond_5

    invoke-interface {v0}, Ljava/util/Iterator;->next()Ljava/lang/Object;

    move-result-object v1

    move-object v2, v1

    check-cast v2, Lcom/reddit/domain/model/ILink;

    if-eqz p2, :cond_4

    .line 10
    invoke-virtual {p2}, LTf/l;->c()Ljava/util/Set;

    move-result-object v3

    invoke-virtual {v2}, Lcom/reddit/domain/model/ILink;->getUniqueId()Ljava/lang/String;

    move-result-object v2

    invoke-interface {v3, v2}, Ljava/util/Set;->contains(Ljava/lang/Object;)Z

    move-result v2

    if-nez v2, :cond_3

    goto :goto_2

    :cond_3
    const/4 v2, 0x0

    goto :goto_3

    :cond_4
    :goto_2
    const/4 v2, 0x1

    :goto_3
    if-eqz v2, :cond_2

    invoke-virtual {p1, v1}, Ljava/util/ArrayList;->add(Ljava/lang/Object;)Z

    goto :goto_1

    :cond_5
    return-object p1
.end method
```
