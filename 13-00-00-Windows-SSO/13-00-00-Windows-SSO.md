# Configuring Single Sign On (SSO) #
In order to configure SSO, the system should be accessible by domain name URL, not IP address nor localhost.

**Ok :**`https://loggui.com:5601/login`. **Wrong :** `https://localhost:5601/login`, `https://10.0.10.120:5601/login`

In order to enable SSO on your system follow below steps. The configuration is made for AD: `dev.example.com`, GUI URL: `loggui.com`
## Configuration steps ##
### Create an **User** Account for Elasticsearch auth plugin ###
In this step, a Kerberos Principal representing Elasticsearch auth plugin is created on the Active Directory. The principal name would be `name@DEV.EXAMPLE.COM`, while the `DEV.EXAMPLE.COM` is the administrative name of the realm. In our case, the principal name will be `esauth@DEV.EXAMPLE.COM`.

Create User in AD. Set "Password never expires" and "Other encryption options" as shown below:

![](/media/media/image107_js.png)

### Define Service Principal Name (SPN) and Create a Keytab file for it ###
Use the following command to create the keytab file and SPN:
> C:> ktpass -out c:\Users\Administrator\\**esauth.keytab** -princ **HTTP/loggui.com@DEV.EXAMPLE.COM** -mapUser **esauth** -mapOp set -pass '**Sprint$123**' -crypto ALL -pType KRB5_NT_PRINCIPAL

Values highlighted in bold should be adjusted for your system. The `esauth.keytab` file should be placed on your elasticsearch node - preferably `/etc/elasticsearch/` with read permissions for elasticsearch user: \
`chmod 640 /etc/elasticsearch/esauth.keytab` \
`chown elasticsearch: /etc/elasticsearch/esauth.keytab`

### Create a file named *krb5Login.conf*:
```
com.sun.security.jgss.initiate{
    com.sun.security.auth.module.Krb5LoginModule required
    principal="esauth@DEV.EXAMPLE.COM" useKeyTab=true
    keyTab=/etc/elasticsearch/esauth.keytab storeKey=true debug=true;
    };
com.sun.security.jgss.krb5.accept {
    com.sun.security.auth.module.Krb5LoginModule required
    principal="esauth@DEV.EXAMPLE.COM" useKeyTab=true
    keyTab=/etc/elasticsearch/esauth.keytab storeKey=true debug=true;
    };
```
Principal user and keyTab location should be changed as per the values created in the step 2. Make sure the domain is in UPPERCASE as shown above.
The `krb5Login.conf` file should be placed on your elasticsearch node, for instance `/etc/elasticsearch/` with read permissions for elasticsearch user:
```
sudo chmod 640 /etc/elasticsearch/krb5Login.conf
sudo chown elasticsearch: /etc/elasticsearch/krb5Login.conf
```
### Append the following JVM arguments (on Elasticsearch node in */etc/sysconfig/elasticsearch*):

> -Dsun.security.krb5.debug=true -Djava.security.krb5.realm=**DEV.EXAMPLE.COM** -Djava.security.krb5.kdc=**AD_HOST_IP_ADDRESS** -Djava.security.auth.login.config=**/etc/elasticsearch/krb5Login.conf** -Djavax.security.auth.useSubjectCredsOnly=false

Change the appropriate values in the bold. This JVM arguments has to be set for Elasticsearch server.

### Add the following additional (sso.domain, service_principal_name, service_principal_name_password) settings for ldap in elasticsearch.yml or properties.yml file wherever the ldap settings are configured:
```
sso.domain: "dev.example.com"
ldaps:
- name: "dev.example.com"
    host: "IP_address"
    port: 389                                                 # optional, default 389
    ssl_enabled: false                                        # optional, default    true
    ssl_trust_all_certs: false                                 # optional, default false
    bind_dn: "Administrator@dev.example.com"                     # optional, skip for anonymous bind
    bind_password: "administrator_password"                                 # optional, skip for anonymous bind
    search_user_base_DN: "OU=lab,DC=dev,DC=it,DC=example,DC=com"
    user_id_attribute: "uid"                                  # optional, default "uid"
    search_groups_base_DN: "OU=lab,DC=dev,DC=it,DC=example,DC=com"
    unique_member_attribute: "uniqueMember"                   # optional, default "uniqueMember"
    service_principal_name: "esauth@DEV.EXAMPLE.COM"
    service_principal_name_password : "Sprint$123"
```
Note: At this moment, SSO works for only single domain. So you have to mention for what domain SSO should work in the above property `sso.domain`

### To apply the changes restart Elasticsearch service
`sudo systemctl restart elasticsearch.service`

### Enable SSO feature  in `kibana.yml` file:

`kibana.sso_enabled: true` \
After that Kibana has to be restarted: \
`sudo systemctl restart kibana.service`

## Client (Browser) Configuration##

### Internet Explorer configuration

1. Goto `Internet Options` from `Tools` menu and click on `Security` Tab:

![](/media/media/image108.png)

2. Select `Local intranet`, click on `Site` -> `Advanced` -> `Add` the url:

![](/media/media/image109_js.png)

After adding the site click close.

3. Click on custom level and select the option as shown below:

![](/media/media/image110_js.png)

### Chrome configuration

For Chrome, the settings are taken from IE browser.

### Firefox configuration

Update the following config:

![](/media/media/image111_js.png)
