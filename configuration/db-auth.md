---
description: >-
  VerneMQ supports multiple ways to authenticate and authorize new client
  connections using a database.
---

# Auth using a database

## Introduction and general setup

VerneMQ supports authentication and authorization using a number of popular databases and the below sections describe how to configure the different databases.

The database drivers are handled using the `vmq_diversity` plugin and it therefore needs to be enabled:

```text
plugins.vmq_diversity = on
```

{% hint style="info" %}
The `vmq_diversity` plugin makes it possible to extend VerneMQ using Lua. The documentation can be found [here](../plugin-development/luaplugins.md).
{% endhint %}

When using database based authentication/authorization the enabled-by-default file based authentication and authorization are most likely not needed and should be disabled:

```text
plugins.vmq_passwd = off
plugins.vmq_acl = off
```

{% hint style="warning" %}
You must set `allow_anonymous = off`, otherwise VerneMQ won't use the database plugin for authentication and authorization.
{% endhint %}

In order to use a database for authentication and authorization the database must be properly configured and the auth-data \(username, clientid, password, acls\) to be present. The following sections show some sample requests that can be used to insert such data.

While the handling of authentication differs among the different databases, the handling of ACLs is roughly identical and make use of a JSON array containing one or many ACL objects per configured client.

{% hint style="info" %}
The database integrations will cache the ACLs when the client connects avoiding expensive database lookups for each publish or subscribe message. The cache entries are evicted when the client disconnects.
{% endhint %}

A minimal publish & subscribe ACL JSON object takes the following form:

**General ACL**

```javascript
{
    "pattern": "a/+/c"
}
```

The pattern is a MQTT topic string that can contain MQTT wildcards, but also the template variables `%m` \(mountpoint\), `%u` \(username\), and `%c` \(client id\) which are automatically substituted with the auth data provided.

**Publish ACL**

The publish ACL makes it possible to control the maximum QoS and payload size that is allowed, and if the message is allowed to be retained.

```javascript
{
    "pattern": "a/+/c",
    "max_qos": 2,
    "max_payload_size": 128,
    "allowed_retain": true
}
```

Moreover, the publish ACL makes it possible to modify the properties of a published message through specifying one or multiple `modifiers`. Please note that the modified message isn't re-validated by the ACL.

```javascript
{
    "pattern": "a/+/c",
    "max_qos": 2,
    "max_payload_size": 128,
    "allowed_retain": true,
    "modifiers": {
        "topic": "new/topic",
        "payload": "new payload",
        "qos": 2,
        "retain": true,
        "mountpoint": "other-mountpoint"
    }
}
```

**Subscribe ACL**

The subscribe ACL makes it possible to control the maxium QoS a client is allowed to subscribe to.

```javascript
{
    "pattern": "a/+/c",
    "max_qos": 2
}
```

Like the publish ACL, the subscribe ACL makes it possible to change the current subscription request by returning a custom set of topic/qos pairs. Please note that the modified subscription isn't re-validated by the ACL.

```javascript
{
    "pattern": "a/+/c",
    "max_qos": 2,
    "modifiers": [
        ["new/topic/1", 1],
        ["new/topic/2", 1]
    ]
}
```

### Password verification and hashing methods

When deciding on which database to use one has to consider which kind of password hashing and key derivation functions are available and required. Different databases provide different mechanisms, for example PostgreSQL provides the `pgcrypto` module which supports verifying hashed and salted passwords, while Redis has no such features. VerneMQ therefore also provides client-side password verification mechanisms such as `bcrypt`.

There is a trade-off between verifying passwords on the client-side versus on the server-side. Verifying passwords client-side of course means doing the computations on the VerneMQ broker and this takes away resources from other tasks such as routing messages. With hashing functions such as `bcrypt` which are designed specifically to be slow \(proportional to the number of rounds\) in order to make brute-force attacks infeasible, this can become a problem. For example, if verifying a password with `bcrypt` takes 0.5 seconds then on a single threaded core 2 verifications/second are possible and using 4 single threaded cores 8 verifications/second. So, the number of rounds/security paramenters have a direct impact on the max number of verifications/second and hence also the maximum arrival rate of new clients per second.

