# OpenShift Cluster Setup on AWS
The OpenShift Container Platform installation program offers four methods for deploying a cluster:

1. **Interactive:** You can deploy a cluster with the web-based Assisted Installer. This is the recommended approach for clusters with networks connected to the internet. The Assisted Installer is the easiest way to install OpenShift Container Platform, it provides smart defaults, and it performs pre-flight validations before installing the cluster. It also provides a RESTful API for automation and advanced configuration scenarios.

2. **Local Agent-based:** You can deploy a cluster locally with the agent-based installer for air-gapped or restricted networks. It provides many of the benefits of the Assisted Installer, but you must download and configure the agent-based installer first. Configuration is done with a commandline interface. This approach is ideal for air-gapped or restricted networks.

3. **Automated:** You can deploy a cluster on installer-provisioned infrastructure and the cluster it maintains. The installer uses each cluster host’s baseboard management controller (BMC) for provisioning. You can deploy clusters with both connected or air-gapped or restricted networks.

4. **Full control:** You can deploy a cluster on infrastructure that you prepare and maintain, which provides maximum customizability. You can deploy clusters with both connected or air-gapped or restricted networks.

## About the installation program
We can use the installation program to deploy each type of cluster. The installation program generates main assets such as Ignition config files for the bootstrap, control plane (master), and worker machines. We can start an OpenShift Container Platform cluster with these three configurations and correctly configured infrastructure.

The OpenShift Container Platform installation program uses a set of targets and dependencies to manage cluster installations. The installation program has a set of targets that it must achieve, and each target has a set of dependencies. Because each target is only concerned with its own dependencies, the installation program can act to achieve multiple targets in parallel with the ultimate target being a running cluster. The installation program recognizes and uses existing components instead of running commands to create them again because the program meets dependencies.

<img src="./screenshots/0-OpenShift-Container-Platform-installation-targets-and-dependencies.png" width="90%" />

Each cluster machine uses Red Hat Enterprise Linux CoreOS (RHCOS) as the operating system. RHCOS is the immutable container host version of Red Hat Enterprise Linux (RHEL) and features a RHEL kernel with SELinux enabled by default. It includes the kubelet, which is the Kubernetes node agent, and the CRI-O container runtime, which is optimized for Kubernetes.

Every control plane machine in an OpenShift Container Platform 4.12 cluster must use RHCOS, which includes a critical first-boot provisioning tool called Ignition. This tool enables the cluster to configure the machines.

### Technology Stack
Following technology stack is being used for the cluster provisioning:
- AWS Route 53
- AWS IAM 
- Domain provider (e.g. Namecheap)
- OpenShift Container Platform

## Pre-Requisites
Following prerequisites must be met before OpenShift cluster provisioning.
- Configure an **AWS account** to host the cluster.
- An **IAM user** with **Admin access**.  
- An **installer machine** to start the provisioning step. 
    - We can have a Virtual Machine on AWS acting as installer machine. 
- A **Domain name** to access the application deployed on OpenShift cluster.
- A RedHat account. 

## Step 1: Create a Public Hosted Zone using the Route 53 
I have already acquired domain name, i.e. **asadhanif.dev**, from Namecheap, Inc. 

Sign in to the **AWS Management Console** and open the **Route 53 console** at https://console.aws.amazon.com/route53/.

**Step 1 (a):** In the Create Hosted Zone pane, enter the name of the domain to route traffic for. Following image is presenting the created public hosted zone. 

<img src="./screenshots/1-public-hosted-zone.png" width="90%" />


**Step 1 (b):** Next step is to update the DNS records at Namecheap. 

<img src="./screenshots/2-namecheap-dns-records.png" width="90%" />

## Step 2: Creating an IAM User on AWS

**Step 2 (a):** Create the **IAM user** with *administrator* access using the AWS Management Console.

<img src="./screenshots/3-iam-user-account.png" width="90%" />

**Step 2 (b):** We also need create access key to enable programmatic access, so that openshift can access and manage the aws account for resource provisioning.  

<img src="./screenshots/4-programmatic-access.png" width="90%" />

