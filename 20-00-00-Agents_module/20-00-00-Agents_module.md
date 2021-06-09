#  Agents module

The Agents module is used for the central management of agents used in Energy Logserver such as *Filebeat*, *Winlogbeat*, *Packetbeat*, *Metricbeat*.

## Component modules ##

The software consists of two modules:

- Agent Module - installation just like any standard Kibana plugin. 

- Agent software - installed on host with agent (like beats);

## Agent Module installation

All necessary components can be found in the installation folder `./install/Agents/masteragent`.

1. Go to installation directory:

```bash
cd ./install/Agents/masteragent
```

2. Generating the certificates:

   ```bash
   cd certificates/
   ```

   - set *DOMAIN* and *DOMAIN_IP* in scripts from `./certificates` directory:

     ```bash
     #!/bin/bash
     DOMAIN="localhost"
     DOMAIN_IP="10.4.3.185"
     ```

   - execute the scripts in the following order:

     ```bash
     ./1_rootca.sh
     ./2_clientcrt.sh
     ./3_createstore.sh
     ```

3. Install the required packages:

   ```bash
   yum install net-tools
   ```

4. Add an exception to the firewall to listen on TCP 8080 and 8081:
   ```bash
   firewall-cmd --permanent --zone public --add-port 8080/tcp
   firewall-cmd --permanent --zone public --add-port 8081/tcp
   ```

