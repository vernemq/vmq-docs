---
description: >-
  VerneMQ supports multiple ways to authenticate and authorize new client
  connections.
---

# Auth using a database

## Authentication and authorization using a database

{% hint style="info" %}
This functionality is still in BETA and some things may change if required.
{% endhint %}

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

### PostgreSQL

To enable PostgreSQL authentication and authorization the following need to be configured in the `vernemq.conf` file:

```text
vmq_diversity.auth_postgres.enabled = on
vmq_diversity.postgres.host = 127.0.0.1
vmq_diversity.postgres.port = 5432
vmq_diversity.postgres.user = vernemq
vmq_diversity.postgres.password = vernemq
vmq_diversity.postgres.database = vernemq_db
```

The following SQL DDL must be applied, the `pgcrypto` extension is required:

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

### MySQL

For MySQL authentication and authorization configure the following in `vernemq.conf`:

```text
vmq_diversity.auth_mysql.enabled = on
vmq_diversity.mysql.host = 127.0.0.1
vmq_diversity.mysql.port = 3306
vmq_diversity.mysql.user = vernemq
vmq_diversity.mysql.password = vernemq
vmq_diversity.mysql.database = vernemq_db
```

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

To enter new ACL entries use a query similar to the following:

```sql
INSERT INTO vmq_auth_acl 
    (mountpoint, client_id, username, 
     password, publish_acl, subscribe_acl)
VALUES 
    ('', 'test-client', 'test-user', PASSWORD('123'), 
     '[{"pattern":"a/b/c"},{"pattern":"c/b/#"}]', 
     '[{"pattern":"a/b/c"},{"pattern":"c/b/#"}]');
```

### MongoDB

For MongoDB authentication and authorization configure the following in `vernemq.conf`:

```text
vmq_diversity.auth_mongodb.enabled = on
vmq_diversity.mongodb.host = 127.0.0.1
vmq_diversity.mongodb.port = 27017
# vmq_diversity.mongodb.login =
# vmq_diversity.mongodb.password = 
# vmq_diversity.mongodb.database =
```

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

### Redis

For Redis authentication and authorization configure the following in `vernemq.conf`:

```text
vmq_diversity.auth_redis.enabled = on
vmq_diversity.redis.host = 127.0.0.1
vmq_diversity.redis.port = 6379
# vmq_diversity.redis.password = 
# vmq_divserity.redis.database = 0
```

Insert the ACL using the `redis-cli` shell or any software library. The `passhash` property contains the bcrypt hash of the clients password. The key is an encoded JSON array containing the mountpoint, username, and client id. Note that no spaces are allowed between the array items.

```text
SET "[\"\",\"test-client\",\"test-user\"]" "{\"passhash\":\"$2a$12$WDzmynWSMRVzfszQkB2MsOWYQK9qGtfjVpO8iBdimTOjCK/u6CzJK\",\"subscribe_acl\":[{\"pattern\":\"a/+/c\"}]}"
```

Note, currently bcrypt version `2a` \(prefix `$2a$`\) is supported.

