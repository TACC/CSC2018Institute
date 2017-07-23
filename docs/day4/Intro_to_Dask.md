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
Start each worker with the following command, passing the IP and port of the scheduler:
```
$  dask-worker 10.12.0.10:8786 &
```
At this point, the scheduler should be communicating with the worker nodes and forming a dask cluster. 


```
Exercise. Create a 3 node dask cluster, with one node having a public IP and running the dask scheduler and the other 2 nodes running the dask worker. 

```
The Ansible playbook in the class git repository can be used to install Anaconda which will include dask and all Python dependencies. The playbook will also start the dask scheduler and worker processes with the correct IPs and ports. 

```
In the hosts file, create a `[compute]` group and move two of the nodes from the `[master]` group to the `[compute]` group. Run the playbook.
```

### Python Libraries and Application Data Files
The dask scheduler coordinates task execution on each of the worker processes. In order to do this efficiently, dask assumes that each worker has access to:
  * the same Python libraries (and the same version)
  * all external files including data files

Strategies for data include:
  * copying all files to each node in the cluster
  * using a network file system such as NFS, HDFS, etc.
  * using dask APIs to reference data files via a remote URL

For simplicity, we'll use the first approach. We have a some sample data in CSV format on the class file server, `https://129.114.17.94` that collects all posts on the Stackoverflow site. The name of the file is `stackoverflow.csv`.

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



