## Ad posts

### Metod 1 (Easy but broken)

Ad posts are baked in the normal post requests so to remove post ads one needs to break the parsing.
To break the ad posts simply rename all occurances of `"AdPost"` to something else (*keep the quoutation marks though*),
this will remove all ad posts in the main feed, but does not remove posts in the custom feeds :(

### Method 2 (Blocks all post ads afaik)

All posts are stored in listables, if you can filter the posts when they get put in those listables you can remove all ad posts.
I wrote some custom smali to filter all posts marked as promoted, in `Listing.smali` (search for `.class public final Lcom/reddit/domain/model/listing/Listing;`)

Edit the contructor like this:
```smali
.method public constructor <init>(Ljava/util/List;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;Z)V
    .locals 4
    .annotation system Ldalvik/annotation/Signature;
        value = {
            "(",
            "Ljava/util/List<",
            "+TT;>;",
            "Ljava/lang/String;",
            "Ljava/lang/String;",
            "Ljava/lang/String;",
            "Ljava/lang/String;",
            "Z)V"
        }
    .end annotation

    const-string v0, "children"

    invoke-static {p1, v0}, Lc32/k;->f(Ljava/lang/Object;Ljava/lang/String;)V

    .line 1
    invoke-direct {p0}, Ljava/lang/Object;-><init>()V

    .line 2
    # START HERE, ALL ABOVE SHOULD BE UNTOUCHED CODE YOU DECOMPILED EXCEPT .locals 4 AT THE TOP
    # p1 is the original list
    check-cast p1, Ljava/lang/Iterable;

    # v1 = new ArrayList();
    new-instance v1, Ljava/util/ArrayList;
    invoke-direct {v1}, Ljava/util/ArrayList;-><init>()V
    check-cast v1, Ljava/util/Collection;

    invoke-interface {p1}, Ljava/lang/Iterable;->iterator()Ljava/util/Iterator;

    # v3 = list.iterator();
    move-result-object v3

    :continue

    # v2 = iterator.hasNext()
    invoke-interface {v3}, Ljava/util/Iterator;->hasNext()Z
    move-result v2

    # Exit loop if no next
    if-eqz v2, :exit_loop

    invoke-interface {v3}, Ljava/util/Iterator;->next()Ljava/lang/Object;
    # v2 = iterator.next()
    move-result-object v2

    # ILink
    instance-of v0, v2, Lcom/reddit/domain/model/ILink;
    if-eqz v0, :afterLink

    # ---------------------------------------
    check-cast v2, Lcom/reddit/domain/model/ILink;
    
    # if (getPromoted()) continue 
    invoke-virtual {v2}, Lcom/reddit/domain/model/ILink;->getPromoted()Z
    move-result v0
    if-nez v0, :continue
    # ---------------------------------------
    :afterLink

    # Add any object which survives the check and continue the loop
    invoke-interface {v1, v2}, Ljava/util/Collection;->add(Ljava/lang/Object;)Z
    goto :continue
    :exit_loop
    iput-object v1, p0, Lcom/reddit/domain/model/listing/Listing;->children:Ljava/util/List;
    # END OF EDITING
    
    .line 3
    iput-object p2, p0, Lcom/reddit/domain/model/listing/Listing;->after:Ljava/lang/String;

    .line 4
    iput-object p3, p0, Lcom/reddit/domain/model/listing/Listing;->before:Ljava/lang/String;

    .line 5
    iput-object p4, p0, Lcom/reddit/domain/model/listing/Listing;->adDistance:Ljava/lang/String;

    .line 6
    iput-object p5, p0, Lcom/reddit/domain/model/listing/Listing;->geoFilter:Ljava/lang/String;

    .line 7
    iput-boolean p6, p0, Lcom/reddit/domain/model/listing/Listing;->hasRecommendations:Z

    return-void
.end method
```

### Method 3 (Some ads slip through when scrolling horizontally)

We can edit the Duplicates filter to also filter out promoted posts :)

Seach for `RemoveDuplicatesFilter.kt` it will contain a single method which filters out posts, edit it like this:

(_Note: Another patch below this code_)

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

Reddit also inserts ads in to the feed when you click on posts, to disable this edit `InFeedAdDuFilters.kt`. Remove everything in line 1-2 besides :labels

```smali
.method public final a(Ljava/util/List;Lgz/j;)Ljava/util/List;
    .locals 2
    .annotation system Ldalvik/annotation/Signature;
        value = {
            "(",
            "Ljava/util/List<",
            "+TT;>;",
            "Lgz/j<",
            "TT;>;)",
            "Ljava/util/List<",
            "TT;>;"
        }
    .end annotation

    const-string p2, "items"

    invoke-static {p1, p2}, Ls42/j;->f(Ljava/lang/Object;Ljava/lang/String;)V

    .line 1
    # iget-object p2, p0, Lly/b;->a:Lcom/reddit/session/o;

    # invoke-interface {p2}, Lzj1/c;->d()Z

    # move-result p2

    # if-eqz p2, :cond_0

    # return-object p1

    .line 2
    :cond_0
    # iget-object p2, p0, Lly/b;->b:Lb00/b;

    # invoke-interface {p2}, Lb00/b;->g()Z

    # move-result p2

    # if-eqz p2, :cond_3

    .line 3
    new-instance p2, Ljava/util/ArrayList;

    invoke-direct {p2}, Ljava/util/ArrayList;-><init>()V

    .line 4
    invoke-interface {p1}, Ljava/lang/Iterable;->iterator()Ljava/util/Iterator;

    move-result-object p1

    :cond_1
    :goto_0
    invoke-interface {p1}, Ljava/util/Iterator;->hasNext()Z

    move-result v0

    if-eqz v0, :cond_2

    invoke-interface {p1}, Ljava/util/Iterator;->next()Ljava/lang/Object;

    move-result-object v0

    move-object v1, v0

    check-cast v1, Lcom/reddit/domain/model/ILink;

    .line 5
    invoke-virtual {v1}, Lcom/reddit/domain/model/ILink;->getPromoted()Z

    move-result v1

    xor-int/lit8 v1, v1, 0x1

    if-eqz v1, :cond_1

    invoke-virtual {p2, v0}, Ljava/util/ArrayList;->add(Ljava/lang/Object;)Z

    goto :goto_0

    :cond_2
    move-object p1, p2

    :cond_3
    return-object p1
.end method
```
