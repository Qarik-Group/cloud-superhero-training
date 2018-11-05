## Lab 3
### Objectives:
At the completion of this lab the student will be able to answer the following questions:

* What is an errand and what are they used for?
* How are errands written?
* How do we find and run existing errands?

### About Errands
Formally, errands are any job that includes a `bin/run` file in its spec file's template section.  When triggered, the Operator will get the output from stdout, stderr, and the exit code. These are frequently used as jobs that "smoke" test/validate various aspects of the deployment.

### The "smoke" test
Let's add a simple smoke test by creating "src/smoke-test.sh" which will use `curl` to validate that a supplied URL is accesssible and respods with a "200" response code.

```bash
#!/bin/bash

if [ -z "$1" ]
then
    echo "usage: smoke-test.sh <url>"
    exit 1
fi

status=`curl --silent --head $1 | head -1 | cut -f 2 -d' '`

if [ "$status" != "200" ]
then
    echo "status was other than '200': was '$status'"
    exit 1
else
    exit 0
fi
```

### Generate a Package
Now we'll use the BOSH cli command `generate-package` to create a package named "smoke-tests" and wire it into our release.

1. Run the `generate-package` command
2. Modify the "spec" file to include "smoke-test.sh" as part of this package.
3. Write out a packaging script to setup the script so it can be executed:
  - creates a "bin" directory beneath `${BOSH_INSTALL_TARGET}`
  - changes the file permissions to make the "smoke-test.sh" script executable
  - writes the "smoke-test.sh" script to the previously mentioned "bin" directory


### Generate a Job
Next, we need to tell our deployment about this new job using the BOSH cli command `generate-job`. To stay consistent, we'll also name this job "smoke-tests".

1. Run the `generate-job` command to create the "smoke-tests" job.
2. Edit the "spec" file and establish the dependency on the "smoke-tests" package.
3. Include a template named "run.sh" that gets placed at the "bin/run" path
4. Create a "run.sh" under the "templates" directory that includes the following content:

  ```bash
  #!/bin/bash
  set -e

  /var/vcap/packages/smoke-tests/bin/smoke-test.sh localhost
  ```

### Create a New Release and Upload
Now we can create a new release (version) and upload it to our Director.

1. To create and upload the release you'll use the same set of commands used when creating the first release in [Lab-1](../lab-1/README.md) and uploading in [Lab-2](../lab-2/README.md).
2. The `bosh releases` command can be used to check the available versions of your project.
3. Finally, you need to tell the deployment manifest about your new "smoke-tests" job:

	```yaml
	...
	jobs:
	- name: nginx
	  release: nginx
	  properties: ...
	- name: smoke-tests   # <<< this part is new
	  release: nginx
	  properties: {}
	...
	```
4. Now you can redeploy using the deployment command used in [Lab-2](../lab-2/README.md).
  - BOSH will analyze the changes to the deployment and highlight any differences before confirming if you want to proceed.

### Running an Errand
Finally, lets check and run our errand.

1. List the available errands using the BOSH cli `errands` command. You'll need to specify both the environment (if you haven't set BOSH_ENVIRONMENT) as well as the deployment you want to refer to.
2. Run the errand with the `run-errand` command.

The errand should pass. In the event it fails review the steps above against the error you encountered and see if maybe a step got missed and then re-release.

###### Experiment
Connect to the host running nginx and assume the root user with the command `sudo su -`, then use the `ctl` script to stop the running server. Exit and re-run the errand; the errand still passes, let's move on to [Lab-4](../lab-4/README.md) to find out why.