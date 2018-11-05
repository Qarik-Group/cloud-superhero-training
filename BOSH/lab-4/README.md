## Lab 4
### Objectives:
In this lab we'll take a closer look at a virtual machine (vm) built and deployed by BOSH. We'll also follow up on our experiment from [Lab-3](../lab-3/README.md) where we successfully ran a "smoke-test" after killing the process that test was dependent on.

At the completion of this lab the student will:

* Be able to list the virtual machines (VMS) within a given environment.
* Know how to connect to an individual virtual machine (VM) and assume the role of an administrative user.
* Understand how to access the packages, jobs, and logs on a VM managed by BOSH
* Have an understanding of the role of the BOSH agent on a managed VM

### Prerequisites
* This lab assumes you have successfully completed the deployment of the nginx application described in [Lab-3](../lab-3/README.md).

### Finding and Connecting to VMS
When investigating issues or failed deployments it is often useful to connect to the instances BOSH creates to manually run scripts or examine system logs.

1. List the instances in the environment using the BOSH cli command `vms` resulting in an output like:

	```
	Using environment 'https://10.4.1.4:25555' as user 'admin' (openid, bosh.admin)
	
	Task 57
	Task 58
	Task 59
	Task 57 done
	
	Task 58 done
	
	Task 59 done
	
	Deployment 'nginx'
	
	Instance                                    Process State  AZ  IPs        VM CID               VM Type  Active
	nginx/14e444f1-48c0-4006-8908-7d535ee88bc0  running        z1  10.4.1.17  i-08e29a11984248a94  small    true
	
	1 vms
	
	Deployment 'training-concourse'
	
	Instance                                     Process State  AZ  IPs           VM CID               VM Type           Active
	db/2dfef814-cf29-4b2b-8372-11bc4b5ba2da      running        z3  10.4.0.20     i-09ed164d958621c6d  small             true
	web/9b38f597-db17-4711-b19c-9961a55e4c34     running        z3  10.4.0.7      i-09d2d1bcb4c1e1b7f  small             true
	                                                                10.4.0.19
	                                                                52.14.43.134
	worker/2a1b0155-1f65-4375-a0e2-d9f665adce74  running        z3  10.4.0.18     i-09f34f428a0cb5e1a  concourse-worker  true
	worker/7d07f1ae-1db0-40a3-ba62-e821420b6fdd  running        z3  10.4.0.17     i-04d633555e31b91f6  concourse-worker  true
	
	4 vms
	
	Deployment 'training-vault'
	
	Instance                                    Process State  AZ  IPs       VM CID               VM Type  Active
	vault/2561b860-6909-4dda-a3de-4d8b586d8294  running        z1  10.4.1.6  i-0f04cf0063cd44f30  default  true
	vault/842adc64-7587-40b7-812b-e026383aa7ac  running        z1  10.4.1.7  i-06e903636a9aab461  default  true
	
	2 vms
	
	Succeeded
	```

2. To connect to an instance with the `ssh` command, you need to first supply both the environment `--environment` and the deployment `--deployment`. Finally, you need to specify the instance to connect to which can be one of the following:

  - The instance role alone, if there is only one instance of that role in the deployment 

	  `-e training -d training-concourse ssh db`

  - The instance role followed by an zero-based index position

	  `-e training -d training-concourse ssh worker/1`

  - The instance role followed by the uuid

     `-e training -d training-concourse ssh worker/7d07f1ae-1db0-40a3-ba62-e821420b6fdd`

3. On connecting, you are automatically logged in as a dynamically created user that has minimal privileges. You can either run commands prefixed by `sudo` or switch to the "root" role using `sudo su -`.

### BOSH Filesystem
By convention, BOSH places all of the files associated with your release on the filesystem under `/var/vcap`; within this you'll find subdirectories for `packages`, `jobs`, and `sys`.

