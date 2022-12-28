# Setting logging config file for Eclipse project
Save From : [java - Setting logging config file for Eclipse project - Stack Overflow](https://stackoverflow.com/questions/20678061/setting-logging-config-file-for-eclipse-project) 

## Content
In case anyone else stumbles across this as I did, you can specify VM arguments by run configuration by right clicking your project, selecting run as -> run configurations and switching to the arguments tab. Then just put in -Djava.util.logging.config.file=/your/file/path/here
## Note