CNES web scraper
================

[![Build Status](https://travis-ci.org/transparenciasjc/datasus_node_scrap.svg)](https://travis-ci.org/transparenciasjc/datasus_node_scrap)
[![Coverage Status](https://coveralls.io/repos/transparenciasjc/datasus_node_scrap/badge.svg)](https://coveralls.io/r/transparenciasjc/datasus_node_scrap)
[![bitHound Score](https://www.bithound.io/github/transparenciasjc/datasus_node_scrap/badges/score.svg?)](https://www.bithound.io/github/transparenciasjc/datasus_node_scrap/master)
[![Dependency Status](https://david-dm.org/transparenciasjc/datasus_node_scrap.svg "Dependencies Checked & Updated Regularly (Security is Important!)")](https://david-dm.org/transparenciasjc/datasus_node_scrap)


## Table of Contents
<!-- toc -->
* [How to debug:](#how-to-debug)
* [How to run:](#how-to-run)
* [Limitations:](#limitations)
* [How to export](#how-to-export)
  * [To CSV:](#to-csv)
  * [To Database dump:](#to-database-dump)

<!-- toc stop -->
<!--
	ps: table of contents generated by [readme-toc](https://www.npmjs.com/package/readme-toc) plugin.
-->

You have to use nodejs version 0.10.x

Download the dependencies:

	npm install

Install MongoDB:

	sudo apt-get install mongodb

# How to debug:

	node-inspector
	node --debug-brk crawler.js

Then, go to http://localhost:8080/debug?port=5858

# How to run:

	node --max-old-space-size=8192 --expose-gc crawler.js

# Limitations:

The script consumes a lot of memory in order of their execution time. On the first minute of their execution, about 150 registers are downloaded, netherless, this measure is going down in order of the memory consumption.

A real fixes to this problem is to study how the V8 garbage collector works, and pay attention to remove the closure variables to improve less memory consumption.

So, an work around to this problem is to kill and reopen the script in determined cycle of time using CRON. To do that, run the following instructions:

create a file with the following content on `/etc/cron.d/crawler` (without the extension '.sh')

	#!/bin/sh
	pkill node
	cd "<PATH_OF_SOURCE>/node_scrap/"
	<YOUR_NODE_PATH>/node --max-old-space-size=8192 index.js > /tmp/crawler.log &

run:

	crontab -e

add this on the last line of the file:

	*/2 * * * * /bin/sh /etc/cron.d/crawler

This will run automatically the script `/etc/cron.d/crawler` on the interval of 2 in 2 minute, It would be kill and re-execute the crawler script.

# Running the crawler
## First Step

First of all you need to change the function "initialize" of the class index.js, which the content is something like that:

```js
downloadModule.processStates();
//downloadModule.processEntities();
```

The first step is to execute the function `processStates()` this function will download all the urls of the entities, in order to make the process synchronous, and it will maintain the control of what register was downloaded.

More of 300.000 url's will be downloaded. You can check It on database:

```shell
 mongo
 use cnes2015
 show collections
 db.entitytodownloads.count();
```

## Second step

You must backup the collection `entityurls` to `entityurls_bak` by using the following command:

`db.entityurls.copyTo('entityurls_bak')`

Then, you have to change the function `initialize` in order to make it call the function that download the entity details:

```js
//downloadModule.processStates();
downloadModule.processEntities();

```

You can now check your log with tail: `tail -f /tmp/crawler.log`

# How to export

## To CSV:

	cd output
	mongoexport --db cnes --collection entities --csv --fieldFile entities_fields.txt --out entities.csv

## To Database dump:

to generate the dump:

	mongodump -d cnes -o output

to restore:

	mongorestore cnes