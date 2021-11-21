## Security

### Authentication

MongoDB supports four different authentication methods.

#### SCRAM

This is the default and most basic form of authentication.

Stands for Salted, Challenge, Respond, Authentication Mechanism, with the important part being challenge and response (aka username and password)

#### X.509

Uses an X.509 certificate for authentication, this is a more secure although more complex to setup method of authentication.

#### LDAP (Lightweight Directory Access Protocol) - Enterprise Edition Only

Microsoft Active Directory is built on LDAP.

#### KERBEROS

Robust authentication protocol.

### Authorisation

MongoDB uses RBAC for authorisation. Each user has one or more roles, each role has one of more privileges, a privilege represents a group of actions and the resources that those actions apply to.

A resource in terms of RBAC can be defined as:

- Specific database and a specific collection
- All databases and all collections
- Specific database and any collection
- Cluster resource

Roles can inherit from other roles.

You can restrict roles with network authentication restrictions, specifying which roles can access from which addresses.

At the very minimum, you should always configure SCRAM-SHA-1 with a single administrative user protected by a strong password.

For simplicity, you should `use admin` and add the users in that database but give the user permissions on different databases, that way the `admin` database can be the single point for authentication.

#### Built in Roles

- Database user (application users)
  - `read`
  - `readWrite`
  - `readAnyDatabase` (cross database role)
  - `readWriteAnyDatabase` (cross database role)
- Database administrator
  - `dbAdmin`
  - `dbAdminAnyDatabase` (cross database role)
  - `userAdmin`
  - `userAdminAnyDatabase` (cross database role)
  - `dbOwner`
- Cluster administrator
  - `clusterAdmin`
  - `clusterManager`
  - `clusterMonitor`
  - `hostManager`
- Backup/restore
  - `backup`
  - `restore`
- Super user
  - `root` (cross database role)

The `dbOwner` role can preform any administrative action on the database. This role combines the privileges granted by the `readWrite`, `dbAdmin` and `userAdmin` roles.

#### Localhost Exception

Allows you to access a MongoDB server that has authentication but does not have any configured users. The shell must be ran from the same host as the server, this loophole is closed as soon as you create your first user.

Always create a user with administrator privileges first.
