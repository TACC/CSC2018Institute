# TACC Computational Science in the Cloud Intitute 


## Stand up Jupyter Notebook on cloud instance

Spin up several Ubuntu 16 boxes. Decide on one to be the master node, create an ssh keypair, and drop the public key in ~/.ssh/authorized_keys on each node, including master. Run something like this and type 'y' to accept the new connections, then run again to make sure you can cleanly ssh to them without error.

    for node in 129.114.104.8 129.114.104.9 129.114.104.10
    do
      ssh $node uptime 
    done
    
    15:56:52 up 15 days,  2:46,  4 users,  load average: 0.39, 0.57, 0.36
    15:56:52 up 15 days, 18 min,  1 user,  load average: 0.16, 0.29, 0.21
    15:56:52 up 15 days, 19 min,  1 user,  load average: 0.19, 0.33, 0.20

Install ansible on master node:

    sudo yum install -y ansible
    
Create an ansible hosts file (/etc/ansible/hosts), containing your master node IP address, compute node IP addresses, and desired password for Jupyter. It should looking something like this:

    [master]
    129.114.104.8 

    [compute]
    129.114.104.9 
    129.114.104.10

    [all:vars]
    jpassword=example_password_100

Now run the playbook:

    ansible-playbook jupyter.yml

At the end you should receive a message like this:

    "*** Jupyter Notebook installed. Launch by re-sourcing your .bashrc and running the notebook: ***", 
    "*** source ~/.bashrc ***",
    "*** jupyter notebook --allow-root ***", 
    "*** Then point your browser at: ***", 
    "*** https://129.114.104.8:8887 ***"

Fire up the jupyter notebook daemon:

    /home/mpackard/anaconda2/bin/jupyter notebook

Then open a browser and connect to the indicated url (https://129.114.104.8:8887 in this case).

## Create some cloud instances 

Visit https://tacc.jetstream-cloud.org, login.
- Domain: tacc
- Username: TACC Username
- Password: TACC Portal password

Click on _Access & Security_, then do 2 things:

- Click on Key Pairs: 
  - Import Key Pair
  - Key Pair Name should match your username exactly (e.g. _mpackard_)
  - Public Key looks something like this: 
    ssh-rsa QQQQB3NzaC1yc2EAAAABIwAAXQEAo/PLU1xr1QvPkQXebWp082YzdI0Uru+WmTXU15e2OTTMxy/PIf3Ey63ZGKBCsiQ/PBRrz78W1gZJuhizjUrube/wQ+dBHrPTd2RXXTvWpM6BrM9LwY753U/krimRQBFWweoKS3TmlLRGoWsftHF3VKi7zAnudIzGql7dmM6GCMRNskno747JqMBQMKfcPq3sRwookieg0J99S+ytgx7qaGwdmxLjL11esIblmVxtYENGMMQe+2VvYz+h+oJ69p6KIQgjz5NUhch3zO6Fga8q+Yny0CPxMFOZQxW/cFVT6p/eL0cc/s+NEFDBfN5AAOrpqApE/UgZ0VWGqY6FnwZBmQ== mpackard
- Click on API Access
  - Download Openstack RC file v3

Then, back on the terminal on your master node:

    . TG-TRA170023-openrc.sh

Launch an instance:
    
    . instance


