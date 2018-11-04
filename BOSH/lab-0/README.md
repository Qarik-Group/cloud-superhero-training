# Lab 0
## Objectives:
At the completion of this lab the student will be able to answer the following questions:

* What is BOSH?
* How do you connect to a BOSH Director and set a default environment?

## What is BOSH
The [Ultimate Guide to BOSH](https://ultimateguidetobosh.com/introduction/#what-is-bosh) explains this really well:

>BOSH is a project of the Cloud Foundry Foundation. It was originally created to help the developers of Cloud Foundry to absolutely describe and test each commit and each release; and to help the site reliability engineers (SREs) tasked with running Cloud Foundry as a service.
>
>Cloud Foundry has a micro-services architecture - bespoke applications written in Ruby, Java, C, and Golang - combined with stateful data services such as PostgreSQL, Redis, and local disks for storing user-uploaded application code. The developers wanted to work with the SREs to reduce the time of upgrades to new releases, to reduce the time between new releases, to reduce the time to deploy security fixes, and to help SREs and developers communicate about issues in production.
>
>TODO [twitter joke](https://twitter.com/oising/status/298464920717099009) about laptop going into production
>
>The solution was:
>
>* to have an absolute declaration of what specific versions of all bespoke and upstream projects combined together to form a "release";
>* to own responsibility for the lifecycle of the underlying infrastructure upon which Cloud Foundry would run, including healing activities after infrastructure failures;
>* to own responsibility for pushing out security patches to the base operating systems, the bespoke code, and the upstream dependencies;
>* to give developers and SREs the same tool to use thus removing "it works on my machine" scenarios
>
>The "tool" that implemented this solution is a running server - a BOSH environment - which:
>
>* receives requests from operators, who primarily use the bosh CLI;
interacts with cloud infrastructures to provision and de-provision cloud servers and disks;
>* interacts with running servers to configure and monitor long-running processes;
>* monitors the health of cloud servers and performs remedial actions to recreate or fix any missing infrastructure
>
>Today, small teams and large businesses are using BOSH to run a wide variety of systems including but not limited to platforms such as Cloud Foundry, Kubernetes, DC/OS, Docker, Habitat, and Nomad. It is used to run database clusters. It can run source control systems. It can run web applications.
>
>Some teams use it only for its provisioning/infrastructure lifecycles features, and use their own packaging, container, and configuration management tools.
>
>Some teams put their BOSH environment behind an API, such as the Open Service Broker API, and dynamically provision and de-provision entire systems on demand. For example, [Pivotal Container Services](https://pivotal.io/platform/pivotal-container-service) is an API driven system to deploy entire Kubernetes clusters, all using BOSH.

## About The Labs
To recap, a BOSH environment is designed to manage resources, run services, oversee lifecycle within a Cloud. The heavy lifting is done by a special machine called the BOSH Director which is managed by sending commands from the `bosh` cli.

For the purposes of this training we have created a shared classroom BOSH Director so the focus can be directed at creating, deploying, and managing an application which is where the majority of time will be spent. In later labs we'll do some exercises to help ensure that the Director doesn't remain an entirely black box.

## Connecting to the Director
### Connect to the Jumpbox
If necessary, start by connecting to the jumpbox:

    cd workspace
    vagrant resume
    vagrant ssh
    ssh classroom

### Setup the Alias
Our bosh director has a set of credentials which have been pre-installed in your home directory on the jumpbox for simplicity (`cat ~/creds.yml`). The URL for the bosh director has also been placed in this file. In order to connect and login we need the ca-cert used by the director, the director URL, and the user password (in this case admin).

Now, we'll set up the 'training' alias to refer to our shared bosh director and login for the first time.

    bosh alias-env training --environment \
      $(bosh int ~/creds.yml --path /bosh_url) \
      --ca-cert <(bosh int ~/creds.yml --path /director_ssl/ca)

Wow, so that has a lot going on. So let's break that down:

    bosh alias-env training -environment [director URL] \
      --ca-cert [certificate]

This sets a new bosh target with an alias of training pointing at a given [director URL] and allowing certificates signed by the CA given in [certificate].

Next, we have the `bosh int` commands:

	$(bosh int ~/creds.yml --path /director_ssl/ca)

The bosh cli has a built-in interpolator (int) for yaml files. What this is doing is running a bosh interpolate in a subshell "$(...)" to retrieve whatever content is found in the file "~/creds.yml" within the hierarchy at "/director_ssl/ca".

### Login to the Bosh Environment
The basic command to login to the bosh director is `bosh -e [alias or env] login` and it will prompt you for a username ("Email()") and password. This of course, is not super convenient, fortunately it will also use the environment variables `BOSH_CLIENT` and `BOSH_CLIENT_SECRET` in place of username and password respectively. Within the creds.yml both the user (`username`) and password (`admin_password`) are stored, so we can use `bosh int` to set these variables for us too.

Try your hand at creating three commands that set these two variables and then login to the director. When done, you should be able to run `bosh -e training vms` and see a list of the vms running on our shared director. Expand the section below for the solution.

<details><summary>SOLUTION:</summary>
     
    export BOSH_CLIENT=$(bosh int ~/creds.yml --path /username)
    export BOSH_CLIENT_SECRET=$(bosh int ~/creds.yml --path /admin_password)
    bosh -e training login

</details>

If you add these commands to your ~/.bash_profile on the jumpbox you'll automatically be logged in when you start your session.


### Bonus and Caution
The bosh cli will also recognize the environment variable "BOSH_ENVIRONMENT". If you export this as well you will be able to run bosh commands without including "-e training". This can be a mixed blessing; when you work with multiple environments it could result in you running a command in the wrong place, but when working consistently with a single environment it will save time. Use with caution.


## Architecture of a Release
Recall that BOSH is designed to create a consistent, reliable, repeatable application deployment experience across diverse architecture and help eliminate the "it works on my machine" outcome. In order to accomplish this, a release needs to be able to package all of the dependencies that the application requires to run on a system, define the external resources the release depends on, and define the resource requirements associated with a given deployment.

…

Packages, Jobs (Tasks), source blobs


## Running a Local Director
It is possible to run a BOSH director on your local system but the performance, consistency, and stability of this can depend on the type of system you use and its capabilities. Additionally, a local BOSH director using tools like bosh-lite and VirtualBox can become corrupted by system reboots, suspend, or hibernates.

Risks aside, running a local director for experimentation and greater understanding can be an excellent means of learning more. For details about running a BOSH director locally for experimentation purposes see the [BOSH Quick Start guide](https://bosh.io/docs/quick-start/).