# Omnifocus `.ofocus` file format

## Overview

The `.ofocus` file is a basically a directory with a set of `.zip` files.
Each `zip` file contains a `contents.xml` file, which is a transaction in `XML` format.

There is a master file named `00000000000000={uuid}+{randomId}.zip` and multiple files named `{date in GMT}={randomID}+{randomID}.zip` which are transactions files that apply over the master file.

## File content

The content of each contents.xml is as follow

Legend
*Optional*

* omnifocus
	* setting(id)
 		* *added(order) : datetime*
		* *modified : datetime*
		* plist(version)
	* context(id)
		* added(order) : datetime
		* *modified : datetime*
		* name : string
		* rank : signed int
		* *location(name, latitude, longitude)*
	* folder(id)
		* folder(idref)
		* added : datetime
		* *modified : datetime*
		* name : string
		* rank : signed int
		* *hidden : bool*
	* task(id)
		* inbox
		* project
			* folder(idref)
			* last-review : datetime
			* next-review : datetime
			* review-interval : string (interval format)
			* status : string (done, inactive, dropped)
			* singleton : bool
		* *task(idref)*
		* *note*
			* *text*
		* added : datetime
		* *modified : datetime*
		* name : string
		* rank : unsigned int (relative to parent task?)
		* *context(idref)*
		* *due : datetime*
		* *start : datetime*
		* *completed : datetime*
		* estimated-minutes : unsigned int
		* *flagged : bool*
		* *order : string (parallel) (if not specified, it means sequential)*
		* *repeat : string (interval format)*
		* *repetition-rule : string (FREQ=MONTHLY, FREQ=WEEKLY;INTERVAL=4)*
		* *repetition-method : string (fixed)*
	* perspective(id)
		* plist(version)

Transaction xml files

* omnifocus
	* element(op=”reference|update|delete”) (not specifying an op = add)

A op=”reference” is basically a copy of the referenced data within the transaction (in case something were to happen to the original reference).

Any id referred to in the transaction file will be included as part of the transaction with an op=”reference” attribute.

### Comments

The id used as attribute is 11 characters long, in what appears to be in base 64 (supports `_` and `-`).

The datetime is formatted according to ISO 8601 (2014-11-03T10:14:09Z).

Interval format is @periodduration (@1d, @1w, @1m, @1y) 

## Reference
[omnifocus-viewer Omnifocus File Format](https://code.google.com/p/omnifocus-viewer/wiki/WikiPageMain)
