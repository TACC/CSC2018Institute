# Automating the Jupyter VM Deployment

In this section we build on what we did in the previous module towards fully automating the installation and configuration of Anaconda and the Jupyter notebook.

The basic steps of the manual installation process were:
  * Download the Anaconda installer
  * Run the Anaconda installer
  * Generating the initial Jupyter config file.
  * Modifying the Jupyter config file
  * Launching the Jupyter notebook

The basic strategy is to use Ansible modules to accomplish each of these steps. We won't work out a complete solution here, but we will work through a series of exercises that will expose the basic concepts needed to get to a complete solution. After working through the exercises, we will check out a complete, working solution from the course web site.


## Ansible playbooks

Running single tasks is nice for ad hoc work, but they aren't convenient for automating an entire installation process. Ansible playbooks allow us to combine tasks into a single script that can be run all at once.

### Putting a Task into a Playbook
Playbooks use the YAML data format. YAML is similar to JSON but uses spaces to improve readability. Lists in YAML are designated by single dash (`-`) characters. Playbooks are ultimately lists of objects, each having at a minimum the following attributes:
  * hosts - the hosts to run the tasks on
  * tasks - itself a list of tasks to run. Each task has:
    * a `name` - any text string. Ansible will print this when running the task.
    * a `module` name and any parameters.
Many other attributes are available, but these give us enough to get started. Let's understand this by looking at an example.

```
Exercise. Use vi to open up a file called playbook.yml in your home directory. Do the following:
  * On the first line, add three dash (-) characters. This is part of the YAML spec and indicates the beginning of the file.
  * On the next line, specify the hosts you want to run the tasks on; in this case, `all`.
  * Next, specify the `tasks` attribute. Note the spacing! the `tasks` keyword should line up under the `hosts` keyword. Let's add a single task to use the `copy` module to copy our `test.txt` file to each remote server.
```

Here is what the file should have:
```
---

- hosts: all
  tasks:
  - name: check the uptime
    copy: src=test.txt dest=/root/remote.txt

```

Now, to run the playbook, use the `ansible-playbook` command and pass the hosts with the `-i` flag and the name of the playbook:

```
$ ansible-playbook -i hosts playbook.yml
```
Be sure to run it at least twice. Is our playbook idempotent?


## More on Ansible Modules

Ansible has many powerful modules to accomplish tasks at a higher-level. We have already seen the `copy` module; here are a few more:

### The get_url module
The get_url module retrieves files from a public URL. This is exactly what we need to download the Anaconda installer. In it's most basic form, the `get_url` module takes two parameters:
  * `url` - the URL to download.
  * `dest` - the path on the remote server to download to. 

```
Exercise. Modify your playbook to change the task to use the get_url module to download the Anaconda installer.
```

Here is what the file should have:
```
---

- hosts: all
  tasks:
  - name: download anaconda installer
    get_url:
      url: "https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh"
      dest: "/root/Anaconda3-4.4.0-Linux-x86_64.sh"
```
Run the playbook at least twice. Is it idempotent? Confirm the results.


### The shell module
The shell module is similar to the command module, but executes the command in a shell. The shell module:
  * Takes a single shell script, enclosed in quotes. Warning: there is no parameter name for this argument!
  * Use the `args` contruct to pass additional parameters to `shell`.

We can use this module to:
  * run the Anaconda installer.
  * generate the self-signed certificate for Jupyter
  * generate the Jupyter config file

```
Exercise. Let's add another task to our playbook to run the Anaconda installer after it is downloaded. 
```
What about that interactive prompt? Fortunately, the installer has an option for us. Use the `-h` flag to see all options:

```
$ bash Anaconda3-4.4.0-Linux-x86_64.sh -h
usage: Anaconda3-4.4.0-Linux-x86_64.sh [options]

Installs Anaconda3 4.4.0

    -b           run install in batch mode (without manual intervention),
                 it is expected the license terms are agreed upon
    -f           no error if install prefix already exists (force)
    -h           print this help message and exit
    -p PREFIX    install prefix, defaults to /root/anaconda3

```

Great, looks like we can pass the `-b` flag and install it without a prompt. Therefore, as a first attempt, we can run the installer with something like:

```
  - name: run the Anaconda installer
    shell: "bash Anaconda3-4.4.0-Linux-x86_64.sh -b"
```
If we append that to our file, the result is:

```
---

- hosts: all
  tasks:
  - name: download anaconda installer
    get_url:
      url: "https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh"
      dest: "/root/Anaconda3-4.4.0-Linux-x86_64.sh"

  - name: run the installer
    shell: "bash Anaconda3-4.4.0-Linux-x86_64.sh -b"
```

Let's run it and see what happens. Running the installer does take some time but it appears to have worked. If we ssh over to one of our hosts we see the `anaconda3` directory in our home dir. However, there are still two issues:
  * If we run the playbook again we get an error
  * Running the Anaconda installer in batch mode did not generate the ~/.bash_rc file.
