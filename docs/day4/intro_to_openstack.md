# Introduction to OpenStack CLI

The 
<a href="https://docs.openstack.org/python-openstackclient/latest/">
OpenStack command line interface (CLI)</a> 
is only one way to interact with OpenStack's 
<a href="https://en.wikipedia.org/wiki/Representational_state_transfer">REST</a>ful 
<a href="https://en.wikipedia.org/wiki/Application_programming_interface">API</a>.
In this exercise we will use the command line clients installed on Jetstream 
instances to OpenStack entities; e.g. 
<a href="https://docs.openstack.org/image-guide/">images</a>, 
<a href="https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/server.html">instances</a>,
<a href="https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/volume.html">volumes</a>,
<a href="https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/object.html">objects</a>, 
<a href="https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/network.html">networks</a>,
etc.

# Some background getting started Jetstream Documentation

### Getting started with the Jetstream's OpenStack API

<a href="https://iujetstream.atlassian.net/wiki/display/JWT/Using+the+Jetstream+API">
https://iujetstream.atlassian.net/wiki/display/JWT/Using+the+Jetstream+API
</a>


### Notes about accessing Jetstream's OpenStack API

<a href="https://iujetstream.atlassian.net/wiki/display/JWT/After+API+access+has+been+granted">
https://iujetstream.atlassian.net/wiki/display/JWT/After+API+access+has+been+granted
</a>

### SDKs for programmatically accessing OpenStack's APIs
<a href="https://developer.openstack.org/firstapp-libcloud/getting_started.html">
https://developer.openstack.org/firstapp-libcloud/getting_started.html
<a>

### openrc.sh for Jetstream's OpenStack API

<a href="https://iujetstream.atlassian.net/wiki/display/JWT/Setting+up+openrc.sh">
https://iujetstream.atlassian.net/wiki/display/JWT/Setting+up+openrc.sh
</a>

# Getting started with the hands on portion of the tutorial

### Insuring that your credentials are in order

Jetstream is an XSEDE resource and you must have an XSEDE account before you
can use it either via the Atmosphere user interface or the OpenStack API.
The following steps must work before proceeding; specifically, accessing
the Horizon dashboard. If you cannot login to the Horizon dashboard, nothing 
else will work.

* Log into the <a href="https://www.xsede.org">XSEDE User portal</a> with your XSEDE username and password
* Problems with the XSEDE portal can be addressed by emailing help@xsede.org
* Authenticate to the Horizon dashboard in the TACC domain using your <b>TACC username and password</b>
* Problems with TACC credentials can be addressed at the <a href="https://portal.tacc.utexas.edu/password-reset/-/password/request-reset">TACC User Portal</a>

## openrc.sh

The openrc.sh script sets the environment variables that are needed to access 
Jetstream's API.  Create a file with this content named openrc.sh

```
# API access for Jetstream is always to the tacc domain
export OS_PROJECT_DOMAIN_NAME=tacc
export OS_USER_DOMAIN_NAME=tacc

# the specific project and user accessing the API
export OS_PROJECT_NAME=TG-TRA170023
export OS_USERNAME=tg455608
export OS_PASSWORD=”xxxxxxx"

# endpoint information; i.e. which cloud
export OS_IDENTITY_API_VERSION=3
export OS_AUTH_URL=https://tacc.jetstream-cloud.org:5000/v3
#export OS_AUTH_URL=https://iu.jetstream-cloud.org:35357/v3
```

To set the environment variables, source the openrc.sh file.
Note: make sure to source the script, not execute it
```
source openrc.sh
```

In the real world you will want not want to save your password in a file.
A much more secure way to set OS_PASSWORD is to read it from the command line
when the openrc is sourced.  E.g. 

```
echo "Please enter your OpenStack Password: "
read -sr OS_PASSWORD_INPUT
export OS_PASSWORD=$OS_PASSWORD_INPUT
```

You can also have the Horizon dashboard create an openrc.sh file for you.
* Log into Horizon
* Click on Identity in the left column
* Click on minor Projects tab at the bottom of the left had side
* Select the project you wish to create the openrc.sh file for
* Click on major Project tab at the top of the left hand column
* Click on Compute on the left side
* Click on Access & Security
* Click on API Access near the top of the page
* Click on Download OpenStack RC File v3
* This script will interactively prompt for the password when it is sourced.
This is much safer then keeping your password into a file.

