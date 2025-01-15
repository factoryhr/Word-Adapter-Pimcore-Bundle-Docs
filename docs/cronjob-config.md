# Cronjob configuration 
In order to insure proper working of word adapter the following cron jobs should be set. 

### 1. Editing mode delete cronjob
This cronjob deletes the rows in WordAdapterEditingMode table. 
The WordAdapterEditingMode is contacted by wordadapter on each heartbeat. 
Heartbeat is just a simple http request word adapters made just to tell that someone is editing the document. 
Once this heartbeat stops for some period of time we need to remove heartbeat that was stored. 

That means that after some period of time the document that was being edited would not considered to be in edit mode .

That period of time that needs to be elapsed before editing mode will be deleted is defined by the config param:

```yml
config:
    #...
    lock_release_interval: 300 #interval in seconds
```

In a nutshell when user opens the document it is considered to be the user who triggered editing mode. If the user 
for example closes word adapter or loses internet connection, his heartbeat would stop for that session. This is when 
the cronjob would kick in and delete heartbeat for that session, which means that user would not be considered as 
editor of the document. 

### Cronjob entry
```yaml
* * * * * /path/to/project/bin/console factory-word-adapter:release-lock > /dev/null 2>&1
```
