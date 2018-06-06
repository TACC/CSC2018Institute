# Introduction to Dask

Dask is a python library for working with datasets that do not fit into the memory of a single computer. 

## Dask and Dask Distributed
Dask Basic concepts:
  * Dynamic task scheduling optimized for “interactive workloads"
  * At runtime, creates a directed acyclic graph (DAG) of tasks, where nodes are tasks and edges are dependencies
  * Dask schedules individual tasks to be computed across a set of processes and automatically combines results as needed
  * With Dask distributed, these processes can live across a cluster of machines

Parallel data collections with an interface familiar to other standard Python libraries:
  * Numpy arrays are replaced by dask arrays
  * Pandas dataframes are replaced by dask dataframes
  * Dask bags can be used for arbitrary Python objects


## Building a Dask Cluster

Creating a dask cluster is quite easy. It amounts to:
  * installing dask on every node
  * starting a dask scheduler process on one machine
  * starting a dask worker process on all other machines, passing the IP and port of the scheduler
The scheduler process should be started first, then the worker processes. Bi-directional communication is required 

### Installing Dask
Dask comes with the Anaconda platform. Alternatively, it can be installed directly with `pip`:
```
$ pip install dask[complete]
```

### Starting the scheduler
Start the scheduler with the following command:
```
$ dask-scheduler &
```
This starts the scheduler on the default port, currently 8786.


### Starting Worker Nodes
To start each worker we use the following command, passing the IP and port of the scheduler:
```
$  dask-worker 10.12.0.10:8786 --nprocs=<number_of_processes>&
```
The `nprocs` flag specifies how many Dask processes to use on the worker host. The default is one but this will generally not give optimal performance.

At this point, the scheduler should be communicating with the worker nodes and forming a dask cluster. 


```
Exercise. Create a 3 node dask cluster, with one node having a public IP and running the dask scheduler and the other 2 nodes running the dask worker. Let's use 

```
The Ansible playbook in the class git repository can be used to install Anaconda which will include dask and all Python dependencies. The playbook will also start the dask scheduler and worker processes with the correct IPs and ports. 

```
In the hosts file, create a `[compute]` group and move two of the nodes from the `[master]` group to the `[compute]` group. Run the playbook.
```

Let's ensure basic connectivity with our Dask clutser. Open a notebook and work through the following commands.

first, make sure we can import dask
```
from dask.distributed import Client
```

Next, we need to instantiate a client. The client is responsible for sending work to the scheduler. To properly instantiate the client, we need to give it the scheduler's address and port. For example:
```
client = Client('10.10.100.15:8786')
```
Change the private IP above to match that of the IP where your scheduler is running (should be the same IP as your notebook server).

Let's evaluate the client object in a Jupyter cell:

```
client
<Client: scheduler='tcp://10.10.100.15:8786' processes=18 cores=18>
```
That's a good first check. It appears to be have access to 18 cores through 18 dask worker processes. A 1:1 process to core ratio is a good rule of thumb to use when running Dask on multicore.

Finally, let's run the following:

```
client.get_versions(check=True)
```
If everything is set up correctly, this will print out a list of dictionaries representing all the Dask processes on the cluster, including the client (this Python notebook), the scheduler, and all worker processes. It will include the versions of all relevant libraries (these must match, see below).


### Python Libraries and Application Data Files
The dask scheduler coordinates task execution on each of the worker processes. In order to do this efficiently, dask assumes that each worker has access to:
  * the same Python libraries (and the same version)
  * all external files including data files

Strategies for data include:
  * copying all files to each node in the cluster
  * using a network file system such as NFS, HDFS, etc.
  * using dask APIs to reference data files via a remote URL

For simplicity, we'll use the first approach. We will make use of a sample data set in CSV format we have made available on the class file server, `https://129.114.17.94`. This dataset collects all posts on the Stackoverflow site. The name of the file is `stackoverflow.csv`.

```
Exercise. Write a short playbook to ensure the sample data file is on every node.
```

We can use something like the following:
```
---

- hosts: all
  tasks:
  - name: download the class data set.
    get_url:
      url: "http://129.114.17.94/stackoverflow.csv"
      dest: "/root/stackoverflow.csv"
```

To really stretch the limits of our cluster, we'll also make use of the NYC Yellow Cab 2015 CSV datasets. Downloading all of these data to all nodes in our cluster might take some time (it took about 7 minutes in a trial run). So, let's get that stated as well. 

This is a good time to introduce Ansible's loop mechanism. The 2015 Yellow Cab dataset is divided into 12 files, one for each month. We want to download all of them to all hosts. 

```
Exercise. Write a playbook to download all 12 Yellow Cab datasets from 2015. Use the items mechanism to minimize typing.
```

The Yellow Cab datasets are all available at URLs with the following form: 

```
https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2015-<month>.csv
```

where <month> is a number (two digits) between 01 and 12. We can solve this with the following playbook:

```
---

- hosts: all
  tasks:
  - name: download the class data set.
    get_url:
      url: "https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2015-{{ item }}"
      dest: "/root/yellow_tripdata_2015-{{ item }}"
    with_items:
      - "01.csv"
      - "02.csv"
      - "03.csv"
      - "04.csv"
      - "05.csv"
      - "06.csv"
      - "07.csv"
      - "08.csv"
      - "09.csv"
      - "10.csv"
      - "11.csv"
```

