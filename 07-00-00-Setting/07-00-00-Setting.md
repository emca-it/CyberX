# Settings #
## General Settings ##

The Settings tab is used to set the audit on different activates or events 
and consists of several fields:

![](/media/media/image60_js.png)

-   **Time Out in minutes** field - this field defines the time after how
    many minutes the application will automatically log you off
-   **Delete Application Tokens (in days)** - in this field we specify
    after what time the data from the audit should be deleted
-   **Delete Audit Data (in days)** field - in this field we specify after
    what time the data from the audit should be deleted
-   Next field are checkboxes in which we specify what kind of events
    are to be logged (saved) in the audit index. The events that can be
    monitored are: logging (Login), logging out (Logout), creating a
    user (Create User), deleting a user (Delete User), updating user
    (Update User), creating a role (Create Role), deleting a role
    (Delete Role), update of the role (Update Role), start of export
    (Export Start), delete of export (Export Delete), queries (Queries),
    result of the query (Content), if attempt was made to perform a
    series of operation (Bulk)
-   **Delete Exported CSVs (in days)** field - in this field we specify
    after which time exported file with CSV extension have to be removed
-   **Delete Exported PDFs (in days)** field - in this field we specify
    after which time exported file with PDF extension have to be removed

To each field is assigned "Submit" button thanks to which we can confirm the changes.
## License (License Info) ##

The License Information tab consists of several non-editable
information fields.

![](/media/media/image61_js.png)

These fields contain information:
- Company field, who owns the license - in this case EMCA S.A.
- Data nodes in cluster field - how many nodes we can put in one
  cluster - in this case 100
- No of documents field - empty field
- Indices field - number of indexes, symbol\[\*\] means that we can
  create any number of indices
- Issued on field - date of issue
- Validity field - validity, in this case for 360000 months
### Renew license

To change the the CyberX license files on a running system, do the following steps.

1. Copy the current license files to the backup folder:

   ```bash
   mv /usr/share/elasticsearch/es_* ~/backup/
   ```

2. Copy the new license files to the Elasticsearch installation directory:

   ```bash
   cp es_* /usr/share/elasticsearch/
   ```

3. Add necessary permission to the new license files:

   ```bash
   chown elasticsearch:elasticsearch /usr/share/elasticsearch/es_*
   ```

4. Reload the license using the License API:

   ```bash
   curl -u $USER:$PASSWORD -X POST http://localhost:9200/_license/reload
   ```

## Special accounts ##

At the first installation of the CyberX application, apart
from the administrative account (logserver), special applications are
created in the application: alert, intelligence and scheduler.

![](/media/media/image62_js.png)

- **Alert Account** - this account is connected to the Alert Module
which is designed to track events written to the index for the
previously defined parameters. If these are met the information
action is started (more on the action in the Alert section)
- **Intelligence Account** - with this account is related to the module
of artificial intelligence which is designed to track events and learn
the network based on previously defined rules artificial intelligence
based on one of the available algorithms (more on operation in the
Intelligence chapter)
- **Scheduler Account** - the scheduler module is associated with this
account, which corresponds to, among others for generating reports
