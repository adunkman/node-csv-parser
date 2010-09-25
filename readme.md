
<pre>
             _   _           _         _____  _______      __
            | \ | |         | |       / ____|/ ____\ \    / /
            |  \| | ___   __| | ___  | |    | (___  \ \  / / 
            | . ` |/ _ \ / _` |/ _ \ | |     \___ \  \ \/ /  
            | |\  | (_) | (_| |  __/ | |____ ____) |  \  /   
            |_| \_|\___/ \__,_|\___|  \_____|_____/    \/   
</pre>

This project provide CSV parsing and has been tested and used on large source file (over 2Gb).

-   Support delimiter, quote and escape characters
-   Line breaks discovery: line breaks in source are detected and reported to destination
-   Data transformation
-   Asynch and event based
-   Support for large datasets
-   Complete test coverage as sample and inspiration

Quick exemple
-------------

Using the library is a 4 steps process:

1.	Create a source
2.	Create a destination (optional)
3.	Transform the data (optional)
4.  Listen to events (optional)

	var csv = require('csv');
	csv()
	.fromPath(__dirname+'/sample.in')
	.toPath(__dirname+'/sample.out')
	.transform(function(data){
		data.unshift(data.pop());
		return data;
	})
	.on('data',function(data,index){
		console.log('#'+index+' '+JSON.stringify(data));
	})
	.on('end',function(count){
		console.log('Number of lines '+count);
	})
	.on('error',function(error){
		console.log(error.message);
	});

Installing
----------

Via git (or downloaded tarball):

    $ git clone git://github.com/ajaxorg/cloud9.git

Then, simply copy or link the lib/csv.js file into your $HOME/.node_libraries folder or inside a declared path folder.

Via [npm](http://github.com/isaacs/npm):

    $ npm install csv

Creating a source
-----------------

Options are:

-   *delimiter*    
    Set the field delimiter, one character only, default to comma.
    
-   *quote*    
    Set the field delimiter, one character only, default to double quotes.
    
-   *escape*    
    Set the field delimiter, one character only, default to double quotes.
    
-   *columns*    
    List of fields or true if autodiscovered in the first CSV lien, impact the `transform` argument and the `data` event by providing an object instead of an array, order matters, see the transform and the columns section below.

The following method are available:

-   *fromPath*    
    Take a file path as first argument and optionnaly on object of options as a second arguments.
    
-   *fromStream*    
    Take a readable stream as first argument and optionnaly on object of options as a second arguments.
    
-   *from*    
    Take a string, a buffer, an array or an object as first argument and optionnaly some options as a second arguments.

Creating a destination
----------------------

Options are:

-   *delimiter*    
    Default to the delimiter read option.
    
-   *quote*    
    Default to the quote read option.
    
-   *escape*    
    Default to the escape read option.
    
-   *columns*    
    List of fields, apply when `transform` return an object, order matters, see the transform and the columns sections below.
    
-   *encoding*    
    Default to 'utf8', apply when a writable stream is created.
    
-   *lineBreaks*    
    String used to delimite record rows or a special value; special values are 'auto', 'unix', 'mac', 'windows', 'unicode'; default to 'auto' (discovered in source).
    
-   *flag*    
    Default to 'w', 'w' to create or overwrite an file, 'a' to append to a file. Apply when using the `toPath` method.
    
-   *bufferSize*    
    Internal buffer holding data before being flush into a stream. Apply when destination is a stream.

The following method are available:

-   *toPath*    
    Take a file path as first argument and optionnaly on object of options as a second arguments.
    
-   *toStream*    
    Take a readable stream as first argument and optionnaly on object of options as a second arguments.

Transforming data
-----------------

You may provide a callback to the `transform` method. The contract is quite simple, you recieve an array of fields for each record and return the transformed record. The return value may be an array, an associative array, a string or null. If null, the record will simply be skipped.

Unless you specify the `columns` read option, `data` are provided as arrays, otherwise they are objects with keys matching columns names.

When the returned value is an array, the fields are merge in order. When the returned value is an object, it will search for the `columns` property in the write or in the read options and smartly order the values. If no `columns` options are found, it will merge the values in their order of appearance. When the returned value is a string, it directly sent to the destination source and it is your responsibility to delimit, quote, escape or define line breaks.

Exemple of transform returning a string

	// node sample/transform.js
	var csv = require('csv');
	
	csv()
	.fromPath(__dirname+'/transform.in')
	.toStream(process.stdout)
	.transform(function(data,index){
		return (index>0 ? ',' : '') + data[0] + ":" + data[2] + ' ' + data[1];
	});
	
	// Print sth like:
	// 82:Zbigniew Preisner,94:Serge Gainsbourg

Events
------

By extending the Node `EventEmitter` class, the library provide a few usefull events:

-	*data* (function(data, index){})
    Thrown when a new row is parsed after the `transform` callback and with the data being the value returned by `transform`. Note however that the event won't be call if transform return `null` since the record is skipped.
	The callback provide two arguements:
	`data` is the CSV line being processed (by default as an array)
	`index` is the index number of the line starting at zero
    
-   *end*
    In case your redirecting the output to a file using the `toPath` method, the event will be called once the writing process is complete and the file closed.
    
-   *error*
    Thrown whenever an error is captured.

Columns
-------

Columns names may be provided or discovered in the first line with the read options `columns`. If defined as an array, the order must match the input source. If set to `true`, the fields are expected to be present in the first line of the input source.

You can define a different order and even different columns in the read options and in the write options. If the `columns` is not defined in the write options, it will default to the one present in the read options. 

When working with fields, the `transform` method and the `data` events recieve their `data` parameter as an object instead of an array where the keys are the field names.

	// node sample/column.js
	var csv = require('csv');
	
	csv()
	.fromPath(__dirname+'/columns.in',{
		columns: true
	})
	.toStream(process.stdout,{
		columns: ['id', 'name']
	})
	.transform(function(data){
		data.name = data.firstname + ' ' + data.lastname
		return data;
	});
	
	// Print sth like:
	// 82,Zbigniew Preisner
	// 94,Serge Gainsbourg

Running the tests
-----------------

Tests are executed with expresso. To install it, simple use `npm install expresso`.

To run the tests
	expresso -I lib test/*

To develop with the tests watching at your changes
	expresso -w -I lib test/*

To instrument the tests
	expresso -I lib --cov test/*

Related projects
----------------

*   Pavel Kolesnikov "ya-csv": http://github.com/wdavidw/ya-csv
*   Chris Williams "node-csv": http://github.com/voodootikigod/node-csv
