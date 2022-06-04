## Ad posts

### Metod 1 (Easy but broken)

Ad posts are baked in the normal post requests so to remove post ads one needs to break the parsing.
To break the ad posts simply rename all occurances of `"AdPost"` to something else (*keep the quoutation marks though*),
this will remove all ad posts in the main feed, but does not remove posts in the custom feeds :(

### Method 2

This one works in custom feeds but is likely to 'break' in following updates and the classes takes some time to find, but it works in Custom feeds.

By simply filtering all items before they get passed to the adapter you can filter out all posts tagged as promoted.

Look up `PresentationListingAdapter.kt`, there is probably multiple of them, but search for one with ~2000 lines of smali, alternatively search for `<T extends MutableListingLinkActions & ListingModeratorActions, Sort> extends ListableAdapter implements SortableLoadMoreListingAdapter` or some sort of substring of that in Jadx. 

There should be a function at the end taking a list of listables looking something like this in jadx:
```java

    @Override // p957Pz.ListingAdapter
    /* renamed from: s1 */
    public void mo6841s1(List<Listable> list) {
        Object add;
        FeedsFeatures m49806I = m49806I();
        if ((m49806I == null ? null : m49806I.mo82387z1()) == SortViewControlVariant.HIDE_SORT_VIEW_CONTROL) {
            Listable mo6842f = mo6842f();
            if (mo6842f != null) {
                list.add(m13601E1(), mo6842f);
            }
            Listable listable = this.f160728W0;
            if (listable != null) {
                list.add(m13599G1(), listable);
            }
        } else {
            Listable listable2 = this.f160728W0;
            if (listable2 != null) {
                list.add(m13599G1(), listable2);
            }
            Listable mo6842f2 = mo6842f();
            if (mo6842f2 != null) {
                list.add(m13601E1(), mo6842f2);
            }
        }
        list.add(this.f160730Y0);
        m49816B(list);
    }
```
The goal is adding a filter before the last line, which can be done with this handwritten smali (After .line 12):
```smali
.method public s1(Ljava/util/List;)V
    .locals 4
    .annotation system Ldalvik/annotation/Signature;
        value = {
            "(",
            "Ljava/util/List<",
            "Lhq/e;",
            ">;)V"
        }
    .end annotation

    .line 1
    invoke-virtual {p0}, Lcom/reddit/frontpage/ui/B;->I()LUf/n;

    move-result-object v0

    if-nez v0, :cond_0

    const/4 v0, 0x0

    goto :goto_0

    :cond_0
    invoke-interface {v0}, LUf/n;->z1()LGb/e;

    move-result-object v0

    :goto_0
    sget-object v1, LGb/e;->HIDE_SORT_VIEW_CONTROL:LGb/e;

    if-ne v0, v1, :cond_3

    .line 2
    invoke-virtual {p0}, Lpm/B;->D1()Lhq/e;

    move-result-object v0

    if-nez v0, :cond_1

    goto :goto_1

    .line 3
    :cond_1
    invoke-virtual {p0}, Lpm/B;->E1()I

    move-result v1

    invoke-interface {p1, v1, v0}, Ljava/util/List;->add(ILjava/lang/Object;)V

    .line 4
    :goto_1
    iget-object v0, p0, Lpm/B;->W0:Lhq/e;

    if-nez v0, :cond_2

    goto :goto_3

    .line 5
    :cond_2
    invoke-virtual {p0}, Lpm/B;->G1()I

    move-result v1

    invoke-interface {p1, v1, v0}, Ljava/util/List;->add(ILjava/lang/Object;)V

    goto :goto_3

    .line 6
    :cond_3
    iget-object v0, p0, Lpm/B;->W0:Lhq/e;

    if-nez v0, :cond_4

    goto :goto_2

    .line 7
    :cond_4
    invoke-virtual {p0}, Lpm/B;->G1()I

    move-result v1

    invoke-interface {p1, v1, v0}, Ljava/util/List;->add(ILjava/lang/Object;)V

    .line 8
    :goto_2
    invoke-virtual {p0}, Lpm/B;->D1()Lhq/e;

    move-result-object v0

    if-nez v0, :cond_5

    goto :goto_3

    .line 9
    :cond_5
    invoke-virtual {p0}, Lpm/B;->E1()I

    move-result v1

    invoke-interface {p1, v1, v0}, Ljava/util/List;->add(ILjava/lang/Object;)V

    .line 10
    :goto_3
    iget-object v0, p0, Lpm/B;->Y0:Lhq/f;

    .line 11
    invoke-interface {p1, v0}, Ljava/util/List;->add(Ljava/lang/Object;)Z

    .line 12
    
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

    # LinkPresentationModel
    instance-of v0, v2, LPw/i;
    if-nez v0, :LinkPresentationModel

    # Pretty sure only LinkPresentationModel is used, but
    # might as well check these too
    # You can remove all below until the goto

    # BlankAdPresentationModel
    instance-of v0, v2, Leq/a;
    if-nez v0, :continue

    # BrandLiftSurveyUiModel
    instance-of v0, v2, Lh9/a;
    if-nez v0, :continue

    # PromotedHeroItemUiModel
    instance-of v0, v2, LHH/b;
    if-nez v0, :continue

    # Skip special case for LinkPresentationModel
    goto :afterLinkPresentationModel

    # ---------------------------------------
    :LinkPresentationModel
    check-cast v2, LPw/i;

    # If is blankAd or is promoted then continue -> filter out the item
    # Get these from searching isBlankAd= from the toString
    # then following the variable to the correct public function

    # isBlankAd=
    invoke-virtual/range {v2 .. v2}, LPw/i;->R2()Z
    move-result v0
    if-nez v0, :continue

    # promoted=
    invoke-virtual/range {v2 .. v2}, LPw/i;->N1()Z
    move-result v0
    if-nez v0, :continue
    # ---------------------------------------

    :afterLinkPresentationModel

    # Add any object which survives the checks and continue the loop
    invoke-interface {v1, v2}, Ljava/util/Collection;->add(Ljava/lang/Object;)Z
    goto :continue

    :exit_loop

    # Finally use the filtered list and return
    invoke-virtual {p0, v1}, Lcom/reddit/frontpage/ui/B;->B(Ljava/util/List;)V

    return-void
.end method
```
