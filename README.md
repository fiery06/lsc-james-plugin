# James LSC plugin

[![Build Status](https://travis-ci.org/lsc-project/lsc-james-plugin.svg?branch=master)](https://travis-ci.org/lsc-project/lsc-james-plugin)

This a plugin for LSC, using James REST API


### Goal

The object of this plugin is to synchronize addresses aliases and users from one referential to a [James server](https://james.apache.org/).

### Address Aliases Synchronization

In the following example, LSC plugin will be used to synchronize the aliases stored in OBM's  LDAP server and James Server(s) of a TMail deployment.

#### Architecture

Given the following LDAP entry:
```
dn: uid=rkowalsky,ou=users,dc=linagora.com,dc=lng
[...]
mail: rkowalsky@linagora.com
mailAlias: remy.kowalsky@linagora.com
mailAlias: remy@linagora.com
```

The corresponding James address alias is:
```bash
$ curl -XGET http://ip:port/address/aliases/rkowalsky@linagora.com

[
  {"source":"remy.kowalsky@linagora.com"},
  {"source":"remy@linagora.com"}
]
```

Because the alias for addresses in James are only created if there are some sources, the LDAP entry without mailAlias attribute won't be synchronized.

The pivot used for the synchronization in LSC connector is email address. For this case,  `rkowalsky@linagora.com` is stored in `email` attribute.

The destination attribute for the LSC aliases connector is named `sources`.

### Users Synchronization

In the following example, LSC plugin will be used to synchronize the users stored in OBM's  LDAP server and James Server(s) of a TMail deployment.

#### Architecture
Given the following LDAP entries:

```
dn: uid=james-user, ou=people, dc=james,dc=org
mail: james-user@james.org
[...]

dn: uid=james-user2, ou=people, dc=james,dc=org
mail: james-user2@james.org
[...]

dn: uid=james-user3, ou=people, dc=james,dc=org
mail: james-user3@james.org
[...]
```

This will be represented as the following James users:  

```bash
$ curl -XGET http://ip:port/users

[
  {"username":"james-user2@james.org"},
  {"username":"james-user4@james.org"}
]
```

If LDAP entry with the `mail` attribute exists but not synchronized, the user will be `created` with a random generated password (24 characters).

If LDAP entry has no `mail` attribute, the user will be `deleted`.

Expected Result:

    - james-user@james.org -> create
    - james-user2@james.org -> nothing happens
    - james-user3@james.org -> create
    - james-user4@james.org -> delete

```bash 
$ curl -XGET http://ip:port/users

[
  {"username":"james-user@james.org"},
  {"username":"james-user2@james.org"},
  {"username":"james-user3@james.org"}
]
```

The pivot used for the synchronization in LSC connector is email address. For this case,  `james-user@james.org` is stored in `email` attribute.


### Configuration

The plugin connection requires a JWT token to connect to James. Set the `password` field of the plugin connection with the JWT token you want to use.

The `url` field of the plugin connection must be set to the URL of James's webadmin.

The `username` field of the plugin is ignored for now.

### Usage

Configuration examples can be found in `sample` directory. The `lsc.xml` file describe a synchronization from an OBM LDAP to a James server.
Available parameters are:
  - `connections.ldapConnection.url`: URL of OBM's LDAP Server.
  - `connections.ldapConnection.username`: LDAP Username which able to read the OBM aliases.
  - `connections.ldapConnection.password`: Password of this user.
  - `connections.pluginConnection.url`: URL of James's WebAdmin.
  - `connections.pluginConnection.password`: The JWT token used to connect to James WebAdmin, must included an admin claim.

  - `tasks.task.ldapSourceService.baseDn`: The search base of the users to synchronize.
  
  
The domain used in the aliases must already created in James.
Otherwise none of the aliases will be added, as user has a single alias pointing to an unknown domain.

James LSC plugin `.jar` must be placed in `lib` directory of your LSC installation.
Launch it with the following command:

```bash
JAVA_OPTS="-DLSC.PLUGINS.PACKAGEPATH=org.lsc.plugins.connectors.james.generated" bin/lsc --config /home/rkowalski/Documents/lsc-james-plugin/sample/ldap-to-james/ --synchronize all --clean all --threads 1
```

If don't want to delete dangling data, remove `--clean all` parameter.

### Packaging

WIP