For each database it is specified which password verification mechanisms are available and if they are client-side or server-side.

{% hint style="info" %} Note that currently bcrypt version `2a` (prefix `$2a$`) is supported.{% endhint %}


## PostgreSQL

To enable PostgreSQL authentication and authorization the following need to be configured in the `vernemq.conf` file:

```text
vmq_diversity.auth_postgres.enabled = on
vmq_diversity.postgres.host = 127.0.0.1
vmq_diversity.postgres.port = 5432
vmq_diversity.postgres.user = vernemq
vmq_diversity.postgres.password = vernemq
vmq_diversity.postgres.database = vernemq_db
vmq_diversity.cockroachdb.password_hash_method = crypt
```

In case your Postgresql database requires SSL, you'll have to tell the plugin: 

```text 
vmq_diversity.ssl.enabled = on
```

Consult the `vernemq.conf` file for more info about additional options: 

```text
## 
## Default: off
## 
## Acceptable values:
##   - on or off
vmq_diversity.auth_postgres.enabled = off

## 
## Default: localhost
## 
## Acceptable values:
##   - text
## vmq_diversity.postgres.host = localhost

## 
## Default: 5432
## 
## Acceptable values:
##   - an integer
## vmq_diversity.postgres.port = 5432

## 
## Default: root
## 
## Acceptable values:
##   - text
## vmq_diversity.postgres.user = root

## 
## Default: password
## 
## Acceptable values:
##   - text
## vmq_diversity.postgres.password = password

## 
## Default: vernemq_db
## 
## Acceptable values:
##   - text
## vmq_diversity.postgres.database = vernemq_db

## Specify if the postgresql driver should use TLS or not.
## 
## Default: off
## 
## Acceptable values:
##   - on or off
vmq_diversity.postgres.ssl = off

## The cafile is used to define the path to a file containing
## the PEM encoded CA certificates that are trusted.
## 
## Default: 
## 
## Acceptable values:
##   - the path to a file
## vmq_diversity.postgres.cafile = ./etc/cafile.pem

## Set the path to the PEM encoded server certificate.
## 
## Default: 
## 
## Acceptable values:
##   - the path to a file
## vmq_diversity.postgres.certfile = ./etc/cert.pem

## Set the path to the PEM encoded key file.
## 
## Default: 
## 
## Acceptable values:
##   - the path to a file
## vmq_diversity.postgres.keyfile = ./etc/keyfile.pem

## The password hashing method to use in PostgreSQL:
## 
## Default: crypt
## 
## Acceptable values:
##   - one of: crypt, bcrypt
vmq_diversity.postgres.password_hash_method = crypt
```

PostgreSQL hashing methods:

| method | client-side | server-side |
| :--- | :---: | :---: |
| bcrypt | ✓ |  |
| crypt |  | ✓ |

### Creating the Postgres tables

The following SQL DDL must be applied, the `pgcrypto` extension is required if using the server-side `crypt` hashing method:

```sql
CREATE EXTENSION pgcrypto;
CREATE TABLE vmq_auth_acl
 (
   mountpoint character varying(10) NOT NULL,
   client_id character varying(128) NOT NULL,
   username character varying(128) NOT NULL,
   password character varying(128),
   publish_acl json,
   subscribe_acl json,
   CONSTRAINT vmq_auth_acl_primary_key PRIMARY KEY (mountpoint, client_id, username)
 );
```

To enter new ACL entries use a query similar to the following:

```sql
WITH x AS (
    SELECT
        ''::text AS mountpoint,
           'test-client'::text AS client_id,
           'test-user'::text AS username,
           '123'::text AS password,
           gen_salt('bf')::text AS salt,
           '[{"pattern": "a/b/c"}, {"pattern": "c/b/#"}]'::json AS publish_acl,
           '[{"pattern": "a/b/c"}, {"pattern": "c/b/#"}]'::json AS subscribe_acl
    )
INSERT INTO vmq_auth_acl (mountpoint, client_id, username, password, publish_acl, subscribe_acl)
    SELECT
        x.mountpoint,
        x.client_id,
        x.username,
        crypt(x.password, x.salt),
        publish_acl,
        subscribe_acl
    FROM x;
```

