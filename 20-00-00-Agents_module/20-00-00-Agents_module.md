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
<td><p class="first last">OSS 6.8.13</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/filebeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-odd"><td><p class="first last">2</p>
</td>
<td><p class="first last">Packetbeat</p>
</td>
<td><p class="first last">OSS 6.8.13</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/packetbeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-even"><td><p class="first last">3</p>
</td>
<td><p class="first last">Winlogbeat</p>
</td>
<td><p class="first last">OSS 6.8.13</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/winlogbeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-odd"><td><p class="first last">4</p>
</td>
<td><p class="first last">Metricbeat</p>
</td>
<td><p class="first last">OSS 6.8.13</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/metricbeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-even"><td><p class="first last">5</p>
</td>
<td><p class="first last">Heartbeat</p>
</td>
<td><p class="first last">OSS 6.8.13</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/heartbeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-odd"><td><p class="first last">6</p>
</td>
<td><p class="first last">Auditbeat</p>
</td>
<td><p class="first last">OSS 6.8.13</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/auditbeat-oss-6-8-13</p>
</td>
</tr>
<tr class="row-even"><td><p class="first last">7</p>
</td>
<td><p class="first last">Logstash</p>
</td>
<td><p class="first last">OSS 6.8.13</p>
</td>
<td><p class="first last">https://www.elastic.co/downloads/past-releases/logstash-oss-6-8-13</p>
</td>
</tr>
</table>