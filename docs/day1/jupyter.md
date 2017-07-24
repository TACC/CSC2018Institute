# Crash Course in Python and Matplotlib using Jupyter


This is an interactive module. Please follow along on your own computer. After taking this module, participants should be able to:

 * Understand how Jupyter Notebooks are utilized
 * Know how to navigate Notebooks
 * Create a Python Notebook
 * Learn the Basics of Coding in Python
 * Learn the Basics of Numpy
 * Do some Basic Plotting using Matplotlip
 
## Introduction to Jupyter

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

#### Exercise 2
```
set a variable to your name.
set a variable to your age.
set a variable to the current year.

print out your name, print out the year your age will be twice your current age
```

#### Reading from the Keyboard
Let's try the following:
```
var = input("Please enter a number: ")
```

Now:
```
var2 = input("Please enter a string: ")
```
Put in <i>Hello</i> as your input

#### Fomatted Output
Making the output prettier
```
print "The number that you wrote was : ", var
print "The number that you wrote was : %d" % var

print "the string you entered was: ", var2
print "the string you entered was: %s" % var2
```

FYI: 
```
\n  for a new line
\t  to insert a tab
%f  for floats
```

#### Writing to a File
In a Code Cell:
```
my_file = open("output_file.txt",'w')
vars = "This is a string\n"
my_file.write(vars)
var3 = 10
my_file.write("\n")
my_file.write(str(var3))
var4 = 20.0
my_file.write("\n")
my_file.write(str(var4))
my_file.close()
```

#### Reading from a File
In a Code Cell:
```
my_file = open(“output_file.txt”,’r’)
content = my_file.read()
print content
my_file.close()
```

When opening a file, you need to decide <i>how</i> you want to open it:
Just read?
Are you going to write to the file?
If the file already exists, what do you want to do with it?

```
r read only (default)
w write mode: file will be overwritten if it already exists
a append mode: data will be appended to the existing file
```

##### Reading it Line By Line
```
my_file = open("output_file.txt",'r')
vars = my_file.readline()
var5 = my_file.readline()
var6 = my_file.readline()
print "String: ", vars
print "Integer: ", var1
print "Float: ", var2
my_file.close()
```

##### Reading it Line By Line, a bit better format
<b>Remember the Magic Number?</b>
```
with open("output_file.txt",'r') as f:
    vars = f.readline()
    var5 = f.readline()
    var6 = f.readline()
    print "String: ", vars
    print "Integer: ", var1
    print "Float: ", var2
```

#### Control Flow

* So far we have been writing instruction after instruction
* Every instruction is executed
* What happens if we want to have instructions that are only executed if a given condition is true?


#### if/else/elif

The if/else construction allows you to define conditions in your program
<b>Remember the Magic Number?</b>
```
    if conditionA:
        statementA
    elif conditionB:
        statementB
    else:
        statementD
    
    this line will always be executed (after the if/else)
```
conditions are a datatype known as booleans, they can only be true or false


#### Booleans

In a Code Cell:
```
a = 2
b = 5

a>b
a<b
a == b
a != b
b>a or a==b
b?a and a==b
```

#### if/else/elif an Example

```
if var>10:
    print "You entered a number greater than 10"
else:
    print "you entered a number less than 10"

```

#### Nesting if statements together

```
if condition1:
    statement1
    if condition2:
        statement2
    else:
        if condition3:
            statement3 # when is this statement executed?
else:  # which ‘if’ does this ‘else’ belong to?
    statement4  # when is this statement executed?
```

#### Exercise 3
enter a number from the keyboard into a variable.

using type casting (or the Modulus function) and if statements, determine if the number is even or odd


#### for loops

When we need to iterate, execute the same set of instructions over and over again… we need to loop!  Introducing range()

