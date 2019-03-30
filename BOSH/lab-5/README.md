## Lab 5
### Objectives:
In this lab we'll pick up with the VM internals and dive deeper into the BOSH Director. At the conclusion of this lab the student will be able to:

* How health monitoring is used and how to take remedial action if required.
* Describe the communication paths between the BOSH Director and Managed VMS.
* Connect to the BOSH director and be familiar with the internal processes and services it uses to manage the environment.

### Prerequisites:
* This lab assumes you have completed [Lab-4](../lab-4/README.md) covering how to list and access your VMS as well as where to find the components deployed by BOSH

### BOSH is So META
The BOSH director itself is packaged and its VM is launched using a [bosh release](https://bosh.io/releases/github.com/cloudfoundry/bosh). Under normal operations, there is almost no need to access the Director via SSH but the "how-to" is important for **Operators** and it provides further understanding of the BOSH Internals.

In our lab deployment, our BOSH Director can be accessed through our jumpbox (often also called a Bastion host). The following command can be used to access our jumpbox:

```bash
ssh -i ~/.ssh/paas_master_key vcap@10.4.1.4
```

***Security Warning!*** normally this key would not be placed on a jumpbox where it is accessible to all users, this has been for this lab specifically to allow the student an opportunity to open the "Black Box" of the Director and explore its internals.

### NATS
[NATS](https://www.nats.io/), or Neural Autonomic Transport System (now you know why we just call it NATS), is a open source, high-performance messaging platform designed to be highly scalable. These features make it an excellent choice for building and managing cloud native distributed systems.

The NATS message bus is used to manage communications between the various components that make up a BOSH deployment.

### Health Monitor

### Director

### Cloud Provider Interface

### Resurection

### Blobstore

### Tasks and Workers


---
**Outstanding** when you are ready you can move on to [Lab 6](../lab-6/README.md).