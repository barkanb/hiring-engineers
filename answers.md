These are the answers and the procedures to accomplish all of the tasks provided in the 'hiring-engineers' exercise . 

* Shared [Dashboard](https://p.datadoghq.com/sb/v730f09bg4m43c7e-b6b7ab5e5d58a5079cf2a7c4246d1b32)




## The Exercise

The answers will be provided as if they are presented as a guide to a client. It will be structured with implementation steps and notes. 
I will also add "developer" notes that should be considered outside of the customer scope and they are intended for Datadog reviews. 

## Prerequisites 

* For the following instructions Ubuntu running on Vagrant was used. 

	```
	Linux vagrant 4.15.0-58-generic #64-Ubuntu SMP Tue Aug 6 11:12:41 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
	```

	```
	DISTRIB_ID=Ubuntu
	DISTRIB_RELEASE=18.04
	DISTRIB_CODENAME=bionic
	DISTRIB_DESCRIPTION="Ubuntu 18.04.3 LTS"
	NAME="Ubuntu"
	VERSION="18.04.3 LTS (Bionic Beaver)"
	ID=ubuntu
	ID_LIKE=debian
	PRETTY_NAME="Ubuntu 18.04.3 LTS"
	VERSION_ID="18.04"
	HOME_URL="https://www.ubuntu.com/"
	SUPPORT_URL="https://help.ubuntu.com/"
	BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
	PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
	VERSION_CODENAME=bionic
	UBUNTU_CODENAME=bionic
	```

* For integration purposes MySQL was installed on the OS. 

	```
	mysql  Ver 14.14 Distrib 5.7.30, for Linux (x86_64) using  EditLine wrapper
	```

* Also make sure that you have an active account with [Datadog](https://app.datadoghq.com/). 

## Installing the agent on the OS

* Instructions are provided in the [Datadog portal](https://app.datadoghq.com/) 

		a. Integration Tab
		<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-1.png" width="40%" height="40%"></a>

		b. Ubuntu instructions. 
		<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-2.png" width="90%" height="90%"></a>

1. One line install: 

	```
	$ DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX bash -c "$(curl -L https://raw.githubusercontent.com/DataDog/datadog-agent/master/cmd/agent/install_script.sh)"
	```

	The portal will provide a command that will incorporate the right API key. 

2. Make sure that the agent was installed correctly and the following message is displayed: 

	```
	Your Agent is running and functioning properly. It will continue to run in the background and submit metrics to Datadog.
	```

3. To check the status of the agent please run the following: 

	```
	$ sudo service datadog-agent status
	```

	Response: 
	```
	● datadog-agent.service - Datadog Agent
	Loaded: loaded (/lib/systemd/system/datadog-agent.service; enabled; vendor preset: enabled)
	Active: active (running) since Fri 2020-06-19 00:04:16 UTC; 3min 23s ago
	Main PID: 2344 (agent)
	Tasks: 9 (limit: 1109)
           ```

4. The machine should appear in the portal's main page and the host map: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-3.png" width="80%" height="80%"></a>




## Collecting Metrics- Adding Tags:

1. Adding tags to the Agent can be done through the config file at /etc/datadog-agent/datadog.yaml. 
	The file will include the API key and can hold other relevant tags. 
	I have designated a hostname and added an 'environment' and 'machineGourp' tags.  

	```
	hostname: BorisMachine

	tags:
 	  - environment:sales-env
 	  - machineGroup:boris1
	```

	- The file will look similar to this: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-5.png" width="55%" height="55%"></a>


	- **Yaml files are extremely sensitive** to syntax and structure issues, it is recommended to back up the file before changing or adding anything to it. 
	
2. Restart the agent service.

	```
	$ sudo service datadog-agent restart
	```

3. Make sure that the new information is provided in the host map. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-4.png" width="80%" height="80%"></a>




## Collecting Metrics- MySQL Integration: 

* Follow the instructions on the portal to integrate with MySQL. 
  The instructions bellow are for MySQL 8.0+, for other version please refer to the portal for instructions. 

  	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-6.png" width="80%" height="80%"></a>

* Make sure the agent is installed and running on the machine. 

1. Create a user with proper permissions to be used by the agent. 

	```
	mysql> CREATE USER 'datadog'@'localhost' IDENTIFIED WITH mysql_native_password by '<UNIQUEPASSWORD>';
	Query OK, 0 rows affected (0.00 sec)
	```

2. Verify user creation with the following commands:

	```
	mysql> SELECT User FROM mysql.user
	    -> ;
	+------------------+
	| User             |
	+------------------+
	| datadog          |
	| debian-sys-maint |
	| mysql.session    |
	| mysql.sys        |
	| root             |
	+------------------+
	``` 

	```
	mysql -u datadog --password=<UNIQUEPASSWORD> -e "show status" | \
	grep Uptime && echo -e "\033[0;32mMySQL user - OK\033[0m" || \
	echo -e "\033[0;31mCannot connect to MySQL\033[0m"
	```

	```
	mysql -u datadog --password=<UNIQUEPASSWORD> -e "show slave status" && \
	echo -e "\033[0;32mMySQL grant - OK\033[0m" || \
	echo -e "\033[0;31mMissing REPLICATION CLIENT grant\033[0m"
	```
3. Grant the user more limited privileges:

	```
	mysql> GRANT REPLICATION CLIENT ON *.* TO 'datadog'@'localhost' WITH MAX_USER_CONNECTIONS 5;
	Query OK, 0 rows affected, 1 warning (0.00 sec)

	mysql> GRANT PROCESS ON *.* TO 'datadog'@'localhost';
	Query OK, 0 rows affected (0.00 sec)
	```

	```
	mysql> ALTER USER 'datadog'@'localhost' WITH MAX_USER_CONNECTIONS 5;
	Query OK, 0 rows affected (0.00 sec)
	```

4. If enabled, metrics can be collected from the performance_schema database by granting an additional privilege:

	```
	mysql> show databases like 'performance_schema';
	+-------------------------------+
	| Database (performance_schema) |
	+-------------------------------+
	| performance_schema            |
	+-------------------------------+
	1 row in set (0.00 sec)

	mysql> GRANT SELECT ON performance_schema.* TO 'datadog'@'localhost';
	Query OK, 0 rows affected (0.00 sec)
	```

5. Configure the agent to collect and send data with a yaml configuration file at /etc/datadog-agent/conf.d/mysql.d/conf.yaml

	```
	init_config:

	instances:
	  - server: 127.0.0.1
	    user: datadog
	    pass: "<YOUR_CHOSEN_PASSWORD>" # from the CREATE USER step earlier
	    port: "<YOUR_MYSQL_PORT>" # e.g. 3306
	    options:
	      replication: false
	      galera_cluster: true
	      extra_status_metrics: true
	      extra_innodb_metrics: true
	      extra_performance_metrics: true
	      schema_size_metrics: false
      	  disable_innodb_metrics: false
	```

	* Example: 

			<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-7.png" width="50%" height="50%"></a>

6. Restart the agent service.

	```
	$ sudo service datadog-agent restart
	```

7. Make sure that the new information is provided in the host map and that the service is communicating. 

	- The host should display MySQL related information and other metrics. 

  	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-8.png" width="70%" height="70%"></a>

  	- The Status Check should also indicate if the integration is operating correctly. 

  	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-9.png" width="70%" height="70%"></a>





## Collecting Metrics- Custom Agent Check: 

1. In order to create a custom check, we need to create a check file that can gather and send the information.
	For this example I have created a Python script that generates a random number and sends it to the portal. 

	* Make sure that the Datadog python package is installed. 

	The script bellow uses the Datadog library to send a random number between 0 and 1000. 

	```python
	# the following try/except block will make the custom check compatible with any Agent version
	try:
	    # first, try to import the base class from new versions of the Agent...
	    from datadog_checks.base import AgentCheck
	except ImportError:
	    # ...if the above failed, the check is running in Agent version < 6.6.0
	    from checks import AgentCheck

	import random

	# content of the special variable __version__ will be shown in the Agent status page
	__version__ = "1.0.0"

	class HelloCheck(AgentCheck):
	    def check(self, instance):
	        self.gauge('custom_check.my_metric', random.randint(0,1000), tags=['TAG_KEY:TAG_VALUE'])
    ```  

2. Place the script in /etc/datadog-agent/checks.d/ . I have named the file **custom_check.py**.

3. Place a yaml configuration file for the custom check at /etc/datadog-agent/conf.d   , 
   The name of the file needs to be the same as the script file, hence we will create a file named **custom_check.yaml**. 

4. In order to set an interval we will populate the file with the min_collection_interval attribute. 

	```
	instances:
  	  - min_collection_interval: 45
    ```

5. Restart the agent. 

	```
	$ sudo service datadog-agent restart
	```

6. Make sure that the check is working with the following command (replace <CHECK_NAME> with the proper check name as seen in the screenshot): 

	```
	$ sudo -u dd-agent -- datadog-agent check <CHECK_NAME>
	```

     <img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-10.png" width="70%" height="70%"></a>

7. The new check and the relevant information should appear in the host map. 
	Notice that the name of the file is not the one used. The API will determine the name of the namespace and the name of the custom check. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-11.png" width="70%" height="70%"></a>


	The custom check can also be without the name space. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-12.png" width="50%" height="50%"></a>


**Bonus Notes:**
The interval is set in the configuration file and not in the python script.  





## Visualizing Data:

1. In order to create a dashboard, a script needs to run with an API key and an APP key. 
	Navigate to the API tab and get the two keys. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-1.png" width="40%" height="40%"></a>


	The keys will be masks, but by hovering over the items the keys will be revealed. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-2.png" width="70%" height="70%"></a>



2. Define the metrics for the new dashboard: these will be used in the script below as we build 3 widgets in the new dashboard. 
	
	* Custom metric my_metric : avg:my_metric{*} 
		- This will be used to get my_metric information which is a random number between 0 and 1000. 
	
	* MySQL connections : anomalies(sum:mysql.net.max_connections{*}, \"basic\", 3, direction=\'above\')
		- Looks at the number of MySQL connections, and detects anomalies with that number. 

	* Custom metric my_metric the rollup function applied, sums up all the points for the past hour : avg:my_metric{*}.rollup(\"sum\", 3600)
		- Gets the sum of my_metric for each hour. 

3. Script to create the dashboard: 

	**Notice the following**
	- API and APP keys are added to the script. 
	- The graphs (widgets) are added as a list that includes the metrics (as identified above in section 2, name and other useful information for display. 



	```python
	from datadog import initialize, api

	options = {
	    'api_key': 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
	    'app_key': 'YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY'
	}

	initialize(**options)

	title = 'Timeboard v2'
	widgets = [
	{
	    'definition': 
	    {
	        'type': 'timeseries',
	        'requests': [
	            {'q': 'avg:my_metric{*}'}
	        ],
	        'title': 'Custom metric my_metric'
	    }
	},{
	    'definition': 
	    {
	        'type': 'timeseries',
	        'requests': [
	            {'q': 'anomalies(sum:mysql.net.max_connections{*}, \"basic\", 3, direction=\'above\')'}     
	        ],
	        'title': 'MySQL connections'
	    }
	},{
	    'definition': 
	    {
	        'type': 'timeseries',
	        'requests': [
	            {'q': 'avg:my_metric{*}.rollup(\"sum\", 3600)'}     
	        ],
	        'title': 'Custom metric my_metric the rollup function applied, sums up all the points for the past hour'
	    }
	}
	]
	layout_type = 'ordered'
	description = 'A dashboard with custom metric information'
	is_read_only = True
	notify_list = ['barkanb@gmail.com']
	template_variables = [{
	    'name': 'host1',
	    'prefix': 'host',
	    'default': 'my-host'
	}]

	saved_view = [{
	    'name': 'Saved views for hostname 2',
	    'template_variables': [{'name': 'host', 'value': '<HOSTNAME_2>'}]}
	]

	api.Dashboard.create(title=title,
	                     widgets=widgets,
	                     layout_type=layout_type,
	                     description=description,
	                     is_read_only=is_read_only,
	                     notify_list=notify_list,
	                     template_variables=template_variables
	                     )
	```

4. Check the widgets on the newly created dashboard- 15 min intervals.

	The following image provides an example for how the graphs should look like, please notice that the interval can be changed on top. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-3.png" width="70%" height="70%"></a>

5. Once the interval is changed you will see the following dashboard.

	Notice that the rollup widget might not display anything since the values are gathered at the hour while our interval window is 15 minutes.

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-4.png" width="70%" height="70%"></a>

6. The widget can be accessed individually to provide more information, as shown below: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-5.png" width="70%" height="70%"></a>

**Bonus Notes**

The anomaly metric can help in identifying and displaying deviating values from the regular plot of the values. 
In the following SQL connection graph we can see that the rise in connections is identified with a red line, that goes out of the shaded line that that designates the standard value plot. 

<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-6.png"></a>

For more information please refer to the [online documentation](https://docs.datadoghq.com/monitors/monitor_types/anomaly/#overview). 

7. To share the graphs, use the sharing option provided on the dashboard. You can use the '@' notation to send an email. 

 	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-7.png" width="70%" height="70%"></a>

 	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-8.png" width="70%" height="70%"></a>

 	* The email will look like this: 

 	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-9.png" width="70%" height="70%"></a>

 	* It will also create an event to show that an item was shared: 

 	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-10.png" width="70%" height="70%"></a>





## Monitoring Data

1. In order to create a metric monitor:
   - Access the monitor tab.
   - Click on "New Monitor".
   - Create a new metric monitor. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/monitor-1.png" width="85%" height="85%"></a>


	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/monitor-2.png" width="60%" height="60%"></a>

2. Create this monitor to look into the values of my_metric and define alerts based on threshold values. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/monitor-3.png" width="70%" height="70%"></a>

3. Set alert conditions as provided in the screenshot below: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/monitor-4.png" width="70%" height="70%"></a>

4. Customize the alerts to reflect the type of alert and provide alert information. 
	
	* Tags could be used to create different messages based on whether the monitor is in an Alert, Warning, or No Data state.

		Tags could also be used to display alert information such as the host and the value. 

	  	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/monitor-5.png" width="70%" height="70%"></a>

5. Set email notification and test it: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/monitor-7.png" width="80%" height="80%"></a>

	The emails will look as follows (test emails will say TEST):

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/monitor-8.png" width="60%" height="60%"></a>

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/monitor-9.png" width="60%" height="60%"></a>

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/monitor-10.png" width="60%" height="60%"></a>


6. Schedule downtimes for this monitor by going to the "Manage downtime" section:

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/time-1.png" width="80%" height="80%"></a>

7. Set the information for downtime on the monitor we just setup. 
	It will need to items since we want to schedule downtime during the week and the weekend. 

	* From 7pm to 9am daily on M-F. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/time-2.png" width="70%" height="70%"></a>

	* Saturday and Sunday. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/time-3.png" width="70%" height="70%"></a>

	* Each of the items can include a message. Example below: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/time-4.png" width="70%" height="70%"></a>


	- Email notification:

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/time-5.png" width="60%" height="60%"></a>



 

## Collecting APM Data:

1. To trace the provided application we will use Datadog's package - ddtrace. We can use pip install to get the package on the machine. 

	```
	pip install ddtrace
	```

	To get the instructions please navigate to the APN tab as shown below: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/trace-1.png" width="60%" height="60%"></a>


2. In our use case we will trace a Flask application. (Make sure that Flask is installed and runs the application correctly). 

	```python
		from flask import Flask
		import logging
		import sys

		# Have flask use stdout as the logger
		main_logger = logging.getLogger()
		main_logger.setLevel(logging.DEBUG)
		c = logging.StreamHandler(sys.stdout)
		formatter = logging.Formatter('%(asctime)s - %(name)s - %(levelname)s - %(message)s')
		c.setFormatter(formatter)
		main_logger.addHandler(c)

		app = Flask(__name__)

		@app.route('/')
		def api_entry():
		    return 'Entrypoint to the Application'

		@app.route('/api/apm')
		def apm_endpoint():
		    return 'Getting APM Started'

		@app.route('/api/trace')
		def trace_endpoint():
		    return 'Posting Traces'

		if __name__ == '__main__':
		    app.run(host='0.0.0.0', port='5050')
	```
	
	The following command will trace the application as it runs: 
	```
	ddtrace-run python app.py
	```

	Running the command will provide the following: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/trace-2.png" width="80%" height="80%"></a>

	- Notice that ddtrace was called from data-agent library, but it provides the same functionality. 


3. Access to the application routes will trigger traces. 

	* Triggering the app: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/trace-3.png" width="50%" height="50%"></a>

	* ddtrace output as the app is triggered: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/trace-4.png" width="80%" height="80%"></a>

4. Traces will appear in the Datadog portal: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/trace-5.png" width="80%" height="80%"></a>


**Bonus Note**
Services provides the application connections, they can do calculations, queries or jobs. 
Resources are the available items to be used by a service like static items or databases. 
 


## Final question

Datadog is great in collecting and displaying information for insights. I think it would be beneficial to use these capabilities to track Covid-19 cases across the US or even the world. 

As live feeds are provided by many resources, it might be of great value to study the results through the lenses of Datadog's products. 











