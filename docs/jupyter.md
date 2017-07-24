# Welcome to Jupyter


This is an interactive module. Please follow along on your own computer. After taking this module, participants should be able to:

 * Understand how Jupyter Notebooks are utilized
 * Know how to navigate Notebooks
 * Create a Python Notebook
 * Learn the Basics of Coding in Python
 * Learn the Basics of Numpy
 * Do some Basic Plotting using Matplotlip
 
 
### What are Jupyter Notebooks

A web-based, interactive computing tool for capturing the whole computation process: developing, documenting, and executing code, as well as communicating the results.

<!--center><img src="../../resources/cloud.jpg" style="height:300px;"></center-->

### How do Jupyter Notebooks Work?

An open notebook has exactly one interactive session connected to a kernel which will execute code sent by the user and communicate back results. This kernel remains active if the web browser window is closed, and reopening the same notebook from the dashboard will reconnect the web application to the same kernel.

#### What's this mean?
Notebooks are an interface to kernel, the kernel executes your code and outputs back to you through the notebook. The kernel is essentially our programming language we wish to interface with.


### Jupyter Notebooks, Structure

* Code Cells
** Code cells allow you to enter and run code
** Run a code cell using Shift-Enter

* Markdown Cells
** Text can be added to Jupyter Notebooks using Markdown cells. Markdown is a popular markup language that is a superset of HTML.


### Jupyter Notebook, Formatting Markdown Cells

You can add headings:
```
# Heading 1
# Heading 2
## Heading 2.1
## Heading 2.2
```

You can add lists
```
1. First ordered list item
2. Another item
⋅⋅* Unordered sub-list. 
1. Actual numbers don't matter, just that it's a number
⋅⋅1. Ordered sub-list
4. And another item.
```

You can do HTML
```
<dl>
  <dt>Definition list</dt>
  <dd>Is something people use sometimes.</dd>

  <dt>Markdown in HTML</dt>
  <dd>Does *not* work **very** well. Use HTML <em>tags</em>.</dd>
</dl>
```
And even, Latex!
```
$e^{i\pi} + 1 = 0$
```

OSX/Linux:

```
 Open the application 'Terminal'
  # make sure your key file has the correct permissions:
  $ chmod 0600 chmod 0600 ~/Downloads/<key_name>.pem
  # connect to the VM over SSH
  $ ssh -i ~/Downloads/<key_name>.pem ubuntu@<IP address>
```

Windows:

```
 Open the application 'PuTTY'
  enter Host Name: <IP address>
  (click 'Open')
  (enter "ubuntu" username)
  (select your key)
```

If all goes well you should see a command prompt that looks something like:

```
Last login: Sat Jul 22 17:52:45 2017 from dhcp-146-6-176-22.tacc.utexas.edu
ubuntu@test-jfs1:~$ 
```