## CockroachDB

To enable PostgreSQL authentication and authorization the following need to be configured in the `vernemq.conf` file:

```text
vmq_diversity.auth_cockroachdb.enabled = on
vmq_diversity.cockroachdb.host = 127.0.0.1
vmq_diversity.cockroachdb.port = 26257
vmq_diversity.cockroachdb.user = vernemq
vmq_diversity.cockroachdb.password = vernemq
vmq_diversity.cockroachdb.database = vernemq_db
vmq_diversity.cockroachdb.ssl = on
vmq_diversity.cockroachdb.password_hash_method = bcrypt
```

Notice that if the CockroachDB installation is secure, then TLS is required. If using an insecure installation without TLS, then `vmq_diversity.cockroachdb.ssl` can be set to `off`.

CockroachDB hashing methods:

| method | client-side | server-side |
| :--- | :---: | :---: |
| bcrypt | ✓ |  |
| sha256 |  | ✓ |

### Creating the CockroachDB tables

The following SQL DDL must be applied:

```sql
CREATE TABLE vmq_auth_acl
 (
   mountpoint character varying(10) NOT NULL,
   client_id character varying(128) NOT NULL,
   username character varying(128) NOT NULL,
   password character varying(128),
   publish_acl json,
   subscribe_acl json,
   CONSTRAINT vmq_auth_acl_primary_key PRIMARY KEY (mountpoint, client_id, username)
 );
```

To enter new ACL entries use a query similar to the following, the example is for the `bcrypt` hashing method:

```sql
WITH x AS (
    SELECT
        ''::text AS mountpoint,
           'test-client1'::text AS client_id,
           'test-user1'::text AS username,
           '$2a$12$97PlnSsouvCV7HaxDPV80.EXfsKM4Fg7DAwWhSbGJ6O5CpNep20n2'::text AS hash,
           '[{"pattern": "a/b/c"}, {"pattern": "c/b/#"}]'::json AS publish_acl,
           '[{"pattern": "a/b/c"}, {"pattern": "c/b/#"}]'::json AS subscribe_acl
    )
INSERT INTO vmq_auth_acl (mountpoint, client_id, username, password, publish_acl, subscribe_acl)
    SELECT
        x.mountpoint,
        x.client_id,
        x.username,
        x.hash,
        publish_acl,
        subscribe_acl
    FROM x;
```

## MySQL

For MySQL authentication and authorization configure the following in `vernemq.conf`:

```text
vmq_diversity.auth_mysql.enabled = on
vmq_diversity.mysql.host = 127.0.0.1
vmq_diversity.mysql.port = 3306
vmq_diversity.mysql.user = vernemq
vmq_diversity.mysql.password = vernemq
vmq_diversity.mysql.database = vernemq_db
vmq_diversity.mysql.password_hash_method = password
```

MySQL hashing methods:

| method | client-side | server-side |
| :--- | :---: | :---: |
| sha256 |  | ✓ |
| md5\* |  | ✓ |
| sha1\* |  | ✓ |
| password |  | ✓ |

It should be noted that all the above options stores unsalted passwords which are vulnerable to rainbow table attacks, so the threat-model should be considered carefully when using these. Also note the methods marked with `*` are no longer considered secure hashes.

### Creating the MySQL tables

The following SQL DDL must be applied:

```sql
CREATE TABLE vmq_auth_acl
(
  mountpoint VARCHAR(10) NOT NULL,
  client_id VARCHAR(128) NOT NULL,
  username VARCHAR(128) NOT NULL,
  password VARCHAR(128),
  publish_acl TEXT,
  subscribe_acl TEXT,
  CONSTRAINT vmq_auth_acl_primary_key PRIMARY KEY (mountpoint, client_id, username)
)
```

To enter new ACL entries use a query similar to the following, the example uses `PASSWWORD` to for password hashing:

