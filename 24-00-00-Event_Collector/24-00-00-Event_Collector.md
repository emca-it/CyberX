# Event Collector

The Event Collector allows to get events from remote Windows computers and store them in the Energy Logserver indexes. The destination log path for the events is a property of the subscription. The Energy Logserver Event Collector allows to define an event subscription on an Energy Logserver collector without defining the event source computers. Multiple remote event source computers can then be set up (using for example a group policy setting) to forward events to the Energy Logserver. The Event Collector don't require installation of any additional applications/agents on Windows source hosts.

## Configuration steps

### Installation of Event Collector

```bash
tar zxf wec_7x-master.tar.gz -C /opt/
mkdir /opt/wec
mv /opt/wec_7x-master/ /opt/wec/
mkdir /etc/wec
cp /opt/wec/sub_manager/config.yaml /etc/wec/config.yaml
```

### Generate certificate

```bash
cd /opt/wec_7x-master/certgen
vim server-certopts.cnf
```

- Set `DNS.1` and `IP.1` for WEC server:

  ```bash
  [req]
  default_bits = 4096
  default_md = sha256
  req_extensions = req_ext
  keyUsage = keyEncipherment,dataEncipherment
  basicConstraints = CA:FALSE
  distinguished_name = dn
  
  [ req_ext ]
  subjectAltName = @alt_names
  extendedKeyUsage = serverAuth,clientAuth
  
  [ alt_names ]
  DNS.1 = wec.local.domain
  IP.1 = 192.168.13.163
  
  [dn]
  ```

- Set `DNS.1` and `IP.1` for client certificate:

  ```bash
  [req]
  default_bits = 4096
  default_md = sha256
  req_extensions = req_ext
  keyUsage = keyEncipherment,dataEncipherment
  basicConstraints = CA:FALSE
  distinguished_name = dn
  
  [ req_ext ]
  subjectAltName = @alt_names
  extendedKeyUsage = serverAuth,clientAuth
  
  [ alt_names ]
  DNS.1 = *local.domain
  
  [dn]
  ```

- Generate the CA certificate and private key, next check fingerprint:

  ```bash
  openssl genrsa -out ca.key 4096
  openssl req -x509 -new -nodes -key ca.key -days 3650 -out ca.crt -subj '/CN=wec.local.domain/O=example.com/C=CA/ST=QC/L=Montreal'
  openssl x509 -in ca.crt -fingerprint -sha1 -noout | sed -e 's/\://g' > ca.fingerprint
  ```

- Generate the client certificate and export it together with the CA in PFX format to be imported into the Windows certificate store:

  ```bash
  openssl req -new -newkey rsa:4096 -nodes -out server.csr -keyout server.key -subj '/CN=wec.local.domain/O=example.com/C=CA/ST=QC/L=Montreal'
  openssl x509 -req -in server.csr -out server.crt -CA ca.crt -CAkey ca.key -CAcreateserial -extfile server-certopts.cnf -extensions req_ext -days 365
  ```

- Generate the server certificate to be used by the WEC:

  ```bash
  openssl req -new -newkey rsa:4096 -nodes -out client.csr -keyout client.key -subj '/CN=wec.local.domain/O=example.com/C=CA/ST=QC/L=Montreal'
  openssl x509 -req -in client.csr -out client.crt -CA ca.crt -CAkey ca.key -CAcreateserial -extfile client-certopts.cnf -extensions req_ext -days 365
  openssl pkcs12 -export  -inkey client.key -in client.crt -certfile ca.crt -out client.p12
  ```

### Event Collector Configuration

- Copy server certificate and server key to Event Collector installation directory:

  ```bash
  cp server.crt server.key /opt/wec/sub_manager/certificates/
  ```

- Edit configuration file `config.yaml`

  ```bash
  vim /etc/wec/config.yaml
  ```

  - set the following options:

  ```bash
  external_host: wec.local.domain
  #check ca.fingerprint file
  ca_fingerprint: 97DDCD6F3AFA511EED5D3312BC50D194A9C9FA9A
  certificate: /opt/wec/sub_manager/certificates/server.crt
  key: /opt/wec/sub_manager/certificates/server.key
  
  ```

  - set the output for Event Collector to Logstash forwarding:

  ```bash
  remote_syslog:
      # forward events to remote syslog server
      address: 192.168.13.170
      port: 5614
  ```

  - set the output  to saving events to local file:

  ```bash
  outputfile: /var/log/wec/events-{:%Y-%d-%m}.log
  ```

  - disable local syslog output:

  ```bash
  local_syslog: false
  ```

  - set the filter section:

  ```bash
  filters:
            # source list
  		
  		- source: 'Security'
            filter: '*[System[(Level=1 or Level=2 or Level=3 or Level=4 or Level=0 or Level=5) and (EventID=4672 or EventID=4624 or EventID=4634)]]'
  	    
  	    - source: 'Application'
            filter: '*[System[(Level=1 or Level=2 or Level=3 or Level=4 or Level=0 or Level=5)]]'
          
          - source: 'System'
            filter: '*[System[(Level=1 or Level=2 or Level=3 or Level=4 or Level=0 or Level=5)]]'
  ```


### Install dependencies

