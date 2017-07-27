# Image: tacc/csc_jupyter
from ubuntu:16.04

# update and install basic packages
RUN apt-get update && apt-get install -y wget

# download the Anaconda installer and run it
RUN wget --quiet https://repo.continuum.io/archive/Anaconda3-4.4.0-Linux-x86_64.sh -O /root/Anaconda3-4.4.0-Linux-x86_64.sh 

# install required gnu utils
RUN apt-get install -y wget bzip2 ca-certificates libglib2.0-0 libxext6 libsm6 libxrender1 git mercurial subversion

# run the installer
RUN /bin/bash /root/Anaconda3-4.4.0-Linux-x86_64.sh -b

# add to the PATH
RUN echo 'export PATH=/root/anaconda3/bin:$PATH' > /root/.bash_profile

# run jupyter config gen 
RUN /root/anaconda3/bin/jupyter notebook --generate-config --allow-root

# generate self-signed cert
RUN openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /root/mykey.key -out /root/mycert.pem -batch

# install additional packages
RUN apt-get install -y vim crudini

# update jupyter config
RUN crudini --set /root/.jupyter/jupyter_notebook_config.py '' c.NotebookApp.certfile \'/root/mycert.pem\'

RUN crudini --set /root/.jupyter/jupyter_notebook_config.py '' c.NotebookApp.ip \'*\'

RUN crudini --set /root/.jupyter/jupyter_notebook_config.py '' c.NotebookApp.keyfile \'/root/mykey.key\'

RUN crudini --set /root/.jupyter/jupyter_notebook_config.py '' c.NotebookApp.notebook_dir \'/root/notebook\'

RUN crudini --set /root/.jupyter/jupyter_notebook_config.py '' c.NotebookApp.open_browser False

RUN crudini --set /root/.jupyter/jupyter_notebook_config.py '' c.NotebookApp.port 8887

RUN crudini --set /root/.jupyter/jupyter_notebook_config.py '' c.NotebookApp.password \'\'

# create notebook dir
RUN mkdir -p /root/notebook

# 
CMD ["/root/anaconda3/bin/jupyter", "notebook", "--allow-root"]
