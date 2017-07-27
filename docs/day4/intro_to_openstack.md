# Introduction to OpenStack CLI

The 
<a href="https://docs.openstack.org/python-openstackclient/latest/">
OpenStack command line interface (CLI)</a> 
is only one way to interact with OpenStack's 
<a href="https://en.wikipedia.org/wiki/Representational_state_transfer">REST</a>ful 
<a href="https://en.wikipedia.org/wiki/Application_programming_interface">API</a>
In this exercise we will use the command line clients installed on Jetstream 
instances to OpenStack entities; e.g. 
<a href="https://docs.openstack.org/image-guide/">images</a>, 
<a href="https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/server.html">instances</a>,
<a href="https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/volume.html">volumes</a>,
<a href="https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/object.html">objects</a>, 
<a href="https://docs.openstack.org/python-openstackclient/latest/cli/command-objects/network.html">networks</a>,
etc.

## Jetstream Wiki Documentation

### Getting started with the Jetstream's OpenStack API

<a href="https://iujetstream.atlassian.net/wiki/display/JWT/Using+the+Jetstream+API">
https://iujetstream.atlassian.net/wiki/display/JWT/Using+the+Jetstream+API
</a>


### Notes about accessing Jetstream's OpenStack API

<a href="https://iujetstream.atlassian.net/wiki/display/JWT/After+API+access+has+been+granted">
https://iujetstream.atlassian.net/wiki/display/JWT/After+API+access+has+been+granted
</a>

### openrc.sh for Jetstream's OpenStack API

<a href="https://iujetstream.atlassian.net/wiki/display/JWT/Setting+up+openrc.sh">
https://iujetstream.atlassian.net/wiki/display/JWT/Setting+up+openrc.sh
</a>

## Insuring that your credentials are in order

Jetstream is an XSEDE resource and you must have an XSEDE account before you
can use it either via the Atmosphere user interface or the OpenStack API.
The following steps are must work before proceeding; specifically, accessing
the Horizon dashboard. If you cannot login to the dashboard, nothing else will work.

* Log into the XSEDE User portal with your XSEDE username and password
* Problems with the XSEDE portal can be addressed by emailing help@xsede.org
* Log into the Horizon dashboard with your <b>TACC username and password</b>
* Problems with TACC credentials can be addressed at
<a href"https://portal.tacc.utexas.edu/password-reset/-/password/request-reset">
https://portal.tacc.utexas.edu/password-reset/-/password/request-reset
</a>

## openrc.sh

The openrc.sh script sets the environment variables needed to access Jetstream's API

