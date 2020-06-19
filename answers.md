These are the answers and the proceedures to accomplish all of the tasks provided in the 'hiring-engineers' excersize. 

## The Exercise

The answers will be provided as if they are presented as a guide to a client. It will be strcutured with implementation steps and notes. 
I will also add "developer" notes that should be considered outside of the customer scope and they are intended for Datadog reviers. 

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

<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-1.png" width="50%" height="50%"></a>
<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-2.png" width="80%" height="80%"></a>

1. One line install: 

	```
	DD_AGENT_MAJOR_VERSION=7 DD_API_KEY=8c550767f7a781935fcd14dac889d7dc bash -c "$(curl -L https://raw.githubusercontent.com/DataDog/datadog-agent/master/cmd/agent/install_script.sh)"
	```

	The portal will provide a command that will incoporate the right API key. 

2. Make sure that the agent was installed correctly and the following message is displayed: 

	```
	Your Agent is running and functioning properly. It will continue to run in the
	background and submit metrics to Datadog.
	```

3. To check the status of the agent please run the following: 

	```
	sudo service datadog-agent status
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
	The file will include the API key and can hold other relavant tags. 
	I have designated a hostname and added an 'anviorment' and 'machineGourp' tags.  

	```
	hostname: BorisMachine

	tags:
 	  - environment:sales-env
 	  - machineGroup:boris1
	```

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-5.png" width="50%" height="50%"></a>


	- Yaml files are extremly sensitive to syntax and stracture issues, it is recomanded to back up the file before changing or adding anything to it. 
	
2. Restart the agent service.

	```
	sudo service datadog-agent restart
	```

3. Make sure that the new information is provided in the host map. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-4.png" width="80%" height="80%"></a>


## Collecting Metrics- MySQL Integration: 

* Follow the instructions on the portal to integrate with MySQL. 
  The instructions bellow are for MySQL 8.0+, for other version please refer to the portal for instructions. 

  		<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-6.png" width="80%" height="80%"></a>


1. Create a user with proper permissions to be used by the agent. 
	* Make sure the agent is installed and running on the machine. 

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
	sudo service datadog-agent restart
	```

7. Make sure that the new information is provided in the host map and that the service is communicating. 

	
  	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-8.png" width="70%" height="70%"></a>
  	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-9.png" width="70%" height="70%"></a>



## Collecting Metrics- Custom Agent Check: 

1. In order to create a custom check, we need to create a file that can gather and send the information.
	For this example I have created a Python script that generates a random number and sends it to the portal. 

	* Make sure that the Datadog python package is installed. 

	The script bellow will use the Datadog library to send a random number between 0 and 1000. 

	```
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

2. Place the script in /etc/datadog-agent/checks.d/
   I have named the file custom_check.py.

3. Place a yaml configuration file for the custom check at /etc/datadog-agent/conf.d 
   The name of the file needs to be the same as the script file, hence we will create a file named custom_check.yaml. 

4. In order to set an interval we will populte the file with the min_collection_interval attribute. 

	```
	instances:
  	  - min_collection_interval: 45
    ```

5. Restart the agent. 

	```
	sudo service datadog-agent restart
	```

6. Make sure that the check is working: 

	```
	sudo -u dd-agent -- datadog-agent check <CHECK_NAME>
	```

     <img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-10.png" width="70%" height="70%"></a>

7. The new check and the relevant information should appear in the host map. 
	Notice that the name of the file is the namespace for the check "my_metric"

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-11.png" width="70%" height="70%"></a>

	The metric can also be set without a name space. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-12.png" width="40%" height="40%"></a>

**Bonus Notes:**
The interval is set in the configuration file and not in the python script.  

## Visualizing Data:

1. In order to create a dashbord, a script needs to run with an API key and an APP key. 
	Navigate to the API tab and get the two keys. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-1.png" width="40%" height="40%"></a>

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-2.png" width="60%" height="60%"></a>



2. Define the metrics for the new dashboard. 
	
	* Custom metric my_metric : avg:my_metric{*} 
		- Random number between 0 and 1000. 
	
	* MySQL connwctions : anomalies(sum:mysql.net.max_connections{*}, \"basic\", 3, direction=\'above\')
		- Looks at the number of connections, and looks for anomalies with that number. 

	* Custom metric my_metric the rollup function applied, sums up all the points for the past hour : avg:my_metric{*}.rollup(\"sum\", 3600)
		- Gets the sum of my_metric for each hour. 

3. Script to create the dashbord: 

	**Notice the following**
	- API and APP keys are added to the script. 
	- The graphs (widgets) are added as a list that includes the metrics, name and other usefull information for display. 

	```
	from datadog import initialize, api

	options = {
	    'api_key': '8c550767f7a781935fcd14dac889d7dc',
	    'app_key': '6dcf98bed5d32a0b6bddb30ddb1108e75a22d4e7'
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
	        'title': 'MySQL connwctions'
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

4. Check the widgets on the newly created dashbord- 15 min intervals. 
	The following image provides an example for how the graphs should look like, please notice that the inteval can be chnaged on top. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-3.png" width="60%" height="60%"></a>

5. Once the interval is changed you will see the following dashboard.
	Notice that the rollup widget will might not display anything since the values are gathered at the hour while our interval window is 15 minutes.

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-4.png" width="60%" height="60%"></a>

6. The widget can be accessed individually to provide more information, as shown below: 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-5.png" width="60%" height="60%"></a>

   **Bonus Notes**
   The anomaly metric can help in identifying and displaying when the values are diviating from the regular plot of the values. 
   In the following SQL connection grapth we can see that the rise in connections is identified with a red line, that goes out of the shaded line that that designates the standart value plot. 

   	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/metric-6.png" width="60%" height="60%"></a>




