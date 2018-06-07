# Singularity @ TACC

We are continuously rolling out new super computers at TACC, and they're all unique.

| System | Cores/Node | Fabric | OS | Pros | Limitations |
|:-------|------------|--------|----|------|:------------|
| [Stampede 2 Phase1](https://portal.tacc.utexas.edu/user-guides/stampede2) | 68 | Intel OPA | CentOS 7 | Thousands of nodes, KNL processors | Slow for serial code                       |
| [Stampede 2 Phase2](https://portal.tacc.utexas.edu/user-guides/stampede2) | 48 | Intel OPA | CentOS 7 | Thousands of nodes, Skylake processors | FAST |
| [Lonestar 5](https://portal.tacc.utexas.edu/user-guides/lonestar5) | 24 | Cray Aries | SUSE 11 | Compute, GPUs, Large-mem | UT only, slow external network                              |
| [Wrangler](https://portal.tacc.utexas.edu/user-guides/wrangler) | 24   | Mellenox FDR | CentOS 7 | SSD Filesystem for fast I/O, Hosted Databases, Hadoop, HDFS | Low node-count           |
| [Jetstream](https://portal.tacc.utexas.edu/user-guides/jetstream) | 24 | 40 Gb Ethernet | ANY | Long running instances, root access | Limited storage                            |
| [Maverick](https://portal.tacc.utexas.edu/user-guides/maverick) | 20   | Mellanox FDR | CentOS 6 | GPUs, high memory nodes                   | Deprecated software stack                 |

This diverse ecosystem requires that each package be compiled from source for each system. To reduce our load and improve usability for users with complex software dependencies or no experience compiling software in user space, we have been supporting [Singularity](http://singularity.lbl.gov/index.html).

<center><a href="https://singularity.lbl.gov"><img height="250" align="middle" src="https://singularity.lbl.gov/images/logo/logo.svg"></a></center>

## Intro to Containers

Containers were created to isolate applications from the host environment. This means that all necessary dependencies are packaged into the application itself, allowing the application to run anywhere containers are supported. With container technology, administrators are no longer bogged down supporting every tool and library under the sun, and developers have complete control over the environment their tools ship with.

### Container technologies

Even if you haven’t run or built a docker container, you have probably heard of the technology. Docker has become extremely popular for both applications and services, but it requires elevated privileges, making it a security risk for shared servers. Singularity was designed to run without root privileges while also providing access to host devices, making it a good fit for traditional HPC environments.

| | Docker | Singularity |
|:--|:-:|:-:|
| Runs docker containers | X | X |
| Edits docker containers | X | X |
| Interacts with host devices | X | X |
| Interacts with host filesystems | X | X |
| Runs without sudo | | X |
| Runs as host user | | X |
| Can become root in container | X | |
| Control network interfaces | X | |
| Configurable capabilities for enhanced security | | X |

### Singularity Workflow

1. (optional) Build and modify containers with root
   * [Singularity recipe](http://singularity.lbl.gov/docs-recipes)
   * [Dockerfile](https://docs.docker.com/engine/reference/builder/)
2. Transfer container to execution system
   * `scp`
   * `singularity pull` from
     * [Singularity Hub](https://singularity-hub.org/)
     * [Docker Hub](https://hub.docker.com/)
3. Execute container

While singularity recipes will always work as expected with singularity, you will get more mileage and a larger community utilizing Docker infrastructure.

## Development Environment

You will need to install

- Docker
- Singularity

on your development systems

### Installing Singularity

```
VERSION=2.3.2
wget https://github.com/singularityware/singularity/releases/download/$VERSION/singularity-$VERSION.tar.gz
tar -xvf singularity-$VERSION.tar.gz
cd singularity-$VERSION
./configure --prefix=/usr/local
make
sudo make install
cd ..
rm -rf singularity*
singularity --version
```

Our HPC systems are still running Singularity 2.3, and 2.5 is not back-compatible.

### Installing Docker

Instructions for

- [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
- [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
- [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)

### Checking the Installation

```
# Check Singularity
singularity pull --size 512 docker://debian:latest
singularity exec debian-latest.img cat /etc/*release
cat /etc/*release

# Check Docker
sudo docker ps
```

Neither of these commands should result in an error.

## Exploration

Lets take a look at different ways to run singularity and docker containers and learn about the environments they present.

We already have a debian image for Singularity (`debian-latest.img`), so lets pull it with docker as well.

```
sudo docker pull debian:latest
```

### Running commands in a container

Similar to `docker run`, Singularity has `singularity exec`. Both commands run a single command and then exit. Lets learn about the environment by trying a few things:

#### What OS is running?

Print out `/etc/*release` with `cat`

| System | Command |
|--------|:---------|
| Host   | `cat /etc/*release` |
| Docker | `sudo docker run --rm -it debian:latest cat /etc/*release` |
| Singularity | `singularity exec debian-latest.img cat /etc/*release` |

> *note*: It is impossible to modify a singularity image without `sudo` so no additional flag like `--rm` is necessary.

#### Who are you running as?

Use the `whoami` command to see who you are running as.

#### When are commands expanded?

Change cat to head. You may notice an error this time since our `/etc/*release` glob is evaluated on host. If we wanted it evaluated in the container, we would need to run something like

```
bash -c 'head /etc/*release'
```

#### Can you chain commands?

Try running `whoami && whoami` in each environment.

> Just like with globs, you would need to encapsulate chained commands.

#### Can you see your files?

Print out all the files in `/home` with

```
find /home
```

in each environment.

> You hopefully noticed that singularity mounted your `/home/[username]` directory. This is a convenience feature for working with data outside the container.

#### Entering a container

You can also enter a container to help prototyping in the actual environment.

```
# Singularity
singularity shell debian-latest.img

# Docker
sudo docker run --rm -it debian:latest bash
```

> In both cases you need to type `exit` to leave the container.

Try creating files in different locations:

- `touch /test1.txt`
- `touch /home/$(whoami)/test2.txt`
- `touch /tmp/test3.txt`

Did any persist outside of the container?

## Singularity @ TACC

When running at TACC, you need to be aware of several things when writing your Dockerfiles.

- Filesystems
 - We currently have overlay disabled when running singularity. This means you need to include filesystem mounts like `/work` in your image.
- MPI Fabric
 - For MPI to work, you need install the same version of MPI that exists on the host so it can correctly communicate with the device drivers and interact with the spawner.
- GPUs
 - As with MPI, the version of CUDA in the container needs to match on the outside.

To make development of Singularity images for use at TACC, I have been assembling a collection of images to start `FROM`.

### Filesystems

My top-level image

- Source - https://github.com/zyndagj/tacc-base
- Docker Hub - https://hub.docker.com/r/gzynda/tacc-base/

starts `FROM` Ubuntu 18.04 LTS and includes

- mounts for every filesystem at TACC
- a `docker-clean` utility for deleting temporary files
 - Docker images track all changes, and grow quickly. You can test this with
 ```
 RUN dd if=/dev/zero of=1g.img bs=1 count=0 seek=1G
 RUN rm 1g.img
 RUN dd if=/dev/zero of=1g.img bs=1 count=0 seek=1G
 RUN rm 1g.img
 ```
 and circumvent this limitation with
 ```
 RUN dd if=/dev/zero of=1g.img bs=1 count=0 seek=1G && rm 1g.img
 RUN dd if=/dev/zero of=1g.img bs=1 count=0 seek=1G && rm 1g.img
 ```

This is a great starting point for simple serial or threaded applications, since it is a barebones system plus mount points for interacting with data.

### MPI

Singularity was designed to support HPC applications, so it naturally supports MPI and communication over the host fabric. Which usually just works because the network is the same inside and outside the container. The more complicated bit is making sure that the container has the right set of MPI libraries. MPI is an open specification, but there are several implementations (OpenMPI, MVAPICH2, and Intel MPI to name three) with some non-overlapping feature sets. If the host and container are running different MPI implementations, or even different versions of the same implementation, hilarity may ensue.

The general rule is that you want the version of MPI inside the container to be the same version or newer than the host. You may be thinking that this is not good for the portability of your container, and you are right. Containerizing MPI applications is not terribly difficult with Singularity, but it comes at the cost of additional requirements for the host system.

I reduce the burdeon on developers by having system-specific containers with MPI stacks pre-installed. For the Maverick supercomputer, my image

- Source - https://github.com/zyndagj/tacc-maverick-cuda8
- Docker Hub - https://hub.docker.com/r/gzynda/tacc-maverick-cuda8/

starts `FROM nvidia/cuda:8.0-cudnn7-devel-ubuntu16.04` to include the correct cuda stack, and then immediately replicates `gzynda/tacc-base`.

> This image would have ideally started from `gzynda/tacc-base`

Then, the image installs

- infiniband libraries
- MVAPICH2-2.2 (matches host)
- gcc toolchain (build-essential)

It also compiles the test program `hellow` to test MPI.

While you may think you should invoke MPI from in the container, and you can, you actually need to call it from the outside. You can test the MPI functionality of this container **on Maverick** as follows

```
# Start a compute job
idev -N 2 -n 40

# Pull the image
module load tacc-singularity cuda/8.0
singularity pull docker://gzynda/tacc-maverick-cuda8:latest

# Run hellow
ibrun -np 40 singularity exec tacc-maverick-cuda8-latest.img hellow
```

### GPUs

For the same reasons as with MPI Fabric devices, GPU computing with Singularity works if the host and container libraries match. The

- `gzynda/tacc-maverick-cuda8`

image serves as a decent starting point, since it contains the correct libraries, but most of Maverick's time lately is running Tensorflow for Machine learning, so I created

- Source - https://github.com/zyndagj/tacc-maverick-ml
- Docker Hub - https://hub.docker.com/r/gzynda/tacc-maverick-ml/

which contains

- tensorflow v1.6.0
- keras
- pytorch
- anaconda python 3

You can test the functionality as follows

```
# Land on a compute node
idev -m 60

# Load Singularity
module load tacc-singularity

# Change to your $WORK directory
cd $WORK

#Get the software
git clone https://github.com/tensorflow/models.git

# Pull the image
singularity pull docker://gzynda/tacc-maverick-ml:latest

# Is the GPU detected?
singularity exec --nv tacc-maverick-ml-latest.img python -c "import tensorflow as tf; tf.test.is_gpu_available()"

# Run the code
singularity exec --nv tacc-maverick-ml-latest.img python $WORK/models/tutorials/image/mnist/convolutional.py
```

## Build your own container

Use any leftover time to create your own container, or explore the examples we provided.

## Additional Resources

### Community Containers

[Docker Hub](https://hub.docker.com/) - Just remember to add mount points

[Biocontainers](https://github.com/BioContainers) - Bioinformatics containers

[Singularity Hub](https://singularity-hub.org/) - Singularity version of docker hub (does not work at TACC)

### Singularity Related Resources

[Singularity Homepage](http://singularity.lbl.gov/)

[Singularity Hub](https://www.singularity-hub.org/)

[University of Arizona Singularity Tutorials](https://docs.hpc.arizona.edu/display/UAHPC/Singularity+Tutorials)

[NIH HPC](https://hpc.nih.gov/apps/singularity.html)

[Dolmades - Windows Apps in Linux Docker-Singularity Containers](http://dolmades.org) *Warning not tested*

### Singularity Talks

Gregory Kurtzer, creator of Singularity has provided two good talks online:

[Introduction to Singularity](https://wilsonweb.fnal.gov/slides/hpc-containers-singularity-introductory.pdf)

[Advanced Singularity](https://www.intel.com/content/dam/www/public/us/en/documents/presentation/hpc-containers-singularity-advanced.pdf>)

Vanessa Sochat, lead developer of Singularity Hub, also has given a great talk on:

[Singularity](https://docs.google.com/presentation/d/14-iKKUpGJC_1qpVFVUyUaitc8xFSw9Rp3v_UE9IGgjM/pub?start=false&loop=false&delayms=3000&slide=id.g1c1cec989b_0_154>)
