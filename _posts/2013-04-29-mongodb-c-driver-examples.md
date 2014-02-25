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
bson_init(&amp;object); /* We need to initialize each bson object */
bson_finish(&amp;object); /* When you are ready to use it, you need to explicitly call this api. */
</pre>

So now we got a empty object, but there's an existing api to finish this job directlty. You can use `bson_init_empty(&amp;object)`.

But notice, in the offical documents or other code examples they may use `bson_empty(&amp;object)`.

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
bson_init(&amp;object);
bson_append_string(&amp;object, "name", "Rock");
bson_append_int(&amp;object, "age", 25);
bson_append_double(&amp;object, "salary", 1.11);
bson_append_bool(&amp;object, "cute", 1);
bson_finish(&amp;object);
bson_print(&amp;object);
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
bson_init(&amp;object);
bson_append_string(&amp;object, "name", "Rock");
bson_append_int(&amp;object, "age", 25);

bson_append_start_array(&amp;object, "hobbies");
bson_append_string(&amp;object, "0", "jogging");
bson_append_string(&amp;object, "1", "hiking");
bson_append_finish_array(&amp;object);

bson_append_start_object(&amp;object, "favorite_apps");
bson_append_string(&amp;object, "ios", "angry bird");
bson_append_string(&amp;object, "android", "wechat");
bson_append_finish_object(&amp;object);

bson_finish(&amp;object);
bson_print(&amp;object);
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

Each time we need to connect to MongoDB first.

<pre>
mongo conn;
if (mongo_client(&amp;conn, "127.0.0.1", 27017) != MONGO_OK) {
	fprintf(stderr, "connect failed..., err:%d\n", conn.err);
	exit(1);
}
</pre>


In this example, we try to connect local mongdb instance, if connect failed error code will be set.


Send Queries And Indicate which fields in documents to return
---

We can use this api to send queries to MongoDB.

`mongo_cursor *mongo_find( mongo *conn, const char *ns, const bson *query, const bson *fields, int limit, int skip, int options );`

<pre>
bson query;
bson_init(&amp;query);
bson_append_string(&amp;query, "name", "rock");
bson_finish(&amp;query);

bson fields;
bson_init(&amp;fields);
bson_append_int(&amp;fields, "age", 1);
bson_append_int(&amp;fields, "_id", 0);
bson_finish(&amp;fields);

mongo_cursor *cursor = mongo_find(&amp;conn, "test.example_collection", &amp;query, &amp;fields, 0, 0, 0);
fprintf(stderr, "cursor:%p\n", cursor);
</pre>

In this example, we first constructed our query conditions and later restricted which fields to return just like `db.example_collection.find({'name':'Rock'}, {'age':1, '_id':0})`. If you want to set `limit` and `skip` some results, you can modify the fifth and sixth parameters. If you don't want to restrict the fields to return, you can set the forth parameter to NULL.

If we want to gain the same goal like `db.example_collection.find({'age':{"$gte": 12}})`, we can define our query like this.
<pre>
bson query;
bson_init(&amp;query);
bson_append_start_object(&amp;query, "age");
bson_append_int(&amp;query, "$gte", 12);
bson_append_finish_object(&amp;query);
bson_finish(&amp;query);
</pre>

If there's no error occured, this api will return back a cursor object, this cursor object is allocated in the heap, so you need to remember to release it. I will explain the release methods in later sections.

Play with the results
---
Till now we know how to construct the queries, how to restrict the fields to return, so let's begin to play with the results like iterating all the matched documents and iterating the subdocuments...

We can use `mongo_cursor_bson(cursor)` to fetch the document current the cursor pointed, but how to iterate all the documents matched? We can write like this:

<pre>
bson *doc;
while (mongo_cursor_next(cursor) == MONGO_OK) {
    doc = (bson *)mongo_cursor_bson(cursor);
}
</pre>

So now we can play with the single document. We need a iterator to iterate the document. You should have familiar with the concept of iterators.

`bson_iterator it;` like other objects, we need to initialize it first. `bson_iterator_init(&amp;it, doc)`, you must supply this iterator aims to iterator which document.

Now we can iterator all the keys, simple example:

<pre>
bson_iterator it;
bson_iterator_init(&amp;it, doc);
while (bson_iterator_next(&amp;it) != BSON_EOO) {
    fprintf(stderr, "key:%s\n", bson_iterator_key(&amp;it));
}
</pre>

`bson_iterator_key` is used to return the key, remember all keys are string type, there's no interger key type. If you know the key like "name", you want to got that object directly, you can use `bson_find(&amp;it, doc, "key")` api to index the key type for you, this api likes with `bson_iterator_next` return the value type. Sometimes you need to check the return value. But how to iterator the sub document?

For the sub document, you need to define a new iterator called `bson_iterator sub_it;`, then call `bson_iterator_subiterator(&amp;it, &amp;sub_it);` to initialize the sub iterator. Please remember the hierarchy first is the parent iterator, seconds is the sub iterator especially when you iterate the deep nested documents. When you want to fetch the current value the iterator pointed to, you can use `bson_iterator_double/int...` to fetch the value.


Release the resource
---
Finally we need to free resources manually, we need to call `mongo_cursor_destroy` to release the cursor object, and call `mongo_destroy` to release the connection object called `mongo_conn`. Remember to pass by the pointer.

In the future, maybe I will write a post to explain the internal implementation of the mongodb c driver including compare with the Python Driver. If my spare time is enough, I can also explain the detail implementation of MongoDB. Maybe this is someone really interested.


Any questions? Don't hesitate to send me emails.