### Our first OpenStack command

Now try a simple command to see if things are working

```
openstack image list
```

## A few notes about openstack commands

### Command structure

* <tt>openstack NOUN VERB PARAMETERS</tt>
* <tt>openstack <b>help</b> [NOUN [VERB [PARAMETER]]]</tt>
* <tt>openstack NOUN VERB <b>-h</b></tt>    will also produce the help documentation
* Common NOUNs include <tt>image, server, volume, network, subnet, router, port,</tt> etc.
* Common verbs are <tt>list, show, set, create, delete,</tt> etc.  
* Two commonly used verbs are <tt>list</tt> and <tt>show</tt>
* <tt>list</tt> will show everything that your project is allowed to view
* <tt>show</tt> takes a name or UUID and shows the details of the specified entity 

E.g.

```
openstack image list
openstack image show JS-API-Featured-CentOS6-Feb-10-2017
openstack image show 21c904b7-b7b0-4f30-bb99-09aa2412bc3c
```

### Names verses UUIDs
* names and <a href="https://en.wikipedia.org/wiki/Universally_unique_identifier">Universally Unique Identifier (UUID)</a>
are interchangeable on the command line

* <b>IMPORTANT POINT TO NOTE:</b> OpenStack will let you name two or more entities with 
the same names.  If you run into problems controlling something via its name, then 
fall back to the UUID of the entity.
* Once you have two entities with the same name, your only recourse is to use the UUID

# Creating the cyberinfrastructure and booting your first instance

We will be following the short tutorial on the Jetstream documentation Wiki<a href="https://iujetstream.atlassian.net/wiki/display/JWT/OpenStack+command+line">https://iujetstream.atlassian.net/wiki/display/JWT/OpenStack+command+line</a>

It is informative  to follow what's happening in the Horizon dashboard as you execute commands.
Keep in mind that in OpenStack everything is project based.  More than likely, everyone in 
this tutorial is in the same OpenStack project.  In the Horizon dashboard you will see 
the results of all the other students commands as they execute them.

### What we're going to do
* Create security group and add rules
* Create and upload ssh keys
* Create and configure the network (this is only done once)
* Start an instance
* Shutdown the instance
* Dismantle what we have built

### Create security group and adding rules to the group

Create the group that we will be adding rules to

```
New command: openstack security group create --description "ssh & icmp enabled" ${OS_USERNAME}-global-ssh
Old command: nova secgroup-create ${OS_USERNAME}-global-ssh "ssh & icmp enabled"
```

Create a rule for allowing ssh inbound from an IP address

```
New command: openstack security group rule create --proto tcp --dst-port 22:22 --src-ip 0.0.0.0/0 ${OS_USERNAME}-global-ssh
Old command: nova secgroup-add-rule global-ssh tcp 22 22 0.0.0.0/0

```

Create a rule that allows ping and other ICMP packets

```
New command: openstack security group rule create --proto icmp ${OS_USERNAME}-global-ssh
Old command: nova secgroup-add-rule global-ssh icmp -1 -1 0.0.0.0/0

```

Optional rule to allow connectivity within a mini-cluster;  i.e. if you boot more
than one instance, this rule allows for comminications amonst all those instances.

```
New command: openstack security group rule create --proto tcp --dst-port 1:65535 --src-ip 10.0.0.0/0 ${OS_USERNAME}-global-ssh
New command: openstack security group rule create --proto udp --dst-port 1:65535 --src-ip 10.0.0.0/0 ${OS_USERNAME}-global-ssh
Old command: nova secgroup-add-rule global-ssh tcp 1 65535 10.0.0.0/24
Old command: nova secgroup-add-rule global-ssh udp 1 65535 10.0.0.0/24

A better (more restrictive) example might be:

New command: openstack security group rule create --proto tcp --dst-port 1:65535 --src-ip 10.X.Y.0/0 ${OS_USERNAME}-global-ssh
New command: openstack security group rule create --proto udp --dst-port 1:65535 --src-ip 10.X.Y.0/0 ${OS_USERNAME}-global-ssh
Old command: nova secgroup-add-rule global-ssh tcp 1 65535 10.X.Y.0/24
Old command: nova secgroup-add-rule global-ssh udp 1 65535 10.X.Y.0/24

```

Adding/removing security groups after an instance is running

