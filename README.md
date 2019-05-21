
# Group-By

with Dell Boomi



Some words about, how you *could* use `group-by` in Dell Boomi.



### Why?

Because ...
<p class="fragment">you won't send</p>
<p class="fragment">thousands of objects</p>
<p class="fragment">with one single request to</p>
SAP ...


... that will getting lame.


So you would like to split these thousands of documents, before sending to SAP,
to speed up the process.


How to speed up?

With *parallel processing*.


But you can not simply slice these documents ...
<p class="fragment">... because of <b>race conditions</b>.</p>


Now you are in need to pack these documents nicely.


... with group-by.



## How?

With caching, indices and some merging.



# Steps



# Caching

Create an initial cache of your documents. 


Add an single identifier to these cache to identify your documents.

This single identifier *should* be your group-by property.


For example, in my case, that's: **TTY_NO**.

An identifier, every object or document *should* contain.



# make unique

For example: There are 1.000 documents in your process. Identified via **TTY_NO**.

But just three TTY_NO are used in these documents.


Create a FF profile with one single property and mark this property as: **Enfore Unique**.

Then map the group-by property of your document to this new profile.


As a result, the *Current Data* will just e.g.:

```
1001
1395
1859
```

Store this IDs as a DPP.



# Mapping

Map this data + the cache, to a new profile:

![image](https://user-images.githubusercontent.com/1620425/58101983-d781a780-7be0-11e9-8339-db2095c8c120.png)


Now you are left with just one document in your process, so you need to split this up per object:

![image](https://user-images.githubusercontent.com/1620425/58102097-0bf56380-7be1-11e9-9ccb-1803e9737d1b.png)


You have, at this example, now three documents: One per TTY_NO.



# Streams

I specified a threshold, how many objects should be send per request to SAP. E.g.: 10.000 objects.


We have TTY_NOs with that amount of objects here:

```yaml
1001: 4.000
1395: 4.000
1859: 5.000
```

`1001, 1395` will be send in the first request, while `1859` will be send in the next request.


To make it possible, to combine several TTY_NO in one document, until we reach the specified threshold (10.000)
we map them to a FF profile. Afterwards, we can join them linewise.


Every document in Boomi is available as a stream in scripting:

```java
InputStream is = dataContext.getStream(i);
```


The FF profile will allow us, to combine several TTY_NO, just per joining the streams (lines):

```java
InputStream is = dataContext.getStream(i);
List<InputStream> streams = Arrays.asList(is);

while (...) {
	i += 1;
	streams = streams.plus(dataContext.getStream(i));
}

InputStream wholeRequest = new SequenceInputStream(Collections.enumeration(streams));    
dataContext.storeStream(wholeRequest, props);
```



# let it flow

Last mapping step is to remap your data from FF to your final format. In my example, the format will be a JSON.

After this is done: Specify via *Flow Control Shape*, how your requests should get simultaneously be send and you are done.



# A process

![image](https://user-images.githubusercontent.com/1620425/58103568-7c9d7f80-7be3-11e9-98e0-39e8126ce5f2.png)
