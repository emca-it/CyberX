# Alert Module #

CyberX allows you to create alerts, i.e. monitoring
queries. These are constant queries that run in the background and
when the conditions specified in the alert are met, the specify action
is taken. 

![](/media/media/image91.PNG)

For example, if you want to know when more than 20
„status:500" response code from on our homepage appear within an one
hour, then we create an alert that check the number of occurrences of
the „status:500" query for a specific index every 5 minutes. If the
condition we are interested in is met, we send an action in the form
of sending a message to our e-mail address. In the action, you can
also set the launch of any script.

## Enabling the Alert Module ##

To enabling the alert module you should:

- generate writeback index for Alert service:

Only applies to versions 6.1.5 and older. From version 6.1.6 and later, the Alert index is created automatically

		/opt/alert/bin/elastalert-create-index --config /opt/alert/config.yaml

- configure the index pattern for alert*

![](/media/media/image96.PNG)

- start the Alert service:
  
		systemctl start alert

## SMTP server configuration

To configuring STMP server for email notification you should:

- edit `/opt/alert/config.yml` and add the following section:

  ```bash
  # email conf
  smtp_host: "mail.example.conf"
  smtp_port: 587
  smtp_ssl: false
  from_addr: "siem@example.com"
  smtp_auth_file: "/opt/alert/smtp_auth_file.yml"
  ```

- add the new `/opt/alert/smtp_auth_file.yml` file:

  ```bash
  user: "user"
  password: "password"
  ```

- restart `alert` service:

  ```bash
  systemctl restat alert
  ```

## Creating Alerts ##

To create the alert, click the "Alerts" button from the main menu bar.

 ![](/media/media/image93.png)

We will display a page with tree tabs: Create new alerts in „Create
alert rule", manage alerts in „Alert rules List" and check alert
status „Alert Status".

 In the alert creation windows we have an alert creation form:

 ![](/media/media/image92.PNG)

 - **Name** - the name of the alert, after which we will recognize and
search for it.
- **Index pattern** - a pattern of indexes after which the alert will be
searched.
- **Role** - the role of the user for whom an alert will be available
- **Type** - type of alert
- **Description** - description of the alert.
- **Example** - an example of using a given type of alert. Descriptive
field
- **Alert method** - the action the alert will take if the conditions are
met (sending an email message or executing a command)
- **Any** - additional descriptive field.# List of Alert rules #

The "Alert Rule List" tab contain complete list of previously created 
alert rules:

![](/media/media/image94.png)

In this window, you can activate / deactivate, delete and update alerts 
by clicking on the selected icon with the given alert: ![](/media/media/image63.png).
## Alerts status ##

In the "Alert status" tab, you can check the current alert status: if
it activated, when it started and when it ended, how long it lasted,
how many event sit found and how many times it worked.

![](/media/media/image95.png)

Also, on this tab, you can recover the alert dashboard, by clicking the "Recovery Alert Dashboard" button.

## Alerts status ##

In the "Alert status" tab, you can check the current alert status: if
it activated, when it started and when it ended, how long it lasted,
how many event sit found and how many times it worked.

![](/media/media/image95.png)

Also, on this tab, you can recover the alert dashboard, by clicking the "Recovery Alert Dashboard" button.

## Alert Types

The various Rule Type classes, defined in the CyberX.
An instance is held in memory for each rule, passed all of the data returned by querying Elasticsearch 
with a given filter, and generates matches based on that data.

### Any

The any rule will match everything. Every hit that the query returns will generate an alert.

### Blacklist

The blacklist rule will check a certain field against a blacklist, and match if it is in the blacklist.

### Whitelist

Similar to blacklist, this rule will compare a certain field to a whitelist, and match if the list does not contain the term.

### Change

This rule will monitor a certain field and match if that field changes.

### Frequency

This rule matches when there are at least a certain number of events in a given time frame.

### Spike

This rule matches when the volume of events during a given time period is spike_height times larger or smaller than during the previous time period.

### Flatline

This rule matches when the total number of events is under a given threshold for a time period.

### New Term

This rule matches when a new value appears in a field that has never been seen before.

### Cardinality

This rule matches when a the total number of unique values for a certain field within
a time frame is higher or lower than a threshold.

### Metric Aggregation

This rule matches when the value of a metric within the calculation window is higher or lower than a threshold.

### Percentage Match

This rule matches when the percentage of document in the match bucket within a calculation window is higher or lower than a threshold.