```
New command: openstack server add    security group ${OS_USERNAME}-api-U-1 ${OS_USERNAME}-global-ssh
New command: openstack server remove security group ${OS_USERNAME}-api-U-1 ${OS_USERNAME}-global-ssh
Old command: nova add-secgroup    ${OS_USERNAME}-api-U-1 global-ssh
Old command: nova remove-secgroup ${OS_USERNAME}-api-U-1 global-ssh

```

Note: that when you change the rules within a security group you are changing them in
real-time on running instances.  When we boot the instance below, we will specify which
security groups we want to associate to the running instance.

### Access to your instances will be via ssh keys

If you do not already have an ssh key we will need to create on.  For this
tutorial we will create a passwordless key.  In the real world, you would
not want to do this

```
ssh-keygen -b 2048 -t rsa -f ${OS_USERNAME}-api-key -P ""
```

Upload your key to OpenStack

```
New command: openstack keypair create --public-key ${OS_USERNAME}-api-key.pub ${OS_USERNAME}-api-key
Old command: nova keypair-add --pub-key ${OS_USERNAME}-api-key.pub ${OS_USERNAME}-api-key

```


### Create and configure the network (this is usually only done once)

Create the network

```
New command: openstack network create ${OS_USERNAME}-api-net
Old command: neutron net-create ${OS_USERNAME}-api-net

```

List the networks; do you see yours?

```
New command: openstack network list
Old command: neutron net-list
```

Create a subnet within your network.  Note the X & Y in the address range.  Each student 
in this class (technically, this OpenStack project) will need to use a unique subnet range

```
New command: openstack subnet create --network ${OS_USERNAME}-api-net --subnet-range 10.X.Y.0/24 ${OS_USERNAME}-api-subnet1
Old command: neutron subnet-create ${OS_USERNAME}-api-net 10.22.22.0/24 --name ${OS_USERNAME}-api-subnet1

```

Create a router

```
New command: openstack router create ${OS_USERNAME}-api-router
Old command: neutron router-create ${OS_USERNAME}-api-router
```

Attach your subnet to the router 

```
New command: openstack router add subnet ${OS_USERNAME}-api-router ${OS_USERNAME}-api-subnet1
Old command: neutron router-interface-add ${OS_USERNAME}-api-router ${OS_USERNAME}-api-subnet1


```

Attach your router to the public (externally routed) network

```
New command: openstack router set --external-gateway public ${OS_USERNAME}-api-router
Old command: neutron router-gateway-set ${OS_USERNAME}-api-router public


```

Note the details of your router

```
New command: openstack router show ${OS_USERNAME}-api-router
Old command: neutron   router-show ${OS_USERNAME}-api-router

```


### Start an instance

Note the flavors (sizes) of instances that create

```
New command: openstack flavor list
Old command: nova      flavor-list

```

Note the possible images that you can use on the API side of Jetstream.  

```
New command: openstack image list --limit 500 | grep JS-API-Featured
Old command: glance image list | grep JS-API-Featured

```

<b>Make sure to pick an image with the string JS-API-Featured in it</b>; unless
you are booting an image created by taking a snapshot of a JS-API-Featured instance.
Images without the -API- string are destined to be boot via Atmosphere.  Atmosphere
runs various scripts during the boot process.  If you are booting via the API then these 
scripts will not get executed and the booted instance may (probably) will not be
usable.

### Time to boot your instance

```
New command: 
openstack server create ${OS_USERNAME}-api-U-1 \
--flavor <FLAVOR> \
--image <IMAGE-NAME> \
--key-name ${OS_USERNAME}-api-key \
--security-group ${OS_USERNAME}-global-ssh \
--nic net-id=${OS_USERNAME}-api-net

Old command: 
nova boot ${OS_USERNAME}-api-c7i-001 \
  --flavor <FLAVOR> \
  --image  <IMAGE-NAME> \
  --key-name ${OS_USERNAME}-api-key \
  --security-groups ${OS_USERNAME}-global-ssh \
  --nic net-name=$${OS_USERNAME}-api-net


```

Note that ${OS_USERNAME}-api-U-1 is the name of the running instance. Pick a 
name that means something to you.  Each instance you boot should have a unique
name; otherwise, you will have to control your instances via the UUID

Create an IP address...

```
New command: openstack floating ip create public
Old command: nova      floating-ip-create public

```

...then add that IP address to your running instance.

