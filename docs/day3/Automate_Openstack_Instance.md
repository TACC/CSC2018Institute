# Automating the Openstack VM Creation

Time permitting, we can look at another playbook which can help deploy an instance in Openstack.

## Examine the playbook

In a previous module you should have checked out this git repository: 

```
$ git clone https://github.com/TACC/CSC2018Institute.git
```
This should have created a directory called `CSC2018Institute` in the current working directory. 

Modify the playbook to include your username and the name of the instance you would like to create:
```
...
  - name: Create a new instance and attaches to a network and passes metadata to the instance
    os_server:
      state: present
      name: your_name_here04
...
      auth: 
        auth_url: https://tacc.jetstream-cloud.org:5000/v3
        username: your_username_here
        project_name: TG-TRA170023
```


The playbook also requires some small changes to your hosts files. 
First, put a single the hosts in the `openstack_client` group by adding
```
[openstack_client]
```
at the top of the hosts file with only 1 host underneath it.

Once your hosts file is updated, run the playbook with the command:

```
$ ansible-playbook -i hosts CSC2018Institute/openstack_client.yml
```

If all goes well you should see messages similar to:
```
What is your Openstack Password?: 

PLAY [web_1] ****************************************************************************************************************************
****
...
```

Once you enter your password interactively, the playbook should run and complete in a couple minutes.

Visit the Openstack Dashboard to verify that your instance was created. 