Let's fix these.


#### Making a shell module task idempotent
Typically, shell and command tasks will not be idempotent. One technique for getting around this is the `creates` argument. The `creates` indicates to Ansible that this task creates some resource (a file or directory). When the resource already exists, Ansible will not run the task.

```
Exercise. Let's modify our playbook to add a `creates` parameter to our installer task. The installer creates the `anaconda3` directory so we'll use that as a check.
```

The modified playbook should look like:

```
---

- hosts: all
  tasks:
  - name: download anaconda installer
    get_url:
      url: "https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh"
      dest: "/root/Anaconda3-4.4.0-Linux-x86_64.sh"

  - name: run the installer
    shell: "bash Anaconda3-4.4.0-Linux-x86_64.sh -b"
    args:
      creates: "/root/anaconda3"
```

Run the playbook again to ensure we do not get an error.

```
Exercise. Let's use the shell module again to ensure that /root/.bash_profile exists. In the step below we'll then ensure it has the correct contents.
```

The modified playbook will look like:
```
---

- hosts: all
  tasks:
  - name: download anaconda installer
    get_url:
      url: "https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh"
      dest: "/root/Anaconda3-4.4.0-Linux-x86_64.sh"

  - name: run the installer
    shell: "bash Anaconda3-4.4.0-Linux-x86_64.sh -b"
    args:
      creates: "/root/anaconda3"

  - name: ensure the .bash_profile file is present
    shell: "touch /root/.bash_profile"
    args:
      creates: "/root/.bash_profile"
```
  
### The lineinfile module
The lineinfile module can be used to manipulate text files in an idempotent way. It has a lot of options, but the key ones we will use are:
  * `path` - the path to the file on the remote server.
  * `line` - the line of text we are concerned with.
  * `state` - "present" or "absent": whether we want the line there or not.
  
```
Exercise. Add a task to the playbook to ensure that the anaconda3 bin directory is on the $PATH. We'll append the bin directory to our $PATH in our .bash_profile file, which we know exists because we ensured it in the previous task.
```
Here's what the file should look like now:

```
---

- hosts: all
  tasks:
  - name: download anaconda installer
    get_url:
      url: "https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh"
      dest: "/root/Anaconda3-4.4.0-Linux-x86_64.sh"

  - name: run the installer
    shell: "bash Anaconda3-4.4.0-Linux-x86_64.sh -b"
    args:
      creates: "/root/anaconda3"

  - name: ensure the .bash_profile file is present
    shell:  "touch /root/.bash_profile"
    args:
      creates: "/root/.bash_profile"

  - name: Ensure Anaconda to bash_profile
    lineinfile:
      state: present
      path: "/root/.bash_profile"
      line: "export PATH=/root/anaconda3/bin:$PATH"
```
Run the playbook a couple of times to make sure it works and is idempotent.



### A Word on Roles
Ansible Roles are a way to package Ansible functionality into a modular component that can be shared across multiple playbooks. Suppose you wanted to have two different Jupyter environments: one geared towards bioinformaticians and one for hazards engineering. To get the most code reuse you could organize things as follows:
  * A `jupyter` role for installing the base Anaconda/Jupyter environment
  * A `bio` role for installing additional bioinformatics libraries
  * A `engineering` role for installing additional engineering libraries.
  
Then you could have two playbooks:
  * Bioinformatics playbook which would invoke:
    * `jupyter` role
    * `bio` role
  * Engineering playbook which would invoke:
    * `jupyter` role
    * `engineering` role


### Checking Out and Running the Full Solution
The complete solution is available from the project git repository. Use the `git clone` command to copy the files to your VM:

```
$ git clone https://github.com/TACC/CSC2018Institute.git
```
This should have created a directory called `CSC2018Institute` in the current working directory. 

The playbook also requires some small changes to your hosts files. 
First, put the hosts in the `master` group by adding
```
[master]
```
at the top of the hosts file.

Second, add an additional line in your hosts file to set the password for Jupyter. Add the following to the bottom of your hosts file:

```

[all:vars]
jpassword=your_password

```

Once your hosts file is updated, run the playbook with the command:

```
$ ansible-playbook -i hosts CSC2018Institute/jupyter.yml
```

If all goes well you should see messages similar to:
```
ok: [day3-1] => {
    "msg": [
        "*** Jupyter Notebook installed. Launch by re-sourcing your .bash_profile and running the notebook: ***", 
        "*** source ~/.bash_profile ***", 
        "*** jupyter notebook --allow-root ***", 
        "*** Then point your browser at: ***", 
        "*** https://YOUR_PUBLIC_IP:8887 ***"
    ]
}
```

On your VM with a public IP, follow the instructions to source your ~/.bash_profile and launch the notebook server.



