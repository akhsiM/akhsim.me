---
title: Python - Context Manager
layout: post-toc
tags: python programming
---
## General

Context managers allow us to properly manage and allocate resources when we want to. The most widely used example of context managers is the `with` statement.

In Python, the recommended way of working with a file object is by using Context Manager.

## Building our own context manager

There are a couple of ways to write out our own context manager:
- Using a class
- Using a function with a decorator

In the below section, we are going to be writing our own context manager using class and function decorator. IRL, this is not needed because Python already comes with `open()` which itself is a context manager function.

### Using a class

First, we'll start with using a class:

```python
class Open_File():

	def __init__(self, filename, mode):
		self.filename = filename
		self.mode = mode

	# set up opening a resource and return it
	# called at the start of 'with' block
	def __enter__(self):
		self.file = open(self.filename, self.mode)
		return self.file

	# make sure the resource gets closed
	# called at the end of 'with' block
	def __exit__(self, exc_type, exc_val, traceback):
		self.file.close()
		

with Open_File('sample.txt', 'w') as f:
	f.write('Testing')

print(f.closed) # Making sure that the resouce has been closed.
```

```
: True
```

Walk through:
	- When the `Open_File()` object is instantiated, the `__init__` method is called, to set the object's attributes.
	- Because the `with` statement is called, the `__enter__` method is called, returning the resource variable. `f` is set to the file object (`return self.file`)
	- Within the context manager, we can do whatever with the file object `f`.
	- Whenever we exit out of the `with` block, i.e the context manager, the `__exit__` method is called.


### Using a function

We'll need the `contextlib` module.

```python
from contextlib import contextmanager

@contextmanager
def open_file(file, mode):
	f = open(file, mode)
	print(f'Before \'with\' body.., setting variable: {f}')

	yield f

	print('after \'with\' body..')
	f.close()

with open_file('sample.txt', 'w') as f:
	f.write('Hello world!')
	print('During \'with\' body')

print(f.closed)

```

```
: Before 'with' body.., setting variable: <_io.TextIOWrapper name='sample.txt' mode='w' encoding='UTF-8'>
: During 'with' body
: after 'with' body..
: True

Looking at the `yield` statement:
- Everything before the `yield` statement is equivalent to the `__enter__` method of the class. 
- The `yield` statement is where the code within the `with` statement is going to run.
- Everything after the `yield` statement is the `__exit__` method of the class
```

### Exception handling

We should be putting our context manager in a try block:

```python
from contextlib import contextmanager

@contextmanager
def open_file(file, mode):
	try:
		f = open(file, mode)
		print(f'Before \'with\' body.., setting variable: {f}')
		yield f
	finally:
		print('after \'with\' body..')
		f.close()

with open_file('sample.txt', 'w') as f:
	f.write('Hello world!')
	print('During \'with\' body')

print(f.closed)
```

```
: Before 'with' body.., setting variable: <_io.TextIOWrapper name='sample.txt' mode='w' encoding='UTF-8'>
: During 'with' body
: after 'with' body..
: True
```

Why? If there is any issue with openning the file, the teardown code will still be executed, making sure that the resource is closed properly.
# IRL
Let's say we are working on a project that involves moving between different directories and list the contents within multiple times.

```python
import os

cwd = os.getcwd()  # save current dir in memory
os.chdir('sample dir')  # cd to 'sample dir'
print(os.listdir())  # do a ls -l
os.chdir(cwd)  # change back to previous cwd


cwd = os.getcwd() 
os.chdir('sample dir') 
print(os.listdir())  
os.chdir(cwd) 

cwd = os.getcwd()  
os.chdir('sample dir')  
print(os.listdir())  
os.chdir(cwd) 

```

#+RESULTS:
: ['samplefile.txt']
: ['samplefile.txt']
: ['samplefile.txt']

Analysis:
- Line 3 and 4 are pretty much setting up for the work that needs to be done.
- Switching back (line 6) is pretty much the teardown.

>> This is a good candidate to be using a context manager.

```python
import os
from contextlib import contextmanager

@contextmanager
def ch_dir_temp(destination):
	try:
		cwd = os.getcwd()
		print(f'Moving to {destination}..')
		os.chdir(destination)
		yield 
	finally:
		os.chdir(cwd)
		print(f'Moved back to {os.getcwd()}')
	

with ch_dir_temp('sample dir'):
	print(os.listdir())


with ch_dir_temp('sample dir'):
	print(os.listdir())


with ch_dir_temp('sample dir'):
	print(os.listdir())
```

```
: Moving to sample dir..
: ['samplefile.txt']
: Moved back to /home/kkennynguyen/pCloudDrive/Documents/OrgNotes/Learning
: Moving to sample dir..
: ['samplefile.txt']
: Moved back to /home/kkennynguyen/pCloudDrive/Documents/OrgNotes/Learning
: Moving to sample dir..
: ['samplefile.txt']
: Moved back to /home/kkennynguyen/pCloudDrive/Documents/OrgNotes/Learning
```

Explanation: We are not working within any resource object so we can just put a `yield` without any variable.