(remember the MAGIC NUMBER!   Hint: it's 4)
```
for x in range(0, 3):
    print "Let's go %d" % (x)
```

#### nesting for loops

```
for x in range(0, 3):
    for y in range(0,5):
       print "Let's go %d %d" % (x,y)
```

#### Exercise 4
Let's try something a bit more challenging

using nested for-loops and nested if statements, write a program that loops from 3 to 100 and print out the number if it is *not* a prime number. 


Here's a hint to get you started:
```
for n in range(3,101):
    for q in range(2,101):
```


#### While Loops
Sometimes we need to loop while a condition is true...


(remember the MAGIC NUMBER!   Hint: it's 4)
```
i = 0    # Initialization
while (i < 10):    # Condition
    print i    # do_something
    i = i + 1 # Why do we need this?
```

#### Lists

* A list is a sequence, where each element is assigned a position (index)
* First position is 0. You can access each position using []
* Elements in the list can be of different type
```
mylist1 = [“first item”, “second item”]
mylist2 = [1, 2, 3, 4]
mylist3 = [“first”, “second”, 3]
print mylist1[0], mylist1[1]
print mylist2[0]
print mylist3
print mylist3[0], mylist3[1], mylist3[2]
print mylist2[0] + mylist3[2]
```

* We can also <i>slice</i> our data:
```
    print mylist3[0:3]
    print mylist3
```
* To change the value of an element in a list, simply assign it a new value:
```
    mylist3[0] = 10
    print mylist3
```

* There’s a function that returns the number of elements in a list
```
    len(mylist2)
```
* Check if a value exists in a list:
```
    1 in mylist2
```
* Delete an element
```
    len(mylist2)
    del mylist2[0]
    print mylist2
```
* Iterate over the elements of a list:
```
      for x in mylist2:
          print x
```

#### Exercise 5

Building from the previous Exercise, let's use lists to build a list of non primes, and then using 'in' build a list of prime numbers

#### Exercise 6

create a 3 lists:
one list, x,  holding numbers going from 0 to 2*pi, in steps of .01
one list, y1, holding x*x
one list, y2, holding x*x*x

write these out to a file with the format:
x, y1, y2

#### Other cool things you can do with Lists

There are more functions
```
    max(mylist), min(mylist) 
```
It’s possible to add new elements to a list:
```
    my_list.append(new_item)
```
We know how to find if an element exists, but there’s a way to return the position of that element:
```
   my_list.index(item)
```
Or how many times a given item appears in the list:
```
    my_list.count(item)
```

### Introduction to Matplotlib

#### What is Matplotlib?
It’s a graphing library for Python. It has a nice collection of tools that you can use to create anything from simple graphs, to scatter plots, to 3D graphs. It is used heavily in the scientific Python community for data visualisation.

#### Matplotlib, First steps
* Let's plot a simple sin wave from 0 to 2 pi.
** First lets, get our code started by importing the necessary modules.
```
%matplotlib inline
import matplotlib.pyplot as plt
import numpy as np
```
** Let's add the following lines, we're setting up x as an array of 50 elements going from 0 to 2*pi
```
x = np.linspace(0, 2 * np.pi, 50)
plt.plot(x, np.sin(x)) 
plt.show() # Show the graph.
```

** Let's plot another curve on the same axis
```
plt.plot(x, np.sin(x), x, np.sin(2 * x))
plt.show()
```

** Let's see if we can make the plots easier to read
```
plt.plot(x, np.sin(x), 'r-o', x, np.cos(x), 'g--')
plt.show()
```

** Colors:
```
Blue – ‘b’
Green – ‘g’
Red – ‘r’
Cyan – ‘c’
Magenta – ‘m’
Yellow – ‘y’
Black – ‘k’ (‘b’ is taken by blue so the last letter is used)
White  – ‘w’
```

** Lines and Common Markers
Lines:
```
Solid Line – ‘-‘
Dashed – ‘–‘
Dotted – ‘.’
Dash-dotted – ‘-:’
```
Often Used Markers:
```
Point – ‘.’
Pixel – ‘,’
Circle – ‘o’
Square – ‘s’
Triangle – ‘^’
```


#### Subplots
```
plt.subplot(2, 1, 1) # (row, column, active area)
plt.plot(x, np.sin(x), 'r')
plt.subplot(2, 1, 2)
plt.plot(x, np.cos(x), 'g')
plt.show()
```

using the subplot() function, we can plot two graphs at the same time within the same "canvas". Think of the subplots as "tables", each subplot is set with the number of rows, the number of columns, and the active area, the active areas are numbered left to right, then up to down.

#### Scatter Plots
```
y = np.sin(x)
plt.scatter(x,y)
plt.show()
```

call the scatter() function and pass it two arrays of x and y coordinates.

#### Adding some color
```
x = np.random.rand(1000)
y = np.random.rand(1000)
size = np.random.rand(1000) * 50
color = np.random.rand(1000)
plt.scatter(x, y, size, color)
plt.colorbar()
plt.show()
```

Let's see what we did:
```
...
plt.scatter(x, y, size, color)
plt.colorbar()
...
```
We brought in two new parameters, size and color. Which will varies the diameter and the color of our points. Then adding the colorbar() gives us nice color legend to the side.

#### Histograms
A histogram is one of the simplest types of graphs to plot in Matplotlib. All you need to do is pass the hist() function an array of data. The second argument specifies the amount of bins to use. Bins are intervals of values that our data will fall into. The more bins, the more bars.
```
plt.hist(x, 50)
plt.show()
```

#### Adding Labels and Legends
```
x = np.linspace(0, 2 * np.pi, 50)
plt.plot(x, np.sin(x), 'r-x', label='Sin(x)')
plt.plot(x, np.cos(x), 'g-^', label='Cos(x)')
plt.legend() # Display the legend.
plt.xlabel('Rads') # Add a label to the x-axis.
plt.ylabel('Amplitude') # Add a label to the y-axis.
plt.title('Sin and Cos Waves') # Add a graph title.
plt.show()
```