```
# API access is always in the tacc domain
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

make sure to source the script, not execute it
```
source openrc.sh
```
Now try a simple command to see if things are working
```
openstack image list
```

## A few notes about openstack commands

### Command structure

* openstack NOUN VERB PARAMETERS
* two commonly used verbs are "list" and "show"
* list will show everything that your project is allowed to view
* show takes a name or UUID and shows the details of a specific entity 
```
openstack image list
openstack image show JS-API-Featured-CentOS6-Feb-10-2017
openstack image show 21c904b7-b7b0-4f30-bb99-09aa2412bc3c
```

### Names verses UUIDs
* names and UUIDs are interchangeable on the command line
* OpenStack will let you name two or more entities with the same names
* Once you have two entities with the same name, your only recourse is to use the UUID

## Creating your first instance

We will be following the short tutorial on the Jetstream documentation Wiki
<a href="https://iujetstream.atlassian.net/wiki/display/JWT/OpenStack+command+line">
https://iujetstream.atlassian.net/wiki/display/JWT/OpenStack+command+line
</a>

It is useful to follow what's happening in the Horizon dashboard as you execute commands.
Keep in mind that everything is project based.  More than likely, everyone in 
this tutorial is in the same class; hence, the same OpenStack project.  You will see 
the results of all the students commands reflected in the Horizon dashboard.

###What we're going to do
* Create security group and add rules
* Create and upload ssh keys
* Create and configure the network (this is only done once)
* Start an instance
* Shutdown the instance
* Dismantle what we have built

###Create security group and add rules

Create the group that we will be adding rules to
```
openstack security group create --description "ssh & icmp enabled" ${OS_USERNAME}-global-ssh
```

Create a rule for allowing ssh inbound from an IP address
```
openstack security group rule create --protocol tcp --dst-port 22:22 --remote-ip 0.0.0.0/0 ${OS_USERNAME}-global-ssh
```

Create a rule that allows ping and other ICMP packets
```
openstack security group rule create --protocol icmp ${OS_USERNAME}-global-ssh
```

Adding/removing security groups after an instance is running
```
openstack server add security group  ${OS_USERNAME}-api-U-1 ${OS_USERNAME}-global-ssh
openstack server remove remove security group ${OS_USERNAME}-api-U-1 ${OS_USERNAME}-global-ssh
```
Note: that when you change the rules within a security group you are changing them in
real-time on running instances.  When we boot the instance below, we will specify which
security groups we want to associate to the running instance.

###Access to your instances will be via ssh keys

If you do not already have an ssh key we will need to create on.  For this
tutorial we will create a passwordless key.  In the real world, you would
not want to do this
```
ssh-keygen -b 2048 -t rsa -f ${OS_USERNAME}-api-key -P ""
```

Upload your key to OpenStack
```
openstack keypair create --public-key ${OS_USERNAME}-api-key.pub ${OS_USERNAME}-api-key
```


###Create and configure the network (this is usually only done once)

Create the network
```
openstack network create ${OS_USERNAME}-api-net
```

List the networks; do you see yours?
```
openstack network list
```

Create a subnet within your network.  Note the X & Y in the address range.  Each student 
in this class (technically, this OpenStack project) will need to use a unique subnet range
```
openstack subnet create --network ${OS_USERNAME}-api-net --subnet-range 10.X.Y.0/24 ${OS_USERNAME}-api-subnet1
```

Create a router
```
openstack router create ${OS_USERNAME}-api-router
```

Attach the router (gateway) to your subnet
```
openstack router add subnet ${OS_USERNAME}-api-router ${OS_USERNAME}-api-subnet1
```

Attach your router to the public external gateway
```
openstack router set --external-gateway public ${OS_USERNAME}-api-router
```

Note the details of your router
```
openstack router show ${OS_USERNAME}-api-router
```


###Start an instance

Note the flavors (sizes) of instances that create
```
openstack flavor list
```

Note the possible images that you can use
```
openstack image list | grep JS-API-Featured
```

```
openstack server create ${OS_USERNAME}-api-U-1 \
--flavor m1.tiny \
--image <IMAGE-NAME> \
--key-name ${OS_USERNAME}-api-key \
--security-group ${OS_USERNAME}-global-ssh \
--nic net-id=${OS_USERNAME}-api-net
```

Create an IP address...
```
openstack floating ip create public
```

...then add that IP address to your running instance.
```
openstack server add floating ip ${OS_USERNAME}-api-U-1 <your.ip.number.here>
```

Is the instance reachable?
```
ping <your.ip.number.here>
ssh root@<your.ip.number.here>
```


###Putting away your instance

Reboot the instance (shutdown -r now)
```
openstack server reboot ${OS_USERNAME}-api-U-1
openstack server reboot ${OS_USERNAME}-api-U-1 --hard
```

Stop the instance (shutdown -h now)
Note that state is not retained and that resources are still reserved on the compute host for when you 
decide restart the instance
```
openstack server stop ${OS_USERNAME}-api-U-1
openstack server start ${OS_USERNAME}-api-U-1
```

Pause the instance
Note that your instance still remains in memory, state is retained, and resources continue
to be reserved on the compute host assuming that you will be restarting the instance.

```
openstack server pause ${OS_USERNAME}-api-U-1
openstack server unpause ${OS_USERNAME}-api-U-1

```

Put the instance to sleep; similar to closing the lid on your laptop.  
Note that resources are still reserved on the compute host for when you 
decide restart the instance
```
openstack server suspend ${OS_USERNAME}-api-U-1
openstack server resume  ${OS_USERNAME}-api-U-1
```

Shut the instance down and move to storage.  Memory state is not maintained. Ephemeral 
storage is maintained. 
```
openstack server shelve ${OS_USERNAME}-api-U-1
openstack server unshelve ${OS_USERNAME}-api-U-1
```


###Dismantling what we have built

Remove the IP from the instance
```
openstack server remove floating ip ${OS_USERNAME}-api-U-1 <your.ip.number.here>
```

Delete the instance
```
openstack server delete ${OS_USERNAME}-api-U-1
```

Unplug your router from the public network
```
openstack router unset --external-gateway ${OS_USERNAME}-api-router
```

Remove the subnet from the network
```
openstack router remove subnet ${OS_USERNAME}-api-router ${OS_USERNAME}-api-subnet1
```

Delete the router
```
openstack router delete ${OS_USERNAME}-api-router
```

Delete the subnet

```
openstack subnet delete ${OS_USERNAME}-api-subnet1
```

Delete the network

```
openstack network delete ${OS_USERNAME}-api-net
```