5. Logstash pipeline configuration:
   ```bash
   /bin/cp -rf ./logstash/agents_template.json /etc/logstash/templates.d/
   mkdir /etc/logstash/conf.d/masteragent
   /bin/cp -rf ./logstash/*.conf /etc/logstash/conf.d/masteragent/
   ```

   - Edit file  `/etc/logstash/pipelines.yml` by uncomment this line(be awer to this lines looks likes other uncomment lines. It's yml file.):

   ```yaml
   - pipeline.id: masteragent
     path.config: "/etc/logstash/conf.d/masteragent/*.conf
   ```
   - Logstash SSL configuration:

   ```bash
   mkdir /etc/logstash/conf.d/masteragent/ssl
   /bin/cp -rf ./certificates/domain.key /etc/logstash/conf.d/masteragent/ssl/
   /bin/cp -rf ./certificates/domain.crt /etc/logstash/conf.d/masteragent/ssl/
   /bin/cp -rf ./certificates/rootCA.crt /etc/logstash/conf.d/masteragent/ssl/
   chown -R logstash:logstash /etc/logstash
   ```

## Linux Agent installation

1. Copy necessary files to destination host:

   ```bash
   /bin/cp -rf ./install/Agents/masteragent/agents/linux/masteragent /opt/masteragent
   /bin/cp -rf ./install/Agents/masteragent/certificates/node_name.p12 /opt/masteragent
   /bin/cp -rf ./install/Agents/masteragent/certificates/root.jks /opt/masteragent
   /bin/cp -rf    ./install/Agents/masteragent/agents/linux/masteragent/masteragent.service /usr/lib/systemd/system/masteragent.service
   ```

2. Set correct IP address of Logstash and Kibana in  **/opt/masteragent/agent.conf** and verify paths for Filebeat, Metricbeat, etc. are correct.

   ```bash
   systemctl daemon-reload
   systemctl enable masteragent
   systemctl start masteragent
   ```

3. Restart logstash:

   ```bash
   systemctl restart logstash
   ```

4. In the GUI, in the **Agents** tab, you can check the status of the newly connected host.

## Windows Agent installation

1. Add an exception to the firewall to listen on **TCP** port **8081**.

2. Add an exception to the firewall enabling connection on **TCP LOGSTASH_IP:8080** port.

3. Copy  content of the **./agents/windows** from installation directory to  **"C:\Program Files\MasterAgnet"**

4. Change IP address of the Kibana GUI server and Logstash server in **"C:\Program Files\MasterAgnet\agent.conf"** file.

5. In order to install the service, start the console as an administrator and execute the following commands: 

    ```powershell
    cd "C:\Program Files\MasterAgent"
    agents.exe install
    agents.exe start
    ```

6. An alternative method of installing the service, run the PowerShell console as administrator and execute the following commands: 

   ```bash
   New-Service -name masteragent -displayName masteragent - binaryPathName "C:\Program Files\MasterAgent\agents.exe"
   ```

7. Check status of service via **services.msc** (if stoped, try start it agian).

8. In the GUI, in the **Agents** tab, you can check the status of the newly connected host.

## Agent module compatibility

The Agents module works with Beats agents in the following versions:

  <table border="1" class="docutils" id="id1">
<colgroup>
<col width="3%" />
<col width="12%" />
<col width="13%" />
<col width="71%" />
</colgroup>
<thead valign="bottom">
<tr class="row-odd"><th class="head">Nr</th>
<th class="head">Agent Name</th>
<th class="head">Beats Version</th>
<th class="head">Link to download</th>
</tr>
</thead>
<tbody valign="top">
<tr class="row-even"><td><p class="first last">1</p>
</td>
<td><p class="first last">Filebeat</p>
</td>
<td><p class="first last">OSS 6.8.14</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/filebeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-odd"><td><p class="first last">2</p>
</td>
<td><p class="first last">Packetbeat</p>
</td>
<td><p class="first last">OSS 6.8.14</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/packetbeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-even"><td><p class="first last">3</p>
</td>
<td><p class="first last">Winlogbeat</p>
</td>
<td><p class="first last">OSS 6.8.14</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/winlogbeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-odd"><td><p class="first last">4</p>
</td>
<td><p class="first last">Metricbeat</p>
</td>
<td><p class="first last">OSS 6.8.14</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/metricbeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-even"><td><p class="first last">5</p>
</td>
<td><p class="first last">Heartbeat</p>
</td>
<td><p class="first last">OSS 6.8.14</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/heartbeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-odd"><td><p class="first last">6</p>
</td>
<td><p class="first last">Auditbeat</p>
</td>
<td><p class="first last">OSS 6.8.14</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/auditbeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-even"><td><p class="first last">7</p>
</td>
<td><p class="first last">Logstash</p>
</td>
<td><p class="first last">OSS 6.8.14</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/logstash-oss-6-8-13</p>
</td>
</tr>
</table>



##  Beats agents installation

### Windows

#### Winlogbeat

##### Installation

1. Copy the Winlogbeat installer from the installation directory `install/Agents/beats/windows/winlogbeat-oss-6.8.14-windows-x86_64.zip` and unpack

2. Copy the installation files to the `C:\Program Files\Winlogbeat` directory

##### Configuration

Editing the file: `C:\Program Files\Winlogbeat\winlogbeat.yml`:

1. In section:

   ```yml
   winlogbeat.event_logs:
     - name: Application
       ignore_older: 72h
     - name: Security
     - name: System
   ```

   change to:

   ```yml
   winlogbeat.event_logs:
     - name: Application
       ignore_older: 72h
     - name: Security
       ignore_older: 72h
     - name: System
       ignore_older: 72h
   ```

2. In section:

   ```yml
   setup.template.settings:
     index.number_of_shards: 1
   ```

   change to:

   ```yml
   #setup.template.settings:
     #index.number_of_shards: 1
   ```

3. In section:

   ```yml
   setup.kibana:
   ```

   change to:

   ```yml
   #setup.kibana:
   ```

4. In section:

   ```yml
   output.elasticsearch:
     # Array of hosts to connect to.
     hosts: ["localhost:9200"]
   ```

   change to:

   ```yml
   #output.elasticsearch:
     # Array of hosts to connect to.
     #hosts: ["localhost:9200"]
   ```

5. In section:

   ```yml
   #output.logstash:
     # The Logstash hosts
     #hosts: ["localhost:5044"]
   ```

   change to:

   ```yml
   output.logstash:
     # The Logstash hosts
     hosts: ["LOGSTASH_IP:5044"]
   ```

6. In section:

   ```yml
   #tags: ["service-X", "web-tier"]
   ```

   change to:

   ```yml
   tags: ["winlogbeat"]
   ```

Run the `PowerShell` console as Administrator and execute the following commands:

```powershell
cd 'C:\Program Files\Winlogbeat'
.\install-service-winlogbeat.ps1

Security warning
Run only scripts that you trust. While scripts from the internet can be useful,
this script can potentially harm your computer. If you trust this script, use
the Unblock-File cmdlet to allow the script to run without this warning message.
Do you want to run C:\Program Files\Winlogbeat\install-service-winlogbeat.ps1?
[D] Do not run  [R] Run once  [S] Suspend  [?] Help (default is "D"): R

```

Output:

```powershell
Status   Name               DisplayName
------   ----               -----------
Stopped    Winlogbeat      Winlogbeat
```

Start Winlogbeat service:

```powershell
sc start Winlogbeat
```

Test configuration:

```cmd
cd 'C:\Program Files\Winlogbeat'
winlogbeat.exe test config
winlogbeat.exe test output
```

#### Filebeat

##### Installation

1. Copy the Filebeat installer from the installation directory `install/Agents/beats/windows/filebeat-oss-6.8.14-windows-x86_64.zip` and unpack

2. Copy the installation files to the `C:\Program Files\Filebeat` directory

##### Configuration

Editing the file: `C:\Program Files\Filebeat\filebeat.yml`:

1. In section:

   ```yml
   - type: log
   
     # Change to true to enable this input configuration.
     enabled: false
   ```

   change to:

   ```yml
   - type: log
   
     # Change to true to enable this input configuration.
     enabled: true
   ```

2. In section:

   ```yml
       paths:
       - /var/log/*.log
       #- c:\programdata\elasticsearch\logs\*
   ```

   change to:

   ```yml
   paths:
       #- /var/log/*.log
       #- c:\programdata\elasticsearch\logs\*
       - "C:\Program Files\Microsoft SQL Server\*\MSSQL\Log\*"
       - "C:\inetpub\logs\*""
   ```

3. In section:

   ```yml
   setup.template.settings:
     index.number_of_shards: 1
   ```

   change to:

   ```yml
   #setup.template.settings:
     #index.number_of_shards: 1
   ```

4. In section:

   ```yml
   setup.kibana:
   ```

   change to:

   ```yml
   #setup.kibana:
   ```

5. In section:

   ```yml
   output.elasticsearch:
     # Array of hosts to connect to.
     hosts: ["localhost:9200"]
   ```

   change to:

   ```yml
   #output.elasticsearch:
     # Array of hosts to connect to.
     #hosts: ["localhost:9200"]
   ```

6. In section:

   ```yml
   #output.logstash:
     # The Logstash hosts
     #hosts: ["localhost:5044"]
   ```

   change to:

   ```yml
   output.logstash:
     # The Logstash hosts
     hosts: ["LOGSTASH_IP:5044"]
   ```

   

7. In section:

   ```yml
   #tags: ["service-X", "web-tier"]
   ```

   change to:

   ```yml
   tags: ["filebeat"]
   ```

Run the `PowerShell` console as Administrator and execute the following commands:

```powershell
cd 'C:\Program Files\Filebeat'
.\install-service-filebeat.ps1

Security warning
Run only scripts that you trust. While scripts from the internet can be useful,
this script can potentially harm your computer. If you trust this script, use
the Unblock-File cmdlet to allow the script to run without this warning message.
Do you want to run C:\Program Files\Filebeat\install-service-filebeat.ps1?
[D] Do not run  [R] Run once  [S] Suspend  [?] Help (default is "D"): R

```

Output:

```powershell
Status   Name               DisplayName
------   ----               -----------
Stopped  Filebeat        Filebeat
```

Start Filebeat service:

```powershell
sc start filebeat
```

You can enable, disable and list Filebeat modules using the following command:

```cmd
cd 'C:\Program Files\Filebeat'
filebeat.exe modules list
filebeat.exe modules apache enable
filebeat.exe modules apache disable
```

Test configuration:

```cmd
cd 'C:\Program Files\Filebeat'
filebeat.exe test config
filebeat.exe test output
```

#### Merticbeat

##### Installation

1. Copy the Merticbeat installer from the installation directory `install/Agents/beats/windows/merticbeat-oss-6.8.14-windows-x86_64.zip` and unpack

2. Copy the installation files to the `C:\Program Files\Merticbeat` directory

##### Configuration

Editing the file: `C:\Program Files\Merticbeat\metricbeat.yml`:

1. In section:

   ```yml
   setup.template.settings:
     index.number_of_shards: 1
     index.codec: best_compression
   ```

   change to:

   ```yml
   #setup.template.settings:
     #index.number_of_shards: 1
     #index.codec: best_compression
   ```

2. In section:

   ```yml
   setup.kibana:
   ```

   change to:

   ```yml
   #setup.kibana:
   ```

3. In section:

   ```yml
   output.elasticsearch:
     # Array of hosts to connect to.
     hosts: ["localhost:9200"]
   ```

   change to:

   ```yml
   #output.elasticsearch:
     # Array of hosts to connect to.
     #hosts: ["localhost:9200"]
   ```

4. In section:

   ```yml
   #output.logstash:
     # The Logstash hosts
     #hosts: ["localhost:5044"]
   ```

   change to:

   ```yml
   output.logstash:
     # The Logstash hosts
     hosts: ["LOGSTASH_IP:5044"]
   ```

5. In section:

   ```yml
   #tags: ["service-X", "web-tier"]
   ```

   change to:

   ```yml
   tags: ["metricbeat"]
   ```

Run the `PowerShell` console as Administrator and execute the following commands:

```powershell
cd 'C:\Program Files\Metricbeat'
.\install-service-metricbeat.ps1

Security warning
Run only scripts that you trust. While scripts from the internet can be useful,
this script can potentially harm your computer. If you trust this script, use
the Unblock-File cmdlet to allow the script to run without this warning message.
Do you want to run C:\Program Files\Metricbeat\install-service-metricbeat.ps1?
[D] Do not run  [R] Run once  [S] Suspend  [?] Help (default is "D"): R

```

Output:

```powershell
Status   Name               DisplayName
------   ----               -----------
Stopped  Metricbeat        Metricbeat
```

Start Filebeat service:

```powershell
sc start metricbeat
```

You can enable, disable and list Metricbeat modules using the following command:

```cmd
cd 'C:\Program Files\Metricbeat'
metricbeat.exe modules list
metricbeat.exe modules apache enable
metricbeat.exe modules apache disable
```

Test configuration:

```cmd
cd 'C:\Program Files\Metricbeat'
metricbeat.exe test config
metricbeat.exe test output
```

#### Packetbeat

##### Installation

1. Copy the Packetbeatinstaller from the installation directory `install/Agents/beats/windows/packetbeat-oss-6.8.14-windows-x86_64.zip` and unpack

2. Copy the installation files to the `C:\Program Files\Packetbeat` directory

##### Configuration

Editing the file: `C:\Program Files\Packetbeat\packetbeat.yml`:

1. In section:

   ```yml
   setup.template.settings:
     index.number_of_shards: 3
   ```

   change to:

   ```yml
   #setup.template.settings:
     #index.number_of_shards: 3
   ```

2. In section:

   ```yml
   setup.kibana:
   ```

   change to:

   ```yml
   #setup.kibana:
   ```

3. In section:

   ```yml
   output.elasticsearch:
     # Array of hosts to connect to.
     hosts: ["localhost:9200"]
   ```

   change to:

   ```yml
   #output.elasticsearch:
     # Array of hosts to connect to.
     #hosts: ["localhost:9200"]
   ```

4. In section:

   ```yml
   #output.logstash:
     # The Logstash hosts
     #hosts: ["localhost:5044"]
   ```

   change to:

   ```yml
   output.logstash:
     # The Logstash hosts
     hosts: ["LOGSTASH_IP:5044"]
   ```

5. In section:

   ```yml
   #tags: ["service-X", "web-tier"]
   ```

   change to:

   ```yml
   tags: ["packetbeat"]
   ```

Run the `PowerShell` console as Administrator and execute the following commands:

```powershell
cd 'C:\Program Files\\Packetbeat'
.\install-service-packetbeat.ps1

Security warning
Run only scripts that you trust. While scripts from the internet can be useful,
this script can potentially harm your computer. If you trust this script, use
the Unblock-File cmdlet to allow the script to run without this warning message.
Do you want to run C:\Program Files\Packetbeat\install-service-packetbeat.ps1?
[D] Do not run  [R] Run once  [S] Suspend  [?] Help (default is "D"): R

```

Output:

```powershell
Status   Name               DisplayName
------   ----               -----------
Stopped  Packetbeat        Packetbeat
```

Start Packetbeat service:

```powershell
sc start packetbeat
```

Test configuration:

```cmd
cd 'C:\Program Files\Packetbeat'
packetbeat.exe test config
packetbeat.exe test output
```

### Linux

#### Filebeat

##### Installation

1. Copy the Filebeat installer from the installation directory `install/Agents/beats/linux/filebeat-oss-6.8.14-x86_64.rpm`

2. Install filebeat with following commadn:

   ```bash
   yum install -y filebeat-oss-6.8.14-x86_64.rpm
   ```

##### Configuration

Editing the file: `/etc/filebeat/filebeat.yml`:

1. In section:

   ```yml
   - type: log
   
     # Change to true to enable this input configuration.
     enabled: false
   ```

   change to:

   ```yml
   - type: log
   
     # Change to true to enable this input configuration.
     enabled: true
   ```

   

2. In section:

   ```yml
   setup.template.settings:
     index.number_of_shards: 1
   ```

   change to:

   ```yml
   #setup.template.settings:
     #index.number_of_shards: 1
   ```

3. In section:

   ```yml
   setup.kibana:
   ```

   change to:

   ```yml
   #setup.kibana:
   ```

4. In section:

   ```yml
   output.elasticsearch:
     # Array of hosts to connect to.
     hosts: ["localhost:9200"]
   ```

   change to:

   ```yml
   #output.elasticsearch:
     # Array of hosts to connect to.
     #hosts: ["localhost:9200"]
   ```

5. In section:

   ```yml
   #output.logstash:
     # The Logstash hosts
     #hosts: ["localhost:5044"]
   ```

   change to:

   ```yml
   output.logstash:
     # The Logstash hosts
     hosts: ["LOGSTASH_IP:5044"]
   ```

6. In section:

   ```yml
   #tags: ["service-X", "web-tier"]
   ```

   change to:

   ```yml
   tags: ["filebeat"]
   ```

Start Filebeat service:

```bash
systemctl start filebeat
```

You can enable, disable and list Filebeat modules using the following command:

```bash
filebeat modules list
filebeat modules apache enable
filebeat modules apache disable
```

Test configuration:

```bash
filebeat test config
filebeat test output
```

#### Merticbeat

##### Installation

1. Copy the Merticbeatinstaller from the installation directory `install/Agents/beats/linux/metricbeat-oss-6.8.14-x86_64.rpm`

2. Install Merticbeat with following command:

   ```bash
   yum install -y metricbeat-oss-6.8.14-x86_64.rpm
   ```

##### Configuration

Editing the file: `/etc/metricbeat/metricbeat.yml`:

1. In section:

   ```yml
   setup.template.settings:
     index.number_of_shards: 1
     index.codec: best_compression
   ```

   change to:

   ```yml
   #setup.template.settings:
     #index.number_of_shards: 1
     #index.codec: best_compression
   ```

2. In section:

   ```yml
   setup.kibana:
   ```

   change to:

   ```yml
   #setup.kibana:
   ```

3. In section:

   ```yml
   output.elasticsearch:
     # Array of hosts to connect to.
     hosts: ["localhost:9200"]
   ```

   change to:

   ```yml
   #output.elasticsearch:
     # Array of hosts to connect to.
     #hosts: ["localhost:9200"]
   ```

4. In section:

   ```yml
   #output.logstash:
     # The Logstash hosts
     #hosts: ["localhost:5044"]
   ```

   change to:

   ```yml
   output.logstash:
     # The Logstash hosts
     hosts: ["LOGSTASH_IP:5044"]
   ```

5. In section:

   ```yml
   #tags: ["service-X", "web-tier"]
   ```

   change to:

   ```yml
   tags: ["metricbeat"]
   ```

Start Filebeat service:

```powershell
systemctl start metricbeat
```

You can enable, disable and list Metricbeat modules using the following command:

```cmd
metricbeat modules list
metricbeat modules apache enable
metricbeat modules apache disable
```

Test configuration:

```cmd
metricbeat test config
metricbeat test output
```

#### Packetbeat

##### Installation

1. Copy the Packetbeat installer from the installation directory `install/Agents/beats/linux/packetbeat-oss-6.8.14-x86_64.rpm`

2. Install Packetbeatwith following command:

   ```bash
   yum install -y packetbeat-oss-6.8.14-x86_64.rpm
   ```

##### Configuration

Editing the file: `/etc/packetbeat/packetbeat.yml`:

1. In section:

   ```yml
   setup.template.settings:
     index.number_of_shards: 3
   ```

   change to:

   ```yml
   #setup.template.settings:
     #index.number_of_shards: 3
   ```

2. In section:

   ```yml
   setup.kibana:
   ```

   change to:

   ```yml
   #setup.kibana:
   ```

3. In section:

   ```yml
   output.elasticsearch:
     # Array of hosts to connect to.
     hosts: ["localhost:9200"]
   ```

   change to:

   ```yml
   #output.elasticsearch:
     # Array of hosts to connect to.
     #hosts: ["localhost:9200"]
   ```

4. In section:

   ```yml
   #output.logstash:
     # The Logstash hosts
     #hosts: ["localhost:5044"]
   ```

   change to:

   ```yml
   output.logstash:
     # The Logstash hosts
     hosts: ["LOGSTASH_IP:5044"]
   ```

5. In section:

   ```yml
   #tags: ["service-X", "web-tier"]
   ```

   change to:

   ```yml
   tags: ["packetbeat"]
   ```

Start Packetbeat service:

```powershell
servicectl start packetbeat
```

Test configuration:

```cmd
packetbeat test config
packetbeat test output
```