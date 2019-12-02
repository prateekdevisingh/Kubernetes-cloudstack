# Kubernetes-cloudstack
This is easiest way to make public and private cloud using CloudStack with Kubernetes Orchestration 
# Kubernetes Using CloudStack (OpenSource)
CloudStack is a software to build public and private clouds based on hardware virtualization principles (traditional IaaS). To deploy Kubernetes on CloudStack there are several possibilities depending on the Cloud being used and what images are made available. CloudStack also has a vagrant plugin available, hence Vagrant could be used to deploy Kubernetes either using the existing shell provisioner or using new Salt based recipes.

CoreOS templates for CloudStack are built nightly. CloudStack operators need to register this template in their cloud before proceeding with these Kubernetes deployment instructions.

This guide uses a single Ansible playbook, which is completely automated and can deploy Kubernetes on a CloudStack based Cloud using CoreOS images. The playbook, creates an ssh key pair, creates a security group and associated rules and finally starts coreOS instances configured via cloud-init.

--------------------
1.Prerequisites

2.Support Level
--------------------

Prerequisites
--------------------------------------------------------------------------------------------------------------------------------------------

  sudo apt-get install -y python-pip libssl-dev

  sudo pip install cs

  sudo pip install sshpubkeys

  sudo apt-get install software-properties-common

  sudo apt-add-repository ppa:ansible/ansible

  sudo apt-get update

  sudo apt-get install ansible

  On CloudStack server you also have to install libselinux-python :

  yum install libselinux-python

  cs is a python module for the CloudStack API.

Set your CloudStack endpoint, API keys and HTTP method used.

You can define them as environment variables: CLOUDSTACK_ENDPOINT, CLOUDSTACK_KEY, CLOUDSTACK_SECRET and CLOUDSTACK_METHOD.

Or create a ~/.cloudstack.ini file:


[cloudstack]
 
 endpoint = <your cloudstack api endpoint>
 
 key = <your api access key>
 
 secret = <your api secret key>
 
 method = post
 
 We need to use the http POST method to pass the large userdata to the coreOS instances.

Clone the playbook
 
 git clone https://github.com/apachecloudstack/k8s
 
 cd kubernetes-cloudstack

 Create a Kubernetes cluster
 You simply need to run the playbook.

 ansible-playbook k8s.yml

Some variables can be edited in the k8s.yml file.

 vars:
   ssh_key: k8s
   k8s_num_nodes: 2
   k8s_security_group_name: k8s
   k8s_node_prefix: k8s2
   k8s_template: <templatename>
   k8s_instance_type: <serviceofferingname>

This will start a Kubernetes master node and a number of compute nodes (by default 2). The instance_type and template are specific, edit them to specify your CloudStack cloud specific template and instance type (i.e. service offering).

Check the tasks and templates in roles/k8s if you want to modify anything.

Once the playbook as finished, it will print out the IP of the Kubernetes master:

TASK: [k8s | debug msg='k8s master IP is {{ k8s_master.default_ip }}'] ********
SSH to it using the key that was created and using the core user.

ssh -i ~/.ssh/id_rsa_k8s core@<master IP>

And you can list the machines in your cluster:

fleetctl list-machines
----------------------------------------
MACHINE        IP             METADATA
----------------------------------------
a017c422...    <node #1 IP>   role=node

ad13bf84...    <master IP>    role=master

e9af8293...    <node #2 IP>   role=node
