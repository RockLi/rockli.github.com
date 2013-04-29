---
layout: post
title: "MongoDB C Driver Examples"
description: ""
category: 
tags: []
---
{% include JB/setup %}

Recent days, I'm working with MongoDB C driver to communicate MongoDB by using capital C. There's almost no detailed articles I can refer, and the documents in MongoDB offical site doesn't reflect the latest C driver code changes. So in this post, I will try to write some codes to demonstrate the basic usage for those new guys.

Background
===
[MongoDB](http://www.mongodb.org/) is a document orinted database, schema less, it comes with lots of client drivers but this article will only cover the [mongo-c-driver](https://github.com/mongodb/mongo-c-driver). I assume you have familar with MongoDB, but have no idea how to communicate with MongoDB by using the C driver or confused with the c driver. I also assume you have successfully installed the mongodb C driver.

We already know MongoDB use the bson format for its internal storage, so I begin with the bson examples first, then continue to demonstrate how to send queries to MongoDB , check errors and play with the results.

BTW: The reason I decide to use MongoDB C driver to write clients, first is one of my client will be written in pure C, and another thing is I compared with Python Driver with the C driver, in my scenario, by using C is 1 times faster than Python at least.


BSON
===

The C BSON implementated is in file bson.h and bson.c, the api is very straitforward, you can always treat the header file as a reference when you forget something.

Empty Object
---------
`{}`

The C implementation:

<pre>
bson object;  /* We define a bson object here. */
bson_init(&object); /* We need to initialize every bson object */
bson_finish(&object); /* When you are ready to use it, you need to explicitly call this api. */
</pre>

So now we got a empty object, but there's an existing api to finish this job directlty. You can use `bson_init_empty(&object)`.

But notice, in the offical documents or other code examples they may use `bson_empty(&object)`.

`bson_empty` is the old api, so please keep update.


Simple K-V Object
----

<pre>
{
    name: 'Rock',
    age: 25,
    salary: 1.11,
    cute: true
}
</pre>

The C Implementation

<pre>
bson object;
bson_init(&object);
bson_append_string(&object, "name", "Rock");
bson_append_int(&object, "age", 25);
bson_append_double(&object, "salary", 1.11);
bson_append_bool(&object, "cute", 1);
bson_finish(&object);
bson_print(&object);
</pre>

This time we define a new bson object and init it. Then call `bson_append_*` apis to set our properties, by using which api depends on the value type. So here we have a string type, an integer type, an double type and the boolean type. Finally we call the `bson_print` api to dump the internal of the bson object passed in. 

The output is similar to this:

	name : 2 	 Rock
	age : 16 	 25
	salary : 1 	 1.110000
	cute : 8 	 true

Please notice the number after the colon, which is the type of the value not the bytes or size ... All the types are defined in the file `bson.h` with the enumerate type `bson_type`.


SubObject(SubDoc) And Array
---

Why I put sub object and array together is that in bson's specification the array is just like the object or called the sub document. We can treat the array type is just like with keys: 0, 1, 2... So when you design the document schema, if you want to replace the subobject to array to reduce the bson objects' size, this may be not right in some scenarios.

In this example, I will try to demostrate this complicated example:

<pre>
{
    name: 'Rock',
    age: 25,
    hobbies: ['jogging', 'hiking'],
    favorite_apps: {
        ios: 'angry bird',
        android: 'wechat',
    }
}
</pre>


The C implementation:

<pre>
	bson object;
	bson_init(&object);
	bson_append_string(&object, "name", "Rock");
	bson_append_int(&object, "age", 25);

	bson_append_start_array(&object, "hobbies");
	bson_append_string(&object, "0", "jogging");
	bson_append_string(&object, "1", "hiking");
	bson_append_finish_array(&object);

	bson_append_start_object(&object, "favorite_apps");
	bson_append_string(&object, "ios", "angry bird");
	bson_append_string(&object, "android", "wechat");
	bson_append_finish_object(&object);

	bson_finish(&object);
	bson_print(&object);
</pre>

Very straitforward, in this example I only introduced the `bson_append_start_*` apis. You can use these apis when you want to set an array type of the subobject. Still remember I explained before the differences between array type and the object type? So when in the array type, you need to supply the keys such as 0, 1, 2 ...

At the last, I use the `bson_print` api to print the internal representation of the object. You can compare the type with the enumeration `bson_type`.

Output:

<pre>
	name : 2 	 Rock
	age : 16 	 25
	hobbies : 4 	 
		0 : 2 	 jogging
		1 : 2 	 hiking

	favorite_apps : 3 	 
		ios : 2 	 angry bird
		android : 2 	 wechat
</pre>


So far I have demenstrated the simple usage of constructing BSON objects including simple objects, subobject and the array type. If you have questions on other bson types, please send me the email, I can add the demo codes in the future maybe.


Comunicate WITH MongoDB
===

Connect
---

Send Queries And Indicate which fields in documents to return
---

Play with the results
---

Release the resource
---





