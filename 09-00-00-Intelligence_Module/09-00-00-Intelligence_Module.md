# Intelligence Module #

A dedicated artificial intelligence module has been built in the 
CyberX system that allows prediction of parameter values
relevant to the maintenance of infrastructure and IT systems. Such
parameters include:

- use of disk resources,
- use of network resources,
- using the power of processors
- detection of known incorrect behaviour of IT systems

To access of the Intelligence module, click the tile icon
from the main meu bar and then go to the „Intelligence" icon (To go
back, click to the „Search" icon).

![](/media/media/image38_js4.png)

There are 4 screens available in the
module:![](/media/media/image64.png)

- **Create AI Rule** - the screen allows you to create artificial
     intelligence rules and run them in scheduler mode or immediately
- **AI Rules List** - the screen presents a list of created artificial
     intelligence rules with the option of editing, previewing and
     deleting them
- **AI Learn** - the screen allows to define the conditions for teaching
     the MLP neural network
- **AI Learn Tasks** - a screen on which the initiated and completed
     learning processes of neural networks with the ability to preview
     learning results are presented.# Create AI Rule #

To create the AI Rule, click on the tile icon from the main menu bar, 
go to the „Intelligence" icon and select "Create AI Rule" tab. 
The screen allows to defining the rules of artificial intelligence
based on one of the available algorithms (a detailed description of
the available algorithms is available in a separate document).

![](/media/media/image65_js.png)

Description of the controls available on the fixed part of screen:

- **Algorithm** - the name of the algorithm that forms the basis of the
artificial intelligence rule
- **Choose search** - search defined in the CyberX system,
which is used to select a set of data on which the artificial
intelligence rule will operate
- **Run** - a button that allows running the defined AI rule or saving it
to the scheduler and run as planned

The rest of the screen will depend on the chosen artificial
intelligence algorithm.

## The fixed part of the screen ##

![](/media/media/image65.png)

Description of the controls available on the fixed part of screen:

- Algorithm - the name of the algorithm that forms the basis of the artificial intelligence rule

- Choose search - search defined in the CyberX system, which is used to select a set of data on which the artificial intelligence rule will operate

- Run - a button that allows running the defined AI rule or saving it to the scheduler and run as planned

The rest of the screen will depend on the chosen artificial intelligence algorithm.

## Screen content for regressive algorithms ##

![](/media/media/image66_js.png)

Description of controls:

- **feature to analyze from search** - analyzed feature (dictated)
- **multiply by field** - enable multiplication of algorithms after
    unique values of the feature indicated here. Multiplication allows
    you to run the AI rule one for e.g. all servers. The value "none" in
    this field means no multiplication.
- **multiply by values** - if a trait is indicated in the „multiply by
    field", then unique values of this trait will appear in this field.
    Multiplications will be made for the selected values. If at least
    one of value is not selected, the „Run" buttons will be inactive.\`

In other words, multiplication means performing an analysis for many values from the indicated field, for example: `sourece_node_host`- which we indicate in `Multiply by field (from search)`.

However, in `Multiply by values (from search)` we already indicate values of this field for which the analysis will be performed, for example: host1, host2, host3, ....

- **time frame** - feature aggregation method (1 minute, 5 minute, 15
    minute, 30 minute, hourly, weekly, monthly, 6 months, 12 months)
- **max probes** - how many samples back will be taken into account for
    analysis. A single sample is an aggregated data according to the
    aggregation method.
- **value type** - which values to take into account when aggregating for
    a given time frame (e.g. maximum from time frame, minimum, average)
- **max predictions** - how many estimates we make for ahead (we take
    time frame)
- **data limit** - limits the amount of date downloaded from the source.
    It speeds up processing but reduces its quality
- **start date** - you can set a date earlier than the current date in
    order to verify how the selected algorithm would work on historical
    data
- **Scheduler** - a tag if the rule should be run according to the plan
    for the scheduler. If selected, additional fields will appear;

![](/media/media/image67.png)

