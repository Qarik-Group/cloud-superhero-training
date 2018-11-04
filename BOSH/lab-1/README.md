## Lab 1 (Creating and Deploying a BOSH Release )

## Objectives:

At the completion of this lab the student should be able to answer the following questions:

-   How do you create a new BOSH release?
    
-   What is the directory structure of a BOSH release?
    
-   What are Jobs, Blobs, Packages, and Sources?
    
-   How do you deploy that bosh release to AWS?
    
-   What is a BOSH director, stemcell, and deployment manifest?
    

## Activity

The following lab is based on the material above and is designed to help you better understand the material presented. You will work independently for about 20 minutes and then discuss the solution as a group.

#### Command Reference

If you are not familiar with a specific command, you can often obtain additional details about it by using --help (-h) or, in the case of a *NIX command man [command]. In this lab you may find the following commands helpful:

-   cat - Concatenate and print the content of files.
    
-   cd - Changes to a specified directory
    
-   cp - Copy a file from one place to another
    
-   ls - List the current or specified directory. (try it with some good flags, like -la)
    
-   mkdir - Makes a new directory in the current folder
    
-   pwd - Print Working (current) Directory
    
-   sha/shasum - Print or Check SHA Checksums.
    
-   source - The source command can be used to load any functions file into the current shell script or a command prompt.
    

### Description and First Steps

