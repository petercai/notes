# Gitblit
[](#H1)Gitblit WAR Installation & Setup
---------------------------------------

1.  Download Gitblit WAR [1.9.3](https://github.com/gitblit/gitblit/releases/download/v1.9.3/gitblit-1.9.3.war) to the webapps folder of your servlet container.
2.  You may have to manually extract the WAR (zip file) to a folder within your webapps folder.
3.  By default, the Gitblit webapp is configured through `WEB-INF/data/gitblit.properties`.  
    Open `WEB-INF/data/gitblit.properties` in your favorite text editor and make sure to review and set:
    *   _git.packedGitLimit_ (set larger than the size of your largest repository)
4.  You may have to restart your servlet container.
5.  Open your browser to [http://localhost/gitblit](http://localhost/gitblit) or whatever the url should be.
6.  Enter the default administrator credentials: **admin / admin** and click the _Login_ button  
    **NOTE:** Make sure to change the administrator username and/or password!!

### [](#H2)WAR Data Location

By default, Gitblit WAR stores all data (users, settings, repositories, etc) in `${contextFolder}/WEB-INF/data`. This is fine for a quick setup, but there are many reasons why you don't want to keep your data within the webapps folder of your servlet container. Specifying an alternate _baseFolder_ also allows for simple upgrades in the future.

Choose a location that is writeable by your servlet container!

After you configure baseFolder and restart your container, Gitblit will copy the contents of the `WEB-INF/data` folder to your specified _baseFolder_ **IF** the file `${baseFolder}/gitblit.properties` does not already exist. This allows you to get going with minimal fuss.

#### [](#H3)Specifying baseFolder via GITBLIT_HOME

You can specify `GITBLIT_HOME` either as an environment variable or as a `-DGITBLIT_HOME` JVM system property.

#### [](#H4)Specifying baseFolder via web.xml

You may specify an external location for your data by directly editing `WEB-INF/web.xml` and manipulating the _baseFolder_ env-entry. Your servlet container may be smart enough to recognize the change and to restart Gitblit.

#### [](#H5)Specifying baseFolder via JNDI

This is a better way to configure your _baseFolder_ because it survives WAR redeploys or deployments of new versions. These directions assume you are running Tomcat as your container, other containers may have different ways to specify global JNDI environment entries.

1.  Open TOMCAT_HOME/conf/context.xml
2.  Insert an _Environment_ node within the _Context_ node.
    
    <Environment name="baseFolder" type="java.lang.String" value="c:/projects/git/gitblit/data" override="false" />
    

### [](#H6)Including Other Properties

SINCE 1.7.0

Gitblit supports loading it's settings from multiple properties files. You can achieve this using the `include=filename` key. This setting supports loading multiple files using a _comma_ as the delimiter. They are processed in the order defined and they may be nested (i.e. your included properties may include properties, etc, etc).