## Step 3: Generating a key pair for cluster node SSH access
During an OpenShift Container Platform installation, we can provide an SSH public key to the installation program. The key is passed to the Red Hat Enterprise Linux CoreOS (RHCOS) nodes through their Ignition config files and is used to authenticate SSH access to the nodes. The key is added to the `~/.ssh/authorized_keys` list for the `core` user on each node, which enables password-less authentication.

After the key is passed to the nodes, we can use the key pair to SSH in to the RHCOS nodes as the user `core`. To access the nodes through SSH, the private key identity must be managed by SSH for your local user.

If we want to SSH in to our cluster nodes to perform installation debugging or disaster recovery, we must provide the SSH public key during the installation process. The `./openshift-install gather` command also requires the SSH public key to be in place on the cluster nodes.

**Step 3 (a):** Create a VM on AWS witht the name **openshift-client**. This VM will be used to start the Ignition process of OpenShift cluster setup. 

<img src="./screenshots/5-openshift-client.png" width="90%" />

**Step 3 (b):** If we do not have an existing SSH key pair on our virtual machine (i.e. `openshift-client`) to use for authentication onto our cluster nodes, create one. SSH to the VM using a Linux operating system, run the following command:

`ssh-keygen -t ed25519 -N ''` 

**Step 3 (c):** View the public key, placed at `~/.ssh/id_ed25519.pub`, using following command  public key:

`cat ~/.ssh/id_ed25519.pub`

**Step 3 (d):** Add the SSH private key identity to the SSH agent for local user, if it has not already been added. SSH agent management of the key is required for password-less SSH authentication onto our cluster nodes, or if we want to use the `./openshift-install gather` command.

If the ssh-agent process is not already running for local user, start it as a background task:

`eval "$(ssh-agent -s)"`

**Step 3 (e):** Add the SSH private key, placed in `~/.ssh/id_ed25519`, to the ssh-agent using following command:

`ssh-add ~/.ssh/id_ed25519`

**Note:** When we install OpenShift Container Platform, we'll provide the SSH public key to the installation program.

## Step 4: Setup Cluster on OpenShift Console 