```sql
INSERT INTO vmq_auth_acl
    (mountpoint, client_id, username,
     password, publish_acl, subscribe_acl)
VALUES
    ('', 'test-client', 'test-user', PASSWORD('123'),
     '[{"pattern":"a/b/c"},{"pattern":"c/b/#"}]',
     '[{"pattern":"a/b/c"},{"pattern":"c/b/#"}]');
```

{% hint style="warning" %}
Note, the `PASSWORD()` hashing method needs to be changed according to the configuration set in `vmq_diversity.mysql.password_hash_method`, it supports the options `password`, `md5`, `sha1` and `sha256`. Learn more about the MySQL equivalent for those methods on [https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html](https://dev.mysql.com/doc/refman/8.0/en/encryption-functions.html).

The default `password` method has been deprecated since MySQL 5.7.6 and not usable with MySQL 8.0.11+. Also, the MySQL authentication method `caching_sha2_password` is not supported. This is the default in MySQL 8.0.4 and later, so you need to add: `default_authentication_plugin=mysql_native_password` under **\[mysqld\]** in e.g. _/etc/mysql/my.cnf_.
{% endhint %}

## MongoDB

For MongoDB authentication and authorization configure the following in `vernemq.conf`:

```text
vmq_diversity.auth_mongodb.enabled = on
vmq_diversity.mongodb.host = 127.0.0.1
vmq_diversity.mongodb.port = 27017
# vmq_diversity.mongodb.login =
# vmq_diversity.mongodb.password =
# vmq_diversity.mongodb.database =
```

VerneMQ supports MongoDB's DNS SRV record lookup to fetch a seed list. Specify the hostname of hosted database as a `srv` option instead of `host` and `port`. VerneMQ will randomly choose a host/port combination from the seed list returned in the DNS SRV record. MongoDB SRV connections use TLS by default. You will need to configure TLS support for MongoDB for most SRV connections.

```text
vmq_diversity.auth_mongodb.enabled = on
vmq_diversity.mongodb.srv = vmqtest.08t1b.mongodb.net
vmq_diversity.mongodb.login = username
vmq_diversity.mongodb.password = secretpass
# vmq_diversity.mongodb.database =
```

MongoDB supports a number of node types in replica sets. The built-in MongoDB support simply connects to the host and port specified. It does not differentiate between primary or secondary nodes in MongoDB replica sets.

MongoDB hashing methods:

| method | client-side | server-side |
| :--- | :---: | :---: |
| bcrypt | ✓ |  |

Insert the ACL using the `mongo` shell or any software library. The `passhash` property contains the bcrypt hash of the clients password.

```javascript
db.vmq_acl_auth.insert({
    mountpoint: '',
    client_id: 'test-client',
    username: 'test-user',
    passhash: '$2a$12$WDzmynWSMRVzfszQkB2MsOWYQK9qGtfjVpO8iBdimTOjCK/u6CzJK',
    publish_acl: [
        {pattern: 'a/b/c'},
        {pattern: 'a/+/d'}
    ],
    subscribe_acl: [
        {pattern: 'a/#'}
    ]
})
```

## Redis

For Redis authentication and authorization configure the following in `vernemq.conf`:

```text
vmq_diversity.auth_redis.enabled = on
vmq_diversity.redis.host = 127.0.0.1
vmq_diversity.redis.port = 6379
# vmq_diversity.redis.user = "default"
# vmq_diversity.redis.password =
# vmq_divserity.redis.database = 0
```

Redis hashing methods:

| method | client-side | server-side |
| :--- | :---: | :---: |
| bcrypt | ✓ |  |

Insert the ACL using the `redis-cli` shell or any software library. The `passhash` property contains the bcrypt hash of the clients password. The key is an encoded JSON array containing the mountpoint, username, and client id. Note that no spaces are allowed between the array items.

```text
SET "[\"\",\"test-client\",\"test-user\"]" "{\"passhash\":\"$2a$12$WDzmynWSMRVzfszQkB2MsOWYQK9qGtfjVpO8iBdimTOjCK/u6CzJK\",\"subscribe_acl\":[{\"pattern\":\"a/+/c\"}]}"
```
