Google Kubernetes Engine; Explain Like I’m Five! – DevOpsLinks – Medium

![](:/e2a1f95aec6ba664dd9ea362c280468d)![0*vMU-8OGKya3pBt2w](../_resources/8caa845317e4575bff9001e4e2dc776f.jpg)

# Introduction to Using Google Kubernetes Engine; Explain Like I’m Five! (Part I)

## Creating your first managed Kubernetes cluster on Google Kubernetes Engine using Terraform, this is what we are going to cover in this tutorial.

[![1*9wCrsIu1cTJzAa0FsWrNIQ.jpeg](../_resources/a819739dad49bf1a7b57f9a560b6eb36.jpg)](https://medium.com/@eon01?source=post_header_lockup)

[Aymen El Amri](https://medium.com/@eon01)
Feb 24·12 min read

### Prerequisites

First of all, you need to have a Google Cloud account, start [here](https://console.cloud.google.com/start).

#### Install the Cloud SDK

If you are running Debian/Ubuntu, use `[apt-get](https://cloud.google.com/sdk/docs/downloads-apt-get)`

If you are running Red Hat Enterprise Linux 7/CentOS 7 use `[yum](https://cloud.google.com/sdk/docs/downloads-yum).`

If you are running Windows or macOS run the [interactive installer](https://cloud.google.com/sdk/docs/downloads-interactive) to install Cloud SDK

#### Create a Project

I recommend that you create a project for easier resource separation, management, and better security.

An “Organization” resource is automatically created the first time a user associated with a G Suite domain creates a project or billing account.

All projects and billing accounts created under your G Suite domain will be children of this Organization.

![](../_resources/42024b8348b627a0c377aa6a31fa3b77.png)![1*jFKPGA_jal-fUSzItMmtdg.png](:/39657be76cc5550473abd7267dffdf7a)

No enter the name of the project and create one:

![](../_resources/dbb4d5b7ac17d8f114e60e0319e6b002.png)![1*-XVMt-2d1bbPAUJAGsbY9g.png](../_resources/62c80c653d121acb819ea0154e36c374.png)

#### Configure the Billing Account

Make sure that billing is enabled for your project.

Go to the Billing dashboard and if you do not have a billing account, create one.

![](:/a7a286b6751eecc3b9a955076c235f96)![1*RrD53bqjojxtATARJCCY7Q.png](../_resources/fa86df78bce89743f00c1dccbb93ab2d.png)

Add the project to the billing account:

![](../_resources/a3b5af6d2df39e7a93f4e91125e90061.png)![1*Ypdar-B-HWF7JLuXiETWZA.png](../_resources/7b23f376e6e95cef6e58091ea09f4a56.png)

#### Add Kubernetes Administration Permissions

Security matters. We will use the IAM admin dashboard to manage who will access to which resource. The [Principle of least privilege](https://en.wikipedia.org/wiki/Principle_of_least_privilege) is always good to follow.

You have two choices:
**1 — Creating a Custom Role**
Here, I create a new role by cloning the Kubernetes Cluster Admin role:

![](../_resources/75047b0d47fe10d3ab1748fd898afbb6.png)![1*26q8aJU_efnw3XSHPWtBDQ.png](../_resources/47decad363c926f81276c3554d98fbec.png)

Give the new role a name and create it.

![](../_resources/5824dcfba331d79a3eb97d3812815cc5.png)![1*TgOUkXfzOqC2Yw7k5jtCxw.png](../_resources/b3e9c5d613676c842ababcd3a1c921df.png)

**2 — Using the Built-in Roles**

Another way to do this is by using the custom roles you can already find in your IAM console. For GKE, 6 roles are available:

- •Kubernetes Engine Admin
- •Kubernetes Engine Cluster Admin
- •Kubernetes Engine Cluster Viewer
- •Kubernetes Engine Developer
- •Kubernetes Engine Host Service Agent User
- •Kubernetes Engine Viewer

#### Creating the Service Account

Whether you used the first or the second way, you need to create a Service Account and add the custom role you created to it (or one of the built-in roles).

In my case, I started by creating a new Service Account:

![](../_resources/ed24cdaaedcce381fc463efacc4bd1f3.png)![1*j1tlEXYF-5YUuT-7xa24eA.png](../_resources/eaa0a3eede525702b2dbde3205cfe5af.png)

Give it a name

![](../_resources/6107d6c1d0323adde9a9a5a1c7fb9b7a.png)![1*hNDNanLKbHBfQ9WAQfpUIg.png](../_resources/c38651e09df8152ff88d385a9a5cb808.png)

And I added the role I created to the Service Account

![](../_resources/70b7c4880a2c50bb6c97586c02b58efe.png)![1*RSBY0fbcd_N8vxLF336fAg.png](../_resources/c18514473272bffeba37e5384a44e427.png)

Finally, add your email id, download a JSON key, save it to a safe place in your local machine and create the Service Account.

![](../_resources/3e5d9bd1f9d099818317ca6979c94527.png)![1*dHLtLNEgOJAqzkLHRglYhw.png](../_resources/ee70e75cca9f6174ce93b285f86af57f.png)

I saved my key to :
/home/$USER/.gcp-key.json

#### Configuring the GCloud CLI

gcloud init

![](../_resources/4f16491d8f7d186979c2128244c2af3d.png)![1*Pj4Rli0-b90Iq46yI4jxEA.png](../_resources/1a1b314fe6aefcef33e6417f97e160b5.png)

Use the generated link to login using your email and select the project we created.

#### Configuring the Service Account

Google defines the Service Account as a special type of Google account that belongs to your application or a virtual machine (VM), instead of to an individual end user.

Your application [assumes the identity of the service account to call Google APIs](https://developers.google.com/identity/protocols/OAuth2ServiceAccount#authorizingrequests), so that the users aren’t directly involved.

A service account can have zero or more pairs of service account keys, which are used to authenticate to Google.

After configuring the CLI, type:
gcloud iam service-accounts list
You will be able to see the Service Accounts you have in your project:

![1*xPtYbkSDlr-U1POtodAcPA.png](../_resources/0b40026d8ac0969715bf776504739a50.png)
Add the role `roles/iam.serviceAccountUser `to this service account:
gcloud projects add-iam-policy-binding mykubernetesproject-001 \

--member serviceAccount:myserviceaccount@mykubernetesproject-001.iam.gserviceaccount.com \

--role roles/iam.serviceAccountUser

When granted alongside `instanceAdmin.v1`, the role `roles/iam.serviceAccountUser` grants access to create VMs, attach disks to, and update metadata on VM instances that can run as a service account.

Activate the service account:

gcloud auth activate-service-account --key-file=/home/eon01/.gcp-key.json --project=mykubernetesproject-001

#### Testing the Cluster Creation

To test your cluster you can execute the following command:

gcloud container clusters create test-cluster --num-nodes 3 --region europe-west1

Make sure to adapt the previous command to your needs, for example:

- •**The cluster name**: test-cluster
- •**The number of nodes**: 1
- •**The region**: europe-west1

#### Installing Kubectl

The installation is quick and easy, in function of your operating system, follow the instructions described in [the official documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl).

For example, Debian and Ubuntu users should follow the following instructions:
`sudo apt-get update && sudo apt-get install -y apt-transport-https`

`curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -`

`echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list`

`sudo apt-get update`
`sudo apt-get install -y kubectl`

#### Destroying the Cluster

For the time being let’s destroy the cluster using:
yes | gcloud container clusters delete test-cluster --region europe-west1

*> If you are interested in receiving the next chapters and accessing the early release, you can *> [*> subscribe to this spam-free announcement list*](https://app.getresponse.com/site2/google_kubernetes_engine_by_examples?u=hWKcs&webforms_id=BTQ6h)*>  and you will be notified.*

### GKE Nodes and Node Pools

A node is simply a Virtual Machine running on the cluster.

When creating a cluster, we are able to select options like the number of nodes but also:

- •The node’s machine type,
- •The disk size
- •The node labels
- •The node image
- •If it is preemptible or not ..etc
- •Its location
- •Its version
- •..and configuration like the max nodes per pool

If we apply all of these configurations to the cluster, we will have a homogeneous cluster where all nodes have the same configuration. However, it is not possible, before the introduction of Node Pools, to create a cluster with different types of machines.

**But what is a pool?**

Well, imagine you have a cluster composed of n1-standard-1 machines and you realized that you need more CPU. With node pools feature, you can add a pool to the same cluster composed of n1-standard-4, n1-standard-8, n1-standard-16 ..etc

A node pool is simply a “pool,” of machines with the same configuration.
This feature allows the creation of heterogeneous GKE clusters.
> A node is simply a Virtual Machine running on the cluster.
> A node pool is a collection of machines with the same configuration.
In order to manipulate node pools, let’s create a new cluster:
gcloud container clusters create test-cluster \
--num-nodes 1 \
--region europe-west1
You can list the different pools of the cluster by typing:
gcloud container node-pools list \
--cluster=test-cluster \
--region europe-west1
At the moment, there is only one pool:
NAME MACHINE_TYPE DISK_SIZE_GB NODE_VERSION
default-pool n1-standard-1 100 1.11.7-gke.4
Say, we need more memory, we can do this by creating a new pool:
gcloud container node-pools create more-memory \
--cluster=test-cluster \
--region europe-west1 \
--machine-type=n1-standard-2 \
--disk-size=10
By default, all new node pools run the latest stable version of Kubernetes.

All GKE clusters come with a default pool, and the default and the minimum number of nodes in a pool is 3.

### Introduction to Using Terraform with Google Cloud

#### Terraform: Introduction

Hashicorp defines Terraform as the tool for building, changing, and versioning infrastructure safely and efficiently.

Terraform can manage existing and popular service providers like Google Cloud as well as custom in-house solutions.

Terraform is mainly an Infrastructure as code tool. It helps us describe the infrastructure using a high-level configuration syntax.

This allows a blueprint of your data center to be versioned and treated as you would any other code.

#### Terraform: Download and Installation

Terraform is distributed as a binary package, it must first be installed on your machine. Download the binary from the [download page](https://www.terraform.io/downloads.html) and install it.

For example, if you are using Linux (x86), run these commands:

wget https://releases.hashicorp.com/terraform/0.11.11/terraform_0.11.11_linux_386.zip

unzip terraform_*.zip -d /usr/bin/
chmod +x /usr/bin/terraform
Make sure Terraform is running by typing `terraform --version.`

#### Terraform: Creating Resources

Create a directory to perform our first Terraform execution then create a file called `main.tf` .

before proceeding, let’s add the Compute instance administration role to our Service Account user.

![](../_resources/80285e1fcd2057f65cdd54d7a1189794.png)![1*jpDQA19gMxh-OxbKj5r1-w.png](../_resources/1e94b3abd759c63042840aa573e0fd02.png)

We already created our account service (JSON) key, we are going to use it with Terraform in order to authenticate to Google Cloud API.

In my case, my file is located under:
/home/eon01/.gcp-key.json
This is the content of our main file:
// Configure the Google Cloud provider
provider "google" {
credentials = "/home/eon01/.gcp-key.json"
project = "mykubernetesproject-001"
region = "europe-west1"
}
// A single Google Cloud Engine instance
resource "google_compute_instance" "default" {
name = "my-vm"
machine_type = "f1-micro"
zone = "europe-west1-b"
boot_disk {
initialize_params {
image = "debian-cloud/debian-9"
}
}
network_interface {
network = "default"
access_config {
// Include this section to give the VM an external ip address
}
}
}

Make sure to adapt the main file to your case. You can change the project id, the region, the zone, the name and the type of the VM.

Now execute the command `terraform init `this will download the initialize the provider plugin, namely “google”.

> You may now begin working with Terraform. Try running “terraform plan” to see
> any changes that are required for your infrastructure. All Terraform commands
> should now work.
> If you ever set or change modules or backend configuration for Terraform,

> rerun this command to reinitialize your working directory. If you forget, other

> commands will detect it and remind you to do so if necessary.
Execute Terraform plan command:
terraform plan
Terraform will execute the following actions:

![](:/872850cd4f365f2250d0a02d946e71c5)![1*EpZd2QmVNYULpxPCk5JcZQ.png](../_resources/a9e40966fa1b821a8e2daad9f0464541.png)

Terraform apply command will not do anything but will print out the tasks that will be executed and the configuration used to execute these tasks.

In order to confirm, use `terraform apply `and a VM called my-vm will be created.

![](../_resources/39aaae3eb2af37d1948d3a9c35357191.png)![1*XmpTPnA4KdUO8-hpulUFNg.png](../_resources/9ab88d136cbfca3b682d3486bce4e9eb.png)

#### Terraform: Variables

This is the initial Terraform main file we used:
// Configure the Google Cloud provider
provider "google" {
credentials = "/home/eon01/.gcp-key.json"
project = "mykubernetesproject-001"
region = "europe-west1"
}
// A single Google Cloud Engine instance
resource "google_compute_instance" "default" {
name = "my-vm"
machine_type = "f1-micro"
zone = "europe-west1-b"
boot_disk {
initialize_params {
image = "debian-cloud/debian-9"
}
}
network_interface {
network = "default"
access_config {
// Include this section to give the VM an external ip address
}
}
}

Variables like the credentials and the project names are hardcoded and it is not a good practice especially if you are working on several projects or if you want to introduce configuration management in your project. To fix this bad practice, we are going to use Terraform variables. Let’s create the file `variables.tf:`

// variables
variable "credentials" {
default = "/home/eon01/.gcp-key.json"
}
variable "project" {
default = "mykubernetesproject-001"
}
variable "region" {
default = "europe-west1"
}
variable "zone" {
default = "europe-west1-b"
}
variable "name" {
default = "my-vm"
}
variable "machine_type" {
default = "f1-micro"
}
variable "image" {
default = "debian-cloud/debian-9"
}
variable "network" {
default = "default"
}

We included all variables in the variable files, we can now call them from the main file using this format:

${var.<variable_name>}
If the variable is a file, we should use this format:
${file("${var.<variable_name>}")}
Therefore, this is how our file looks like:
// Configure the Google Cloud provider
provider "google" {
credentials = "${file("${var.credentials}")}"
project = "${var.project}"
region = "${var.region}"
}
// A single Google Cloud Engine instance
resource "google_compute_instance" "default" {
name = "${var.name}"
machine_type = "${var.machine_type}"
zone = "${var.zone}"
boot_disk {
initialize_params {
image = "${var.image}"
}
}
network_interface {
network = "${var.network}"
access_config {
// Include this section to give the VM an external ip address
}
}
}
Make sure that you are in the same directory where your mail.tf file exists.

#### Terraform: The State

When we created a VM, Terraformstored the state about our infrastructure and its configuration.

This state is used by Terraform to map real-world resources to your configuration

It also helps to keep track of metadata and to improve performance for large infrastructures.

The state of our Terraform execution is stored in the `terraform.tfstate `file.
Run `cat terraform.tfstate `to see the content of this file.

The state is a JSON file that Terraform refreshes prior to any operation, to update it with the real infrastructure.

#### Terraform: States Backends

Say you are using Terraform to create a resource called “my-resource” and you are collaborating with a team of 2 or 3 other engineers. When someone else in the same team wants to create the resource, he/she should be able to rely on the actual state before any operation.

By default, Terraform stores the state locally, if your colleague wants to create/update/destroy the same resource, you should provide him with the latest state that you have locally.

This kind of operations can be exhausting and confusing, if each time you and your team should exchange the state files, problems may happen.

A good solution is using a backend to store and use the state files without confusing.

When you create a resource, the state is remotely saved to a backend like:

- •[Artifactory](https://www.terraform.io/docs/backends/types/artifactory.html)
- •[Azurerm](https://www.terraform.io/docs/backends/types/azurerm.html)
- •[Consul](https://www.terraform.io/docs/backends/types/consul.html)
- •[etcd](https://www.terraform.io/docs/backends/types/etcd.html)
- •[etcdv3](https://www.terraform.io/docs/backends/types/etcdv3.html)
- •[G](https://www.terraform.io/docs/backends/types/gcs.html)CS
- •[HTTP](https://www.terraform.io/docs/backends/types/http.html)
- •[Manta](https://www.terraform.io/docs/backends/types/manta.html)
- •[S3](https://www.terraform.io/docs/backends/types/s3.html)
- •[Swift](https://www.terraform.io/docs/backends/types/swift.html)
- •[Terraform enterprise](https://www.terraform.io/docs/backends/types/terraform-enterprise.html)

Since we are using Google, we will use Google Cloud Storage (GCS) to store the remote state.

Before proceeding, make sure to give the actual user enough rights to create buckets and objects.

![](:/56d5f088d553ffcb8830580fa66bb282)![1*K96BvCBd3S-1PpdgoWxnWA.png](../_resources/af534179225bcbe078980b638f8c6090.png)

Now create the bucket:
gsutil mb \
-c regional \
-l europe-west1 \
gs://terraform-state-$(date|md5sum|awk '{print $1}')
The latest command creates a bucket with a unique name:
terraform-state-$(date|md5sum|awk '{print $1}')
The bucket is regional:
"-c regional -l europe-west1"
Note that there are 4 types of buckets:

- •Multi-Regional
- •Regional
- •Nearline
- •Coldline

Let’s make sure the bucket was created using:
gsutil list

# output:

gs://terraform-state-1e1817b8049755e600d7e4ec52b850b9/
Now let’s add this code to the Terraform main file:
data "terraform_remote_state" "remote-state" {
backend = "gcs"
config {
bucket = "${var.bucket}"
prefix = "${var.prefix}"
}
}
Add the variables values to the variables file:
variable "bucket" {
default = "terraform-state-1e1817b8049755e600d7e4ec52b850b9"
}
variable "prefix" {
default = "dev"
}
These are the final files
**main.tf:**
// Configure the Google Cloud provider
provider "google" {
credentials = "${file("${var.credentials}")}"
project = "${var.project}"
region = "${var.region}"
}
data "terraform_remote_state" "remote-state" {
backend = "gcs"
config {
bucket = "${var.bucket}"
prefix = "${var.prefix}"
}
}
// A single Google Cloud Engine instance
resource "google_compute_instance" "default" {
name = "${var.name}"
machine_type = "${var.machine_type}"
zone = "${var.zone}"
boot_disk {
initialize_params {
image = "${var.image}"
}
}
network_interface {
network = "${var.network}"
access_config {
// Include this section to give the VM an external ip address
}
}
}
**variables.tf:**
// variables
variable "credentials" {
default = "/home/eon01/.gcp-key.json"
}
variable "project" {
default = "mykubernetesproject-001"
}
variable "region" {
default = "europe-west1"
}
variable "zone" {
default = "europe-west1-b"
}
variable "name" {
default = "my-vm"
}
variable "machine_type" {
default = "f1-micro"
}
variable "image" {
default = "debian-cloud/debian-9"
}
variable "network" {
default = "default"
}
variable "bucket" {
default = "terraform-state-1e1817b8049755e600d7e4ec52b850b9"
}
variable "prefix" {
default = "dev"
}

#### Terraform: Destroying Resources

If everything is working fine, we do not need our VM anymore, we can destroy it using Terraform:

terraform detroy

### What’s Next?

This article is part of “**Google Kubernetes Engine by Examples**”, a course I am working on right now to help developers understand how GKE works (networking, deployment, administration) and how to use GKE and other tools like Docker, GKE, GCS, Stackdrvier, IAM, Ansible, Terraform and [Ambassador](https://www.getambassador.io/).

This course will help you understand Google Cloud ecosystem to **build**, **run**, **deploy**, **scale**, **secure**, **log**, and **monitor**  **stateless** and **stateful**, **stable** and **scalable**  **microservices** applications.

> If you are interested in **> receiving the next chapters **> and accessing **> the early release**> , you can > [**> subscribe to this spam-free announcement list**](https://app.getresponse.com/site2/google_kubernetes_engine_by_examples?u=hWKcs&webforms_id=BTQ6h)>  and you will be notified.