# Discovery #

## Time settings and refresh ##


In the upper right corner there is a section in which it defines the
range of time that CyberX will search in terms of conditions contained in
the search bar. The default value is the last 15 minutes.

![](/media/media/image7.png)

After clicking this selection, we can adjust the scope of search by
selecting one of the three tabs in the drop-down window:

- **Quick**: contain several predefined ranges that should be clicked.
- **Relative**: in this windows specify the day from which CyberX should search for data.
- **Absolute**: using two calendars we define the time range for which the search results are to be returned.

![](/media/media/image8.png)
## Fields ##

CyberX in the body of searched events, it recognize fields
that can be used to created more precision queries. The extracted
fields are visible in the left panel. They are divided on three types: 
timestamp, marked on clock icon 
![](/media/media/image9.png); text, marked with the letter "t"
![](/media/media/image10.png) and digital, marked witch hashtag
![](/media/media/image11.png).

Pointing to them and clicking on icon
![](/media/media/image12.png), they are automatically transferred to
the „Selected Fields" column and in the place of events a table with 
selected columns is created on regular basis. In the "Selected Fields" 
selection you can also delete specific fields from the table by clicking
![](/media/media/image13.png) on the selected element.

![](/media/media/image14_js.png)
## Filtering and syntax building ##


We use the query bar to search interesting events. For example, after
entering the word „error", all events that contain the word will be
displayed, additional highlighting them with an yellow background.

![](/media/media/image15_js.png)

### Syntax ###
Fields can be used in the similar way by defining conditions that
interesting us. The syntax of such queries is:

	<fields_name:<fields_value>

Example:

	status:500

This query will display all events that contain the „status" fields 
with a value of 500.

### Filters ###
The field value does not have to be a single, specific value. For
digital fields we can specify range in the following scheme:

	<fields_name:[<range_from TO <range_to] 

Example: 

	status:[500 TO 599]

This query will return events with status fields that are in the 
range 500 to 599.

### Operators ###
The search language used in CyberX allows to you use logical operators
„AND", „OR" and „NOT", which are key and necessary to build more
complex queries.

-   **AND** is used to combined expressions, e.g. „error AND „access
   denied". If an event contain only one expression or the words
   error and denied but not the word access, then it will not be
   displayed.

-   **OR** is used to search for the events that contain one OR other
   expression, e.g. „status:500" OR "denied". This query will display
   events that contain word „denied" or status field value of 500. CyberX
   uses this operator by default, so query „status:500" "denied" would
   return the same results.

-   **NOT** is used to exclude the following expression e.g. „status:[500
   TO 599] NOT status:505" will display all events that have a status
   fields, and the value of the field is between 500 and 599 but will
   eliminate from the result events whose status field value is exactly 505.

-   **The above methods** can be combined with each other by building even
   more complex queries. Understanding how they work and joining it, is
   the basis for effective searching and full use of CyberX.
   
   Example of query built from connected logical operations:
   
	status:[500 TO 599] AND („access denied" OR error) NOT status:505

Returns in the results all events for which the value of status fields
are in the range of 500 to 599, simultaneously contain the word
„access denied" or „error", omitting those events for which the status
field value is 505.
## Saving and deleting queries ##


Saving queries enables you to reload and use them in the future. 

### Save query ###
To save query, click on the "Save" button under on the query bar:

![](/media/media/image16.png) 

This will bring up a window in which we give the query a name and then
click the button ![](/media/media/image17.png).

![](/media/media/image18_js.png)

Saved queries can be opened by going to „Open" from the main menu
at the top of the page, and select saved search from the search list:

![](/media/media/image19.png)

Additional you can use "Saved Searchers Filter.." to filter the search list.

### Open query ###
To open a saved query from the search list, you can click on the name of the query 
you are interested in.

After clicking on the icon 
![](/media/media/image21.png) on the name of the saved query and chose 
"Edit Query DSL", we will gain access to the advanced editing mode, 
so that we can change the query on at a lower level. 

![](/media/media/image21.png)

It is a powerful tool designed for advanced users, designed to modify 
the query and the way it is presented by CyberX.

### Delete query ###
To delete a saved query, open it from the search list, and
then click on the button ![](/media/media/image23.png) . 

If you want delete many saved queries simultaneously go to the "Management Object"
 -> "Saved Object" -> "Searches" select it in the list (the icon ![](/media/media/image22.png) 
to the left of the query name), and then click "Delete" button. 

![](/media/media/image24_js.png)

From this level, you can also export saved queries in the same way. To
do this, you need to click on
![](/media/media/image25.png) and choose the save location. The file
will be saved in .JSON format. If you then want to import such a file to
CyberX, click on button
![](/media/media/image26.png), at the top of the page and select the
desired file.

## Manual incident

The `Discovery` module allows you to manually create incidents that are saved in the `Incidents` tab of the `Alerts` module. Manual incidents are based on search results or filtering.
For a manual incident, you can save the following parameters:

- Rule name
- Time
- Risk
- Message

![](/media/media/image141.png)

After saving the manual incident, you can go to the Incident tab in the Alert module to perform the incident handling procedure.

![](/media/media/image142.png)

## Change the default width of columns

To improve the readability of values in Discovery columns, you can set a minimum column width. The column width setting is in the configuration files:

```bash
/usr/share/kibana/built_assets/css/plugins/kibana/index.dark.css
/usr/share/kibana/built_assets/css/plugins/kibana/index.light.css
```

To set the minimum width for the columns, e.g. 150px, add the following entry `min-width: 150px` in the configuration files:

```css
.kbnDocTableCell__dataField 
   min-width: 150px;
   white-space: pre-wrap; }

```