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

starts from Ubuntu 18.04 LTS and includes

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

I reduce the burdeon on developers by having system-specific containers with MPI stacks pre-installed. For example, my

[Runscript Documentation](http://singularity.lbl.gov/docs-run#defining-the-runscript)
### Your first container

Create an empty 1024 MB image and format it as an ext3 filesystem.

```
singularity create -s 1024 myimage.img
```

## Import an operating system

We are running ubuntu on our Jetstream instances, so lets import a centos system.

```
singularity import myimage.img docker://centos:latest
```

## Enter the container

We can enter the container and launch a shell with

```
singularity shell myimage.img
```

You can then interact with the container as you would a normal CLI. Exit your container, and lets do a couple sanity checks.

#### From Jetstream instance

```
[jetstream]$ whoami
[jetstream]$ cat /etc/*release
```
#### From Singularity shell

```
[jetstream]$ singularity shell myimage.img
[singularity]$ whoami
[singularity]$ cat /etc/*release
```

## Running Commands

You can also run commands in the container without entering it.

```
[jetstream]$ singularity exec myimage.img cat /etc/*release
```

While this location is inside your container, your $HOME directory gets passed into the container automatically.

```
[jetstream]$ echo "OUTSIDE CONTAINER" > ~/test.txt
[jetstream]$ singularity exec myimage.img cat $HOME/test.txt
```

## Modifying a container

By default, you cannot modify an image

```
[jetstream]$ singularity shell myimage.img
[singularity]$ touch /test.file
```

You can only modify a container with sudo, and specifying that you want to mount the container as writeable.

```
[jetstream]$ sudo singularity shell --writeable myimage.img
[singularity]$ touch /test.file
[singularity]$ exit
[jetstream]$ singularity exec myimage.img ls /
```

## Passing in filesystems

In normal versions of Singularity, you can specify which folders get mounted in a container when it is run. At TACC, we specifically limit that to our shared filesystems as a security precaution. This means you will need to always add

- /corral
- /home1
- /work
- /scratch

folders in your containers if you wish to interact with them like you did your `$HOME` folder.

## Community containers

We already know about pulling images like centos directly from docker, but there are whole communities like biocontainers for bioinformatics and singularity hub to deliver software. Lets try pulling qiime from biocontainers since it has a complicated install process.

```
[jetstream]$ singularity pull --size 2048 docker://quay.io/biocontainers/qiime:1.9.1--np112py27_1
[jetstream]$ singularity exec qiime-1.9.1--np112py27_1.img pick_de_novo_otus.py -h
```

Just make sure to

```
[jetstream]$ sudo singularity exec --writable qiime-1.9.1--np112py27_1.img mkdir /{work,scratch,home1,corral}
```

if you want to run this at TACC. Lets also transfer this image to Stampede2

```
[jetstream]$ scp qiime-1.9.1--np112py27_1.img username@stampede2.tacc.utexas.edu:
```

## Running containers at TACC

Singularity 2.3.1 is installed on Stampede2, but it can only be run from compute nodes.

```
[jetstream]$ ssh username@stampede2.tacc.utexas.edu
[stampede2]$ module load tacc-singularity
[stampede2]$ singularity -h
```

You should recieve a warning message because you are currently on a login node.

Login nodes are the nodes that all users land on when they ssh to a TACC system. These are designed to

- Handle network connections
- Stage data
- Schedule jobs

To ensure that all users have a responsive experience at TACC, we disabled Singularity on login nodes. This is because your container spawns many processes and utilizes loop devices when mounting the image. Luckily, TACC makes interactive compute sessions fairly accessible.

```
[jetstream]$ idev -m 30
[compute]$ singularity -h
```

You should now be able to run/exec/shell your the qiime container you transferred to Stampede2!

```
[compute]$ singularity exec qiime-1.9.1--np112py27_1.img pick_de_novo_otus.py -h
[compute]$ singularity shell qiime-1.9.1--np112py27_1.img
[singularity]$ ls /work
[singularity]$ ls /scratch
```

# Sinularity Bootstrap

Besides pulling pre-built docker images, you can build your own by writing a [definition file](http://singularity.lbl.gov/bootstrap-image) and bootstrapping (building) the image on your own.

A singularity file contains a header, which specifies the manager and base OS to build from.

## Docker header

```
Bootstrap: docker
From: ubuntu:latest
```

## Centos header

```
BootStrap: yum
OSVersion: 7
MirrorURL: http://mirror.centos.org/centos-%{OSVERSION}/%{OSVERSION}/os/$basearch/
Include: yum
```

## Definition sections

After you make your header, you just need to write the sections of your container.

- `%setup` - When you need to run commands and copy files into the container before `%post`
- `%post` - The actual setup commands
  - Making directories
  - yum/apt commands
  - git clone
  - make
- `%labels` - Any metadata you want associated with your container
  - NAME VALUE
- `%environment` - Environment values that are sources whenever using the container
  - NAME VALUE
- `%runscript` - This is what runs when you `singularity run` the container
  - Prefix the execution command with `exec`
- `%test` - A test to make sure the container was built correctly
  - Runs after `%post`
  - Run anytime using `singularity test`

## Example definition file

```
BootStrap: docker
From: fedora:latest

%post

yum -y install samtools BEDTools git

%runscript

echo "Arguments received: $*"
exec bedtools "$@"

%test

bv=$(bedtools --version)
if [ "$bv" == "bedtools v2.26.0" ]
then
    echo "PASS - $bv found"
else
    echo "FAIL - $bv not found"
fi

%files

%environment

MYVAR=cats

%labels

AUTHOR username
```

## Bootstrapping

After creating your image, you can bootstrap and run it as follows

```
singularity create -F -s 1024 bedtools.img
sudo singularity bootstrap bedtools.img bedtools.def
singularity test bedtools.img
singularity run bedtools.img intersect -h
singularity exec bedtools.img bedtools intersect -h
```
