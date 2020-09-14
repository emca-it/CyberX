# Visualizations #


Visualize enables you to create visualizations of the data
in your CyberX indices. You can then build dashboards
that display related visualizations. Visualizations are based
on CyberX queries. By using a series of CyberX
aggregations to extract and process your data, you can create
charts that show you the trends, spikes, and dips.

## Creating visualization ##

### Create ###
To create visualization, go to the „Visualize" tab from the main menu.
A new page will be appearing where you can create or load
visualization.

### Load ###
To load previously created and saved visualization, you
must select it from the list.

![](/media/media/image89.PNG)

In order to create a new visualization,
you should choose the preferred method of data presentation.

![](/media/media/image89_js.PNG)

Next, specify whether the created visualization will be based on a new or
previously saved query. If on new one, select the index whose
visualization should concern. If visualization is created from a saved
query, you just need to select the appropriate query from the list, or
(if there are many saved searches) search for them by name.

![](/media/media/image90.PNG)

## Vizualization types ##

Before the data visualization will be created, first you have to
choose the presentation method from an existing list. Currently there
are five groups of visualization types. Each of them serves different purposes.
If you want to see only the current number of products
sold, it is best to choose „Metric", which presents one value.

![](/media/media/image27_js.png)

However, if we would like to see user activity trends on pages in
different hour and days, a better choice will be „Area chart", which
displays a chart with time division.

![](/media/media/image28_js.png)

