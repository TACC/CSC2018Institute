# Working with Agave Cloud Storage

One of the strengths of the Agave API is its ability to interact with a remote storage systems. Agave allows users to register and share (virtual) storage systems with other users, and interact with the associated files and folders on such systems.

In preparation for this class, we set up a file server in JetStream and used Agave's systems API to register it and share it with each of you. We will not go into all the details of Agave's Systems service, but we will point out that it has a fine grained permissions model which allows us to give each of you access to a different part of the file system. For more information on Agave's systems service, see: http://developer.agaveapi.co/#systems

Today, we will focus on Agave's files service, which allows us to interact with the files and folders that we have access to on the Agave system. There are three activities we will explore:
  * Listing files and folders
  * Uploading and downloading files
  * Creating directories


### Working with the Files Service

All actions through the files service are in reference to a specific Agave storage system. One specifies the system one wishes to interact with by providing the system's `id`. We will be exclusively using the storage system we set up for this class.
> The id for the class storage system is `csc.2018.storage`

Basic information about the Agave Files service:
  * The base URL for the Files service is https://api.tacc.utexas.edu/files/v2
  * Use the `listings` endpoint for listing files and folders: https://api.tacc.utexas.edu/files/v2/listings
  * Use the `media` endpoint for working with the raw file content (`https://api.tacc.utexas.edu/files/v2/media`)
    * downloading is done via a GET. This returns the raw bytes of the file.
    * uploading is done via a POST passing raw bytes to the `fileToUpload` POST parameter.
    * management commands (such as directory creation) are done via a PUT request, passing JSON containing:
      * `action` - the action to take (e.g. `mkdir`)
      * `path` - the path on the remote URL.
  * Incorporate the system ID by appending `system/<system_id>` after `listings` or `media`. 
  * Incorporate a path on the remote Agave system by appending it to the end.
  * Examples:
    * (download) `https://api.tacc.utexas.edu/files/v2/media/system/csc.2018.storage/<remote_path>`
    * (list) `https://api.tacc.utexas.edu/files/v2/listings/system/csc.2018.storage/<remote_path>`


```
Exercise. Use the files service to list the contents of your home directory. Your home directory is equal to your username.

Exercise. Create a directory called `test` inside your home directory. Explore the response. Note the status returned.

Exercise. List the content of your home directory again and confirm that the `test` directory is there.

Exercise. Create a little text file with some text in it on your JetStream VM and upload it to your home directory on the class storage system. (There are several ways to create the file locally). Explore the response and note the status returned.

Exercise. Confirm that your file is uploaded by doing another listing. Make two listing request. First, list your home directory and the list the file itself.

Exercise. Create an empty directory on your Jetstream VM and download the test file from your home directory on the class storage system to the empty directory. Display the contents of the newly downloaded file and confirm the contents are the same as the original file.
```

### Register Your Instance as a Storage System

We can turn our VM into an Agave storage server. In fact, the steps we are going to take to do so will work with any SSH/SFTP server you have in your own data center.

#### Generate SSH Keys
We need to generate a pair of disposable SSH keys to register our system with Agave. Open the terminal and enter:
```
$ ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

At the prompt:
```
Enter file in which to save the key (/root/.ssh/id_rsa)
```
enter a name such as `agave`

and at the prompt to enter a password, leave it blank.

> Agave cannot use an SSH key that has a password at this time.

That command should have generated `<file>.key` and `<file>.pub` in the pwd.

#### Add the public key to `/root/.ssh/authorized_keys`
Open up the authorized_keys file in an editor and be sure to use the precise file contents, not those with carriage returns.

#### Create a system.json file
Next, download the storage template file from the class github site:

```
$ wget https://raw.githubusercontent.com/TACC/CSC2018Institute/master/docs/day2/vm_storage_system_template.json -O system.json
```

Open up a Python notebook and read your public and private keys into a string object and write print them:

```
with open('agave', 'r') as f:
    s = f.read()
s
```
  * Copy/Paste the public and private key strings into the `system.json` file.
  * Add your VM's IP in the `host` entry.
  * Modify the id to somthing unique


#### Register your system
```
rsp = requests.post('https://api.tacc.utexas.edu/systems/v2', files={'fileToUpload': open('storage.json', 'rb')}, headers=headers)
```

#### Test your system

Let's list files using the files endpoint:

```
rsp = requests.get('https://api.tacc.utexas.edu/files/v2/listings/system/jfs.cscint.storage//root', headers=headers)
```

#### Share your system with others (optional)

```
data = {'username': 'some_tacc_user', 'role': 'USER'}
rsp = requests.post('https://api.tacc.utexas.edu/systems/v2/jfs.cscint.storage/roles', json=data, headers=headers)
```