**Step 4 (a):** Go to the [OpenShift Console](https://console.redhat.com/openshift) to manage the cluster. 

<img src="./screenshots/6-openshift-console.png" width="90%" />

**Step 4 (b):** Click on the **Create Cluster** button, following screen will be shown. 

<img src="./screenshots/7-Select-an-OpenShift-cluster-type.png" width="90%" />

**Step 4 (c):** Scrolldown to see the **Run it yourself** option, as shown in following image.

<img src="./screenshots/8-Run-it-yourself.png" width="90%" />

**Step 4 (d):** Now click on **AWS (x86_64)** option. Following screen will be shown to choose the installation type. 

<img src="./screenshots/9-select-the-installation-type.png" width="90%" />

**Step 4 (e):** Click on **Full control** option to see the installation steps. 

<img src="./screenshots/10-Install-OpenShift-on-AWS-steps.png" width="90%" />

## Step 5: Obtaining the installation program

**Step 5 (a):** Go to the AWS Managment Console and see the list of **EC2 instances**. Get the access of **openshift-client** VM, created in step 3 (a), via SSH. 

**Step 5 (b):** Create a directory to place the installation configuration files, using following command:

`mkdir openshift-dir`

**Step 5 (c):** Download and extract the install program, i.e. OpenShift installer, on VM. 

To download the installer use following command: 

`wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-install-linux.tar.gz`

To extract the installer use following command: 

`tar -xvf openshift-install-linux.tar.gz`

<img src="./screenshots/11-extract-the-installer.png" width="90%" />

**Step 5 (c):** Move the `openshift-install` directory to the `openshift-dir` directory using following command. 

`mv openshift-install openshift-dir/`

**Step 5 (d):** Copy the pull secret from the [Red Hat OpenShift Cluster Manager](https://console.redhat.com/openshift/install/pull-secret) and store into `openshift-dir/pull-secret.txt` file using following command.

`nano openshift-dir/pull-secret.txt`

This pull secret allows you to authenticate with the services that are provided by the included authorities, including Quay.io, which serves the container images for OpenShift Container Platform components.

**Step 5 (e):** Download the OpenShift command-line tools, i.e. `oc` and `kubctl`, and add them to your PATH.

To download the OpenShift command-line tools use following command: 

`wget https://mirror.openshift.com/pub/openshift-v4/x86_64/clients/ocp/stable/openshift-client-linux.tar.gz`

To extract the OpenShift command-line tools use following command: 

`tar -xvf openshift-client-linux.tar.gz`

**Step 5 (f):** Move the `oc` and `kubectl` directory to the `openshift-dir` directory using following commands. 

`mv oc openshift-dir/`

`mv kubectl openshift-dir/`


## Step 6: Creating the Installation Configuration File
We can customize the OpenShift Container Platform cluster, that is being installed on Amazon Web Services (AWS), with the help of `install-config.yaml` file.

**Step 6 (a):** Change to the directory that contains the installation program, i.e. `openshift-dir/`, and create the `install-config.yaml` file with the help of running the following command:

`./openshift-install create install-config --dir openshift`

When specifying the directory:
- Verify that the directory has the `execute` permission. This permission is required to run Terraform binaries under the installation directory.
- Use an empty directory. Some installation assets, such as bootstrap X.509 certificates, have short expiration intervals, therefore you must not reuse an installation directory. If you want to reuse individual files from another cluster installation, you can copy them into your directory. However, the file names for the installation assets might change between releases. Use caution when copying installation files from an earlier OpenShift Container Platform version.

**Step 6 (b):** At the prompts, provide the configuration details for AWS cloud:
- Optional: Select an SSH key to use to access your cluster machines.
    - For production OpenShift Container Platform clusters on which you want to perform installation debugging or disaster recovery, specify an SSH key that your `ssh-agent` process uses.
- Select AWS as the platform to target.
- If you do not have an Amazon Web Services (AWS) profile stored on your computer, enter the **AWS access key ID** and **secret access key** for the user that you configured to run the installation program.
- Select the AWS region to deploy the cluster to.
- Select the base domain for the Route 53 service that we configured for our cluster. 
    - For our case it is `asadhanif.dev`.
- Enter a descriptive name for the cluster. 
    - For our case it is `asad-test-cluster`.
- Paste the `pull secret` from the Red Hat OpenShift Cluster Manager.

<img src="./screenshots/12-the-configuration-details-for-AWS-cloud.png" width="90%" />


**Step 6 (c):** Modify the `install-config.yaml` file as per our requirements. 

**Original** configuration file. 

```
additionalTrustBundlePolicy: Proxyonly
apiVersion: v1
baseDomain: asadhanif.dev
compute:
- architecture: amd64
  hyperthreading: Enabled
  name: worker
  platform: {}
  replicas: 3
controlPlane:
  architecture: amd64
  hyperthreading: Enabled
  name: master
  platform: {}
  replicas: 3
metadata:
  creationTimestamp: null
  name: asad-test-cluster
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  machineNetwork:
  - cidr: 10.0.0.0/16
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  aws:
    region: us-east-1
publish: External
```
We need to modify the **compute** and **control plane** sections as following: 

```
compute:
- architecture: amd64
  hyperthreading: Enabled
  name: worker
  platform: 
    aws:
      type: t3a.large
  replicas: 2
controlPlane:
  architecture: amd64
  hyperthreading: Enabled
  name: master
  platform: 
    aws:
      type: t3a.xlarge
  replicas: 1
```

Once done, save the `install-config.yaml` file. 

## Step 7: Deploying the cluster
Now we can install OpenShift Container Platform on AWS cloud platform.

**Step 7 (a):** Change to the directory that contains the installation program and initialize the cluster deployment using following command:

`./openshift-install create cluster --dir openshift --log-level=info` 



## Step x: Installing a cluster on AWS with customizations
In OpenShift Container Platform version 4.12, we can install a customized cluster on infrastructure that the installation program provisions on Amazon Web Services (AWS). To customize the installation, we need to modify parameters in the `install-config.yaml` file before installing the cluster.

