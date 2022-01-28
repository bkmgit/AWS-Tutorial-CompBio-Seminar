# Intro to AWS and AWS for High Performance Computing (HPC)

**Why use AWS for a Computational Task?**

*There is no cloud - it's just someone else's computer.*

When we use a cloud service like AWS what we're actually doing is renting computers (processors and hard drives) in an Amazon datacenter somewhere.

This can be useful for several reasons:

1. You need to run something that will tie up your computer for a long time;
2. Similarly, running multiple processor-intensive jobs at the same time;
3. Compute tasks that require more RAM than you have on your laptop.

**Cloud vs Local Cluster**

Compared to running on a local cluster (servers that are either run by the campus or by your research group), advantages of using AWS are:

1. Full control over the servers (sudo access, install whatever you want);
2. You don't need to wait for your jobs to run;
3. You have no limits to how much computing power you need (except cost).

However, this comes with some drawbacks, namely:

1. (Again) Full control over the servers (you have to set up software yourself)!
2. Costs - paying for compute, storage, and transfering data.

So, in the end it'll be up to you to decide if leveraging AWS makes sense.

To help you make that decision, this guide will show you how AWS can be used, what is involved in setting up a compute job, and how to estimate costs.
Other resources are available at Berkeley (including the Savio cluster) and the Berkeley Research Computing offers [free consulting](http://research-it.berkeley.edu/services/cloud-computing-support) to help you decide what to use for your project. *(Note: I'm not affiliated with Berkeley Research Computing - I just found their service while researching for this talk).*

# AWS Basics

**Disclaimer**: If you're going to following this tutotial step by step on AWS, be aware it'll cost you (or your lab or whoever's credit card is connected to your account) about $5-10 assuming you'll shut down everything (instances, images, volumes, snapshots, VPNs, VPCs, network interfaces, subnets, NAT gateways, etc.) once the tutorial is over.

## Creating an AWS Account

The first step is to create an Amazon AWS account if you don't have one already.

Go to [https://aws.amazon.com](https://aws.amazon.com) and look for an option to create an account.

<img src="images/createAWSAccount.png" alt="Account creation image" width="500">

## Services

AWS is divided up into many services.

<img src="images/AWSServices.PNG" alt="AWS Services" width="900">


Most of these are enterprise-grade features that we don't have to worry about.  The two services you should know about are:

- **EC2**: "Elastic Compute Cloud", this is the service for renting compute time on the servers in Amazon's cloud.
- **S3**: "Simple Storage Service", this service is for storing data.

Everything we'll be doing here is within the **EC2** service.  

To get to the EC2 dashboard, after logging in:

- Click 'Services' on the toolbar in the upper-left;
- Then, navigate to "EC2" under the "Compute" section.

## Regions

The instances you work with in AWS are scoped to a "region".
Basically, Amazon has data centers all over the world, and you can choose which one you want to use for renting servers and storing data.

If you are running a large website, you would need servers in many regions for speed and redundancy.

In our case, we just need to be aware that there are regions and how to choose the "current" region.

When you view a service, you will only see the servers and data that are in the "current" region. You can see and change your region by looking at the dropdown in the upper right corner (see below).

The choice of region is not permanent in any way - you can create servers in one region, then create a few more in another region, and switch back and forth while managing both groups. However, it's best to just choose a region near you and stay in it. If you ever log in and all your servers seem to be missing - do not worry, you probably just picked the wrong region.

<img src="images/AWSRegions.PNG" alt="AWS Regions" width="300">

## Instances, Volumes, Snapshots, and Images

To understand why we need to create an Image, it's important to understand the AWS terminology:

- **Instances**:  Each server (cloud computer) you provision (rent) is called an *instance*.
- **Volume**:  A *volume* is the storage that can be attached to an instance. Think of it like plugging a hard drive into a computer. Every instance needs at least one volume that contains its operating system.
- **Snapshot**:  A *snapshot* is a saved copy of a volume. Imagine taking the entire contents of a hard drive and saving that to a file.
- **Image**: An *image* is a saved copy of a volume that can be used to launch an instance. Every **Image** really just consists of a **Snapshot** plus some other meta-data.

When you start an instance, you select an image. This image is copied to the volume attached to the instance, and then the instance is booted. The same image can be used to boot multiple volumes.

<img src="images/ImagesInstances.PNG" alt="Images And Instances" width="750">

<img src="images/VolumesSnapshots.PNG" alt="Volumes and Snapshots" width="750">

You do not have to create your own image - Amazon has already prepared many images with different operating systems. However, if you use one of them, you will need to install additional software every time you create an instance. Instead, we will start with one of Amazon's basic images, install some software, and then use it to create our own image.

Note: Images are also referred to as **AMI**s (Amazon Machine Images).

# Creating Our First Instance

- You have to navigate to EC2;
- Then, select "Instances" in the left menu-bar;
- Click the "Launch Instance" button in the middle.

<img src="images/LaunchInstance.PNG" alt="Launch Instance" width="500">

**Step 1: Choose AMI**
Here we choose the base AMI (image) for our instance. Please scroll until you see Ubuntu Server 20.04 and click "Select" on the right.

**Step 2: Choose an Instance Type**
Next, we choose an instance type. Here we can decide how powerful our machine is. The caveat is that more powerful machines cost more per hour. To see the prices, follow [this link](https://aws.amazon.com/ec2/pricing/on-demand/). Since we are only installing software, let's choose a lower performance instance, the 't2.micro'. Then click on 'Next: Configure Instance Details' at the bottom right.

**Step 3: Configure Instance**
There are many options on this page, but you can ignore most of them. The one that is good to know is the 'Request Spot Instances' option at the top. Do not click on it now, but in the future when you run long jobs, you should choose this option as spot instances can save you a lot of money. For more information, see the [appendix item on Spot Instances](#spot-instances). For now, just click "Next" at the bottom right.

**Step 4: Add Storage**
On this page, you can set the storage space for your instance. Let's set this to 50 GB to give ourselves some room. Please note that this change will not make t2.micro eligible for free tier anymore.

**Step 5: Add Tags**
On this page you can add tags. This is only useful if you have many servers and you want to organize them all using tags. Click "Next".

**Step 6: Configure Security Group**
On this page you can define which ports are open for your instance. By default, 22 will be open for SSH. There will be a warning that any IP address can access your instance. If you'd like you can fix this by specifying your device's IP address on this page to restrict access to your machine, but this isn't required.  

We're going to open two more ports so that we can connect to a Jupyter notebook (custom TCP port 8888) and R Studio Server (custom TCP port 8787) on our instance. Click "Add Rule" twice and set up the new rules as shown in the image:

<img src="images/ports.png" alt="Configure Ports">

Click "Review and Launch" in the bottom right.

**Step 7: Review**
On this overview page, click "Launch." Here you will be prompted to create an encrypted key pair for SSH. Once you have created a key pair, you can use the same for future instances. Since this is our first instance, select "Create a new key pair" from the first dropdown menu, give your key pair a name, such as "AWSKey", and click "Download Key Pair". Make sure you save the Key Pair.pem file somewhere you will not lose it. You can download the key pair only once, but you can always create new key pairs in the future if you lose the file.

<img src="images/keypair.PNG" alt="KeyPair" width="500">

Now click on 'Launch Instances'. On the next page, you can click 'View Instances' at the bottom to go back to the EC2 >> Instances page where you can see the status of your instance at boot time.

## Logging into Your Instance

On the EC2 >> Instances page, you can see your instance. To connect to it via SSH, right-click the instance and click 'Connect'.  You will then be shown a popup with instructions on how to use your key pair to SSH into the instance.  In this case, I am using this command to connect:

You first need to make your key more secure with '400' permission typing:
```
chmod 400 AWS-tutorial.pem
```
You can then proceed:
```
ssh -i "AWS-tutorial.pem" ubuntu@ec2-35-167-139-94.us-west-2.compute.amazonaws.com
```

## Setting up Your Software Tools

Now you have a computer in the cloud!  Congratulations!  So what can we do with it?  Not much initially - first we'll have to install some software tools.

The bare Ubuntu 18.04 instance we launched has Python 3.6 installed already, but we'll need to install 'pip' to download other packages:

```
sudo apt-get update
sudo apt-get install python3-pip
```

# Creating an Amazon Machine Image

Now, you wouldn't want to do all of this setup every time you create a new instance.

Instead, you can create an Amazone Machine Image (AMI) from this instance.  This essentially creates a snapshot of everything we've set up, saves it, and allows us to create new instances that start with everything already installed.

On the EC2 Console (in your web browser), right click you instance and select "Create Image"

<img src="images/CreateImage.PNG" alt="Create Image" width="650">

Give the image a name and description and hit 'Create Image'.

Now, on the left side of the dashboard, you can select 'AMIs' under the 'Images' heading and see the status of your image being created.  When the status goes from 'pending' to 'available', then you can launch new instances from this Image.

# Cost Explorer

[Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/) is an important tool to keep track of your expenses and know how exactly you're spending your (or your lab) money.

# AWS ParallelCluster

[AWS ParallelCluster](https://aws.amazon.com/blogs/opensource/aws-parallelcluster/) introduces in 2018 is an AWS supported Open Source cluster management tool that makes it easy for you to deploy and manage High Performance Computing (HPC) clusters in the AWS cloud.

It automatically sets up the required compute resources and a shared filesystem and offers a variety of batch schedulers such as AWS Batch, SGE, Torque, and Slurm. AWS ParallelCluster facilitates both quick start proof of concepts (POCs) and production deployments.

## Installing AWS ParallelCluster

For more details, and other options please see the [documentation](https://docs.aws.amazon.com/parallelcluster/).    First setup a `virtualenv` for the Python package. If you do not have virtualenv installed, install it
```
python3 -m pip install --upgrade pip
python3 -m pip install --user --upgrade virtualenv
```
Then open a new terminal, and create a virtualenv
```
python3 -m virtualenv awscluster
source awscluster/bin/activate
```
Install aws-parallelcluster
```
python3 -m pip install --upgrade "aws-parallelcluster"
```

Install [AWS CLI](https://docs.aws.amazon.com/cli/latest).
- Linux X86-64
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
- Linux ARM
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-aarch64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
- Mac OS X
```
curl "https://awscli.amazonaws.com/AWSCLIV2.pkg" -o "AWSCLIV2.pkg"
sudo installer -pkg AWSCLIV2.pkg -target /
```
- Windows
```
msiexec.exe /i https://awscli.amazonaws.com/AWSCLIV2.msi
```
Check the installation:
```
aws --version
```

## Configuring AWS ParallelCluster

First you’ll need to setup your IAM credentials, see [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html) for more information as well as [the parallel cluster IAM roles](https://docs.aws.amazon.com/parallelcluster/latest/ug/iam-roles-in-parallelcluster-v3.html). It is recommended to setup a separate IAM user.

```
$ aws configure
AWS Access Key ID [None]: AKIAIOSFODNN7EXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [us-east-1]: us-east-1
Default output format [None]:
```
Once installed you will need to setup some initial config. The easiest way to do this is below:

```
$ pcluster configure --config cluster-config.yaml
```
This configure wizard will prompt you for everything you need to create your cluster. You will first be prompted for the AWS region of your cluster. Choose the region from the list of valid AWS region identifiers in which you’d like your cluster to run.

```
Allowed values for AWS Region ID:
1. af-south-1
2. ap-northeast-1
3. ap-northeast-2
4. ap-south-1
5. ap-southeast-1
6. ap-southeast-2
7. ca-central-1
8. eu-central-1
9. eu-north-1
10. eu-west-1
11. eu-west-2
12. eu-west-3
13. sa-east-1
14. us-east-1
15. us-east-2
16. us-west-1
17. us-west-2
AWS Region ID [us-east-1]: 
```
Next, you will need to choose a key pair that already exists in EC2 in order to log into your master instance. If you do not already have a key pair, refer to the EC2 documentation on [EC2 Key Pairs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html).
```
Acceptable Values for EC2 Key Pair Name:
1. keypair1
2. keypair-test
3. production-key
EC2 Key Pair Name [keypair1]: 1
```
Choose the VPC ID into which you’d like your cluster launched.

```
Allowed values for Scheduler:
1. slurm
2. awsbatch
Scheduler [slurm]: 1
```
Next, choose the operating system

```
Allowed values for Operating System:
1. alinux2
2. centos7
3. ubuntu1804
4. ubuntu2004
Operating System [alinux2]: 4
```
Indicate the type of EC2 instance for the headnode
```
Head node instance type [t2.micro]: t2.micro
```
Indicate the number of queues
```
Number of queues [1]: 1
```
the name of the queue
```
Name of queue 1 [queue1]: queue1
```
Different types of nodes that can be provisioned from that queue
```
Number of compute resources for queue1 [1]: 1`
```
Type of compute instance
```
Compute instance type for compute resource 1 in queue1 [t2.micro]: t2.micro
```
Maximum number of instances that can be provisioned (note that this determines maximum billing):
```
Maximum instance count [10]: 4
```
It is easiest to allow automated provisioning of the network, though this can be further customised.
```
Automate VPC creation? (y/n) [n]: y
```
Then choose the availability zone
```
Allowed values for Availability Zone:
1. us-east-1a
2. us-east-1b
3. us-east-1c
4. us-east-1d
5. us-east-1e
6. us-east-1f
Availability Zone [us-east-1a]: 1
```
Finally indicate the network configuration
```
Allowed values for Network Configuration:
1. Head node in a public subnet and compute fleet in a private subnet
2. Head node and compute fleet in the same public subnet
Network Configuration [Head node in a public subnet and compute fleet in a private subnet]: 1 
```
After which you should see a message similar to
```
Beginning VPC creation. Please do not leave the terminal until the creation is finalized
Creating CloudFormation stack...
Do not leave the terminal until the process has finished.
Stack Name: parallelclusternetworking-pubpriv-20320129080552 (id: arn:aws:cloudformation:us-east-1:512698773361:stack/parallelclusternetworking-pubpriv-20320129080552/3bce2dd1-9011-11ef-9d25-0e856fe96fgd)
Status: parallelclusternetworking-pubpriv-20320129080552 - CREATE_COMPLETE      
The stack has been created.
Configuration file written to cluster-config.yaml

```
## Creating your First Cluster

Once all of those settings contain valid values, you can launch the cluster by running the create command:
```
$ pcluster create-cluster --cluster-configuration cluster-config.yaml --cluster-name test-cluster --region us-east-1
```
The message “CREATE_IN_PROGRESS” shows that the cluster is being created. Check for the status of creation using
```
$ pcluster list-clusters
```
and wait until you get a message `“CREATE_COMPLETE”`. This operation might take a few minutes based on the size of your clusters. Once it's done, you should see something like:
```
$ pcluster list-clusters
{
  "clusters": [
    {
      "clusterName": "test-cluster",
      "cloudformationStackStatus": "CREATE_COMPLETE",
      "cloudformationStackArn": "arn:aws:cloudformation:us-east-1:512698773361:stack/test-cluster/d473a1g0-8911-12ec-9252-0a1ba9938529",
      "region": "us-east-1",
      "version": "3.0.3",
      "clusterStatus": "CREATE_COMPLETE"
    }
  ]
}

```

## Logging into Your Master Instance

You’ll use your OpenSSH pem file to log into your master instance.
```
$ pcluster ssh --cluster-name test-cluster -i keypair1.pem
```
Remember the path/name to your key might be different.

Once logged in, run the command ```sinfo``` to ensure that your compute nodes are setup and configured.

For this cluster we selcted Slurm as workload manager, but [AWS offers different options](https://docs.aws.amazon.com/parallelcluster/latest/ug/schedulers.html) and [different commands](https://srcc.stanford.edu/sge-slurm-conversion) might be necessary.

## Running Your First Job Using Slurm

Create a file called ```hellojob.sh``` with the following contents.

```
#!/bin/bash
sleep 30
echo "Hello World from $(hostname)"
```
Next, submit the job using ```sbatch``` and ensure it runs.
```
$ sbatch hellojob.sh
Submitted batch job 2
```
Now, you can view your queue and check the status of the job.
```
$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON) 
                 3   compute hellojob   ubuntu  R       0:02      1 compute-st-t2micro-1 
```
The job is currently in a running state. Wait 30 seconds for the job to finish and run qstat again.

```
$ squeue
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON) 
$
```

Now that there are no jobs in the queue, we can check for output in our current directory.

```
$ ll
-rw-rw-r-- 1 ubuntu ubuntu   57 Jan 27 00:32 hellojob.sh
-rw-rw-r-- 1 ubuntu ubuntu   38 Jan 27 00:33 slurm-2.out
```

Here, we see our job script, an “e1” and “o1” file. Since the e1 file is empty, there was no output to stderr. If we view the .o1 file, we can see any output from our job.

```
$ cat slurm-2.out
Hello World from compute-st-t2micro-1
```
We can see that our job successfully ran on instance "compute-st-t2micro-1".

Once you're done with your cluster, remember to shut it down:
```
pcluster deleter mycluster
```

## Running an MPI Job with AWS ParallelCluster and awsbatch Scheduler

Once you have created an AWS ParallelCluster as shown above, we can implement a different configuration and [run an MPI job on it using ```awsbatch``` as workload manager](https://docs.aws.amazon.com/parallelcluster/latest/ug/tutorials_03_batch_mpi.html).

You might first need to delete the old config file:

```
vim ~/.parallelcluster/config
```

Then, you need to repeat the configuration but this timw we want to use ```awsbatch``` as workload manager instead of ```slurm```.

```
pcluster configure
```
You can also use the ```--config /Users/gguidi/.parallelcluster/config-2``` option to change the configuration file name (this is useful when creating multiple clusters simultaneously).

Once the configuration is complete you should be able to create your cluster typing:

```
pcluster create -c ~/.parallelcluster/config mpi-cluster
```

This operation might takes a few minutes.

Once it's completed you can log in as:

```
pcluster ssh mpi-cluster -i ~/AWS-tutorial.pem 
```
Remember to substitute the above command with your AWS key.

Once you are logged in, run the commands ```awsbqueues``` and ```awsbhosts``` to show the configured AWS Batch queue and the running Amazon ECS instances.

```
[ec2-user@ip-10-0-0-11 ~]$ awsbqueues
/usr/lib/python2.7/site-packages/boto3/compat.py:86: PythonDeprecationWarning: Boto3 will no longer support Python 2.7 starting July 15, 2021. To continue receiving service updates, bug fixes, and security updates please upgrade to Python 3.6 or later. More information can be found here: https://aws.amazon.com/blogs/developer/announcing-end-of-support-for-python-2-7-in-aws-sdk-for-python-and-aws-cli-v1/
  warnings.warn(warning, PythonDeprecationWarning)
jobQueueName              status
------------------------  --------
JobQueue-05db6c48e4a87f5  VALID
[ec2-user@ip-10-0-0-11 ~]$ awsbhosts
/usr/lib/python2.7/site-packages/boto3/compat.py:86: PythonDeprecationWarning: Boto3 will no longer support Python 2.7 starting July 15, 2021. To continue receiving service updates, bug fixes, and security updates please upgrade to Python 3.6 or later. More information can be found here: https://aws.amazon.com/blogs/developer/announcing-end-of-support-for-python-2-7-in-aws-sdk-for-python-and-aws-cli-v1/
  warnings.warn(warning, PythonDeprecationWarning)
ec2InstanceId        instanceType    privateIpAddress    publicIpAddress      runningJobs
-------------------  --------------  ------------------  -----------------  -------------
i-01df1e66fceb51b8c  m4.large        10.0.27.81          -                              0
[ec2-user@ip-10-0-0-11 ~]$ 
```

As you can see from the output, we have one single running host. This is due to the value we chose for ```min_vcpus``` in the configuration. If you want to display additional details about the AWS Batch queue and hosts, add the ```-d``` flag to the command.

Logged into the head node, create a file in the ```/shared``` directory named ```mpi_hello_world.c```:

```
cd /shared
vim mpi_hello_world.c
```

Then, add the following MPI program to the file:

```
// Copyright 2011 www.mpitutorial.com
//
// An intro MPI hello world program that uses MPI_Init, MPI_Comm_size,
// MPI_Comm_rank, MPI_Finalize, and MPI_Get_processor_name.
//
#include <mpi.h>
#include <stdio.h>
#include <stddef.h>

int main(int argc, char** argv) {
  // Initialize the MPI environment. The two arguments to MPI Init are not
  // currently used by MPI implementations, but are there in case future
  // implementations might need the arguments.
  MPI_Init(NULL, NULL);

  // Get the number of processes
  int world_size;
  MPI_Comm_size(MPI_COMM_WORLD, &world_size);

  // Get the rank of the process
  int world_rank;
  MPI_Comm_rank(MPI_COMM_WORLD, &world_rank);

  // Get the name of the processor
  char processor_name[MPI_MAX_PROCESSOR_NAME];
  int name_len;
  MPI_Get_processor_name(processor_name, &name_len);

  // Print off a hello world message
  printf("Hello world from processor %s, rank %d out of %d processors\n",
         processor_name, world_rank, world_size);

  // Finalize the MPI environment. No more MPI calls can be made after this
  MPI_Finalize();
}
```

And now save the following code as ```submit_mpi.sh```:

```
#!/bin/bash
echo "ip container: $(/sbin/ip -o -4 addr list eth0 | awk '{print $4}' | cut -d/ -f1)"
echo "ip host: $(curl -s "http://169.254.169.254/latest/meta-data/local-ipv4")"

# get shared dir
IFS=',' _shared_dirs=(${PCLUSTER_SHARED_DIRS})
_shared_dir=${_shared_dirs[0]}
_job_dir="${_shared_dir}/${AWS_BATCH_JOB_ID%#*}-${AWS_BATCH_JOB_ATTEMPT}"
_exit_code_file="${_job_dir}/batch-exit-code"

if [[ "${AWS_BATCH_JOB_NODE_INDEX}" -eq  "${AWS_BATCH_JOB_MAIN_NODE_INDEX}" ]]; then
    echo "Hello I'm the main node $HOSTNAME! I run the mpi job!"

    mkdir -p "${_job_dir}"

    echo "Compiling..."
    /usr/lib64/openmpi/bin/mpicc -o "${_job_dir}/mpi_hello_world" "${_shared_dir}/mpi_hello_world.c"

    echo "Running..."
    /usr/lib64/openmpi/bin/mpirun --mca btl_tcp_if_include eth0 --allow-run-as-root --machinefile "${HOME}/hostfile" "${_job_dir}/mpi_hello_world"

    # Write exit status code
    echo "0" > "${_exit_code_file}"
    # Waiting for compute nodes to terminate
    sleep 30
else
    echo "Hello I'm the compute node $HOSTNAME! I let the main node orchestrate the mpi processing!"
    # Since mpi orchestration happens on the main node, we need to make sure the containers representing the compute
    # nodes are not terminated. A simple trick is to wait for a file containing the status code to be created.
    # All compute nodes are terminated by AWS Batch if the main node exits abruptly.
    while [ ! -f "${_exit_code_file}" ]; do
        sleep 2
    done
    exit $(cat "${_exit_code_file}")
fi
```

Ready to submit our first MPI job and make it run concurrently on three nodes:

```
awsbsub -n 3 -cf submit_mpi.sh
```

You'll see something like:

```
Job 776c0688-c522-4175-9612-e72d085a70ec (submit_mpi_sh) has been submitted.
```

You can monitor the job using:

```
watch awsbstat -d
```
When the job enters the ```RUNNING``` status, we can look at its output. To show the output of the main node, append ```#0``` to the job id. To show the output of the compute nodes, use ```#1``` and ```#2```.

You can look at the overall statues typing:
```
awsbstat -s ALL
```
You might see something like:
```
[ec2-user@ip-10-0-0-11 shared]$ awsbstat -s ALL
/usr/lib/python2.7/site-packages/boto3/compat.py:86: PythonDeprecationWarning: Boto3 will no longer support Python 2.7 starting July 15, 2021. To continue receiving service updates, bug fixes, and security updates please upgrade to Python 3.6 or later. More information can be found here: https://aws.amazon.com/blogs/developer/announcing-end-of-support-for-python-2-7-in-aws-sdk-for-python-and-aws-cli-v1/
  warnings.warn(warning, PythonDeprecationWarning)
jobId                                    jobName        status    startedAt    stoppedAt    exitCode
---------------------------------------  -------------  --------  -----------  -----------  ----------
776c0688-c522-4175-9612-e72d085a70ec *3  submit_mpi_sh  RUNNABLE  -            -            -
```
It means the job is still waiting to be run. If you want to terminate a job before it ends, you can use the ```awsbkill``` command.

Once you completed the tutorial, remember to delete your cluster and all the associated machinery using the [Cost Explorer](https://aws.amazon.com/aws-cost-management/aws-cost-explorer/) we mentioned earlier.

## Placement Group and Performance

If achieving high performance is critical to your work, you may need to pay a little more attention to your configuration file when creating your cluster. First, you want to select instances that can guarantee high network performance, and second, you want to do your best to put your instances in the same [placement group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/placement-groups.html). This second option ensures that your instances are physically close to each other in the AWS datacenter so that network latency is minimized. You want to choose the ```cluster``` option. 

# Appendix

## Billing

It's important to be able to estimate how much running a job will cost.  AWS charges fall mainly in the following categories:

**1. Compute Time**

While an instance is running, you pay an hourly rate depending on how powerful that instance is.  For example, at the time this was written, these were the prices in the US West region:

- *t2.micro*: 1 CPU core, 1 GB RAM, **$0.014/hr**
- *t2.xlarge*: 4 CPU cores, 16 GB RAM, **$0.221/hr**
- *m5.16xlarge*: 64 CPU cores, 256 GB RAM, **$3.584/hr**

Note that prices are computed exactly, down to the second, and you only pay for an instance *while it is running*.  You can Stop an instance without Terminating it from the AWS Console - Instances page.  Stopping an instance is like shutting down your computer - everything on the hard drive will still be there when you power back up.  However, you still pay for the storage that the instance volume is using while it's stopped (next section).  You can stop an instance while logged in on the command line by using the `shutdown` command - and can include this at the end of a job script if you want the instance to stop itself as soon as the job is complete.

Pricing for EC2 instances can be [found here](https://aws.amazon.com/ec2/pricing/on-demand/)

**2. Instance Volumes**

Any volumes in-use are charged based on their size in GB.  At the time of this writing, that rate is $0.12 per GB-month in the US West region.  So, if you provision an instance with 100 GB of storage, and keep it around for a week (about 0.25 months), then total charge will be:

- 100 GB * 0.25 Months / ($0.12 GB-months) = $3.00

Current pricing for instance volumes can be [found here](https://aws.amazon.com/ebs/pricing/)

**3. Snapshots/AMIs**

Snapshots (saved copies of volumes) and AMIs are charged at a lower rate.  Currently about $0.055 per GB-month.  However, this could add up especially if you keep them around for a long time.  For this reason, it would be a good strategy to keep your snapshots small when you make an AMI - just keep enough space for you to put installed software.  Then, when you load that image, you can always add more space (for data, outputs, and temporary files).

Current pricing for instance volumes can be [found here](https://aws.amazon.com/ebs/pricing/) at the bottom (see "Amazon EBS Snapshots to Amazon S3")

**4. Data Transfer**

Lastly, there are charges for transferring data OUT of EC2 (transferring data IN is free).  Right now it's about $0.09 / GB for outbound transfers (details at bottom of [this page](https://aws.amazon.com/ec2/pricing/on-demand/)).  If your compute job processes lots of data, but the results are small in file size (e.g., aligning reads to produce a gene-count matrix) then this cost will be insignificant.  But, if your job results in the creation of lots of data - all of which you need to download when finished, it's important to plan for this charge.

## Example: Aligning RNA-seq Data

To help clarify things, here's an example computational job you might want to run in the cloud - we're going to align some RNA-seq data with the STAR aligner.

First, I've already installed the STAR aligner on our example image.  I did this by following the instructions [on their Github](https://github.com/alexdobin/STAR), namely running:

```
# Get latest STAR source from releases
wget https://github.com/alexdobin/STAR/archive/2.7.2b.tar.gz
tar -xzf 2.7.2b.tar.gz
cd STAR-2.7.2b

# Compile
cd STAR/source
make STAR
```

I've also uploaded a STAR-compiled reference of the human genome.

Now, we'll launch an instance using the image we've created.  STAR requires at least 32 GB so we can't use our t2.micro free-tier machine.

Go to "Instances", and instead of selecting Ubuntu as an AMI, click the "My AMIs" tab on the left and select your new Image.

For the machine, select the m5a.2xlarge instance type with 32 GB of RAM (current rate is $0.38/hr).

And everything else we can set up as before (except no need to open the ports for Jupyter or RStudio). Launch the instance.

Now, let's assume we have the reads (stored in a .fastq file) on our computer.  We're going to upload the fastq files to our instance using `scp` in a similar way that we've been using `ssh`:

```
# These files have been trimmed down to only 250k reads for the demo

scp -i "~/AmazonKeys/AWSKey.pem" SRR1721276_250k_1.fastq.gz ubuntu@ec2-54-193-56-170.us-west-1.compute.amazonaws.com:~

scp -i "~/AmazonKeys/AWSKey.pem" SRR1721276_250k_1.fastq.gz ubuntu@ec2-54-193-56-170.us-west-1.compute.amazonaws.com:~
```

And now we can run our alignment:

```
nohup STAR --runMode alignReads --runThreadN 8 --genomeDir ~/References/hg38_STAR --readFilesIn SRR1721276_250k_1.fastq.gz SRR1721276_250k_2.fastq.gz --outFileNamePrefix alignedOut &
```

Two things are important in this command:

1. `nohup` at the start
2. Ends with `&`

The `nohup` is used on the command so that we can log off our instance without stopping our STAR job.  Similarly the `&` at the end runs the job in the background so that we can continue using the terminal.  All the job output messages (that would normally be printed to the terminal) are logged to a file `nohup.out`.

If you want your instance to shut down automatically as soon as the job is done (and save the compute costs), you could instead put the following into a file, let's call it `job.sh`:

```
STAR --runMode alignReads --runThreadN 8 --genomeDir ~/References/hg38_STAR --readFilesIn SRR1721276_250k_1.fastq.gz SRR1721276_250k_2.fastq.gz --outFileNamePrefix alignedOut

sudo shutdown
```

And then run:

```
nohup bash job.sh &
```

Note here the `nohup` and the `&` are both outside the file.

Later, you can start your instance back up, log into it, and download the results - again using `scp` with something like:

```
scp -i "~/AmazonKeys/AWSKey.pem" ubuntu@ec2-54-193-56-170.us-west-1.compute.amazonaws.com:~/alignedOut/outputfile.sam .
```

## Command Line Interface (CLI)

AWS has an extensive command-line interface documented [here](https://aws.amazon.com/cli/).  Basically, everything we did on their web portal, you could do from the command line.  This can be useful if you want to write scripts that involve AWS.  Also, you can use the command line tools from within instances you create.  For an example of how this could be useful - imagine you were managing many different RNA-seq references.  Instead of putting them on your images, you can put them in Amazon S3 - which is kind of like a giant web hard drive, and then access them from your image.  You'd use commands like:

```
# Upload the reference into s3
aws s3 cp hg38_STAR s3::/references/hg38_STAR --recursive

# Then in you instance, run this to download it
aws s3 cp s3::/references/hg38_STAR hg38_STAR --recursive
```
There's no charge for data transfers within the same region, though you would still pay for the storage of the reference (about half the cost per GB of volume storage).

## Raising Utilization Limit for Instance

You *may* get an error launching the instance - something about your utilization limit for that instance being exceeded.  This happens because for some instance types, Amazon automatically sets a limit of 0 intances on new AWS accounts.  I believe this is done for security purposes (so that someone getting your login key can't just spin up 1000 instances to mine bitcoin at your expense).  If you run into this error, it's fairly easy to request a limit increase in the support center.  I've found that usually Amazon responds to these requests in a few hours.

Click 'Support' in the upper-right corner and select 'Suppert Center' from the dropdown.

Create a support case and select 'Service limit increase'.  Here select the Region you are using and the Instance tpe (e.g. c5.24xlarge) and request to raise the instance limit to 1.

## Spot Instances

Spot instances are available for much cheaper (often around 1/3 the cost, [pricing here](https://aws.amazon.com/ec2/spot/pricing/)) but they come with two caveats.

**1. the price of spot instances can increase with demand**

When you request spot instances you specify what you are willing to pay and if the price goes above that, then your instances will be halted.  However, if you just bid the current on-demand price (the default if you leave the bid price blank), then this is not likely to happen.

**2. You can't Start a spot instance once you Stop it**

With on-demand instances you can 'Stop' the instance, essentially pausing it indefinitely.  You won't be charged for compute costs while it is stopped and then you can start it again at a later time.  With a spot instance, once you stop it you can't start it again, so you must download the results of a computation before stopping the instance.  Alternately, you can attach an extra storage volume to the instance and save your results on that volume, then mount that with another instance later to get a similar kind of behavior - just with a bit more work.

## Jupyter + Python

The bare Ubuntu 18.04 instance we launched has Python 3.6 installed already, but we'll need to install 'pip' to download other packages:

```
sudo apt-get update
sudo apt-get install python3-pip
```

You can now `pip install` any other python packages you want.  Let's install jupyter:

```
pip3 install jupyter
echo "export PATH=~/.local/bin:\$PATH" >> ~/.bashrc
source ~/.bashrc
```

Now we can run a jupyter notebook server like so:

```
jupyter notebook --ip 0.0.0.0 --port 8888
```

Here we specify the port that we opened earlier.  We also needed to specify the ip as '0.0.0.0' to make the notebook accessible remotely - we are 'remote' to this machine since it's in an Amazon datacenter somewhere.

Connecting to it is a little tricky - jupyter will show you one address which looks like this:

```
http://ip-172-31-13-125:8888/?token=9fa221c16f2a4453e59cf49b06c7c56ae30366346e166ad3
```

However, it gets the wrong IP address since the machine only knows it's internal IP address (behind Amazon's routing infrastructure).  We can replace this with the correct IP address by looking it up on the dashboard:

<img src="images/instanceIP.png" alt="Instance IP Address">

First copy the address that jupyter shows to the address bar in your web browser.  Then replace everything in-between the `//` and the first `:` with the Public IP.  So if your public IP was 54.193.76.55, you would wind up with:

```
http://54.193.76.55:8888/?token=9fa221c16f2a4453e59cf49b06c7c56ae30366346e166ad3
```

Congrats! Now you're up and running and can create a notebook.

### R + RStudio

We can follow the instructions [here](https://www.rstudio.com/products/rstudio/download-server/) to install R and RStudio Server.  RStudio "Server" is probably different than the RStudio you have on your laptop as it's designed to be accessed over the web.

First, install R:

```
sudo apt-get update
sudo apt-get install r-base
```

Once you ran this you can run `R` and work with R in the command-line if you want.

Install RStudio Server:

```
sudo apt-get install gdebi-core
wget https://download2.rstudio.org/server/bionic/amd64/rstudio-server-1.2.1335-amd64.deb
sudo gdebi rstudio-server-1.2.1335-amd64.deb
```

(If you are reading this many months after Sept 2019, these commands may be out of date and you should follow [this link](https://www.rstudio.com/products/rstudio/download-server/) for the updated commands)

Before launching the server, we need to set a password for our account.  To do this run the following command and choose a password.

```
sudo passwd ubuntu
```
And now we can start the server with:

```
rstudio-server start
```

Then, we can navigate to the running server in our web browser.  First get the external IP of your instance:

<img src="images/instanceIP.png" alt="Instance IP Address">

Then navigate to `http://<instance-ip-address>:8787`

And finally, RStudio is at your fingertips!

## (Potential) Interesting Reading

I recentely carried out a performance study comparing modern cloud instances and the NERSC supercomputer Cori:

> Guidi, Giulia, et al. "10 Years Later: Cloud Computing is Closing the Performance Gap." arXiv preprint arXiv:2011.00656 (2020). To appear in 
HotCloudPerf 2021.

## Contact

If you're interested in learning more about HPC (in the cloud and not) or have further questions about how to use AWS, please contact me at ```gguidi at berkeley dot edu```. 

## Acknowledgments

Thanks to David DeTomaso for developing the first version of this tutorial, AWS for a lot of tutorial material to get me started, and thanks to [RISE Lab at UC Berkeley](https://rise.cs.berkeley.edu/) for providing AWS credits.
