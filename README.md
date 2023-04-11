# OpenShift Cluster Setup on AWS
In OpenShift Container Platform version 4.12, we can install a customized cluster on infrastructure that the installation program provisions on Amazon Web Services (AWS). To customize the installation, we need to modify parameters in the `install-config.yaml` file before installing the cluster.

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
