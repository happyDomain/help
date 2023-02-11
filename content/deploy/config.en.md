---
data: 2023-02-11T11:58:55+01:00
title: Configuration
weight: 20
---

happyDomain respects the methodology [*12 factor*](https://12factor.net/) and allows to act on the application configuration in several ways.

## How do I configure happyDomain?

It is possible to configure happyDomain in three different ways: configuration file, environment, command line. All options are available for each of these mechanisms.

The precedence, when an option is defined by several mechanisms simultaneously, is that an option present in a configuration file will be overwritten by the environment, which will be overwritten by an option passed on the command line


### Configuration by file

When the application is launched, the first configuration file from the following list will be used:

- `./happydomain.conf`
- `$XDG_CONFIG_HOME/happydomain/happydomain.conf`
- `/etc/happydomain.conf`

**Only the first existing file is taken into account.** It is not possible to have part of its options in `/etc/happydomain.conf` and part in `./happydomain.conf`, only the latter configuration file will be taken into account.

It is possible to specify a custom path by adding it as an additional parameter to the command line. Thus, to use the configuration file located at `/etc/happydomain/config`, we would use :

```
./happydomain /etc/happydomain/config
```

#### Configuration file format

Comments line has to begin with `#`, it is not possible to have comments at the end of a line, by appending `#` followed by a comment.

Place on each line the name of the config option and the expected value, separated by `=`. For example:

```
storage-engine=leveldb
leveldb-path=/var/lib/happydomain/db/
```

### Configuration by the environment

When happyDomain is started, all variables beginning with `HAPPYDOMAIN_` are scanned for valid configuration options.

You can do the same thing as in the previous example, with the following environment variables:

```
HAPPYDOMAIN_STORAGE_ENGINE=leveldb
HAPPYDOMAIN_LEVELDB_PATH=/var/lib/happydomain/db/
```

You just have to replace dash by underscore.


### Command line configuration

Finally, the command line can be used to pass options, according to the usual UNIX format.

To continue the previous example, we can perform the same configuration with the following command line:

```
./happydomain -storage-engine leveldb -leveldb-path /var/lib/happydomain/db/
```

or by using the `=` sign to clearly assign the value.

```
./happydomain -storage-engine=leveldb -leveldb-path=/var/lib/happydomain/db/
```


## Configuration items

The complete list of configurable items can be listed by calling `happyDomain` with the `-h` or `--help` option.

Here is a list of the main options:

### General parameters

`bind`
: Bind port/socket to use to expose happyDomain.

`admin-bind`
: Bind port/socket to use to expose the administration API.

`default-ns`
: Address and port of the name resolver server to be used by default when name resolution is required.

`dev`
: URL to which all requests related to the graphical interface will be returned.

`externalurl`
: URL of the service, as it should appear in emails and content to the public.


#### Page layout

`custom-head-html`
: String to be placed before the end of the HTML header.

`custom-body-html`
: String to be placed before the end of the HTML body.


### Data storage

`storage-engine`
: Allows you to choose the data storage mechanism among all supported mechanisms.

### LevelDB (`storage-engine=leveldb`)

`leveldb-path`
: Path to the folder containing the LevelDB database to use.

### E-Mail parameters

We use [`go-mail`](https://github.com/go-mail/mail) as a library to send mails.

`mail-from`
: Defines the name and address of the sender of emails sent by the service.

Note that without the `mail-smtp-*` options, happyDomain will use the `sendmail` binary to send mail. This can be coupled with the `msmtp` or `ssmtp` packages, for example, to set the parameters for the whole system.

`mail-smtp-host`
: IP or host name of the SMTP server to use.

`mail-smtp-port`
: Port to use on the remote server.

`mail-smtp-username`
: When authentication is required on the remote server, username to use.

`mail-smtp-password`
: When authentication is required on the remote server, password to use.


### Authentication

`no-auth`
: Disables the notion of users and access control. A default account is used.

`external-auth`
: URL base of the authentication and registration service to be used instead of the embedded login system.

`jwt-secret-key`
: Secret key used to verify JWT tokens.


### Specific to registrars

Some registrars require third-party applications to identify themselves in addition to the user.

#### OVH

Please refer to this documentation in order to generate application credentials: <https://docs.ovh.com/gb/en/api/api-rights-delegation/#application-registration>.

`ovh-application-key`
: Application key for OVH API.

`ovh-application-secret`
: Secret key for OVH API.
