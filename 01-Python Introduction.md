# 01.Python Introduction

**What is Python?**

Python is a popular programming language. It was created by Guido van
Rossum, and released in 1991.

It is used for:

- web development (server-side),

- software development, ● mathematics,

- system scripting.

**What can Python do?**

- Python can be used on a server to create web applications.

- Python can be used alongside software to create workflows.

- Python can connect to database systems. It can also read and modify
  files.

- Python can be used to handle big data and perform complex mathematics.

- Python can be used for rapid prototyping, or for production-ready
  software development.

**Why Python?**

- Python works on different platforms (Windows, Mac, Linux, Raspberry
  Pi, etc).

- Python has a simple syntax similar to the English language.

- Python has a syntax that allows developers to write programs with
  fewer lines than some other programming languages.

- Python runs on an interpreter system, meaning that code can be
  executed as soon as it is written. This means that prototyping can be
  very quick.

- Python can be treated in a procedural way, an object-oriented way, or
  a functional way.

**Good to know**

- The most recent major version of Python is Python 3, which we shall be
  using in this tutorial. However, Python 2, although not being updated
  with anything other than security updates, is still quite popular.

- In this tutorial, Python will be written in a text editor. It is
  possible to write Python in an Integrated Development Environment,
  such as Thonny, Pycharm, Netbeans, or Eclipse which are particularly
  useful when managing larger collections of Python files.

**Python Syntax compared to other programming languages**

- Python was designed for readability and has some similarities to the
  English language with influence from mathematics.

- Python uses new lines to complete a command, as opposed to other
  programming languages which often use semicolons or parentheses.

- Python relies on indentation, using whitespace, to define scope; such
  as the scope of loops, functions, and classes. Other programming
  languages often use curly brackets for this purpose.

**Example**

## print(\"Hello, World!\")

**Definition and Usage**

The print() function prints the specified message to the screen or
another standard output device.

The message can be a string or any other object, the object will be
converted into a string before written to the screen.

**Syntax**

print*(object(s)*, sep=*separator*, end=*end*, file=*file*,
flush=*flush*)

**Parameter Values**

  ------------------------------------------------------------------------------
  **Parameter**         **Description**
  --------------------- --------------------------------------------------------
  *object(s)*           Any object, and as many as you like. Will be converted
                        to a string before printed

  sep=\'*separator*\'   Optional. Specify how to separate the objects, if there
                        is more than one. Default is \' \'

  end=\'*end*\'         Optional. Specify what to print at the end. Default is
                        \'\\n\' (line feed)

  *file*                Optional. An object with a write method. Default is
                        sys.stdout

  *flush*               Optional. A Boolean, specifying if the output is flushed
                        (True) or buffered (False). Default is False
  ------------------------------------------------------------------------------