1. Python 3.8 installation:

   ```bash
   sudo yum -y update
   sudo yum -y groupinstall "Development Tools"
   sudo yum -y install openssl-devel bzip2-devel libffi-devel
   sudo yum -y install wget
   wget https://www.python.org/ftp/python/3.8.3/Python-3.8.3.tgz
   tar xvf Python-3.8.3.tgz
   cd Python-3.8*/
   ./configure --enable-optimizations
   sudo make altinstall
   python3.8 --version
   ```

2. Python requirements installation:

   ```bash
   pip3.8 install PyYAML
   pip3.8 install sslkeylog
   ```

### Running Event Collector service

   ```bash
   vim /etc/sysconfig/wec.service
   ```

   ```bash
   
   [Unit]
   Description=WEC Service
   After=network.target
   
   [Service]
   Type=simple
   ExecStart=/usr/local/bin/python3.8 /opt/wec/sub_manager/run.py -c /etc/wec/config.yaml
   Restart=on-failure
   RestartSec=42s
   StandardOutput=syslog
   StandardError=syslog
   SyslogIdentifier=wecservice
   
   [Install]
   WantedBy=multi-user.target
   
   ```

   ```bash
   systemctl daemon-reload
   systemctl start wc
   ```



### Windows host configuration

1. Open the `Microsoft Management Console (mmc.exe)`, select `File -> Add/Remove Snap-ins`, and add the `Certificates` snap-in.

2. Select `Computer Account`.

3. Right-click the `Personal` node, and select `All Tasks > Import`.

4. Find and select the client certificate (client.p12) and import this file.

5. The `PKCS #12` archive contains the CA certificate as well. 

6. Move the CA certificate  to the `Trusted Root Certification Authorities` node after the import.

7. Give `NetworkService` access to the private key  file of the client authentication certificate:

8. To forward security logs:
   - In `CompMgmt.msc`, under `Local Users and Groups`, click `Groups > Event Log Readers` to open `Event Log Readers Properties`.
   - Add the "NETWORK SERVICE" account to the `Event Log Readers group`.

9. Make sure collector server is reachable from windows machine

10. Run `winrm qc` and accept changes on windows machine

11. Run `winrm set winrm/config/client/auth @{Certificate="true"}` on windows machint to enable certificate authentication

12. Open `gpedit.msc`

13. Under the `Computer Configuration node`, expand the `Administrative Templates` node, then expand the `Windows Components` node, and then select the `Event Forwarding` node.

14. Select the **SubscriptionManagers** setting and enable it. Click the **Show** button to add a subscription (use the CA thumbprint you saved earlier):

    ```cmd
    Server=https://<FQDN of the collector>:5986/wsman/SubscriptionManager/WEC,Refresh=<Refresh interval in seconds>,IssuerCA=<Thumbprint of the root CA>
    ```

    For example:

    ```cmd
    Server=HTTPS://logserver.diplux.com:5986/wsman/SubscriptionManager/WEC,Refresh=60,IssuerCA=549A72B56560A5CAA392078D9C38B52458616D2
    5
    ```

    NOTE: If you wish to set up multiple subscriptions because you want to forward Windows events to multiple event collectors (such as WEC), then you can do that here.

15. Run the `cmd` console with administrative privileges and make following command

    ```cmd
    gpupdate /force
    ```

### Logstash pipeline configuration

```bash
mkdir /etc/logstash/conf.d/syslog_wec
vim /etc/logstash/conf.d/syslog_wec/syslog_wec.conf
```

```bash
input {
        tcp {
                port => 5614
        }
}
 filter {
      grok {
        overwrite => ["message"]
        match => { "message" => [ "<%{INT:syslog_pri}>%{GREEDYDATA:message}" ] }
      }
      syslog_pri {}
      xml {
        source => "message"
        target => "windata"
      }
    }
output {
      elasticsearch {
                hosts => ["localhost:9200"]
                user => "logserver"
                password => "logserver"
                index => "syslog_wec-%{+YYYY.MM.dd}"
        }
     file {
        path => "/var/log/logstash/syslog.out"
        codec => line {
                        format => "%{message}"
                }
    }
}

```

```bash
vim /etc/logstash/pipeline.yml
```

```bash
- pipeline.id: syslog_wec
  path.config: "/etc/logstash/conf.d/syslog_wec/*.conf"
```

### Building the subscription filter

1. Browse to Event Viewer

2. Right click **Subscriptions** and **create subscription**

3. Click on Select Events and choose the type of logs that you want, for example: Event Level, Event Logs, Include Exclude Event ID, Keyword, etc.

4. Switch to XML view tab;

5. Copy the value of the `Select Path` key, for example:

   ```xml
   <QueryList>
     <Query Id="0" Path="Security">
       <Select Path="Security">*[System[(Level=1  or Level=2 or Level=3) and (EventID=4672 or EventID=4624 or EventID=4634)]]</Select>
     </Query>
   </QueryList>
   ```

   string to copy:

   ```xml
   *[System[(Level=1  or Level=2 or Level=3) and (EventID=4672 or EventID=4624 or EventID=4634)]]
   ```

6. Paste the above definition into the Event Collector configuration file in `filters` section:

   ```bash
   vim /etc/wec/config.yaml
   ```

   ```yaml
         filters:
           - source: 'Security'
             filter: '*[System[(Level=1  or Level=2 or Level=3) and (EventID=4672 or EventID=4624 or EventID=4634)]]'
   
   ```
   
   Restart Event Collector service 
   
   ```bash
   systemctl restart wec
   ```
   
   