```
New command: openstack server add floating ip ${OS_USERNAME}-api-U-1 <your.ip.number.here>
Old command: nova floating-ip-associate ${OS_USERNAME}-api-U-1 <your.ip.number.here>


```

Is the instance reachable?

```
ping <your.ip.number.here>
ssh root@<your.ip.number.here>

```


### Putting our instance into a non-running state


Reboot the instance (shutdown -r now).
```
New command: openstack server reboot ${OS_USERNAME}-api-U-1
New command: openstack server reboot ${OS_USERNAME}-api-U-1 --hard
Old command: nova reboot ${OS_USERNAME}-api-U-1


```

Stop the instance (shutdown -h now).
Note that state is not retained and that resources are still reserved on the compute
host so that when you decide restart the instance, resources are available to
activate the instance.

```
New command: openstack server stop ${OS_USERNAME}-api-U-1
New command: openstack server start ${OS_USERNAME}-api-U-1
Old command: nova stop  ${OS_USERNAME}-api-U-1
Old command: nova start ${OS_USERNAME}-api-U-1

```

Pause the instance
Note that your instance still remains in memory, state is retained, and resources continue
to be reserved on the compute host assuming that you will be restarting the instance.

```
New command: openstack server pause   ${OS_USERNAME}-api-U-1
New command: openstack server unpause ${OS_USERNAME}-api-U-1
Old command: nova pause   ${OS_USERNAME}-api-U-1
Old command: nova unpause ${OS_USERNAME}-api-U-1

```

Put the instance to sleep; similar to closing the lid on your laptop.  
Note that resources are still reserved on the compute host for when you 
decide restart the instance

```
New command: openstack server suspend ${OS_USERNAME}-api-U-1
New command: openstack server resume  ${OS_USERNAME}-api-U-1
Old command: nova suspend ${OS_USERNAME}-api-U-1
Old command: nova resume  ${OS_USERNAME}-api-U-1


```

Shut the instance down and move to storage.  Memory state is not maintained. Ephemeral 
storage is maintained. 

```
New command: openstack server shelve ${OS_USERNAME}-api-U-1
New command: openstack server unshelve ${OS_USERNAME}-api-U-1
Old command: the shelve concept did not exist before the openstack command came along

```


### Dismantling what we have built

Remove the IP from the instance

```
New command: openstack server remove floating ip ${OS_USERNAME}-api-U-1 <your.ip.number.here>
Old command: nova floating-ip-disassociate ${OS_USERNAME}-api-U-1  <your.ip.number.here>

```

Return the IP to the pool

```
New command: openstack floating ip delete <your.ip.number.here>
Old command: nova floating-ip-delete <your.ip.number.here>

```

Delete the instance

```
New command: openstack server delete ${OS_USERNAME}-api-U-1
Old command: nova delete ${OS_USERNAME}-api-U-1

```

Unplug your router from the public network

```
New command: openstack router unset --external-gateway ${OS_USERNAME}-api-router
Old command: neutron router-gateway-clear ${OS_USERNAME}-api-router

```

Remove the subnet from the network

```
New command: openstack router remove subnet ${OS_USERNAME}-api-router ${OS_USERNAME}-api-subnet1
Old command: neutron router-interface-delete ${OS_USERNAME}-api-router ${OS_USERNAME}-api-subnet1

```

Delete the router

```
New command: openstack router delete ${OS_USERNAME}-api-router
Old command: neutron router-delete ${OS_USERNAME}-api-router

```

Delete the subnet

```
New command: openstack subnet delete ${OS_USERNAME}-api-subnet1
Old command: neutron subnet-delete ${OS_USERNAME}-api-net  --name ${OS_USERNAME}-api-subnet1

```

Delete the network

```
New command: openstack network delete ${OS_USERNAME}-api-net
Old command: neutron net-delete ${OS_USERNAME}-api-net


```

## For further investigation...

A tutorial was presented at the PEARC17 conference on 
<a href="https://github.com/ECoulter/Tutorial_Practice">
how to build a SLURM HPC cluster with OpenStack
</a>  
The tutorial assumes that a node at IP 149.165.157.95 is running 
that you need to login to as a first step.  This node was provided as
an easy way to run the class and its only purpose was to provide a 
host with the openstack CLI clients installed.  You can safely skip
this step and proceed with executing the openstack commands you see
in the tutorial. 



