# Install Eclipse plugins using  p2f file


In Eclipse, you can import a .p2f (Product2Feature) file using the command line by using the Eclipse director application, which is part of the Eclipse IDE. Here's how you can do it:

1. Navigate to the directory where your Eclipse director application is located. The director application is typically found in the "eclipse" directory of your Eclipse installation.
    
2. Use the following command to import a .p2f file:
    
    
    `eclipse -application org.eclipse.equinox.p2.director -repository file:/path/to/your/p2f/file.p2f -installIU your.target.feature.id`
    
    - Replace `/path/to/your/p2f/file.p2f` with the actual file path to your .p2f file.
    - Replace `your.target.feature.id` with the ID of the feature you want to install.

To specify and install multiple features using the Eclipse director application, you can provide a comma-separated list of `feature.id` values in the `-installIU` parameter.

To install all features listed in a .p2f (Product2Feature) file, you can use the Eclipse director application with a specific parameter. Here's how you can do it:

`eclipse -application org.eclipse.equinox.p2.director -repository file:/path/to/your/p2f/file.p2f -installIU all`

[RCP bookmarks](rcp.p2f)
[STS bookmarks](sts.p2f)

# Remove old versions of Eclipse plugins

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

# Install plugin from local archive

To install the product activation kit from a local file, run the following command:

- Windows
```
eclipsec.exe 
-application org.eclipse.equinox.p2.director 
-installIU com.ibm.zod.perm.lumkit.feature.feature.group 
-repository jar:file:/C:/temp/com.ibm.zod.perm.lumkit.update-site-1.0.0-20181120.164442-4.zip!/
```

- MacOS
```
eclipse
-application org.eclipse.equinox.p2.director
-installIU com.ibm.zod.perm.lumkit.feature.feature.group 
-repository 'jar:file:/Users//Downloads/com.ibm.zod.perm.lumkit.update-site-1.0.0-20181120.164442-4.zip!/'
```