The „Markdown widget" views is used to place text e.g. information
about the dashboard, explanations and instruction on how to navigate.
Markdown language was used to format the text (the most popular use is
GitHub). 
More information and instruction can be found at this link:
[https://help.github.com/categories/writing-on-github/](https://help.github.com/categories/writing-on-github/)

## Edit visualization and saving ##

### Edititing ###
Editing a saved visualization enables you to directly modify the object 
definition. You can change the object title, add a description, and modify
the JSON that defines the object properties.
After selecting the index and the method of data presentation, you can enter
the editing mode. This will open a new window with empty
visualization.

![](/media/media/image29_js.png)

At the very top there is a bar of queries that cat be edited
throughout the creation of the visualization. It work in the same way
as in the "Discover" tab, which means searching the raw data, but
instead of the data being displayed, the visualization will be edited.
The following example will be based on the „Area chart". The
visualization modification panel on the left is divided into three tabs:
„Data", "Metric & Axes" and „Panel Settings".

In the „Data" tab, you can modify the elements responsible for which data
and how should be presented. In this tab there are two sectors:
"metrics", in which we set what data should be displayed, and
„buckets" in which we specify how they should be presented. 

Select the Metrics & Axes tab to change the way each individual metric is shown 
on the chart. The data series are styled in the Metrics section, while the axes 
are styled in the X and Y axis sections.

In the „Panel Settings" tab, there are settings relating mainly to visual
aesthetics. Each type of visualization has separate options. 

To create the first graph in the char modification panel, in the „Data" tab we
add X-Axis in the "buckets" sections. In „Aggregation" choose „Histogram", 
in „Field" should automatically be located "timestamp" and "interval": 
"Auto" (if not, this is how we set it). Click on the
icon on the panel. Now our first graph should show up.

Some of the options for „Area Chart" are:

   **Smooth Lines** - is used to smooth the graph line.

![](/media/media/image30.png)

-   **Current time marker** -- places a vertical line on the graph that
    determines the current time.

-   **Set Y-Axis Extents** -- allows you to set minimum and maximum
    values for the Y axis, which increases the readability of the
    graphs. This is useful, if we know that the data will never be
    less then (the minimum value), or to indicate the goals the
    company (maximum value).

-  **Show Tooltip** -- option for displaying the information window
    under the mouse cursor, after pointing to the point on the graph.
   
   ![](/media/media/image31.png)
   
### Saving ###
To save the visualization, click on the "Save" button under on the query bar:
![](/media/media/image16.png)
give it a name and click the button
![](/media/media/image17.png). 

### Load ###
To load the visualization, go to the "Management Object"
 -> "Saved Object" -> "Visualizations" select it from the list. From this place, 
we can also go into advanced editing mode. To view of the visualization 
use ![](/media/media/image17_js.png) button.

## Dashboards ##
Dashboard is a collection of several visualizations or searches.
Depending on how it is build and what visualization it contains, it
can be designed for different teams e.g.: 
- SOC - which is responsible for detecting failures or threats in 
the company; 
- business - which thanks to the listings can determine the popularity
of products and define the strategy of future sales and promotions; 
- managers and directors - who may immediately have access to information
about the performance units or branches. 

### Create ###
To create a dashboard from previously saved visualization and queries,
go to the „Dashboard" tab in the main menu. When you open it, a
new page will appear.

![](/media/media/image32.png)

Clicking on the icon "Add" at the top of page select "Visualization" or "Saved Search"
tab.

![](/media/media/image32_js.png)

and selecting a saved query and / or visualization from the list will 
add them to the dashboard. If, there are a large number of saved objects, 
use the bar to search for them by name. 

Elements of the dashboard can be enlarged arbitrarily (by clicking on
the right bottom corner of object and dragging the border) and moving
(by clicking on the title bar of the object and moving it). 

### Saving ###
To save a dashboard, click on the "Save" button to the up of the query bar 
and give it a name. 

### Load ###
To load the Dashboard, go to the "Management Object"
 -> "Saved Object" -> "Dashborad" select it from the list. From this place, 
we can also go into advanced editing mode. To view of the visualization 
use ![](/media/media/image17_1_js.png) button.

## Sharing dashboards ##

The dashboard can be share with other CyberX users as well
as on any page - by placing a snippet of code. Provided that it cans
retrieve information from CyberX.

To do this, create new dashboard or open the saved dashboard and click 
on the "Share" to the top of the page. A window
will appear with generated two URL. The content of the first one "Embaded iframe"
is used to provide the dashboard in the page code, and the second "Link" is a link
that can be passed on to another user. There are two option for each, 
the first is to shorten the length of the link, and second on
copies to clipboard the contest of the given bar.

![](/media/media/image37.png)

## Dashboard drilldown ##

In discovery tab search for message of  Your interest

![](/media/media/image125.png)

----------

Save Your search

![](/media/media/image126.png)

Check You „Shared link” and copy it

![](/media/media/image127.png)
![](/media/media/image128.png)

**! ATTENTION !** Do not copy „?_g=()” at the end.

----------

Select Alerting module

![](/media/media/image129.png)

----------

Once Alert is created use `ANY` frame to add the following directives:

	Use_kibana4_dashboard: paste Your „shared link” here

`use_kibana_dashboard:` - The name of a Kibana dashboard to link to. Instead of generating a dashboard from a template, Alert can use an existing dashboard. It will set the time range on the dashboard to around the match time, upload it as a temporary dashboard, add a filter to the query_key of the alert if applicable, and put the url to the dashboard in the alert. (Optional, string, no default).

----------

	Kibana4_start_timedelta

`kibana4_start_timedelta:` Defaults to 10 minutes. This option allows you to specify the start time for the generated kibana4 dashboard. This value is added in front of the event. For example,

	`kibana4_start_timedelta: minutes: 2`

----------


	Kibana4_end_timedelta`

`kibana4_end_timedelta:` Defaults to 10 minutes. This option allows you to specify the end time for the generated kibana4 dashboard. This value is added in back of the event. For example,

	kibana4_end_timedelta: minutes: 2

----------
Sample:
![](/media/media/image130.png)


----------
Search for triggered alert in Discovery tab. Use alert* search pattern.

![](/media/media/image131.png)

Refresh the alert that should contain url for the dashboard.
Once available, kibana_dashboard field can be exposed to dashboards giving You a real drill down feature.