1. Connect one of your "nginx" VMS and as the superuser using the `sudo` command mentioned previously and change to the "/var/vcap/packages" directory.
2. Run the command `ls -lh`

	```bash
	lrwxrwxrwx 1 root root 70 Nov  5 22:58 nginx -> /var/vcap/data/packages/nginx/30ecedf1888e4ca6ee2df1cfd738f7d20ab8b6a1
	```
3. This is the nginx binary that we packaged back in [Lab-1](../lab-1/README.md). The package is actually stored as a shasum so that the BLOB can be safely and reliably be stored separately from the source code to avoid bloating code respositories. This also helps to ensure the correct version of a given package is installed and used by the VM. Examining the "nginx/" directory we find the standard layout we would expect for an nginx installation:
	
	```bash
	drwx------ 2 nobody root 4.0K Nov  5 22:58 client_body_temp
	drwxr-xr-x 2 root   root 4.0K Nov  5 22:58 conf
	drwx------ 2 nobody root 4.0K Nov  5 22:58 fastcgi_temp
	drwxr-xr-x 2 root   root 4.0K Nov  5 22:58 html
	drwxr-xr-x 2 root   root 4.0K Nov  5 22:58 logs
	drwx------ 2 nobody root 4.0K Nov  5 22:58 proxy_temp
	drwxr-xr-x 2 root   root 4.0K Nov  5 22:58 sbin
	drwx------ 2 nobody root 4.0K Nov  5 22:58 scgi_temp
	drwx------ 2 nobody root 4.0K Nov  5 22:58 uwsgi_temp
	```

4. Recall that our packaging script copied our config for "nginx" to "${BOSH\_INSTALL\_TARGET}/conf/", so if we view "/var/vcap/packages/nginx/conf/nginx.conf" we should see something like:

	```nginx
	user nobody vcap;
	error_log /var/vcap/sys/log/nginx/nginx.err.log;
	
	events {
	  worker_connections 1024;
	}
	
	http {
	  server {
	    listen *:80;
	    location / {
	      root /var/vcap/jobs/nginx/html;
	      index index.html;
	      access_log /var/vcap/sys/log/nginx/nginx.access.log combined;
	    }
	  }
	}
	```

5. Now challenge yourself to find the other files we worked with during packaging, specifically:
	- index.html
	- the control script for nginx
	- the smoke-test errand script
	- the monit config file
4. Use `wget` to make a request against nginx with the url `http://127.0.0.1`.
5. See if you can locate the nginx error and access logs.

