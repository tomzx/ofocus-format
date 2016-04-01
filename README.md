# Omnifocus `.ofocus` file format

**For v2 `.ofocus` file format, please check the [2.0 branch](https://github.com/tomzx/ofocus-format/tree/2.0).**

This document is written by reverse engineering an Omnifocus 1.10 `.ofocus` file.

## Overview

The `.ofocus` file is basically a directory with a set of `.zip` files.
Each `.zip` file contains a `contents.xml` file, which is a transaction (a set of data) in `XML` format.

There is a master file named `00000000000000={randomId}+{randomId}.zip` and multiple files named `{date in GMT}={randomID}+{randomID}.zip` which are transactions files that apply over the master file. The `randomId` values have the same format as the random ids used within the file (see `id` in the [Formats](#formats) section). 

To build the history, one will start with the master file and read the chain of transactions files, something that will look like the following:

```
00000000000000={randomId}+{A}.zip (master file)
{date in GMT}={A}+{B}.zip (transaction file 1)
{date in GMT}={B}+{C}.zip (transaction file 2)
{date in GMT}={C}+{D}.zip  (transaction file 3)
...
{date in GMT}={Y}+{Z}.zip  (transaction file n)
```

## History merge

Whenever you sync with the server, a file with the format `{date in GMT}={X}+{Y}+{Z}.zip`, where `X` is the  `randomId` of the oldest transaction file and `Y` is the `randomId` of the youngest transaction file, will be generated. This file does not contain any transaction in itself and only serves to indicate the junction of two disjoint history branches.

**Oldest transaction file**
{date in GMT}={A}+{X}.zip

**Youngest transaction file**
{date in GMT}={B}+{Y}.zip

**Resulting transaction file**
Because `date in GMT of the oldest` < `date in GMT of the youngest`,
{date in GMT}={X}+{Y}+{Z}.zip

Any transaction file generated next will henceforth be named `{date in GMT}={Z}+{C}.zip`, where `Z` is the `randomId` generated for the resulting transaction file.

My impression is that conflict resolution is simply handled by taking the latest change that occured to a task and consider it the source of truth. If both my iPhone and my Desktop have differences on a task status/name, the changes that will have been done last will be the one preserved, which makes sense in the context of time.

## History compression

Omnifocus tracks clients by using files with the format `{date in GMT}={randomId}.client`. Such files contain a plist with a dictionary in it with various properties of the client device such as the number of cpu, the OS version, an identifier of the Omnifocus client version used as well as one very important piece of information: an array of `tailIdentifiers`. As one can deduce from the name, it tracks the last synced transaction file, indicated by the second `randomId` of the transaction file's name.

Note that it is likely that you will see multiple client files for your device but with different date timestamp. When looking for the most up to date data, make sure to use the latest client file available.

Whenever Omnifocus realizes that all known clients are synchronized up to a certain transaction file, it is free to compress the transaction history back into the master file.

### Hypothetical scenario

```
00000000000000={randomId}+{A}.zip (master file)
{date in GMT}={A}+{B}.zip (transaction file 1)
{date in GMT}={B}+{C}.zip (transaction file 2)
{date in GMT}={C}+{D}.zip  (transaction file 3)
```

... Compression ...

```
00000000000000={randomId}+{D}.zip (master file, with transaction files 1, 2, 3 merged in)
```

## File content

### Master file

The content of each `contents.xml` is as follow:

Legend
(...) Attributes
*Optional*

* omnifocus
	* setting(id : id)
 		* *added(order : unsigned int) : datetime*
		* *modified : datetime*
		* plist(version : string) : string/xml
	* context(id : id)
		* added(order : unsigned int) : datetime
		* *modified : datetime*
		* name : string
		* rank : signed int
		* *context(idref : id)* - reference to parent context
		* *location(name : string, latitude : string, longitude : string)* - latitute and longitude are signed decimal with a precision of 4
	* folder(id : id)
		* *folder(idref : id)* - reference to parent folder
		* added : datetime
		* *modified : datetime*
		* name : string
		* rank : signed int
		* *hidden : bool* (not hidden by default)
	* task(id : id) - has either a project or task child. A project child declares the project's properties while a task child declares the parent of the current task
		* *project* - declares a project/single-action list
			* folder(idref : id) - reference to parent folder
			* last-review : datetime
			* *next-review : datetime*
			* review-interval : string (interval format)
			* *status : string (active, inactive, done, dropped) (if not specified, active by default)*
			* *singleton : bool (if not specified, false by default)* - indicate if the project is composed of single actions
		* *task(idref : id)* - reference to parent task
		* *inbox* - indicate if this task is part of the inbox
		* *note*
			* *text* (formatted as HTML)
		* added(order : unsigned int) : datetime
		* *modified : datetime*
		* name : string
		* rank : signed int (relative to parent task?)
		* *context(idref : id)* - reference to context
		* *due : datetime*
		* *start : datetime*
		* *completed : datetime*
		* *estimated-minutes : unsigned int*
		* *flagged : bool*
		* *order : string (parallel) (if not specified, sequential by default)*
		* *repeat : string (interval format)*
		* *repetition-rule : string (FREQ=MONTHLY, FREQ=WEEKLY;INTERVAL=4)*
		* *repetition-method : string (fixed)*
	* perspective(id : string/id) - either a predefined string (ProcessInbox, ProcessProjects, ProcessContexts, ProcessDueSoon, ProcessFlagged, ProcessReview, ProcessCompleted) or an id (see below)
		* plist(version : string) : string/xml

### Transaction xml files

* omnifocus
	* element(op="reference|add|update|delete") (not specifying an op = add)

A `op="reference"` is basically a copy of the referenced data within the transaction (in case something were to happen to the original reference).

Any `id` referred to in the transaction file will be included as part of the transaction with an `op="reference"` attribute.

### Formats

The `id` used as attribute is 11 characters long, in what appears to be base 64 (`A-Za-z0-9\_\-`).

The `datetime` is formatted according to [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) with precision to the milliseconds (`2014-11-03T10:14:09.123Z`).

`Interval format` is @periodduration (@1d, @1w, @1m, @1y) 

The `rank` appears to be a global value shared by all data types. It is a signed integer (32 bits). There can be duplicate ranks. It would appear that ranks might only be unique within the same level of the hierarchy in order to provide ordering.

## Reference

* [omnifocus-viewer Omnifocus File Format](https://code.google.com/p/omnifocus-viewer/wiki/WikiPageMain)
* Omnifocus forum, Omnifocus Data Storage thread [part 1](http://forums.omnigroup.com/showpost.php?p=13772), [part 2](http://forums.omnigroup.com/showpost.php?p=13786)
* Omnifocus blog, [OmniFocus: What We've Learned So Far (Engineering)](https://www.omnigroup.com/blog/OmniFocus_What_Weve_Learned_So_Far_Engineering)
