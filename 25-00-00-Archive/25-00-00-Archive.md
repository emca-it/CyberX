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

## Archive Task

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

If the archiving task finds an existing archive file that matches the data being archived, it will check the number of documents in the archive and the number of documents in the index. If there is a difference in the number of documents then new documents will be added to the archive file.

## Archive Search

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

## Command Line tools

Archive files can be handled by the following commands `zstd`, `zstdcat`, `zstdgrep`, `zstdless`, `zstdmt`.

### zstd

The command for decompress `*.zstd` the Archive files, for example:

```bash
zstd -d winlogbeat-2020.10_2020-10-23.json.zstd -o
 winlogbeat-2020.10_2020-10-23.json
```

### zstdcat

The command for concatenate  `*.zstd` Archive files and print content on the standard output, for example:

```bash
zstdcat winlogbeat-2020.10_2020-10-23.json.zstd
```

### zstdgrep

The command for print lines matching a pattern from `*.zstd` Archive files, for example:

```bash
zstdgrep "optima" winlogbeat-2020.10_2020-10-23.json.zstd
```

Above example is searching documents contain the "optima" phrase in winlogbeat-2020.10_2020-10-23.json.zstd archive file.

### zstdless

The command for viewing Archive `* .zstd` files, for example:

```bash
zstdless winlogbeat-2020.10_2020-10-23.json.zstd
```

### zstdmt

The command for compress and decompress Archive `*.zdtd` file useing multiple CPU core (default is 1), for example:

```bash
zstdmt -d winlogbeat-2020.10_2020-10-23.json.zstd -o winlogbeat-2020.10_2020-10-23.json
```

