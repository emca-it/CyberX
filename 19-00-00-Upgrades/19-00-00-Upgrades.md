# Upgrades #

 You can check the current version using the API command:
```bash
curl -u $USER:$PASSWORD -X GET http://localhost:9200/license
```

## Upgrade from version 7.0.4

### General note

1. The following indices `.agents`, `audit`, `alert` currently uses rollover for rotation, after upgrade please use dedicated AIP for migration:

   ```bash
   curl -u $USER:$PASSWORD -X POST http://localhost:9200/license_logserver/prepareindex/$indexname
   ```

2. The Wiki plugin require open port `tcp/5603`

## Preferred Upgrade steps

1. Run upgrade script:
   
   ```bash
   /install.sh -u
   ```
   
2. Restart services:

   ```bash
    systemctl restart elasticsearch alert kibana cerebro wiki
   ```

3. Migrate Audit index to new format (the next call will display the current status of the task):

   ```bash
   curl -X POST 'http://localhost:9200/_logserver/prepareindex/audit' -u $USER:$PASSWORD
   ```

4. Migrate Alert index to new format (the next call will display the current status of the task):

   ```bash
   curl -XPOST 'http://localhost:9200/_logserver/prepareindex/alert' -u $USER:$PASSWORD
   ```

5. Migrate Agents index to new format (the next call will display the current status of the task):

   ```bash
    curl -XPOST 'http://localhost:9200/_logserver/prepareindex/.agents' -u $USER:$PASSWORD
   ```

6. Open `tcp/5603` port for Wikipedia plugin:
   ```bash
   firewall-cmd --zone=public --add-port=5603/tcp --permanent
   ```

   ```bash
   firewall-cmd --reload
   ```


## Alternative Upgrade steps (without install.sh script)

1. Stop services:
   
   ```bash
   systemctl stop elasticsearch alert kibana cerebro
   ```
   
2. Upgrade client-node (includes alert engine):
   
   ```bash
   yum update ./PACKAGE_NAME_VARIABLE-client-node-VERSION_TEMPLATE_VARIABLE-1.el7.x86_64.rpm
   ```
   
3. Upgrade data-node:
   
   ```bash
   yum update ./PACKAGE_NAME_VARIABLE-data-node-VERSION_TEMPLATE_VARIABLE-1.el7.x86_64.rpm
   ```
   
4. Start services:
   
   ```bash
   systemctl start elasticsearch alert kibana cerebro wiki
   ```
   
5. Migrate Audit index to new format (the next call will display the current status of the task):
   
   ```bash
   curl -XPOST 'http://localhost:9200/_logserver/prepareindex/audit' -u $USER:$PASSWORD
   ```
   
6. Migrate Alert index to new format (the next call will display the current status of the task):
   
   ```bash
   curl -XPOST 'http://localhost:9200/_logserver/prepareindex/alert' -u $USER:$PASSWORD
   ```
   
7. Migrate Agents index to new format (the next call will display the current status of the task):
   
   ```bash
   curl -XPOST 'http://localhost:9200/_logserver/prepareindex/.agents' -u $USER:$PASSWORD
   ```
   
8. Open `tcp/5603` port for Wikipedia plugin:
   ```bash
   firewall-cmd --zone=public --add-port=5603/tcp --permanent
   ```
   
   ```bash
   firewall-cmd --reload
   ```
