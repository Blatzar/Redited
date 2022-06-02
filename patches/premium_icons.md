## Premium app icons

In MyAccount.smali

```smali
.method public isPremiumSubscriber()Z
    .locals 1

    .line 1

    # Comment out this line below
    # iget-boolean v0, p0, Lcom/reddit/domain/model/MyAccount;->isPremiumSubscriber:Z

    const/4 v0, 0x1 # This is the addition, simply always return true

    return v0
.end method
```
