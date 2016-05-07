explicit-object-mapper
======================

Map named fields from one json object to another with optional transforms. Any fields not named in the map will not be copied to the destination object

[![Build Status](https://travis-ci.org/opentable/explicitobjectmap-node.svg?branch=master)](https://travis-ci.org/opentable/explicit-object-mapper)

[![NPM](https://nodei.co/npm/explicit-object-mapper.png)](https://nodei.co/npm/explicit-object-mapper)

Mappings consist of a simple javascript array containing mapping instructions:

	[
		'simpleA', //will just copy accross the field
		'simpleB',
		'simpleC',
		'simpleD',
		{'oldname':'newname'}, //will rename the field from oldname to newname
		{	//will rename the field then run the custom tranform on the result
			srcName:'complexoldname',
			dstName:'complexnewname',
			customTransform: function (srcObj, val){
				return val.toUpperCase();
			}
		},
        {	//will rename the field then run the mapper on that value. This allows embedding mappers inside mappers
            srcName:'sourceobjectname',
            dstName:'newname',
            mapper: explicitMapper(['simpleE'])
        },
		{'deep.childA': 'baby'}, //dot notation is currently only supported when renaming fields
		function(srcObj,dstObj){
			dstObj.CustomField = 'whatever'; //post mapping function ran after all the other maps are ran
		}
	]

### Installation
	npm install explicit-object-mapper

### usage

	var explicitObjectMapper = require('explicit-object-mapper');

	var mapObj =
	[
		'simpleA',
		{'oldname':'myVal'},
	];

	var srcObj = {
		simpleA: 'alpha',
		oldname: 'changedName'
	};

	var mapper = explicitObjectMapper(mapObj);
	var dstObj = mapper.map(srcObj);

The output from the above would be:

	{
		simpleA: 'alpha',
		changedName: 'myVal'
	}


If an array of objects is passed in then all objects will be mapped and returned in an array.

### speed
There is some overhead to the mapping process depending on map size and the amount of source data; this can be mitigated a little by creating the mappings ahead of time and reusing them.

### changes in 1.0.0

#### added ability to embed mappers inside maps
We can now add mappers inside maps, for example:


    var objectToMap = { Name: { First: 'Bob', Last: 'Smith' }};
    var childMap = ['Firstname'];
    var rootMap = [
        {
            srcName:'Name',
            dstName:'IncompleteName',
            mapper: childMap
        }
    ];
    var mappedObject = rootMap.map(objectToMap); // { IncompleteName: { Firstname: 'Bob' } }

### changes in 2.0.0
Now needs node > 4.2
Uses es6 features to clean up codebase

### changes in 0.0.6

#### null values now handled correctly
Null values were still not handled properly; They will now get mapped

### changes in 0.0.5

#### falsy values now handled correctly
previously if a source value could be evaluated as false (null, 0, false) then the relevant field would not be mapped, now the field is mapped as long as the source field exists

### changes in 0.0.4

#### optional args to map
map can be called with an optional options variable:

	mapper.map(srcObj, {myVal: true, myOtherVal:'biscuit'});

This object is then passed into any custom mapping functions:

	[
		'simpleA',
		{'oldname':'newname'},
		{
			srcName:'complexoldname',
			dstName:'complexnewname',
			customTransform: function (srcObj, val, options){
				return val.toUpperCase() + options.myOtherVal;
			}
		},
		function(srcObj,dstObj, options){
			dstObj.CustomField = 'whatever'; //post mapping function ran after all the other maps are ran
		}
	]
