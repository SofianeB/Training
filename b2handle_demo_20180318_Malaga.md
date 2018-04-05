Using B2Handle
==============

Training material for EOSC-HUB week 18-19 April,
2018, Malaga.

Sofiane Bendoukha, 18th April, 2018

What is B2Handle?
-----------------

> The B2Handle Python library is a client library for
interaction with a [Handle System](https://handle.net) server, using the native
REST interface introduced in Handle System 8. The library offers methods to
create, update and delete Handles as well as advanced functionality such as
searching over Handles using an additional search servlet and managing multiple
location entries per Handle.

> The library currently supports Python 2.6, 2.7
and 3.5, and requires at least a Handle System server 8.1.
The library requires
OpenSSL v1.0.1 or higher.

> In this training, we will B2Handle 1.0.2.

> For
more information on the methods offered by the library, please consult the
[technical documentation](http://eudat-b2safe.github.io/B2HANDLE). The
documentation also contains information on how to set up correct certificates
for the Handle Server so it accepts modification REST requests and how to set up
client authentication using public keys.


Link to this Demo
-----------------
https://bit.ly/2Gk17og

(1) Import
==========

First, we have to import the PyHandle library. The
library is used by creating a client object and using its methods to interact
with the Global Handle System.

The object that we are going to interact with is
called the `Client`, so let's import it directly.

(If you use B2Handle 1.0.x, use this import statement instead:)

```python
from b2handle.handleclient import EUDATHandleClient as Client
```

The help() method gives us useful information about its methods.

```python
# help(Client) # Attention, this gives a long result!
```

> Task 1a: Import pyhandle from the provided egg.

> Task 1b: Use the help()
function to find out which methods the Client provides.


(2) Resolving handles
=================

It is easy to resolve a handle and read its handle record
using the Pyhandle library. For this, we instantiate the client in read-mode and
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
handle = '21.T14998/TEST'
record = client.retrieve_handle_record(handle)
print type(record)
```

```python
print record
```

But it's nicer to pretty-print the record:

```python
#pretty_record = client.pretty_print_handle_record(handle)
#print pretty_record
```

We can access individual values using:

```python
value1 = client.get_value_from_handle(handle, 'URL')
value2 = client.get_value_from_handle(handle, 'IS_PART_OF')
print value1
print value2
```

> Task 2.1a: Instantiate the pyhandle client for read-only access.

> Task 2.1b:
Retrieve entire record of handle *10876.test/rdapidxyz* as a dictionary.

> Task
2.1c: Retrieve a single handle value from the record.

> Task 2.1d: Pretty-print
the handle record.

> Task 2.1e: [advanced] Pretty-print the handle record
including the handle record indices. Hint: with_index=True


(2.2) Decrease
server interactions [optional]
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
#print client.pretty_print_handle_record(handle, record_json)
```

```python
print client.get_value_from_handle(handle, 'CREATION_DATE', record_json)
```

> Task 2.2: Retrieve the handle record as a json object and use the read methods
without accessing the Handle Server


(3) Updating handle records
=======================

Now we'd like to modify one of the values of that
handle record. The client provides a method for this:
`modify_handle_value(handle, ...)`.

Let's try it - let's change the creation
date from '2016-05-24' to '2016-05-25':
[Attention - this command will throw an
error, this is expected!]

```python
client.modify_handle_value(handle, CREATION_DATE='2016-05-25')
```

Ups - that did not work. It tells us *'HandleAuthenticationError: Insufficient
permission: No credentials passed. Read access only.'*.

(3.1) Write access to a
Handle Server
-------------------------------------

For
modifying/creating/reading handle records, we first need to authenticate. In
this tutorial, we will use a username and a password. (There is other methods,
e.g. using client certificates.)

```python
user = '300:10876.test/USERLIST'
pw = 'lunchbreak'
```

Also, we need to specify the URL of a handle server to write to. It is not
possible to write to the Global Handle Server. For each prefix, there is only
one server with write access.

```python
url = 'https://handle8.dkrz.de'
```

This server is the write server for the test prefix we are going to use:

```python
prefix = '10876.test'
```

For providing these necessary configuration values, we can use the following
method to create our client object:

```python
client = Client.instantiate_with_username_and_password(url, user, pw, HTTPS_verify=False)
```

> Task 3.1: Instantiate the client for write access, using the above username
and password.

(3.2) Modifying handle records
-------------------------------
### Modify existing values

Now we can try to modify again:

```python
client.modify_handle_value(handle, CREATION_DATE='2016-05-25')
```

Did it work? Let's check!

```python
print client.get_value_from_handle(handle, 'CREATION_DATE')
```

### Adding new values
With the same method, we can add new values to a handle
record:

```python
client.modify_handle_value(handle, PLEASE_ADD_ME='added value')
print client.pretty_print_handle_record(handle)
```

To prevent adding new values (for example in case of typos) we can set a flag:

```python
client.modify_handle_value(handle, CREATION_DATE='2016-05-26', add_if_not_exist=False)
client.modify_handle_value(handle, RCEATION_DATE='2016-05-27', add_if_not_exist=False)
print client.pretty_print_handle_record(handle)
```

### Deleting values
In case we did not set that flag and accidentally wrote a
wrong entry, the `delete_handle_value()` method allows to delete that entry:

```python
client.modify_handle_value(handle, RCEATION_DATE='2016-05-28')

print 'Added wrong value:'
print client.get_value_from_handle(handle, 'RCEATION_DATE')

client.delete_handle_value(handle, 'RCEATION_DATE')

print 'Deleted wrong value:'
print client.get_value_from_handle(handle, 'RCEATION_DATE')
```

> Task 3.2a: Modify an existing handle value in your test handle
`10876.test/rdapid[xyz]` (with [xyz] being your assigned number).

> Task 3.2b:
Attempt to modify handle value that does not exist yet and prevent the library
from creating it.

> Task 3.2c: Add a handle value to the handle record.

> Task
3.2d: Add several new handle values to the handle record at the same time (with
one command).

> Task 3.2e: Delete one of the handle values previously added.
(4) Registering handles (creating handle records)
=================================================

In their most simple form,
PIDs are simple redirection to a URL. In this case, all they have is an entry
that stores the URL. You can simply create such a handle using the method
`register_handle()`:

```python
newhandle = prefix+'/myhandle'
url = 'www.dkrz.de'
client.register_handle(newhandle, url)
```

If we execute this code a second time - or if another participant of the course
has already executed it -, we run into an error: "HandleAlreadyExistsException".
Of course - that handle record has already been created!
If we are really sure
we would like to overwrite it, we can specify this:

```python
client.register_handle(newhandle, url, overwrite=True)
```

To avoid running into this problem and having to decide whether to overwrite or
not, it is always preferrable to use UUIDs as handle suffixes. Using "speaking
names", i.e. suffixes with semantics, is strongly discouraged.

The library
provides an easy way to generate such a handle name. In this case, don't forget
to store the handle name in a variable for further use.

```python
uuidhandle = client.generate_PID_name(prefix)
print uuidhandle
```

And it's even easier to generate the name and register the handle at the same
time. Again, don't forget to store the name.

```python
uuidhandle = client.generate_and_register_handle(prefix, url)
print uuidhandle
```

We can check the contents of this newly created handle record:

```python
print client.pretty_print_handle_record(uuidhandle)
```

> Task 4a: Register a handle with a uuid name and a simple record (containing
just a URL).

> Task 4b: Add a value to it.

> Task 4c: Register a handle with a
uuid name and several values at the same time.

(5) Authentication - how does it
work?
======================================

(5.1) Username and password
----------------------------

Above, we have seen authentication using a
username and a password. The username was `300:10876.test/USERLIST`, where
`10876.test/USERLIST` is a handle under the `10876.test` prefix, and 300 is an
index value to point to a specific entry in this handle.
Let's have a look at
this handle:

```python
adminhandle = '10876.test/USERLIST'
adminrecord = client.retrieve_handle_record_json(adminhandle)
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
(5.2) Authentication using client certificates [OPTIONAL]
==============================================

There is an alternative to using
password, which is more secure, which is using client-side certificates. For
this, the user provides his private key and his certificate with every write
request. (Advanced: Creating the key pair and the certificate file in the
required format and with the required information is out of scope here. Please
check out B2Handle documentation).

### How does it work?

Similar to above,
where the username points to "HS_SECKEY", where the password is stored, the
username now points to an entry of type "HS_PUBKEY", where the public key is
stored. The handle record above contains various of such public keys.

NOTE:
Usernames in the handle systems are strings of the form "index:prefix/suffix"
pointing to an entry of type "HS_SECKEY" containing the password of that user,
or to an entry "HS_PUBKEY" containing the public key of that user.

### Usage of
client-side certificates

To use client-side certificates for authentication,
the user has to pass a certificate and a private key along with every write
request. Just like the password authentication, the library handles this for the
user.

For this, the library either needs a file containing private key and
certificate, or both as separate files. To simplify those three different ways
of authenticating, there is a special class, called PIDClientCredentials.

```python
from pyhandle.clientcredentials import PIDClientCredentials
cred = PIDClientCredentials(
    handle_server_url='https://handle8.dkrz.de',
    username='301:10876.test/USERLIST',
    private_key='/tmp/dummykeyfile.pem',
    certificate_only='/tmp/dummycertfile.pem'
)
```

...or, even more simple...

```python
cred = PIDClientCredentials.load_from_JSON('/tmp/credentials_workshop.json')
client = Client.instantiate_with_credentials(cred)
```

where the path points to a JSON file containing the necessary info, e.g. like
this:

~~~
{
    "handle_server_url"="https://handle8.dkrz.de",
    "username":
"index:prefix/suffix",
    "private_key"="/path/to/private/key/file.pem",
"certificate_only"="/path/to/certificate/file.pem"

}
~~~

The other
authentication methods can also be passed as JSON, such as:

~~~
{
"handle_server_url"="https://handle8.dkrz.de",
    "username":
"index:prefix/suffix",
    "password": "blablabla"
}
~~~

Or:

~~~
{
"handle_server_url"="https://handle8.dkrz.de",
    "username":
"index:prefix/suffix",
"certificate_and_key"="/path/to/certificate/and/key/file.pem"
}
~~~

The class
PIDClientCredentials reads the JSON file and checks if the necessary information
for at least one of these authentication methods are present.

(6) Who may
modify my handle record? - Authorisation

So, can I modify and update any
handle, provided that I have a username and password (or key pair)? Obviously
not.

To modify a handle, two things are necessary:

1. The user needs to be
given write permission in general by the owner of the prefix ("Authorisation").
2. The user needs to be given update permissions for the handle record in
question.

A little introduction to authorisation is in the next section (but
will not be looked into deeply, as this mainly concerns Handle Server
administrators).

Update permissions for specific handles are more important, so
let's go.

(6) Handle owners
==================

For every handle, it has to be
determined who has the right to update or even delete the handle. In other
words, every handle has an "owner" (Note that the handle prefix owner always has
to rights to change/delete a handle). A handle owner can be one user, or a group
of users.

The library by default sets the user group "200:0.NA/prefix" as the
handle owner. In general, the entry with index 200 of the prefix owner handle
record points to a group of users containing the handle server administrators.
This default can be overwritten by adding a "handleowner" to the credentials
JSON file. We recommend that users set themselves as the handle owners, or that
sensible user groups are defined.

~~~
{
"handle_server_url"="https://handle8.dkrz.de",
    "username":
"300:10876.test/admin",
    "password": "blablabla",
    "handleowner":
"300:10876.test/admin"
}
~~~

(Info: User groups are defined using entries of
type "HS_VLIST" - for more details on this, see section on Authorisation or the
B2Handle documentation).


(7) Advanced: Authorisation [OPTIONAL]
======================================

This section is presented using slides.
This is only a very short summary.

For a username and password or client
certificate to work, the user needs to be given write permission by the owner of
the prefix.

This is done using entries of the type "HS_ADMIN" and "HS_VLIST".
The prefix owner can specify a set of permissions in an entry of type
"HS_ADMIN", in the prefix owner handle ("0.NA/prefix"):

~~~
{
    "index": 100,
"type": "HS_ADMIN",
    "data": {
         "format": "admin",
         "value":
{
             "handle": "10876.test/admin",
             "index: 300,
"permissions":"011111110011"
          }
      }
 }
 ~~~

(This gives a specific
set of permissions to our user, 300:10876.test/admin.)

Permissions are stored
as a string of 12 bits, each of which represents a specific permission that a
user/user group can have or not. The prefix owner can grant a set of permissions
to one user or to a group of users. To give them to a group of users, he can
list these users in an entry of type "HS_VLIST".

An HS_VLIST can again point to
a HS_VLIST, possibly creating a cascading chain of users. This allows for a
detailed user management. For more details on this, and for a suggestion of Best
Practices, please check the online documentation of the B2Handle library
(http://eudat-b2safe.github.io/B2HANDLE/givingpermissiontousers.html).

> Task
7: [optional] Retrieve entire record of handle *10876.test/rdapidxyz* as a
dictionary, including the HS_ADMIN. Hint: hide_complex_types=False.

Wrap-Up
========

## Using pyhandle

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

* Use client certificates for
authentication
* Store credentials in JSON file
* Set yourself or a group as
handle owner



## Documentation on ReadTheDocs:

Check out B2Handle
documentation (and soon pyhandle documentation) for detailed infos!
http://eudat-b2safe.github.io/B2HANDLE

Detailed instructions for:

* Usage of
the library
* Authenticati
* Authorisation / User management
* Handle Server
administration
* Creation of client certificates and key pairs


