![Logo](logo.png)

# Backing up Docker's data in Oracle Container Cloud Service in Oracle Storage Cloud Service

Copied from my <a href="https://www.linkedin.com/pulse/test-mika-rinne">linked in article</a>.

## Step 1. Creating the Oracle Storage Cloud software appliance

The first thing to do is to install the OSCSA to the cloud. OSCSA is the component that talks REST to the Oracle Storage Cloud and hosts the logical file system that can be then NFS mounted on any of the cloud VM's to store data, including Oracle Container Cloud VM's for running the Docker instances. Here is the workflow for doing this.

After running the appliance installation script on step 5 of the workflow from my OSX I got the appliance nicely setup in the cloud and ready to go.

## Step 2. NFS mounting the appliance file system on the Container Cloud VM's

After installing the appliance the next step is to mount it's file system on client VM's.

Since we are using the VM's that run the Dockers in Container Cloud Service there is a few more steps than on regular compute instances, but it is not much. I describe them here.

When setting up an Oracle Container Cloud Service to the cloud, you specify how many worker nodes that service consists of. Each worker node will be then an individual VM of a desired shape as part of the Container Cloud Service instance and can be accessed with ssh and the public key you have.

The first thing to do is to ssh login to a worker node VM with it's public IP address and the opc user. The address is visible in the Container Cloud Service console as well as in the excellent Container Console of the service to run and orchestrate your Dockers.

When you login (for example: ssh opc@140.86.9.30) you can see the VM runs on Oracle Linux Server release 6.6.x

The next step is to install nfs-utils for NFS mounting and to do this there are three things:

Using sudo vi /etc/yum.repos.d/public-yum-ol6.repo copy the contents of this file into it. (However, if the file is there you don't have add or edit it. On regular OL6.6 compute nodes this is the case.)
Using sudo vi /etc/yum.conf remove the proxy configurations in the end of the file. You can comment (#) them out.
Finally do sudo yum install nsf-utils.
After this is done let's create the OSCSA NFS mounting point for Dockers to use. This is used to store files in the Oracle Storage Cloud service.

First, let's look at the example stack YAML. The volume specifies the location in the VM's file system to MongoDB to use for it's persistent data. The data does not disappear even the Docker stack instance is deleted and created again.

Do sudo mkdir /var/lib/mongodb to create this.

Next follow these steps to complete the NFS mounting. When adding the mounting point to /etc/fstab it's best to use tabs between values.

After saving do sudo mount -a.

## Step 3. Reboot the Docker on the Container Cloud VM

It appears that before Docker stack instances can actually access the mounting point for reading and writing the Docker daemon needs to be rebooted.

Do sudo service docker stop and then sudo service docker start for this as a the last step of the installation.

Now all should be set and you can try spinning up a Docker stack instance from the Container Console of this VM. To do this you can use the stack from this example YAML to save MongoDB data to the mounting point.

## Step 4. Testing the mounting point on another Container Cloud Service VM

By following the step 2 on another VM the MongoDB data should be automatically loaded and sync'ed to /var/lib/mongodb after doing the sudo mount -a. 

Do ls -la /var/lib/mongodb to see the same MongoDB files on the this second VM as in the first one. Voilà!

If you want to fully test MondoDB working in this example, do the following:

Call the example stack from browser using haproxy port 8886 and running some "laps" for a user named "demo" (use the IP address of the example stack's haproxy instance on the first VM and then click the tab "Userid test" from the menu). See the value counting for the user "demo".
Reboot the Docker daemon for second VM, too, as described on the step 3.
Stop the stack example instance on the first VM.
Spin up the the stack demo on the second VM. Repeat step 1 for this VM using it's example stack's haproxy instace's IP address and port 8886. The expected result should be that the example Docker stack for user "demo" continues counting from the point where it was left in the step 1.
You may try iterating this a few times to see how it works!
