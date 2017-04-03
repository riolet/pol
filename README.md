# poline

[![Build Status](https://jenkins-poline.hotbed.io/buildStatus/icon?job=poline-poline)](https://jenkins-poline.hotbed.io/job/poline-poline/)

poline lets you do awk-like one liners in python.

For example, the following will graph the number of connections for each hosts.


```bash
 netstat -an | grep ESTABLISHED | pol "|url(_4).hostname" "counter(_)" ":x, c: Cols(17,40).f(x,'*' * c)"
```

Example output:

```
216.58.193.78	    *****
74.125.199.189	    ****
198.252.206.25	    ***
74.125.199.188	    **
192.30.253.125	    **
127.0.0.1	    **
216.58.193.67	    *
```

The equivalent awk version can be found on [commandlinefu](http://www.commandlinefu.com/commands/view/2012/graph-of-connections-for-each-hosts).

# Installation

poline is compatible with Python 2.7 to Python 3.6.

You can easily install poline is via pip:

```
pip install poline
```

or

```
pip<version> install poline
```

Where `<version>` is the version of python you wish to use. For example, if you want to use poline with Python 3.6 installed, you should use

```
pip3.6 install poline
```

# Usage

```
pol [-h] [-F SEPARATOR] [-s] [-q] expression1 [expression2]

positional arguments:
  expression            python expression

optional arguments:
  -h, --help            show this help message and exit
  -F SEPARATOR, --separator SEPARATOR
                        split each line by SEPARATOR
  -s, --split           split each line
  -q, --quiet           don't implicitly print results

```

poline stores *stdin* in the variable *_* (underscore) in the form of a generator of strings.

You can see what's inside *_* with:

```bash
ls -la | pol "x for x in _"
'total 20'
'drwxr-xr-x  2 default root  104 Apr  2 23:34 .'
'drwxr-xr-x 10 default root  230 Apr  2 23:34 ..'
'-rw-r--r--  1 default root    0 Mar 31 14:33 __init__.py'
'-rw-r--r--  1 default root 1740 Apr  2 23:31 _com_collections.py'
'-rw-r--r--  1 default root 4239 Apr  2 23:33 core.py'
'-rw-r--r--  1 default root  750 Apr  2 23:34 fields.py'
'-rw-r--r--  1 default root 3387 Apr  2 23:31 utilfuncs.py'
```

If you call `pol` with `-s` option, you the `_` is returned in the form of a generator of lists.

```bash
ls -la | pol -s "pformat(x,width=100) for x in _"
['total', '20']
['drwxr-xr-x', '2', 'default', 'root', '104', 'Apr', '2', '23:34', '.']
['drwxr-xr-x', '10', 'default', 'root', '230', 'Apr', '2', '23:34', '..']
['-rw-r--r--', '1', 'default', 'root', '0', 'Mar', '31', '14:33', '__init__.py']
['-rw-r--r--', '1', 'default', 'root', '1740', 'Apr', '2', '23:31', '_com_collections.py']
['-rw-r--r--', '1', 'default', 'root', '4239', 'Apr', '2', '23:33', 'core.py']
['-rw-r--r--', '1', 'default', 'root', '750', 'Apr', '2', '23:34', 'fields.py']
['-rw-r--r--', '1', 'default', 'root', '3387', 'Apr', '2', '23:31', 'utilfuncs.py']
```

## Field class
Each list item is of the class `Field`, which is derived from `String`. `Field` has a number of useful addition methods:

| Method        | Explanation           |
| ------------- |:-------------:|
| i()      | returns int value of the field |
| f()      | returns int value of the field |
| h() | returns field as number of bytes in a human readable format. [See bytesize()](#bytesizex-u--none-s--false)|



## Chaining expressions

You can chain expressions as if you were chaining commands on bash using pipes.

```
netstat -an | grep ESTABLISHED | pol "|url(_4).hostname" "counter(_)" ":x, c: Cols(17,40,None).f(x, get([' '.join(l[1:]) for l in sh(['whois', x],s=T) if 'OrgName' in l[0]], 0), '*' * c)"
```

## Expression types

### Awk-like expressions
`|epxression`

`%FS%expression`

Works on the last result item by item (or line by line if the last result was ```stdin```)
The split() is called `|epxression` and split(FS) is called with `%FS%expression`.
The list generated by the split is available in `__`.
Individual fields are available as `_0`, `_1`, `_2` etc.

```bash
ps aux | pol "|Cols(10,None).f(_0,_10)"
USER      	COMMAND
default   	bash
default   	ps
default   	/tmp/poline/poline_venv/bin/python
```


### lambda-like expressions
`:*args:expression`
Works on the last result item by item (or line by line if the last result was ```stdin```)
Receives the n-tuple *args.

Example

```
ps aux | pol "skip(_)" "|(_0,_10)" ":user,cmd:Cols(50,None).f(next(sh('id',user)),cmd)"
uid=1001(default) gid=0(root) groups=0(root)      	bash
uid=1001(default) gid=0(root) groups=0(root)      	ps
uid=1001(default) gid=0(root) groups=0(root)      	/tmp/poline/poline_venv/bin/python
```

# Utility Functions

We are in the process of adding utility functions to poline. Contributions are most welcome.

## barchart(x, p = False, w = 10)

Returns the value *x* as a barchart. If *p* is true, x is intepreted as a percentage, with 100% being *w* wide.

```
df -B1 | pol "|Cols(20,10,10,10,5,None,10).f(_0,_1.h(),_2.h(),_3.h(),_4, barchart(_2.i()/_1.f(),p=True) if _1.isdigit() else ' '*10,_5)"
Filesystem          	1B-blocks 	Used      	Available 	Use% 	          	Mounted
/dev/mapper/docker-8	  9.99 G  	  8.06 G  	  1.93 G  	81%  	▓▓▓▓▓▓▓▓░░	/
tmpfs               	 31.37 G  	  0.00 B  	 31.37 G  	0%   	░░░░░░░░░░	/dev
tmpfs               	 31.37 G  	  0.00 B  	 31.37 G  	0%   	░░░░░░░░░░	/sys/fs/cg
/dev/sda3           	211.08 G  	 26.49 G  	173.84 G  	14%  	▓░░░░░░░░░	/etc/hosts
shm                 	 64.00 M  	  0.00 B  	 64.00 M  	0%   	░░░░░░░░░░	/dev/shm

```

## bytesize(x, u = None, s = False)

Returns the number of bytes *x* in a human readable string with a 'B', 'K', 'M', 'G', 'T', 'P' prefix.

If x is a string, bytesize tries to convert it to a float, or returns the string as is.

*u* specifies that bytes are in a different unit, for example u='K' if the size is in 1-K blocks.

*s* specifies that byte size should be strict.

Example:

```
$ pol "bytesize(972693249)"
927.63 M
```

## counter(l, n=10)

Sorts a list by descending order, and returns a 2-tuple with element and number of appearances in l:

```
[(<element>, <count>), ...]
```

Example:

```
$ pol -q "pprint(counter('information'), width=20)"
[('i', 2),
 ('n', 2),
 ('o', 2),
 ('f', 1),
 ('r', 1),
 ('m', 1),
 ('a', 1),
 ('t', 1)]
```

## get (l, i, d=None)

get *i*th element from list *l* if the *i*th element exists, or return value d

```python
>>> get([1, 2, 3, 4],1)
2
>>> get([1, 2, 3, 4],4,0)
0
```


## sh (*args, **kwargs)

Executes shell command and arguments specified in *args*, returns stdout in the form of a list of lists.

Recognized kwargs:
F SEPARATOR     split each line by SEPARATOR
s True|False    split each line

Example:

The following displays the inode of each file using *stat*

Python>=3.6
```
$ ls | pol "f'{l:20.20}\t%s' % [i[3] for i in sh(['stat',l]) if 'Inode:' in i[2]][0] for l in _"
```

Python>=2.7
```
$ ls | pol "[columns(20,None).format(l,i[3]) for i in sh('stat',l,s=True) if 'Inode:' in i[2]][0] for l in _"
```

```
LICENSE   	360621
Makefile  	360653
pol       	360606
pol.c     	360637
pol.o     	360599
README.md 	360623
```

**N.B.** The syntax of *sh* has changed, but old syntax is still supported for the sake of backward compatibility.

As well, the popular shell commands *cp*, *df*, *docker*, *du*, *find*, *git*, *history*, *ln*, *ls*, *lsof*, *mv*, *netstat*, *nmcli*, *ps*, *rm* are all functions. For example

They behave as if the command is being passed to *sh* above.

```
pol "ls('-lah')"
```

```
total 20K
drwxr-xr-x  2 default root  104 Apr  2 23:34 .
drwxr-xr-x 10 default root  230 Apr  2 23:34 ..
-rw-r--r--  1 default root    0 Mar 31 14:33 __init__.py
-rw-r--r--  1 default root 1.7K Apr  2 23:31 _com_collections.py
-rw-r--r--  1 default root 4.2K Apr  2 23:33 core.py
-rw-r--r--  1 default root  750 Apr  2 23:34 fields.py
-rw-r--r--  1 default root 3.4K Apr  2 23:31 utilfuncs.py
```

## skip(iter, n=1)

Skip the first ```n``` items from the Iterator ```iter```.

```
pol "ls('-lah')" "skip(_,n=3)"
```

```
-rw-r--r--  1 default root    0 Mar 31 14:33 __init__.py
-rw-r--r--  1 default root 1.7K Apr  2 23:31 _com_collections.py
-rw-r--r--  1 default root 4.2K Apr  2 23:33 core.py
-rw-r--r--  1 default root  750 Apr  2 23:34 fields.py
-rw-r--r--  1 default root 3.4K Apr  2 23:31 utilfuncs.py
```


# Utility classes

## Cols

Cols is used to manage formatted output in the form of columns

### `__init__(*args, **kwargs)`
The constructor generators columns by unpacking *args. For example:
```Cols(10,10,20,None)``` generates a format string as ```'{10.10}\t{10.10}\t{20.20}\t{}'```

### `f(*args, **kwargs)`
Unpacks ```*args``` to pass to ```String.format()```. If format string has not been set, sets the format string as `{}\t{}`

# Examples

#### The top ten commands you use most often
```
history | pol "|_1" "counter(_)" ":x,c:Cols(5,None).f(x,c)"
```