### Unique Long Term

This rule matches when there are values of compare_key in each checked timeframe.

### Find Match

Rule match when in defined period of time, two correlated documents match certain strings.

### Consecutive Growth

Rule matches for value difference between two aggregations calculated for different periods in time.

### Logical

Rule matches when a complex, logical criteria is met. Rule can be use for alert data correlation.

An example of using the Logical rule type.

![](/media/media/image149.png)

Alerts that must occur for the rule to be triggered:

- Switch - Port is off-line - the alert must appear 5 times.
  - OR
- Switch - Port is on-line - the alert must appear 5 times.

If both of the above alerts are met within no more than 5 minutes and the values of the "port_number" field are related to each other, the alert rule is triggered. It is possible to use logical connectives such as: OR, AND, NOR, NAND, XOR.

### Chain

Rule matches when a complex, logical criteria is met. Rule can be use for alert data correlation.

An example of using the Chain rule type.

![](/media/media/image148.png)

Alerts that must occur for the rule to be triggered:

- Linux - Login Failure - the alert must appear 10 times.
- AND
- Linux - Login Success - 1 time triggered alert.

If the sequence of occurrence of the above alerts is met within 5 minutes and the values of the "username" field are related to each other, the alert rule is triggered. The order in which the component alerts occur is important.

### Difference

This rule calculates percentage difference between aggregations for two non-overlapping time windows.

Let’s assume x represents the current time (i.e. when alert rule is run) then the relation between historical and present time windows is described by the inequality:

```
<x – agg_min – delta_min; x – delta_min> <= <x – agg_min; x>; where x – delta_min <= x – agg_min => delta_min >= agg_min
```

The percentage difference is then described by the following equation:

```
d = | avg_now – avg_history | / max(avg_now, avg_history) * 100; for (avg_now – avg_history != 0; avg_now != 0; avg_history != 0)
d = 0; (in other cases)
```

`avg_now` is the arithmetic mean of `<x – agg_min; x>`
`avg_history` is the arithmetic mean of `<x – agg_min – delta_min; x – delta_min>`

Required parameters:

- Enable the rule by setting type field.
  `type: difference`
- Based on the compare_key field aggregation is calculated.
  `compare_key: value`
- An alert is triggered when the percentage difference between aggregations is higher than the specified value.
  `threshold_pct: 10`
- The difference in minutes between calculated aggregations.
  `delta_min: 3`
- Aggregation bucket (in minutes).
  `agg_min: 1`

Optional parameters:

If present, for each unique `query_key` aggregation is calculated (it needs to be of type keyword).
`query_key: hostname`

## Alert Methods

When the alert rule is fulfilled, the defined action is performed - the alert method.
The following alert methods have been predefined in the system:
- email;
- commands;
- user;

### Email 

Method that sends information about an alert to defined email addresses.

### User

Method that sends information about an alert to defined system users.

### Command

 A method that performs system tasks. For example, it triggers a script that creates a new event in the customer ticket system.

Below is an example of an alert rule definition that uses the "command" alert method to create and recover an ticket in the client's ticket system:

```bash
index: op5-*
name: change-op5-hoststate
type: change

compare_key: hoststate
ignore_null: true
query_key: hostname

filter:
- query_string:
    query: "_exists_: hoststate AND datatype: \"HOSTPERFDATA\" AND _exists_: hostname"

realert:
  minutes: 0
alert: "command"
command: ["/opt/alert/send_request_change.sh", "5", "%(hostname)s", "SYSTEM_DOWN", "HOST", "Application Collection", "%(hoststate)s", "%(@timestamp)s"]
```

The executed command has parameters which are the values of the fields of the executed alert. Syntax: `%(fields_name)`.

