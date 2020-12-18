# Archive

The Archive module allows you to create compressed data files ([zstd](https://github.com/facebook/zstd)) from Elasticsearch indexes. The archive checks the age of each document in the index and if it is older than defined in the job, it is copied to the archive file.

## Configuration

### Enabling module

To configure module edit `kibana.yml` configuration file end set path to the archive directory - location where the archive files will be stored:

```bash
vim /etc/kibana/kibana.yml
```

remove the comment from the following line and set the correct path to the archive directory:

```vim
archive.archivefolderpath: '/var/lib/elastic_archive_test'
```

## Archive task

### Create Archive task

1. From the main navigation go to the "Archvie" module.

   ![](/media/media/image155.png) 

2. On the "Archive" tab select "Create Task" and define the following parameters:

   - `Index pattern`- for the indexes that will be archive, for example `syslog*` ;
   - `Older than (days)` - number of days after which documents will be archived;
   - `Schedule task` (crontab format) - the work schedule of the ordered task.
   
   ![](/media/media/image156.png) 

### Task List

In the `Task List` you can follow the current status of ordered tasks. You can modify task scheduler or delete ordered task.

![](/media/media/image157.png) 

## Archive search

The Archive Search module can search archive files for the specific content and back result in the `Task List`

### Create Search task

1. From the main navigation go to the `Archive` module.
2. On the `Search` tab select `Create Task` and define the following parameters:

   - `Search text` - field for entered the text to be searched.
   - `File name` - list of archive file that will be searched.

![](/media/media/image158.png) 

### Task list

The searching process will can take long time. On the `Task List` you can follow the status of the searching process. Also you can view result and delete tasks.

![](/media/media/image159.png) 

## Archive Upload

The Archive Upload module move data from archive to Elasticsearch index and make it online.

### Create Upload task

1. From the main navigation go to the `Archive` module.

2. On the `Upload` tab select `Create Task` and define the following parameters:

   - `Destination index` - If destination index does not exist it will be created. If exists data will append.
- `File name` - list of archive file that will be recover to Elasticsearch index.
   

![](/media/media/image160.png) 

### Task List

The process will index data back into Elasticsearch. Depend on archive size the process can take long time. On the `Task List` you can follow the status of the recovery process. Also you can view result and delete tasks.

![](/media/media/image161.png) 




