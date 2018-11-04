# Cloud Superhero Training

This training session is designed to introduce BOSH and familiarize the student with both its features as well as its place in the ecosystem.

## Course Description
This course is designed to provide the student with a firm understanding of working with BOSH in an Operations/Support capacity. The course is intended to be instructor led as part of a classroom setting, but where possible we've included tips for those that want to interact with the course independently, or review the materials further outside of the classroom.

### Purpose
At the conclusion of this course, the student will:

* Understand the purpose of BOSH and the basics of deploying a basic application with BOSH.
* Have a general understanding and familiarity with the most common BOSH commands.
* Be able to create and deploy a basic application using BOSH.
* Recognize and be familiar with the architecture of a BOSH deployment.
* Have the ability to operate and perform basic troubleshooting of a BOSH-based environment.

### Requirements
* [VirtualBox is installed](https://www.virtualbox.org/wiki/Downloads)
* [Vagrant is installed](https://www.vagrantup.com/docs/installation/) (note the [special instructions](https://www.vagrantup.com/docs/installation/#windows-virtualbox-and-hyper-v) for Windows-based systems)
* A copy of the Stark & Wayne curated Vagrant image (`vagrant init jhunt/vagabond`)
* An Internet connection is available
* Familiarity with the *NIX console and basic command line tools
* An individual public Github account (www.github.com)

### Working Locally
This training is designed to be used by the greatest possible audience while providing a reliable and consistent experience. As a result, this training relies on an external shared BOSH director which has been supplied, and which will be removed at the conclusion of the training session.

It is possible to run a BOSH director on your local system; however, results can vary dependent on the type of system you use and its capabilities. In addition to system performance and OS concerns, a local BOSH director using tools like bosh-lite and VirtualBox can become corrupted by system reboots, suspend, or hibernates.

We do recommend and encourage independently reviewing this training after hours, or after the conclusion of the formal training using your own director. For details about running a BOSH director locally for experimentation purposes see the [BOSH Quick Start guide](https://bosh.io/docs/quick-start/).