- ### The Hive

  The alert module can forward information about the alert to *Security Incident Response Platform* **TheHive**.

  The configuration of the **Hive Alert** should be done in the definition of the `Rule Definition` alert using the following options:

  - `hive_alert_config_type: classic` - allows the use of variables to build The Hive alert
  - `hive_alert_config`:
    - `title` (text) : title of the alert (ignored in `classic` config type)
    - `description` (text) : description of the alert (ignored in `classic` config type)
    - `severity` (number) : severity of the alert (1: low; 2: medium; 3: high) **default=2**
    - `date` (date) : date and time when the alert was raised **default=now**
    - `tags` (multi-string) : case tags **default=empty**
    - `tlp` (number) : [TLP](https://www.us-cert.gov/tlp) (`0`: `white`; `1`: `green`; `2: amber`; `3: red`) **default=2**
    - `status` (AlertStatus) : status of the alert (*New*, *Updated*, *Ignored*, *Imported*) **default=New**
    - `type` (string) : type of the alert (read only)
    - `source` (string) : source of the alert (read only)
    - `sourceRef` (string) : source reference of the alert (read only)
    - `artifacts` (multi-artifact) : artifact of the alert. It is a array of JSON object containing artifact attributes **default=empty**
    - `follow` (boolean) : if true, the alert becomes active when updated **default=true**
  - `hive_observable_data_mapping` - mapping field values to the The Hive alert.

  **Note:** When use: `hive_alert_config_type: classic` the following parameters are ignored:

  ```yaml
  hive_alert_config:
      title: title of the alert
      description: description of the alert
  ```

  and you should use:

  ```yaml
  alert_subject: "title of the alert"
  alert_text: "description of the alert"
  ```

Example of configuration:

```bash
hive_alert_config_type: classic

hive_alert_config:
  type: 'test'
  source: 'elastalert-{rule[name]}'
  severity: 3
  tags: ['malicious behavior']
  tlp: 2
  status: 'New'
  follow: True

hive_observable_data_mapping:
  - ip: "{match[field1]}"
  - source: "{match[field2]}"
```

## Alert Content

There are several ways to format the body text of the various types of events. In EBNF::

    rule_name           = name
    alert_text          = alert_text
    ruletype_text       = Depends on type
    top_counts_header   = top_count_key, ":"
    top_counts_value    = Value, ": ", Count
    top_counts          = top_counts_header, LF, top_counts_value
    field_values        = Field, ": ", Value

Similarly to ``alert_subject``, ``alert_text`` can be further formatted using standard Python formatting syntax.
The field names whose values will be used as the arguments can be passed with ``alert_text_args`` or ``alert_text_kw``.
You may also refer to any top-level rule property in the ``alert_subject_args``, ``alert_text_args``, ``alert_missing_value``, and ``alert_text_kw fields``.  However, if the matched document has a key with the same name, that will take preference over the rule property.

By default::

    body                = rule_name
    
                          [alert_text]
    
                          ruletype_text
    
                          {top_counts}
    
                          {field_values}

With ``alert_text_type: alert_text_only``::

    body                = rule_name
    
                          alert_text

With ``alert_text_type: exclude_fields``::

    body                = rule_name
    
                          [alert_text]
    
                          ruletype_text
    
                          {top_counts}

With ``alert_text_type: aggregation_summary_only``::

    body                = rule_name
    
                          aggregation_summary

ruletype_text is the string returned by RuleType.get_match_str.

field_values will contain every key value pair included in the results from Elasticsearch. These fields include "@timestamp" (or the value of ``timestamp_field``),
every key in ``include``, every key in ``top_count_keys``, ``query_key``, and ``compare_key``. If the alert spans multiple events, these values may
come from an individual event, usually the one which triggers the alert.

When using ``alert_text_args``, you can access nested fields and index into arrays. For example, if your match was ``{"data": {"ips": ["127.0.0.1", "12.34.56.78"]}}``, then by using ``"data.ips[1]"`` in ``alert_text_args``, it would replace value with ``"12.34.56.78"``. This can go arbitrarily deep into fields and will still work on keys that contain dots themselves.

## Example of rules ##

### Unix - Authentication Fail ###

- index pattern: 

		syslog-*

- Type:

		Frequency

- Alert Method:

		Email

- Any:

		num_events: 4
		timeframe:
		  minutes: 5
		
		filter:
		- query_string:
		    query: "program: (ssh OR sshd OR su OR sudo) AND message: \"Failed password\""

### Windows - Firewall disable or modify ###

- index pattern: 

		beats-*

- Type:

		Any

- Alert Method:

		Email

- Any:

filter:

		- query_string:
		       query: "event_id:(4947 OR 4948 OR 4946 OR 4949 OR 4954 OR 4956 OR 5025)"

### SIEM Rules

Beginning with version 6.1.7, the following SIEM rules are delivered with the product.

````md
| Nr. | Architecture/Application | Rule Name                                                    | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Index name   | Requirements                                     | Source                       | Time definition            | Threashold |
|-----|--------------------------|--------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|--------------------------------------------------|------------------------------|----------------------------|------------|
| 1   | Windows                  | Windows - Admin night logon                                  | Alert on Windows login events when detected outside business hours                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 2   | Windows                  | Windows - Admin task as user                                 | Alert when admin task is initiated by regular user.  Windows event id 4732 is verified towards static admin list. If the user does not belong to admin list AND the event is seen than we generate alert. Static Admin list is a logstash  disctionary file that needs to be created manually. During Logstash lookup a field user.role:admin is added to an event.4732: A member was added to a security-enabled local group                                                                                                                                                                                                                                       | winlogbeat-* | winlogbeatLogstash admin dicstionary lookup file | Widnows Security Eventlog    | Every 1min                 | 1          |
| 3   | Windows                  | Windows - diff IPs logon                                     | Alert when Windows logon process is detected and two or more different IP addressed are seen in source field.  Timeframe is last 15min.Detection is based onevents 4624 or 1200.4624: An account was successfully logged on1200: Application token success                                                                                                                                                                                                                                                                                                                                                                                                          | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min, for last 15min | 1          |
| 4   | Windows                  | Windows - Event service error                                | Alert when Windows event 1108 is matched1108: The event logging service encountered an error                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 5   | Windows                  | Windows - file insufficient privileges                       | Alert when Windows event 5145 is matched5145: A network share object was checked to see whether client can be granted desired accessEvery time a network share object (file or folder) is accessed, event 5145 is logged. If the access is denied at the file share level, it is audited as a failure event. Otherwise, it considered a success. This event is not generated for NTFS access.                                                                                                                                                                                                                                                                       | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min, for last 15min | 50         |
| 6   | Windows                  | Windows - Kerberos pre-authentication failed                 | Alert when Windows event 4625 or 4771 is matched 4625: An account failed to log on 4771: Kerberos pre-authentication failed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 7   | Windows                  | Windows - Logs deleted                                       | Alert when Windows event 1102 OR 104 is matched1102: The audit log was cleared104: Event log cleared                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 8   | Windows                  | Windows - Member added to a security-enabled global group    | Alert when Windows event 4728 is matched4728: A member was added to a security-enabled global group                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 9   | Windows                  | Windows - Member added to a security-enabled local group     | Alert when Windows event 4732 is matched4732: A member was added to a security-enabled local group                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 10  | Windows                  | Windows - Member added to a security-enabled universal group | Alert when Windows event 4756 is matched4756: A member was added to a security-enabled universal group                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 11  | Windows                  | Windows - New device                                         | Alert when Windows event 6414 is matched6416: A new external device was recognized by the system                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 12  | Windows                  | Windows - Package installation                               | Alert when Windows event 4697 is matched 4697: A service was installed in the system                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 13  | Windows                  | Windows - Password policy change                             | Alert when Windows event 4739 is matched4739: Domain Policy was changed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 14  | Windows                  | Windows - Security log full                                  | Alert when Windows event 1104 is matched1104: The security Log is now full                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 15  | Windows                  | Windows - Start up                                           | Alert when Windows event 4608 is matched 4608: Windows is starting up                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 16  | Windows                  | Windows - Account lock                                       | Alert when Windows event 4740 is matched4740: A User account was Locked out                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 17  | Windows                  | Windows - Security local group was changed                   | Alert when Windows event 4735 is matched4735: A security-enabled local group was changed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 18  | Windows                  | Windows - Reset password attempt                             | Alert when Windows event 4724 is matched4724: An attempt was made to reset an accounts password                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 19  | Windows                  | Windows - Code integrity changed                             | Alert when Windows event 5038 is matched5038: Detected an invalid image hash of a fileInformation:    Code Integrity is a feature that improves the security of the operating system by validating the integrity of a driver or system file each time it is loaded into memory.    Code Integrity detects whether an unsigned driver or system file is being loaded into the kernel, or whether a system file has been modified by malicious software that is being run by a user account with administrative permissions.    On x64-based versions of the operating system, kernel-mode drivers must be digitally signed.The event logs the following information: | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 20  | Windows                  | Windows - Application error                                  | Alert when Windows event 1000 is matched1000: Application error                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | winlogbeat-* | winlogbeat                                       | Widnows Application Eventlog | Every 1min                 | 1          |
| 21  | Windows                  | Windows - Application hang                                   | Alert when Windows event 1001 OR 1002 is matched1001: Application fault bucket1002: Application hang                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | winlogbeat-* | winlogbeat                                       | Widnows Application Eventlog | Every 1min                 | 1          |
| 22  | Windows                  | Windows - Audit policy changed                               | Alert when Windows event 4719 is matched4719: System audit policy was changed                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 23  | Windows                  | Windows - Eventlog service stopped                           | Alert when Windows event 6005 is matched6005: Eventlog service stopped                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 24  | Windows                  | Windows - New service installed                              | Alert when Windows event 7045 OR 4697 is matched7045,4697: A service was installed in the system                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 25  | Windows                  | Windows - Driver loaded                                      | Alert when Windows event 6 is matched6: Driver loadedThe driver loaded events provides information about a driver being loaded on the system. The configured hashes are provided as well as signature information. The signature is created asynchronously for performance reasons and indicates if the file was removed after loading.                                                                                                                                                                                                                                                                                                                             | winlogbeat-* | winlogbeat                                       | Widnows System Eventlog      | Every 1min                 | 1          |
| 26  | Windows                  | Windows - Firewall rule modified                             | Alert when Windows event 2005 is matched2005: A Rule has been modified in the Windows firewall Exception List                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 27  | Windows                  | Windows - Firewall rule add                                  | Alert when Windows event 2004 is matched2004: A firewall rule has been added                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
| 28  |                          | Windows - Firewall rule deleted                              | Alert when Windows event 2006 or 2033 or 2009 is matched2006,2033,2009: Firewall rule deleted                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | winlogbeat-* | winlogbeat                                       | Widnows Security Eventlog    | Every 1min                 | 1          |
````

## Playbooks ##

CyberX has a set of predefined set of rules and activities (called Playbook) that can be attached to a registered event in the Alert module.
Playbooks can be enriched with scripts that can be launched together with Playbook.

### Create Playbook ###

To add a new playbook, go to the **Alert** module, select the **Playbook** tab and then **Create Playbook**

![](/media/media/image116.png)

In the **Name** field, enter the name of the new Playbook.

In the **Text** field, enter the content of the Playbook message.

In the **Script** field, enter the commands to be executed in the script.
G
To save the entered content, confirm with the **Submit** button.

### Playbooks list  ###

To view saved Playbook, go to the **Alert** module, select the **Playbook** tab and then **Playbooks list**:

![](/media/media/image117.png)

To view the content of a given Playbook, select the **Show** button.

To enter the changes in a given Playbook or in its script, select the **Update** button. After making changes, select the **Submit** button.

To delete the selected Playbook, select the **Delete** button.

### Linking Playbooks with alert rule ###

You can add a Playbook to the Alert while creating a new Alert or by editing a previously created Alert.

To add Palybook to the new Alert rule, go to the **Create alert rule** tab and in the **Playbooks** section use the arrow keys to move the correct Playbook to the right window.

To add a Palybook to existing Alert rule, go to the **Alert rule list** tab with the correct rule select the **Update** button and in the **Playbooks** section use the arrow keys to move the correct Playbook to the right window.

### Playbook verification ###

When creating an alert or while editing an existing alert, it is possible that the system will indicate the most-suited playbook for the alert. For this purpose, the Validate button is used, which starts the process of searching the existing playbook and selects the most appropriate ones.

![](/media/media/image132.png)

## Risks ##

CyberX allows you to estimate the risk based on the collected data. The risk is estimated based on the defined category to which the values from 0 to 100 are assigned.

Information on the defined risk for a given field is passed with an alert and multiplied by the value of the Rule Importance parameter.

### Create category ###

To add a new risk Category, go to the **Alert** module, select the **Risks** tab and then **Create Cagtegory**. 

![](/media/media/image118.png)

Enter the **Name** for the new category and the category **Value**.

### Category list ###

To view saved Category, go to the **Alert** module, select the **Risks** tab and then **Categories list**:

![](/media/media/image119.png)

To view the content of a given Category, select the **Show** button.

To change the value assigned to a category, select the **Update** button. After making changes, select the **Submit** button.

To delete the selected Category, select the **Delete** button.

### Create risk ###

To add a new playbook, go to the Alert module, select the Playbook tab and then Create Playbook

![](/media/media/image120.png)

In the **Index pattern** field, enter the name of the index pattern.
Select the **Read fields** button to get a list of fields from the index.
From the box below, select the field name for which the risk will be determined.

From the **Timerange field**, select the time range from which the data will be analyzed.

Press the **Read valules** button to get values from the previously selected field for analysis.

Next, you must assign a risk category to the displayed values. You can do this for each value individually or use the check-box on the left to mark several values and set the category globally using the **Set global category** button. To quickly find the right value, you can use the search field.

![](/media/media/image122.png)

After completing, save the changes with the **Submit** button.

### List risk ###

To view saved risks, go to the **Alert** module, select the **Risks** tab and then **Risks list**:

![](/media/media/image121.png)

To view the content of a given Risk, select the **Show** button.

To enter the changes in a given Risk, select the **Update** button. After making changes, select the **Submit** button.

To delete the selected Risk, select the **Delete** button.

### Linking risk with alert rule ###

You can add a Risk key to the Alert while creating a new Alert or by editing a previously created Alert.

To add Risk key to the new Alert rule, go to the **Create alert rule** tab and after entering the index name, select the **Read fields** button and in the **Risk key** field, select the appropriate field name.
In addition, you can enter the validity of the rule in the **Rule Importance** field (in the range 1-100%), by which the risk will be multiplied.

To add Risk key to the existing Alert rule, go to the **Alert rule list**, tab with the correct rule select the **Update** button. 
Use the **Read fields** button and in the **Risk key** field, select the appropriate field name.
In addition, you can enter the validity of the rule in the **Rule Importance**.

### Risk calculation algorithms ###

The risk calculation mechanism performs the aggregation of the risk field values.
We have the following algorithms for calculating the alert risk (Aggregation type):

- min - returns the minimum value of the risk values from selected fields;
- max - returns the maximum value of the risk values from selected fields;
- avg - returns the average of risk values from selected fields;
- sum - returns the sum of risk values from selected fields;
- custom - returns the risk value based on your own algorithm

### Adding a new risk calculation algorithm ###

The new algorithm should be added in the `./elastalert_modules/playbook_util.py` file in the `calculate_risk` method. There is a sequence of conditional statements for already defined algorithms:

	#aggregate values by risk_key_aggregation for rule
	if risk_key_aggregation == "MIN":
	    value_agg = min(values)
	elif risk_key_aggregation == "MAX":
	    value_agg = max(values)
	elif risk_key_aggregation == "SUM":
	    value_agg = sum(values)
	elif risk_key_aggregation == "AVG":
	    value_agg = sum(values)/len(values)
	else:
	    value_agg = max(values)

To add a new algorithm, add a new sequence as shown in the above code:

	elif risk_key_aggregation == "AVG":
	    value_agg = sum(values)/len(values)
	elif risk_key_aggregation == "AAA":
	    value_agg = BBB
	else:
	    value_agg = max(values)

where **AAA** is the algorithm code, **BBB** is a risk calculation function.

### Using the new algorithm ###

After adding a new algorithm, it is available in the GUI in the Alert tab.

To use it, add a new rule according to the following steps:

- Select the `custom` value in the `Aggregation` type field;
- Enter the appropriate value in the `Any` field, e.g. `risk_key_aggregation: AAA`

The following figure shows the places where you can call your own algorithm:

![](/media/media/image123.png)

### Additional modification of the algorithm (weight) ###

Below is the code in the `calcuate_risk` method where category values are retrieved - here you can add your weight:

            #start loop by tablicy risk_key
            for k in range(len(risk_keys)):
                risk_key = risk_keys[k]
                logging.info(' >>>>>>>>>>>>>> risk_key: ')
                logging.info(risk_key)
                key_value = lookup_es_key(match, risk_key)
                logging.info(' >>>>>>>>>>>>>> key_value: ')
                logging.info(key_value)
                value = float(self.get_risk_category_value(risk_key, key_value))
                values.append( value )
                logging.info(' >>>>>>>>>>>>>> risk_key values: ')
                logging.info(values)
            #finish loop by tablicy risk_key
            #aggregate values by risk_key_aggregation form rule
            if risk_key_aggregation == "MIN":
                value_agg = min(values)
            elif risk_key_aggregation == "MAX":
                value_agg = max(values)
            elif risk_key_aggregation == "SUM":
                value_agg = sum(values)
            elif risk_key_aggregation == "AVG":
                value_agg = sum(values)/len(values)
            else:
                value_agg = max(values)

`Risk_key` is the array of selected risk key fields in the GUI.
A loop is made on this array and a value is collected for the categories in the line:

	value = float(self.get_risk_category_value(risk_key, key_value))

Based on, for example, Risk_key, you can multiply the value of the value field by the appropriate weight.
The value field value is then added to the table on which the risk calculation algorithms are executed.

## Incidents

The Incident module allows you to handle incidents created by triggered alert rules. 

![](/media/media/image154.PNG)

Incident handling allows you to perform the following action:

- *Show incident* - shows the details that generated the incident;
- *Verify* - checks the IP addresses of those responsible for causing an incident with the system reputation lists;
- *Preview* - takes you to the Discover module and to the raw document responsible for generating the incident;
- *Update* - allows you to change the Incident status or transfer the incident handling to another user. Status list: *New, Ongoing, False, Solved.*
- *Playbooks* - enables handling of Playbooks assigned to an incident;
- *Note* - User notes about the incident;

### Incident Escalation

The alarm rule definition allows an incident to be escalated if the incident status does not change (from New to Ongoing) after a defined time.

Configuration parameter

- *escalate_users* - an array of users who get an email alert about the escalation;
- *escalate_after* - the time after which the escalation is triggered;

Example of configuration:

```yaml
escalate_users:["user2", "user3"]
escalate_after:
	- hours: 6
```

### Context menu for Alerts::Incidents

In this section, you will find steps and examples that will allow you to add custom items in the actions context menu for the Incidents table. This allows you to expand on the functionalities of the system.

#### Important file paths

- `/usr/share/kibana/plugins/alerts/public/reactui/incidenttab.js`
- `/usr/share/kibana/optimize/bundles/`

#### List element template

```javascript
{
  name: 'Name of the Action to add',
  icon: 'Name of the chosen icon',
  type: 'icon',
  onClick: this.runActionFunction,
}
```

You should pick the icon from available choices. After listing `ls /usr/share/kibana/built_assets/dlls/icon*` if you want to use:

- `icon.editor_align_center-js.bundle.dll.js`  
  The for `icon: ` you should set:  
- `editorAlignCenter`  
  Use the same transformation for each icon.

#### Action function template

```javascript
runActionFunction = item => {
  // Functino logic to run => information from "item" object can be used here
};
```

Object "item" contains information about the incident that action was used on.

#### Steps to add the first custom action to the codebase

1. Create backup of a file you are about to modify:

   ```bash
   cp /usr/share/kibana/plugins/alerts/public/reactui/incidenttab.js ~/incidenttab.js.bak
   ```

   

2. Working example for the onClick function and action item:

   ```javascript
   showMyLocation = () => {
     const opt = {
       enableHighAccuracy: true,
       timeout: 5000,
       maximumAge: 0
     };
     const success = pos => {
       const crd = pos.coords;
       alert(`Your current position is:\nLatitude: ${
         crd.latitude
       }\nLongitude: ${
         crd.longitude
       }\nMore or less ${
         crd.accuracy
       } meters.`);
     }
     const err = err => {
       alert(`ERROR(${err.code}): ${err.message}`);
     }
     navigator.geolocation.getCurrentPosition(success, err, opt);
   }
   
   const customActions = [
     {
       name: 'Show my location',
       icon: 'broom',
       type: 'icon',
       onClick: this.showMyLocation,
     }
   ];
   incidentactions.push(...customActions);
   ```

   

3. The "showMyLocation" function code should be placed in `/usr/share/kibana/plugins/alerts/public/reactui/incidenttab.js` under:

   ```javascript
     showIncidentModal = incident => {
       const updateIncident = incident;
       this.setState({ showIncidentModal: true, updateIncident });
     };
   
     // paste function here
   
     render() {
   ```

   

4. Custom action with a `push` function should be placed:

   ```javascript
         {
           name: 'Note',
           icon: 'pencil',
           type: 'icon',
           isPrimary: true,
           color: 'danger',
           onClick: this.note,
         },
       ];
   
       // insert HERE your action with function 'push'
   
       const incidentcolumns = [
   ```

   

5. For the changes to take effect run below commands on the client serwer (as root or with sudo):

   ```bash
   systemctl stop kibana
   rm -rf /usr/share/kibana/optimize/bundles
   systemctl start kibana
   # verify that process runs correctly afterwards
   journalctl -fu kibana
   # in case of errors restore backup
   ```

   

6. You should now be able to see an additional item in the action context menu in GUI Alerts::Incidents:

   ![](/media/media/image202.png)

   

7. Running the action will resolve into an alert:  

   ![](/media/media/image203.png)

   

#### Steps to add a second and subsequent custom actions

1. Execute identicly as in the last section.

2. Example of a function that uses `item` object. It will open a new tab in the browser with the default [Alert] dashboard with a custom filter and time set, based on information from the passed `item` variable:

   ```javascript
   openAlertDashboardWithFilter = item => {
     const ruleName = `"${item.rule_name}"`;
     const startT = new Date(item.match_time);
     startT.setHours(0);
     const endT = new Date(item.match_time);
     endT.setHours(24);
     const alertDashboardPath =
       '/app/kibana#/dashboard/777ace50-d200-11e8-98f8-31520a7f9701';
     const timeQuery = 
       `_g=(time:(from:'${startT.toISOString()}',to:'${endT.toISOString()}'))`;
     const nameQuery = 
       `_a=(query:(language:lucene,query:'rule_name:${encodeURIComponent(
         ruleName
       )}'))`;
     const dashboardLocation = `${alertDashboardPath}?${timeQuery}&${nameQuery}`;
     window.open(dashboardLocation, '_blank');
   };
   ```

   

3. Execute identicly as in the last section.

4. The difference in adding subsequent action is that you append a new one to `customActions` array variable. The rest should stay the same:

   ```javascript
   const customActions = [
     {
       name: 'Show my location',
       icon: 'broom',
       type: 'icon',
       onClick: this.showMyLocation,
     },
     {
       name: 'Show on Dashboard',
       icon: 'arrowRight',
       type: 'icon',
       onClick: this.openAlertDashboardWithFilter,
     },
   ];
   incidentactions.push(...customActions);
   ```

   

5. Execute identicly as in the last section.

6. Now both actions should be present on the context menu:

   ![](/media/media/image204.png)

7. Using it will open dashboard in new tab:

   ![](/media/media/image205.png)

#### Updating system

When updating the system your changes might be overwritten. You should in that case save a backup of your changes and restore them after the update with the use of this instruction. Or for instance, with `vimdiff` compare your changes with the original file:

```bash
vimdiff ~/incidenttab.js.bak /usr/share/kibana/plugins/alerts/public/reactui/incidenttab.js
```

## Indicators of compromise (IoC)

CyberX has the Indicators of compromise (IoC) functionality, which is based on the Malware Information Sharing Platform (MISP).
IoC observes the logs sent to the system and marks documents if their content is in MISP signature.
Based on IoC markings, you can build alert rules or track incident behavior.

### Configuration

#### Bad IP list update

To update bad reputation lists and to create `.blacklists` index, you have to run following scripts:

```bash
/etc/logstash/lists/bin/badreputation_iplists.sh
/etc/logstash/lists/bin/misp_threat_lists.sh
```

#### Scheduling bad IP lists update

This can be done in `cron` (host with Logstash installed):

```bash
0 1 * * * logstash /etc/logstash/lists/bin/badreputation_iplists.sh
0 6 * * * logstash /etc/logstash/lists/bin/misp_threat_lists.sh
```

or with Kibana Scheduller app (**only if Logstash is running on the same host**).

- Prepare script path:

```bash
/bin/ln -sfn /etc/logstash/lists/bin /opt/ai/bin/lists
chown logstash:kibana /etc/logstash/lists/
chmod g+w /etc/logstash/lists/
```

- Log in to CyberX GUI and go to **Scheduler** app. Set it up with below options and push "Submit" button:

```bash
Name:           BadReputationList
Cron pattern:   0 1 * * *
Command:        lists/badreputation_iplists.sh
Category:       logstash
```

and second:

```bash
Name:           MispThreatList
Cron pattern:   0 1 * * *
Command:        lists/misp_threat_lists.sh
Category:       logstash
```

After a couple of minutes check for blacklists index:

```bash
curl -sS -u user:password -XGET '127.0.0.1:9200/_cat/indices/.blacklists?s=index&v'
health status index       uuid                   pri rep docs.count docs.deleted store.size pri.store.size
green  open   .blacklists Mld2Qe2bSRuk2VyKm-KoGg   1   0      76549            0      4.7mb          4.7mb
```

#### Configuration alert rule

## Calendar function

The alert rule can be executed based on a schedule called Calendar.

### Create a calendar system

The configuration of the **Calendar Function** should be done in the definition of the `Rule Definition` alert using the `calendar` and `scheduler` options, in **Crontab** format.

For example, we want to have an alert that:

- triggers only on working days from 8:00 to 16:00;

- only triggers on weekends;

```bash
calendar:
  schedule: "* 8-15 * * mon-fri"
```

If `aggregation` is used in the alert definition, remember that the aggregation schedule should be the same as the defined calendar.