### Monit
[Monit](https://mmonit.com/) is a tool for *Nix systems designed to help monitor and manage the behavior of services/applications running on a host. It provides rich configuration that allow monit to monitor processes for activity, traffic, and resource utilization and take preset actions based on those findings.

*TL;DR - It starts and keeps your processes up on your VMS.*

1. From the jumpbox run the `instances` BOSH command for your deployment. Like the `vms` command, this prints out the Process State for each of your VMS. This is an aggregate state of all processes running on the VM. Right now we only have "nginx" deployed, but if we had other co-located services, they would also be part of the aggregate.
2. Let's connect to one of your nginx VMS and run `monit summary`.

	```bash
	The Monit daemon 5.2.5 uptime: 16h 54m
	
	Process 'nginx'                     running
	System 'system_localhost'           running
	```
	
	This yields the current status of all individual processes that are running on the VM which can be useful during troubleshooting if one particular service is failing.

3. Let's stop our "nginx" process and then watch monit to see what happens:

	```bash
	/var/vcap/jobs/nginx/bin/ctl stop && watch monit status
	```

### The BOSH Agent
Every VM created by the director has a `bosh-agent` process. The BOSH Agent keeps tabs on process status and manages the VM while communicating back and forth with the director through the BOSH Registry.

Let's watch the BOSH Agent through its log file located at "/var/vcap/bosh/log/current".

###### BOSH and Monit
1. To get started we'll use `tail -f` to monitor the log.
	* There isn't a lot happening here right now since our instance is relatively quiet, but when the Agent is active such as during a heartbeat it produces a lot of traffic. We'll use `grep` to reduce the noise, and then create some traffic by stopping processes.
2. On a regular basis, the `bosh-agent` will ask monit to report on it's status so the agent can report up to the Registry, and by extension the Director:

	```bash
	grep -A 6 'monit status' /var/vcap/bosh/log/current
	```

3. If we restart monit, let's see how that affects the logs of the `bosh-agent`. Since `runit` is used for starting monit we need to use the `sv` command to restart monit:

	```bash
	sv stop monit
	```

4. Running our log monitoring command again will show output something like:

	```bash
	2018-11-06_16:38:36.16986 [monitJobSupervisor] 2018/11/06 16:38:36 DEBUG - Getting monit status
	2018-11-06_16:38:36.16988 [http-client] 2018/11/06 16:38:36 DEBUG - status function called
	2018-11-06_16:38:36.16988 [http-client] 2018/11/06 16:38:36 DEBUG - Monit request: url='http://127.0.0.1:2822/_status2?format=xml' body=''
	2018-11-06_16:38:36.16989 [attemptRetryStrategy] 2018/11/06 16:38:36 DEBUG - Making attempt #0 for *httpclient.RequestRetryable
	2018-11-06_16:38:36.16989 [clientRetryable] 2018/11/06 16:38:36 DEBUG - [requestID=eb14370c-c9ec-460f-7c99-dc3edff25279] Requesting (attempt=1): Request{ Method: 'GET', URL: 'http://127.0.0.1:2822/_status2?format=xml' }
	2018-11-06_16:38:36.49404 [attemptRetryStrategy] 2018/11/06 16:38:36 DEBUG - Making attempt #1 for *httpclient.RequestRetryable
	2018-11-06_16:38:36.49405 [clientRetryable] 2018/11/06 16:38:36 DEBUG - [requestID=8475e14b-3c55-459d-4c6b-9d6c92420d71] Requesting (attempt=2): Request{ Method: 'GET', URL: 'http://127.0.0.1:2822/_status2?format=xml' }
	```
	
	The output shows that the `bosh-agent` tries multiple times to connect to monit but those attempts fail since the service is offline.

5. If we check the instance status from our jumpbox we will get a message indicating that the agent is not healthy:

	```bash
	Using environment 'https://10.4.1.4:25555' as user 'admin' (openid, bosh.admin)
	
	Task 196. Done
	
	Deployment 'nginx'
	
	Instance                                    Process State       AZ  IPs
	nginx/14e444f1-48c0-4006-8908-7d535ee88bc0  unresponsive agent  z1  10.4.1.17
	
	1 instances
	```

6. Once `monit` has been restored using `sv start monit`, the `bosh-agent` will connect successfully and move on with its subsequent checks.
	
	```bash
	2018-11-06_16:39:06.17128 [monitJobSupervisor] 2018/11/06 16:39:06 DEBUG - Getting monit status
	2018-11-06_16:39:06.17129 [http-client] 2018/11/06 16:39:06 DEBUG - status function called
	2018-11-06_16:39:06.17129 [http-client] 2018/11/06 16:39:06 DEBUG - Monit request: url='http://127.0.0.1:2822/_status2?format=xml' body=''
	2018-11-06_16:39:06.17129 [attemptRetryStrategy] 2018/11/06 16:39:06 DEBUG - Making attempt #0 for *httpclient.RequestRetryable
	2018-11-06_16:39:06.17130 [clientRetryable] 2018/11/06 16:39:06 DEBUG - [requestID=baa47c4f-365a-485e-68e2-4bcb0bc17144] Requesting (attempt=1): Request{ Method: 'GET', URL: 'http://127.0.0.1:2822/_status2?format=xml' }
	2018-11-06_16:39:06.17151 [File System] 2018/11/06 16:39:06 DEBUG - Checking if file exists /var/vcap/monit/stopped
	```

###### BOSH and the OS Layer
When the VM fist spins up the BOSH agent is responsible for configuring the OS resources (e.g. network, disks) as needed. The BOSH agent then continues to run to keep the director informed on the VM status and to ensure services are running as expected.

1. Use `killall` to terminate the `bosh-agent` process.
2. Let's look at the logs to see what happened next

	```bash
	2018-11-06_18:13:37.15669 [NATS Handler] 2018/11/06 18:13:37 DEBUG - Message Payload
	2018-11-06_18:14:07.14076 [main] 2018/11/06 18:14:07 DEBUG - Starting agent
	2018-11-06_18:14:07.14362 [unlimitedRetryStrategy] 2018/11/06 18:14:07 DEBUG - Making attempt #0
	2018-11-06_18:14:07.14365 [DelayedAuditLogger] 2018/11/06 18:14:07 DEBUG - Starting logging to syslog...
	2018-11-06_18:14:07.14589 [httpClient] 2018/11/06 18:14:07 DEBUG - Sending GET request to endpoint 'http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key'
	2018-11-06_18:14:07.14591 [attemptRetryStrategy] 2018/11/06 18:14:07 DEBUG - Making attempt #0 for *httpclient.RequestRetryable
	2018-11-06_18:14:07.14593 [clientRetryable] 2018/11/06 18:14:07 DEBUG - [requestID=5fb39161-2f84-4d23-70ed-d50ec79b04f6] Requesting (attempt=1): Request{ Method: 'GET', URL: 'http://169.254.169.254/latest/meta-data/public-keys/0/openssh-key' }
	2018-11-06_18:14:07.15456 [settingsService] 2018/11/06 18:14:07 DEBUG - Loading settings from fetcher
	2018-11-06_18:14:07.15536 [httpClient] 2018/11/06 18:14:07 DEBUG - Sending GET request to endpoint 'http://169.254.169.254/latest/user-data'
	2018-11-06_18:14:07.15536 [attemptRetryStrategy] 2018/11/06 18:14:07 DEBUG - Making attempt #0 for *httpclient.RequestRetryable
	2018-11-06_18:14:07.15536 [clientRetryable] 2018/11/06 18:14:07 DEBUG - [requestID=fbe5c771-c4d0-46f7-5d81-4942b3cfd9c5] Requesting (attempt=1): Request{ Method: 'GET', URL: 'http://169.254.169.254/latest/user-data' }
	2018-11-06_18:14:07.15665 [registryProvider] 2018/11/06 18:14:07 DEBUG - Using http registry at http://registry-user:deBaTjI9cS5y9ar612OfY4atOwfGK9aqkvJ361NIRByxDyMxSdwgSPcY6TvoG7SS@10.4.1.4:25777
	2018-11-06_18:14:07.15715 [httpClient] 2018/11/06 18:14:07 DEBUG - Sending GET request to endpoint 'http://169.254.169.254/latest/meta-data/instance-id'
	2018-11-06_18:14:07.15715 [attemptRetryStrategy] 2018/11/06 18:14:07 DEBUG - Making attempt #0 for *httpclient.RequestRetryable
	2018-11-06_18:14:07.15715 [clientRetryable] 2018/11/06 18:14:07 DEBUG - [requestID=71d83f2d-e6f3-4aa4-5715-a6e01a0d3cb8] Requesting (attempt=1): Request{ Method: 'GET', URL: 'http://169.254.169.254/latest/meta-data/instance-id' }
	2018-11-06_18:14:07.15889 [httpClient] 2018/11/06 18:14:07 DEBUG - Sending GET request to endpoint 'http://169.254.169.254/latest/user-data'
	2018-11-06_18:14:07.15890 [attemptRetryStrategy] 2018/11/06 18:14:07 DEBUG - Making attempt #0 for *httpclient.RequestRetryable
	2018-11-06_18:14:07.15890 [clientRetryable] 2018/11/06 18:14:07 DEBUG - [requestID=e04f021a-bcb7-4d9e-6a7f-e9f83afe7beb] Requesting (attempt=1): Request{ Method: 'GET', URL: 'http://169.254.169.254/latest/user-data' }
	2018-11-06_18:14:07.15962 [httpClient] 2018/11/06 18:14:07 DEBUG - Sending GET request to endpoint 'http://registry-user:deBaTjI9cS5y9ar612OfY4atOwfGK9aqkvJ361NIRByxDyMxSdwgSPcY6TvoG7SS@10.4.1.4:25777/instances/i-08e29a11984248a94/settings'
	2018-11-06_18:14:07.15963 [attemptRetryStrategy] 2018/11/06 18:14:07 DEBUG - Making attempt #0 for *httpclient.RequestRetryable
	2018-11-06_18:14:07.15963 [clientRetryable] 2018/11/06 18:14:07 DEBUG - [requestID=645ed5d1-034c-4290-65ab-4c9d7766fdae] Requesting (attempt=1): Request{ Method: 'GET', URL: 'http://registry-user:deBaTjI9cS5y9ar612OfY4atOwfGK9aqkvJ361NIRByxDyMxSdwgSPcY6TvoG7SS@10.4.1.4:25777/instances/i-08e29a11984248a94/settings' }
	2018-11-06_18:14:07.16355 [settingsService] 2018/11/06 18:14:07 DEBUG - Successfully received settings from fetcher
	2018-11-06_18:14:07.19260 [interfaceConfigurationCreator] 2018/11/06 18:14:07 DEBUG - Creating network configuration with settings: type: 'manual', ip: '10.4.1.17', netmask: '255.255.255.0', gateway: '10.4.1.1', mac: '02:12:47:61:64:aa', resolved: 'true', preconfigured: 'false', use_dhcp: 'true'
	2018-11-06_18:14:07.19261 [interfaceConfigurationCreator] 2018/11/06 18:14:07 DEBUG - Using dhcp networking
	2018-11-06_18:14:07.19432 [arping] 2018/11/06 18:14:07 DEBUG - Broadcasting MAC addresses
	2018-11-06_18:14:14.99523 [virtioDevicePathResolver] 2018/11/06 18:14:14 DEBUG - Failed to get device real path by disk ID: ''. Error: 'Disk ID is not set', timeout: 'false'
	2018-11-06_18:14:14.99524 [virtioDevicePathResolver] 2018/11/06 18:14:14 DEBUG - Using mapped resolver to get device real path
	2018-11-06_18:14:14.99525 [linuxPlatform] 2018/11/06 18:14:14 DEBUG - Existing ephemeral mount `/var/vcap/data' is not empty. Contents: [/var/vcap/data/blobs /var/vcap/data/jobs /var/vcap/data/lost+found /var/vcap/data/nginx /var/vcap/data/packages /var/vcap/data/root_log /var/vcap/data/root_tmp /var/vcap/data/smoke-tests /var/vcap/data/sys /var/vcap/data/tmp]
	2018-11-06_18:14:14.99527 [linuxPlatform] 2018/11/06 18:14:14 DEBUG - Getting device size of `/dev/xvdb'
	2018-11-06_18:14:14.99643 [linuxPlatform] 2018/11/06 18:14:14 DEBUG - Calculating partition sizes of `/dev/xvdb', with available size 3221225472B
	2018-11-06_18:14:15.00354 [linuxPlatform] 2018/11/06 18:14:15 DEBUG - Found root partition: `/dev/xvda1'
	2018-11-06_18:14:15.00413 [linuxPlatform] 2018/11/06 18:14:15 DEBUG - Symlink is: `/dev/xvda1'
	```

- One of the first things the agent does is read about itself by reading various points of meta-data from AWS `http://169.254.169.254/latest/meta-data/...`
- The VM then sets its networking configuration:
	
	```
	2018-11-06_18:14:07.19260 [interfaceConfigurationCreator] 2018/11/06 18:14:07 DEBUG - Creating network configuration with settings: type: 'manual', ip: '10.4.1.17', netmask: '255.255.255.0', gateway: '10.4.1.1', mac: '02:12:47:61:64:aa', resolved: 'true', preconfigured: 'false', use_dhcp: 'true'
	```
- and finally, the agent sets/confirms the volume mount points:
	
	```
	2018-11-06_18:14:14.99525 [linuxPlatform] 2018/11/06 18:14:14 DEBUG - Existing ephemeral mount `/var/vcap/data' is not empty. Contents: [/var/vcap/data/blobs /var/vcap/data/jobs /var/vcap/data/lost+found /var/vcap/data/nginx /var/vcap/data/packages /var/vcap/data/root_log /var/vcap/data/root_tmp /var/vcap/data/smoke-tests /var/vcap/data/sys /var/vcap/data/tmp]
	2018-11-06_18:14:14.99527 [linuxPlatform] 2018/11/06 18:14:14 DEBUG - Getting device size of `/dev/xvdb'
	2018-11-06_18:14:14.99643 [linuxPlatform] 2018/11/06 18:14:14 DEBUG - Calculating partition sizes of `/dev/xvdb', with available size 3221225472B
	2018-11-06_18:14:15.00354 [linuxPlatform] 2018/11/06 18:14:15 DEBUG - Found root partition: `/dev/xvda1'
	2018-11-06_18:14:15.00413 [linuxPlatform] 2018/11/06 18:14:15 DEBUG - Symlink is: `/dev/xvda1'
	```
	
	

###### Hearbeats and Alerts
The BOSH Agent keeps tabs on the Monit process which in turn keeps tabs on the services that are intended to be running on the VM. Most of this is handled internal to the VM, but some traffic does go outbound. The BOSH Agent, for example, is responsible for sending the Director a "heartbeat" every 30 seconds. The presence of a "heartbeat" is designed to help the operator distinguish between a missing alerts due to normal behavior and missing alerts as the result of a system failure.

1. Use `grep` with `-A 4` to explore the "bosh-agent" logs surrounding the `"message 'heartbeat"` log entry.
	
	```bash
	2018-11-06_18:37:45.10753 [NATS Handler] 2018/11/06 18:37:45 INFO 	Sending hm message 'heartbeat'
	2018-11-06_18:37:45.10753 [NATS Handler] 2018/11/06 18:37:45 DEBUG - Message Payload
	2018-11-06_18:37:45.10754 ********************
	2018-11-06_18:37:45.10754 {"deployment":"nginx","job":"nginx","index":0,"job_state":"running","vitals":{"cpu":{"sys":"0.0","user":"0.0","wait":"0.1"},"disk":{"ephemeral":{"inode_percent":"0","percent":"1"},"system":{"inode_percent":"31","percent":"42"}},"load":["0.00","0.00","0.00"],"mem":{"kb":"188412","percent":"19"},"swap":{"kb":"0","percent":"0"},"uptime":{"secs":75615}},"node_id":"14e444f1-48c0-4006-8908-7d535ee88bc0"}
	2018-11-06_18:37:45.10754 ********************
	```
    
2. Use `killall` to force "nginx" to exit. Use `grep` with `-A 4` to look for the `"message 'alert'"` reported upstream:

	```bash
	2018-11-06_18:45:30.31658 [NATS Handler] 2018/11/06 18:45:30 INFO - Sending hm message 'alert'
	2018-11-06_18:45:30.31658 [NATS Handler] 2018/11/06 18:45:30 DEBUG - Message Payload
	2018-11-06_18:45:30.31659 ********************
	2018-11-06_18:45:30.31659 {"id":"1541529930.1213040550@localhost","severity":1,"title":"nginx (10.4.1.17) - Does not exist - restart","summary":"process is not running","created_at":1541529930}
	2018-11-06_18:45:30.31660 ********************
	```


---
**Great** if you haven't taken a break in a while, now is a good time to remind you to get up and walk around, see if the sun is still shining, or get some coffee before diving in to [Lab-5](../lab-5/README.md).