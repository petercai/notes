# How to remove old versions of Eclipse plugins? 
Save From : [How to remove old versions of Eclipse plugins? - Stack Overflow](https://stackoverflow.com/questions/221476/how-to-remove-old-versions-of-eclipse-plugins) 

## Content
11

This answer is not useful

Save this answer.

[](/posts/52577085/timeline)

Show activity on this post.

I use the following command:

```
eclipse -application org.eclipse.equinox.p2.garbagecollector.application -profile epp.package.jee

```

**Notes:**

1.  This is documented in [Equinox/p2/FAQ](https://wiki.eclipse.org/Equinox/p2/FAQ#Why_aren.27t_bundles_being_removed_when_their_associated_feature_has_been_removed.3F), see "Why aren't bundles being removed when their associated feature has been removed?"
    
    The FAQ answer references [an Eclipse Community Forum thread](http://www.eclipse.org/forums/index.php?t=msg&goto=543500) "Plug-in jar remains after feature uninstall" (June 2010) as the origin for this recipe.
    
    The recipe is still valid nowadays, with Eclipse 4.8 Photon.
    
2.  The `-profile` argument depends on what packaging of Eclipse IDE you are using. The above one (`epp.package.jee`) is for "Eclipse for Java EE Developers". I found it in the file `configuration/config.ini`. The relevant line there is the following:
    
    ```
    eclipse.p2.profile=epp.package.jee
    ```
## Note