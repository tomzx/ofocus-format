# Omnifocus `.ofocus` file format

This document is written by reverse engineering an Omnifocus 1.10 `.ofocus` file. I have not yet looked into the format of the v2, so if you do, please share!

## Overview

The `.ofocus` file is a basically a directory with a set of `.zip` files.
Each `zip` file contains a `contents.xml` file, which is a transaction in `XML` format.

There is a master file named `00000000000000={uuid}+{randomId}.zip` and multiple files named `{date in GMT}={randomID}+{randomID}.zip` which are transactions files that apply over the master file.

To build the history, one will start with the master file and read the chain of transactions files, something that will look like the following:

```
00000000000000={uuid}+A.zip (master file)
{date in GMT}={A}+{B}.zip (transaction file 1)
{date in GMT}={B}+{C}.zip (transaction file 2)
{date in GMT}={C}+{D}.zip  (transaction file 3)
...
{date in GMT}={Y}+{Z}.zip  (transaction file n)
```

## File content

### Master file

The content of each `contents.xml` is as follow:

Legend
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
		* *location(name : string, latitude : string, longitude : string)*
	* folder(id : id)
		* *folder(idref : id)* - reference to parent folder
		* added : datetime
		* *modified : datetime*
		* name : string
		* rank : signed int
		* *hidden : bool*
	* task(id : id) - has either a project or task child which represents this task parent
		* *inbox* - indicate if this task is part of the inbox
		* *project*
			* folder(idref : id) - reference to parent folder
			* last-review : datetime
			* *next-review : datetime*
			* review-interval : string (interval format)
			* *status : string (done, inactive, dropped)*
			* *singleton : bool (if not specified, false by default)*
		* *task(idref : id)* - reference to parent task
		* *note*
			* *text* (formatted as HTML)
		* added(order : unsigned int) : datetime
		* *modified : datetime*
		* name : string
		* rank : unsigned int (relative to parent task?)
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
	* perspective(id)
		* plist(version : string) : string/xml

### Transaction xml files

* omnifocus
	* element(op="reference|add|update|delete") (not specifying an op = add)

A `op="reference"` is basically a copy of the referenced data within the transaction (in case something were to happen to the original reference).

Any id referred to in the transaction file will be included as part of the transaction with an `op="reference"` attribute.

### Comments

The `id` used as attribute is 11 characters long, in what appears to be base 64 (`A-Za-z0-9\_\-`).

The `datetime` is formatted according to [ISO 8601](https://en.wikipedia.org/wiki/ISO_8601) with precision to the milliseconds (`2014-11-03T10:14:09.123Z`).

`Interval format` is @periodduration (@1d, @1w, @1m, @1y) 

The `rank` appears to be a global value shared by all data types.

## Reference
[omnifocus-viewer Omnifocus File Format](https://code.google.com/p/omnifocus-viewer/wiki/WikiPageMain)
