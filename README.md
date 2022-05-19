AD-Promise for Node
=========

AD-Promise is a fork of the great module node-activedirectory by gheery. In addition to that I stripped out the need of the async library (which always crashed in my environment if I was dealin with a lot of simultaneaously requests). In addition all the methods described downwards can not only be used with a callback, but will also return a promise. If you already use the original module you should be able to use this module aswell without having to adjustice anything. This module is still in testing. I would be happy if you would post any issues you might find so they can be corrected.

Now to the original Readme with some additions on how to use the promises alternatively (of course you can also use await I just wrote down the way with .then())

ActiveDirectory is an ldapjs client for authN (authentication) and authZ (authorization) for Microsoft Active Directory with range retrieval support for large Active Directory installations. This code was a port of an existing C# library (not published) that I had written a few years ago. Here are the key features

  - Authenticate
  - Authorization (via group membership information)
  - Nested groups support
  - Range specifier / retrieval support (http://msdn.microsoft.com/en-us/library/dd358433.aspx)
  - Automatic paging support (Active Directory results (MaxPageSize) limited to 1000 per request by default)
  - Recycle bin (tombstone) query support
  - Referral support

Required Libraries
-----------

ActiveDirectory uses the following additional node modules:

* [underscore] - a utility-belt library for JavaScript that provides a lot of the functional programming support
* [ldapjs] - A pure JavaScript, from-scratch framework for implementing LDAP clients and servers in Node.js
* [bunyan](https://github.com/trentm/node-bunyan) - A simple and fast JSON logging module for node.js services
* [limitpromises](https://github.com/relief-melone/limitpromises) - lightweight module for making sure you only run a limited number 
of promises at once. Originally the async library was used instead of limitpromises. But as it does not limit how many async calls
will be made at once which will make you run into problems if you have very larger AD-Groups (10k+ users)

Installation
--------------

```sh
npm install ad-promise
```

Usage
--------------

```js
var ActiveDirectory = require('ad-promise');
var config = { url: 'ldap://dc.domain.com',
               baseDN: 'dc=domain,dc=com',
               username: 'username@domain.com',
               password: 'password' }
var ad = new ActiveDirectory(config);
```

The username and password specified in the configuration are what are used for user and group lookup operations. You can also specify different baseDNs for user and group
(e.g. you have organisiation units in your root ou apart from groups and users and don't want to include them in your search). You can either set user, group or both,
but you will need to supply a default baseDN

```js
var ActiveDirectory = require('ad-promise');
var config = { url: 'ldap://dc.domain.com',
               baseDNs: {
                  user : 'ou=user,ou=someroot,dc=domain,dc=com',
                  group : 'ou=group,ou=someroot,dc=domain,dc=com',
                  default : 'ou=someroot,dc=domain,dc=com'
               }
               username: 'username@domain.com',
               password: 'password' }
var ad = new ActiveDirectory(config);
```

Documentation
--------------
* [authenticate](#authenticate)
* [isUserMemberOf](#isUserMemberOf)
* [find](#find)
* [findUser](#findUser)
* [findGroup](#findGroup)
* [findUsers](#findUsers)
* [findGroups](#findGroups)
* [groupExists](#groupExists)
* [userExists](#userExists)
* [getGroupMembershipForGroup](#getGroupMembershipForGroup)
* [getGroupMembershipForUser](#getGroupMembershipForUser)
* [getUsersForGroup](#getUsersForGroup)
* [getRootDSE](#getRootDSE)
* [findDeletedObjects](#findDeletedObjects)
* [maxSearchesAtOnce](#maxSearchesAtOnce)

---------------------------------------

<a name="authenticate" />

### authenticate(username, password, callback)

Authenticates the username and password by doing a simple bind with the specified credentials.

__Arguments__

* username - The username to authenticate.
* password - The password to use for authentication.
* callback(err, authenticated) - A callback which is called after authentication is completed.

__Example__

```js
var ad = new ActiveDirectory(config);
var username = 'john.smith@domain.com';
var password = 'password';

//With callback;
ad.authenticate(username, password, function(err, auth) {
  if (err) {
    console.log('ERROR: '+JSON.stringify(err));
    return;
  }
  
  if (auth) {
    console.log('Authenticated!');
  }
  else {
    console.log('Authentication failed!');
  }
});

// With Prmomise 

ad.authenticate(username, password).then((res) => {
  console.log('Authenticated!')
}, (err) => {
  console.log('Authentication failed!');
});
```

---------------------------------------

<a name="isUserMemberOf" />

### isUserMemberOf(opts, username, groupName, callback)

Checks to see if a user is a member of the specified group. This function will also check for group membership inside of a group.  Even if a user is not explicity listed as a member of a particular group, if a group that the user is a member of belongs to the group, then this function will return true.

__Arguments__
* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts).
* username - The username to check for membership. Can be specied as a sAMAccountName, userPrincipalName or distinguishedName (dn)
* groupName - The group to check for membership. Can be a commonName (cn) or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, result: {Boolean})

__Example__

```js
var username = 'user@domain.com';
var groupName = 'Employees';

var ad = new ActiveDirectory(config);
var ad.isUserMemberOf(username, groupName, function(err, isMember) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  console.log(username + ' isMemberOf ' + groupName + ': ' + isMember);
});
```

---------------------------------------

<a name="groupExists" />

### groupExists(opts, groupName, callback)

Checks to see if the specified group exists.

__Arguments__
* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts).
* groupName - The group to check if is defined. Can be a commonName (cn) or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, result: {Boolean})

__Example__

```js
var groupName = 'Employees';

var ad = new ActiveDirectory(config);

//Using a callback
ad.groupExists(groupName, function(err, exists) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  console.log(groupName + ' exists: ' + exists);
});

//Using a Promise
ad.groupExists(groupName).then(exists => {
  console.log(groupName + ' exists: ' + exists);
}, err => {
  console.log('ERROR: ' + JSON.stringify(err));
});
```

---------------------------------------

<a name="userExists" />

### userExists(opts, username, callback)

Checks to see if the specified user exists.

__Arguments__
* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts).
* username - The username to check if it exists. Can be a sAMAccountName, userPrincipalName or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, result: {Boolean})

__Example__

```js
var username = 'john.smith@domain.com';

var ad = new ActiveDirectory(config);

//Using a Callback
ad.userExists(username, function(err, exists) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  console.log(username + ' exists: ' + exists);
});

//Using a Promise
ad.userExists(username).then(exists => {
  console.log(username + ' exists: ' + exists);
}, err => {
  console.log('ERROR: ' + JSON.stringify(err));
})
```

---------------------------------------

<a name="getUsersForGroup" />

### getUsersForGroup(opts, groupName, callback)

For the specified group, retrieve all of the users that belong to the group. If the group contains groups, then the members of those groups are recursively retrieved as well to build a complete list of users that belong to the specified group.

__Arguments__
* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts).
* groupName - The name of the group to retrieve membership from. Can be a commonName (cn) or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, groups: {Array[User]})

__Example__

```js
var groupName = 'Employees';

var ad = new ActiveDirectory(config);

//Using a Callback
ad.getUsersForGroup(groupName, function(err, users) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if (! users) console.log('Group: ' + groupName + ' not found.');
  else {
    console.log(JSON.stringify(users));
  }
});

//Using a Promise
ad.getUsersForGroup(gorupName).then(users => {
  console.log(JSON.stringify(users));
}, err => {
  console.log('ERROR: ' + JSON.stringify(err));
});

```


---------------------------------------

<a name="getGroupMembershipForUser" />

### getGroupMembershipForUser(opts, username, callback)

For the specified username, retrieve all of the groups that a user belongs to. If a retrieved group is a member of another group, then that group is recursively retrieved as well to build a complete hierarchy of groups that a user belongs to.

__Arguments__
* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts).
* username - The name of the user to retrieve group membership for. Can be a sAMAccountName, userPrincipalName, or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, groups: {Array[Group]})

__Example__

```js
var sAMAccountName = 'john.smith@domain.com';

var ad = new ActiveDirectory(config);

//Using a Callback
ad.getGroupMembershipForUser(sAMAccountName, function(err, groups) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if (! groups) console.log('User: ' + sAMAccountName + ' not found.');
  else console.log(JSON.stringify(groups));
});

//Using a Promise
ad.getGroupMembershipForUser(SAMAccountName).then(groups => {
  console.log(JSON.stringify(groups));
}, err => {
  console.log('ERROR: ' + JSOBN.stringify(err));
})
```

---------------------------------------

<a name="getGroupMembershipForGroup" />

### getGroupMembershipForGroup(opts, groupName, callback)

For the specified group, retrieve all of the groups that the group is a member of. If a retrieved group is a member of another group, then that group is recursively retrieved as well to build a complete hierarchy of groups that a user belongs to.

__Arguments__
* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts).
* groupName - The name of the user to retrieve group membership for. Can be a sAMAccountName, userPrincipalName, or a distinguishedName (dn)
* callback - The callback to execute when completed. callback(err: {Object}, groups: {Array[Group]})

__Example__

```js
var groupName = 'Employees';

var ad = new ActiveDirectory(config);

//Using a Callback
ad.getGroupMembershipForGroup(groupName, function(err, groups) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if (! groups) console.log('Group: ' + groupName + ' not found.');
  else console.log(JSON.stringify(groups));
});

//Using a Promise

ad.getGroupMembershipForGroup(groupname).then(groups => {
  console.log(JSON.stringify(groups));
}, err => {
  console.log('ERROR: ' + JSON.stringify(err));
});
```

---------------------------------------

<a name="find" />

### find(opts, callback)

Perform a generic search for the specified LDAP query filter. This function will return both
groups and users that match the specified filter. Any results not recognized as a user or group
(i.e. computer accounts, etc.) can be found in the 'other' attribute / array of the result.

__Arguments__
* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts). If only a string is provided, then the string is assumed to be an LDAP filter
* callback - The callback to execute when completed. callback(err: {Object}, groups: {Array[Group]})

__Example__

```js
var _ = require('underscore');
var query = 'cn=*Exchange*';
var opts = {
  includeMembership : [ 'group', 'user' ], // Optionally can use 'all'
  includeDeleted : false
};

var ad = new ActiveDirectory(config);
ad.find(query, function(err, results) {
  if ((err) || (! results)) {
    console.log('ERROR: ' + JSON.stringify(err));
    return;
  }

  console.log('Groups');
  _.each(results.groups, function(group) {
    console.log('  ' + group.cn);
  });

  console.log('Users');
  _.each(results.users, function(user) {
    console.log('  ' + user.cn);
  });

  console.log('Other');
  _.each(results.other, function(other) {
    console.log('  ' + other.cn);
  });
});
```

---------------------------------------

<a name="findDeletedObjects" />

### findDeletedObjects(opts, callback)

If tombstoning (recycle bin) is enabled for the Active Directory installation, use findDeletedObjects to retrieve
items in the recycle bin.
 
More information about tombstoning and enabling can be found at:
 
* [Enable the Active Directory Recycle Bin (and other New Features)](http://technet.microsoft.com/en-us/magazine/ff793473.aspx)
* [Reanimating Active Directory Tombstone Objects](http://technet.microsoft.com/en-us/magazine/2007.09.tombstones.aspx)

Note: That when an LDAP entry / object is tombstoned, not all attributes for that item are retained. 
This is a limitation of Active Directory itself and not the library itself.

__Arguments__
* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts). If only a string is provided, then the string is assumed to be an LDAP filter
* callback - The callback to execute when completed. callback(err: {Object}, result: {Array})

If the baseDN is not specified, then a RootDSE query will be performed on the attached URL and 'ou-Deleted Objects' 
will be appended.

__Example__

```js
var url = 'ldap://yourdomain.com';
var opts = {
  baseDN: 'ou=Deleted Objects, dc=yourdomain, dc=com',
  filter: 'cn=*Bob*'
};
ad.findDeletedObjects(opts, function(err, result) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  console.log('findDeletedObjects: '+JSON.stringify(result));
});
```

---------------------------------------

<a name="findUser" />

### findUser(opts, username, callback)

Looks up or finds a username by their sAMAccountName, userPrincipalName, distinguishedName (dn) or custom filter. If found, the returned object contains all of the requested attributes. By default, the following attributes are returned:

* userPrincipalName, sAMAccountName, mail, lockoutTime, whenCreated, pwdLastSet, userAccountControl, employeeID,  sn,  givenName, initials, cn, displayName, comment, description

__Arguments__

* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts).
* username - The username to retrieve information about. Optionally can pass in the distinguishedName (dn) of the user to retrieve.
* callback(err, user) - The callback to execute when completed. callback(err: {Object}, user: {User})

__Example__

```js
// Any of the following username types can be searched on
var sAMAccountName = 'username';
var userPrincipalName = 'username@domain.com';
var dn = 'CN=Smith\\, John,OU=Users,DC=domain,DC=com';

// Find user by a sAMAccountName
var ad = new ActiveDirectory(config);
ad.findUser(sAMAccountName, function(err, user) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if (! user) console.log('User: ' + sAMAccountName + ' not found.');
  else console.log(JSON.stringify(user));
});
```

---------------------------------------

<a name="findUsers" />

### findUsers(opts, callback)

Perform a generic search for users that match the specified filter. The default LDAP filter for users is
specified as (&(|(objectClass=user)(objectClass=person))(!(objectClass=computer))(!(objectClass=group)))

__Arguments__
* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts). If only a string is provided, then the string is assumed to be an LDAP filter that will be appended as the last parameter in the default LDAP filter. If an Array is provided, AD-Promise will assume its multiple sAMAccountNames.
* callback - The callback to execute when completed. callback(err: {Object}, users: {Array[User]})

__Example__

```js
let query = 'cn=*George*';
let usernames = ["username1", "username2"]; 

const ad = new ActiveDirectory(config);
// Working with a query
ad.findUsers(query, function(err, users) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if ((! users) || (users.length == 0)) console.log('No users found.');
  else {
    console.log('findUsers: '+JSON.stringify(users));
  }
});
// Working with an array of usernames
ad.findUsers(usernames, function(err, users) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if ((! users) || (users.length == 0)) console.log('No users found.');
  else {
    console.log('findUsers: '+JSON.stringify(users));
  }
});

```

---------------------------------------

<a name="findGroup" />

### findGroup(opts, groupName, callback)

Looks up or find a group by common name (CN) which is required to be unique in Active Directory or optionally by the distinguished name. Supports groups with range retrieval specifiers. The following attributes are returned by default for the group:

* objectCategory, distinguishedName, cn, description, member

__Arguments__

* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts).
* groupName -  The group (cn) to retrieve information about. Optionally can pass in the distinguishedName (dn) of the group to retrieve.
* callback(err, group) - The callback to execute when completed. callback(err: {Object}, group: {Group})


__Example__

```js
// Any of the following group names can be searched on
var groupName = 'Employees';
var dn = 'CN=Employees,OU=Groups,DC=domain,DC=com'

// Find group by common name
var ad = new ActiveDirectory(config);
ad.findGroup(groupName, function(err, group) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if (! user) console.log('Group: ' + groupName + ' not found.');
  else {
    console.log(group);
    console.log('Members: ' + (group.member || []).length);
  }
});
```

---------------------------------------

<a name="findGroups" />

### findGroups(opts, callback)

Perform a generic search for groups that match the specified filter. The default LDAP filter for groups is
specified as (&(objectClass=group)(!(objectClass=computer))(!(objectClass=user))(!(objectClass=person)))

__Arguments__
* opts - Optional parameters to extend or override functionality. See [optional parameters](#opts). If only a string is provided, then the string is assumed to be an LDAP filter that will be appended as the last parameter in the default LDAP filter.
* callback - The callback to execute when completed. callback(err: {Object}, groups: {Array[Group]})

__Example__

```js
var query = 'CN=*Admin*';

var ad = new ActiveDirectory(config);
ad.findGroups(query, function(err, groups) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  if ((! groups) || (groups.length == 0)) console.log('No groups found.');
  else {
    console.log('findGroups: '+JSON.stringify(groups));
  }
});
```

---------------------------------------

<a name="getRootDSE" />

### getRootDSE(url, attributes, callback)

Retrieves the root DSE for the specified url. Can be called statically.

__Arguments__
* url - The url to retrieve the root DSE for.
* attributes - The optional list of attributes to retrieve. Returns all if not specified.
* callback - The callback to execute when completed. callback(err: {Object}, result: {Object})

__Example__

```js
var url = 'ldap://yourdomain.com';
ActiveDirectory.prototype.getRootDSE(url, [ 'defaultNamingContext' ], function(err, result) {
  if (err) {
    console.log('ERROR: ' +JSON.stringify(err));
    return;
  }

  console.log('getRootDSE: '+JSON.stringify(result));
});

// Or can be called with an instance...
var ad = new ActiveDirectory(config);
ad.getRootDSE(function(err, result) {
  //...
});
```

---------------------------------------
## Advanced Usage

### Attributes

By default, the following attributes are returned for users and groups:

* user - distinguishedName, userPrincipalName, sAMAccountName, mail, lockoutTime, whenCreated, pwdLastSet, userAccountControl, employeeID,  sn,  givenName, initials, cn, displayName, comment, description
* group - distinguishedName, objectCategory, cn, description

If you need to override those defaults, then you can override them when you create your ActiveDirectory instance:

```js
var ad = new ActiveDirectory({ url: 'ldap://dc.domain.com',
                               baseDN: 'dc=domain,dc=com',
                               username: 'username@domain.com',
                               password: 'password',
                               attributes: {
                                 user: [ 'myCustomAttribute', 'mail', 'userPrinicipalName' ],
                                 group: [ 'anotherCustomAttribute', 'objectCategory' ]
                               }
                              });
```

If overriding the 'user' or 'group' attribute, you must specify ALL of the attributes you want. The existing defaults
will be overridden. Optionally, you can override the attributes on a per call basis using the 'opts' parameter.

### Referrals

By default, referral chasing is disabled. To enable it, specify a referrals attribute when you create your instance.
The referrals object has the following syntax:

```js
{
  referrals: {
    enabled: false,
    excluded: [
      'ldaps?://ForestDnsZones\./.*',
      'ldaps?://DomainDnsZones\./.*',
      'ldaps?://.*/CN=Configuration,.*'
    ]
  }
}
```

The 'excluded' options is a list of regular expression filters to ignore specific referrals. The default exclusion list
is included above, ignoring the special partitions that ActiveDirectory creates by default. To specify these options,
override them as follows:

```js
var ad = new ActiveDirectory({ url: 'ldap://dc.domain.com',
                               baseDN: 'dc=domain,dc=com',
                               username: 'username@domain.com',
                               password: 'password',
                               attributes: { ... },
                               referrals: {
                                 enabled: true,
                                 excluded: [ ]
                               }
                              });
```

If you enable referral chasing, the specified username MUST be a userPrincipalName.

### Custom Entry Parsing

if you want to manipulate the search entry in a different way or perhaps augment the search
result with additional data, you can pass a custom parser. This is useful, for example, in case 
you want to change the objectSid or GUID which are binary values.

Example:

```js
function customEntryParser(entry, raw, callback){
    if (raw.hasOwnProperty("objectSid")){
        entry.objectSid = raw.objectSid;
    }
    if (raw.hasOwnProperty("objectGUID")){
        entry.objectGUID = raw.objectGUID;
    }
    callback(entry);
};
```

If you want to specify your own parser you can override the default parser as follows:

```js
var ad = new ActiveDirectory({ url: 'ldap://dc.domain.com',
                               baseDN: 'dc=domain,dc=com',
                               username: 'username@domain.com',
                               password: 'password',
                               attributes: { ... },
                               referrals: { ... },
                               entryParser : customEntryParser
                              });
```

Optionally, you can specify your custom entry parser as part of the 'opts' object. See [optional parameters](#opts)
for more information.

```js
var opts = function(entry, raw, callback) {
  entry.retrievedAt = new Date();
  callback(entry);
};
ad.findUser(opts, 'userPrincipalName=bob@domain.com', function(err, user) {
 ...
});
```
 
<a name="opts" />

### Optional Parameters / Extended Functionality

Any method which takes an 'opts' parameter allows for additional options. Options for both activedirectory.js
and the internal ldapjs client are supported. 
 
Currently supported ldapjs opts are:

* url - a valid LDAP url.
* host - the host name to connect to (used with port in lieu of url)
* port - the port to connect to (used with hostname in lieu of url)
* secure - indicates if ldaps:// vs ldap:// is used. (used with hostname/port in lieu of url)
* tlsOptions - additrional tls options (see ldapjs for more information)
* socketPath - If you're running an LDAP server over a Unix Domain Socket, use this.
* log - You can optionally pass in a bunyan instance the client will use to acquire a logger. The client logs all messages at the trace level.
* timeout - How long the client should let operations live for before timing out. Default is Infinity.
* idleTimeout - How long the client should wait before timing out on TCP connections. Default is up to the OS.
* bindDN - The DN all connections should be bound as.
* bindCredentials - The credentials to use with bindDN.
* scope	- One of base, one, or sub. Defaults to base.
* filter - A string version of an LDAP filter (see below), or a programatically constructed Filter object. Defaults to (objectclass=*).
* attributes - attributes to select and return (if these are set, the server will return only these attributes). Defaults to the empty set, which means all attributes.
* sizeLimit - the maximum number of entries to return. Defaults to 0 (unlimited).
* timeLimit - the maximum amount of time the server should take in responding, in seconds. Defaults to 10. Lots of servers will ignore this.

Options for activedirectory.js:

* baseDN - The alternative baseDN to use than the one specified in the ctor.
* includeMembership - Indicates that a search or find operation should enumerate the group memberships of the specified result types. Supported values are [ 'user', 'group', 'all' ].
* includeDeleted - Indicates that results should include tombstoned / deleted items. Please see [findDeletedObject](#findDeletedObjects) for additional notes and caveats.
* entryParser - Allows for a custom function to be specified for parsing of the resulting ldap object. Examples include augmenting ldap data with external data from an RDBMs. function onParse(entry, raw, callback) { callback(entry); } If null is returned, the result is excluded.

#### Example

```js
var opts = {
  scope: 'sub',
  filter: 'objectClass=User',
  includeMembership: [ 'user' ],
  entryParser: function(entry, raw, callback) {
    // returning null with exclude result
    if (entry.ignore) return(null);

    entry.retrievedAt = new Date();
    entry.preferredServer = getPreferredServerFromDatabase(entry.userPrincipalName);

    callback(entry);  
  }
};
```

<a name="maxSearchesAtOnce" />

### maxSearchesAtOnce

Originally node-activedirectory does not limit how many searches will be sent via LDAP parallel. This will cause problems with
very large AD-Groups (10k+ users). ad-promise will limit this to 5000 searches which seems to be a good trade off between performance
and stability. You can however change this value if you want to. You can set the parameter like this

```js
var ActiveDirectory = require('ad-promise');
var config = { 
  url: 'ldap://dc.domain.com',
  baseDN: 'dc=domain,dc=com',
  username: 'username@domain.com',
  password: 'password',
  maxSearchesAtOnce: 3000  
}
var ad = new ActiveDirectory(config);
```

------------------------------------------------

  [underscore]: http://underscorejs.org/
  [async]: https://github.com/caolan/async
  [ldapjs]: http://ldapjs.org/