- **Prediction cycle** - plan definition for the scheduler, i.e. the
    cycle in which the prediction rule is run (e.g. once a day, every
    hour, once a week). In the field, enter the command that complies
    with the cron standard. Enable -- whether to immediately launch
    the scheduler plan or save only the definition
- **Role** - only users with the roles selected here and the
    administrator will be able to run the defend AI rules The selected
    „time frame" also affects the prediction period. If we choose
    "time frame = monthly", we will be able to predict a one month
    ahead from the moment of prediction (according to the "prediction
    cycle" value)
## Screen content for the Trend algorithm ##

![](/media/media/image68_js.png)

Description of controls:

- **feature to analyze from search** - analyzed feature (dictated)
- **multiply by field** - enable multiplication of algorithms after
    unique values of the feature indicated here. Multiplication allows
    you to run the AI rule one for e.g. all servers. The value "none" in
    this field means no multiplication.
- **multiply by values** - if a trait is indicated in the „multiply by
    field", then unique values of this trait will appear in this field.
    Multiplications will be made for the selected values. If at least
    one of value is not selected, the „Run" buttons will be inactive.\`

In other words, multiplication means performing an analysis for many values from the indicated field, for example: `sourece_node_host`- which we indicate in `Multiply by field (from search)`.

However, in `Multiply by values (from search)` we already indicate values of this field for which the analysis will be performed, for example: host1, host2, host3, ....

- **time frame** - feature aggregation method (1 minute, 5 minute, 15
    minute, 30 minute, hourly, weekly, monthly, 6 months, 12 months)
- **max probes** - how many samples back will be taken into account for
    analysis. A single sample is an aggregated data according to the
    aggregation method.
- **value type** - which values to take into account when aggregating for
    a given time frame (e.g. maximum from time frame, minimum, average)
- **max predictions** - how many estimates we make for ahead (we take
    time frame)
- **data limit** - limits the amount of date downloaded from the source.
    It speeds up processing but reduces its quality
- **start date** - you can set a date earlier than the current date in
    order to verify how the selected algorithm would work on historical
    data
- **Scheduler** - a tag if the rule should be run according to the plan
    for the scheduler. If selected, additional fields will appear;

![](/media/media/image67.png)

- **Prediction cycle** - plan definition for the scheduler, i.e. the
    cycle in which the prediction rule is run (e.g. once a day, every
    hour, once a week). In the field, enter the command that complies
    with the cron standard. Enable -- whether to immediately launch
    the scheduler plan or save only the definition
- **Role** - only users with the roles selected here and the
    administrator will be able to run the defend AI rules The selected
    „time frame" also affects the prediction period. If we choose
    "time frame = monthly", we will be able to predict a one month
    ahead from the moment of prediction (according to the "prediction
    cycle" value)
- **Threshold** - default values -1 (do not search). Specifies the
    algorithm what level of exceeding the value of the feature „feature
    to analyze from cheese" is to look for. The parameter currently used
    only by the "Trend" algorithm.
## Screen content for the neural network (MLP) algorithm ##

![](/media/media/image69_js.png)

Descriptions of controls:

- **Name** - name of the learned neural network
- **Choose search** - search defined in CyberX, which is used
    to select a set of data on which the rule of artificial intelligence
    will work
- **Below**, on the left, a list of attributes and their weights based on
    teaching ANN will be defined during the teaching. The user for each
    attribute will be able to indicate the field from the above
    mentioned search, which contain the values of the attribute and
    which will be analyzed in the algorithm. The presented list (for
    input and output attributes) will have a static and dynamic part.
    Static creation by presenting key with the highest weights. The key
    will be presented in the original form, i.e. perf\_data./ The second
    part is a DropDown type list that will serve as a key update
    according to the user's naming. On the right side, the attribute
    will be examined in a given rule / pattern. Here also the user must
    indicate a specific field from the search. In both cases, the input
    and output are narrowed based on the search fields indicated in
    Choose search.
- **Data limit** - limits the amount of data downloaded from the source.
    It speeds up the processing, but reduces its quality.
- **Scheduler** - a tag if the rule should be run according to the plan
    or the scheduler. If selected, additional fields will appear:

![](/media/media/image67.png)

- **Prediction cycle** - plan definition for the scheduler, i.e. the
     cycle in which the prediction rule is run (e.g. once a day, every
     hour, once a week). In the field, enter the command that complies
     with the *cron* standard
- **Enable** - whether to immediately launch the scheduler plan or save
     only the definition
- **Role** - only users with the roles selected here and the
     administrator will be able to run the defined AI rules
## AI Rules List ##

![](/media/media/image70.png)

Column description:

- **Status**:
    - ![](/media/media/image71.png)- the process is being processed (the pid of the process is in brackets)
    - ![](/media/media/image72.png) - process completed correctly
    - ![](/media/media/image73.png) - the process ended with an error
- **Name** - the name of the rule
- **Search** - the search on which the rule was run
- **Method** - an algorithm used in the AI rule
- **Actions** - allowed actions:
    - **Show** - preview of the rule definition
    - **Enable/Disable** - rule activation /deactivation
    - **Delete** - deleting the rule
    - **Update** - update of the rule definition
    - **Preview** - preview of the prediction results (the action is
        available after the processing has been completed correctly).
## AI Learn ##

![](/media/media/image74.png)

Description of controls:

- **Search** - a source of data for teaching the network
- **prefix name** - a prefix added to the id of the learned model that
  allows the user to recognize the model
- **Input cols** - list of fields that are analyzed / input features.
  Here, the column that will be selected in the output col should
  not be indicated. Only those columns that are related to processing
  should be selected. **
- **Output col** - result field, the recognition of which is learned by
  the network. **This field should exist in the learning and testing
  data, but in the production data is unnecessary and should not
  occur. This field cannot be on the list of selected fields in "input
  col".**
- **Output class category** - here you can enter a condition in SQL
  format to limit the number of output categories e.g. `if((outputCol)
  \< 10,(floor((outputCol))+1), Double(10))`. This condition limits the
  number of output categories to 10. **Such conditions are necessary
  for fields selected in "output col" that have continuous values.
  They must necessarily by divided into categories. In the Condition,
  use your own outputCol name instead of the field name from the index
  that points to the value of the "output col" attribute.**
- **Time frame** - a method of aggregation of features to improve their
  quality (e.g. 1 minute, 5 minutes, 15 minutes, 30 minutes, 1 hour, 1
  daily).
- **Time frames output shift** - indicates how many time frame units to
  move the output category. This allows teaching the network with
  current attributes, but for categories for the future.
- **Value type** - which values to take into account when aggregating for
  a given time frame (e.g. maximum from time frame, minimum, average)
- **Output class count**- the expected number of result classes. **If
  during learning the network identifies more classes than the user
  entered, the process will be interrupted with an error, therefore it
  is better to set up more classes than less, but you have to keep in
  mind that this number affects the learning time.**
- **Neurons in first hidden layer (from, to)** - the number of neurons in
  the first hidden layer. Must have a value \> 0. Jump every 1.
- **Neurons in second hidden layer (from, to)** - the number of neurons
  in second hidden layer. If = 0, then this layer is missing. Jump
  every 1.
- **Neurons in third hidden layer (from, to)** - the number of neurons in
  third hidden layer. If = 0 then this layer is missing. Jump every 1.
- **Max iter** (from, to) - maximum number of network teaching
  repetitions (the same data is used for learning many times in
  internal processes of the neural network). The slower it is. Jump
  every 100. The maximum value is 10, the default is 1.
- **Split data to train&test** - for example, the entered value of 0.8
  means that the input data for the network will be divided in the
  ratio 0.8 to learning, 0.2 for the tests of the network learned.
- **Data limit** - limits the amount of data downloaded from the source.
  It speeds up the processing, but reduces its quality.
- **Max probes** - limits the number of samples taken to learn the
  network. Samples are already aggregated according to the selected
  "Time frame" parameter. It speed up teaching but reduces its
  quality.
- **Build** - a button to start teaching the network. The button contains
  the number of required teaching curses. You should be careful and
  avoid one-time learning for more than 1000 courses. It is better to
  divide them into several smaller ones. One pass after a full data
  load take about 1-3 minutes on a 4 core 2.4.GHz server.
**The module has implemented the best practices related to the number
of neurons in individual hidden layers. The values suggested by the
system are optimal from the point of view of these practices, but the
user can decide on these values himself.**

Under the parameters for learning the network there is an area in
which teaching results will appear.

After pressing the "Refresh" button, the list of the resulting models
will be refreshed.

Autorefresh - selecting the field automatically refreshes the list of
learning results every 10s.

The following information will be available in the table on the left:

- **Internal name** - the model name given by the system, including the
    user - specified prefix

- **Overall efficiency** - the network adjustment indicator - allow to
    see at a glance whether it is worth dealing with the model. The
    grater the value, the better.

After clicking on the table row, detailed data collected during the
learning of the given model will be displayed. This data will be
visible in the box on the right.

The selected model can be saved under its own name using the "Save
algorithm" button. This saved algorithm will be available in the
"Choose AI Rule" list when creating the rule (see Create AI Rule).
## AI Learn Tasks ##

The "AI Learn Task" tab shows the list of processes initiated teaching the ANN network
with the possibility of managing processes.

Each user can see only the process they run. The user in the role of
Intelligence sees all running processes.

![](/media/media/image75.png)

Description of controls:

- **Algorithm prefix** - this is the value set by the user on the AI
    Learn screen in the Prefix name field
 - **Progress** - here is the number of algorithms generated / the number of all to be generated
- **Processing time** - duration of algorithm generation in seconds (or
     maybe minutes or hours)
- **Actions**:
    - **Cancel** - deletes the algorithm generation task (user require
        confirmation of operation)
    - **Pause / Release** - pause / resume algorithm generation process.

AI Learn tab contain the Show in the preview mode of the ANN hyperparameters
After completing the learning activity or after the user has interrupted it, 
the "Delete" button appears in "Action" field. This button allows you to permanently 
delete the learning results of a specific network.

![](/media/media/image76.png)
## Scenarios of using algorithms implemented in the Intelligence module ##


### Teaching MLP networks and choosing the algorithm to use: ###

1. Go to the AI Learn tab,
1. We introduce the network teaching parameters,
1. Enter your own prefix for the names of the algorithms you have learned,
1. Press Build.
1. We observe the learned networks on the list (we can also stop the observation at any moment and go to other functions of the system. We will return to the learning results by going to the AI Learn Tasks tab and clicking the show action),
1. We choose the best model from our point of view and save it under our own name,
1. From this moment the algorithm is visible in the Create AI Rule tab.

### Starting the MLP network algorithm: ###

1. 	Go to the Create AI Rule tab and create rules,
1. 	Select the previously saved model of the learned network,
1. 	Specify parameters visible on the screen (specific to MLP),
1. 	Press the Run button.

### Starting regression algorithm: ###

1. Go to the Create AI Rule tab and create rules,
1. We choose AI Rule, e.g. Simple Moving Average, Linear Regression or Random Forest Regression, etc.,
1. Enter your own rule name (specific to regression),
1. Set the parameters of the rule ( specific to regression),
1. Press the Run button.

### Management of available rules: ###

1. Go to the AI Rules List tab,
1. A list of AI rules available for our role is displayed,
1. We can perform the actions available on the right for each rule.# Results of algorithms #

The results of the "AI algorithms" are saved to the index „intelligence" specially created
for this purpose. The index with the prediction result. These
following fields are available in the index (where xxx is the name of
the attribute being analyzed):

-   **xxx\_pre** - estimate value
-   **xxx\_cur** - current value at the moment of estimation
-   **method\_name** - name of the algorithm used
-   **rmse** - avarage square error for the analysis in which \_cur values
     were available. **The smaller the value, the better.**
-   **rmse\_normalized** - mean square error for the analysis in which
     \_cur values were available, normalized with \_pre values. **The
     smaller the value, the better.**
-   **overall\_efficiency** - efficiency of the model. **The greater the
     value, the better. A value less than 0 may indicate too little
     data to correctly calculate the indicator**
-   **linear\_function\_a** - directional coefficient of the linear
     function y = ax + b. **Only for the Trend and Linear Regression
     Trend algorithm**
-   **linear\_function\_b** - the intersection of the line with the Y axis
     for the linear function y = ax + b. **Only for the Trend and
     Linear Regression Trend algorithm.**

Visualization and signals related to the results of data analysis
should be created from this index. The index should be available to
users of the Intelligence module.
## Scheduler Module ##

CyberX has a built-in task schedule. In this module, we can
define a command or a list of commands whose execution we instruct the
application in the form of tasks. We can determine the time and
frequency of tasks. Tasks can contain a simple syntax, but they can
also be associated with modules, e.g. with Intelligence module.

To go to the Scheduler window, select the tile icon from the main menu
bar and then go to the „Scheduler" icon (To go back, go to the
„Search" icon)

![](/media/media/image38_js5.png)

The page with three tabs will be displayed: Creating new tasks in the
„Create Scheduler Job", managing tasks in the „Job List" and checking
the status of tasks in „Jobs Status"

In the window for creating new tasks we have a form consisting of
fields:

- **Name** - in which we enter the name of the task
- **Cron Pattern** - a field in which in cron notation we define the time
   and frequency of the task
- **Command** - we give the syntax of the command that will be executed
   in this task. These can be simple system commands, but also
   complex commands related to the Intelligence module. In the task
   management window, we can activate /deactivate, delete and update
   the task by clicking on the selected icon for a given task
   ![](/media/media/image63.png)

In the task status windows you can check the current status of the
task: if it activated, when it started and when it ended, how long it
took. This window is not editable and indicates historical data.
## Permission ##

Permission have been implemented in the following way:

- Only the user in the admin role can create / update rules.
- When creating rules, the roles that will be able to enables /
  disengage / view the rules will be indicated.

We assume that the Learn process works as an administrator.

We assume that the visibility of Search in AI Learn is preceded by
receiving the search permission in the module object permission.

The role of "Intelligence" launches the appropriate tabs.

An ordinary user only sees his models. The administrator sees all
models.

## Register new algorithm ##

For register new algorithm:

- **Login** to the CyberX
- Select **Intelligence**
- Select **Algorithm**
- Fill Create algorithm form and press **Submit** button

Form fields:

	| Field   | Description                                                                                                      |
	|---------|------------------------------------------------------------------------------------------------------------------|
	| Code    | Short name for algorithm                                                                                         |
	| Name    | Algorithm name                                                                                                   |
	| Command | Command to execute. The command must be in the directory pointed to by the parameter elastscheduler.commandpath. |

CyberX execute command:

	<command> <config> <error file> <out file>

Where:

- command	-	Command from command filed of Create algorithm form.
- config		-	Full path of json config file. The name of file is id of process status document in index .intelligence_rules
- error file	-	Unique name for error file. Not used by predefined algorithms.
- out file		-	Unique name for output file. Not used by predefined algorithms.

Config file:

Json document:

	| Field                  | Value                                                                               | Screen field (description)                                           |
	|------------------------|-------------------------------------------------------------------------------------|----------------------------------------------------------------------|
	| algorithm_type         | GMA, GMAL, LRS, LRST, RFRS, SMAL, SMA, TL                                           | Algorithm. For customs method field Code from Create algorithm form. |
	| model_name             | Not empty string.                                                                   | AI Rule Name.                                                        |
	| search                 | Search id.                                                                          | Choose search.                                                       |
	| label_field.field      |                                                                                     | Feature to analyse.                                                  |
	| max_probes             | Integer value                                                                       | Max probes                                                           |
	| time_frame             | 1 minute, 5 minutes, 15 minutes, 30 minutes, 1 hour, 1 day, 1 week, 30 day, 365 day | Time frame                                                           |
	| value_type             | min, max, avg, count                                                                | Value type                                                           |
	| max_predictions        | Integer value                                                                       | Max predictions                                                      |
	| threshold              | Integer value                                                                       | Threshold                                                            |
	| automatic_cron         | Cron format string                                                                  | Automatic cycle                                                      |
	| automatic_enable       | true/false                                                                          | Enable                                                               |
	| automatic              | true/false                                                                          | Automatic                                                            |
	| start_date             | YYYY-MM-DD HH:mm or now                                                             | Start date                                                           |
	| multiply_by_values     | Array of string values                                                              | Multiply by values                                                   |
	| multiply_by_field      | None or full field name eg.: system.cpu                                             | Multiply by field                                                    |
	| selectedroles          | Array of roles name                                                                 | Role                                                                 |
	| last_execute_timestamp |                                                                                     | Last execute                                                         |

	| Not screen fields     |                                     |
	|-----------------------|-------------------------------------|
	| preparation_date      | Document preparation date.          |
	| machine_state_uid     | AI rule machine state uid.          |
	| path_to_logs          | Path to ai machine logs.            |
	| path_to_machine_state | Path to ai machine state files.     |
	| searchSourceJSON      | Query string.                       |
	| processing_time       | Process operation time.             |
	| last_execute_mili     | Last executed time in milliseconds. |
	| pid                   | Process pid if ai rule is running.  |
	| exit_code             | Last executed process exit code.    |

The command must update the process status document in the system during operation. It is elastic partial document update.

	| Process status         | Field (POST body)          | Description                                    |
	|------------------------|----------------------------|------------------------------------------------|
	| START                  | doc.pid                    | System process id                              |
	|                        | doc.last_execute_timestamp | Current timestamp. yyyy-MM-dd HH:mm            |
	|                        | doc.last_execute_mili      | Current timestamp in millisecunds.             |
	| END PROCESS WITH ERROR | doc.error_description      | Error description.                             |
	|                        | doc.error_message          | Error message.                                 |
	|                        | doc.exit_code              | System process exit code.                      |
	|                        | doc.pid                    | Value 0.                                       |
	|                        | doc.processing_time        | Time of execute process in seconds.            |
	| END PROCESS OK         | doc.pid                    | Value 0.                                       |
	|                        | doc.exit_code              | System process exit code. Value 0 for success. |
	|                        | doc.processing_time        | Time of execute process in seconds.            |

The command must insert data for prediction chart.

	| Field             | Value             | Description                                            |
	|-------------------|-------------------|--------------------------------------------------------|
	| model_name        | Not empty string. | AI Rule Name.                                          |
	| preparationUID    | Not empty string. | Unique prediction id                                   |
	| machine_state_uid | Not empty string. | AI rule machine state uid.                             |
	| model_uid         | Not empty string. | Model uid from config file                             |
	| method_name       | Not empty string. | User friendly algorithm name.                          |
	| <field>           | Json              | Field calculated. For example: system.cpu.idle.pct_pre |

Document sample:

			{
			  "_index": "intelligence",
			  "_type": "doc",
			  "_id": "emca_TL_20190304_080802_20190531193000",
			  "_version": 2,
			  "_score": null,
			  "_source": {
			    "machine_state_uid": "emca_TL_20190304_080802",
			    "overall_efficiency": 0,
			    "processing_time": 0,
			    "rmse_normalized": 0,
			    "predictionUID": "emca_TL_20190304_080802_20190531193000",
			    "linear_function_b": 0,
			    "@timestamp": "2019-05-31T19:30:00.000+0200",
			    "linear_function_a": 0.006787878787878788,
			    "system": {
			      "cpu": {
			        "idle": {
			          "pct_pre": 0.8213333333333334
			        }
			      }
			    },
			    "model_name": "emca",
			    "method_name": "Trend",
			    "model_uid": "emca_TL_20190304_080802",
			    "rmse": 0,
			    "start_date": "2019-03-04T19:30:01.279+0100"
			  },
			  "fields": {
			    "@timestamp": [
			      "2019-05-31T17:30:00.000Z"
			    ]
			  },
			  "sort": [
			    1559323800000
			  ]
			}