In this activity we will create and deploy our very first BOSH release. We will do this using the BOSH CLI, so if you have not already installed it you can do so by following the instructions [HERE](https://bosh.io/docs/cli-v2-install/). After we have the BOSH CLI installed, we will need to make a new directory somewhere to use as a playground so that we can create a new release. (perhaps something like ‘~/workspace/bosh-course). Next, we will need something easy to use as an example of how to package up and create a release. In this case, we will use the popular web server [nginx](https://www.nginx.com/). We will need a directory to store the release. Using best practices, we should name it something self descriptive like ‘nginx-release’.

  
  
  

### Creating a BOSH Release:

-  Initializing the Release Framework
    

   - Now that we have a place to initialize our release, we can use BOSH’s ‘bosh init-release --git --dir <release_name>’ command to make it happen! Be sure to use BOSH’s help (-h) flag to learn what arguments the BOSH commands might be expecting and what flags you might be able to use. I suggest that we include the --git option to make our target directory a git repository. Also, it is a best practice to use dashes in the release name. Use underscores for all other filenames.
    
   - Congratulations! We have just successfully initialized our our BOSH release, let’s take a minute to look at the directory structure. This structure is the foundation for all BOSH releases and it is important that we understand the purpose of each file and folder. You can cd and cat out the folders and files to get more familiar with them (Note to future self, when we deploy our release, we will notice that this exact same folder structure also appears on the job VM in the var/vcap directory.)
    

     - The jobs folder holds the pieces of the service or application you are releasing. Jobs are generally made up of a set of packages.
    
     - The packages folder is where source code and dependencies for the jobs are compiled and stored. Packages give BOSH the information needed to prepare the binaries and dependencies for your jobs.
    
     - The src (source) and blobs folders hold the non-binary and binary files respectively for packages. Sources and Blobs are generally the downloaded tar files or other binaries that you will need for your release, such as a programming language or, in our case, a web server like nginx. BOSH does not really differentiate between `blob` and `src` resources in the context of a package. It treats these both as simple ‘files’ that must be available for packaging.
    

-   Adding Dependencies (blobs)
    

    - [Download the nginx source code](http://nginx.org/en/download.html) and import it into BOSH as a blob. Use BOSH’s ‘bosh add-blob PATH=<path_to_blob_on_local_system> BLOB_PATH=<package_name>’ command to attach the nginx tar file that we downloaded. Let’s use best practices and name the package_name ‘nginx.tar.gz.’
    
    - Since we acquired nginx from an untrusted source (the Internet), let’s use BOSH’s ‘blobs’ command and BASH’s shasum command to compare the SHA and verify the integrity of the file.
    
    - Next we have to create a configuration file in the src directory. We should call it nginx.conf, and is the main configuration file. Here is a [sample configuration file](https://gist.github.com/Bunter/9e01904fea530734e7eca354ea7760ba) we can inspect and use for this lab.
    

-   Packaging the Dependencies
    

    - At compile time, BOSH takes the source files referenced in the package specs, and renders them into the executable binaries and scripts that your deployed jobs need.
    
    - Packaging scripts are instructions to tell BOSH how to do this. The instructions may involve some combination of copying, compilation, and related procedures.
    
    - We can create a package skeleton using the ‘bosh generate-package <dependency_name>’ command using ‘nginx’ as the dependency_name.
    
    - We tell BOSH about this dependency in the ‘spec’ file in the ‘packages/nginx’ directory. Here is a [sample spec file](https://gist.github.com/Bunter/378ed9c9ef9b7362a7dab97f8e3956a1) we can view, and use for this lab. The files listed in this spec will now be available to our packaging script.
    
    - Now, we can set up the ‘packaging’ file in the same directory. Here is a [sample packaging file](https://gist.github.com/Bunter/e128bd316875a922b3c2bbde4abf0909) we can use for this lab. We must edit our skeleton file to include this bash script. Note that each compilation step is surrounded by a `pushd` and `popd` pair distinguishes each step. Also, note that any copying, installing or compiling deposits the resulting code to the install target directory (the BOSH_INSTALL_TARGET environment variable)
    

-   Jobs - describing the runtime configuration of your compiled binaries.
    

    - For each job, we create a job skeleton using the ‘bosh generate-job <job_name>’ command, of course using best practices, our job_name should be ‘nginx’
    
    - Jobs are made up of control scripts, monit files, and spec files. The generate-job command that we just used should have created these skeleton files in our ‘jobs/nginx’ directory
    

      - Every job needs a way to start and stop in a controlled manner. Control scripts are responsible for doing this through an embedded ruby file.
    
      - Let’s create one for our job named ‘ctl.erb’ in the `jobs/nginx/templates` directory. Here is a [sample control script](https://gist.github.com/Bunter/07445263e9cb666f30b49c387ceef23d) we can use.
    
      - Monit is a free, open source process supervision tool which provides BOSH with its first defence against unwanted downtime. The monit daemon executes the commands in our control script to monitor our job at runtime through a monit file
    
      - We need to edit our monit file, in the ‘jobs/nginx’ directory to let BOSH know how to use our control scripts. Here is a [sample monit file](https://gist.github.com/Bunter/8377784f4e24c500201bd5fe45515eb4) we can look at and use for this lab.
    
      - Like all web servers, nginx needs at minimum a root level index page (again in the format of an ERB template.) We can use the resulting `index.html` page to test nginx.
    
      - Create a file named `index.html.erb` in the `jobs/nginx/templates` directory. Here is a very simple [sample index page](https://gist.github.com/Bunter/4cb29a2e4531b00cf4cb595ef2563f1c) we can use.
    
      - The Job Spec file pulls everything together, referencing the control script, the index page and the compiled nginx package. The template names and file paths are among the metadata for each job that resides in the job spec file.
    
      - Edit the `spec` file in the `jobs/nginx` directory to include this [sample spec file](https://gist.github.com/Bunter/74e6c85af47afd6dec94905b54237376) we can use for this lab.
    

-   Shrink Wrap our Release
    

    - All of the elements needed to create our release should now be in place, so we can go ahead and use the ‘create-release’ command. We should use the --force and --tarball flags with the command in this case.
    
    - The --force flag forces BOSH to use our local copies of our blobs.
    
    - Without the --force option, BOSH requires blobs to be uploaded before you run bosh create-release.
    
    - The --tarball flag specifies the name of the release file. In this case we should pass it release.tgz
    
    - By using the --tarball flag, we can see that the BOSH CLI has created a .tgz file for us in the root of our release. This is useful for us so that we can inspect the release structure and contents of our completed release in it’s commonly shipped format.
    

  

Now that we have successfully used the BOSH CLI to codify the structure of all of the parts of our release, we need to take that code structure and turn it into infrastructure. BOSH uses a server-side component known as the BOSH director which handles that translation. For this lab, we have pre-provisioned a BOSH director for us to use so that we can stay focused on the mechanics of creating and deploying. In lab 0, we should have already configured the BOSH CLI by ‘sourcing’ the provided environment file into our current shell. Let’s check and make sure our connection works, and we are targeting the correct director by running the ‘bosh env’ command.

### Deploying our BOSH Release:

-   Upload nginx release
    

    - Make sure we are in the directory of the release we want to deploy, in this case the nginx-release directory
    
    - Upload the release using BOSH’s ‘upload-release’ command. Assuming we are in the release directory, no path is needed with the command.
    
    - We can verify that the BOSH director knows about our release using the ‘releases’ command.
    

- Create the manifest and upload the stemcell
    

    - The stem-cell represents the base operating system image used to run all of the software in the release. These images are published and maintained by the BOSH core team.
    
     - Each deployment documents the contents and arrangement for a release in a manifest yaml file. This file should be located in the root directory of the release. Here is a [sample manifest file](https://gist.github.com/Bunter/f393c614f2f93ae8e83cb18fa01cb4ca) that we can use for this lab. Take a quick look at the sample manifest to make sure we understand [what each section is used for](https://gist.github.com/Bunter/c42ca5e7669fe79d01296f3baaa59f28)..
    
    - Download the version of the stemcell listed in our sample manifest from [bosh.io](http://bosh.io/stemcells) and upload it to the BOSH director using the `upload-stemcell` command.
    

-    Deploy! Deploy! Deploy!
    

     - One way to conceptualize what is going on in a BOSH deployment is to view it as two states, the actual state and the desired state. Since we haven’t deployed anything yet, these two states are currently aligned, meaning nothing has been requested yet, and also nothing exists. When we tell the BOSH director to deploy, we are actually diverging the actual and desired states. The response of the BOSH director is to realign these states.
    
     - So let’s deploy our release! Use BOSH’s ‘deploy’ command to update our desired state using the ‘--deployment’ flag which will take the ‘name’ from our manifest as the argument, and the ‘--non-interactive’ flag to suppress the deployment diff, since this is our first deploy and it would be uninteresting.
    
     - As the deployment runs, pay attention to the following phases
    

       - Compilation - identifies and compiles the packages
    
       - VM Creation - Instantiates VMs to match the desired state described in the instance_groups
    
       - Update - Places the jobs inside of their respective VMs to match the desired state described in the instance_groups
    

     - When the deploy has finished, we can confirm that our nginx release has been successfully deployed by running the following command from our terminal
    

    bosh --deployment nginx ssh nginx --command "curl localhost"

-   Updating Deployments (test case: environment expansion)
    

    - Any time we make an update to the manifest, we will redeploy the release.
    
    - Let’s test out how this works by upgrading the number of instances we deploy our release to. In the manifest yaml, in the ‘instance_groups’ block, let’s change the number of ‘instances’ from 1 to 2.
    
    - Now, let’s deploy the new desired state using the ‘deploy’ command, but this time let’s use the --no-redact’ flag, and drop the ‘--non-interactive’ flag. By dropping the latter flag, a diff of the changes we made will be displayed, and we will be prompted to confirm our changes.
    
    - After the deploy has finished, we can confirm that our changes have been made, and that we do indeed have two instances running by running the following commands from our terminal
    

          bosh --deployment nginx ssh nginx/0 --command "curl localhost"

          bosh --deployment nginx ssh nginx/1 --command "curl localhost"

Congrats!!! We have now successfully created and deployed our first BOSH release!