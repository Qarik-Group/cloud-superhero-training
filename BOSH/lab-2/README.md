## Lab 2
### Objectives:
At the completion of this lab the student will be able to answer the following questions:

- How do you deploy a bosh release to AWS?
- What is a BOSH director, stemcell, and deployment manifest?
- How do you modify an existing deployment?

## Deploying our First BOSH Release:
Now that we have successfully used the BOSH CLI to codify the structure of all of the parts of our release, we need to take that code structure and turn it into infrastructure. BOSH uses a server-side component known as the BOSH director which handles that translation.

For this lab, we have pre-provisioned a BOSH director for us to use so that we can stay focused on the mechanics of creating and deploying. In [Lab 0](../lab-0/README.md), we should have already configured the BOSH CLI by "sourcing" the provided environment file into our current shell. Let’s check and make sure our connection works, and we are targeting the correct director by running the `bosh -e training env` command.

(note: since we have a preprovisioned director, and we all have a release with the same name, the director will be confused.  we can fix this by updating the name of our release in the final.yml file.  Let's change this from nginx to something like nginx-<user_name> to avoid this collision.  Now we can upload our releases without any collisions.)

#### Upload nginx release
1. Make sure we are in the directory of the release we want to deploy, in this case the "nginx-release" directory.
2. If you have not already done so in Lab 0, we need to authenticate ourselves with the Director. To do this, first check our creds.yml to find pull the username (admin) and password. Then we can use the `bosh login` command using the username for the email prompt, and the password for the password.  After doing this we can log off. 
3. Upload the release using BOSH’s `upload-release` command. Assuming we are in the release directory, no path is needed with the command.
4. We can verify that the BOSH director knows about our release using the ‘releases’ command.
    

#### Create the manifest and upload the stemcell
The stemcell represents the base operating system image used to run all of the software in the release. These images are published and maintained by the BOSH core team.
    
Each deployment documents the contents and arrangement for a release in a manifest yaml file. This file should be located in the root directory of the release. Here is a [sample file](https://gist.github.com/Bunter/f393c614f2f93ae8e83cb18fa01cb4ca) for this lab:

```yaml
name: nginx

releases:
- name: nginx
  version: latest

stemcells:
- alias: default
  os: ubuntu-trusty
  version: 3586.16

instance_groups:
- name: nginx
  instances: 1
  azs: [z1]
  jobs:
  - name: nginx
    release: nginx
  vm_type: small
  stemcell: default
  networks:
  - name: default

update:
  canaries: 1
  max_in_flight: 10
  canary_watch_time: 1000-30000
  update_watch_time: 1000-30000
```

Take a quick look at the sample manifest to make sure we understand what each section is used for:

- `name`: identifies the deployment manifest itself
- `releases`: is a catalogue of the software in the deployment
- `stemcells`: is a catalogue of the BOSH-compatible OSs in the deployment
- `instance_groups`: describe the union of release jobs (i.e. processes) and stemcells combined with network and scaling requirements
- `update`: is a mandatory set of instructions which control how new versions or configuration changes will be rolled out by the BOSH director
    
Download the version of the stemcell listed in our sample manifest from [bosh.io](http://bosh.io/stemcells) and upload it to the BOSH director using the `upload-stemcell` command.

#### Deploy! Deploy! Deploy!
One way to conceptualize what is going on in a BOSH deployment is to view it as two states, the actual state and the desired state. Since we haven’t deployed anything yet, these two states are currently aligned, meaning nothing has been requested yet, and also nothing exists. When we tell the BOSH director to deploy, we are actually diverging the actual and desired states. The response of the BOSH director is to realign these states.
    
So let’s deploy our release!

1. Use BOSH’s `deploy` command to update our desired state using the `--deployment` flag which will take the ‘name’ from our manifest as the argument, and the `--non-interactive` flag to suppress the deployment diff, since this is our first deploy and it would be uninteresting.
    
  - As the deployment runs, pay attention to the following phases
       - Compilation - identifies and compiles the packages
       - VM Creation - Instantiates VMs to match the desired state described in the instance_groups
       - Update - Places the jobs inside of their respective VMs to match the desired state described in the instance_groups

2. When the deploy has finished, we can confirm that our nginx release has been successfully deployed by running the following command from our terminal

        bosh -e [environment] --deployment nginx ssh nginx --command "curl localhost"

## Modifying our BOSH Release
Any time we make an update to the manifest, we will redeploy the release. Let’s test out how this works by upgrading the number of instances we deploy our release to.

1. In the manifest yaml, in the "instance_groups" block, let’s change the number of "instances" from 1 to 2.
2. Now, let’s deploy the new desired state using the `deploy` command, but this time let’s use the `--no-redact` flag, and skip the `--non-interactive` flag. By dropping the latter flag, a diff of the changes we made will be displayed, and we will be prompted to confirm our changes.
3. After the deploy has finished, we can confirm that our changes have been made, and that we do indeed have two instances running by running the following commands from our terminal

          bosh --deployment nginx ssh nginx/0 --command "curl localhost"
          bosh --deployment nginx ssh nginx/1 --command "curl localhost"

**Congrats!!!** We have now successfully created and deployed our first BOSH release!


