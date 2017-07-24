# Welcome to Jupyter


This is an interactive module. Please follow along on your own computer. After taking this module, participants should be able to:

 * Understand how Jupyter Notebooks are utilized
 * Know how to navigate Notebooks
 * Create a Python Notebook
 * Learn the Basics of Coding in Python
 * Learn the Basics of Numpy
 * Do some Basic Plotting using Matplotlip
 
## Jupyter Basics

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


### Jupyter Notebooks, Workflow

Typically, you will work on a computational problem in pieces, organizing related ideas into cells and moving forward once previous parts work correctly. This is much more convenient for interactive exploration than breaking up a computation into scripts that must be executed together, as was previously necessary, especially if parts of them take a long time to run.

Let a traditional paper lab notebook be your guide:
* Each notebook keeps a historical (and dated) record of the analysis as it’s being explored.
* The notebook is not meant to be anything other than a place for experimentation and development.
* Notebooks can be split when they get too long.
* Notebooks can be split by topic, if it makes sense.

### Jupyter Notebooks, Navigating and Shortcuts
* Shift-Enter: run cell
** Execute the current cell, show output (if any), and jump to the next cell below. If Shift-Enteris invoked on the last cell, a new code cell will also be created. Note that in the notebook, typing Enter on its own never forces execution, but rather just inserts a new line in the current cell. Shift-Enter is equivalent to clicking the Cell | Run menu item.

* Ctrl-Enter: run cell in-place
** Execute the current cell as if it were in “terminal mode”, where any output is shown, but the cursor remains in the current cell. The cell’s entire contents are selected after execution, so you can just start typing and only the new input will be in the cell. This is convenient for doing quick experiments in place, or for querying things like filesystem content, without needing to create additional cells that you may not want to be saved in the notebook.

* Alt-Enter: run cell, insert below
** Executes the current cell, shows the output, and inserts a new cell between the current cell and the cell below (if one exists). (shortcut for the sequence Shift-Enter,Ctrl-m a. (Ctrl-m a adds a new cell above the current one.))

* Esc and Enter: Command mode and edit mode
** In command mode, you can easily navigate around the notebook using keyboard shortcuts. In edit mode, you can edit text in cells.

## Let's Start Coding

### Introduction to Python

Remember: The magic number is 4!

* Hello World!
* Data types
* Variables
* Arithmetic operations
* Relational operations
* Input/Output
* Control Flow
* More Data Types!
* Matplotlib

#### Hello World
Let's type the following into a Code Cell:
```
print “Hello World!”
```
Hit Ctrl-Enter

#### Some more quick examples
Let's type the following into another Code Cell:
```
print 5
print 1+1
```
Hit Ctrl-Enter

#### Variables

* You will need to store data into variables
* You can use those variables later on
* You can perform operations with those variables
* Variables are declared with a <b>name</b>, followed by ‘=‘ and a <b>value</b>
** An integer, string,…
** When declaring a variable, <b>capitalization</b> is important: 
*** ‘A’ <> ‘a’

#### Variables by Example

In a Code Cell:
```
five = 5
one = 1
print five
print one + one
message = “This is a string”
print message
```
Note: we are not <i>typing</i> out variable, we are only setting them and allowing Python to determine the <i>type</i> for us

#### Exercise 1

set a variable to your name.
set a variable to your age.
set a variable to the current year.

print out your name, and age

#### Data Types
In a Code Cell:
```
integer_variable = 100
floating_point_variable = 100.0
string_variable = “Name”
```

Just because we're not <i>typing</i> our data, doesn't mean that our data doesn't have a <i>type</i>

* Variables have a type

* You can check the type of a variable by using the type() function:
```
print type(integer_variable)
```

* It is also possible to change the type of some basic types:
```
str(int/float): converts an integer/float to a string
int(str): converts a string to an integer
float(str): converts a string to a float
```

Be careful: you can only convert data that actually makes sense to be transformed

#### Arithmetic Operations
```
+    Addition      1 + 1 = 2
-    Subtraction   5 – 3 = 2
/    Division      4 / 2 = 2
%    Modulo      5 % 2 = 1
*    Multiplication    5 * 2 = 10
//   Floor division    5 // 2 = 2
**   To the power of 2 ** 3 = 8
```

#### Arithmatic Operations by Example

In a Code Cell:
```
print 5/2
print 5.0/2
print "hello" + "world"
print "some" + 1
print "number" * 5
print 3+5*2
```

#### Data Conversions

```
number1 = 5.0/2
number2 = 5/2
```
what type() are they?
```
type(number1)
type(number2)
```
now, convert number2 to an integer:
```
int(number2)
```

#### Excercise 2
```
set a variable to your name.
set a variable to your age.
set a variable to the current year.

print out your name, print out the year your age will be twice your current age
```

