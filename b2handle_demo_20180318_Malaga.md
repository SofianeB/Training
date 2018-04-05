Using B2Handle (Demo)
=====================

Training material for EOSC-HUB week 18-19 April,
2018, Malaga.

Sofiane Bendoukha, 18th April, 2018

What is B2Handle?
-----------------

The B2Handle Python library is a client library for
interaction with a [Handle System](https://handle.net) server, using the native
REST interface introduced in Handle System 8. The library offers methods to
create, update and delete Handles as well as advanced functionality such as
searching over Handles using an additional search servlet and managing multiple
location entries per Handle.

The library currently supports Python 2.6, 2.7
and 3.5, and requires at least a Handle System server 8.1.
The library requires
OpenSSL v1.0.1 or higher.

For this demo, we will use B2Handle 1.1.1.

For more information on the methods offered by the library, please consult the
[technical documentation](http://eudat-b2safe.github.io/B2HANDLE). The
documentation also contains information on how to set up correct certificates
for the Handle Server so it accepts modification REST requests and how to set up
client authentication using public keys.


Link to this Demo
-----------------

If you want to try this out, use this link:

https://bit.ly/2Gk17og

(1) Import
==========

First, we have to import the b2handle library. The
library is used by creating a client object and using its methods to interact
with the Global Handle System.

The object that we are going to interact with is
called the `Client`, so let's import it directly.


```python
from b2handle.handleclient import EUDATHandleClient as Client
```

The help() method gives us useful information about its methods.

```python
# help(Client) # Attention, this gives a long result!
```


(2) Resolving handles
=================

It is easy to resolve a handle and read its handle record
using the b2handle library. For this, we instantiate the client in read-mode and
use its reading methods.

(2.1) Instantiation of the client
---------------------------

For using the client in read-only mode, no further
information is necessary. It works out-of-the-box:

```python
client = Client.instantiate_for_read_access()
```

```python
#from b2handle.handleclient import HandleClient as Client
```

Now we can use its various reading methods, for example
`get_value_from_handle(handle)` or `retrieve_handle_record(handle)`.

For
example, `retrieve_handle_record(handle)` returns a dictionary of the record's
entries:

```python
handle = '21.T14998/TESTHANDLE'
record = client.retrieve_handle_record(handle)
print(type(record))
```

```python
print(record)
```


We can access individual values using:

```python
value1 = client.get_value_from_handle(handle, 'URL')
value2 = client.get_value_from_handle(handle, 'IS_PART_OF')
print(value1)
print(value2)
```

> Task 2.1a: Instantiate the b2handle client for read-only access.

> Task 2.1b:
Retrieve entire record of handle *21.T14998/TESTHANDLE* as a dictionary.

> Task
2.1c: Retrieve a single handle value from the record.


(2.2) Decrease server interactions [optional]
---------------------------------------------
The method `get_value_from_handle()` accesses the Handle Server each time. This
is a performance slowdown. To avoid it, it is possible to retrieve the record
once and then pass it on to the reading methods.

This retrieves the record from
the server:

```python
record_json = client.retrieve_handle_record_json(handle)
```

If you pass it to the reading methods, these do not access the Handle Server
anymore:

```python
print(client.get_value_from_handle(handle, 'CREATION_DATE', record_json))
```

> Task 2.2: Retrieve the handle record as a json object and use the read methods
without accessing the Handle Server


(3) Registering handles (creating handle records)
=================================================

In their most simple form, PIDs are simple redirection to a URL. In this case, all they have is an entry
that stores the URL. You can simply create such a handle using the method
`register_handle()`.

In this Demo we are going to create a PID for a file hosted in _figshare_.

```python
prefix = '21.T14998'
newhandle = prefix+'/file_handle'
url = 'https://ndownloader.figshare.com/files/10793093'
client.register_handle(newhandle, url)
```

Ups - that did not work. It tells us *'HandleAuthenticationError: Insufficient
permission: No credentials passed. Read access only.'*.


(3.1) Write access to a Handle Server
-------------------------------

For modifying/creating/reading handle records, we first need to authenticate. In
this tutorial, we will use a username and a password. (There is other methods,
e.g. using client certificates.)

```python
user = '300:21.T14998/USER_TRAINING'
pw = '*password*'
```

Also, we need to specify the URL of a handle server to write to. It is not
possible to write to the Global Handle Server. For each prefix, there is only
one server with write access.

```python
handle_server_url = 'https://handle.dkrz.de:8004'
```

This server is the write server for the test prefix we are going to use:

For providing these necessary configuration values, we can use the following
method to create our client object:

```python
client = Client.instantiate_with_username_and_password(handle_server_url, user, pw, HTTPS_verify=False)
```

> Task 3.1: Instantiate the client for write access, using the above username
and password.

(3.2) Let's try again
---------------------

```python
client.register_handle(newhandle, url)
```

If we execute this code a second time - or if another participant of the course
has already executed it -, we run into an error: "HandleAlreadyExistsException".
Of course - that handle record has already been created!

If we are really sure we would like to overwrite it, we can specify this:

```python
client.register_handle(newhandle, url, overwrite=True)
```

(3.3) Best practice
-------------------

To avoid running into the problem and having to decide whether to overwrite or
not, it is always preferrable to use UUIDs as handle suffixes. Using "speaking
names", i.e. suffixes with semantics, is strongly discouraged.

The library provides an easy way to generate such a handle name. In this case, don't forget
to store the handle name in a variable for further use.

```python
uuid = client.generate_PID_name(prefix)
print(uuid)
```

And it's even easier to generate the name and register the handle at the same
time. Again, don't forget to store the name.

```python
uuidhandle = client.generate_and_register_handle(prefix, url)
print(uuidhandle)
```

We can check the contents of this newly created handle record:

```python
print(client.retrieve_handle_record(uuidhandle)
```

> Task 3a: Register a handle with a uuid name and a simple record (containing
just a URL).

> Task 3b: Add a value to it.

> Task 3c: Register a handle with a
uuid name and several values at the same time.


(4) Updating handle records
===========================

Now we'd like to modify one of the values of that
handle record. The client provides a method for this:
`modify_handle_value(handle, ...)`.


Let's try it - let's add the creation date and file type to the Handle record.

```python
client.modify_handle_value(handle, CREATION_DATE='2016-05-25')
```


- ***adding new values (create some Metadata)***

With the same method, we can add new values to the Handle record.

<br></br>
```python
client.modify_handle_value(newhandle, TYPE='file')
print(client.retrieve_handle_record(newhandle))
```

To prevent adding new values (for example in case of typos) we can set a flag:

```python
client.modify_handle_value(newhandle, STATUS='published', add_if_not_exist=False)
print(client.retrieve_handle_record(handle))
```

- ***deleting values***

In case we did not set the flag and accidently wrote a wrong entry, the _delete_handle_value()_
methos allows to delete that entry:

```python
client.modify_handle_value(handle, RCEATION_DATE='18-04-2018')
print('added wrong value:')
print(client.get_value_from_handle(handle, 'RCEATION_DATE'))
```

```python
client.delete_handle_value(handle, 'RCEATION_DATE')
print('deleted wrong value')
print(client.get_value_from_handle(handle, 'RCEATION_DATE'))
```

> Task 4a: Modify an existing handle value in your test handle
`21.T14998/uuidname` (with uuidname being your assigned number).

> Task 4b:
Attempt to modify handle value that does not exist yet and prevent the library
from creating it.

> Task 4c: Add a handle value to the handle record.

> Task
4d: Add several new handle values to the handle record at the same time (with
one command).

> Task 5e: Delete one of the handle values previously added.


(5) Authentication - how does it work?
======================================

(5.1) Username and password
----------------------------

Above, we have seen authentication using a
username and a password. The username was `300:21.T14998/USER_TRAINING`, where
`21.T14998/USER_TRAINING` is a handle under the `21.T14998` prefix, and 300 is an
index value to point to a specific entry in this handle.
Let's have a look at
this handle:

```python
adminhandle = '21.T14998/USER_TRAINING'
adminrecord = client.retrieve_handle_record(adminhandle)
print client.pretty_print_handle_record(adminhandle, adminrecord, print_special_types=True, with_index=True)
```

We see no entry with index 300. This is because it's hidden, and this is because
it is our password. If it was not hidden, the record would look as follows:

~~~
*********************************************************************
*
10876.test/USERLIST                                               *
*********************************************************************
* 100:
HS_ADMIN: {u'index': 201, u'handle': u'10876.TEST/ADMIN' ... *
* 300: HS_VLIST:
lunchbreak                                         *
* 301: HS_VLIST:
coffeebreak                                        *
*********************************************************************
~~~

The
entry with index 300 is of the type "HS_SECKEY" and contains our password.

Wrap-Up
========

## Using b2handle

* Instantiate the client for read-only or
for write access
* Read-only allows to read the whole record or individual
values
* Write-access requires access information (credentials) to a Handle
Server (at least 8.1)
* Credentials can be username+password or username+client
certificate
* Credentials can be stored in JSON files (use with
PIDClientCredentials)


## Recommendations

* Use client certificates for authentication
* Store credentials in JSON file
* Set yourself or a group as handle owner


## Documentation on ReadTheDocs:

Check out B2Handle documentation for detailed infos!
http://eudat-b2safe.github.io/B2HANDLE

Detailed instructions for:

* Usage of the library
* Authentication
* Authorisation / User management
* Handle Server administration
* Creation of client certificates and key pairs


