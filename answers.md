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


## Collecting Metrics:

1. Adding tags to the Agent can be done through the config file at /etc/datadog-agent/datadog.yaml. 
	The file will include the API key and can hold other relavant tags. 
	I have designated a hostname and added an 'anviorment' and 'machineGourp' tags.  

	```
	hostname: BorisMachine

	tags:
 	  - environment:sales-env
 	  - machineGroup:boris1
	```

	- Yaml files are extremly sensitive to syntax and stracture issues, it is recomanded to back up the file before changing or adding anything to it. 
	
2. Restart the agent service.

	```
	sudo service datadog-agent restart
	```

3. Make sure that the new information is provided in the host map. 

	<img src="https://github.com/barkanb/hiring-engineers/blob/master/Images/agent-4.png" width="80%" height="80%"></a>

