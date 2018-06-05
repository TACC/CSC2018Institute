# Introduction To Ansible

Suppose we wanted to provide a few of our colleagues with an identical Jupyter environment. If we just had one or two colleagues, maybe the manual approach wouldn't be too bad. But what if we had a lab of 5? 10? What if we wanted to provide a Jupyter environment for each student in a class? Also, what happens when Joe accidentally destroys his environment and we have to recreate it from scratch? Pretty soon we'd be spending all our time installing Jupyter environments. We need a way to automate it. Ansible is one possible tool.


### New VMs

During the course of this morning's session, we will be reconstructing our Jupyter VM from scratch. Before we begin, start up three new VMs using the Horizon interface (https://tacc.jetstream-cloud.org -- Ensure you're on the TG-TRA170023
 project). This time, we will use a slightly different image. Here are the details to use on the Launch Instance pop-up:

  * Details: give your instance a name and put `3` for the Count, and click Next (Don't click launch instance)
  * Source: Click `No` under `Create New Volume`
  * Source: Enter `Ubuntu 16.04 Devel and Docker` in the search and click the plus (+) to select the 5/26/18 image.
  * Flavor: Click the plus next to m1.medium
  * Networks: Choose the TG-TRA170023-subnet by clicking plus.
  * (Skip Network Ports by clicking next)
  * Security Groups: Select `default` and `jupyter` security groups.
  * Key Pair: Select the key you created on day 1.
  * Click Launch instance.

This should spin up 3 instances for you. Once the first instance is spawned and in the running state:
  * Associate a floating IP
  * Make sure you can ssh to it. For this instance, ssh as the `root` user (e.g. `ssh -i <key> root@<ip>`).

Next, you will need to copy your SSH key to your first instance. On Mac/Unix you can use the `scp` utility:

```
$ scp -i <your_key> <your_key> root@<ip>:~
```

On Windows, use `WinSCP`. Make sure the key is copied into a file on the destination called `/root/.ssh/id_rsa`

Aside: SSH keys work like real-world padlocks and keys, except you pass out as many padlocks as you want and keep the one private key to yourself. So you can give another person (or computer) the padlock for them to install on a door (or computer) and you will be the only one who can get in, because you're the only one with the key.

Once your key is copied to your first instance, make sure you can use it to SSH to your other two instances. To do so, we use our key and the private IP of our other instance.

```
$ hostname
mpackard03-01
$ ssh -i <key> root@10.10.100.x 
# hostname
mpackard03-02
# exit
```


## Basic Notions

Ansible is a tool for automating system administration and operation tasks. It uses SSH to talk to a collection of servers and do tasks such as: configure aspects of the server, install or upgrade software, and pretty much any other task you could do at the command line.

Ansible does not run any agents on the remote servers. Instead, it connects over SSH to each server and runs a (typically small) program to accomplish each task. Users write Ansible scripts to perform multiple tasks on a given set of servers. Users then execute these scripts from any machine that has access to Ansible and 

Make sure Ansible is installed on your VM:
```
$ ansible --version

```
If not, install it with:
```
$ sudo apt-get install ansible
```

### Hosts Files
Ansible has the notion of a hosts or inventory file. This is a plain text file that describes all of the servers you want Ansible to interact with. Each host listed in the file should provide the basic SSH connectivity information for Ansible to use to perform tasks on them. Hosts can be also be collected into groups.

Here is an example hosts file with three hosts, two in the "web" group and one in the "databases" group.

```
[web]
web_1 ansible_ssh_host=129.114.18.27 ansible_ssh_private_key_file=~/web.key ansible_ssh_user=ubuntu
web_2 ansible_ssh_host=129.114.18.28 ansible_ssh_private_key_file=~/web.key ansible_ssh_user=ubuntu

[databases]
mysql ansible_ssh_host=129.114.19.33 ansible_ssh_private_key_file=~/db.key ansible_ssh_user=centos

```

Note that each line corresponding to a host begins with a name, which can be any identifier we want, and then provides two variables `ansible_ssh_host` and `ansible_ssh_private_key_file`. 

```
Exercise. Use `vi` to create a hosts file (called `hosts`) for your new hosts. Don't worry about putting them in a group yet, but for each line be sure to:
  * add the private key variable with value equal to your private key
  * add the ssh user variable with value `root`
  * add the ssh host variable using the private subnet IP `10.10.100.x`).
```

> Each new instance we create is automatically given an IP on a private network. This IP cannot be reached from the public internet, but it *can* be reached from any other IP on the same network. 

Once we have a hosts file, we can already run basic commands against the host(s). To run a command on all hosts in a hosts file, use the following:

```
$ ansible all -i <hosts_file> -a '<command>'
```

For example, let's check the uptime of the server by running the "uptime" command:
```
$ ansible all -i hosts -a 'uptime'
```
The first time you run a command you may need to answer `yes` at the prompt to continue.


```
Exercise. Use Ansible to check:
  * the available memory on each host and 
  * the number of processors on each host.
(The commands `free` and `nproc` will provide this information for a given host).
```


### Modules and Tasks

Part of Ansible's power comes from its rich library of modules. Modules are configurable programs that can be used to accomplish specific tasks. For example, we have the "command" module for simply running arbitrary commands (this is what we were using above). But there are many other modules for describing tasks at a higher level. For example, Ansible provides the "copy" module for ensuring files from the Ansible host machine are present on the remote server.

Module basics:
  * specify a module to use by passing the name of the module to the `-m` flag
  * specify parameters to the module using the `-a` flag and a string of `<name>=<value>` pairs, separated by a space.
  
The copy module requires `src` and `dest` parameters. 

Let's use the copy module to copy a file called "test.txt" in the current directory to `/root` in the remote server. We'll name it "remote.txt" on the remote:
```
$ touch test.txt
$ ansible all -i hosts -m copy -a "src=test.txt dest=/root/remote.txt"
10.10.100.7 | SUCCESS => {
    "changed": true, 
    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709", 
    "dest": "/root/remote.txt", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "d41d8cd98f00b204e9800998ecf8427e", 
    "mode": "0644", 
    "owner": "root", 
    "size": 0, 
    "src": "/root/.ansible/tmp/ansible-tmp-1500597877.29-186213310577275/source", 
    "state": "file", 
    "uid": 0
}

. . .
```

After running that command we see some output (note the color). In particular, we see a `"changed": true` . Ansible detected that we it needed to actually change the host to ensure that the file was there. Let's try running the command again and see what we get:
```
$ ansible all -i hosts -m copy -a "src=test.txt dest=/root/remote.txt"
10.10.100.7 | SUCCESS => {
    "changed": false, 
    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709", 
    "dest": "/root/remote.txt", 
    "gid": 0, 
    "group": "root", 
    "mode": "0644", 
    "owner": "root", 
    "path": "/root/remote.txt", 
    "size": 0, 
    "state": "file", 
    "uid": 0
}
```
This time, the output is green and Ansible indicates to us that it didn't change anything on the remote server. Indeed, Ansible's copy module first checks if there is anything to do, and if the file already there, it doesn't do anything. This is a key part of Ansible's power which we explore in the next section. But first, let's change the contents of our test.txt and then re-run our command.

```
# open test.txt in vi and change the contents:
$ vi test.txt
# . . .
# now run the command:
$ ansible all -m copy -a "src=test.txt dest=/root/remote.txt"
10.10.100.9 | SUCCESS => {
    "changed": true, 
    "checksum": "4e1243bd22c66e76c2ba9eddc1f91394e57f9f83", 
    "dest": "/root/remote.txt", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "d8e8fca2dc0f896fd7cb4cb0031ba249", 
    "mode": "0644", 
    "owner": "root", 
    "size": 5, 
    "src": "/root/.ansible/tmp/ansible-tmp-1500598336.43-267287598231809/source", 
    "state": "file", 
    "uid": 0
}
```

### Idempotence
Ansible's modules provide us with the power to ensure a given Ansible script we write is `idempotent`. A given task is said to be idempotent if performing the task once produces the same result as performing it repeatedly, assuming no other intervening actions.

Idempotence is very important for ensuring re-runnability of your provisioning scripts. Suppose you wanted to automate maintenance of a database server. The pseudo code for such a script might look like:

```
1. Create a linux user account for the MySQL service to run under
2. Install the MySQL server package
3. Start the mysql daemon
4. Create the database
5. Run the latest migrations on the database to install the schema
```

The problem is that, if we just use basic commands such as `useradd` and `mysql create database`, the script will run on a brand new server but it will fail every subsequent time. With idempotent tasks we can avoid this issue. In pseudo code, it could look like:

```
1. Ensure a linux user account exists for the MySQL service to run under
2. Ensure the MySQL server package is installed
3. Ensure the mysql daemon is running
4. Ensure the database has been created
5. Ensure the database has the latest migrations
```



