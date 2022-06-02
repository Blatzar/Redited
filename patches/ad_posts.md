## Ad posts

Ad posts are baked in the normal post requests so to remove post ads one needs to break the parsing.
To break the ad posts simply rename all occurances of `"AdPost"` to something else (*keep the quoutation marks though*),
this will remove all ad posts in the main feed.

*Does not seem to work on Custom Feeds!* 

I have a potential patch to Custom feeds, but since they are broken even in the normal application it is hard to test it.
