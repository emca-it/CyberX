# Reports #


CyberX contains a module for creating reports that can be
run cyclically and contain only interesting data, e.g. a weekly sales
report.

To go to the reports windows, select to tiles icon from the main menu
bar, and then go to the „Reports" icon (To go back, go to the „Search"
icon).

![](/media/media/image38_js8.png)
## CSV Report ##

To export data to CSV Report click the ***Reports*** icon, you immediately go 
to the first tab - ***Export Data***

In this tab we have the opportunity to specify the source from which
we want to do export. It can be an index pattern. After selecting it,
we confirm the selection with the Submit button and a report is
created at the moment. The symbol
![](/media/media/image40.png)can refresh the list of reports and see
what its status is.

![](/media/media/image41_js.png)

We can also create a report by pointing to a specific index from the
drop-down list of indexes.

![](/media/media/image42_js.png)

We can also check which fields are to be included in the report. The
selection is confirmed by the Submit button.

![](/media/media/image43_js.png)

When the process of generating the report (Status:Completed) is
finished, we can download it (Download button) or delete (Delete
button). The downloaded report in the form of \*.csv file can be
opened in the browser or saved to the disk.

![](/media/media/image44_js.png)

In this tab, the downloaded data has a format that we can import into
other systems for further analysis.
## PDF Report ##

In the Export Dashboard tab we have the possibility to create
graphic reports in PDF files. 
To create such a report, just from the drop-down list of previously 
created and saved Dashboards, indicate the one we are interested in, 
and then confirm the selection with the Submit button. A newly 
created export with the Processing status will appear on the list 
under Dashboard Name. 
When the processing is completed, the Status changes to Complete and it
will be possible to download the report.

![](/media/media/image45.png)

By clicking the Download button, the report is downloaded to the disk
or we can open it in the PDF file browser. There is also to option
of deleting the report with the Delete button.

![](/media/media/image46.png)

Below is an example report from the Dashboard template generated and
downloaded as a PDF file.

![](/media/media/image47.png)

## PDF report from the table visualization

Data from a table visualization can be exported as a PDF report. 

To export a table visualization data, follow these steps:

1. Go to the 'Report' module and then to the 'Report Export' tab,
2. Add the new task name in 'Task Name' field,
3. Toggle the switch 'Enable Data Table Export':

   ![](/media/media/image215.png)
   
4. Select the table from the 'Table Visualization' list,
5. Select the time range for which the report is to be prepared,
6. You can select a logo from the 'Logo' list,
7. You can add a report title using the 'Title' field,
8. You can add a report comment using the 'Comments' field,
9. Select the 'Submit' button to start creating the report,
10. You can follow the progress in the 'Task List' tab,
11. After completing the task, the status will change to 'Complete' and you can download the PDF report via 'Action' -> 'Download':

    ![](/media/media/image216.png)

## Scheduler Report (Schedule Export Dashboard) ##


In the Report selection, we have the option of setting the Scheduler
which from Dashboard template can generate a report at time intervals. 
To do this goes to the Schedule Export Dashboard tab.

![](/media/media/image48_js.png)

Scheduler Report (Schedule Export Dashboard)

In this tab mark the saved Dashboard.

![](/media/media/image49_js.png)

*Note: The default time period of the dashboard is last 15 minutes.*

*Please refer to **Discovery > Time settings and refresh** to change the time period of your dashboard.*

In the Email Topic field, enter the Message title, in the Email field
enter the email address to which the report should be sent. From
drop-down list choose at what frequency you want the report to be generated and sent. 
The action configured in this way is confirmed with the Submit button.

![](/media/media/image50_js.png)

The defined action goes to the list and will generate a report to the
e-mail address, with the cycle we set, until we cannot cancel it with
the Cancel button.

![](/media/media/image51_js.png)
