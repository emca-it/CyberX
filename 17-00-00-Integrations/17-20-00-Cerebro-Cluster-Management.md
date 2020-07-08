## Cerebro - elasticsearch web admin tool

### Software Requirements
1. Cerebro v0.8.4

		bash
		wget 'https://github.com/lmenezes/cerebro/releases/download/v0.8.4/cerebro-0.8.4.tgz'

1. Java 11+ [for basic-auth setup]

		bash
		yum install java-11-openjdk-headless.x86_64


1. Java 1.8.0 [without authorization]

		bash
		yum install java-1.8.0-openjdk-headless

### Firewall Configuration

	bash
	firewall-cmd --permanent --add-port=5602/tcp
	firewall-cmd --reload

### Cerebro Configuration

1. Extract archive & move directory

		bash
		tar -xvf cerebro-0.8.4.tgz -C /opt/
		mv /opt/cerebro-0.8.4/ /opt/cerebro

1. Add Cerebro service user

		bash
		useradd -M -d /opt/cerebro -s /sbin/nologin cerebro

1. Change Cerbero permissions

	bash
	chown -R cerebro:cerebro /opt/cerebro && chmod -R 700 /opt/cerebro

1. Install cerbero service

		[Unit]
		Description=Cerebro
		
		[Service]
		Type=simple
		User=cerebro
		Group=cerebro
		ExecStart=/opt/cerebro/bin/cerebro "-Dconfig.file=/opt/cerebro/conf/application.conf"
		Restart=always
		WorkingDirectory=/opt/cerebro
		
		[Install]
		WantedBy=multi-user.target


		bash
		cp cerebro.service /usr/lib/systemd/system/
		systemctl daemon-reload
		systemctl enable cerebro


1. Customize configuration file: /opt/cerebro/conf/application.conf

	- Authentication

			auth = {
			  type: basic
			    settings: {
			      username = "logserver"
			      password = "logserver"
			    }
			}

	- A list of known Elasticsearch hosts

			hosts = [
			  {
			    host = "https://192.168.3.11:9200"
			    name = "cyberx"
			    auth = {
			      username = "logserver"
			      password = "logserver"
			    }
			  }
			]

			play.ws.ssl {
			  trustManager = {
			    stores = [
			      { type = "PEM", path = "/etc/elasticsearch/ssl/rootCA.crt" }
			    ]
			  }
			} 
			play.ws.ssl.loose.acceptAnyCertificate=true


	- SSL access to cerebro

			http = {
			  port = "disabled"
			}
			https = {
			  port = "5602"
			}
			
			# SSL access to cerebro - no self signed certificates
			#play.server.https {
			#  keyStore = {
			#    path = "keystore.jks",
			#    password = "SuperSecretKeystorePassword"
			#  }
			#}
			
			#play.ws.ssl {
			#  trustManager = {
			#    stores = [
			#      { type = "JKS", path = "truststore.jks", password = "SuperSecretTruststorePassword"  }
			#    ]
			#  }
			#}


1. Start the service

		bash
		systemctl start cerebro
		goto: https://127.0.0.1:5602

### Optional:

1. Register backup/snapshot repository for Elasticsearch

		bash
		curl -k -XPUT "https://127.0.0.1:9200/_snapshot/backup?pretty" -H 'Content-Type: application/json' -d'
		{
		  "type": "fs",
		  "settings": {
		    "location": "/var/lib/elasticsearch/backup/"
		  }
		}' -u logserver:logserver


1. Login using curl/kibana

		bash
		curl -k -XPOST 'https://192.168.3.11:5602/auth/login' -H 'mimeType: application/x-www-form-urlencoded' -d 'user=logserver&password=logserver' -c cookie.txt
		curl -k -XGET 'https://192.168.3.11:5602' -b cookie.txt

